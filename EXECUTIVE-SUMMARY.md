# Discovery Agents: Executive Summary

**Project:** Autonomous AI Content Discovery System  
**Status:** Production Ready  
**Last Updated:** 2026-02-26  
**Next Milestone:** First automated run (2026-02-27 8:00 AM CST)

---

## Vision & Purpose

### Core Problem
The AI landscape moves too fast for manual monitoring. By the time you see a Reddit discussion about a new model release or coding technique, dozens of relevant articles, papers, and tools have already been published. Traditional RSS readers create notification fatigue and lack intelligent filtering.

### Our Solution
An autonomous agent that:
1. **Monitors** 19 curated RSS sources across 3 AI topics
2. **Filters** by relevance (60+ threshold) using LLM-based scoring
3. **Synthesizes** cross-source patterns and insights
4. **Delivers** scannable mobile-first digests to Discord
5. **Learns** from your feedback (engagement scoring)

### Design Philosophy
**"Information scent over information overload"**

Based on behavioral science research:
- **Progressive disclosure:** 3-layer format (5s scan → 30s synthesis → deep dive)
- **Serial position effect:** Most important findings first
- **Minimal friction:** Mobile-optimized, no jargon without definitions
- **Actionability:** Every insight should suggest what to do with it

---

## Key Decisions & Rationale

### Architecture Decisions

**1. RSS-Only Approach**
- ✅ Universal, free, no API dependencies
- ✅ Works across YouTube, Substack, newsletters, blogs
- ❌ Twitter/X blocked (all Nitter instances down)
- **Impact:** 58 working sources after cleanup

**2. 3-Phase Agent Pipeline**
- **Prep Agent** (Flash-Lite): Validate sources, create plan
- **Fetch Agent** (Flash-Lite): Test RSS URLs
- **Score Agent** (Pro): Parse RSS, score items, create notes
- **Why 3 phases:** Flash-Lite can't handle complex multi-step tasks
- **Discovery:** Sub-agents can't spawn other sub-agents
- **Solution:** Main session orchestrates, spawns all sub-agents

