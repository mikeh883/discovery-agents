# Cross-Source Synthesis Engine

How to generate Layer 2 (The Synthesis) from multiple discoveries.

## Input

Array of discoveries from this run:
```javascript
discoveries = [
  {
    title: "...",
    creator: "...",
    platform: "youtube|substack|reddit|...",
    topic: ["LLM Architecture", "..."],
    relevance: 85,
    summary: "...",
    key_points: ["...", "..."],
    url: "..."
  },
  // ... more discoveries
]
```

## Process

### Step 1: Group by Topic/Theme

**LLM Prompt:**
```
Analyze these [N] discoveries and identify 2-3 cross-cutting themes.

Discoveries:
[List of titles + summaries]

What are the main themes? Look for:
- Topics mentioned in 3+ discoveries
- Conceptual connections between different sources
- Emerging patterns across platforms

Return: 
- Theme name
- Which discoveries relate to it
- Brief description (max 10 words)
```

**Output:**
```
Themes:
1. "Inference-Time Scaling" - 5 discoveries (YouTube, Substack, News)
2. "Local LLM Viability" - 3 discoveries (Reddit, GitHub, Blog)
3. "Cost vs. Quality Trade-offs" - 4 discoveries (multiple platforms)
```

### Step 2: Detect Consensus vs. Conflict

**LLM Prompt:**
```
For theme: "[Theme Name]"

Related discoveries:
1. [Title] (YouTube) - [summary]
2. [Title] (Reddit) - [summary]
3. [Title] (News) - [summary]

Do these sources agree or disagree?

Identify:
- Consensus points (what everyone says)
- Conflicts (where sources diverge)
- Platform patterns (YouTube vs. Reddit sentiment)

Format:
"Multiple sources ([icons/names]) agree that [consensus]. 
Conflict exists between [platform A] ([sentiment]) and [platform B] ([sentiment]) regarding [specific point]."
```

**Example Output:**
```
Multiple sources (📺 Yannic Kilcher, 📝 Sebastian Raschka, 📰 AI Business) agree that inference-time scaling is becoming the primary LLM improvement method. Conflict exists between 🧵 Reddit (concerned about costs) and 📺 YouTube (optimistic about ROI).
```

### Step 3: Sentiment Heatmap

**Per Platform Analysis:**

For each platform (YouTube, Reddit, News, Blogs, etc.):

1. **Count discoveries** from that platform
2. **Analyze sentiment** in summaries/key points:
   - Positive indicators: "breakthrough", "impressive", "game-changing", "finally"
   - Negative indicators: "concerning", "overblown", "skeptical", "disappointing"
   - Neutral indicators: "interesting", "notable", "worth watching"
3. **Measure intensity:**
   - High: Lots of superlatives, exclamation points, strong language
   - Medium: Measured enthusiasm or criticism
   - Low: Factual, neutral reporting
4. **Context from engagement** (if available):
   - High engagement + positive = "Highly Enthusiastic"
   - Mixed signals = "Cautiously Optimistic"
   - Low engagement = "Neutral"

**Sentiment Scale:**
```
Positive Direction:
- Highly Enthusiastic (80%+ positive, strong language)
- Enthusiastic (60-80% positive)
- Optimistic (mixed but leaning positive)

Neutral:
- Neutral-Positive (factual but favorable tone)
- Neutral (pure reporting, no clear lean)
- Neutral-Negative (factual but cautious tone)

Negative Direction:
- Skeptical (mixed but leaning negative)
- Critical (60-80% negative)
- Highly Critical (80%+ negative, strong language)
```

**Output Format:**
```
📺 YouTube: Highly Enthusiastic (5 videos, 82% positive mentions, demo-heavy)
🧵 Reddit: Cautiously Optimistic (mixed sentiment - excited about tech, worried about cost)
📰 News: Neutral-Positive (industry trend coverage, citing major players)
📝 Blogs: Enthusiastic (3 technical deep-dives, working code, practical focus)
```

### Step 4: Generate Consensus Statement

**Template:**
```
[Quantifier] sources ([list with icons]) [verb] that [main finding]. 

[Optional conflict]: Conflict exists between [platform/source A] ([their view]) and [platform/source B] ([their view]) regarding [specific aspect].

[Optional outlier]: [Source] offers a contrarian view, arguing [position].
```

**Quantifiers:**
- 5+ sources: "Multiple sources" or "Unanimous agreement"
- 3-4 sources: "Several sources" or "Broad consensus"
- 2 sources: "Both X and Y" (name specifically)
- 1 source + theme across discoveries: "Emerging pattern"

**Verbs:**
- Strong agreement: "agree", "confirm", "validate"
- Leaning: "suggest", "indicate", "point to"
- Conflict: "disagree", "diverge", "conflict"

### Step 5: Cross-Reference with User Interests

