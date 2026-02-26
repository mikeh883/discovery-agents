# Discovery Agents - System Architecture

**Status:** Design Phase  
**Created:** 2026-02-24  
**Purpose:** Autonomous content discovery, curation, and Obsidian integration

## Overview

Two-tier agent system:
1. **Meta-Discovery Agent** - Finds contributors to follow (weekly)
2. **Content Monitors** - Tracks those contributors for new content (every 4-6 hours)

## Core Workflow

```
User defines topic
    ↓
Meta-Discovery Agent (weekly)
    → Web search for top contributors
    → Build follow list
    → Save to discoveries/sources/
    ↓
Content Monitor Agents (4-6 hours)
    → Check follow lists
    → Discover new content
    → Filter by relevance (LLM)
    → Auto-add to Obsidian
    → Discord notification
    ↓
User reviews in Obsidian
    → Mark engaging items (feedback loop)
    → Request deeper analysis ("drill in")
```

## Initial Topics

Priority areas to track:
1. **LLM Architecture** - Model designs, training techniques, inference optimization
2. **AI News** - Industry developments, funding, releases
3. **AI Model Releases** - New models, benchmarks, capabilities
4. **Agents** - Agent frameworks, autonomous systems, multi-agent
5. **AI Frameworks** - LangChain, LlamaIndex, new tooling
6. **Vibe Coding** - AI-assisted development, coding agents
7. **OpenClaw** - Updates, community, related projects
8. **Local LLM** - Self-hosted models, privacy, optimization

## Content Sources

### Phase 1 (No API required)
- **YouTube** - Channel monitoring via RSS feeds (no API needed!)
- **Substack** - RSS feeds per publication
- **RSS** - Direct feed subscription
- **Twitter/X** - RSS bridges (nitter.net, etc.) or web scraping

### Phase 2 (If we get API access)
- **YouTube Data API** - Better metadata, trending, recommendations
- **Twitter API** - Real-time monitoring, thread extraction
- **Reddit API** - Subreddit tracking

### Technical Notes
- YouTube channels have RSS: `https://www.youtube.com/feeds/videos.xml?channel_id=CHANNEL_ID`
- Substack newsletters have RSS: `https://PUBLICATION.substack.com/feed`
- No authentication needed for RSS—perfect for starting

## Agent 1: Meta-Discovery Agent

**Frequency:** Weekly (Sunday 2 AM CST)

**Process:**
1. Take topic (e.g., "LLM Architecture")
2. Web search: "top YouTube channels about LLM architecture"
3. Web search: "best newsletters about LLM architecture"
4. Web search: "LLM architecture Twitter accounts to follow"
5. Extract contributor names, URLs, handles
6. Validate (check if active, has RSS, quality signals)
7. Build follow list with metadata
8. Save to `discoveries/sources/[topic].md`

**Output Format:**
```markdown
# LLM Architecture - Sources

**Last Updated:** 2026-02-24
**Next Update:** 2026-03-02

## YouTube Channels
- **Yannic Kilcher** - [URL] - RSS: [feed URL] - Focus: Paper reviews
- **Andrej Karpathy** - [URL] - RSS: [feed URL] - Focus: Deep dives

## Substack/Newsletters
- **The Batch** (Andrew Ng) - [URL] - RSS: [feed URL]
- **Import AI** - [URL] - RSS: [feed URL]

## Twitter/X Accounts
- **@karpathy** - [URL] - RSS bridge: [feed URL]
- **@AnthropicAI** - [URL] - RSS bridge: [feed URL]

## RSS Feeds (Other)
- **Hacker News AI** - [URL]
- **Papers with Code** - [URL]

## Metadata
- Total sources: 15
- Added this week: 3
- Inactive (removed): 1
```

**LLM Prompt:**
```
Topic: [topic]

Find the top 10-15 most valuable content creators for this topic.
Include:
- YouTube channels
- Substack/newsletter authors
- Twitter accounts
- RSS feeds

For each, provide:
- Name
- URL
- RSS feed (if available)
- Brief description of their focus
- Why they're valuable for this topic

Prioritize:
- Active creators (recent content)
- High quality (expertise, depth)
- Diverse perspectives
- Regular posting schedule
```

