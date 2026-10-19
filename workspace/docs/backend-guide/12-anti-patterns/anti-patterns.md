# Anti-Patterns — The Catalog of Known Bad Ideas

> **Part of:** Anti-Patterns — Backend Project Guide
> **Read when:** Reviewing a design, smelling something wrong, or catching yourself (AI agents especially) mid-implementation of one of these. Each entry: what it looks like → why it happens → the damage → the fix (deep treatment in the referenced file).
> **Related:** every other file in this guide · `../00-navigation/agent-workflow.md` §3 (the complexity discipline)
> **Status:** Living catalog · Generated code reproduces these patterns at a high enough rate that this file is the cheap insurance policy of the whole guide — read it against your own output, not only against inherited code

---

## How to Use This Catalog

Anti-patterns are **attractive nuisances**: every one of them looks reasonable in a small context and fails at scale, under attack, or at 3AM. When any entry matches your current design, don't argue with it — read the referenced file's failure-modes section and count how many you're about to inherit.

---

## A. Structure & Architecture

| # | Anti-pattern | Signature | Fix |
|---|---|---|---|
| A1 | **Fat controller / business logic in routes** | handler loads, decides, writes, emails inline | extract use case; edge translates (`../02-domain-and-api/domain-logic.md`) |
| A2 | **Service-layer dumping ground** | `OrderService` with 40 unrelated methods, imported by everyone | intention-shaped use cases; split by capability |
| A3 | **Giant module / god object** | one module owns "everything about users" incl. auth, billing, prefs | split by change-frequency and data ownership (`../01-foundation/architecture.md` §4) |
| A4 | **Utils/common dumping ground** | `shared/utils.ts` importing nothing, imported by everything, containing business words | admission bar (`../01-foundation/project-structure.md` §4) |
| A5 | **Premature microservices** | 3-person team, 6 services, one shared database, all deploy together | modular monolith default (`../01-foundation/architecture.md` §2–3) |
| A6 | **Distributed monolith** | services calling each other synchronously for every operation | boundaries along domains, async where decoupling is real (`../06-async-and-scale/events-and-messaging.md` §2) |
| A7 | **Unnecessary queues / event buses** | broker introduced for a single consumer that wanted a function call | honesty test (`../06-async-and-scale/events-and-messaging.md` §2) |
| A8 | **Excessive abstraction / pattern stuffing** | `AbstractFactoryStrategyProvider` for a two-state workflow; pass-through layers forwarding calls | abstraction must be earned (`../01-foundation/complexity-control.md` §6) |
| A9 | **Anemic domain model** | entity classes as getter/setter bags; rules live in services (or nowhere) | invariants in domain, or honestly collapse to transaction scripts (`../02-domain-and-api/domain-logic.md` §4) |
| A10 | **Boundary theater** | domain-named folders; shared tables; cross-module imports everywhere | ownership rules + import linting (`../01-foundation/project-structure.md` §3, §7) |
| A11 | **Global mutable state** | singletons mutated per-request (current user, tenant, "context") — race factory under concurrency | explicit context propagation as parameters (`../11-specialized-concerns/multi-tenancy.md` §3) |
| A12 | **The unused flexibility layer** | "generic" config/options/interfaces built for imagined future requirements that arrive different | YAGNI + complexity bar (`../01-foundation/complexity-control.md` §2) |

## B. API & Contracts

| # | Anti-pattern | Signature | Fix |
|---|---|---|---|
| B1 | **Everything-200 API** | errors returned as `{success:false}` with 200 | status discipline (`../02-domain-and-api/api-design.md` §3) |
| B2 | **Inconsistent error shapes** | each route improvises its error JSON | one envelope + central mapping (`../02-domain-and-api/api-design.md` §4) |
| B3 | **Unpaginated lists** | `GET /orders` returns ALL orders, forever | pagination from day one (§5 same file) |
| B4 | **Client-trusted server values** | prices/totals/roles/tenant IDs accepted from request bodies | server-owned values rule (`../02-domain-and-api/validation.md` §4) |
| B5 | **Leaky error responses** | stack traces/SQL/driver errors serialized to clients | envelope + correlation ID (`../04-security/security-foundations.md` §6) |
| B6 | **Break-and-bump versioning reflex** | v3 shipped because a field was renamed "cleanly" | additive evolution + deprecation discipline (`../02-domain-and-api/api-design.md` §9) |
| B7 | **Verbs-in-URL / RPC-shaped REST** | `/createOrder`, `/getUserAndTheirInvoices` | resource design (§2 same file) |
| B8 | **Unbounded write endpoints** | POST accepting 10MB arrays / 500-line batches | payload caps + batching semantics (§6, §8 same file) |

## C. Data & Consistency

