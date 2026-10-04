# CI/CD — The Pipeline That Guards Production

> **Part of:** Delivery — Backend Project Guide
> **Read when:** Designing or improving the pipeline: stages, gates, artifact handling, deployment automation, rollback workflows.
> **Related:** `../09-testing/testing-strategy.md` §6, `deployment.md`, `../05-data/migrations.md` §5, `../10-delivery/dependencies.md` §4
> **Status:** Stable principles · Platform syntax (GitHub Actions-class) in platform guide

---

## 1. Position

CI is a **contract**: nothing reaches production except through gates that the team trusts. Its value is negative-space — the regressions, vulnerabilities, and broken migrations that never ship. A pipeline that can be talked around (merge-with-red, "I'll fix it after") is decoration with compute costs.

## 2. The Stage Map (Canonical Order)

```
commit/PR
  ↓
1. Lint + format check + typecheck        (seconds — catch machine-catchable issues)
2. Unit tests                              (fast, no I/O)
3. Integration tests + API/contract tests  (containers: real DB per ../09-testing/§3)
4. Migration gate                          (chain applies clean; volume-shaped check for hot tables)
5. Security gates                          (dependency audit, secret scan, SBOM where required)
6. Build artifact                          (one immutable artifact — image/bundle, tagged by commit)
7. Deploy to staging                       (auto on main)
8. Post-deploy smoke tests                 (health/ready, contract probes, money-path smoke)
9. Deploy to production                    (gated: approval or auto per policy)
10. Post-deploy verification               (error-rate/metrics watch window — automated rollback trigger)
```

Design rules:

- **Fail fast, fail loud:** cheap stages first; a failure names the stage, the test, and the reason (CI output is a debugging interface — treat its readability as a feature).
- **Stages are isolated jobs** with explicit inputs (the artifact from stage 6 is THE deployable — never "rebuild from source in the deploy job", which reintroduces drift between what was tested and what ships).
- **PR-time vs merge-time:** everything that blocks merge runs on the PR; long-tail checks (volume migration tests, full E2E) may run post-merge pre-deploy — but nothing unverified reaches the deploy gate.

## 3. The Migration Gate (Where Pipelines Meet the Database)

- CI applies the full chain from empty DB (runs clean, in order — `../05-data/migrations.md` §6).
- Schema-affecting PRs run against a **realistic-volume database** (generated data): lock-time and duration assertions for ALTERs/CREATE INDEX on hot tables (§2 stage 4).
- Deploy-time sequencing (migrate-then-rollout, advisory lock against concurrent migrators) lives in `deployment.md` §2 — CI proves the migration is safe; deploy executes it in order.
- Rollback story visible in the pipeline metadata per release (which contract-phase steps it contains determines rollback posture).

## 4. Security Gates

- Dependency audit blocking on high/critical (override path = ticket + owner + expiry — `../10-delivery/dependencies.md` §4)
- Secret scanning on every push (history-aware)
- Linter/security-static-analysis rules for the high-value classes: parameterized queries, auth-check presence on new routes — cheap heuristics catch systemic omissions early
- Image scanning of the built artifact (base-image CVEs, baked-in secrets)
- CI-credential hygiene: pipelines hold scoped, environment-limited tokens; a compromised PR must not be able to exfiltrate prod secrets (fork PRs: zero access to secrets; branch protections: no self-approval to gate-skips — supply-chain attacks in 2025–2026 explicitly targeted CI as the persistence vector)

## 5. Deployment Flows (In-Pipeline)

| Strategy | Mechanics | Fits |
|---|---|---|
| **Rolling** | instances replaced gradually behind LB; readiness-gated | stateless tiers, default |
| **Blue-green** | parallel env, smoke, traffic switch, keep-old-for-rollback | DB-schema-coupled releases, fast-rollback needs |
| **Canary** | 1%→10%→100% with automated health analysis (error-budget burn per `../08-observability/observability.md` §4) | high-traffic products, risky changes |

Automation floor: **deploy and rollback are both one command / one button.** Manual runbook deploys are outage-shaped under pressure. Automated rollback trigger: post-deploy window watches error-rate/latency vs pre-deploy baseline and reverts without a human in the loop (the metric definitions live in `../08-observability/observability.md` §4).

## 6. Pipeline Hygiene

- **Speed is a feature:** >10-min pipelines push people toward merging on hope. Parallelize, cache dependencies, split slow checks to post-merge.
- **Flaky-test quarantine with ticket** (`../09-testing/testing-strategy.md` §6) — flakiness in CI destroys the contract.
- **Trunk-based + short-lived branches** make the pipeline the integration point; long-lived branches make CI a lie (unmergeable by the time they're green).
- **Pipeline-as-code, reviewed** like application code: a modified pipeline is a modified contract — branch-protect the pipeline files.
- **Secrets in CI** are scoped per environment and masked in logs (`../04-security/secrets-management.md` §2).

## 7. Failure Modes

- **The green-with-asterisks culture:** red builds merged "temporarily" — CI's authority collapses; the gates become advisory (§1).
- **Rebuild-instead-of-promote:** prod artifact built fresh in the deploy job with different inputs than the tested one — the tested-shipped gap is a real exploit/defect window (§2).
- **Untested migrations in the pipeline:** code gates perfect, migration runs first time in prod with a table-locking surprise (§3).
- **Approval theater:** prod deploy "requires approval" from the same person who opened the PR — rubber stamps are not gates.
- **Snowflake environments:** staging auto-deploys, prod deploys manually via a tribal-knowledge runbook that drifts quarterly — the pipeline ends at the edge of production, exactly where the risk concentrates.
- **Rollback-by-heroics:** "rollback" means restoring from backup and praying — because deploy automation never included the reverse path (§5 automation floor).
- **CI-asset compromise:** broad tokens + fork-PR secret access + unprotected workflow files = the exact anatomy of the 2025–2026 supply-chain incidents (§4).
