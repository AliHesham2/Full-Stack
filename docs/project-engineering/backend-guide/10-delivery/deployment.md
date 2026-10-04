# Deployment — Releases, Sequencing, and Rollback

> **Part of:** Delivery — Backend Project Guide
> **Read when:** Planning a release, sequencing migrations with code, designing rollback, or defining production readiness.
> **Related:** `ci-cd.md` (the pipeline that gets you here), `../05-data/migrations.md` (the schema half of every release), `../07-reliability/reliability.md` (health gates)
> **Status:** Stable principles · Platform mechanics (k8s/PaaS/VM) in platform guide

---

## 1. Release Model

- **One artifact, many environments:** the exact bytes tested in staging are the bytes deployed to prod; environment differences are config ONLY (`configuration.md` §3 runtime-vs-build-time).
- **Releases are small and boring:** small diffs = small blast radius, readable changelogs, bisectable failures. Big-bang releases concentrate risk into single terrifying moments (and their rollbacks are restores, not reverts).
- **Every release carries:** artifact tag, code SHA, config diff, migration list, rollback posture (§4), owner. Recorded where the on-call can find it under pressure — the deploy log IS an observability surface (`../08-observability/observability.md` §6.5).

### 1.1 Deploy ≠ Release (Feature Activation)

The safest pattern for risky features: **deploy dark, activate via flag**. The deploy (code delivery) ships the feature behind a flag that defaults to OFF; the release (behavior change) is a flag toggle — instant, reversible, no deploy required. This separates "shipping code" from "changing user behavior":

- **Deploy = code delivery** — the artifact is live but the feature is invisible
- **Release = behavior change** — flag activation, gradually or fully
- **Rollback of a behavior bug = flag toggle**, not a redeploy

When this matters: any feature touching money, auth, data migrations, or critical paths. The flag lifecycle rules in `../11-specialized-concerns/feature-flags.md` §2–§4 apply: name the class (release toggle vs kill switch), set the 40-day ceiling, test both states, remove with rollout completion. A release toggle that outlives its purpose is architecture debt.

## 2. Sequencing (Migrations × Code)

The canonical zero-downtime deploy:

```
1. Pre-deploy gate: pipeline green, artifact built, staging smoke-passed
2. Run schema migrations (expand-phase only — backward-compatible: additive,
   nullable, no destructive ops; advisory lock prevents concurrent migrators)
3. Health check: migration job success is a hard precondition — failure blocks rollout
4. Roll out instances gradually (rolling/blue-green/canary per ci-cd.md §5),
   each instance gated by readiness (../07-reliability/reliability.md §2)
5. Post-deploy window: automated baseline watch (error rate, latency) with
   auto-rollback trigger
```

Invariants:

- **Old code must survive new (expanded) schema** — that's what expand-phase migrations guarantee (`../05-data/migrations.md` §2). A release containing a contract-phase (destructive) migration ships separately, with its own reversal plan, NEVER bundled with the feature that stopped using the old shape.
- **Jobs/workers roll with the API tier** or after it — consumers of schema changes deploy on the same expand-phase discipline.
- **Feature activation decoupled from deployment** where the feature is risky: deploy dark, activate via flag (`../11-specialized-concerns/feature-flags.md`) — deploy = code delivery, release = behavior change; separating them converts "roll back a broken feature" from an emergency deploy into a toggle.

## 3. Zero-Downtime Techniques (What Actually Keeps It Up)

| Concern | Technique |
|---|---|
| In-flight requests during instance replacement | graceful shutdown drain (`../07-reliability/reliability.md` §5) — tested under load |
| New instances not ready | readiness gating (not liveness) — traffic shifts only when ready |
| DB locks during migration | CONCURRENTLY index builds, NOT VALID→VALIDATE constraints, batched backfills (`../05-data/migrations.md` §2 table) |
| Cache stampede on deploy (cold caches) | pre-warm critical keys / TTL tiers; watch hit-rate in the post-deploy window |
| Background jobs mid-deploy | visibility timeouts + idempotency make job handoff safe (`../06-async-and-scale/background-jobs.md` §3) |
| Scheduler double-fire during overlap | distributed lock / leader election (`../06-async-and-scale/scalability.md` §4) |

## 4. Rollback (Design It Before You Need It)

- **Code rollback = redeploy previous artifact** (one command — `ci-cd.md` §5 automation floor). The artifact registry retains the last N known-good.
- **Schema rollback is NOT the reverse of a code rollback:** expanded schema stays (it's harmless to old code — that was the point of expand-phase); contract-phase steps have written reversal runbooks per release. NEVER "roll back the database" by running destructive down-migrations against live data — data written by the new code doesn't un-write.
- **Data written by bad code:** rollback stops the bleeding; a data-repair job/plan (with idempotent, reviewed, backed-up-before-execution repair scripts) cleans up. Identify affected rows via the release window + correlation IDs (`../08-observability/observability.md` §6).
- **Rehearse:** rollback executed only under real incident stress is rollback-shaped hope. Canaries/blue-green keep the old env warm enough to actually revert traffic.

## 5. Production Readiness Review (Per Service, Before First Real Traffic)

- [ ] Health/ready endpoints wired to the platform's gating (`../07-reliability/reliability.md` §2)
- [ ] Graceful shutdown fits the platform's termination grace (tested, §3)
- [ ] Migrations: expand-phase compliant; backfills are jobs with progress; advisory lock present
- [ ] Observability minimum: structured logs with correlation IDs, RED metrics, error-rate alert with runbook link (`../08-observability/observability.md` §3–4)
- [ ] Timeouts + retry policy on every outbound dependency; degradation mode defined per critical dependency
- [ ] Secrets from the secret manager (not env-committed, not build-baked); rotation runbook exists
- [ ] Rate limits + payload caps at the edge; security headers verified
- [ ] Rollback: previous artifact redeployable in one command; reversal runbooks for any contract-phase migration in this release
- [ ] Capacity: known saturation point + alert margin (`../06-async-and-scale/scalability.md` §6)
- [ ] Runbook: "service is down at 3AM" — first three checks written (`../10-delivery/documentation-and-decisions.md`)

## 6. Failure Modes

- **Deploy ≠ release confusion:** features activate on deploy with no flag, forcing rollbacks for behavior bugs — toggle-decoupled releases (§2) turn incidents into config changes.
- **The untested migration surprise:** code perfect in staging; the `CREATE INDEX` lock in prod (100× the data) freezes the site mid-rollout — volume-shaped migration tests exist for this (`../05-data/migrations.md` §6).
- **Readiness-blind rollout:** traffic routed to instances still booting (or with unrun migrations) — 500s during every deploy, accepted as weather.
- **Bundle-of-doom releases:** 3 features + 1 migration + 1 framework upgrade in one release: bisecting the failure is impossible; rollback granularity is zero (§1).
- **Rollback theatre:** "we can roll back" (untested for 6 months; the schema has since moved; the old artifact's config vars no longer exist) — rehearsed rollback or no rollback (§4).
- **Config-drift deploys:** same artifact, different env, plus three "small manual fixes" applied to prod directly — the diff between staging and prod IS the incident waiting list; manual changes go through the pipeline or they don't happen.
