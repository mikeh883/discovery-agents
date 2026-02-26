# Content Monitor Orchestrator

**Purpose:** Coordinate the two-phase hybrid content discovery system  
**Model Strategy:**
- **Development/Orchestration:** Claude Sonnet 4.5 (this session)
- **Discovery Agents (8x):** Gemini Flash 2.0 (cheap, fast, high throughput)
- **Synthesis Agent (1x):** Gemini Pro 1.5 (1M context for cross-source analysis)
**Runtime:** ~10-15 minutes

## Overview

This is the main entry point called by the cron job. It coordinates:
1. **Phase 1:** 8 parallel Claude agents for discovery (one per topic)
2. **Phase 2:** 1 Gemini agent for synthesis (cross-source digest)
3. **Phase 3:** Discord notification and cleanup

## Implementation

### Phase 1: Spawn Discovery Agents

**Topics to process:**
```
1. llm-architecture
2. ai-news
3. ai-model-releases
4. agents
5. ai-frameworks
6. vibe-coding
7. openclaw
8. local-llm
```

**For each topic, spawn a sub-agent:**

```javascript
// Topic-specific discovery agent
sessions_spawn({
  task: `
    You are a topic-specific discovery agent for: ${topic}
    
    CRITICAL: You are CREATING a new manifest file, not reading an existing one.
    
    STEP-BY-STEP PROCESS:
    
    1. READ SOURCE LIST
       - File: discoveries/sources/${topic}.md
       - Extract all RSS feed URLs
       - Count total sources
    
    2. FETCH RSS FEEDS (with retry logic)
       - For each URL, use web_fetch with extractMode: 'text'
       - If fetch fails (404/timeout):
         * Retry once after 5 seconds
         * If still fails: Log error, mark source as failed, CONTINUE to next feed
       - Parse XML to extract: title, link, pubDate, description
       - Track: successful fetches, failed fetches
    
    3. FILTER NEW CONTENT
       - Load discoveries/.discoveries-tracking.json (if exists, otherwise assume empty)
       - For each RSS item:
         * If URL already in tracking → skip
         * If pubDate > 24 hours ago → skip
         * Otherwise → add to "newContent" list
    
    4. SCORE RELEVANCE (batch processing)
       - For each new item (batch 5-10 at a time):
         * Generate 2-3 sentence summary from title + description
         * Score 0-100 based on:
           - Topic match to ${topic}
           - Novelty (unique angle vs obvious)
           - Actionability (can user do something with this)
         * Provide: score, reasoning (1-2 sentences), keyPoints (3-5 bullets), tags (2-4)
    
    5. CREATE MINIMAL OBSIDIAN NOTES (for items 60+)
       - File path: discoveries/[platform]/YYYY-MM-DD-[slug-from-title].md
       - Content template:
         ---
         source: [platform]
         creator: [creator]
         topic: [${topic}]
         discovered: [ISO timestamp]
         relevance: [score]
         url: [URL]
         ---
         
         # [Title]
         
         [2-3 sentence summary]
         
         ## Key Points
         [Bullet list from scoring]
         
         ## Source
         [Creator] • [Platform] • [Date]
         [URL]
       
       - Track created note paths for manifest
    
    6. UPDATE TRACKING
       - Load discoveries/.discoveries-tracking.json
       - Add all new URLs with current timestamp
       - Write back to file
    
    7. CREATE MANIFEST (this is a NEW file you're creating)
       - Path: discoveries/meta/.manifests/${timestamp}-${topic}.json
       - Content:
         {
           "topic": "${topic}",
           "timestamp": "[ISO timestamp when run started]",
           "discoveries": [
             {
               "title": "...",
               "url": "...",
               "creator": "...",
               "platform": "youtube|substack|twitter|rss",
               "relevance": 85,
               "tags": ["tag1", "tag2"],
               "keyPoints": ["point1", "point2", "point3"],
               "reasoning": "why this matters",
               "notePath": "discoveries/platform/YYYY-MM-DD-slug.md"
             }
           ],
           "stats": {
             "sourcesTotal": 15,
             "feedsChecked": 12,
             "feedsFailed": 3,
             "failedSources": ["URL1", "URL2"],
             "newContent": 8,
             "addedToVault": 3,
             "avgRelevance": 72
           }
         }
    
    DECISION MAKING:
    - If RSS feed fails after retry → Skip and continue (mark in failedSources)
    - If .discoveries-tracking.json doesn't exist → Assume empty, create new
    - If scoring LLM call fails → Use simple keyword match (score 40-50), note in reasoning
    - If note write fails → Log path to manifest but mark with "writeError": true
    
    VALIDATION CHECKPOINTS:
    - After step 2: Confirm you have RSS data (even if some failed)
    - After step 3: Confirm you have newContent list (can be empty)
    - After step 4: Confirm you have scored items
    - After step 7: Confirm manifest file was created
    
    RETURN FORMAT:
    {
      "manifestPath": "discoveries/meta/.manifests/[timestamp]-${topic}.json",
      "stats": {
        "sourcesTotal": 15,
        "feedsSuccessful": 12,
        "feedsFailed": 3,
        "newItems": 8,
        "notesCreated": 3,
        "avgRelevance": 72
      },
      "issues": [
        "Feed failed: URL (404)",
        "Note write failed: path"
      ]
    }
  `,
  label: `discover-${topic}`,
  model: 'google/gemini-2.5-flash-lite',
  runTimeoutSeconds: 600
})
```

**Wait for all 8 agents to complete:**
- Use `subagents(action='list')` to check status
- Timeout: 10 minutes total
- Collect manifest paths from results

### Phase 2: Spawn Synthesis Agent

**After all discovery agents complete:**

```javascript
sessions_spawn({
  task: `
    You are the cross-source synthesis agent. Generate a 3-layer digest.
    
    Input:
    - All manifests from discoveries/meta/.manifests/${timestamp}-*.json
    - Format spec: discoveries/meta/digest-format-spec.md
    - Engagement history: discoveries/.discoveries-engagement.json
    
    Process:
    1. Load all 8 topic manifests
    2. Aggregate all discoveries (typically 10-30 items total)
    3. Perform cross-source analysis:
       - Consensus: What do multiple sources agree on?
       - Conflicts: Where do platforms disagree?
       - Themes: What are the meta-patterns?
       - Sentiment: Per-platform emotional tone
       - Urgency: Calculate 1-5 scores based on impact/timeliness
    
    4. Generate 3-layer digest:
       - Layer 1 (The Signal): Single headline + 3 key findings + urgency
       - Layer 2 (The Synthesis): Consensus, themes, sentiment heatmap
       - Layer 3 (The Evidence): All discoveries with golden quotes
    
    5. Save to: discoveries/digests/${timestamp}-digest.md
    
    6. Post to Discord #digest (ID: 1476043075355938880):
       - Layer 1 (The Signal) - full
       - Quick synthesis (1-2 sentences from Layer 2)
       - Top 2 evidences from Layer 3
       - Link to full digest
    
    Follow discoveries/meta/digest-format-spec.md EXACTLY.
    
    Return: Path to digest file
  `,
  label: 'synthesize-digest',
  model: 'google/gemini-2.5-pro',  // 2M context for cross-source synthesis
  runTimeoutSeconds: 300  // 5 min for synthesis
})
```

### Phase 3: Aggregate and Report

**Collect results from all phases:**

```javascript
{
  "run": {
    "timestamp": "2026-02-25T14:30:00Z",
    "duration_ms": 845000
  },
  "phase1": {
    "topics_processed": 8,
    "topics_failed": 0,
    "total_feeds_checked": 87,
    "total_new_content": 23,
    "total_discoveries_added": 12,
    "manifests": [
      "discoveries/meta/.manifests/2026-02-25-1430-llm-architecture.json",
      // ... 7 more
    ]
  },
  "phase2": {
    "model": "google/gemini-3.1-pro",
    "synthesis_complete": true,
    "digest_path": "discoveries/digests/2026-02-25-1430-digest.md",
    "discord_posted": true
  }
}
```

**Return summary to user:**
```
✅ Content monitoring complete!

**Phase 1 (Discovery):**
- 8 topics processed
- 87 RSS feeds checked
- 23 new items found
- 12 added to vault (60+ relevance)

**Phase 2 (Synthesis):**
- Cross-source analysis complete
- 3-layer digest generated
- Discord notification sent

**Digest:** discoveries/digests/2026-02-25-1430-digest.md
**Discord:** #digest channel

**Next run:** 6 PM CST
```

## Error Recovery

### Phase 1 Failures
If any topic agent fails:
- Log the error with topic name
- Continue with successful topics
- Phase 2 synthesizes what's available
- Note missing topics in final summary

### Phase 2 Failure
If synthesis agent fails:
- Create fallback simple digest (no cross-source analysis)
- List all discoveries by relevance score
- Post to Discord with error notice
- Full synthesis will retry next run

### Complete Failure
If orchestrator itself fails:
- Cron will retry on next scheduled run (6 hours)
- No data loss (all tracking files persist)
- User gets error notification in Discord

## Optimization Notes

### Parallel Processing
- Phase 1: All 8 topics run simultaneously
- Estimated time: ~5-7 minutes (longest topic)
- Each topic independent (no blocking)

### Context Management
- Phase 1 agents: Small context (single topic, ~10-15 sources)
- Phase 2 agent: Large context (all discoveries, ~10-30 items)
- Gemini 1M context handles full cross-source synthesis

### Cost Efficiency
- Claude for quality relevance scoring (Phase 1)
- Gemini for bulk synthesis (Phase 2, cheaper)
- Estimated cost per run: ~$0.05-0.10

## Testing

### Manual Test Run
```javascript
// In main session, trigger orchestrator
sessions_spawn({
  task: "Run content monitoring orchestrator. Follow projects/discovery-agents/ORCHESTRATOR.md",
  label: "content-monitor-test",
  runTimeoutSeconds: 1200
})
```

### Validation
After test run, check:
1. ✅ All 8 manifests created in `.manifests/`
2. ✅ New discoveries in vault (discoveries/*/YYYY-MM-DD-*.md)
3. ✅ Digest file created with 3 layers
4. ✅ Discord post in #digest channel
5. ✅ Tracking JSON updated

## Cron Integration

Update cron job prompts to reference this orchestrator:

```javascript
{
  "name": "Content Monitor",
  "schedule": {
    "kind": "cron",
    "expr": "0 */6 * * *",  // Every 6 hours
    "tz": "America/Chicago"
  },
  "payload": {
    "kind": "agentTurn",
    "message": "Run content monitoring orchestrator. Follow /Users/mikehollowaydev/Documents/Obsidian Vault/projects/discovery-agents/ORCHESTRATOR.md exactly. Return summary with digest link.",
    "model": "google/gemini-2.5-flash-lite",  // Budget orchestrator
    "timeoutSeconds": 1200
  },
  "sessionTarget": "isolated",
  "enabled": true,
  "notify": false
}
```

---

**Status:** Ready for implementation
**Next:** Test manual run, then enable cron jobs
