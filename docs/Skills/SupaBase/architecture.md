# Supabase Architecture

> Source review: 2026-09-27. This is a template reference, not a running project.
> Historical execution/measurement statements below are author reports without a supplied
> reproduction harness; they are not acceptance evidence. This review ran no Supabase
> CLI, SQL, deployed function, or real provider integration.

> Project boundary: these are provider capabilities, not frontend permissions. All provider
> SDK calls, DB/Auth/Storage/Realtime integrations run in Express on Node.js for this project.
> Browser → Next.js → Express → Supabase. Provider-specific URLs/keys/tokens stay behind Express.
> The frontend uses same-origin relays; generic direct-client examples are not this project's flow.


## What it is

Supabase is an open-source backend platform built on PostgreSQL. It is not a database abstraction — every project gets a full Postgres database. The platform layers services on top: auto-generated REST API, authentication, realtime, file storage, serverless functions, and vector search.

Source: https://supabase.com/docs/guides/getting-started/architecture

## Core Components

| Component | Technology | Role |
|-----------|-----------|------|
| Database | PostgreSQL | Full Postgres, not a proxy or abstraction |
| REST API | PostgREST | Auto-generates RESTful endpoints from schema |
| Auth | GoTrue | User management, JWTs, OAuth providers |
| Realtime | Elixir (Phoenix) | WebSocket server for live data |
| Storage | S3-compatible | File storage with CDN and image transforms |
| Edge Functions | Deno (edge-runtime) | Server-side TypeScript at the edge |
| API Gateway | Envoy (C++) | Routes requests to internal services — confirmed against the official architecture page |
| Connection Pooler | Supavisor | Shared Elixir-based pooler; paid dedicated transaction pooler uses PgBouncer |
| CLI | Go binary | Local dev, migrations, project management |

## Request Flow

```
Client → Envoy Gateway → PostgREST (REST API)
                       → GoTrue (Auth)
                       → Realtime (WebSockets)
                       → Storage API
                       → Edge Functions
                       → Supavisor → PostgreSQL
```

1. Hosted HTTP API requests use the gateway; raw PostgreSQL TCP connections use database or pooler endpoints.
2. Envoy routes to the appropriate service based on the URL path.
3. Raw database clients choose direct access, shared Supavisor or the paid dedicated PgBouncer pooler; the HTTP diagram is not a TCP connection diagram.
4. Realtime subscribes to Postgres WAL (via logical replication) to detect changes.

## Key Design Implications

- **Schema = API**: PostgREST generates your REST API directly from your database schema. Tables exposed in the `public` schema (or other exposed schemas) automatically get CRUD endpoints. Rename a column → the API changes.

- **RLS is the backstop, Express is the authorization layer**: Supabase's own model assumes
  browsers query it directly, with RLS as the only guard. In this project no browser touches
  Supabase: Express authorizes every request in its use cases (backend-guide/03-auth/authorization.md),
  and RLS stays on every table as the second line, enforced for the per-request USER client. See
  [security-keys.md](./security-keys.md) and [rls-patterns.md](./rls-patterns.md).

- **Connection endpoints vary**: shared Supavisor offers session 5432 and transaction
  6543; paid dedicated PgBouncer offers transaction pooling on `db.<ref>.supabase.co:6543`.
  Direct access is `db.<ref>.supabase.co:5432`. Copy the Connect-panel URL rather than
  guessing the host. Direct/dedicated access normally needs IPv6 or the IPv4 add-on.
  See [database.md](./database.md).

- **Edge Functions are Deno, but npm and Node built-ins work**: they run on Supabase's
  `edge-runtime` (a Deno fork), so the module system is ESM and the globals are Web APIs —
  but `npm:`, `node:` and `jsr:` specifiers are all supported. "No Node.js compatibility"
  is outdated advice; the CLI's own generated function imports `npm:@supabase/server`.
  What you don't get is a Node *runtime*: no `require`, no `process.cwd()`-style
  filesystem assumptions, no native addons. See [edge-functions.md](./edge-functions.md).

## Self-hosting

All components are open-source and can be self-hosted via Docker Compose. This is non-trivial to operate at scale. Supavisor, connection pooling configuration, and Realtime scaling are the hardest parts. Most teams should use the managed platform unless they have specific compliance or cost requirements.

Source: https://github.com/supabase/supabase

## When to use Supabase

Good fit: SaaS apps, AI products with vector needs, internal tools, prototypes, mobile backends, realtime collaborative features.

Poor fit: Heavy OLAP workloads (use a data warehouse), projects needing non-Postgres databases, extremely high-write throughput requiring custom sharding.