**Check against config topics:**
```javascript
user_topics = [
  "LLM Architecture",
  "AI News",
  "AI Model Releases",
  "Agents",
  "AI Frameworks",
  "Vibe Coding",
  "OpenClaw",
  "Local LLM"
]

// For each theme, note which user topics it relates to
theme_relevance = {
  "Inference-Time Scaling": ["LLM Architecture", "Agents", "Local LLM"],
  "Cost Trade-offs": ["Local LLM", "AI Frameworks"]
}
```

This helps explain "Why Flagged" in Layer 3.

---

## Example Implementation

**Input:** 8 discoveries from morning run

**Step 1 - Themes:**
```
1. Inference-Time Scaling (5 discoveries)
2. Local LLM Viability (3 discoveries)
3. Security Updates (2 discoveries - OpenClaw CVEs)
```

**Step 2 - Consensus (Theme 1):**
```
Multiple sources (📺 Yannic Kilcher, 📝 Sebastian Raschka, 📰 TechCrunch, 🧵 r/LocalLLaMA) agree that inference-time compute is replacing model size as the primary scaling law. Conflict exists between 🧵 Reddit (skeptical of costs) and 📺 YouTube (optimistic about ROI).
```

**Step 3 - Sentiment (Theme 1):**
```
📺 YouTube: Highly Enthusiastic (3 videos, demos, "game-changing")
🧵 Reddit: Cautiously Optimistic (excited about capability, concerned about production costs)
📰 News: Neutral-Positive (industry trend, citing OpenAI/Anthropic shifts)
📝 Blogs: Enthusiastic (working code, 3.5× improvement results)
```

**Step 4 - Output (Layer 2):**
```markdown
### CONSENSUS
Multiple sources (📺 Yannic Kilcher, 📝 Sebastian Raschka, 📰 TechCrunch, 🧵 r/LocalLLaMA) agree that inference-time compute is replacing model size as the primary scaling law. Conflict exists between 🧵 Reddit (skeptical of costs) and 📺 YouTube (optimistic about ROI).

### THEMES
**[Inference-Time Scaling]** Search and self-refinement replacing "bigger is better" paradigm  
**[Local LLM Viability]** 7B models + inference techniques now competitive with 70B  
**[Security]** OpenClaw CVE patches released, update recommended

### SENTIMENT HEATMAP
📺 **YouTube:** Highly Enthusiastic (3 videos, demos, "game-changing" language)  
🧵 **Reddit:** Cautiously Optimistic (excited about tech, worried about costs)  
📰 **News:** Neutral-Positive (industry trend coverage)  
📝 **Blogs:** Enthusiastic (working code, 3.5× measured improvements)
```

---

## Quality Checks

**Good synthesis:**
- ✓ Shows connections between sources
- ✓ Highlights agreement AND disagreement
- ✓ Platform-specific sentiment patterns
- ✓ Actionable themes
- ✓ Explains WHY this matters

**Bad synthesis:**
- ✗ Just lists discoveries separately
- ✗ No cross-referencing
- ✗ Vague themes ("AI stuff")
- ✗ No sentiment analysis
- ✗ Missing conflicts/nuance

---

## LLM Prompts (Production-Ready)

### Theme Extraction Prompt
```
Analyze these discoveries and identify 2-3 cross-cutting themes:

[Discovery 1]: [Title] ([Platform]) - [Summary]
[Discovery 2]: ...
...

Identify themes that:
- Appear in 3+ discoveries (or 2 if high-quality sources)
- Are actionable/specific
- Connect different platforms/viewpoints

Return JSON:
{
  "themes": [
    {
      "name": "Theme Name",
      "description": "Brief description (max 10 words)",
      "discovery_ids": [1, 3, 5],
      "relevance_to_user": ["LLM Architecture", "Agents"]
    }
  ]
}
```

### Consensus Detection Prompt
```
For theme: "[Theme Name]"

Related discoveries:
1. [Discovery details with platform, creator, summary]
2. ...

Analyze consensus and conflicts:
1. What do sources agree on?
2. Where do they disagree?
3. Are there platform-based patterns? (e.g., YouTube optimistic, Reddit skeptical)

Return:
- Consensus statement (1-2 sentences)
- Conflict description (if exists)
- Outliers (if any)
```

### Sentiment Analysis Prompt
```
For each platform with discoveries in this run:

Platform: YouTube
Discoveries:
1. [Title] - [Summary] - [Key points]
2. ...

Analyze sentiment:
- Overall tone (Highly Enthusiastic / Enthusiastic / Optimistic / Neutral / Skeptical / Critical)
- Evidence (quotes, language intensity, engagement if available)
- Context (why this sentiment?)

Return format: "[Platform]: [Sentiment] ([context])"
```

---

*This synthesis engine transforms individual discoveries into **insight** - the real value of aggregation.*
