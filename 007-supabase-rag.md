# Spec 007: Supabase RAG Layer

> Acceleration layer for paid users. Drive remains source of truth.

## Principle

```
Free:  Google Drive only. No server-side data. No RAG.
Paid:  Drive (lossless) + Supabase (lossy copy for search + RAG).
```

User-facing framing: "Your data lives in your Google Drive. We build a search index to make it useful — like Google indexes the web, but just for you."

## Data Flow

### Free User

```
iOS / Extension → Google Drive
                  (done)
```

### Paid User

```
iOS / Extension → Google Drive       (lossless, always)
               → POST /api/ingest   (lossy copy to Supabase)
                   ├─ insert source record
                   ├─ chunk text (800 chars, 100 overlap)
                   ├─ embed chunks (text-embedding-3-small, 1536d)
                   └─ insert chunks + embeddings
```

The iOS app and extension handle the dual-write: save to Drive first (existing flow), then call our API if user is on paid plan. No Drive-to-Supabase sync needed — avoids `drive.readonly` scope and CASA audit requirement.

## Schema

### `sources` — What the user saved

```sql
CREATE TABLE sources (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES auth.users NOT NULL,
  drive_file_id TEXT,
  source_url TEXT,
  domain TEXT,                    -- "twitter.com", "reddit.com"
  title TEXT,
  content TEXT,                   -- extracted full text (lossy copy)
  content_type TEXT NOT NULL,     -- 'link' | 'conversation' | 'screenshot' | 'note'
  platform TEXT,                  -- 'twitter' | 'reddit' | 'claude.ai' | 'chatgpt.com'
  topic_name TEXT,                -- matches Drive folder name
  saved_from TEXT,                -- 'ios' | 'extension' | 'web'
  metadata JSONB DEFAULT '{}',    -- flexible: author, likes, thread_id, etc.
  hash TEXT,                      -- SHA-256 for dedup (same as Drive sidecar)
  created_at TIMESTAMPTZ DEFAULT now()
);

CREATE INDEX idx_sources_user ON sources (user_id);
CREATE INDEX idx_sources_topic ON sources (user_id, topic_name);
CREATE UNIQUE INDEX idx_sources_dedup ON sources (user_id, hash);
```

### `chunks` — RAG retrieval units

```sql
CREATE EXTENSION IF NOT EXISTS vector WITH SCHEMA extensions;

CREATE TABLE chunks (
  id BIGINT PRIMARY KEY GENERATED ALWAYS AS IDENTITY,
  source_id UUID REFERENCES sources ON DELETE CASCADE NOT NULL,
  user_id UUID REFERENCES auth.users NOT NULL,
  chunk_text TEXT NOT NULL,
  chunk_index INT NOT NULL,
  embedding extensions.vector(1536),
  created_at TIMESTAMPTZ DEFAULT now()
);

CREATE INDEX idx_chunks_user ON chunks (user_id);
CREATE INDEX idx_chunks_embedding ON chunks
  USING hnsw (embedding vector_cosine_ops)
  WITH (m = 16, ef_construction = 64);
```

### `topics` — Mirrors Drive topic folders

```sql
CREATE TABLE topics (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES auth.users NOT NULL,
  name TEXT NOT NULL,              -- "rag-architecture" (slug, matches Drive)
  display_name TEXT,               -- "RAG Architecture"
  summary TEXT,                    -- AI-generated, updated on new source
  source_count INT DEFAULT 0,
  status TEXT DEFAULT 'active',    -- 'active' | 'archived'
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now(),
  UNIQUE(user_id, name)
);
```

### RLS Policies

```sql
ALTER TABLE sources ENABLE ROW LEVEL SECURITY;
ALTER TABLE chunks ENABLE ROW LEVEL SECURITY;
ALTER TABLE topics ENABLE ROW LEVEL SECURITY;

CREATE POLICY "users own data" ON sources
  FOR ALL USING (auth.uid() = user_id);
CREATE POLICY "users own data" ON chunks
  FOR ALL USING (auth.uid() = user_id);
CREATE POLICY "users own data" ON topics
  FOR ALL USING (auth.uid() = user_id);
```

### Search Function

```sql
CREATE OR REPLACE FUNCTION match_chunks(
  query_embedding extensions.vector(1536),
  query_user_id UUID,
  match_threshold FLOAT DEFAULT 0.5,
  match_count INT DEFAULT 8
)
RETURNS TABLE (
  chunk_text TEXT,
  source_id UUID,
  title TEXT,
  source_url TEXT,
  platform TEXT,
  topic_name TEXT,
  similarity FLOAT
)
LANGUAGE sql STABLE
AS $$
  SELECT
    c.chunk_text,
    s.id AS source_id,
    s.title,
    s.source_url,
    s.platform,
    s.topic_name,
    1 - (c.embedding <=> query_embedding) AS similarity
  FROM chunks c
  JOIN sources s ON c.source_id = s.id
  WHERE c.user_id = query_user_id
    AND 1 - (c.embedding <=> query_embedding) > match_threshold
  ORDER BY c.embedding <=> query_embedding
  LIMIT match_count;
$$;
```

