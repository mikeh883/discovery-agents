# Content Monitor Agent Specification

**Purpose:** Check source lists for new content, filter by relevance, add to Obsidian  
**Frequency:** Every 6 hours (6 AM, 12 PM, 6 PM, 12 AM CST)  
**Runtime:** ~10-20 min per run

## Agent Definition

This is a **two-phase hybrid architecture**:
- **Phase 1 (Discovery):** 8 parallel Claude sub-agents, one per topic
- **Phase 2 (Synthesis):** 1 Gemini 3.1 Pro sub-agent for cross-source digest
- Runs every 6 hours via cron
- Returns discovery summary + digest link

## Why Hybrid?

**Gemini Flash 2.0 for Discovery (Phase 1):**
- Very cheap (~$0.075 per 1M tokens input)
- High rate limits (handles 5-8 parallel agents)
- Fast RSS parsing and relevance scoring
- Good enough for structured tasks
- Avoids Claude Pro rate limits

**Gemini Pro 1.5 for Synthesis (Phase 2):**
- 1M token context window
- Can see ALL discoveries across ALL topics simultaneously
- Critical for Layer 2 (consensus/conflicts/themes across sources)
- Cheap bulk processing (~$1.25 per 1M tokens input)
- Superior cross-source pattern detection

**Cost per run:** ~$0.01-0.02 vs $0.05+ with Claude

## Input

**Source Lists:** All files in `discoveries/sources/*.md`
**Tracking:** `.discoveries-tracking.json` (seen content)
**Engagement:** `.discoveries-engagement.json` (user feedback)

## Process Overview

**Phase 1:** Discovery (8 parallel Claude agents, one per topic)  
**Phase 2:** Synthesis (1 Gemini agent, cross-source digest)  
**Phase 3:** Delivery (Discord notification)

---

## PHASE 1: Discovery (Per-Topic Claude Agents)

### Orchestrator Task
The main cron job spawns 8 parallel sub-agents, one for each topic:
1. LLM Architecture
2. AI News
3. AI Model Releases
4. Agents
5. AI Frameworks
6. Vibe Coding
7. OpenClaw
8. Local LLM

Each agent runs independently and produces a discovery manifest.

### Per-Topic Agent Process

#### Step 1: Load Topic Source List
Read single topic file from `discoveries/sources/[topic].md`:
- Extract RSS feed URLs for this topic only
- ~10-15 feeds per topic

#### Step 2: Fetch RSS Feeds
For each feed:
```javascript
const feed = await web_fetch(rssUrl, { extractMode: 'text' })
// Parse XML → extract entries
// Fields: title, link, pubDate, description
```

**Batch processing:**
- Fetch feeds sequentially or 3-5 in parallel
- Timeout: 30 sec per feed
- Skip if timeout or error (log it)

#### Step 3: Filter New Content
Load `.discoveries-tracking.json`:
```json
{
  "seen": {
    "https://youtube.com/watch?v=xyz": "2026-02-24T12:00:00Z",
    "https://example.substack.com/p/post": "2026-02-23T08:00:00Z"
  }
}
```

For each RSS entry:
- If `entry.link` in seen → skip
- If `entry.pubDate` > 24 hours ago → add to new content list
- Otherwise → skip (too old for first-time discovery)

**New content list:** ~2-5 items per topic per run

#### Step 4: Relevance Scoring (LLM Batch)
For each new content item:

**Prompt:**
```
Content:
Title: [title]
Source: [creator/channel name]
Description: [RSS description or first paragraph]
Published: [date]
Topic: [topic this source is tagged with]

User's interest areas:
- LLM Architecture
- AI News
- AI Model Releases
- Agents
- AI Frameworks
- Vibe Coding
- OpenClaw
- Local LLM

User's recent Obsidian activity:
[Last 10 note titles from vault]

User's engagement history:
[Top 5 engaged-with topics/creators]

---

Score this content's relevance to the user's interests (0-100).

Consider:
1. Direct topic match (e.g., "transformers" → LLM Architecture)
2. Related to active projects (e.g., recent notes about X)
3. Quality signals (source reputation, engagement if available)
4. Novelty (unique angle, not obvious/repeated info)
5. Actionability (can user do something with this?)

Provide:
- **Score:** 0-100
- **Primary Topic:** [which of the 8 topics]
- **Tags:** [2-4 relevant tags]
- **Reasoning:** [1-2 sentences]
- **Key Points:** [3-5 bullet points of main content]

Threshold for auto-add: 60+
```

**Batch scoring:**
- Process 5-10 items per LLM call (cheaper, faster)
- Return structured JSON

