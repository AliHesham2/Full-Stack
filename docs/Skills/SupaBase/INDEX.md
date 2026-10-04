# Supabase Knowledge System

> Source review: 2026-09-27. This is a template reference, not a running project.
> Historical execution/measurement statements below are author reports without a supplied
> reproduction harness; they are not acceptance evidence. This review ran no Supabase
> CLI, SQL, deployed function, or real provider integration.

Evidence-based reference for AI agents building on Supabase. Research conducted August 2026.

## File Map

| File | Scope | When to read |
|------|-------|-------------|
| `architecture.md` | Components, request flow, services | Understanding the full stack |
| `security-keys.md` | API keys (legacy vs new), RLS fundamentals, key hygiene | **Read first for any new project** |
| `rls-patterns.md` | Production RLS policies, multi-tenant, testing, performance | Securing table access |
| `database.md` | Schema design, PostgREST, connections, pooling, indexes, JSONB | Data modeling and queries |
| `auth.md` | Providers, JWT, sessions, the Express (server-side) setup | User authentication |
| `realtime.md` | Postgres Changes, Broadcast, Presence, limits | Live data features |
| `storage.md` | Buckets, policies, image transforms, CDN | File management |
| `edge-functions.md` | Deno runtime, patterns, background tasks, caveats | Server-side logic |
| `vector-ai.md` | pgvector, embeddings, semantic search | AI features |
| `cli-migrations.md` | Local dev, migrations, branching, CI/CD | Developer workflow |
| `client-libs.md` | supabase-js v2, TypeScript, the two Express clients | Client integration |
| `gotchas.md` | Common mistakes, pitfalls, what to avoid | **Read before deploying** |
| `pricing.md` | Plans, limits, real-world cost expectations | Planning and budgeting |

## Cross-references

- Security: `security-keys.md` and `rls-patterns.md` are companion files. Keys = identity; RLS = authorization.
- Database design decisions in `database.md` affect RLS policy complexity in `rls-patterns.md`.
- `auth.md` JWT claims are consumed by RLS policies via `auth.uid()` and `auth.jwt()`.
- `cli-migrations.md` covers the workflow to deploy everything in `database.md` and `rls-patterns.md`.
- `gotchas.md` is the negative-space complement to all other files.

## Freshness and evidence

Source-checked on **2026-09-27**: SDK **2.117.2** (Node >=22), stable CLI **2.118.0**.
These are recommended pins for future implementation, not installed or tested here.

The earlier authors reported executions on CLI 2.116.0, SDK 2.113.0, PostgreSQL
17/18 and pgvector 0.8.1. Their harness/results are absent. SQL correctness,
RLS isolation, auth rotation, uploads, Realtime and deployment still require
verification in a disposable environment when an actual project exists.

Read **SOURCE-VERIFIED** as documentation checked against primary sources.
Read **REQUIRES-RUNTIME-VERIFICATION** as an implementation check still pending.
Do not report production readiness or a numerical product-quality guarantee from this kit.

Architecture remains Browser -> Next.js -> Express on Node.js -> Supabase.
Edge Functions are optional provider adjuncts requiring an explicit architecture decision,
not a second application backend.

Sources: https://registry.npmjs.org/@supabase/supabase-js/latest
and https://github.com/supabase/cli/releases/tag/v2.118.0
and https://supabase.com/docs

