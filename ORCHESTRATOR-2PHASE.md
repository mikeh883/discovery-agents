# Content Monitor Orchestrator (2-Phase Architecture)

**Purpose:** Coordinate content discovery with prep + execution pattern  
**Model Strategy:**
- **Prep Agents:** Gemini 2.5 Flash-Lite (planning, validation)
- **Execution Agents:** Gemini 2.5 Flash-Lite (RSS fetch, scoring, notes)
- **Synthesis Agent:** Gemini 2.5 Pro (cross-source digest with 2M context)
**Runtime:** ~15-20 minutes (prep + execution + synthesis)

## Why 2-Phase?

**Vibe Engineering Principles:**
- Smaller, focused agents (easier to debug)
- Clear separation: read vs write operations  
- Better error isolation (prep failure ≠ execution failure)
- Validation gates between phases
- Autonomous decision-making within bounded scope

## Architecture

### Phase 1: Discovery (Per-Topic)
For each of 8 topics, run sequentially:
1. **Prep agent** → validates sources, creates execution plan
2. **Execution agent** → fetches RSS, scores, creates notes, generates manifest

### Phase 2: Synthesis (Cross-Topic)
After all 8 topics complete:
1. **Synthesis agent** → loads all manifests, generates 3-layer digest, posts to Discord

### Phase 3: Cleanup
Orchestrator aggregates stats and returns summary

## Implementation

### Phase 1a: Prep Agent (per topic)