#### Step 5: Auto-Add to Obsidian
For items scoring 60+:

**Create note:**
```
Path: discoveries/[platform]/[date]-[slug].md

Content:
---
source: [platform]
creator: [creator name]
topic: [Primary Topic, Secondary Topic]
discovered: [ISO timestamp]
relevance: [score]
url: [link]
---

# [Title]

> [One-line description]

## Summary
[3-4 sentence summary from LLM]

## Key Points
[Bullet list from LLM scoring]

## Why This Matters
[LLM reasoning about relevance]

## Related Notes
[Search vault for related notes, suggest links]
- [[Potential related note 1]]
- [[Potential related note 2]]

## Source
**Creator:** [Name]
**Platform:** [YouTube/Substack/Twitter/RSS]
**Published:** [Date]
**Link:** [URL]

---

*Discovered by Content Monitor Agent*
*Relevance Score: [score]/100*
*Topic: [Primary Topic]*
```

**Update tracking:**
Add to `.discoveries-tracking.json`:
```json
{
  "seen": {
    "[url]": "[timestamp]",
    ...
  }
}
```

#### Step 6: Generate Topic Manifest
Each topic agent creates a manifest file for Phase 2 synthesis:

**Path:** `discoveries/meta/.manifests/[date]-[time]-[topic].json`

**Content:**
```json
{
  "topic": "LLM Architecture",
  "timestamp": "2026-02-25T14:30:00Z",
  "discoveries": [
    {
      "title": "...",
      "url": "...",
      "creator": "...",
      "platform": "youtube|substack|twitter|rss",
      "relevance": 85,
      "tags": ["transformers", "attention"],
      "keyPoints": ["...", "...", "..."],
      "reasoning": "...",
      "notePath": "discoveries/youtube/2026-02-25-..."
    }
  ],
  "stats": {
    "feedsChecked": 12,
    "newContent": 8,
    "addedToVault": 3
  }
}
```

**Return to orchestrator:** Path to manifest file

---

## PHASE 2: Synthesis (Gemini 3.1 Pro Agent)

### Input
- All 8 topic manifests from Phase 1
- Full behavioral science format spec (`discoveries/meta/digest-format-spec.md`)
- Engagement history (`.discoveries-engagement.json`)

### Step 1: Load All Manifests
Read all 8 manifest files from `discoveries/meta/.manifests/[date]-[time]-*.json`

Aggregate into single context:
```json
{
  "allDiscoveries": [
    // All items from all 8 topics, ~10-30 total
  ],
  "byTopic": {
    "LLM Architecture": [...],
    "AI News": [...],
    // etc.
  },
  "totalCount": 23,
  "avgRelevance": 72
}
```

### Step 2: Cross-Source Synthesis
**This is where Gemini's 1M context shines.**

