# Discovery Agents - Project Overview

**Status:** Design Complete, Ready to Build  
**Created:** 2026-02-24

## What Is This?

A two-tier autonomous content discovery system that finds, filters, and curates content from across the web, automatically adding relevant discoveries to your Obsidian vault.

**You get:** Personalized daily digests of high-quality content without manual hunting.

## How It Works

```
You define topics (e.g., "LLM Architecture", "AI Agents")
    ↓
Meta-Discovery Agent (weekly)
    → Finds top YouTube channels, Substacks, Twitter accounts
    → Builds curated follow lists
    ↓
Content Monitor Agents (every 6 hours)
    → Checks RSS feeds for new content
    → Scores relevance with LLM (60+ = relevant)
    → Auto-adds to Obsidian
    → Sends Discord digest
    ↓
You review discoveries in Obsidian
    → Mark engaging (feedback loop)
    → Drill in for deep analysis
    → Connect to your vault
```

## Initial Topics

1. **LLM Architecture** - Model designs, training, inference
2. **AI News** - Industry developments, funding, releases
3. **AI Model Releases** - New models, benchmarks, capabilities
4. **Agents** - Agent frameworks, autonomous systems, multi-agent
5. **AI Frameworks** - LangChain, LlamaIndex, new tooling
6. **Vibe Coding** - AI-assisted development, coding agents
7. **OpenClaw** - Updates, community, related projects
8. **Local LLM** - Self-hosted models, privacy, optimization

## Core Features

### Discovery
- **Automated monitoring:** YouTube, Substack, Twitter, RSS
- **Smart filtering:** LLM-based relevance scoring (0-100)
- **Quality over quantity:** Only add 60+ relevance content
- **No API required:** RSS-based (free, universal)

### Curation
- **Auto-add to Obsidian:** Formatted notes with metadata
- **Daily digests:** Aggregated summaries by run
- **Organized by platform:** youtube/, substack/, twitter/, rss/
- **Deduplication:** Never add the same content twice

### Intelligence
- **Related notes:** Automatically links to your vault
- **Engagement learning:** Tracks what you like, adjusts future scoring
- **Deep dive:** "Drill in" command for full analysis
- **Pattern recognition:** Topics, creators, insights

### Notifications
- **Discord integration:** Digest summaries after each run
- **Customizable:** 4x daily or end-of-day consolidated
- **Actionable:** Links to notes, commands for drill-in

## Commands

**Manual triggers:**
- `"run meta-discovery for [topic]"` - Find new sources
- `"drill in [title]"` - Deep analysis of a discovery
- `"mark engaging [title]"` - Improve recommendations
- `"not interested [title]"` - Filter out similar content
- `"show me discoveries from today"` - Review digest

**Automatic (cron):**
- Content monitoring: Every 6 hours
- Meta-discovery: Weekly (Sunday 2 AM)

## File Structure

```
discoveries/
  sources/                    # Follow lists per topic
    llm-architecture.md       # YouTube, Substack, Twitter, RSS
    ai-news.md
    agents.md
    ...
  
  youtube/                    # Platform-specific discoveries
    2026-02-24-yannic-kilcher-attention.md
  substack/
    2026-02-24-import-ai-gpt5.md
  twitter/
    2026-02-24-karpathy-thread.md
  rss/
    2026-02-24-hackernews-ai.md
  
  digests/                    # Daily digests
    2026-02-24-digest.md
  
  meta/                       # Stats, logs, config
    stats.json
    config.json
  
  .discoveries-tracking.json  # Seen content (deduplication)
  .discoveries-engagement.json # User feedback (learning)
```

## Implementation Status

- [x] Architecture designed
- [x] Meta-discovery agent spec
- [x] Content monitor agent spec
- [x] Implementation roadmap (3 weeks)
- [ ] Directory structure created
- [ ] Initial meta-discovery run (test)
- [ ] RSS fetcher built
- [ ] Relevance scoring tested
- [ ] First discoveries created
- [ ] Cron jobs scheduled

## Next Steps

1. **Day 1:** Create directory structure, run meta-discovery for "LLM Architecture" (test)
2. **Week 1:** Build RSS fetcher, test relevance scoring, create first discoveries
3. **Week 2:** Scale to all 8 topics, set up cron jobs, Discord notifications
4. **Week 3:** Engagement tracking, drill-in feature, related notes, tuning

## Documentation

- **[ARCHITECTURE.md](./ARCHITECTURE.md)** - Full system design, workflow, technical details
- **[META-DISCOVERY-AGENT.md](./META-DISCOVERY-AGENT.md)** - How to find sources
- **[CONTENT-MONITOR-AGENT.md](./CONTENT-MONITOR-AGENT.md)** - How to monitor content
- **[IMPLEMENTATION.md](./IMPLEMENTATION.md)** - Build roadmap, testing, rollout

## Tech Stack

- **OpenClaw tools:** web_search, web_fetch, Write, Read, message, cron, sessions_spawn
- **Data sources:** RSS feeds (YouTube, Substack, nitter for Twitter)
- **Intelligence:** LLM scoring, summarization, topic extraction
- **Storage:** Obsidian markdown notes, JSON tracking files
- **Notifications:** Discord

**No external dependencies.** Everything uses RSS (free, no API keys).

## Success Criteria

### Week 1
- 8 source lists created (10-15 sources each)
- 5-10 discoveries per run
- 20-40 discoveries daily

### Week 2
- 70%+ relevance rate (not dismissed)
- 20%+ engagement rate (marked engaging)
- User finds 2-3 high-value insights daily

### Week 3+
- Saves 1-2 hours of manual discovery time daily
- Makes 3-5 vault connections per week
- System runs autonomously with minimal tuning

## Why This Matters

**The Problem:** You spend hours hunting for good content across YouTube, Twitter, Substack. You miss things. You see duplicates. Discovery is friction.

**The Solution:** Agents do the hunting. You do the synthesis. Wake up to curated, relevant content waiting in Obsidian. Focus on ideas, not hunting.

**The Secret Sauce:** It's not just aggregation—it's *intelligent curation* that learns what you care about and connects to your existing knowledge.

---

**Ready to build?** Start with Day 1 in IMPLEMENTATION.md or ask to run the first meta-discovery test.
