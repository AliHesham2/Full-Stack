# Migrations — Evolving Schema Without Breaking Production

> **Part of:** Data — Backend Project Guide
> **Read when:** Changing any table structure, adding/removing columns or indexes, changing constraints or enums — or reviewing a release that includes schema changes.
> **Related:** `data-architecture.md`, `../10-delivery/deployment.md`, `../10-delivery/ci-cd.md`
> **Status:** Stable principles · Tool syntax lives in the ORM/migration tool guide

---

## 1. The Prime Rule

> **Deployed schema changes must be backward-compatible with BOTH the old and new application code, because at some moment both are running.**

This single rule generates nearly everything below. Rolling deploys, crashed deploys, rollbacks — all mean old code + new schema coexists. The pattern that honors the rule is **expand → migrate (backfill) → contract**, never big-bang.

## 2. Expand–Contract Playbook

Changing column `users.name` (text) → `first_name` + `last_name`:

**Release N (expand):**
1. Add new columns nullable, no defaults that rewrite the table (a default with table rewrite locks the table on big tables — engine-dependent, verify)
2. Deploy code that writes BOTH old and new columns (dual-write), reads OLD
3. Compatibility achieved: old & new code both run correctly

**Release N.x (backfill):**
4. Backfill in **batched** background work (keyset-selected batches (PostgreSQL: bounded SELECT in a CTE, then UPDATE ... FROM that batch; UPDATE itself has no LIMIT)), never one giant UPDATE (lock duration, replication lag, WAL spikes)
5. Verify: row counts, NULL counts, sample diffs

**Release N+1 (switch):**
6. Code reads NEW, keeps dual-writing (rollback safety)

**Release N+2 (contract):**
7. Stop writing OLD; then, in a later maintenance moment, drop OLD column / add NOT NULL + constraint

**Never in one release:** add column + backfill + drop old + switch reads. Each arrow is a deploy gate.

### Operation-specific cautions

| Operation | Safe shape | Danger shape |
|---|---|---|
| Add column | nullable, no rewrite-triggering default | NOT NULL + DEFAULT on huge table in one step |
| Add index | CREATE INDEX CONCURRENTLY-equivalent (no table lock) | plain CREATE INDEX on the hot table at peak |
| Add constraint | NOT VALID first, then VALIDATE separately | immediate VALID constraint scanning+locking the table |
| Change column type | expand new column + dual-write + backfill | in-place ALTER on a live column |
| Rename column | expand new + dual-write + backfill + switch + drop | in-place rename (breaks old code instantly) |
| Drop column | stop writing → wait ≥1 full release → drop | drop in the same release that stops using it |
| Enum value add | append-only in most engines | reordering/renaming/deleting values |
| Not-null on existing | backfill first, then NOT NULL | constraint with rows violating it |

## 3. Migration Hygiene

- **Migrations are ordered, immutable history** — committed as code, applied in order, never edited after merging to main (a migration that ran somewhere is history; fix-forward with a new migration).
- **One logical change per migration** — reviewable, revertible-by-hand units.
- **Deterministic and idempotent where the tool supports it** — same result on every environment; no environment-specific branches inside migrations.
- **Seed data ≠ migrations:** reference/seed data ships as a separate, re-runnable script; test data never ships at all.
- **Every migration has a down-path story:** either a real `down` or a written manual reversal. "We'll figure it out" = irreversible by admission (`../01-foundation/complexity-control.md` §5).

## 4. Data Migrations (Backfills) vs Schema Migrations

- Schema changes and long-running data backfills are **separate artifacts**: schema migrations must be fast/lock-light so deploys stay instant; backfills are jobs — bounded batches, progress tracking, resumable, observable (`../06-async-and-scale/background-jobs.md`).
- Backfill correctness: dual-write keeps new rows converging while the job walks old rows; the job must tolerate the moving frontier (iterate by keyset, re-check conditions).
- Post-backfill verification queries run as part of the release checklist — count mismatches block the switch.

## 5. Running Migrations in Deployments

Sequence (mechanics in `../10-delivery/deployment.md`):

1. Schema migrations run as a **gated pre-deploy step** (migration job acquires an advisory lock preventing concurrent migrators — two instances migrating simultaneously is a corruption classic)
2. Health-gate: migration success is a deploy precondition; failure blocks, never "continues anyway"
3. App rollout proceeds only on compatible schema (expand-phase changes)
4. Contract-phase steps scheduled explicitly with their own releases

Rollback story per release: expand-phase releases roll back cleanly (old code works on expanded schema); a release containing a contract step needs its own documented reversal — this asymmetry is exactly why they don't ship together.

## 6. Testing Migrations

- CI applies the full migration chain from empty DB → schema asserts (the chain must run clean, in order, every time)
- CI runs the **migration on production-scale-shaped data** (generated volume) for anything touching hot tables — lock durations that are invisible on 100 rows are outages on 100M
- Up-AND-down tests where downs exist (proves they're real)
- Post-deploy smoke: read/write path over changed tables before traffic shifts

## 7. Failure Modes

- **The big-bang refactor migration:** "rename + drop + retype" in one release, deployed Friday, rolled back Sunday into a restore-from-backup incident. Expand-contract or accept the risk in writing.
- **Lock-blind ALTERs:** CREATE INDEX on a 50M-row table during peak — the site freezes; CONCURRENTLY variants exist precisely for this.
- **Migration drift:** someone hotfixed staging with a manual ALTER; now staging ≠ chain ≠ production. Manual changes go through a new migration, always.
- **Concurrent migrators:** first instance boots and migrates while the old instance is still serving with old code AND a second new instance starts migrating too — advisory lock + expand-phase discipline (§5.1, §2).
- **Backfill by UPDATE-everything:** one statement, hours of locks, replication lag swallowing replicas. Batched job or nothing.
- **The edited-applied migration:** "small fix" to a migration already applied in staging diverges staging's history from everyone else's — fix-forward only.
