# Agent Workflow — How to Use This Guide

> **Part of:** Navigation — Backend Project Guide
> **Read when:** You are an AI agent about to start ANY backend task. This is the entry point. Read this before writing any code.
> **Related:** `decision-index.md` (where does X belong), `../README.md`
> **Status:** Stable methodology · Current as of 2026

---

## 1. What This Guide Is (and Is Not)

This guide sits **between backend technologies and backend implementation**. It does not replace technology-specific guides (Node, Express, Supabase, PostgreSQL, Redis, auth libraries). It tells you:

- **WHAT decisions must be made** before writing code
- **HOW to reason about those decisions** (criteria, trade-offs, failure modes)
- **WHICH specialized guide** to consult for implementation details

It never tells you to build a specific product. It assumes you are building a real project whose requirements come from the project context, not from this guide.

**Non-negotiable stance:** multiple 2024–2026 studies find that a large fraction of
AI-generated code ships with security or design defects — the reported rates vary widely with
language, task and methodology, so **this guide deliberately quotes no single percentage**
(a precise-looking number invites more confidence than the evidence supports, which is the
exact failure this section exists to prevent). The direction is not in dispute, and it is all
the instruction you need:

**You are an AI agent. Treat your own first draft as untrusted input.** Verify it against this
guide before considering any task complete — specifically the validation order in §2 step 6.

---

## 2. The Core Loop

Execute this loop for every task, feature, or fix. Do not skip steps because the task "seems small" — small tasks are where security holes and architectural drift accumulate.

Start from [Roster](../../../rostar/roster.md), the assigned role prompt/Skill and [delivery cycle](../../../agents/orchestrator/skillz/delivery-cycle.md). This loop supplies engineering knowledge for the assigned phase; it does not appoint operators or authorize taking over other roles. Planner Output and the software/design TODOs provide current requirements and task scope at the paths supplied in the brief.

```
1. UNDERSTAND     — What is being asked? What phase is the project in?
2. INSPECT        — What exists already? (structure, patterns, conventions in the repo)
3. LOCATE         — Which knowledge applies? (use decision-index.md, read ONLY what you need)
4. PLAN           — Decide: placement, contracts, data, failure behavior, security boundary
5. IMPLEMENT      — Small, incremental, verifiable steps
6. VALIDATE       — Correctness → tests → security → failure behavior → architecture consistency
7. FINISH         — Clean up, report every decision with its source, no unnecessary redesign
```

### Step details

**1. UNDERSTAND** — Classify the task: new project? new feature? bug fix? performance issue? migration? refactor? If the task is ambiguous, the answer is NOT to invent requirements; flag ambiguity explicitly. Determine the project phase:

```
Project idea → Initialization → Foundation → Phase 1 → Validation → Phase 2 → Validation → ...
```

- **Initialization:** nothing exists yet → read `01-foundation/initialization.md` first
- **Foundation:** skeleton exists, features don't → read `01-foundation/*`
- **Feature phases:** foundation is stable → read only the domain files relevant to the feature

**2. INSPECT** — Never impose structure on a repo that already has one. Read the existing structure, find its conventions, and follow them unless they violate a security or correctness rule in this guide. Consistency with the existing codebase outranks theoretical purity. If the repo's structure conflicts with this guide's recommendations, prefer the guide only for security/correctness matters; for everything else, adapt.

**3. LOCATE** — Use `decision-index.md` to route your question to the minimal set of files. Do not read the entire guide for every task. Typical routing:

| Task type | Read first | Read if relevant |
|---|---|---|
| New project bootstrap | `01-foundation/initialization.md` | `01-foundation/architecture.md`, `10-delivery/configuration.md` |
| New endpoint/feature | `02-domain-and-api/api-design.md`, `02-domain-and-api/domain-logic.md` | `03-auth/*`, `05-data/*` as touched |
| Touches auth | `03-auth/authentication.md` or `authorization.md` | `04-security/security-foundations.md` |
| Touches data | `05-data/data-architecture.md` | `05-data/*` for the specific concern |
| Slow / hot path | `06-async-and-scale/performance.md` | `06-async-and-scale/caching.md`, `05-data/data-access.md` |
| Weird failure / race | `05-data/concurrency-and-idempotency.md` | `07-reliability/error-handling.md` |
| Adding a dependency | `10-delivery/dependencies.md` | — |
| Deploying / migrating | `10-delivery/deployment.md`, `05-data/migrations.md` | `10-delivery/ci-cd.md` |

**4. PLAN** — Before implementing, answer these questions (adapted from `decision-index.md`):

1. Where does this logic live? (which module, which layer)
2. Who owns this data? (which domain, which tables)
3. What is the API contract? (resources, verbs, status codes, error shape)
4. Where does validation happen? (syntactic at boundary, business in use-case)
5. Where does authorization happen? (which boundary, which policy)
6. Is this transactional? What consistency is required?
7. What happens if any dependency fails mid-operation?
8. Can this operation be safely retried? Is it idempotent?
9. What gets logged, and what must NEVER be logged?
10. What must be tested, including failure paths?

If you cannot answer these, you are not ready to write code.

