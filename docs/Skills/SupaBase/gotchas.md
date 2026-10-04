# Gotchas & Pitfalls

> Source review: 2026-09-27. This is a template reference, not a running project.
> Historical execution/measurement statements below are author reports without a supplied
> reproduction harness; they are not acceptance evidence. This review ran no Supabase
> CLI, SQL, deployed function, or real provider integration.

Research-sourced. Each entry has evidence.

## 1. Forgetting to Enable RLS

**Impact**: CRITICAL — data exposure.
**Symptom**: Tables are readable/writable by anyone with the anon key.
**Fix**: Enable RLS on every table. Revoke default grants. Test with a real user JWT.
**Source**: https://supabase.com/blog/the-vibe-coders-guide-to-supabase-environments (Aug 2025)

## 2. Deploying to the Wrong Environment

**Impact**: HIGH — schema changes applied to production instead of staging.
**Symptom**: Migration runs against wrong project.
**Fix**: Use different terminal themes for production. Verify the linked project reference and environment, then use `supabase migration list --linked` and `supabase db push --linked --dry-run`. `status` describes only the local stack. Use CI/CD instead of manual deploys. Never store production credentials locally.
**Source**: https://supabase.com/blog/the-vibe-coders-guide-to-supabase-environments (Aug 2025)

## 3. Testing RLS with service_role

**Impact**: HIGH — false confidence in security.
**Symptom**: "Everything works" in testing but data leaks in production.
**Why**: `service_role` bypasses RLS entirely. You never exercise the policies.
**Fix**: Test with a real authenticated user session. For automated testing use pgTAP with both a restricted role and transaction-local claims in a disposable fixture. Setting claims while remaining postgres does not test RLS.

## 4. `auth.uid() IS NOT NULL` as a Policy

**Impact**: HIGH — any authenticated user sees all rows.
**Why**: This checks "is logged in" not "owns this row". It is an identity check, not authorization.
**Fix**: Use `user_id = auth.uid()` or membership-based policies. See [rls-patterns.md](./rls-patterns.md).

## 5. Treating Postgres Like a Schemaless DB (JSONB Abuse)

**Impact**: MEDIUM — harder validation and query planning when stable relational fields are hidden in JSONB.
**Symptom**: All data in one `metadata` JSONB column. Queries using `->>` operators everywhere.
**Fix**: Use real columns for stable fields. Reserve JSONB for optional/sparse metadata. See [database.md](./database.md).
**Source**: https://www.postgresql.org/docs/current/datatype-json.html

## 6. No Indexes Until the App Gets Slow

**Impact**: MEDIUM — degrading performance as data grows.
**Symptom**: Queries fast with 100 rows, slow with 100K rows.
**Fix**: Design indexes for actual query operators and plans; measure with EXPLAIN and
`pg_stat_statements`. Do not blindly index every filter column.
**Source**: https://www.postgresql.org/docs/current/datatype-json.html

## 7. Overusing Realtime for High-Churn Data

**Impact**: MEDIUM — noisy subscriptions, unnecessary load, missed events.
**Symptom**: Every insert/update triggers a broadcast to all subscribers.
**Fix**: Use Realtime for collaborative features with few writers. For high-churn data, use polling or debounced updates.
**Source**: https://supabase.com/docs/guides/realtime/postgres-changes

## 8. Putting Business Logic Only in the Frontend

**Impact**: MEDIUM — bypassable rules, race conditions, inconsistent state.
**Symptom**: Validation only in React/Vue components, no database constraints.
**Fix**: Express enforces business rules and authorization. Database constraints and
narrowly scoped functions enforce invariants and atomic transactions.
**Source**: https://www.postgresql.org/docs/current/datatype-json.html

## 9. A Supabase Key in the Frontend

**Impact**: CRITICAL — secrets exposed in browser bundles.
**Symptom**: Service role key visible in page source / network tab.
**Fix**: In this project no Supabase key or URL exists in the frontend at all, not even the
publishable key in `NEXT_PUBLIC_*`: every Supabase variable lives in Express's environment.

## 10. `with check (true)` on an INSERT Policy

**Impact**: HIGH — any authenticated user can write rows owned by anyone.
**Why**: INSERT policies have **no `USING` clause**; `WITH CHECK` is the only gate.
Verified on PostgreSQL 18: with `with check (true)` a user successfully inserted a row
carrying another user's `user_id`; with `with check (user_id = (select auth.uid()))` the
same statement was rejected.
**Fix**: every INSERT policy needs a real `WITH CHECK`.

**Correction to common advice**: "always add `WITH CHECK` to UPDATE too" is not required.
For UPDATE, Postgres stores `polwithcheck` as `null` when you omit it and falls back to the
`USING` expression — verified: a USING-only UPDATE policy **rejected** an attempt to
reassign a row to another user. Add an explicit `WITH CHECK` on UPDATE only when it should
differ from `USING`. See [rls-patterns.md](./rls-patterns.md).

## 11. CLI `db push` Target Confusion

**Impact**: MEDIUM — accidentally pushing to remote when intending local.
**Symptom**: Local migration appears on production.
**Fix**: Choose `db push --local` or `db push --linked` explicitly. For linked pushes verify the project reference, environment and dry run; `status` is local-only. Use CI/CD.
**Source**: https://github.com/supabase/cli/issues/3493 (Apr 2025)

## 12. Views and `security definer` RPCs Silently Bypassing RLS

**Impact**: CRITICAL — full table exposure through a path you thought was protected.
**Why**: Postgres creates views as `security definer` owned by `postgres` (which has
`bypassrls`), and a `security definer` function runs with its creator's privileges.
Verified on PostgreSQL 18: the table returned 2 rows to a user, the plain view over it
returned **all 3**.
**Fix**: `create view … with (security_invoker = true)`; pin `set search_path = ''` on every
definer function, fully qualify names, restrict execution grants and check authorization
inside it. Pinning search_path alone does not restore RLS. Run Database Advisor → Security. See [rls-patterns.md](./rls-patterns.md).

## 13. Following a Supabase + Next.js Tutorial

**Impact**: HIGH — it puts Supabase into the frontend, which this project forbids.
**Symptom**: `@supabase/ssr`, `createBrowserClient`, `NEXT_PUBLIC_SUPABASE_*` or a Supabase call
in the client project (the frontend). Supabase is used only here, in Express.
**Fix**: Supabase lives only in Express. See [auth.md](./auth.md) and
backend-guide/03-auth/authentication.md §0.1.

## 14. Using `@supabase/auth-helpers-*`

**Impact**: MEDIUM — unmaintained.
**Status**: formally deprecated on npm — *"Package no longer supported."*
**Fix**: not used at all in this project (no Supabase in the frontend). See [client-libs.md](./client-libs.md).

## 15. Session Mode on Port 6543

**Impact**: Connection errors.
**Status**: **DEPRECATED**. Session mode on port 6543 was removed 2025-02-28. Use port 5432 for session mode (ORMs with prepared statements).
**Source**: https://supabase.com/changelog/32755
