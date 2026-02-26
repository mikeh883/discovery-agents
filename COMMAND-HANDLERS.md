# Discovery Agent Command Handlers

Implementation logic for user commands.

## Handler: Drill In

**Trigger detection:**
```
if message.match(/drill in(to)?\s+(.+)/i):
  title_query = match[2]
  execute_drill_in(title_query)
```

**Process:**

### Step 1: Find Discovery Note
```javascript
// Search discoveries/ for matching title
const files = glob('discoveries/**/*.md')
const matches = files.filter(f => {
  const content = read(f)
  const title = extract_title(content)
  return fuzzy_match(title, title_query)
})

if (matches.length === 0):
  return "Couldn't find that discovery. Try: 'search discoveries [query]'"
if (matches.length > 1):
  return "Multiple matches: [list]. Which one?"
  
const note = matches[0]
```

### Step 2: Get Full Content
```javascript
const metadata = parse_frontmatter(note)
const url = metadata.url
const platform = metadata.source

// Fetch full content based on platform
if (platform === 'youtube'):
  transcript = fetch_youtube_transcript(url)
  full_content = transcript
else if (platform === 'substack'):
  article = web_fetch(url, {extractMode: 'markdown'})
  full_content = article
else:
  full_content = web_fetch(url, {extractMode: 'text'})
```

### Step 3: Deep Analysis (LLM)
```
Prompt:
---
Perform a deep analysis of this content:

**Content:**
[full_content]

**Original summary:**
[brief summary from discovery note]

**User's interests:**
- Topics: [from config]
- Recent vault activity: [recent note titles]
- Engaged with: [similar engaged items]

Provide:

1. **Detailed Summary** (500+ words)
   - Main arguments/findings
   - Supporting evidence
   - Context and background

2. **Technical Breakdown**
   - Key concepts explained
   - Implementation details (if applicable)
   - Architecture/methodology

3. **Key Insights & Takeaways**
   - 5-7 main points
   - What makes this significant
   - Implications for the field

4. **Connections to User's Vault**
   - Related notes (based on vault search)
   - How this connects to their work
   - Suggested links to add

5. **Action Items**
   - Things to try/build
   - Follow-up resources
   - Questions to explore

6. **Related Resources**
   - Papers/articles cited
   - Tools/libraries mentioned
   - Similar content to explore

Format as markdown for insertion into discovery note.
```

### Step 4: Update Note
```javascript
// Insert deep analysis after "## Summary" section
const updated = insert_after_section(
  note_content,
  "## Summary",
  `
## Deep Dive Analysis

${deep_analysis}

---

*Deep dive generated: ${timestamp}*
*Triggered by user command: "drill in"*
`
)

write(note_path, updated)
```

### Step 5: Reply to User
```
📊 **Deep Dive: [Title]**

[First 2-3 paragraphs of detailed summary]

**Key Insights:**
[Bullet list of top 5 insights]

**Connections:**
- [[Related note 1]]
- [[Related note 2]]

Full analysis added to: [note path]
Want me to find related resources? Say "more like this"
```

---

## Handler: Mark Engaging

**Trigger detection:**
```
if message.match(/mark (engaging|interesting|good)/i):
  if has_recent_context():
    title = last_shown_item.title
  else if message.includes(title_query):
    title = extract_title_from_message()
  else:
    return "Which discovery? (provide title or say 'the last one')"
    
  execute_mark_engaging(title)
```

**Process:**

### Step 1: Find Discovery
```javascript
const note = find_discovery_by_title(title)
const metadata = parse_frontmatter(note)
```

### Step 2: Update Engagement File
```javascript
const engagement = read_json('.discoveries-engagement.json')

engagement.engaged.push({
  url: metadata.url,
  title: extract_title(note),
  topics: metadata.topic, // array
  creator: metadata.creator,
  platform: metadata.source,
  relevance_score: metadata.relevance,
  marked_at: new Date().toISOString(),
  note_path: note_path
})

write_json('.discoveries-engagement.json', engagement)
```

### Step 3: Analyze Pattern
```javascript
// Extract patterns from all engaged items
const topics = engagement.engaged.flatMap(e => e.topics)
const top_topics = count_frequency(topics).slice(0, 3)

const creators = engagement.engaged.map(e => e.creator)
const top_creators = count_frequency(creators).slice(0, 3)

const platforms = engagement.engaged.map(e => e.platform)
const preferred_platforms = count_frequency(platforms)
```

