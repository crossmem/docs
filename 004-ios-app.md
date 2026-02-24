# Spec 004: iOS App MVP

> Share Sheet → Topic Feed → Drive. That's the whole app.

## Core Flow

```
Any App (Twitter, Safari, Notes, ...)
  → Share Sheet → CrossMem
  → Extract: URL, text, image (OCR if needed)
  → Claude API: generate title + summary + 3 suggested topics
  → User taps topic (or creates new)
  → Save to Drive: crossmem/topics/{topic-name}/{date}_{source}.md
  → Topic feed updates
  → Done. MCP can now find it. Any AI agent can read it.
```

## Screens

### 1. Topic Feed (home)
Slack-like list of topics, most recently updated first.

```
┌─────────────────────────────┐
│ CrossMem              🔍 ⚙️ │
├─────────────────────────────┤
│                             │
│ ┌─────────────────────────┐ │
│ │ 📁 React Server Comp.   │ │
│ │ 3 sources · 2h ago      │ │
│ │ Latest: RSC streaming…  │ │
│ └─────────────────────────┘ │
│                             │
│ ┌─────────────────────────┐ │
│ │ 📁 Pricing Strategy     │ │
│ │ 7 sources · 1d ago      │ │
│ │ Latest: SaaS pricing…   │ │
│ └─────────────────────────┘ │
│                             │
│ ┌─────────────────────────┐ │
│ │ 📁 MCP Protocol         │ │
│ │ 12 sources · 3d ago     │ │
│ │ Latest: Anthropic ann…  │ │
│ └─────────────────────────┘ │
│                             │
│            ＋                │
│      New Topic              │
└─────────────────────────────┘
```

### 2. Share Sheet (primary input)
Standard iOS Share Extension. Receives URL, text, or image.

```
┌─────────────────────────────┐
│ Save to CrossMem            │
├─────────────────────────────┤
│                             │
│ 📄 "How to price your SaaS" │
│    blog.example.com         │
│                             │
│ Select topic:               │
│                             │
│ ⭐ Pricing Strategy (suggested) │
│ ○ Marketing                 │
│ ○ SaaS Playbook            │
│ ＋ New topic...             │
│                             │
│ ┌─────────────────────────┐ │
│ │       Save              │ │
│ └─────────────────────────┘ │
└─────────────────────────────┘
```

Topic suggestion comes from:
1. Keyword match against existing topic names/tags
2. If no match: Claude API haiku call (title + content → top 3 existing topics or "new")

### 3. Topic Detail
List of sources + AI summary (if generated).

```
┌─────────────────────────────┐
│ ← Pricing Strategy    ⋯    │
├─────────────────────────────┤
│                             │
│ 📝 Summary                  │
│ Research on SaaS pricing    │
│ models: freemium vs usage   │
│ vs seat-based. Key insight: │
│ ... [Show more]             │
│                             │
│ ─────────────────────────── │
│ Sources (7)                 │
│                             │
│ 🔗 How to price your SaaS   │
│    blog.example.com · 2h    │
│                             │
│ 🔗 Stripe pricing page      │
│    stripe.com · 1d          │
│                             │
│ 💬 Claude chat: pricing     │
│    claude.ai · 3d           │
│                             │
│ 📸 Screenshot: competitor   │
│    iOS share · 5d           │
│                             │
└─────────────────────────────┘
```

### 4. Search
Global search across all topics and sources.

```
┌─────────────────────────────┐
│ 🔍 pricing model            │
├─────────────────────────────┤
│                             │
│ In "Pricing Strategy":      │
│   How to price your SaaS    │
│   "...usage-based pricing   │
│   model works best for..."  │
│                             │
│ In "SaaS Playbook":         │
│   Revenue benchmarks 2026   │
│   "...median pricing at..." │
│                             │
└─────────────────────────────┘
```

## Tech Stack

- **Language**: Swift, SwiftUI
- **Target**: iOS 17+
- **Auth**: Google Sign-In SDK (same OAuth as extension)
- **Drive**: GoogleAPIClientForREST/Drive
- **AI**: Claude API (Haiku) for topic suggestion + summary
- **OCR**: Apple Vision framework (on-device, for screenshots)
- **Storage**: Local cache in UserDefaults / SwiftData for topic index
- **Share Extension**: App Group for shared container

## Drive Integration

Uses same OAuth scope as extension: `drive.file` (only files created by CrossMem).

```swift
// Save source to topic
let topicFolder = "crossmem/topics/\(topicName)"
let fileName = "\(date)_source_\(slugTitle).md"
let content = """
# \(title)
Source: \(url)
Saved: \(date) via iOS

---

\(extractedContent)
"""
// Upload to Drive folder
// Update _topic.json (increment sourceCount, update updatedAt)
```

## AI Integration

### Topic Suggestion (on share)
```
Model: claude-haiku-4-5
Prompt: Given this content title and URL, suggest the best matching topic
        from this list: [{existing topics}]. If none match, suggest a new
        topic name. Return JSON: { "suggestions": ["topic1", "topic2", "new: Topic Name"] }
Token budget: ~200 tokens (cheap, fast)
```

### Research Summary (background, per topic)
```
Model: claude-haiku-4-5
Prompt: Summarize the key insights from these {N} sources about "{topic}".
        Focus on: what I should know, what's actionable, what's surprising.
        Keep under 300 words.
Token budget: ~1000 tokens
Trigger: when topic gets 3+ sources, or manually
```

## Offline Behavior

- Topic list cached locally (SwiftData)
- Share Sheet works offline → queue upload, sync when online
- Search requires network (queries Drive)

## Phase 2: Auto-Clustering (future, not MVP)

When a source is shared without selecting a topic:
1. Generate embedding (Claude API or on-device ML)
2. Compare against existing topic embeddings
3. If similarity > threshold → auto-assign
4. If no match → create new topic with AI-suggested name
5. User can always reassign via drag-and-drop in topic feed

This is explicitly NOT in MVP. MVP always asks user to pick/create topic.
