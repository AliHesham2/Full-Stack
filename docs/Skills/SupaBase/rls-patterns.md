# Row Level Security Patterns

> Project contract: Browser -> Next.js -> Express on Node.js -> Supabase.
> SDKs, credentials, authorization and provider integration live in Express only.

> Source review: 2026-09-27. This is a template reference, not a running project.
> Historical execution/measurement statements below are author reports without a supplied
> reproduction harness; they are not acceptance evidence. This review ran no Supabase
> CLI, SQL, deployed function, or real provider integration.

## How RLS Works

Policies are Postgres's rule engine. Each policy attaches to a table and acts as an added WHERE clause on every query. For restricted roles with SELECT grants, no matching policy means no visible rows;
write checks can reject writes, and missing SQL grants produce permission errors.

Supabase uses two default roles:
- `anon` — unauthenticated requests
- `authenticated` — logged-in users (JWT present)

`service_role` bypasses RLS entirely. This is why it must stay server-side.

Source: https://supabase.com/docs/guides/database/postgres/row-level-security

## The Mandatory Procedure (Every Table)

```sql
-- 1. Enable RLS
ALTER TABLE your_table ENABLE ROW LEVEL SECURITY;

-- 2. Revoke default grants (recommended for new projects)
REVOKE ALL ON your_table FROM anon;
REVOKE ALL ON your_table FROM authenticated;

-- 3. Grant only the required operations (read-only in this example)
GRANT SELECT ON your_table TO authenticated;

-- 4. Create policies
CREATE POLICY "Users can read own rows"
  ON your_table FOR SELECT
  TO authenticated
  USING (user_id = auth.uid());
```

Table grants and RLS policies are separate gates. Revoke unnecessary operations, then GRANT only required operations back. A policy cannot restore a revoked table privilege. For writes, grant the matching operation and define its policy; UPDATE also needs a SELECT policy. Test as the actual authenticated role, never only as postgres.

## Pattern 1: Simple Ownership

User owns their rows via `user_id` column.

```sql
CREATE POLICY "select_own" ON todos FOR SELECT TO authenticated
  USING (user_id = auth.uid());
CREATE POLICY "insert_own" ON todos FOR INSERT TO authenticated
  WITH CHECK (user_id = auth.uid());
CREATE POLICY "update_own" ON todos FOR UPDATE TO authenticated
  USING (user_id = auth.uid())
  WITH CHECK (user_id = auth.uid());
CREATE POLICY "delete_own" ON todos FOR DELETE TO authenticated
  USING (user_id = auth.uid());
```

**Caveat**: `auth.uid()` reads trusted claim context. The Data API validates the JWT
and installs that context. Raw SQL needs a restricted role and transaction-local claims
installed by trusted server/test setup; it does not automatically authenticate a JWT.

## Pattern 2: Organization/Team Membership (Multi-Tenant)

For B2B or team-based apps. Join through a membership table.

```sql
-- Tables
CREATE TABLE organizations (id UUID PRIMARY KEY DEFAULT gen_random_uuid());
CREATE TABLE organization_members (
  org_id UUID REFERENCES organizations(id) ON DELETE CASCADE,
  user_id UUID NOT NULL,
  role TEXT NOT NULL DEFAULT 'member' CHECK (role IN ('owner','admin','member')),
  PRIMARY KEY (org_id, user_id)
);

-- RLS policy checking membership
CREATE POLICY "org_members_select" ON projects FOR SELECT TO authenticated
  USING (
    org_id IN (
      SELECT org_id FROM organization_members
      WHERE user_id = auth.uid()
    )
  );
```

**Membership prerequisite**: enable RLS and grant authenticated SELECT with a
self-membership policy (`user_id = auth.uid()`). Restrict membership writes and role
changes to authorized server operations; users must not promote themselves. Avoid
membership policies recursively querying their own table. Examples assume matching
operation grants and RLS-enabled target tables.

**Why this pattern**: Policies based on membership joins scale better than hardcoding exceptions across tables. Centralize authorization logic in the membership table.

Source: https://supabase.com/docs/guides/database/postgres/row-level-security

## Pattern 3: Public Read, Authenticated Write

```sql
CREATE POLICY "public_read" ON posts FOR SELECT TO anon, authenticated
  USING (true);
CREATE POLICY "auth_insert" ON posts FOR INSERT TO authenticated
  WITH CHECK (user_id = auth.uid());
```

## Pattern 4: Role-Based Access

This example assumes reports has an org_id column. An organization admin is scoped to that organization; platform-wide admins require a separate explicit policy.

```sql
CREATE POLICY "admin_all" ON reports FOR ALL TO authenticated
  USING (
    EXISTS (
      SELECT 1 FROM organization_members
      WHERE user_id = auth.uid() AND role = 'admin'
        AND org_id = reports.org_id
    )
  );
```

## `USING` vs `WITH CHECK` — the rule most guides get wrong

Verified by running each case on PostgreSQL 18:

| Command | `USING` | `WITH CHECK` |
|---|---|---|
| `SELECT` | which rows are visible | n/a |
| `DELETE` | which rows may be deleted | n/a |
| `INSERT` | **does not exist** | the only gate — the new row must satisfy it |
| `UPDATE` | which rows may be targeted | the resulting row — **defaults to the `USING` expression when omitted** |

```sql
-- INSERT: this is the dangerous one
create policy i on notes for insert to authenticated with check (true);
--            → a user successfully inserted a row owned by someone else

create policy i on notes for insert to authenticated
  with check (user_id = (select auth.uid()));
--            → the same insert was rejected

-- UPDATE: a USING-only policy is already safe
create policy u on notes for update to authenticated
  using (user_id = (select auth.uid()));
--            → "update notes set user_id = <someone else>" was REJECTED
```

`pg_policy` shows why: for that UPDATE policy `polwithcheck` is `null`, and Postgres
falls back to the `USING` expression when checking the new row. Add an explicit
`WITH CHECK` to an UPDATE policy only when it must **differ** from `USING` — for example,
letting a user move a row into a different org they also belong to.

**So the real rule is**: `WITH CHECK` is mandatory on INSERT, optional on UPDATE, and
meaningless on SELECT/DELETE.

## Two ways RLS is bypassed entirely

Enabling RLS on a table does **not** mean every path to that data is protected. Both of
these are ordinary things to build, and both silently return every row.

### 1. Views bypass RLS by default

Postgres creates views as `security definer` — they run as the view's **owner**, which on
Supabase is `postgres`, which has `bypassrls`. Expose that view through PostgREST and any
anon caller reads the whole underlying table.

```sql
-- ✗ Leaks: runs as postgres, ignores every policy on orders
create view public.order_summary as
  select id, total, user_id from public.orders;

-- ✓ Postgres 15+: the view obeys the caller's policies
create view public.order_summary
  with (security_invoker = true)
  as select id, total, user_id from public.orders;
```

Fix existing views with
`alter view public.order_summary set (security_invoker = true);`. On older Postgres, keep
views out of exposed schemas or revoke `anon`/`authenticated` access.

**Audit query — run this on every project:**
```sql
select n.nspname as table_schema, c.relname as table_name
from pg_class c join pg_namespace n on n.oid = c.relnamespace
where n.nspname = 'public' and c.relkind = 'v'
  and not ('security_invoker=true' = any(coalesce(c.reloptions, array[]::text[])));
```

Materialized views do not re-evaluate underlying RLS for each reader. Restrict their
access explicitly instead of treating them like security-invoker ordinary views.

### 2. `security definer` functions in an exposed schema

A `security definer` function runs as the role that **created** it. Created by `postgres`,
it carries `bypassrls` — and PostgREST makes it callable as an RPC by anyone.

```sql
-- ✗ Callable over the Data API, runs with bypassrls, and search_path is hijackable
create function public.get_all_orders() returns setof public.orders
  language sql security definer as $$ select * from orders $$;

-- ✓ If you truly need definer rights: pin search_path and check authorization yourself
create function public.get_my_orders() returns setof public.orders
  language plpgsql security definer
  set search_path = ''                      -- mandatory: blocks search_path attacks
as $$
begin
  return query select * from public.orders where user_id = (select auth.uid());
end; $$;
revoke execute on function public.get_my_orders() from public, anon;
grant execute on function public.get_my_orders() to authenticated;
```

**The `search_path` attack is not theoretical.** Run on PostgreSQL 18 with a
`security definer` function reading an unqualified `secrets` table:

```
function                            caller's search_path        returns
public.unpinned()   (no pin)        default                     "REAL_SECRET"
public.unpinned()   (no pin)        evil, public                "HIJACKED"   ← attacker's table
public.pinned()     search_path=''  evil, public                "REAL_SECRET"
```

Any role with the necessary database/schema CREATE privileges (inspect actual grants)
can plant a table that shadows yours, then call your definer function and have it read
their table **with `postgres` privileges**. Pinning `search_path = ''` and fully qualifying
every object closes it.

**Rules**: default to `security invoker` (the Postgres default) unless you have a specific
reason; every `security definer` function must `set search_path = ''` and fully qualify
every table; each must do its own authorization; revoke `execute` from `anon` unless
anonymous access is intended. Also `revoke create on schema public from public;` on new
projects so untrusted roles cannot plant shadow objects at all.

Supabase's **Database Advisor → Security** flags both classes. Run it before every release.

## Performance

RLS adds per-row checks. On large tables this matters:

- **Wrap function calls in a `select`** — the single highest-impact fix, straight from the
  Supabase docs. `(select auth.uid())` becomes an `initPlan` that Postgres evaluates
  **once per statement**; a bare `auth.uid()` is re-evaluated **once per row**.
  ```sql
  -- ✗ auth.uid() runs for every row scanned
  using (user_id = auth.uid())
  -- ✓ evaluated once, cached for the statement
  using (user_id = (select auth.uid()))
  ```
  Wrap only row-independent claim/helper expressions. STABLE alone does not make
  row-dependent functions safe to evaluate once for the whole statement.