**3. Model Strategy**
- **Orchestrator:** Claude Sonnet 4.5 (reliable orchestration)
- **Simple tasks:** Gemini Flash-Lite ($0.10/1M)
- **Complex tasks:** Gemini Pro ($1.25/1M)
- **Why hybrid:** 60% cost savings vs all-Pro, 100% reliability vs all-Flash-Lite
- **Failed attempt:** All Flash-Lite (agents started but didn't complete)

**4. Two-Tier Content Pipeline**
- **Daily runs:** Process RSS feeds, create notes, generate digests
- **Weekly meta-discovery:** Find new sources, validate existing, remove dead feeds
- **Why separate:** Daily runs stay fast/cheap, weekly cleanup keeps quality high

### Topic Selection

**Active Topics (19 sources):**
1. **AI News** (10 sources)
   - AI Weekly, AI Explained, TechCrunch AI, MarkTechPost, etc.
   - **Why:** Broad coverage, multiple perspectives
   
2. **AI Model Releases** (4 sources)
   - Hugging Face, Papers with Code, GitHub trending
   - **Why:** Track new models as they drop
   
3. **Agents** (5 sources)
   - LangChain, AutoGPT, Anthropic blog
   - **Why:** Core to OpenClaw use cases

**Inactive Topics (39 sources):**
- LLM Architecture, AI Frameworks, Vibe Coding, OpenClaw, Local LLM
- **Why paused:** Cost optimization during validation phase
- **Can reactivate:** Add to cron task, scales at ~$0.75 per topic

### Frequency & Schedule

**Initial Plan:** 4x daily (6am, 12pm, 6pm, 12am)
- **Cost:** $680/month
- **Rationale:** Catch breaking news quickly

**Current Plan:** 2x daily (8am, 8pm)
- **Cost:** $132/month
- **Rationale:** AI news cycles are ~12 hours, not 6 hours
- **User validation required:** Assess real-world value before scaling up

**Future Considerations:**
- Smart scheduling based on source update patterns
- On-demand runs for breaking news
- Weekend vs weekday schedules

### Relevance Scoring

**Threshold:** 60+ (out of 100)
- **Below 60:** Tracked but not surfaced
- **60-79:** Good signal, worth reviewing
- **80-100:** High value, direct relevance

**Scoring Criteria:**
1. **Topic match:** How relevant to the specific topic (ai-news, agents, etc.)
2. **Novelty:** Unique angle vs obvious/duplicate
3. **Actionability:** Can the user do something with this insight?

**Engagement Boosters:**
- Creator engaged before: +15
- Topic overlap: +10 per related topic
- Preferred platform: +5

**Engagement Penalties:**
- Creator dismissed: -20
- Topic dismissed: -10

### Digest Format (Behavioral Science)

**3-Layer Progressive Disclosure:**

**Layer 1: Signal (5-second scan)**
```
🔥 URGENCY: 3/5
📊 26 discoveries across 3 topics

Key Findings:
1. [Most important insight]
2. [Second most important]
3. [Third most important]
```
- **Why:** Serial position effect - people remember first and last
- **Mobile-first:** Works on phone lock screen
- **Urgency score:** Immediate triage (1=routine, 5=breaking)

**Layer 2: Synthesis (30-second read)**
```
Consensus: What multiple sources agree on
Conflicts: Where platforms disagree
Themes: Meta-patterns across topics
Sentiment Heatmap: Per-source tone
```
- **Why:** Cross-source validation builds trust
- **Pattern detection:** Surfaces non-obvious connections
- **Information scent:** Helps decide whether to dig deeper

**Layer 3: Evidence (deep dive)**
```
All 26 discoveries with:
- Title + creator
- 3-5 key points
- Relevance score
- Link to full note
```
- **Why:** Complete audit trail
- **Progressive disclosure:** Only read if Layers 1-2 are interesting
- **No jargon:** Every technical term explained

**Format Evolution:**
- Started with flat lists
- Added synthesis layer after realizing users wanted "what does this mean?"
- Moved to 3-layer after studying information architecture research
- Next: User feedback integration (mark engaging/not interested commands)

---

## Progress Timeline

### Phase 1: Research & Design (Feb 24)
- ✅ Defined 8 topics with 82 sources
- ✅ Designed 3-layer digest format
- ✅ Created behavioral science framework
- ✅ Built engagement scoring system

### Phase 2: Architecture Validation (Feb 25-26)
- ✅ Tested Gemini 2.5 Flash-Lite (failed complex tasks)
- ✅ Tested Gemini 2.5 Pro (succeeded)
- ✅ Discovered sub-agent spawning limitation
- ✅ Validated 3-phase pipeline (prep → fetch → score)
- ✅ Tested end-to-end with openclaw topic (6 notes created)

### Phase 3: Source Cleanup (Feb 26)
- ✅ Tested all 82 RSS feeds
- ✅ Removed 40% dead feeds (404, 403, timeouts)
- ✅ Reduced to 58 working sources
- ✅ Updated all source files with cleanup notes

### Phase 4: Cost Optimization (Feb 26)
- ✅ Reduced from 8 topics to 3 (ai-news, ai-model-releases, agents)
- ✅ Reduced from 4x daily to 2x daily
- ✅ Cost: $680/month → $132/month (81% reduction)
- ✅ Validated model strategy (Flash-Lite + Pro hybrid)

### Phase 5: Production Deployment (Feb 26)
- ✅ Created cron jobs (8am, 8pm CST)
- ✅ Configured orchestration (Claude Sonnet main session)
- ✅ Documented production architecture
- ✅ Committed to GitHub (discovery-agents repo)
- ⏳ First automated run: Feb 27 8:00 AM CST

---

## Current State

### What's Working
1. **3-phase pipeline:** Prep, Fetch, Score validated end-to-end
2. **Model strategy:** Flash-Lite + Pro hybrid proven
3. **Source quality:** 58 working feeds after cleanup
4. **Orchestration:** Main session spawning sub-agents reliably
5. **Documentation:** Complete architecture + production guides

### What's Not Implemented Yet
1. **Synthesis agent:** Created spec but not tested end-to-end
2. **Discord posting:** Tested individually but not in full pipeline
3. **Engagement commands:** `drill in`, `mark engaging`, `not interested` designed but not wired up
4. **Weekly meta-discovery:** Cron job exists but agent spec needs refinement
5. **Smart bundling:** Grouping related discoveries (future enhancement)

### Known Issues
1. **Twitter/X feeds:** All blocked (Nitter instances down, Cloudflare blocks RSS bridges)
   - **Workaround:** Removed from source lists
   - **Future:** Direct Twitter API (costs $$$)

2. **YouTube channel IDs:** Some return 404
   - **Status:** Need manual verification
   - **Impact:** ~5-10% of YouTube sources

3. **Agent timeouts:** Flash-Lite occasionally slow on large RSS feeds
   - **Observed:** 620k tokens for 10-source topic
   - **Mitigation:** 180s timeout for fetch, 600s for score

4. **Gemini 3.1 Pro Preview:** Not available in OpenClaw yet
   - **Workaround:** Using 2.5 Pro (has needed 2M context)
   - **Impact:** None (2.5 Pro working well)

---

## Cost Analysis

### Per-Run Breakdown (3 topics)

| Phase | Agent | Model | Tokens | Cost |
|-------|-------|-------|--------|------|
| Prep (3×) | Validate sources | Flash-Lite | ~120k | $0.012 |
| Fetch (3×) | Test URLs | Flash-Lite | ~960k | $0.096 |
| Score (3×) | Parse, score, notes | Pro | ~1.4M | $1.750 |
| Synthesis (1×) | Cross-topic digest | Pro | ~500k | $0.625 |
| **Total** | | | **~3M** | **$2.48** |

**Observed Range:** $1.90 - $2.50 per run

### Monthly Projection

| Frequency | Runs/Month | Cost/Month |
|-----------|------------|------------|
| 2× daily (current) | 60 | $132-150 |
| 1× daily | 30 | $66-75 |
| 3× daily | 90 | $198-225 |
| 4× daily (original) | 120 | $264-300 |

**Current target met:** ✅ Under $150/month

### Cost Optimization Levers

**Immediate (no code changes):**
1. **Reduce frequency:** 2× daily saves $132/month vs 4× daily
2. **Reduce topics:** 2 topics = $90/month, 1 topic = $45/month
3. **Raise threshold:** 70+ saves ~20% (fewer notes to create)

**Short-term (code changes):**
1. **Incremental fetching:** Only new items since last run
2. **Smart batching:** Group feeds by update frequency
3. **Cache RSS feeds:** Reduce redundant fetches

**Long-term (architecture changes):**
1. **Custom fine-tuned model:** Optimized for RSS scoring
2. **Embedding-based pre-filter:** LLM only scores high-probability items
3. **Differential synthesis:** Only synthesize changed topics

---

## Target Roadmap

### Immediate (Next 7 Days)
- [ ] **Validate production pipeline** (Feb 27 8am run)
- [ ] **Monitor first week** of automated runs
- [ ] **Measure actual costs** vs projected
- [ ] **Test synthesis agent** end-to-end
- [ ] **Verify Discord posting** works reliably

**Success Criteria:**
- 14 successful runs (2× daily for 7 days)
- Discord digests posted reliably
- Cost stays under $3 per run
- 10-20 discovery notes per run

### Short-Term (2-4 Weeks)
- [ ] **Engagement commands:** Wire up `drill in`, `mark engaging`, `not interested`
- [ ] **Weekly meta-discovery:** Test source validation/cleanup automation
- [ ] **Quality assessment:** Review note relevance, adjust threshold if needed
- [ ] **Source expansion:** Add 1-2 high-value topics if budget allows

**Success Criteria:**
- User actively using engagement commands
- Source list stays current (automated cleanup)
- Relevance threshold optimal (60+ vs 70+ analysis)

### Medium-Term (1-3 Months)
- [ ] **Smart bundling:** Group related discoveries
- [ ] **Trend tracking:** Identify recurring themes over time
- [ ] **Conflict detection:** Flag when sources disagree
- [ ] **Multi-channel support:** Telegram, Slack, email options

**Success Criteria:**
- Digests show cross-discovery patterns
- Historical trend analysis available
- Users can choose delivery channel

### Long-Term (3-6 Months)
- [ ] **Custom embedding model:** Pre-filter before LLM scoring
- [ ] **Adaptive scheduling:** Learn optimal run times per source
- [ ] **Collaborative filtering:** Learn from multiple users
- [ ] **API for third-party integration:** Let other tools consume discoveries

**Success Criteria:**
- Cost per run under $1.00
- Runtime under 5 minutes
- Multiple users benefiting from shared discovery
- External tools consuming discovery data

---

## Behavioral Science Integration

### Core Principles Applied

**1. Progressive Disclosure**
- **Research:** Miller's Law (7±2 items in working memory)
- **Application:** 3-layer digest (Signal → Synthesis → Evidence)
- **Why:** Prevents cognitive overload, enables scanning

**2. Information Scent**
- **Research:** Information Foraging Theory (Pirolli & Card)
- **Application:** Each layer provides clues about next layer's value
- **Why:** Users can bail early if not relevant (respects their time)

**3. Serial Position Effect**
- **Research:** People remember first and last items best
- **Application:** Most important findings in positions 1-3
- **Why:** Key insights land even during quick scans

**4. Recognition Over Recall**
- **Research:** Recognition requires less cognitive effort than recall
- **Application:** Show creators, topics, tags prominently
- **Why:** Users can quickly pattern-match to their interests

**5. Closure**
- **Research:** Zeigarnik Effect (incomplete tasks create tension)
- **Application:** Each layer ends with "Continue to..." prompt
- **Why:** Creates natural flow through layers

**6. Actionability**
- **Research:** Implementation Intentions (Gollwitzer)
- **Application:** Every insight tagged with "why this matters"
- **Why:** Users know what to do with the information

### Format Refinements Based on Research

**Before (flat list):**
```
1. New AI model released
2. OpenAI announces feature
3. Blog post about prompting
...
26. Another article
```
**Problem:** Users had to read everything to find what matters

**After (3-layer):**
```
Layer 1: 3 key findings (5s scan)
Layer 2: Cross-source synthesis (30s read)
Layer 3: All 26 discoveries (deep dive)
```
**Improvement:** Users can bail at Layer 1 if not relevant

**Mobile Optimization:**
- No markdown formatting in Discord (bare URLs)
- Emoji for urgency (🔥 1-5 scale)
- Short paragraphs (3-4 lines max)
- Bullet points over prose

**Future Refinements:**
- [ ] A/B test different urgency scales
- [ ] Measure Layer 1 → Layer 2 → Layer 3 conversion rates
- [ ] Test different synthesis formats (narrative vs bullets)
- [ ] Add "estimated read time" per layer

---

## Success Metrics

### Primary KPIs
1. **System Reliability:** % of runs that complete successfully
   - **Target:** >95% (57/60 runs per month)
   
2. **Cost Efficiency:** Average cost per run
   - **Target:** <$3.00
   
3. **Discovery Quality:** % of notes marked "engaging"
   - **Target:** >30% (when engagement tracking enabled)

4. **User Engagement:** Notes reviewed vs created
   - **Target:** >50% of notes get opened

### Secondary KPIs
1. **Source Health:** % of RSS feeds working
   - **Current:** 58/82 (71%)
   - **Target:** >80% (automated cleanup helps)

2. **Synthesis Quality:** Cross-source patterns found
   - **Target:** >2 patterns per digest

3. **Response Time:** Digest posted within X minutes of run start
   - **Target:** <15 minutes

### Qualitative Measures
- Does the digest surface insights you wouldn't have found manually?
- Do the relevance scores match your actual interest?
- Is the synthesis layer adding value vs just listing discoveries?
- Are the engagement commands helping refine future results?

---

## Risk & Mitigation

### Technical Risks

**Risk:** API rate limits during production runs
- **Likelihood:** Low (tested, limits are generous)
- **Impact:** High (missed digest)
- **Mitigation:** Timeouts (20min), retry next scheduled run

**Risk:** Flash-Lite agent failures
- **Likelihood:** Medium (seen timeouts during testing)
- **Impact:** Medium (individual topic skipped)
- **Mitigation:** Error handling in orchestrator, can upgrade to Pro

**Risk:** Sub-agent orchestration breaks
- **Likelihood:** Low (architecture validated)
- **Impact:** High (entire run fails)
- **Mitigation:** Rollback to manual testing, detailed logging

### Cost Risks

**Risk:** Token usage higher than projected
- **Likelihood:** Medium (depends on RSS feed sizes)
- **Impact:** Medium (20-30% cost overrun = still under budget)
- **Mitigation:** Monitor first week, adjust topics if needed

**Risk:** Model price changes
- **Likelihood:** Low (Gemini pricing stable)
- **Impact:** High (could 2× costs overnight)
- **Mitigation:** Model abstraction layer, can swap providers

### Quality Risks

**Risk:** Relevance scores drift over time
- **Likelihood:** Medium (sources change focus)
- **Impact:** Medium (more noise in digests)
- **Mitigation:** Weekly meta-discovery, engagement tracking

**Risk:** Sources go stale/offline
- **Likelihood:** High (seen 40% dead feeds)
- **Impact:** Low (automated cleanup)
- **Mitigation:** Weekly validation, manual review monthly

---

## Lessons Learned

### What Worked Well

1. **Start small, validate early**
   - Tested with 1 topic (openclaw) before scaling to 3
   - Caught Flash-Lite limitations early
   - Saved weeks of debugging

2. **Document as you go**
   - ORCHESTRATOR-3PHASE.md captured working architecture
   - PRODUCTION-READY.md saved for future reference
   - GitHub repo enables rollback

3. **Cost-first thinking**
   - Started with all topics, realized $680/month was too high
   - Reduced early, validated later
   - Now have room to scale up selectively

4. **Behavioral science grounding**
   - 3-layer format designed upfront
   - Prevents "redesign later" churn
   - Mobile-first saves future rework

### What Didn't Work

1. **LLM orchestration (Gemini)**
   - Tried using Flash-Lite and Pro to orchestrate
   - Both misinterpreted instructions
   - **Fix:** Main session (Claude) orchestrates

2. **All Flash-Lite pipeline**
   - Cost-optimized but unreliable
   - Agents started but didn't complete
   - **Fix:** Hybrid Flash-Lite (simple) + Pro (complex)

3. **4× daily schedule**
   - Cost too high before validation
   - AI news cycles don't need 6-hour updates
   - **Fix:** 2× daily, assess real value first

4. **Untested sources**
   - Started with 82 sources, 40% dead
   - Wasted token budget testing bad feeds
   - **Fix:** Cleanup before production, weekly validation

### If We Started Over

1. **Test source quality first** (before building pipeline)
2. **Start with 1 topic, 2× daily** (validate value before scaling)
3. **Use Claude for orchestration from day 1** (don't assume Gemini can do it)
4. **Build engagement tracking earlier** (data-driven threshold tuning)
5. **Add cost monitoring from start** (know per-run costs immediately)

---

## Appendix: Technical Architecture

### Agent Communication Flow

```
Cron (8am/8pm CST)
  ↓
Main Session (Claude Sonnet) wakes
  ↓
Spawns: Prep Agent (Flash-Lite) → ai-news
  ├─ Reads: discoveries/sources/ai-news.md
  ├─ Validates: RSS URLs
  └─ Writes: discoveries/meta/.plans/prep-{timestamp}-ai-news.json
  ↓
Spawns: Fetch Agent (Flash-Lite) → ai-news
  ├─ Reads: prep plan
  ├─ Tests: Each RSS URL (web_fetch)
  └─ Writes: discoveries/meta/.raw/fetch-{timestamp}-ai-news.json
  ↓
Spawns: Score Agent (Pro) → ai-news
  ├─ Reads: fetch results, tracking file
  ├─ Fetches: Working RSS feeds
  ├─ Scores: Each item 0-100
  ├─ Creates: Markdown notes (60+)
  ├─ Updates: tracking file
  └─ Writes: discoveries/meta/.manifests/score-{timestamp}-ai-news.json
  ↓
[Repeat for ai-model-releases, agents]
  ↓
Spawns: Synthesis Agent (Pro)
  ├─ Reads: All 3 manifests
  ├─ Analyzes: Cross-source patterns
  ├─ Generates: 3-layer digest
  ├─ Writes: discoveries/digests/{timestamp}-digest.md
  └─ Posts: Discord #digest channel
  ↓
Main Session returns summary
  ↓
Cron announces completion
```

### File Structure

```
discoveries/
├── sources/              # Topic source lists (user-editable)
│   ├── ai-news.md
│   ├── ai-model-releases.md
│   └── agents.md
├── rss/                  # Discovery notes (auto-generated)
│   └── YYYY-MM-DD-{slug}.md
├── digests/              # Daily digests (auto-generated)
│   └── YYYY-MM-DD-HHMM-digest.md
├── meta/
│   ├── .plans/           # Prep agent output
│   ├── .raw/             # Fetch agent output
│   ├── .manifests/       # Score agent output
│   ├── config.json       # System config
│   └── digest-format-spec.md
├── .discoveries-tracking.json    # Seen URLs
└── .discoveries-engagement.json  # User feedback
```

### GitHub Repository

**URL:** https://github.com/mikeh883/discovery-agents

**Key Files:**
- `PRODUCTION-READY.md` - Deployment guide
- `ORCHESTRATOR-3PHASE.md` - Working architecture
- `MODEL-STRATEGY.md` - Cost/model decisions
- `sources/*.md` - Topic source lists (cleaned)

---

## Contact & Support

**Primary Contact:** Mike Holloway  
**System:** OpenClaw voice-ideation agent  
**Documentation:** This file + GitHub repo  
**Discord:** #digest channel (1476043075355938880)

**Troubleshooting:**
1. Check `openclaw cron runs {jobId}` for run history
2. Review `discoveries/meta/.manifests/` for agent outputs
3. Check Discord #digest for posted digests
4. See PRODUCTION-READY.md for rollback procedures

**Next Review:** 2026-03-05 (after 1 week of production runs)

---

**Version:** 1.0  
**Created:** 2026-02-26  
**Next Update:** After first week of production data
