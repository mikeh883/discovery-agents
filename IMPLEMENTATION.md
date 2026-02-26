# Discovery Agents - Implementation Roadmap

**Goal:** Get from design → working system in 2-3 weeks  
**Approach:** Iterative - start simple, add features progressively

## Phase 1: Foundation (Days 1-3)

### ✓ Design Complete
- [x] Architecture documented
- [x] Meta-discovery spec written
- [x] Content monitor spec written
- [x] Initial topics defined (8 topics)

### Day 1: Directory Structure & Templates
- [ ] Create `discoveries/` folder structure
- [ ] Create note templates
- [ ] Create tracking files (`.json`)
- [ ] Test file creation in Obsidian

**Structure:**
```
discoveries/
  sources/           # Follow lists per topic
  youtube/           # YouTube discoveries
  substack/          # Newsletter discoveries
  twitter/           # Twitter/X discoveries
  rss/               # Other RSS discoveries
  digests/           # Daily digests
  meta/              # Stats, logs, config
  .discoveries-tracking.json
  .discoveries-engagement.json
```

### Day 2-3: Meta-Discovery Agent (Manual Test)
- [ ] Write meta-discovery prompt for one topic
- [ ] Test web search for "LLM Architecture"
- [ ] Manually validate results
- [ ] Generate first source list (`llm-architecture.md`)
- [ ] Verify RSS feeds work (YouTube, Substack)
- [ ] Refine prompts based on results

**Test Command:**
```
"Run meta-discovery for 'LLM Architecture'. 
Find top YouTube channels, Substacks, Twitter accounts.
Save to discoveries/sources/llm-architecture.md"
```

**Success Criteria:**
- Source list created with 10-15 sources
- RSS feeds validated
- Markdown formatting correct

## Phase 2: Basic Content Monitoring (Days 4-7)

### Day 4: RSS Feed Fetcher
- [ ] Build RSS fetch & parse logic
- [ ] Test with 3-5 feeds from source list
- [ ] Extract: title, link, pubDate, description
- [ ] Handle errors gracefully

**Test:**
Fetch Yannic Kilcher's YouTube RSS, extract last 5 videos

### Day 5: Relevance Scoring
- [ ] Create relevance scoring prompt
- [ ] Test with 5 pieces of content
- [ ] Verify scores make sense (60+ = relevant)
- [ ] Tune prompt if needed

**Test:**
Score 5 YouTube videos, expect 2-3 to score 60+

### Day 6: Obsidian Note Creation
- [ ] Create first discovery note from scored content
- [ ] Verify formatting, metadata, links
- [ ] Test with 5 notes
- [ ] Check they appear correctly in Obsidian

### Day 7: End-to-End Test
- [ ] Run full pipeline: fetch → score → create notes
- [ ] Process one topic (LLM Architecture)
- [ ] Generate digest
- [ ] Review output quality

**Success Criteria:**
- 5-10 discoveries created
- Digest looks good
- Notes are useful & well-formatted

## Phase 3: Automation & Scaling (Days 8-14)

### Day 8-9: Discord Notifications
- [ ] Create Discord notification format
- [ ] Test sending digest summary
- [ ] Adjust formatting based on readability
- [ ] Test with different discovery counts (1, 5, 20)

### Day 10-11: Tracking & Deduplication
- [ ] Implement `.discoveries-tracking.json`
- [ ] Test: same content should not be added twice
- [ ] Add "seen" items to tracking file
- [ ] Verify deduplication works

### Day 12: Scale to All Topics
- [ ] Run meta-discovery for remaining 7 topics
- [ ] Generate all source lists
- [ ] Verify RSS feeds for each
- [ ] Total sources: ~60-80 across all topics

### Day 13-14: Cron Job Setup
- [ ] Create content monitor cron (every 6 hours)
- [ ] Create meta-discovery cron (weekly, Sunday 2 AM)
- [ ] Test cron triggers manually
- [ ] Monitor first 24 hours of automated runs

**Cron Schedule:**
- Content Monitor: 6 AM, 12 PM, 6 PM, 12 AM CST
- Meta-Discovery: Sunday 2 AM CST

## Phase 4: Feedback & Enhancement (Days 15-21)

### Day 15-16: Engagement Tracking
- [ ] Implement "mark engaging" command
- [ ] Implement "not interested" command
- [ ] Store in `.discoveries-engagement.json`
- [ ] Test feedback affects future scoring

### Day 17-18: Deep Dive ("Drill In")
- [ ] Create drill-in prompt
- [ ] Test: fetch full content, generate deep analysis
- [ ] Expand note with full details
- [ ] Test with 3-5 different content types

**Test Command:**
```
"Drill in to the Yannic Kilcher video about transformers"
```

### Day 19-20: Related Notes (Vault Search)
- [ ] Search vault for related content when creating note
- [ ] LLM: determine which notes are most related
- [ ] Add [[wikilinks]] to Related Notes section
- [ ] Test with various discovery types

### Day 21: Metrics & Dashboard
- [ ] Implement stats logging (`meta/stats.json`)
- [ ] Track: discoveries per run, engagement rate, etc.
- [ ] Create simple metrics view (Obsidian note)
- [ ] Review first week of data

