# Decision Index — Route Your Question to the Right Knowledge

> **Part of:** Navigation — Backend Project Guide
> **Read when:** You have a concrete decision to make and need to know which file owns it.
> **Related:** `agent-workflow.md` (the loop)
> **Status:** Stable · Updated as the guide evolves

---

## How to Use

Find your question (or the nearest one). Read the **Primary** file — it contains the decision framework. Read **Also** files only if your situation touches those areas. Do not read entire directories speculatively.

---

## A. Placement & Structure Decisions

| Question | Primary | Also |
|---|---|---|
| Where does this business logic belong? | `02-domain-and-api/domain-logic.md` | `01-foundation/project-structure.md` |
| Which module owns this responsibility? | `01-foundation/architecture.md` § boundaries | `02-domain-and-api/domain-logic.md` § ownership |
| Should this be a service/use-case/domain component? | `02-domain-and-api/domain-logic.md` § anatomy | `01-foundation/complexity-control.md` |
| Does this justify a new directory/module? | `01-foundation/project-structure.md` § new-module test | — |
| Should I add another abstraction layer? | `01-foundation/complexity-control.md` | `01-foundation/architecture.md` § abstraction |
| Layered vs modular vs microservices? | `01-foundation/architecture.md` | `01-foundation/complexity-control.md` |
| What do I bootstrap first in a new project? | `01-foundation/initialization.md` | `10-delivery/configuration.md` |

## B. API Decisions

| Question | Primary | Also |
|---|---|---|
| How do I design this endpoint/contract? | `02-domain-and-api/api-design.md` | — |
| Which status code / error format? | `02-domain-and-api/api-design.md` § errors | `07-reliability/error-handling.md` |
| Pagination approach? | `02-domain-and-api/api-design.md` § pagination | `05-data/data-access.md` |
| How do I version / evolve the API safely? | `02-domain-and-api/api-design.md` § evolution | — |
| Is REST right here, or something else? | `02-domain-and-api/api-design.md` § beyond REST | `06-async-and-scale/realtime.md`, `11-specialized-concerns/webhooks.md` |
| Rate limiting / abuse prevention? | `02-domain-and-api/api-design.md` § abuse | `04-security/security-foundations.md` |
| Where does validation go? | `02-domain-and-api/validation.md` | — |
| Client sent a price/total — do I trust it? | `02-domain-and-api/validation.md` § trust | `11-specialized-concerns/time-and-money.md` |

## C. Auth Decisions

| Question | Primary | Also |
|---|---|---|
| Sessions or tokens for this client type? | `03-auth/authentication.md` | — |
| How do I store/handle passwords? | `03-auth/authentication.md` § passwords | — |
| Refresh token rotation & revocation? | `03-auth/authentication.md` | — |
| Service-to-service auth? | `03-auth/authentication.md` § s2s | `04-security/security-foundations.md` |
| RBAC / ABAC / ownership checks? | `03-auth/authorization.md` | — |
| User can access another user's object? | `03-auth/authorization.md` § BOLA | `09-testing/testing-strategy.md` § denial tests |
| Multi-tenant isolation? | `11-specialized-concerns/multi-tenancy.md` | `03-auth/authorization.md` |

## D. Data Decisions

| Question | Primary | Also |
|---|---|---|
| Who owns this data / these tables? | `05-data/data-architecture.md` § ownership | `01-foundation/architecture.md` |
| Relational or non-relational? | `05-data/data-architecture.md` | — |
| What ID type should I use? | `05-data/data-architecture.md` § 4.1 | `02-domain-and-api/api-design.md` §2 (public IDs) |
| Index design for this query? | `05-data/data-architecture.md` § indexes | `06-async-and-scale/performance.md` |
| Soft delete or hard delete? | `05-data/data-architecture.md` § lifecycle | `11-specialized-concerns/privacy-compliance-audit.md` |
| ORM or raw queries here? | `05-data/data-access.md` | — |
| How big should the connection pool be? | `05-data/data-access.md` §5.1 | `06-async-and-scale/scalability.md` |
| N+1 / slow query? | `05-data/data-access.md` | `06-async-and-scale/performance.md` |
| Should this be transactional? | `05-data/transactions.md` | — |
| Optimistic or pessimistic locking? | `05-data/transactions.md` § locking | — |
| Can this request arrive twice? (double-submit, retry) | `05-data/concurrency-and-idempotency.md` | `02-domain-and-api/api-design.md` § idempotency |
| How do I change this schema safely? | `05-data/migrations.md` | `10-delivery/deployment.md` |

