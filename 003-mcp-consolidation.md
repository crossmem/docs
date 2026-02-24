# Spec 003: MCP Server Consolidation

> Merge 4 MCP servers into 1 unified `crossmem-mcp`.

## Current State (4 servers)

```
mcp-server/          → conversations: search, get, list
sheets-mcp/          → sheets: read, write, search, create
files-mcp/           → files: upload, download, list
chrome-webstore-mcp/ → CWS: (niche, rarely used)
```

User's MCP config requires 4 separate entries. Painful to set up.

## Target State (1 server)

```
crossmem-mcp/
  ├── index.ts              ← entry point, tool router
  ├── tools/
  │   ├── conversations.ts  ← search, get, list, save
  │   ├── topics.ts         ← list, get, create, add_source, search
  │   ├── artifacts.ts      ← list, get, search
  │   ├── files.ts          ← upload, download, list
  │   └── sheets.ts         ← read, write, search, create
  ├── lib/
  │   ├── drive.ts          ← shared Google Drive client
  │   └── auth.ts           ← shared OAuth
  └── package.json
```

## Tool Catalog

### conversations
| Tool | Description |
|------|-------------|
| `search_conversations` | Full-text search across all saved conversations |
| `get_conversation` | Get full content by file ID |
| `list_conversations` | List recent conversations, filter by platform/date |
| `save_conversation` | Save a conversation to Drive (used by extension) |

### topics
| Tool | Description |
|------|-------------|
| `list_topics` | List all topics with metadata |
| `get_topic` | Get topic details + list of sources |
| `create_topic` | Create new topic folder + `_topic.json` |
| `add_to_topic` | Add a source (URL, text, file) to a topic |
| `search_topics` | Search across all topics' content |
| `update_topic` | Update topic metadata (tags, status, summary) |

### artifacts
| Tool | Description |
|------|-------------|
| `list_artifacts` | List extracted artifacts |
| `get_artifact` | Get artifact content by ID |
| `search_artifacts` | Search artifacts by type/content |

### files
| Tool | Description |
|------|-------------|
| `upload_file` | Upload any file to Drive |
| `download_file` | Download file from Drive to local |
| `list_files` | List files in crossmem folders |

### sheets
| Tool | Description |
|------|-------------|
| `read_sheet` | Read data from Google Sheet |
| `write_sheet` | Write/append to Google Sheet |
| `search_sheet` | Search across sheets |
| `create_sheet` | Create new spreadsheet |

## Migration Plan

1. Create `crossmem-mcp/` directory in core monorepo
2. Move shared Drive/auth logic to `lib/`
3. Port each tool set with same function signatures
4. Test: existing MCP calls should produce identical results
5. Update user setup docs (one config entry instead of four)
6. Deprecate old directories (keep for 1 release, then delete)

## User Config (after)

```json
{
  "mcpServers": {
    "crossmem": {
      "command": "node",
      "args": ["/path/to/crossmem/core/crossmem-mcp/index.js"],
      "env": {
        "GOOGLE_CREDENTIALS_PATH": "/path/to/credentials.json"
      }
    }
  }
}
```

One entry. Done.
