# Discovery Agents

**Autonomous AI content discovery system for Obsidian**

Monitor 82+ RSS sources across 8 topics, filter by relevance, create detailed notes, and deliver scannable digests to Discord.

## Features

- **RSS-Based Monitoring**: No API keys required, universal compatibility
- **8 AI Topics**: LLM Architecture, AI News, Model Releases, Agents, Frameworks, Vibe Coding, OpenClaw, Local LLM
- **Smart Filtering**: LLM-powered relevance scoring (60+ threshold)
- **Automated Discovery**: Runs 4x daily (6 AM, 12 PM, 6 PM, 12 AM CST)
- **3-Layer Digest**: Behavioral science format optimized for mobile scanning
  - **Layer 1 (The Signal)**: 5-second scan - headline + 3 key findings + urgency
  - **Layer 2 (The Synthesis)**: 30-second evaluation - consensus, themes, sentiment
  - **Layer 3 (The Evidence)**: On-demand deep dive - quotes, credibility, detailed notes
- **Cross-Source Synthesis**: Detect patterns, consensus, and conflicts across platforms
- **Discord Integration**: Push notifications to digest channel
- **Obsidian Sync**: Auto-created discovery notes with backlinks
- **Learning Feedback Loop**: "mark engaging" / "not interested" commands improve scoring

## Architecture

**Hybrid Model Approach:**
- **Phase 1 (Discovery)**: 8 parallel Gemini Flash 2.0 agents (one per topic)
  - RSS fetching, parsing, relevance scoring
  - Creates Obsidian notes for 60+ items
  - Generates topic manifests
- **Phase 2 (Synthesis)**: 1 Gemini Pro 1.5 agent (cross-source analysis)
  - 1M token context window
  - Consensus/conflict detection
  - Theme extraction, sentiment analysis
  - 3-layer digest generation
- **Phase 3 (Delivery)**: Discord notification + Obsidian storage

**Cost**: ~$0.02-0.03 per run (~$3/month for 4x daily)

## Installation

1. **Clone this repo into your Obsidian vault:**
   ```bash
   cd /path/to/vault/projects
   git clone https://github.com/YOUR_USERNAME/discovery-agents.git
   ```

2. **Create required directories:**
   ```bash
   cd ../../
   mkdir -p discoveries/{sources,youtube,substack,twitter,rss,digests,meta/.manifests}
   ```

3. **Copy source lists:**
   ```bash
   cp projects/discovery-agents/sources/*.md discoveries/sources/
   ```

4. **Set up Discord webhook** (optional):
   - Create a #digest channel
   - Note the channel ID for config

5. **Configure OpenClaw cron jobs:**
   - Follow `ORCHESTRATOR.md` for cron setup
   - Use Gemini Flash 2.0 for orchestrator
   - Use Gemini Pro 1.5 for synthesis

## Documentation

- **[ARCHITECTURE.md](ARCHITECTURE.md)** - System overview, data flow
- **[ORCHESTRATOR.md](ORCHESTRATOR.md)** - Main coordination logic
- **[CONTENT-MONITOR-AGENT.md](CONTENT-MONITOR-AGENT.md)** - Discovery agent spec
- **[MODEL-STRATEGY.md](MODEL-STRATEGY.md)** - Gemini/Claude hybrid approach
- **[META-DISCOVERY-AGENT.md](META-DISCOVERY-AGENT.md)** - Weekly source updates
- **[COMMANDS.md](COMMANDS.md)** - User interaction patterns
- **[digest-format-spec.md](../discoveries/meta/digest-format-spec.md)** - 3-layer format spec

## Usage

### Commands

```
drill in [title]           - Deep analysis with full content fetch
mark engaging [title]      - Boost similar content (+15/+10/+5)
not interested [title]     - Filter similar content (-20/-10)
show today                 - View latest digest
search discoveries [query] - Full-text search
```

### Cron Schedule

- **Morning**: 6 AM CST
- **Noon**: 12 PM CST
- **Evening**: 6 PM CST
- **Midnight**: 12 AM CST
- **Meta-Discovery**: Sunday 2 AM CST (weekly source updates)

## File Structure

```
discoveries/
├── sources/               # RSS source lists (8 topics)
├── youtube/              # YouTube discoveries
├── substack/             # Substack discoveries
├── twitter/              # Twitter/X discoveries
├── rss/                  # Generic RSS discoveries
├── digests/              # Daily digest markdown files
├── meta/
│   ├── .manifests/       # Topic manifests (temp)
│   ├── config.json       # System configuration
│   └── digest-format-spec.md  # Format specification
├── .discoveries-tracking.json   # Seen content tracking
└── .discoveries-engagement.json # User feedback data
```

## Customization

### Add New Sources

Edit source files in `discoveries/sources/[topic].md`:

```markdown
## YouTube
- [Channel Name](https://youtube.com/@channel) - RSS: https://youtube.com/feeds/...

## Substack
- [Newsletter Name](https://newsletter.substack.com) - RSS: https://newsletter.substack.com/feed
```

### Adjust Relevance Threshold

Edit `discoveries/meta/config.json`:
```json
{
  "relevanceThreshold": 60,
  "engagementBoost": {
    "creator": 15,
    "topic": 10,
    "platform": 5
  }
}
```

### Customize Digest Format

Edit `discoveries/meta/digest-format-spec.md` to adjust:
- Layer 1 structure (signal)
- Layer 2 synthesis approach
- Layer 3 evidence presentation

## Technical Details

**Models:**
- Discovery: Gemini Flash 2.0 (~$0.075 per 1M tokens)
- Synthesis: Gemini Pro 1.5 (~$1.25 per 1M tokens)

**Rate Limits:**
- Max 5 parallel sub-agents per session
- Solution: Batch processing (5 topics, wait, then 3 more)

**RSS Parsing:**
- Timeout: 30 sec per feed
- Retries: 3 failures → mark source inactive

**Tracking:**
- Seen content: 24-hour window for new discoveries
- Engagement: Persistent across sessions

## Roadmap

- [ ] Smart bundling (group related discoveries)
- [ ] Trend tracking (topic frequency over time)
- [ ] Conflict detection (highlight disagreements)
- [ ] Creator reputation scoring
- [ ] Push notifications for urgency=5 items
- [ ] Voice digest (audio summary)
- [ ] Readwise/Pocket integration
- [ ] Email digest option

## Contributing

This is a personal project, but ideas welcome! Open an issue to discuss.

## License

MIT

---

**Built with:** OpenClaw, Claude Sonnet 4.5, Gemini Flash 2.0, Gemini Pro 1.5  
**Platform:** Obsidian + Discord  
**Cost:** ~$3/month for 4x daily monitoring