| # | Anti-pattern | Signature | Fix |
|---|---|---|---|
| C1 | **Missing constraints** | nullable-everything, no FKs/uniques — "the app validates" | DB as last defender (`../05-data/data-architecture.md` §4) |
| C2 | **Hidden transactions** | multi-statement writes without a transaction (autocommit obliviousness) | explicit transaction discipline (`../05-data/transactions.md` §2) |
| C3 | **Giant transaction** | BEGIN at request start, external API + email inside, COMMIT at end | shrink to atomic core; pending-state pattern (§7 same file) |
| C4 | **Unsafe retries** | blanket retry middleware on non-idempotent POSTs | classify + idempotency keys (`../05-data/concurrency-and-idempotency.md` §3.3, §6) |
| C5 | **Check-then-act races** | `if (stock >= qty) update` as separate steps | atomic guards / unique constraints (§4 same file) |
| C6 | **N+1 queries** | 1 list query + N per-item queries (often ORM-lazy, invisible) | batch/join + query-count tests (`../05-data/data-access.md` §4) |
| C7 | **Unbounded queries** | SELECT without LIMIT; deep OFFSET pages | bounds + cursor pagination (`../02-domain-and-api/api-design.md` §5) |
| C8 | **Business logic in SQL fragments** | the same WHERE-clause business rule copy-pasted in 12 queries | rules in domain functions; repos execute storage only (`../02-domain-and-api/domain-logic.md` §8) |
| C9 | **Big-bang migrations** | rename+drop+retype in one release, deployed at once | expand-contract (`../05-data/migrations.md` §2) |
| C10 | **In-place balance mutation** | UPDATE balances with no ledger; money without history | append-only ledger + derived balances (`../11-specialized-concerns/time-and-money.md` §2.3) |
| C11 | **Float money** | prices/proration in float; cents evaporate | integer minor units / exact decimals (§2.1 same file) |
| C12 | **Soft-delete-everywhere** | `deleted_at` on all tables, forgotten filters leaking zombie rows | per-table decision + partial uniques (`../05-data/data-architecture.md` §6) |

## D. Security

| # | Anti-pattern | Signature | Fix |
|---|---|---|---|
| D1 | **BOLA (missing object-level authz)** | `GET /orders/:id` without ownership/tenant check | per-object authorization + denial tests (`../03-auth/authorization.md` §4) |
| D2 | **Edge-guard-only authorization** | admin middleware passed ⇒ no further checks | two-layer enforcement (§2 same file) |
| D3 | **Trust of client input** | IDs, enums, "internal" fields accepted raw; client validation as the only validation | boundary validation (`../02-domain-and-api/validation.md`) |
| D4 | **String-concatenated SQL** | "we escaped the quotes" | parameterization is binary (`../04-security/security-foundations.md` §3) |
| D5 | **JWT cargo-cult** | long-lived tokens in localStorage, no revocation path | model-per-client decision (`../03-auth/authentication.md` §2.1) |
| D6 | **Secrets in code/logs/config files** | `DATABASE_URL` committed "temporarily"; tokens in log lines | secret manager + redaction (`../04-security/secrets-management.md`) |
| D7 | **Enumeration-friendly responses** | "user not found" vs "wrong password" distinction | uniform errors + timing (`../03-auth/authentication.md` §7) |
| D8 | **Obfuscation as access control** | unguessable IDs, secret URLs, hidden endpoints | authorization, not obscurity (`../03-auth/authorization.md` §4.3) |
| D9 | **CORS wide-open** | `*` + credentials "to make it work" | no browser origin at all and no `cors` middleware: only authorized external server callers use this API (`../03-auth/authentication.md` §0, `../04-security/security-foundations.md` §9) |
| D10 | **Unsigned webhooks** | inbound provider events processed on arrival | HMAC + timestamp + replay window (`../11-specialized-concerns/webhooks.md` §2.2) |
| D11 | **Mass assignment** | `update(req.body)` binding whole objects; `role` field honored | strict schemas + field allowlists (`../02-domain-and-api/validation.md` §3) |

## E. Operations & Reliability