Analyze ALL discoveries simultaneously:
- Identify consensus (multiple sources agree)
- Detect conflicts (YouTube enthusiastic, Reddit skeptical)
- Extract themes (what's the meta-story?)
- Generate sentiment heatmap (per platform)
- Calculate urgency scores (1-5)

### Step 3: Generate 3-Layer Digest
Create comprehensive digest following `discoveries/meta/digest-format-spec.md`

**Path:** `discoveries/digests/[date]-[time]-digest.md`

**Path:** `discoveries/digests/[date]-[time]-digest.md`  
Example: `discoveries/digests/2026-02-24-1200-digest.md` (noon run)

**Design Goal:** 
- Layer 1: Scan in 5 seconds (mobile, on-the-go)
- Layer 2: Evaluate in 30 seconds (cross-source synthesis)
- Layer 3: Deep dive on-demand (evidence & verification)

**Format Philosophy:**
- **Layer 1 = Signal** - Instant decision-making (headline + 3 findings + urgency)
- **Layer 2 = Synthesis** - The value-add (consensus, themes, sentiment across sources)
- **Layer 3 = Evidence** - Verification (golden quotes, credibility, detailed notes)
- **Mobile-first:** Assume reading on phone, 30-second attention window

**CRITICAL:** Follow `discoveries/meta/digest-format-spec.md` exactly

**Content (3-Layer Format):**
```markdown
# Discovery Digest - [Date] [Time]

## 🔥 THE SIGNAL (Layer 1)

**HEADLINE:** [Single punchy sentence, max 15 words - most critical finding]

**KEY FINDINGS:**
• [Most actionable/surprising finding - max 15 words]
• [Second finding - max 15 words]
• [Most important finding - max 15 words]

**URGENCY:** [X/5] - [Brief justification]

---

## 📊 THE SYNTHESIS (Layer 2)

### CONSENSUS
[What multiple sources agree on. Note conflicts between platforms. E.g., "Multiple sources (📺 YouTube, 🧵 Reddit, 📰 TechCrunch) agree that X. Conflict exists between YouTube (enthusiastic) and Reddit (skeptical) regarding Y."]

### THEMES
**[Theme 1]** [Brief description]  
**[Theme 2]** [Brief description]  
**[Theme 3]** [Brief description]

### SENTIMENT HEATMAP
📺 **YouTube:** [Sentiment] ([context/percentage])  
🧵 **Reddit:** [Sentiment] ([context])  
📰 **News:** [Sentiment] ([context])  
📝 **Blogs:** [Sentiment] ([context])

---

## 🔍 THE EVIDENCE (Layer 3)

### 📺 YouTube - [Creator] (Relevance: [score])

**GOLDEN QUOTE:** "[1-2 high-impact direct quotes]"

**SOURCE:** [URL]  
**CREDIBILITY:** [High/Medium/Low] ([justification - credentials, followers, expertise])  
**TYPE:** 📺 YouTube

**WHY FLAGGED:** [Brief explanation of relevance to user interests, pattern rationale]

📄 [Detailed analysis](obsidian://open?vault=Obsidian%20Vault&file=discoveries/[platform]/[filename].md)

---

### [Repeat for each discovery with icon for source type]
[Use: 📺 YouTube, 🧵 Forum/Reddit, 📰 News, 📝 Blog/Newsletter, 💻 GitHub]

---

## 📊 Run Stats

- Sources checked: [count]
- RSS feeds: [success]/[total] ✓
- New content: [count]
- Above threshold (60+): [count]
- Average relevance: [score]

---

## 💡 Quick Actions

- Click any title to read full analysis
- `mark engaging [title]` - Boost similar content
- `drill in [title]` - Get deeper analysis
- `not interested [title]` - Hide similar

---

*Next run: [time]*
*View all discoveries: [discoveries/](../)*

## 🎯 Commands

- `drill in [title]` - Get deep analysis of any item
- `mark engaging [title]` - Improve future recommendations
- `not interested [title]` - Filter out similar content
- `show me more like [title]` - Find related content

---

*Generated by Content Monitor Agent*
*Next run: [next time]*
```

### Step 4: Discord Notification (Layer 1 + Top Layer 3)
Send **mobile-optimized** notification to Discord #digest channel.

**Channel:** #digest (ID: 1476043075355938880)

**Strategy:** Post Layer 1 (The Signal) + top 2 evidences from Layer 3. Full Layer 2 synthesis lives in Obsidian digest.

**Format:**
```
🔥 **THE SIGNAL** - [Date] [Time]

**HEADLINE:** [Single punchy sentence from Layer 1]

**KEY FINDINGS:**
• [Finding 1]
• [Finding 2]
• [Finding 3]

**URGENCY:** [X/5]

---

📊 **QUICK SYNTHESIS**
[1-2 sentence consensus from Layer 2]

**Top theme:** [Most important theme]

---

🔍 **TOP EVIDENCE**

📺 **[Creator]** ([score]) - [Title]
"[Golden quote]"
🔗 [URL]
📄 obsidian://open?vault=Obsidian%20Vault&file=discoveries/[path]

---

📝 **[Creator]** ([score]) - [Title]
"[Golden quote]"
🔗 [URL]
📄 obsidian://open?vault=Obsidian%20Vault&file=discoveries/[path]

---

📋 **Full 3-layer digest:** discoveries/digests/[date]-[time]-digest.md
⏰ **Next:** [time]
```

**Mobile UX:**
- Discord shows Layer 1 immediately (scan in 5 sec)
- Top 2 evidences give taste of Layer 3
- "Full digest" link for Layer 2 synthesis (Obsidian)
- Each evidence: quote + URL + Obsidian link

**Notification Frequency:**
- Per-run (4x daily) - user requested
- Each post: Signal + top 2 discoveries
- Full synthesis/sentiment in Obsidian only (too long for Discord)

---

## PHASE 3: Orchestration

### Main Cron Job Process

The cron job is the orchestrator that coordinates both phases:

```
1. Spawn 8 parallel Claude sub-agents (Phase 1)
   - Pass topic name to each
   - Wait for all to complete (or timeout after 10 min)
   - Collect manifest paths

2. Check Phase 1 results
   - If any topic failed → log it, continue with successful ones
   - Aggregate stats (total feeds checked, discoveries, etc.)

3. Spawn 1 Gemini sub-agent (Phase 2)
   - Pass all manifest paths
   - Model: google/gemini-3.1-pro (1M context)
   - Wait for digest generation
   - Get digest path

4. Return summary
   - Link to digest
   - Stats from all phases
   - Any errors/warnings

5. Cleanup
   - Archive manifests (optional, for debugging)
   - Log metrics
   - Update last-run timestamp
```

### Error Handling

**Phase 1 topic failure:**
- Continue with other topics
- Phase 2 synthesizes what's available
- Note missing topics in digest

**Phase 2 synthesis failure:**
- Fall back to simple aggregation
- Create basic digest without cross-source analysis
- Alert user

**Complete failure:**
- Log error
- Send Discord notification with error details
- Retry on next scheduled run

## Advanced Features

### Related Notes (Vault Search)
When creating a note, search vault for related content:

**Strategy:**
1. Extract key terms from title/summary
2. Search vault: `grep -r "term1\|term2\|term3" --include="*.md"`
3. LLM: "Which of these notes is most related?"
4. Add [[wikilinks]] to Related Notes section

### Engagement Feedback
Track what user does with discoveries:

**User commands:**
- `mark engaging [title]` → Add to `.discoveries-engagement.json`
- `not interested [title]` → Add to dismissed list
- `drill in [title]` → Counts as high engagement

**Learning:**
- Engaged sources → boost in future relevance scoring
- Dismissed sources → lower threshold or skip similar
- Frequently engaged topics → prioritize in searches

### Content Deduplication
Before adding, check if similar content already exists:

**Strategy:**
1. Search vault for similar titles
2. LLM: "Is this a duplicate/similar to existing note?"
3. If yes → merge or skip
4. If no → create new note

## Error Handling

**RSS feed timeout:**
- Log it, skip for this run
- Try again next run
- After 3 failures → mark source as inactive (notify user)

**LLM scoring failure:**
- Use fallback heuristic (keyword matching)
- Lower default score (40) to be safe
- Retry on next run

**Obsidian write failure:**
- Log content to temp file
- Retry on next run
- Notify user if persistent

## Performance Optimization

**Parallel processing:**
- Fetch RSS feeds in parallel (10 at a time)
- Score relevance in batches (5-10 items per LLM call)
- Write Obsidian notes in batch

**Caching:**
- Cache RSS feed fetches for 5 min (same URL)
- Cache vault search results
- Cache LLM summaries (by content hash)

**Rate limiting:**
- Max 100 RSS fetches per run
- Max 50 Obsidian notes created per run
- If exceeded → prioritize highest relevance scores

## Cron Jobs

**Four jobs per day:**

```javascript
// 6 AM
{
  "name": "Content Monitor - Morning",
  "schedule": {
    "kind": "cron",
    "expr": "0 6 * * *",
    "tz": "America/Chicago"
  },
  "payload": {
    "kind": "agentTurn",
    "message": "Run content monitoring. Follow CONTENT-MONITOR-AGENT.md. Check all sources, filter new content, add to Obsidian, send Discord notification.",
    "timeoutSeconds": 1200  // 20 min
  },
  "sessionTarget": "isolated",
  "enabled": true,
  "notify": false
}

// Repeat for 12 PM, 6 PM, 12 AM
```

**Or:** Single cron that runs every 6 hours:
```javascript
"expr": "0 */6 * * *"  // Every 6 hours
```

## Metrics & Logging

Track in `discoveries/meta/stats.json`:
```json
{
  "runs": [
    {
      "timestamp": "2026-02-24T12:00:00Z",
      "feeds_checked": 87,
      "new_content": 23,
      "after_filter": 12,
      "auto_added": 8,
      "avg_relevance": 72,
      "duration_ms": 845000,
      "errors": []
    }
  ],
  "totals": {
    "total_runs": 120,
    "total_discoveries": 960,
    "avg_per_run": 8,
    "engaged_rate": 0.23  // 23% engagement rate
  }
}
```

## Tuning Parameters

Adjust based on metrics:

**Relevance threshold:**
- Start: 60+
- If too many low-quality → raise to 70+
- If missing good content → lower to 50+

**Run frequency:**
- Start: Every 6 hours (4x daily)
- If too much content → 8-12 hours
- If not enough → 4 hours

**Sources per topic:**
- Start: 5-10 sources per topic
- Scale up to 15-20 as needed

**Notification style:**
- Start: After each run (4x daily)
- User prefers less → end-of-day digest only
- User wants more → real-time high-priority alerts

---

**Next:** Implementation plan & initial testing
