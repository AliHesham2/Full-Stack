# Backend Initialization — Building the Foundation

> **Part of:** Foundation — Backend Project Guide
> **Read when:** Starting a new backend project or evaluating whether an existing one has a sound foundation.
> **Related:** `architecture.md`, `../02-domain-and-api/validation.md`, `../07-reliability/error-handling.md`, `../10-delivery/configuration.md`, `../08-observability/observability.md`
> **Status:** Stable principles · Current as of 2026

---

## 1. Goal

A strong initial foundation that makes every future feature cheaper and safer — **without speculative infrastructure**. The foundation is finished when a service can: start, validate its configuration, handle a request, return a structured error, log with correlation IDs, report health, and pass THE CHECK (`npm run check`, §2.8). It is NOT finished when Kubernetes, Kafka, Redis, and a service mesh are wired up. Those solve problems this project does not have yet (see `../01-foundation/complexity-control.md`).

## 2. Bootstrap Sequence (Order Matters)

Establish foundations in this order — each step depends on the previous one being usable:

```
1. Project skeleton & boundaries      → structure before code
2. Configuration + validation         → the app must fail fast on bad config
3. Logging + correlation IDs          → you cannot debug what you cannot see
4. Error handling taxonomy            → before routes exist
5. Health/readiness endpoints         → before any deploy target exists
6. Validation foundation              → before the first endpoint accepts input
7. Database connection + migrations   → before the first table
8. Test scaffold + THE CHECK          → before the first feature (see the note below)
9. Security baseline                  → headers, CORS policy, request limits
10. Developer tooling                 → lint, format, typecheck, pre-commit
```

**In this project, THE CHECK comes right after the skeleton:** step 1 also sets up §2.8's test
runner and `npm run check`, and §2.10's lint and typecheck config, because every later step is
verified with actual evidence under the assigned role procedure and shared delivery cycle. The rest keeps this order.

### 2.1 Project skeleton
The app lives in `backend/` at the project root, with its own `package.json` (this guide's README and project-structure define its app boundary). Read `architecture.md` and `project-structure.md` before creating directories. Decide module boundaries early — renaming directories is cheap in week 1 and expensive in month 6. Create only the structure needed for the first feature plus the shared foundations above; do not pre-create `services/`, `workers/`, `events/` directories with nothing to put in them.

### 2.2 Configuration
- All configuration flows through **one typed, validated config object** loaded at startup. Scattered `process.env` reads across the codebase are a defect (untraceable, untestable, typo-prone).
- **Fail fast:** missing or malformed required config must crash the process at startup with a precise message, never at first use in production traffic. A config schema violation is a deployment error, not a runtime error.
- Secrets are injected, never committed. See `../10-delivery/configuration.md` and `../04-security/secrets-management.md`.
- Separate **deployment variance** (URLs, hosts, pool sizes) from **business behavior** (feature settings). The latter may become feature flags (`../11-specialized-concerns/feature-flags.md`).

### 2.3 Logging
- Structured logs (JSON) from the first line of the framework — retrofitting structure onto string logs is a rewrite.
- A correlation/request ID generated at the edge and propagated through every layer, including background jobs. See `../08-observability/observability.md`.
- Levels used with discipline: `error` = something is broken and needs a human; `warn` = degraded but handled; `info` = lifecycle events; `debug` = local detail. If everything logs `info`, nothing is observable.

### 2.4 Error handling
Decide the error taxonomy before the first route: operational vs programmer errors, domain errors, validation errors, and their HTTP mapping. See `../07-reliability/error-handling.md`. The foundation ships one shared error type hierarchy and one central error-to-response translator — not per-route ad-hoc error shapes.

### 2.5 Health & readiness
- `GET /health` (liveness): "is the process alive" — must not touch the database.
- `GET /ready` (readiness): "can this instance serve traffic" — checks critical dependencies (DB reachable, migrations applied). Used by deploy targets to gate traffic. See `../07-reliability/reliability.md`.

### 2.6 Validation
One schema-validation library chosen and wrapped in a single boundary helper. Every request body, query, and param is validated at the edge — no exceptions, including "internal" endpoints. See `../02-domain-and-api/validation.md`.

### 2.7 Database
Connection pool with explicit size budget, timeouts, and a startup connectivity check. Migration tooling wired from day one — the first migration is the project's `init` migration, not a retrofitted script. See `../05-data/migrations.md`.

### 2.8 Tests & CI
Test runner and a database isolation strategy (per-test database or transaction rollback). Establish this before the first feature or the first feature will not be tested. See `../09-testing/testing-strategy.md`. THE CHECK below supplies local quality evidence; git, CI and release operations require their actual owner/host authorization. Repository remotes do not change role ownership or prove verification. A CI pipeline for deployment (`../10-delivery/ci-cd.md`) is the user's to set up. The implementer writes no test: the testers write every test, setup's included, each in its own folder (`tests/<tester>/`), in the test round, so the `test` script passes with no tests until the first one exists.

**Test-discovery gate:** `--passWithNoTests` is temporary for untested bootstrap only.
A zero-test bootstrap check verifies tooling, not product behavior. The Roster/brief identifies the actual Tester operator(s) and their permitted test folders; no fixed clients or mandatory pair is implied. After the assigned initial implementation test phase produces nonempty discovered/executed tests, Orchestrator reviews every result and the required AC coverage.

Orchestrator routes the strict-script change through Planner's approved software TODO/change record and briefs Implementer to remove the temporary flag (`vitest run`). Implementer edits application configuration and reports the new candidate; Orchestrator personally reruns the existing suites and quality gates. Do not automatically send the fix back to Tester. If new/changed tests or unavailable verification are required, keep that scope blocked for the owner's decision under the shared delivery cycle.

Before setup is recommended ready, verify nonempty execution in each assigned test scope, the strict script and actual evidence for applicable automatable ACs. Missing/skipped/unverified required checks block that readiness. Pending owner manual E2E remains explicit and blocks final product acceptance until the actual owner check. A passing script alone is not acceptance.

Run a production build before readiness.

**THE CHECK — `npm run check`.** This command supplies necessary configured quality-gate evidence; it does not alone prove acceptance. Orchestrator verifies actual candidate results, and a worker's "done" never substitutes for them:

```json
"scripts": {
  "lint": "eslint . --max-warnings 0",
  "typecheck": "tsc --noEmit",
  "test": "vitest run --passWithNoTests",
  "check": "npm run lint && npm run typecheck && npm run test"
}
```

The project's fixed auth rule (Express never sets a browser cookie; `../03-auth/authentication.md`
§0) is enforced as a lint error, not just written down. The rule matches the property on ANY
object, so `response.cookie(…)` cannot slip past a check that only knew the name `res`. Verified in
a real setup (2026-09-23): `res.cookie`, `response.cookie` and `clearCookie` fail; returning
`{ token }` in JSON passes.