## Phase 5: Refinement (Ongoing)

### Week 4+
- [ ] Tune relevance thresholds based on engagement
- [ ] Adjust source lists (add/remove based on quality)
- [ ] Optimize prompts for better summaries
- [ ] Add new topics as needed
- [ ] Explore API access (Twitter, YouTube)

## Quick Start (If You Want to Jump Ahead)

### Option A: Manual Single Run (30 min)
Test the concept before building automation:

1. Pick one topic: "LLM Architecture"
2. Web search: "best YouTube channels about LLM architecture"
3. Manually create source list
4. Fetch one RSS feed (e.g., Yannic Kilcher)
5. Score latest video for relevance
6. Create one discovery note
7. Review quality

### Option B: Build Meta-Discovery First (1-2 days)
Get source lists ready before monitoring:

1. Create directory structure
2. Run meta-discovery for all 8 topics (manual or sub-agent)
3. Validate RSS feeds
4. Review source lists
5. Ready for content monitoring

### Option C: Full Build (2-3 weeks)
Follow the roadmap above sequentially.

## Tools & Dependencies

**OpenClaw Tools Used:**
- `web_search` - Meta-discovery, research
- `web_fetch` - RSS fetching, content extraction
- `Write` - Create Obsidian notes, tracking files
- `Read` - Load source lists, tracking data
- `message` - Discord notifications
- `cron` - Schedule agents
- `sessions_spawn` - Run isolated agents

**External (None Required):**
- All RSS-based, no API keys needed
- YouTube RSS, Substack RSS, nitter for Twitter
- LLM scoring uses OpenClaw's default model

**Optional (Future):**
- YouTube Data API (if we get key)
- Twitter API (if we get access)
- Custom RSS parser library

## Testing Strategy

### Unit Tests (Manual)
- Test RSS fetch for each platform
- Test relevance scoring with edge cases
- Test note creation with various content types
- Test Discord notification formatting

### Integration Tests
- End-to-end: source list → discoveries → digest
- Test with 0 discoveries (should handle gracefully)
- Test with 100 discoveries (should not overwhelm)
- Test cron triggers

### User Acceptance
- Review discovery quality (are they useful?)
- Check relevance scores (do they match intuition?)
- Test commands (drill in, mark engaging, etc.)
- Verify Discord notifications are helpful

## Rollout Strategy

### Week 1: Single Topic, Manual
- LLM Architecture only
- Manual runs, no cron
- User reviews every discovery
- Tune prompts heavily

### Week 2: Multi-Topic, Semi-Auto
- All 8 topics
- Cron runs, but user reviews before auto-add
- Refine relevance thresholds
- Scale up source lists

### Week 3: Full Auto
- Auto-add discoveries (60+ relevance)
- Engagement tracking live
- User reviews in natural workflow
- Continuous tuning

### Week 4+: Optimization
- Adjust based on engagement data
- Add advanced features (drill in, related notes)
- Explore new sources/topics
- Consider API integration

## Success Metrics

### Quantity (Week 1)
- [ ] 8 source lists created (10-15 sources each)
- [ ] 5-10 discoveries per run
- [ ] 4 runs per day = 20-40 discoveries daily

### Quality (Week 2)
- [ ] 70%+ of discoveries are relevant (not dismissed)
- [ ] 20%+ engagement rate (marked engaging or opened)
- [ ] 5%+ drill-in rate (user wants deep analysis)

### Value (Week 3+)
- [ ] User finds 2-3 high-value insights per day
- [ ] Saves 1-2 hours of manual discovery time daily
- [ ] Makes 3-5 vault connections per week (related notes)

### System Health (Ongoing)
- [ ] 95%+ RSS fetch success rate
- [ ] <20 min per content monitor run
- [ ] <30 min per meta-discovery run
- [ ] Zero missed cron jobs

## Risk Mitigation

**Risk: Too much content (overwhelm)**
- Mitigation: Start with high threshold (70+), lower if too few
- Mitigation: User can adjust in feedback

**Risk: Too little content (not enough discoveries)**
- Mitigation: Lower threshold to 50+
- Mitigation: Add more sources via meta-discovery

**Risk: Low quality discoveries**
- Mitigation: Tune prompts based on engagement data
- Mitigation: Remove low-quality sources

**Risk: RSS feeds break**
- Mitigation: Graceful error handling, skip and retry
- Mitigation: Weekly meta-discovery refreshes sources

**Risk: User doesn't engage**
- Mitigation: Start with Discord notifications
- Mitigation: Make digest scannable (top picks only)
- Mitigation: "Drill in" makes engagement easy

## Next Steps

**This week:**
1. Create directory structure
2. Run meta-discovery for 2-3 topics (test)
3. Build RSS fetcher
4. Test end-to-end with one topic

**Next week:**
1. Scale to all 8 topics
2. Set up cron jobs
3. Implement Discord notifications
4. Monitor and tune

**Week 3:**
1. Add engagement tracking
2. Build drill-in feature
3. Optimize relevance scoring
4. Review metrics

---

**Ready to start?** Let's begin with Day 1: directory structure and first meta-discovery test.