## Agent 2: Content Monitor Agents

**Frequency:** Every 6 hours (6 AM, 12 PM, 6 PM, 12 AM CST)

**Process:**
1. Load all source lists from `discoveries/sources/`
2. For each RSS feed:
   - Fetch latest entries
   - Check if already seen (track by URL in `.seen-content.json`)
   - If new → filter by relevance
3. For new relevant content:
   - Generate summary (LLM)
   - Extract key points
   - Assign topics/tags
   - Create Obsidian note
   - Add to daily digest
4. Send Discord notification (digest summary)

**Relevance Filtering (LLM):**
```
Content: [title + description]
User's topics: [topic list]
User's recent Obsidian activity: [recent note titles/tags]

Score this content's relevance (0-100).
Consider:
- Direct topic match
- Related to active projects
- Novel information (not duplicate)
- Quality signals (engagement, source reputation)

Threshold: 60+ = auto-add
Return: score, reasoning, suggested tags
```

**Output Locations:**

**Individual Notes:**
```
discoveries/
  youtube/
    2026-02-24-yannic-kilcher-attention-is-all-you-need.md
  substack/
    2026-02-24-import-ai-gpt5-rumors.md
  twitter/
    2026-02-24-karpathy-thread-transformers.md
```

**Daily Digest:**
```
discoveries/
  digests/
    2026-02-24-digest.md
```

**Note Template:**
```markdown
---
source: youtube
creator: Yannic Kilcher
topic: [LLM Architecture, Agents]
discovered: 2026-02-24T12:34:56Z
relevance: 85
url: https://youtube.com/watch?v=...
---

# [Video Title]

## Summary
[LLM-generated summary, 3-4 sentences]

## Key Points
- Point 1
- Point 2
- Point 3

## Why This Matters
[LLM reasoning about relevance to your interests]

## Related
- [[Link to related note in your vault]]
- [[Another related note]]

## Full Content
[Link to original]

---
*Discovered by Content Monitor Agent*
*Relevance Score: 85/100*
```

**Digest Template:**
```markdown
# Daily Discovery Digest - 2026-02-24

**Total Discoveries:** 12  
**By Topic:** LLM Architecture (4), AI News (3), Agents (3), Local LLM (2)  
**By Source:** YouTube (6), Substack (4), Twitter (2)

## 🔥 Top Picks (Relevance 80+)

### [Title] - Yannic Kilcher (YouTube)
Brief summary... [[Full note]](link)

### [Title] - Import AI (Substack)
Brief summary... [[Full note]](link)

## 📚 Worth Reading (Relevance 60-79)

### [Title] - Source
Brief summary... [[Full note]](link)

## 🔍 For Later

- [Title] - Source - [[link]]
- [Title] - Source - [[link]]

---

**Commands:**
- "drill into [title]" - Get deeper analysis
- "mark engaging" - Improve recommendations
- "not interested" - Adjust filters
```

## Discord Notifications

**Format:**
```
🔍 **Daily Discovery Digest** - 2026-02-24

Found **12 items** across your topics:
• 🔥 4 high-priority
• 📚 6 worth reading  
• 🔍 2 for later

**Top pick:** [Title] - Yannic Kilcher
"Summary of the content..."

View full digest: [Obsidian link]
```

**Frequency:**
- After each monitoring run (4x daily)
- Or: consolidated end-of-day digest (11 PM)

Your preference?

## Feedback Loop

**Engagement Tracking:**
Store in `.discoveries-engagement.json`:
```json
{
  "engaged": [
    {
      "url": "https://...",
      "title": "...",
      "topics": ["LLM Architecture"],
      "marked_at": "2026-02-24T14:30:00Z",
      "action": "marked_engaging"
    }
  ],
  "dismissed": [
    {
      "url": "https://...",
      "reason": "not_interested",
      "topics": ["AI News"]
    }
  ]
}
```

