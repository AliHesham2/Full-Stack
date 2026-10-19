# Delivery and Observability

How code reaches production safely, and how you know what is happening after it gets there. Initialization sets up THE CHECK, the gate every step of the work passes ([initialization.md](../01-foundation/initialization.md)); the deployment pipeline is the user's to set up, and this file defines what it must validate, how releases roll out, and what telemetry runs in production.

## CI Pipeline

Every pull request runs these gates before merge. Order matters — fail fast on the cheapest checks:

```
install (npm ci, cached)
  → lint          # eslint . --max-warnings 0        [04-quality/code-quality]
  → typecheck     # tsc --noEmit
  → unit + component tests   # vitest run            [04-quality/testing]
  → build         # next build (Turbopack)           [01-foundation/initialization]
  → E2E + a11y    # only if the user automates E2E: playwright + @axe-core/playwright
  → bundle budget check                      # fails PR over budget  [04-quality/performance]
  → npm audit     # fails on high/critical           [04-quality/dependencies]
```

Rules:

- **The build gate exists because Turbopack builds are not guaranteed by dev.** A PR that only passed `npm run dev` has not proven it ships.
- **E2E tests run against the production build** served locally or in preview — not the dev server.
- **Required status checks** on GitHub: every gate above blocks merge; no manual overrides for individual PRs. If a check is flaky, fix or quarantine the test the same day ([testing.md](../04-quality/testing.md)) — never disable a gate to unblock a release.
- Keep total pipeline time under ~10 minutes; parallelize test shards when it grows.

## Preview Deployments

Every PR gets an isolated preview URL with its own environment variables namespace ([environments-and-config.md](./environments-and-config.md)). Purpose: reviewers, designers, and QA verify behavior without local setup.

- Seed preview databases with realistic data; do not share staging data across concurrent previews unless read-only.
- Post the preview link on the PR automatically.
- Teardown previews on merge/close to control cost.

## Release Flow

```
PR merged to main
  → production build
  → deploy to staging slot
  → smoke verification (health check + critical-path E2E, if the user automates E2E)
  → promote to production
  → post-deploy verification
  → monitor error rates for the next window
```

**Deployment strategy:** platform-native immutable deploys with instant rollback (Vercel/Netlify-style atomic deployments). For self-hosted targets, blue-green or rolling with health checks — never in-place mutation of running servers.

**Rollback:** redeploying the previous known-good build must be a single, rehearsed action. If rollback requires manual database surgery or config archaeology, the deployment design is wrong. Database migrations are the exception — they must be written to be backward-compatible (expand → migrate → contract) so the previous app version still runs against the new schema.

**Production verification after every deploy:** hit the health endpoint, check the critical paths end to end against production or a production-mirror (by hand, or with the E2E suite if the user automates it), confirm error-tracking dashboards show no new-issue spike. Deployment is not complete when the CDN says "ready" — it's complete when verification passes.

## Observability

You cannot maintain what you cannot see. Three signals, wired from day one:

### Error Tracking

Use a real error tracker (Sentry or equivalent) — console logs nobody reads are not observability.

- **Client:** install the SDK once at the root; source maps uploaded at build time so stack traces point at original TypeScript. Never ship source maps publicly.
- **Server:** wrap Server Actions and route handlers so server errors carry request context (route, user id when available, request id).
- **React boundaries:** report caught errors from `error.tsx` via `reportError`/SDK capture — otherwise boundary-swallowed errors are invisible.
- Triage discipline: every new issue type gets acknowledged, assigned, and either fixed or ticketed. An ignored error feed trains everyone to ignore errors.

### Real-User Monitoring

Wire `web-vitals` reporting to your analytics endpoint from day one ([performance.md](../04-quality/performance.md)). Lab scores in CI catch regressions; RUM catches what users actually experience on real devices and networks. Segment by route and device class — a median LCP of 1.8s means nothing if the p75 mobile checkout page is 5s.

### Structured Logging

Server-side only. Log *events* with context (`{ event: 'signup_failed', reason, userId }`), not prose strings. Never log secrets, tokens, full request bodies, or personal data. Client-side logging ships nothing beyond error reports.

## Privacy Boundaries

Telemetry is user data and is regulated (GDPR/CCPA):

- Collect the minimum needed to debug and measure: no keystrokes, no full URLs containing tokens or personal identifiers, no session recordings without explicit consent and a legal basis.
- Document every third-party script that receives user data ([security.md](../04-quality/security.md)); each one is both a privacy obligation and a supply-chain risk.
- Consent requirements depend on jurisdiction and data collected — default to consent-gating analytics in the EU; wire the consent state before the analytics SDK loads, not after.

## Documentation and Decisions

Use the current task's approved Planner Output and its canonical DEC/CHG records, linked from Orchestrator's single control record. Planner owns product/architecture proposals and approved decision updates; Orchestrator owns coordination/review state. Workers report source-backed recommendations rather than editing those records. Decisions made during execution follow the same ownership and actual approval rules as planning decisions; no parallel decision log is created.

Each consequential decision records context, options and reasons, actual authorized choice, consequences, assumptions/revisit triggers, source/version and status. Package choices record the app, need, compatible source/version and actual authorization in the same planning/task record. Retain superseded history and reconcile the software/design TODOs and affected evidence when a decision changes. See [backend decision-record guidance](../../backend-guide/10-delivery/documentation-and-decisions.md#4-adrs-architecture-decision-records) for the common fields. Neither documentation nor a reviewed plan grants deployment or release permission.

Keep README current: setup steps, env var list pointer to `.env.example`, scripts, and a one-paragraph architecture summary.