### Step 4: Reply
```
✓ **Marked as engaging**

I'll prioritize:
- More from ${creator} (+15 relevance boost)
- ${top_topics.join(', ')} content (+10 boost)
- ${platform} discoveries (+5 boost)

**Your engagement patterns:**
Favorite topics: ${top_topics.join(', ')}
Favorite creators: ${top_creators.join(', ')}
Total engaged: ${engagement.engaged.length}

This helps me learn what you find valuable!
```

---

## Handler: Not Interested

**Similar to Mark Engaging, but:**
- Adds to `engagement.dismissed` array
- Applies negative boosts in future scoring
- Option to remove note: "Remove the note too? (yes/no)"

---

## Relevance Scoring with Engagement

**Updated scoring algorithm:**

```javascript
function score_relevance(content, metadata) {
  // Base LLM score (0-100)
  let score = llm_score_relevance(content, user_topics)
  
  // Load engagement data
  const engagement = read_json('.discoveries-engagement.json')
  
  // Boost if creator previously engaged
  const engaged_creators = engagement.engaged.map(e => e.creator)
  if (engaged_creators.includes(metadata.creator)) {
    score += 15
    console.log(`+15: Previously engaged with ${metadata.creator}`)
  }
  
  // Boost if topic overlap with engaged items
  const engaged_topics = engagement.engaged.flatMap(e => e.topics)
  const topic_overlap = metadata.topics.filter(t => 
    engaged_topics.includes(t)
  )
  if (topic_overlap.length > 0) {
    score += 10 * topic_overlap.length
    console.log(`+${10 * topic_overlap.length}: Topic overlap`)
  }
  
  // Boost if same platform as engaged items
  const engaged_platforms = engagement.engaged.map(e => e.platform)
  const platform_freq = count_frequency(engaged_platforms)
  if (platform_freq[metadata.platform] > 3) {
    score += 5
    console.log(`+5: Preferred platform (${metadata.platform})`)
  }
  
  // Penalty if creator previously dismissed
  const dismissed_creators = engagement.dismissed.map(d => d.creator)
  if (dismissed_creators.includes(metadata.creator)) {
    score -= 20
    console.log(`-20: Previously dismissed ${metadata.creator}`)
  }
  
  // Penalty if topic dismissed
  const dismissed_topics = engagement.dismissed.flatMap(d => d.topics)
  const dismissed_overlap = metadata.topics.filter(t =>
    dismissed_topics.includes(t)
  )
  if (dismissed_overlap.length > 0) {
    score -= 10 * dismissed_overlap.length
    console.log(`-${10 * dismissed_overlap.length}: Dismissed topics`)
  }
  
  // Clamp score to 0-100
  return Math.max(0, Math.min(100, score))
}
```

---

## Handler: Show Today's Discoveries

```
User: "show today" or "what did you find today"

Agent:
📋 **Today's Discoveries** - [Date]

Found [N] items across [M] topics

🔥 High Priority ([count]):
1. [Title] - [Creator] ([score])
2. ...

⭐ Worth Reading ([count]):
1. [Title] - [Creator] ([score])
2. ...

Full digest: discoveries/digests/[date]-[time]-digest.md

Want to drill into any? Say "drill in [title]"
```

---

## Context Tracking

**Maintain session context:**
```javascript
session_context = {
  last_digest_shown: null,
  last_item_shown: null,
  current_topic_filter: null,
  recent_searches: []
}

// Update when showing digest
session_context.last_digest_shown = digest_path
session_context.last_item_shown = digest.items[0]

// Allow contextual commands
if (message === "drill in" || message === "drill in this"):
  drill_in(session_context.last_item_shown.title)
  
if (message === "mark engaging"):
  mark_engaging(session_context.last_item_shown.title)
```

---

## Implementation Priority

**Phase 1 (Now):**
- ✅ Command spec written
- 🔄 Drill in handler
- 🔄 Mark engaging handler

**Phase 2 (Soon):**
- Not interested handler
- Show today handler
- Context tracking

**Phase 3 (Later):**
- More like this
- Search discoveries
- Analytics

---

Ready to test? Try:
1. "drill in raschka" - Deep analysis
2. "mark engaging raschka" - Learning feedback
3. "show today" - See digest