## E. Scale & Async Decisions

| Question | Primary | Also |
|---|---|---|
| Should this be cached? | `06-async-and-scale/caching.md` | — |
| How do I invalidate the cache? | `06-async-and-scale/caching.md` § invalidation | — |
| Should this run in background? | `06-async-and-scale/background-jobs.md` | — |
| Job retries / dead letters? | `06-async-and-scale/background-jobs.md` | — |
| Events or direct call? | `06-async-and-scale/events-and-messaging.md` | `01-foundation/complexity-control.md` |
| Need to publish events transactionally? | `06-async-and-scale/events-and-messaging.md` § outbox | `05-data/transactions.md` |
| File uploads / storage? | `06-async-and-scale/file-and-object-storage.md` | `04-security/security-foundations.md` |
| WebSockets vs SSE vs polling? | `06-async-and-scale/realtime.md` | — |
| It's slow — what now? | `06-async-and-scale/performance.md` | `05-data/data-access.md` |
| Do we need to scale horizontally / shard? | `06-async-and-scale/scalability.md` | `01-foundation/complexity-control.md` |

## F. Failure & Operations Decisions

| Question | Primary | Also |
|---|---|---|
| How do I classify/handle this error? | `07-reliability/error-handling.md` | — |
| Can this error be retried safely? | `07-reliability/error-handling.md` § retry | `05-data/concurrency-and-idempotency.md` |
| Timeouts / circuit breakers? | `07-reliability/reliability.md` | — |
| Health/readiness checks? | `07-reliability/reliability.md` | `10-delivery/deployment.md` |
| Graceful shutdown? | `07-reliability/reliability.md` § shutdown | — |
| What do I log here? | `08-observability/observability.md` | — |
| Debugging a production failure? | `08-observability/observability.md` § debugging | `07-reliability/*` |
| What metrics/alerts matter? | `08-observability/observability.md` | — |

## G. Quality & Delivery Decisions

| Question | Primary | Also |
|---|---|---|
| What tests does this change need? | `09-testing/testing-strategy.md` | — |
| Mock the DB or use a real one? | `09-testing/testing-strategy.md` | — |
| Should I add this npm package? | `10-delivery/dependencies.md` | `04-security/security-foundations.md` § supply chain |
| How do I structure config/env/secrets? | `10-delivery/configuration.md` | `04-security/secrets-management.md` |
| CI pipeline stages? | `10-delivery/ci-cd.md` | — |
| Deployment sequencing & rollback? | `10-delivery/deployment.md` | `05-data/migrations.md` |
| Deploy dark vs immediate activation? | `10-delivery/deployment.md` § 1.1 | `11-specialized-concerns/feature-flags.md` |
| Record this decision? | `10-delivery/documentation-and-decisions.md` | — |
| Feature flag or not? | `11-specialized-concerns/feature-flags.md` | — |
| GDPR / PII / audit requirements? | `11-specialized-concerns/privacy-compliance-audit.md` | `08-observability/observability.md` § audit logs |
| Backups / disaster recovery? | `11-specialized-concerns/disaster-recovery.md` | — |
| Timezones / currency / rounding? | `11-specialized-concerns/time-and-money.md` | — |
| Inbound/outbound webhooks? | `11-specialized-concerns/webhooks.md` | — |
| "Something feels wrong in this design" | `12-anti-patterns/anti-patterns.md` | — |

---

## Cross-Cutting Question Templates

When a task touches multiple areas, compose the readings. Examples:

**"Add a checkout endpoint that charges payment and emails a receipt"**
→ `02-domain-and-api/api-design.md` (contract, idempotency) + `02-domain-and-api/domain-logic.md` (use-case) + `05-data/transactions.md` (what is transactional) + `06-async-and-scale/background-jobs.md` (email async) + `11-specialized-concerns/time-and-money.md` (amounts) + `09-testing/testing-strategy.md`

**"Users report seeing other users' data occasionally"**
→ `03-auth/authorization.md` (BOLA) + `11-specialized-concerns/multi-tenancy.md` + `05-data/concurrency-and-idempotency.md` (if caching involved) + `09-testing/testing-strategy.md` (add denial tests)

**"API is slow under load"**
→ `06-async-and-scale/performance.md` (measure first) + `05-data/data-access.md` (queries) + `06-async-and-scale/caching.md` (then cache) + `06-async-and-scale/scalability.md` (only if truly needed)
