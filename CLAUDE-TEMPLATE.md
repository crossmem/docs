# CLAUDE.md Template for CrossMem Repos

> Copy this block into the top of each repo's CLAUDE.md.

```markdown
## CrossMem Specs (shared across all repos)

This repo is part of the CrossMem org. Before making architectural decisions, read the shared specs:

- **Architecture**: https://github.com/crossmem/specs/blob/main/ARCHITECTURE.md
- **Drive Schema**: https://github.com/crossmem/specs/blob/main/002-drive-schema.md
- **MCP Protocol**: https://github.com/crossmem/specs/blob/main/003-mcp-consolidation.md
- **iOS App**: https://github.com/crossmem/specs/blob/main/004-ios-app.md
- **Web App**: https://github.com/crossmem/specs/blob/main/005-web-app.md

Until the org is created, specs live at: /home/ydwu/claude_projects/crossmem/specs/

Key principles:
1. User data stays in user's Google Drive (never our servers)
2. Agent-first design (MCP is the primary API)
3. Sidecar metadata (`.crossmem-meta.json` / `_topic.json`)
4. File naming: `{date}_{source-type}_{slugified-title}.md`
```
