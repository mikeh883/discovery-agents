# Discovery Agent Commands

User commands for interacting with discoveries after digests are posted.

## Available Commands

### 1. Drill In (Deep Analysis)

**Trigger:** "drill in [title]" or "drill in [partial title]"

**What it does:**
1. Finds the discovery note by title match
2. Fetches full content (YouTube transcript, full article text)
3. Performs deep LLM analysis:
   - Detailed summary (500+ words)
   - Technical breakdown
   - Key insights & takeaways
   - Connections to user's vault
   - Action items / follow-up questions
   - Related resources
4. Expands the Obsidian note with full analysis
5. Returns deep dive in chat

**Example:**
```
User: "drill in raschka inference scaling"

Agent: 
📊 **Deep Dive: Categories of Inference-Time Scaling**

[Full detailed analysis, 800+ words]

**Technical Breakdown:**
- Chain-of-Thought: Forces reasoning steps...
- Self-Consistency: Generate N solutions, pick most common...
[etc.]

**Key Insights:**
1. Inference scaling is now standard in production LLMs
2. o1/o3 likely use search-based methods...
[etc.]

**Connections to Your Vault:**
- Links to [[Reasoning Models]]
- Related to your [[Agent Architecture]] notes
[etc.]

I've updated the note with this full analysis.
```

**Implementation:**
- Search `discoveries/` for matching title
- Use `web_fetch` to get full content
- LLM prompt: "Deep dive analysis of [content]"
- Update note, reply with summary

---

### 2. Mark Engaging

**Trigger:** "mark engaging [title]" or "this is engaging" (on last shown item)

**What it does:**
1. Finds the discovery note
2. Adds to `.discoveries-engagement.json`:
```json
{
  "engaged": [
    {
      "url": "...",
      "title": "...",
      "topics": ["LLM Architecture"],
      "creator": "Sebastian Raschka",
      "platform": "substack",
      "marked_at": "2026-02-25T12:30:00Z"
    }
  ]
}
```
3. Feedback loop: Future relevance scoring will:
   - Boost similar topics (+10 points)
   - Boost same creator (+15 points)
   - Boost same platform type (+5 points)
4. Confirms: "✓ Marked engaging - I'll prioritize similar content"

**Example:**
```
User: "mark engaging raschka"

Agent: 
✓ **Marked as engaging**

I'll prioritize:
- More from Sebastian Raschka
- Inference optimization content
- Technical deep dives with code
- Substack newsletters in this style

Your engagement helps me learn what you find valuable!
```

**Learning over time:**
- After 10+ engaged items, patterns emerge
- After 50+, recommendations get highly tuned
- Weekly meta-discovery uses engagement data to find similar creators

---

### 3. Not Interested

**Trigger:** "not interested [title]" or "hide this"

**What it does:**
1. Adds to `.discoveries-engagement.json` dismissed list
2. Future scoring penalizes:
   - Same creator (-20 points)
   - Similar topics (-10 points)
3. Optionally removes note from discoveries/

**Example:**
```
User: "not interested in that twitter thread"

Agent:
✓ **Marked as not interested**

I'll filter out:
- Similar Twitter content
- This creator's future posts
- This topic/style

You can always adjust filters later.
```

---

### 4. Show Me More Like This

**Trigger:** "more like [title]" or "find similar to this"

**What it does:**
1. Extracts key characteristics (topic, creator, platform, style)
2. Searches vault for related discoveries
3. Web searches for additional similar content
4. Returns 3-5 recommendations

**Example:**
```
User: "more like the raschka article"

Agent:
🔍 **Similar to "Categories of Inference-Time Scaling"**

**From your discoveries:**
1. [Another Raschka article from last week]
2. [LangChain inference optimization guide]

**New finds:**
- "Optimizing LLM Inference at Scale" - Chip Huyen
- "Production Inference Patterns" - Modal Labs

Want me to add any of these to your discoveries?
```

---

### 5. Search Discoveries

**Trigger:** "search discoveries [query]"

**What it does:**
- Full-text search across all discovery notes
- Returns ranked results
- Can filter by: topic, creator, platform, date range

**Example:**
```
User: "search discoveries agents"

Agent:
📋 **Found 12 discoveries about "agents"**

Top results:
1. [Matt Schlicht's AI Newsletter - Agents]
2. [LangChain Blog - Multi-Agent Systems]
3. [CrewAI Release Notes]
...
```

---

### 6. Show Today's Discoveries

**Trigger:** "show today" or "what did you find today"

**What it does:**
- Shows latest digest
- Quick summary of discoveries
- Option to drill into any item

---

## Implementation Notes

**Context tracking:**
- Track last shown digest/item in session
- Allows "drill in this" without specifying title
- "mark engaging" defaults to last item

**Title matching:**
- Fuzzy match (allow typos, partial titles)
- If multiple matches, show options
- If no match, search by topic/creator

**Engagement scoring algorithm:**
```
Base score (LLM relevance): 0-100

Adjustments:
+ Same creator previously engaged: +15
+ Similar topic previously engaged: +10
+ Same platform type: +5
+ Creator previously dismissed: -20
+ Topic previously dismissed: -10

Final score = Base + Adjustments
Threshold: 60+ = auto-add
```

**Privacy:**
- All engagement data stays local
- `.discoveries-engagement.json` is user-editable
- Can bulk import/export preferences

---

## Future Enhancements

**Smart suggestions:**
- "Based on your interests, you might like..."
- Weekly digest of highly relevant but below-threshold content
- "Trending in your topics" summary

**Social features:**
- Share discoveries with others
- Import someone else's engaged list
- Collaborative filtering

**Analytics:**
- "Show my engagement stats"
- Topic heat map
- Creator leaderboard
- Time-of-day preferences

---

Ready to implement these commands. Priority?
1. Drill in (most requested)
2. Mark engaging (learning feedback)
3. Both now