**5. IMPLEMENT** — Incremental steps, each leaving the system in a working state. Prefer boring, standard solutions over clever ones. Every line of code is a liability; every dependency is a liability with a supply-chain attack surface (see `10-delivery/dependencies.md` — npm ecosystems suffered wormable supply-chain attacks in 2025–2026).

**6. VALIDATE** — The validation order matters:

1. **Correctness** — does it do what was asked, and NOT do what wasn't asked?
2. **Tests** — happy path + validation failures + authorization denials + failure paths. See `09-testing/testing-strategy.md`. The testers write them, each in its own folder (`tests/<tester>/`); the implementer runs every suite with `npm run check` and never writes or edits a test.
3. **Security** — run the question set in `04-security/security-foundations.md` § threat checklist. For AI agents this step is mandatory, not optional: unverified AI code is the documented top source of introduced vulnerabilities.
4. **Failure behavior** — what happens when the DB is down? When the same request arrives twice? When the response times out?
5. **Architecture consistency** — does the change respect existing boundaries and dependency direction? (`01-foundation/architecture.md`)

**7. FINISH** — Remove debris (dead code, TODOs you won't do, commented-out experiments). Report through the assigned role's Output with actual candidate/check evidence and source-backed decision proposals. Planner owns canonical DEC/CHG records; Orchestrator owns the linked control record (`10-delivery/documentation-and-decisions.md` §4). Do not write other roles' records or automatic follow-up assignments. Do not "improve" unrelated code. Do not redesign.

---

## 3. The Complexity Discipline

The single most common AI-agent failure mode is **cargo-cult architecture**: generating microservices, message queues, caches, caches for the caches, abstraction layers, and "enterprise" patterns for problems that do not exist yet. This guide is explicitly hostile to that.

Before introducing ANY structural complexity, you must satisfy the justification bar in `01-foundation/complexity-control.md`. Summary:

- **Default architecture:** modular monolith with clear internal boundaries. Not microservices.
- **Default async strategy:** synchronous request handling. Background jobs only for work that is slow, retryable, or not needed in the response.
- **Default caching strategy:** no cache. Add caching only when measurement shows a real read problem.
- **Default consistency strategy:** strong, transactional, boring. Eventual consistency only when scale or integration genuinely requires it.

Each of these defaults reverses with evidence, never with enthusiasm.

---

## 4. Phase Playbooks

### Phase: Initialization (nothing exists)
1. Read `01-foundation/initialization.md` end to end.
2. Establish: structure, config validation, logging, error handling, health endpoints, test scaffold.
3. Do NOT add: queues, caches, microservice scaffolding, feature-flag systems, Kubernetes manifests. Foundation ≠ maximal infrastructure.
4. Output: a running service with `/health` and a passing THE CHECK (`npm run check`).

### Phase: First features
1. Follow the Core Loop per feature.
2. Each feature must land with: validation, authorization, tests (including denial tests), error mapping.
3. Watch for boundary erosion — the first features set the pattern every future feature will copy.

### Phase: Growth / change requests
1. Inspect before planning. The existing architecture is a constraint, not a suggestion (unless it's a security/correctness violation).
2. Schema changes → `05-data/migrations.md` (expand-contract; never destructive in one step).
3. Behavior changes → check API compatibility rules in `02-domain-and-api/api-design.md`.

### Phase: Debugging production issues
1. Route through `08-observability/observability.md`: correlation ID → trace → logs → metrics.
2. Verify the failure class in `07-reliability/error-handling.md` (operational vs programmer error) before choosing retry vs fix.
3. Race conditions → `05-data/concurrency-and-idempotency.md`.

---

## 5. Hard Rules (Non-Negotiable)

These apply regardless of project phase, stack, or convenience:

1. **Authorization is checked server-side on every object access.** BOLA (Broken Object Level Authorization) is the #1 API vulnerability in the OWASP API Security Top 10 (2023 edition, still current). "The UI hides that button" is not authorization.
2. **Client input is never trusted**, including IDs, prices, tenant identifiers, and "internal" fields sent by clients.
3. **Secrets never appear** in source control, logs, error messages, or client responses.
4. **Every write endpoint that can be retried gets idempotency treatment** (key or natural idempotency) — see `05-data/concurrency-and-idempotency.md`.
5. **Transactions stay short** — no network calls, no slow work inside a DB transaction.
6. **Errors are never swallowed.** Empty catch blocks and `catch {} → return null` are bugs, not patterns.
7. **Destructive DB migrations never ship in the same release** as the code that stops using the old shape. See expand-contract.
8. **Every new dependency is a security decision** post-2025 supply-chain attacks, not a convenience decision.
9. **Tests are written for the failure paths**, not only the happy path — AI agents systematically under-test failure paths.
10. **Time is stored UTC, money is stored as integer minor units.** See `11-specialized-concerns/time-and-money.md`.

---

## 6. Reading Discipline (Token Efficiency)

- Read `README.md` once per session to re-orient.
- Read only the files routed by `decision-index.md` for the current task.
- Cross-references inside files are pointers, not obligations — follow only if the current task touches that area.
- Freshness labels in each file header mark items labeled `emerging` or `ecosystem-volatile`: follow them as written, and if one looks outdated, report it in the mailbox. Never silently override a project decision. Report source-backed discrepancies through Orchestrator to the authorized source owner. Research or guide updates require actual authority and preserve approved architecture and the local syntax policy.
