# Meta-Discovery Agent Specification

**Purpose:** Find top contributors for a given topic and build follow lists  
**Frequency:** Weekly (Sunday 2 AM CST)  
**Runtime:** ~15-30 min per topic

## Agent Definition

This is a **sub-agent** spawned via `sessions_spawn`:
- Isolated session
- Long-running (30 min timeout)
- Returns curated source list
- Updates existing lists (doesn't replace, merges)

## Input

**Topic:** String (e.g., "LLM Architecture", "Agents", "Local LLM")

**Search Strategy:**
1. "best YouTube channels about [topic]"
2. "top [topic] newsletters"
3. "[topic] Substack publications"
4. "influential [topic] Twitter accounts"
5. "[topic] RSS feeds"
6. "who to follow for [topic]"

## Process

### Step 1: Web Search (6 queries)
For each search strategy, use `web_search`:
- Get top 10 results
- Extract mentions of creators, channels, publications
- Collect URLs

### Step 2: Validation
For each potential source:
- **YouTube:** Extract channel ID, check RSS feed availability
- **Substack:** Check if RSS feed exists
- **Twitter:** Check if account is active (via nitter or profile page)
- **Other RSS:** Validate feed URL

### Step 3: Quality Assessment (LLM)
For each validated source, evaluate:
- Posting frequency (active vs. dormant)
- Content quality (depth, expertise)
- Relevance to topic (direct match or tangential)
- Audience engagement (if available)
- Diversity (not all the same perspective)

**Score 0-100, keep sources 70+**

### Step 4: Metadata Extraction
For each kept source:
- Name
- URL (main)
- RSS feed URL
- Platform (youtube/substack/twitter/rss)
- Description (1-2 sentences)
- Last post date (if available)
- Follower/subscriber count (if available)
- Focus area (what makes them unique)

### Step 5: Generate Source List
Create markdown file: `discoveries/sources/[topic-slug].md`

Format:
```markdown
# [Topic] - Sources

**Topic:** [Full topic name]
**Last Updated:** [Date]
**Next Update:** [Date + 7 days]
**Total Sources:** [Count]

## YouTube Channels

### [Creator Name]
- **URL:** [Channel URL]
- **RSS:** [Feed URL]
- **Focus:** [What they cover]
- **Why valuable:** [Reasoning]
- **Last active:** [Date of last video]

## Substack/Newsletters

### [Publication Name]
- **URL:** [Substack URL]
- **RSS:** [Feed URL]
- **Author:** [Name]
- **Focus:** [What they cover]
- **Why valuable:** [Reasoning]
- **Frequency:** [Weekly/daily/etc.]

## Twitter Accounts

### [@handle]
- **URL:** [Twitter profile]
- **RSS Bridge:** [nitter.net feed or scraping method]
- **Focus:** [What they tweet about]
- **Why valuable:** [Reasoning]

## RSS Feeds (Other)

### [Feed Name]
- **URL:** [Feed URL]
- **Source:** [Website/platform]
- **Focus:** [Content type]

---

## Discovery Log

**2026-02-24:** Initial discovery - found 15 sources
**2026-03-02:** Weekly update - added 2 new, removed 1 inactive
```

### Step 6: Merge with Existing
If source list already exists:
- Load existing list
- Keep all current sources
- Add new sources not already present
- Mark inactive sources (no content in 90+ days)
- Update metadata

Don't remove sources automatically—user may want dormant ones.

## Output

**Return to main session:**
```
Meta-Discovery: [Topic]

✓ Searched 6 query strategies
✓ Found 47 potential sources
✓ Validated 32 active sources
✓ Scored and filtered → 15 high-quality sources

Added to: discoveries/sources/[topic-slug].md

Breakdown:
- YouTube channels: 6
- Substack newsletters: 4
- Twitter accounts: 3
- RSS feeds: 2

Top source: [Name] - [Why they're #1]

Ready for content monitoring!
```

## LLM Prompts

### Quality Assessment
```
Source: [name]
Platform: [youtube/substack/twitter]
URL: [url]
Topic: [topic]

Recent content:
[List of 3-5 recent posts/videos]

Evaluate this source for automated monitoring:

1. **Relevance** (0-100): How well does this match "[topic]"?
2. **Quality** (0-100): Depth, expertise, production value?
3. **Frequency** (0-100): Posting consistency?
4. **Value** (0-100): Unique insights vs. rehashed content?

Overall Score (0-100): [weighted average]

Reasoning: [2-3 sentences]

Focus area: [What makes this source unique]

Recommendation: [Keep / Skip / Borderline]
```

### Description Generation
```
Source: [name]
Platform: [platform]
Recent content titles: [list]

Write a 1-2 sentence description of what this source covers.
Focus on their unique angle or expertise.

Format: "[Creator] focuses on [specific area], with emphasis on [unique approach]."
```

## Technical Notes

**RSS Feed Discovery:**

YouTube:
```
Channel URL: https://www.youtube.com/@ChannelName
→ Get channel ID (from page source or via redirect)
→ RSS: https://www.youtube.com/feeds/videos.xml?channel_id=CHANNEL_ID
```

Substack:
```
Publication URL: https://publication.substack.com
→ RSS: https://publication.substack.com/feed
```

Twitter (via nitter):
```
Twitter: https://twitter.com/username
→ Nitter: https://nitter.net/username/rss
(Alternative instances: nitter.poast.org, nitter.1d4.us)
```

**Handling Failures:**
- If RSS feed doesn't work → note it, keep source anyway
- If can't validate → lower score, but don't skip
- If search returns few results → broaden query

## Cron Job Setup

```javascript
{
  "name": "Meta-Discovery: [Topic]",
  "schedule": {
    "kind": "cron",
    "expr": "0 2 * * 0",  // Sunday 2 AM
    "tz": "America/Chicago"
  },
  "payload": {
    "kind": "agentTurn",
    "message": "Run meta-discovery for topic: [topic]. Follow the process in META-DISCOVERY-AGENT.md. Save results to discoveries/sources/[topic-slug].md",
    "timeoutSeconds": 1800  // 30 min
  },
  "sessionTarget": "isolated",
  "enabled": true,
  "notify": false
}
```

**Create one cron job per topic**, or:

**Single weekly job that processes all topics:**
```javascript
"message": "Run meta-discovery for all topics: LLM Architecture, AI News, AI Model Releases, Agents, AI Frameworks, Vibe Coding, OpenClaw, Local LLM. Process each topic sequentially."
```

## Initial Run

Before setting up cron, run manually for all 8 topics:
1. LLM Architecture
2. AI News
3. AI Model Releases
4. Agents
5. AI Frameworks
6. Vibe Coding
7. OpenClaw
8. Local LLM

This establishes baseline source lists for content monitoring to use immediately.

## Maintenance

**Weekly updates:**
- Find 2-3 new sources per topic
- Check existing sources still active
- Adjust quality scores based on engagement data

**Monthly review:**
- User reviews source lists
- Removes sources manually if needed
- Adds manual suggestions
- Agent incorporates feedback

**Tuning:**
- If too many low-quality sources → raise score threshold
- If missing key sources → broaden search queries
- If too niche → add more diversity in search

---

**Next:** Content Monitor Agent specification