- **Consider indexes for RLS predicates**: measure representative query plans and table
  sizes. Avoid blanket claims that every unindexed predicate forces a scan on every request.
- **Add explicit filters to queries**: RLS filters rows, it does not tell the planner what
  you want. Keeping `.eq('user_id', userId)` in the client query lets the index be used.
- **Specify `TO authenticated` / `TO anon`** on every policy. A policy with no role is
  evaluated for *all* roles, including requests it could never match.
- **Avoid correlated subqueries in policies on hot tables**: a membership join is fine at
  moderate scale; at high scale denormalize the access column onto the row.

Every claim in this file's "bypass", "USING vs WITH CHECK" and "Performance" sections was
verified by executing it on PostgreSQL 18 (PGlite) with Supabase-shaped roles
(`anon`/`authenticated`) and an `auth.uid()` reading `request.jwt.claims`. Measured results:
a plain view returned **all 3 rows** where the table returned **2**; `security_invoker = true`
brought it back to 2; a `security definer` RPC returned 3 while the `security invoker`
version returned 2; an RLS-enabled table with no policy returned **0**; and
`using (user_id = (select auth.uid()))` produced the plan
`Filter: (user_id = (InitPlan 1).col1)` — the per-statement caching described above.

Source: https://supabase.com/docs/guides/database/postgres/row-level-security

## Testing RLS

```sql
-- SQL Editor normally uses postgres, which bypasses RLS.
-- Use a real fixture UUID and the restricted role in an isolated transaction.
BEGIN;
SET LOCAL ROLE authenticated;
SELECT set_config('request.jwt.claims', '{"sub": "user-uuid-here", "role": "authenticated"}', true);
-- Then run the queries your policy should gate; reset after:
ROLLBACK; -- resets role and transaction-local claims
```

Executed on PostgreSQL 18: with the policy `using (user_id = (select auth.uid()))`, setting
`sub` to user 1111 returned row `[1]` and user 9999 returned row `[2]` — the simulation
genuinely isolates users, so it is a valid basis for tests.

### pgTAP: assert the policies themselves, not just the rows

`supabase start` ships the local DB; `create extension pgtap;` and write policy tests as
files under `supabase/tests/database/`, not production migrations. Run `supabase test db`
on a disposable stack when implementing. Historically reported available in pgTAP **1.3.5** — these are the assertions worth knowing,
because they identify missing policies directly; pair them with positive and negative row tests:

| Assertion | Catches |
|---|---|
| `policies_are('public','todos', ARRAY['own_select','own_insert'])` | a policy that was dropped or never created |
| `policy_cmd_is('public','todos','own_select','SELECT')` | a policy attached to the wrong command |
| `policy_roles_are('public','todos','own_select', ARRAY['authenticated'])` | a policy accidentally granted to `anon` |
| `table_privs_are('public','todos','anon', ARRAY[]::text[])` | default grants never revoked |
| `is_empty($$ select … $$)` | a query that must return nothing for this role |
| `results_eq($$ … $$, $$ … $$)` | exact expected rows per user |

```sql
begin;
select plan(3);

select policies_are('public', 'todos', array['own_select']);
select policy_roles_are('public', 'todos', 'own_select', array['authenticated']);

set local role authenticated;
select set_config('request.jwt.claims', '{"sub":"…-1111"}', true);
select results_eq($$ select id from todos order by id $$, $$ values (1) $$,
                  'user 1111 sees only their own row');

select * from finish();
rollback;
```

**Why the policy assertions matter more than they look**: a row test on an
RLS-enabled table with **no policy at all** can pass a negative empty-row test for the
wrong reason. Positive authorized-row tests must fail in that case. `policies_are` is what tells you the policy still exists.

**Critical**: Test with normal user JWTs, not service_role/secret keys. They bypass RLS entirely and will never catch missing or incorrect policies.

## Common Mistakes

1. **Enabling RLS but writing no policies** → SELECT returns no visible rows for restricted roles with grants; writes can fail policy checks. Not a security hole, but a confusing bug.
2. **Policy checks `auth.uid() IS NOT NULL`** → This just checks "is logged in", not ownership. Any authenticated user sees all rows.
3. **`with check (true)` on an INSERT policy** → anyone may write any row. INSERT has no
   `USING` clause at all, so `WITH CHECK` is the only gate. (For UPDATE the rule is
   different — see below; the usual "always add WITH CHECK to both" advice is wrong.)
4. **Testing only with service_role** → You will never catch RLS gaps this way.
5. **Exposing a view over an RLS-protected table** → the view bypasses RLS unless created
   `with (security_invoker = true)`. See above.
6. **`security definer` RPCs without `set search_path = ''`** → callable over the Data API
   with `bypassrls`. See above.
7. **Bare `auth.uid()` in a policy** → re-evaluated per row; wrap it in `(select …)`.

See [gotchas.md](./gotchas.md) for more pitfalls.