# Spec 002: Drive Schema

> Canonical schema for Google Drive storage. All repos must follow this.

## Folder Structure

```
Google Drive/
  crossmem/
    ├── conversations/          ← Extension: auto-saved AI conversations
    │   ├── claude.ai/
    │   │   ├── 2026-02-24_React-Architecture.md
    │   │   └── .crossmem-meta.json
    │   ├── chatgpt.com/
    │   └── gemini.google.com/
    │
    ├── topics/                 ← iOS App + Web: user-curated knowledge clusters
    │   ├── react-server-components/
    │   │   ├── _topic.json            ← topic metadata
    │   │   ├── 2026-02-24_source_twitter-thread.md
    │   │   ├── 2026-02-24_source_blog-post.md
    │   │   └── 2026-02-25_summary.md  ← AI-generated research summary
    │   └── pricing-strategy/
    │       ├── _topic.json
    │       └── ...
    │
    ├── artifacts/              ← Extension: extracted code blocks, SVGs, etc.
    │   ├── claude.ai/
    │   └── chatgpt.com/
    │
    └── web/                    ← Extension "Save This Page" + Telegram /save
        ├── 2026-02-24_hackernews.com_Show-HN-CrossMem.md
        └── .crossmem-meta.json
```

## Sidecar Metadata: `.crossmem-meta.json`

One per folder (conversations, web) or per topic (`_topic.json`).

### Conversation metadata (per file in `.crossmem-meta.json`)

```json
{
  "files": {
    "2026-02-24_React-Architecture.md": {
      "platform": "claude.ai",
      "title": "React Architecture Discussion",
      "hash": "sha256:abc123...",
      "savedAt": "2026-02-24T10:30:00Z",
      "messageCount": 24,
      "fileId": "drive-file-id"
    }
  }
}
```

### Topic metadata (`_topic.json`)

```json
{
  "name": "react-server-components",
  "displayName": "React Server Components",
  "createdAt": "2026-02-24T10:00:00Z",
  "updatedAt": "2026-02-25T14:00:00Z",
  "sourceCount": 5,
  "tags": ["react", "frontend", "architecture"],
  "summary": "Research on RSC patterns, streaming, and migration strategies.",
  "status": "active"
}
```

### Web page metadata (per file in `.crossmem-meta.json`)

```json
{
  "files": {
    "2026-02-24_hackernews.com_Show-HN.md": {
      "sourceUrl": "https://news.ycombinator.com/item?id=12345",
      "domain": "hackernews.com",
      "title": "Show HN: CrossMem",
      "savedAt": "2026-02-24T10:30:00Z",
      "savedFrom": "extension|ios|telegram",
      "hash": "sha256:def456...",
      "fileId": "drive-file-id",
      "topicName": null
    }
  }
}
```

## File Naming Convention

```
{date}_{source-type}_{slugified-title}.md

Examples:
  2026-02-24_React-Architecture.md              (conversation)
  2026-02-24_source_twitter-thread.md            (topic source)
  2026-02-24_summary.md                          (AI research summary)
  2026-02-24_hackernews.com_Show-HN-CrossMem.md  (web page)
```

## Deduplication

- SHA-256 of first 10KB of markdown content
- Hash stored in sidecar metadata
- Before upload: check hash against existing entries
- If match: skip upload, return existing fileId

## Topic Lifecycle

```
created → active → archived
           ↑
           └── new source added (reactivate if archived)
```

Topics are never deleted — only archived. This preserves MCP search results.