| # | Anti-pattern | Signature | Fix |
|---|---|---|---|
| E1 | **Swallowed errors** | empty catch / `catch → null` / logged-to-nowhere | taxonomy + never-swallow (`../07-reliability/error-handling.md` §8) |
| E2 | **No timeouts** | default (infinite) client timeouts; zombie requests pinning the fleet | explicit per-dependency timeouts (`../07-reliability/reliability.md` §3) |
| E3 | **Retry storms** | retries without backoff/jitter/budgets amplifying an outage | bounded, jittered, classified retries (`../05-data/concurrency-and-idempotency.md` §6) |
| E4 | **Circuit-breaker cosplay** | breakers everywhere / never calibrated | tools-with-failure-modes, not virtues (§4 same file) |
| E5 | **Invisible async failures** | unhandled rejections vanishing; DLQ nobody monitors | crash on programmer errors + DLQ alerts (`../06-async-and-scale/background-jobs.md` §5) |
| E6 | **Cache-first architecture** | cache before measurement; staleness bugs; debug-opacity | decision-first caching (`../06-async-and-scale/caching.md` §1) |
| E7 | **Unenumerable invalidation** | write path can't name the keys it invalidates | versioned keys/generations (§4–§5 same file) |
| E8 | **Metrics-blind system** | deployed without RED metrics; first problem found by users | instrument at birth (`../08-observability/observability.md` §3) |
| E9 | **String logs, no correlation** | prose logs; every incident starts with grepping | structured + correlation IDs (§2 same file) |
| E10 | **Untested backups** | backup dashboard green; restore never exercised | drills + measured RTO (`../11-specialized-concerns/disaster-recovery.md` §4) |
| E11 | **Deploy-without-drain** | every release = error spike "noise" | graceful shutdown tested (§5 same file) |

## F. Process & AI-Specific

| # | Anti-pattern | Signature | Fix |
|---|---|---|---|
| F1 | **Mock-pyramid testing** | 90% mocks; SQL/wiring untested; green + useless | integration-first (`../09-testing/testing-strategy.md` §3) |
| F2 | **Happy-path-only tests** | all inputs valid, all dependencies cooperative | failure-path battery mandatory (§5 same file) |
| F3 | **Coverage theater** | `toBeDefined()` × 300 for the number | tests name the defect they catch (§1) |
| F4 | **Flag landfill** | hundreds of flags, no owners, no removal | lifecycle + 40-day alarms (`../11-specialized-concerns/feature-flags.md` §4) |
| F5 | **Dependency maximalism** | package-per-problem; hallucinated/unvetted names; install scripts trusted | selection bar + supply-chain hostility (`../10-delivery/dependencies.md` §2–3) |
| F6 | **Config sprawl** | 47 files reading process.env directly | one typed config (`../10-delivery/configuration.md` §2) |
| F7 | **ADR amnesia** | decisions live in chat history; repo shows only consequences | decision records with revisit triggers (`../10-delivery/documentation-and-decisions.md` §4) |
| F8 | **Scope-creep generation (AI)** | asked for an endpoint, delivered an endpoint + repository pattern + cache + queue + factory | match the blast radius of the request (`../00-navigation/agent-workflow.md` §6) |
| F9 | **Confident-wrong freshness (AI)** | deprecated APIs/patterns generated from stale training assumptions; OAuth implicit grants; composition-rule passwords | freshness labels + verify-external for volatile items (`../00-navigation/agent-workflow.md` §6) |
| F10 | **Tutorial-shaped production (AI)** | maximum machinery from teaching examples shipped as architecture | boring-defaults discipline (`../01-foundation/complexity-control.md` §6) |
| F11 | **Self-trust (AI)** | first draft treated as verified; security checklist skipped because "it looks right" | validation order is mandatory (`../00-navigation/agent-workflow.md` §2 step 6) |
| F12 | **Framework/tutorial archaeology** | 2026 codebase carrying 2015 ceremony because the template said so | yearly structure audit (`../01-foundation/architecture.md` §8) |
| F13 | **Hallucinated dependencies / slopsquatting** | AI generates plausible-but-wrong package names; attackers register the names | verify every package exists in the real registry, with the real version, from the real scope — before it enters the lockfile (`../10-delivery/dependencies.md` §2) |
| F14 | **Premature abstraction** | AI generates "base classes" and "generic utilities" before two concrete cases exist | abstractions must be earned by repetition, not anticipated (`../01-foundation/complexity-control.md` §6) |
| F15 | **Cargo-cult observability** | AI instruments everything with metrics and traces without knowing what questions to answer | instrument for questions you actually ask, not for "completeness" (`../08-observability/observability.md` §1 — metrics answer questions cheaply; logs answer questions expensively; choose per question) |

---

## Meta-Pattern: How Anti-Patterns Get In

Every entry above enters through one of five doors — watch the doors and the catalog mostly writes itself:

1. **Copying** (tutorials, templates, other codebases, AI training data) without asking "what problem does this solve HERE?"
2. **Optimism** ("we'll add validation/limits/tests later" — later never comes; later is a different person/context)
3. **Fear of boredom** (new tech, distributed everything, enterprise patterns for a CRUD app)
4. **Deadline compression** (the shortcut that becomes the architecture)
5. **Invisibility** (failure modes that don't show up until load, attack, or 3AM — which is most of them)

The countermeasure for all five is the same: the discipline of the workflow loop — inspect before building, justify complexity, test failure paths, and treat your own (especially AI-generated) first draft as unverified until checked against this guide.