## API Routes

### `POST /api/ingest`

Called by iOS app / extension after saving to Drive.

```
Request:
{
  source_url: "https://twitter.com/karpathy/status/123",
  domain: "twitter.com",
  title: "Why RAG needs graph",
  content: "Thread: 1/ RAG is great but...",
  content_type: "link",
  platform: "twitter",
  topic_name: "rag-architecture",
  saved_from: "ios",
  drive_file_id: "1xYz...",
  metadata: { author: "karpathy", likes: 4200 }
}

Response:
{ source_id: "abc-123", chunks_count: 3 }
```

### `POST /api/search`

Semantic search across user's knowledge base.

```
Request:  { query: "graph RAG" }
Response: { results: [{ chunk_text, title, source_url, similarity }] }
```

### `POST /api/chat`

RAG-powered conversation with user's knowledge base.

```
Request:  { messages: [{ role: "user", content: "What did I save about RAG?" }] }
Response: Streamed LLM response grounded in user's sources.
```

## Chunking Strategy

```
Input text
  → Split by paragraph (\n\n)
  → Merge small paragraphs up to 800 chars
  → Overlap: 100 chars between chunks
  → Short content (<800 chars): single chunk, no split
```

Tweets and short posts become 1 chunk. Long threads and conversations become multiple.

## Account Deletion

User deletes account or downgrades to free:

```sql
-- CASCADE handles chunks via source_id FK
DELETE FROM sources WHERE user_id = $1;
DELETE FROM topics WHERE user_id = $1;
-- Drive data is untouched — user still has everything
```

## Cost Estimates

### Per-User Assumptions

| Metric | Conservative | Active User |
|--------|-------------|-------------|
| Sources saved / month | 30 | 100 |
| Avg content size | 1,500 chars | 2,000 chars |
| Chunks per source | 2 | 3 |
| Chunks / month | 60 | 300 |
| RAG queries / month | 20 | 100 |

### Embedding Cost (OpenAI text-embedding-3-small)

Price: $0.02 / 1M tokens. ~4 chars per token.

```
Ingestion (per user/month):
  Active user: 100 sources x 2,000 chars = 200K chars = 50K tokens
  Cost: 50K / 1M x $0.02 = $0.001

Search queries (per user/month):
  100 queries x 20 chars avg = 2K chars = 500 tokens
  Cost: negligible

Total embedding cost per active user: ~$0.001/month
  → 1,000 paid users = $1/month
  → 10,000 paid users = $10/month
```

### LLM Cost (RAG chat, Claude Haiku or GPT-4o-mini)

Per RAG query: ~8 chunks x 200 tokens + query + system prompt ≈ 2,500 input tokens, ~500 output tokens.

```
Claude Haiku:  $0.25/1M input + $1.25/1M output
  Per query:   2,500 x $0.00000025 + 500 x $0.00000125 = $0.0013
  Active user (100 queries): $0.13/month

GPT-4o-mini:   $0.15/1M input + $0.60/1M output
  Per query:   2,500 x $0.00000015 + 500 x $0.0000006 = $0.0007
  Active user (100 queries): $0.07/month
```

### Supabase Cost

Free tier: 500MB database, 1GB file storage, 50K monthly active users.

```
Per active user storage:
  sources: ~100 rows x 3KB = 300KB
  chunks:  ~300 rows x 7KB (text + 1536d vector) = 2.1MB
  Monthly growth: ~2.4MB per active user

Supabase free tier (500MB):
  → Fits ~200 active paid users

Supabase Pro ($25/month, 8GB):
  → Fits ~3,300 active paid users

Supabase Pro + $0.125/GB overage:
  → 10,000 users x 2.4MB = 24GB = $25 + (16GB x $0.125) = $27/month
```

### Total Cost Per 1,000 Paid Users

| Component | Monthly Cost |
|-----------|-------------|
| Embedding (OpenAI) | $1 |
| LLM / RAG chat (Haiku) | $130 |
| Supabase Pro | $25 |
| Vercel Pro (if needed) | $20 |
| **Total** | **~$176/month** |

→ **$0.18 per paid user per month** in infrastructure cost.

At $5/month pricing → **96% gross margin**.
At $3/month pricing → **94% gross margin**.

### Scaling Milestones

| Paid Users | Infra Cost | Revenue ($5/mo) | Margin |
|-----------|-----------|-----------------|--------|
| 100 | ~$30 | $500 | 94% |
| 1,000 | ~$176 | $5,000 | 96% |
| 10,000 | ~$1,500 | $50,000 | 97% |

LLM cost dominates. If users chat heavily, consider:
- Rate limiting free RAG queries (e.g., 20/month on base paid plan)
- Caching common queries per user
- Using GPT-4o-mini instead of Haiku ($0.07 vs $0.13 per user)

## What This Spec Does NOT Cover

- Auto-clustering (see 006-auto-clustering.md) — will use topic centroids in Supabase later
- Point cloud visualization — reads from topics table, separate spec
- Topic summary generation — triggered on source_count change, separate spec
- Billing / subscription management — Stripe integration, separate spec