```javascript
sessions_spawn({
  task: `
    DISCOVERY PREP AGENT for: ${topic}
    
    Your job: Validate sources and create execution plan. DO NOT fetch RSS yet.
    
    PROCESS:
    1. Read discoveries/sources/${topic}.md
    2. Extract all sources
    3. For each source:
       - Identify platform (youtube/substack/twitter/rss)
       - Extract creator name
       - Find RSS URL
       - Validate URL format (don't fetch, just check syntax)
    4. Count totals
    
    OUTPUT (save as JSON):
    {
      "topic": "${topic}",
      "timestamp": "[ISO]",
      "sources": [
        {
          "platform": "youtube",
          "creator": "Yannic Kilcher",
          "rssUrl": "https://www.youtube.com/feeds/videos.xml?channel_id=...",
          "valid": true
        }
      ],
      "stats": {
        "totalSources": 15,
        "validUrls": 12,
        "invalidEntries": 3
      }
    }
    
    Save to: discoveries/meta/.plans/${timestamp}-${topic}-plan.json
    Return: {"planPath": "...", "validUrls": 12, "invalidEntries": 3}
  `,
  label: `prep-${topic}`,
  model: 'google/gemini-2.5-flash-lite',
  runTimeoutSeconds: 120
})
```

### Phase 1b: Execution Agent (per topic)

```javascript
// Wait for prep to complete, then spawn execution
sessions_spawn({
  task: `
    DISCOVERY EXECUTION AGENT for: ${topic}
    
    INPUT: Plan from discoveries/meta/.plans/${timestamp}-${topic}-plan.json
    
    PROCESS:
    
    1. LOAD PLAN
       - Read ${timestamp}-${topic}-plan.json
       - Get list of valid RSS URLs and metadata
    
    2. FETCH RSS (with retry)
       - For each validUrl in plan:
         * Use web_fetch(url, {extractMode: 'text'})
         * If fails: wait 5sec, retry once
         * If still fails: log error, mark failed, CONTINUE
       - Parse XML: title, link, pubDate, description
       - Track: successful fetches, failed fetches
    
    3. FILTER NEW CONTENT
       - Load discoveries/.discoveries-tracking.json (if missing, assume {})
       - For each RSS item:
         * Skip if URL in tracking
         * Skip if pubDate > 24h ago
         * Add to newContent list
    
    4. SCORE RELEVANCE (batch 5-10 items)
       - For each new item:
         * Summarize (2-3 sentences from title + description)
         * Score 0-100:
           - Topic match to ${topic}
           - Novelty (unique angle vs obvious)
           - Actionability (user can do something)
         * Output: {score, reasoning, keyPoints[3-5], tags[2-4]}
    
    5. CREATE NOTES (for 60+ scores)
       - Path: discoveries/[platform]/YYYY-MM-DD-[slug].md
       - Template:
         ---
         source: [platform]
         creator: [creator]
         topic: [${topic}]
         discovered: [ISO]
         relevance: [score]
         url: [URL]
         ---
         
         # [Title]
         
         [Summary]
         
         ## Key Points
         [Bullets]
         
         ## Source
         [Creator] • [Platform] • [Date]
         [URL]
       - Track created paths
    
    6. UPDATE TRACKING
       - Load .discoveries-tracking.json
       - Add {"[url]": "[timestamp]"} for all new items
       - Save back
    
    7. CREATE MANIFEST
       - Path: discoveries/meta/.manifests/${timestamp}-${topic}.json
       - Content:
         {
           "topic": "${topic}",
           "timestamp": "[ISO]",
           "discoveries": [
             {
               "title": "...",
               "url": "...",
               "creator": "...",
               "platform": "youtube|substack|twitter|rss",
               "relevance": 85,
               "tags": ["tag1", "tag2"],
               "keyPoints": ["...", "...", "..."],
               "reasoning": "...",
               "notePath": "discoveries/platform/YYYY-MM-DD-slug.md"
             }
           ],
           "stats": {
             "feedsAttempted": 12,
             "feedsSuccessful": 10,
             "feedsFailed": 2,
             "failedUrls": ["...", "..."],
             "newItems": 8,
             "notesCreated": 3,
             "avgRelevance": 72
           }
         }
    
    DECISION RULES:
    - RSS fail after retry → skip, mark in failedUrls
    - Tracking file missing → create new
    - Note write fail → log in manifest with "writeError": true
    - Score LLM fail → use keyword match (score 40-50), note in reasoning
    
    OUTPUT (return this):
    {
      "manifestPath": "discoveries/meta/.manifests/${timestamp}-${topic}.json",
      "stats": {
        "feedsAttempted": 12,
        "feedsSuccessful": 10,
        "newItems": 8,
        "notesCreated": 3,
        "avgRelevance": 72
      },
      "issues": ["Feed X failed (404)", "Note write failed: path"]
    }
  `,
  label: `execute-${topic}`,
  model: 'google/gemini-2.5-flash-lite',
  runTimeoutSeconds: 600
})
```

### Phase 1 Orchestration Logic

```javascript
// For each topic, run prep → execution sequentially
for (const topic of topics) {
  // Phase 1a: Prep
  const prepResult = await sessions_spawn({...prepAgent...})
  
  // Check prep result
  if (prepResult.validUrls === 0) {
    log(`Topic ${topic}: no valid URLs, skipping execution`)
    continue
  }
  
  // Phase 1b: Execution
  const execResult = await sessions_spawn({...executionAgent...})
  
  // Collect manifest path
  manifests.push(execResult.manifestPath)
}
```

### Phase 2: Synthesis Agent

```javascript
// After all 8 topics complete
sessions_spawn({
  task: `
    SYNTHESIS AGENT - Generate 3-layer digest
    
    INPUT:
    - All manifests: discoveries/meta/.manifests/${timestamp}-*.json
    - Format spec: discoveries/meta/digest-format-spec.md
    - Engagement: discoveries/.discoveries-engagement.json
    
    PROCESS:
    1. Load all 8 topic manifests
    2. Aggregate discoveries (~10-30 items total)
    3. Cross-source analysis:
       - Consensus: What do multiple sources agree on?
       - Conflicts: Where do platforms disagree?
       - Themes: Meta-patterns across topics
       - Sentiment: Per-platform tone
       - Urgency: 1-5 scores (impact + timeliness)
    
    4. Generate 3-layer digest:
       - Layer 1: Signal (headline + 3 findings + urgency)
       - Layer 2: Synthesis (consensus, themes, sentiment heatmap)
       - Layer 3: Evidence (all discoveries with quotes)
    
    5. Save: discoveries/digests/${timestamp}-digest.md
    
    6. Post to Discord #digest (ID: 1476043075355938880):
       - Layer 1 full
       - Quick synthesis (1-2 sentences from Layer 2)
       - Top 2-3 evidences from Layer 3
       - Link to full digest
    
    Follow discoveries/meta/digest-format-spec.md EXACTLY.
    
    OUTPUT:
    {
      "digestPath": "...",
      "discordMessageId": "...",
      "stats": {
        "totalDiscoveries": 26,
        "topics": 8,
        "urgency": 3
      }
    }
  `,
  label: 'synthesize-digest',
  model: 'google/gemini-2.5-pro',  // 2M context
  runTimeoutSeconds: 300
})
```

### Phase 3: Return Summary

```javascript
return {
  "run": {
    "timestamp": "2026-02-25T21:30:00Z",
    "durationMs": 980000
  },
  "phase1": {
    "topicsProcessed": 8,
    "prepSuccessful": 8,
    "executionSuccessful": 7,
    "executionFailed": 1,
    "totalDiscoveries": 26,
    "manifests": [...]
  },
  "phase2": {
    "digestPath": "...",
    "discordPosted": true
  }
}
```

## Error Recovery

**Prep failure:**
- Log which topic
- Skip execution for that topic
- Continue with other topics

**Execution failure:**
- Retry once
- If still fails: create empty manifest with error note
- Continue with synthesis (will process remaining manifests)

**Synthesis failure:**
- Create fallback digest (simple list by score)
- Post to Discord with error note
- Full synthesis will retry next run

## Testing

**Single topic test:**
```javascript
// Test prep
const prep = await sessions_spawn({task: prepAgent('openclaw'), ...})

// Test execution  
const exec = await sessions_spawn({task: executionAgent('openclaw'), ...})

// Test synthesis
const syn = await sessions_spawn({task: synthesisAgent, ...})
```

**Full orchestration:**
Run all 8 topics + synthesis, validate complete workflow

---

**Status:** 2-phase architecture ready for production
**Next:** Test one topic end-to-end, then enable cron
