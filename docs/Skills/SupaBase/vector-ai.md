# Vector & AI

> Source review: 2026-09-27. This is a template reference, not a running project.
> Historical execution/measurement statements below are author reports without a supplied
> reproduction harness; they are not acceptance evidence. This review ran no Supabase
> CLI, SQL, deployed function, or real provider integration.

> Project boundary: these are provider capabilities, not frontend permissions. All provider
> SDK calls, DB/Auth/Storage/Realtime integrations run in Express on Node.js for this project.
> Browser → Next.js → Express → Supabase. Provider-specific URLs/keys/tokens stay behind Express.
> The frontend uses same-origin relays; generic direct-client examples are not this project's flow.


## pgvector

Supabase includes the `pgvector` extension, pre-installed and ready to enable.

```sql
CREATE EXTENSION IF NOT EXISTS vector;
```

Stores vector embeddings alongside your relational data. No separate vector database needed for most use cases.

Source: https://supabase.com/docs/guides/ai/semantic-search

## Basic Setup

Abbreviated DDL, not an exposed production schema: add owner/tenant fields, restricted
grants and RLS policies before exposing documents. Express must use the USER client
for user-owned embeddings and authorize inserts and searches.

```sql
-- Create a table with a vector column
CREATE TABLE documents (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  content TEXT NOT NULL,
  embedding VECTOR(1536)  -- match your model exactly; see the table below
);

-- Index for similarity search
CREATE INDEX ON documents
  USING HNSW (embedding vector_cosine_ops)
  WITH (m = 16, ef_construction = 64);
```

**The dimension is not negotiable** — `vector(n)` rejects any embedding of a different
length, and changing it later means an `alter table` plus re-embedding your entire corpus.
Pick the model first.

| Model | Dimensions |
|---|---|
| OpenAI `text-embedding-3-small` | 1536 (reducible via the `dimensions` param) |
| OpenAI `text-embedding-3-large` | 3072 (reducible) |
| OpenAI `text-embedding-ada-002` | 1536 — **legacy**, superseded by 3-small |
| `Supabase/gte-small` (built-in, see below) | 384 |

**Storage limits and index limits are different numbers** (from the pgvector README):

| Type | Max stored | Max **indexable** (hnsw / ivfflat) |
|---|---|---|
| `vector` | 16,000 | **2,000** |
| `halfvec` | 16,000 | **4,000** |
| `bit` | 64,000 | 64,000 |
| `sparsevec` | 16,000 non-zero | 1,000 non-zero (HNSW only; no IVFFlat sparse index) |

Executed on pgvector **0.8.1** — the limits are enforced at `create index`, with these exact errors:

```
vector(2000)  + hnsw → index created
vector(2001)  + hnsw → ERROR: column cannot have more than 2000 dimensions for hnsw index
vector(3072)  + hnsw → ERROR: column cannot have more than 2000 dimensions for hnsw index
halfvec(3072) + hnsw → index created
halfvec(4001) + hnsw → ERROR: column cannot have more than 4000 dimensions for hnsw index

create table t(e vector(3072)); insert into t values (…);   → succeeds
```

That last line is the trap: **the dimension is valid for storage**. Other constraints, grants, RLS and resource limits can still reject inserts. The column accepts 3072 dimensions, the
inserts work, the queries return correct results — and every one of them is a sequential
scan, because `create index` was the step that errored and someone dismissed it.

> A `vector(3072)` column from `text-embedding-3-large` **cannot use a plain vector HNSW/IVFFlat index** — every similarity query silently becomes a sequential scan, which reads as
> "pgvector is slow" rather than "there is no index". Either request fewer dimensions from
> the model (`dimensions: 1536`), or store `halfvec(3072)`, which indexes up to 4,000. A halfvec expression index on
> a vector column is another option; the query must use the matching cast/operator.

## Inserting Embeddings

```typescript
// Generate embedding (e.g., via OpenAI)
const embedding = await generateEmbedding(text)

await supabase.from('documents').insert({
  content: text,
  embedding: embedding
})
```

## Generating embeddings without an external API

Supabase Edge Functions offer built-in inference without an OpenAI key. Express still
makes a network call to the provider function, and function invocation/compute charges
apply; this is not a cost-free local model inside Express:

```ts
const session = new Supabase.ai.Session('gte-small');
const embedding = await session.run(text, { mean_pool: true, normalize: true });
// → 384-dimensional array, matching vector(384)
```

The documented built-in `gte-small` produces 384 dimensions, is **English-only**, and
truncates inputs to 512 tokens. It is not a default for Arabic or multilingual content.
Select and benchmark a suitable model before fixing the schema. Do not infer general
model availability or retrieval quality from worker memory limits or unsourced scores.

Source: https://supabase.com/docs/guides/functions/ai-models

## Similarity Search

```sql
-- Cosine similarity (most common)
SELECT id, content, 1 - (embedding <=> $1) AS similarity
FROM documents
ORDER BY embedding <=> $1
LIMIT 5;
```

```typescript
// Express only. Define match_documents in a migration first; the SDK does not create it.
const { data } = await supabase.rpc('match_documents', {
  query_embedding: embedding,
  match_threshold: 0.7,
  match_count: 5
})
```

The RPC signature, authorization and bounded result count must be defined explicitly.
Default to SECURITY INVOKER, pin search_path, restrict execution grants, and ensure
tenant/ownership policies apply. The 0.7 threshold is an example requiring calibration.

## Verified against pgvector 0.8.1

The DDL and query above were executed, not transcribed:

```
create extension vector                                             → OK (0.8.1)
create index … using hnsw (embedding vector_cosine_ops)
                       with (m = 16, ef_construction = 64)          → OK
create index … using ivfflat (embedding vector_cosine_ops)
                       with (lists = 1)                             → OK
select 1 - (embedding <=> '[1,0,0]') … order by embedding <=> …     → exact=1.000
                                                                       near=0.994
                                                                       orthogonal=0.000
explain: Limit → Index Scan using docs_ivf on documents
                 Order By: (embedding <=> '[1,0,0]')                → index IS used
```

Note the query shape: you must `order by embedding <=> query` for the index to be used.
Ordering by the computed `similarity` alias instead (`order by similarity desc`) is the
common mistake — it is algebraically equivalent and the planner will **not** use the index.

## Index Types

| Index | Use case | Notes |
|-------|----------|-------|
| HNSW | Low-latency, approximate | Measure recall, memory and build cost; tune parameters. |
| IVFFlat | Approximate, lower build cost | Train with representative data; rebuild when distribution changes. VACUUM does not retrain it. |
| Exact (no index) | Small datasets | Measure latency; 10K rows is a heuristic, not a universal cutoff. |

## When pgvector Is Enough

For most projects: semantic search, RAG, recommendation, classification. Choose based on measured workload, recall, cost and operational needs.

When to consider a dedicated vector DB: billions of vectors, sub-millisecond latency at massive scale, advanced filtering requirements.

Source: https://github.com/pgvector/pgvector
