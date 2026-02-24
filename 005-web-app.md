# Spec 005: Web App — Search + Point Cloud

> crossmem.dev: landing page (now) → search + visualization (next)

## Overview

The web app is NOT a dashboard for managing files. It's a **knowledge map** — a bird's-eye view of everything you've captured, with search to dive in.

Agent-first design: the web app is for humans who want to visually explore. For querying, use your AI agent via MCP.

## Screens

### 1. Landing Page (exists)
Keep current: value prop + CWS install CTA + waitlist.
Add: link to /explore (public demo with sample data).

### 2. Search (`/search`)
Full-text search across all Drive content. Auth required (Google OAuth, same as extension).

```
┌────────────────────────────────────────────┐
│ CrossMem                    [Sign In]      │
├────────────────────────────────────────────┤
│                                            │
│  🔍 [search across all your AI memory   ]  │
│                                            │
│  Filters: [All] [Conversations] [Topics]   │
│           [Web Pages] [Artifacts]          │
│                                            │
│  ┌──────────────────────────────────────┐  │
│  │ 💬 React Architecture Discussion     │  │
│  │ claude.ai · 2026-02-24 · 24 msgs    │  │
│  │ "...server components streaming..."  │  │
│  └──────────────────────────────────────┘  │
│  ┌──────────────────────────────────────┐  │
│  │ 📁 Topic: React Server Components    │  │
│  │ 5 sources · updated 2h ago          │  │
│  │ "...RSC migration strategy..."      │  │
│  └──────────────────────────────────────┘  │
│                                            │
└────────────────────────────────────────────┘
```

### 3. Point Cloud (`/explore`)
Visual map of all captured knowledge. Each dot = one piece of content. Proximity = semantic similarity.

```
┌────────────────────────────────────────────┐
│ CrossMem · Knowledge Map       [2D] [3D]  │
├────────────────────────────────────────────┤
│                                            │
│     ·  ·                                   │
│    · React ·    · ·                        │
│     · · ·      · Pricing ·                 │
│                 · · ·                      │
│   · ·                      · ·             │
│  · MCP ·                  · Infra ·        │
│   · · ·                    · ·             │
│         · ·                                │
│        · ML · ·                            │
│         · ·                                │
│                                            │
│  Hover: show title                         │
│  Click: open source / topic detail         │
│  Colors: by topic / by platform / by date  │
│                                            │
└────────────────────────────────────────────┘
```

## Tech Stack

- **Framework**: Next.js (existing)
- **Auth**: Google OAuth (existing extension credentials, same project)
- **Drive access**: Google Drive API (server-side, via OAuth token)
- **Search**: Drive full-text search API (`q: "fullText contains 'query'"`)
- **Point cloud**: Three.js or react-three-fiber (3D) / d3-force (2D)
- **Embeddings for visualization**: Claude API batch embed on save, cache in `_topic.json`

## Embedding Strategy (for point cloud only, not RAG)

On save (any source — extension, iOS, telegram):
1. Extract first 500 tokens of content
2. Call embedding API (or hash-based approximation for MVP)
3. Store 2D/3D coordinates in sidecar metadata
4. Web app reads coordinates, renders point cloud

MVP shortcut: use TF-IDF + t-SNE on titles/tags instead of real embeddings. Fast, no API cost, good enough for visualization.

## Auth Flow

```
User clicks "Sign In" on crossmem.dev
  → Google OAuth (same client ID as extension)
  → Scope: drive.file (read/write crossmem files only)
  → Token stored in httpOnly cookie
  → Server-side Drive API calls
```

User's data never leaves Google's infra — web app is just a viewer.

## Phase 2 Features (not MVP)

- **Topic management**: rename, merge, archive topics from web
- **Conversation viewer**: rendered markdown with syntax highlighting
- **Activity heatmap**: when do you save the most? By platform?
- **Sharing**: public link for a topic (read-only)
- **RAG search**: vector search instead of Drive full-text (requires embedding DB)
