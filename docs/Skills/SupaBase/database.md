# Database

> Project contract: Browser -> Next.js -> Express on Node.js -> Supabase.
> SDKs, credentials, authorization and provider integration live in Express only.

> Source review: 2026-09-27. This is a template reference, not a running project.
> Historical execution/measurement statements below are author reports without a supplied
> reproduction harness; they are not acceptance evidence. This review ran no Supabase
> CLI, SQL, deployed function, or real provider integration.

## Schema = API

PostgREST auto-generates REST endpoints from your schema. This is a core design principle, not a convenience feature.

- **Tables in exposed schemas** (default: `public`) become API endpoints.
- **Column names** become query parameters.
- **Relationships** (foreign keys) enable nested resource fetching.
- **Rename a column** → the API field name changes. No versioning.

Use migrations for all schema changes. See [cli-migrations.md](./cli-migrations.md).

Source: https://supabase.com/docs/guides/api

## Connections & Pooling

### Direct, shared Supavisor and dedicated PgBouncer

Supavisor is the shared connection pooler. Paid projects also offer a dedicated PgBouncer transaction pooler.

**The host matters as much as the port** — the same port 5432 means two different things
depending on which hostname you use:

| Connection | Host : Port | Network | Prepared statements | Use for |
|---|---|---|---|---|
| **Direct** | `db.<ref>.supabase.co:5432` | **IPv6** by default (IPv4 is a paid add-on) | Yes | Migrations, `pg_dump`, long-lived VMs/containers, ORMs on a persistent backend |
| **Supavisor — session** | `aws-<index>-<region>.pooler.supabase.com:5432` | IPv4 | Yes | Persistent backends stuck on IPv4-only networks |
| **Dedicated PgBouncer — transaction** | `db.<ref>.supabase.co:6543` | IPv6 by default / IPv4 add-on | **No** | Paid projects, transaction pooling |
| **Supavisor — transaction** | `aws-<index>-<region>.pooler.supabase.com:6543` | IPv4 | **No** | Serverless / edge / short-lived clients |

**The IPv6 trap**: the direct connection resolves to IPv6 only. Most CI runners, many
corporate networks, and several serverless platforms are IPv4-only, so a connection string
that works locally times out there with no useful error. That is what Supavisor's session
mode can solve. Session and transaction modes also differ in connection/session semantics.
Copy the actual Connect-panel hostname and username; shared pooling typically uses
`postgres.<ref>`, not just `postgres`.

**Transaction mode has no prepared statements**, so ORM/driver settings must match the selected mode.
Either use a mode that supports them, or disable them in the driver
(`?pgbouncer=true` for Prisma, `prepare: false` for postgres.js).

**DEPRECATED**: session mode on port 6543 was removed on 2025-02-28. Port 6543 is
transaction-only.

Source: https://supabase.com/docs/guides/database/connecting-to-postgres, https://supabase.com/changelog/32755

### When to use which

- **supabase-js client**: uses the REST API (PostgREST) — no raw Postgres connection from Express.
  PostgREST still connects to Postgres; raw-driver pool settings do not configure the SDK.
- **Prisma/Drizzle on a persistent server**: direct connection (IPv6, or session mode if
  you are IPv4-only).
- **Prisma/Drizzle on serverless**: transaction mode + disable prepared statements.
- **pgAdmin/psql/migrations/CLI**: direct connection.
- **Serverless functions**: prefer the REST API; otherwise transaction mode.

Source: https://supabase.com/docs/guides/database/connecting-to-postgres

## Schema Design Rules

### Rule: Use real columns, not JSONB for everything

JSONB is good for optional, sparse, or evolving metadata. Bad as a replacement for stable relational fields.

**Trade-offs**: stable relational fields benefit from explicit types, foreign keys and
ordinary indexes. JSONB supports CHECK constraints, expression indexes, generated
columns and indexed joins on extracted values; it is not inherently unconstrained or
unjoinable. Choose GIN/B-tree/expression indexes for actual operators and query plans.

**When JSONB is appropriate**: evolving metadata, webhook payloads and sparse preferences.

Source: https://www.postgresql.org/docs/current/datatype-json.html

### Rule: Index before you need them

PostgREST queries are just Postgres queries. Every `.eq()`, `.order()`, `.ilike()` becomes a WHERE/ORDER BY clause.

```sql
-- For queries like: supabase.from('posts').eq('user_id', id).order('created_at')
CREATE INDEX idx_posts_user_created ON posts(user_id, created_at DESC);
```

Use `pg_stat_statements` (enabled by default on Supabase) and the Database Advisor to find missing indexes.

### Rule: Use generated columns or views for computed data

Don't store derived data in client code. Use Postgres computed columns or views.

```sql
ALTER TABLE posts ADD COLUMN search_vector tsvector
  GENERATED ALWAYS AS (
    setweight(to_tsvector('english', coalesce(title,'')), 'A') ||
    setweight(to_tsvector('english', coalesce(body,'')), 'B')
  ) STORED;
```

## Useful Extensions

Some extensions are pre-enabled; others you enable per project. Key ones:

| Extension | Purpose |
|-----------|---------|
| `uuid-ossp` | UUID generation |
| `pgvector` | Vector embeddings and similarity search → [vector-ai.md](./vector-ai.md) |
| `pg_cron` | Scheduled jobs |
| `pg_stat_statements` | Query performance tracking (enabled by default) |
| `postgis` | Geospatial queries (enable explicitly) |
| `pg_net` | Async HTTP requests from SQL (powers Database Webhooks) |
| `pg_graphql` | Auto-generated GraphQL API |

Avoid the legacy `pgjwt` extension — use `pgcrypto` and Supabase's built-in auth functions instead.

Enable extensions via Dashboard or SQL:
```sql
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
```

## Database Webhooks

Postgres-level webhooks that fire on INSERT/UPDATE/DELETE. Built on `pg_net`.

- Useful for sending data to external services (Slack, Stripe, etc.)
- Can be combined with `pg_cron` for scheduled webhook retries
- Supabase also supports Edge Functions as webhook targets

Source: https://supabase.com/docs/guides/database/database-webhooks

## Triggers & Functions

For business logic that must run on every write (audit logs, denormalization, notifications):

```sql
CREATE OR REPLACE FUNCTION update_updated_at()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = now();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER set_updated_at
  BEFORE UPDATE ON your_table
  FOR EACH ROW EXECUTE FUNCTION update_updated_at();
```

**Rule**: Express owns business logic and authorization. Database constraints and narrowly scoped functions enforce invariants and atomic transactions; frontend checks are not enforcement. See [gotchas.md](./gotchas.md).
