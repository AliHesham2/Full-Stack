# CLI & Migrations

> Project contract: Browser -> Next.js -> Express on Node.js -> Supabase.
> SDKs, credentials, authorization and provider integration live in Express only.

> Source review: 2026-09-27. This is a template reference, not a running project.
> Historical execution/measurement statements below are author reports without a supplied
> reproduction harness; they are not acceptance evidence. This review ran no Supabase
> CLI, SQL, deployed function, or real provider integration.

## Setup

```bash
# Install
brew install supabase/tap/supabase  # macOS
# Or: https://supabase.com/docs/guides/cli

# Initialize in existing project
supabase init

# Link to remote project
supabase link --project-ref your-project-id

# Start local stack (Postgres, Auth, Storage, etc.)
supabase start
```

`supabase start` spins up the full Supabase stack locally via Docker: Postgres, GoTrue,
Realtime, Storage, Studio and Edge Functions.

Historical defaults reported for CLI **2.116.0**; not rerun here. Current stable pin:
**2.118.0**, source-checked 2026-09-27. Recheck generated config when implementing.
Source: https://github.com/supabase/cli/releases/tag/v2.118.0

| Setting | Default |
|---|---|
| Postgres major version | **17** (`[db] major_version = 17`) |
| API (Kong) port | 54321 |
| Postgres port | 54322 |
| Studio port | 54323 |
| Shadow DB port | 54320 |
| Local pooler | `[db.pooler] enabled = false` — opt in, modes: transaction or session |

Pin the CLI version in CI (`supabase/setup-cli@v1` with an explicit `version:`, not
`latest`) — `config.toml` keys move between CLI minors, and a floating version turns an
unrelated CI run into a config error.

Source: https://supabase.com/docs/guides/local-development

## Migration Workflow

### Starting from scratch

```bash
# After schema changes in local DB:
supabase db diff --schema public > supabase/migrations/20260825_create_tables.sql

# Apply to remote
supabase db push --linked
```

### Starting from an existing project

```bash
# Pull remote schema as a migration
supabase db pull
```

### Daily workflow

```
1. supabase start
2. supabase migration new <name>   # or edit SQL, or use Studio locally
3. supabase db diff -f <name>      # generate migration from local changes
4. supabase db reset               # verify full chain applies cleanly
5. supabase db push --linked                # apply pending migrations to remote
```

Source: https://supabase.com/docs/guides/deployment/database-migrations

## Migration Conflicts

After a Git merge, if two migrations have similar timestamps:

1. Rename only migrations never applied in any shared environment. Applied migration
   history is immutable; resolve it with a new forward corrective migration.
2. Run `supabase db reset` locally to verify the full migration chain applies cleanly.
3. `supabase db push --linked` to remote.

Source: https://supabase.com/blog/the-vibe-coders-guide-to-supabase-environments (Aug 2025)

## Multi-Environment Setup

Create separate Supabase projects for each environment:

```
main branch         → production Supabase project
develop branch      → staging Supabase project
feat/* branches     → preview branches (only with configured branching integration)
```

## CI/CD with GitHub Actions

```yaml
name: Deploy Migrations to Production
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    env:
      SUPABASE_ACCESS_TOKEN: ${{ secrets.SUPABASE_ACCESS_TOKEN }}
      SUPABASE_DB_PASSWORD: ${{ secrets.PRODUCTION_DB_PASSWORD }}
      SUPABASE_PROJECT_ID: ${{ secrets.PRODUCTION_PROJECT_ID }}
    steps:
      - uses: actions/checkout@v4
      - uses: supabase/setup-cli@v1
        with: { version: '2.118.0' }
      - run: supabase link --project-ref $SUPABASE_PROJECT_ID
      - run: supabase db push --linked
```

The `SUPABASE_ACCESS_TOKEN` (personal access token) and DB password secrets are **required** — linking and pushing fail without them. Use separate workflow files for staging/production projects.

Source: https://supabase.com/docs/guides/deployment/managing-environments

## How `db push` Tracks State

`db push` compares local `supabase/migrations/` against the remote `supabase_migrations.schema_migrations` table and applies only pending files, in timestamp order. Useful companions:

- `supabase db push --linked --dry-run` — preview what would run
- `supabase migration list --linked` — compare local vs remote history
- `supabase migration repair <version> --status applied|reverted` — edits history only; does not apply or undo SQL. Reconcile actual schema first.

**Golden rule**: never modify the remote schema directly (Dashboard SQL/Table Editor) once you use migrations — it bypasses migration history and breaks future pushes. If it happens, capture with `supabase db pull`.

Source: https://supabase.com/docs/guides/local-development/cli-workflows

## Local vs Remote Credentials

`supabase status` reports the local stack and its keys; available key formats depend on CLI version. Keep local credentials in Express, not Next.js.

**Target verification**: `supabase status` does not verify the linked remote project.
Use `db push --local` for local work. Before remote work explicitly verify the linked
project reference and intended environment, inspect `migration list --linked`, and run
`db push --linked --dry-run`. These are future workflow examples, not commands run here.

Source: https://github.com/supabase/cli/issues/3493