```js
// eslint.config.mjs — the rule block, added after the TypeScript preset
{
  rules: {
    'no-restricted-properties': ['error',
      { property: 'cookie', message: 'Express never sets cookies — return { token } in JSON (03-auth/authentication.md §0).' },
      { property: 'clearCookie', message: 'Express never sets cookies — logout deletes the session record (03-auth/authentication.md §0.1).' },
    ],
  },
}
```

### 2.9 Security baseline
Applied at the edge middleware before the first endpoint exists:
- Request body size limits and request timeouts
- Helmet-equivalent security headers; HSTS once TLS is terminated (relevant even for APIs)
- CORS: the current contract is server-to-server with no browser origin allowed. Clients are external authorized services, not a required Next.js app. CORS is not authentication; use actual service/user/resource controls. Browser/cookie support needs an explicit approved change. Never use wildcard origins with credentials.
- Rate limiting skeleton on the router, even if the initial limits are generous. See `../04-security/security-foundations.md`.

### 2.10 Tooling
Formatter, linter, typechecker pinned in config; editor/CI agreement enforced. Automated formatting removes an entire category of review noise and AI-generated style drift.

## 3. What NOT to Build at Initialization

| Tempting | Why not | Revisit when |
|---|---|---|
| Microservice split | Single team, single domain → operational cost with zero benefit | Independent deployment pressure is real and measured |
| Message broker / event bus | No events exist yet; a queue for zero jobs is pure liability | async work becomes a real requirement (`../06-async-and-scale/background-jobs.md`) |
| Cache layer | No measured read problem; caches add consistency risk | query profiling shows hot reads (`../06-async-and-scale/caching.md`) |
| Custom auth | Security-critical; prefer established providers/libraries (`../03-auth/authentication.md`) | almost never |
| Feature-flag platform | A config value suffices initially | gradual rollouts become a real need |
| Admin dashboard / CRUD generator | Speculative surface area | real operational need |
| Multi-region anything | One region has not been exhausted | latency/compliance data exists |

## 4. Foundation Quality Checklist

An initialized project passes when:

- [ ] Process refuses to start with invalid/missing config (tested)
- [ ] Every request carries a correlation ID through logs
- [ ] Unknown routes and unhandled errors return ONE consistent error shape
- [ ] `/health` and `/ready` exist and behave differently
- [ ] Request validation is mandatory at the edge and tested with invalid payloads
- [ ] DB migrations run against a real database in the test run
- [ ] THE CHECK (lint + typecheck + tests) passes on every step: the orchestrator runs it
- [ ] Security headers, body limits and timeouts applied; no browser origin allowed by CORS (§2.9)
- [ ] No secrets in the repository (verified by scanning)
- [ ] README explains how to run, test, and migrate — a new engineer (or agent) succeeds from it alone

## 5. Failure Modes of Initialization

- **Framework-maximalism:** every middleware, plugin, and scaffold enabled — most become attack surface and upgrade burden. Enable on need.
- **Config drift:** the same setting defined in code defaults, `.env.example`, and docs, disagreeing. The typed config object is the single source; docs point to it.
- **Test scaffold deferred:** "we'll add tests after the first feature" — the first feature then defines the pattern of untested features. Non-negotiable: scaffold ships with foundation.
- **Premature DRY:** foundations built as generic abstractions ("BaseController", "GenericRepository") before two concrete cases exist. Abstractions must be earned by repetition (`complexity-control.md`).