**Learning:**
- Engaged content → boost similar sources/topics
- Dismissed content → lower relevance threshold
- Track which creators you engage with most
- Adjust meta-discovery to find more like that

**Commands:**
- "mark engaging" - Mark current note
- "not interested" - Dismiss and adjust
- "more like this" - Find similar content
- "drill in [title]" - Deep dive analysis

## Deep Dive ("Drill In")

When you want more detail on a digest item:

**Command:** "drill in [title]"

**Process:**
1. Load the content (fetch full text/transcript)
2. LLM deep analysis:
   - Detailed summary (not brief)
   - Technical breakdown
   - Key insights & takeaways
   - Connections to your vault
   - Action items / follow-up questions
3. Expand the Obsidian note with full analysis
4. Return analysis in chat

**Example:**
```
You: "drill in the Yannic video about attention"

Agent:
📊 **Deep Dive: "Attention Is All You Need" - Yannic Kilcher**

[Full detailed analysis, 500+ words]

I've updated the note in discoveries/youtube/... with:
- Full transcript summary
- Technical breakdown of attention mechanism
- Connections to your [[Transformer Architecture]] note
- 3 follow-up resources

Want me to find related papers or implementations?
```

## Technical Implementation

**Tools Required:**
- `web_search` - For meta-discovery
- `web_fetch` - For RSS feed parsing, content extraction
- RSS parser library (built-in or fetch XML directly)
- Obsidian file management (write tool)
- Discord messaging (message tool)
- Cron scheduling (cron tool)
- Sub-agents for parallel processing

**Data Storage:**
- `discoveries/sources/` - Follow lists per topic
- `discoveries/youtube/`, `/substack/`, etc. - Individual notes
- `discoveries/digests/` - Daily digests
- `.discoveries-tracking.json` - Seen content (URL → timestamp)
- `.discoveries-engagement.json` - User feedback
- `discoveries/meta/` - Agent logs, stats, tuning

**No API Access Workarounds:**
- YouTube: RSS feeds (no API needed)
- Twitter: nitter.net RSS bridges or web scraping
- Substack: Native RSS support
- General: RSS is universal and free

**Future API Integration:**
If we get API access later, we can:
- Switch to official APIs
- Get richer metadata
- Enable real-time monitoring
- Access trending/recommendations

## Deployment Plan

### Phase 1: Meta-Discovery (Week 1)
- [ ] Build meta-discovery agent
- [ ] Run for all 8 initial topics
- [ ] Generate source lists
- [ ] Review and refine

### Phase 2: Basic Monitoring (Week 2)
- [ ] Build RSS feed fetcher
- [ ] Implement relevance filtering
- [ ] Create Obsidian note templates
- [ ] Test with one topic (LLM Architecture)

### Phase 3: Full Monitoring (Week 3)
- [ ] Scale to all topics
- [ ] Add Discord notifications
- [ ] Implement digest generation
- [ ] Set up 6-hour cron jobs

### Phase 4: Feedback & Drill-In (Week 4)
- [ ] Engagement tracking
- [ ] "Drill in" deep analysis
- [ ] Recommendation tuning
- [ ] Quality metrics

### Phase 5: Optimization (Ongoing)
- [ ] Adjust relevance thresholds
- [ ] Add new sources
- [ ] Remove inactive sources
- [ ] Fine-tune topics

## Success Metrics

**Quantity:**
- Discoveries per day (target: 10-20)
- Sources monitored (target: 50-100)
- Topics covered (8 initially)

**Quality:**
- Engagement rate (% marked engaging)
- Dismiss rate (% not interested)
- Drill-in rate (% requesting deep dive)

**Value:**
- Time saved (vs. manual discovery)
- Novel insights gained
- Connections made (vault links)

## Open Questions

1. **Discord notification frequency:** 4x daily or 1x end-of-day?
2. **Auto-add threshold:** Start at relevance 60+ or higher?
3. **Digest format:** Full digest or just top picks?
4. **RSS refresh:** 6-hour polling or 4-hour?
5. **Deep dive trigger:** Automatic for 90+ relevance or always manual?

---

**Next:** Build meta-discovery agent specification
