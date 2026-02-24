# CrossMem Architecture

> The canonical reference for all CrossMem repos.
> Every repo's CLAUDE.md should point here: `See: https://github.com/crossmem/specs/ARCHITECTURE.md`

## Vision

CrossMem = Tailscale for AI memory.

Not SSH mesh, but knowledge mesh. Every content source is a node, every AI agent is a node, Google Drive + MCP is the mesh layer.

"I see something → my AI team instantly gets it → immediately actionable."

## System Overview

```
                    ┌─────────────────────────────────────┐
                    │         Google Drive (Mesh)          │
                    │                                     │
                    │  crossmem/                          │
                    │    ├── conversations/{platform}/    │
                    │    ├── topics/{topic-name}/         │
                    │    ├── artifacts/                   │
                    │    └── .crossmem-meta.json (each)   │
                    └──────────┬──────────────────────────┘
                               │
           ┌───────────────────┼───────────────────┐
           │                   │                   │
    ┌──────┴──────┐    ┌──────┴──────┐    ┌──────┴──────┐
    │   Inputs    │    │   Storage   │    │   Outputs   │
    │             │    │   + Index   │    │             │
    │ Extension   │    │             │    │ Claude.ai   │
    │ iOS App     │    │ Drive API   │    │ ChatGPT     │
    │ Telegram    │    │ MCP Server  │    │ Claude Code │
    │ Share Sheet │    │             │    │ Cursor      │
    │ Web Clipper │    │             │    │ Any MCP     │
    └─────────────┘    └─────────────┘    └─────────────┘
```

## Org Structure

```
github.com/crossmem/
  ├── core         Extension + Web + MCP servers (private, monorepo)
  ├── ios          iOS app — Share Sheet + topic feed (private)
  ├── docs         User-facing docs + SEO (public, GitHub Pages)
  └── specs        ← YOU ARE HERE. Design specs for all repos (private)
```

## Repos

### core (current monorepo)
- **extension/**: Chrome Extension (Manifest V3, vanilla JS, no build step)
  - Auto-save AI conversations (Claude.ai, ChatGPT, Gemini)
  - Save This Page (any webpage, one click)
  - Platform-specific DOM extractors
- **web/**: Next.js app (crossmem.dev)
  - Landing page + search + point cloud visualization
  - Deployed on Vercel (Root Directory: `web/`)
- **mcp-server/**: Unified MCP server (consolidating 4 → 1)
  - conversations, topics, artifacts, files, sheets
- **sheets-mcp/**, **files-mcp/**, **chrome-webstore-mcp/**: Legacy, merging into mcp-server

### ios
- Swift/SwiftUI, iOS 17+
- Share Sheet extension (primary input)
- Topic feed (Slack-like, grouped by topic)
- Topic detail (sources + AI summary)
- Google Drive API via GoogleSignIn SDK

### docs
- Public GitHub Pages site
- User docs, quickstart, MCP setup guide
- Privacy policy, terms
- SEO-optimized for "AI memory", "cross-platform AI"

### specs (this repo)
- Architecture (this file)
- Drive schema
- MCP protocol
- Feature specs per component
- Shared by all repos' AI agents via CLAUDE.md pointer

## Principles

1. **Your data, your Drive**: Zero cloud dependency. Everything in user's Google Drive.
2. **Agent-first**: Users interact through their AI agents, not dashboards.
3. **Any input, any output**: Capture from anywhere, use from any AI platform.
4. **MCP-native**: MCP is the API. No proprietary SDK.
5. **Sidecar metadata**: Content as markdown, metadata as `.crossmem-meta.json`. Human-readable, git-diffable.
