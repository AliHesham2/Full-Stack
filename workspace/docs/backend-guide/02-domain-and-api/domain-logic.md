# Domain Logic — Where Business Rules Live

> **Part of:** Domain & API — Backend Project Guide
> **Read when:** Writing or reviewing any business rule, use case, invariant, or orchestration — or deciding where logic belongs.
> **Related:** `../01-foundation/architecture.md`, `../01-foundation/project-structure.md`, `validation.md`, `../05-data/transactions.md`
> **Status:** Stable principles

---

## 1. Position

Business logic is the reason the system exists; everything else is plumbing that serves it. The recurring failure (human and AI) is logic **leaking** into plumbing: rules scattered across controllers, queries, and client code until no one can say where the truth lives. This file defines placement, ownership, and the anatomy of a use case.

## 2. The Placement Rule

> **Business rules live in the domain layer. Orchestration lives in use cases. Everything else — routes, controllers, repositories, clients — transports data and calls in. It never decides.**

Diagnostic translations:

- "Can a user cancel this order?" → domain (a rule about order state)
- "Cancel the order, refund the payment, send the email, publish the event" → use case (orchestration)
- "Parse `orderId` from the URL and validate its shape" → edge (transport)
- "Write the cancellation row" → repository (plumbing)

A controller containing `if (order.status === 'shipped')` is a defect. A domain entity importing an HTTP library is a defect. Both mean the placement rule was violated.

## 3. The Use Case (Application Layer) Anatomy

One use case = one user-visible or system-triggered intention ("CancelOrder", "IssueRefund", "RenewSubscription"). Recommended shape:

```
UseCase(input, context) → Result
  1. authorize        — can this actor perform this action? (deny early)
  2. load             — fetch what's needed (repo reads, by ID)
  3. validate business rules  — invariants the domain enforces (not input shape!)
  4. decide/compute   — pure domain functions produce new state + side-effect intents
  5. persist          — transactional write (what MUST be atomic goes here)
  6. side effects     — jobs/events/emails queued AFTER commit (see §6)
  7. return           — a DTO, not an entity
```

Properties that make use cases reviewable:

- **Authorization first.** Never load-then-leak: authorize before doing expensive work, but note that ownership checks require the loaded object — both checks happen server-side, always (`../03-auth/authorization.md`).
- **One transaction boundary, explicitly stated.** Everything inside step 5 is atomic; nothing outside is. See `../05-data/transactions.md`.
- **Side effects after commit.** Emails, events, webhooks, cache invalidations are enqueued once data is durably written — otherwise a rollback leaves an email sent for an order that doesn't exist. Use the transactional outbox when the event MUST match the write (`../06-async-and-scale/events-and-messaging.md`).
- **Deterministic and unit-testable:** with repo interfaces faked, a use case test exercises rules and orchestration without I/O.

## 4. Domain Layer: Entities & Invariants

The domain layer models **state + rules**, and is pure (no I/O, no framework):

- **Entities** have identity and lifecycle (`Order`), **value objects** are compared by value (`Money`, `Address`).
- **Invariants** are constructor/mutation guards: "an order total cannot be negative", "a shipped order cannot be edited". If a rule can be violated by constructing an object, the object model has failed — invalid states should be unrepresentable where practical.
- **Pure functions first:** prefer `Order.canBeCancelled(now)` returning a decision over `Order.cancel()` mutating hidden state — decisions are trivially testable and composable.
- **Anemic warning:** if domain classes are getter/setter bags and rules live in services, the "domain layer" is decorative. Either model the rules properly or honestly collapse to a transaction-script style for that module (acceptable for CRUD-heavy domains; see `../01-foundation/architecture.md` §5).

### Business validation ≠ input validation

| | Input (syntactic) validation | Business validation |
|---|---|---|
| Question | "Is this shape valid?" | "Is this action allowed/correct now?" |
| Example | `qty` is an integer ≥ 1 | "this SKU has 3 left; you ordered 5" |
| Lives at | edge, schema-declarative (`validation.md`) | domain/use-case, code/tests |
| Failure response | 400 with field errors | 403/409/422 with domain error code |
| Depends on DB state? | No | Often |

Duplicating shape checks in both layers adds noise; omitting rule checks in the domain adds breaches. Keep them separate and each in its place.

## 5. Domain Ownership

- Every business concept has exactly **one owning module** (`../01-foundation/architecture.md` §4.2). Other modules consume it via its published use cases/DTOs.
- **Cross-module rules** are a design smell: if "refund" rules must know "fraud score" internals, either extract the shared concept, or represent fraud as a query/service call from the refund use case with the dependency pointing one way.
- Rule duplication across modules (two implementations of "is this user premium") is worse than the coupling of sharing — consolidate into the owner and expose a contract.

## 6. Side Effects & Timing

| Side effect | Timing | Mechanism |
|---|---|---|
| DB writes that must be atomic with the decision | inside the transaction | repositories |
| Notification (email/push) | after commit | background job (`../06-async-and-scale/background-jobs.md`) |
| Event for other modules/services | after commit, reliably | outbox pattern (`../06-async-and-scale/events-and-messaging.md`) |
| External API call (payment capture) | usually inside the use case but OUTSIDE the DB transaction; reconcile states explicitly | see below |
| Cache invalidation | after commit | versioned keys / explicit invalidation (`../06-async-and-scale/caching.md`) |

**The external-call-inside-transaction trap:** holding a DB transaction open across a payment-API call couples your locks to a third party's latency. Pattern instead: (1) write "pending" state, commit; (2) call external API; (3) write outcome, commit; (4) reconciliation job sweeps stale pendings. More states, but each step is retryable and observable — the trade is deliberate (`../05-data/transactions.md` §7).

## 7. Testing Implications

- Use cases: unit tests with faked repos — cover each rule, each orchestration branch, each authorization denial.
- Domain functions: pure unit tests, no mocks needed.
- Placement test: a rule change (e.g., "cancellation window is now 48h") must touch exactly ONE module's files. If it touches controllers, SQL strings, and a frontend copy — placement failed.

## 8. Failure Modes

- **Fat controllers:** handler does load + rule-check + write + email inline. Refactor: extract use case; handler becomes parse→call→serialize.
- **Service-layer dumping ground:** `OrderService` with 40 methods and no coherent intention; every module imports it. Split into intention-shaped use cases; the class name "XService" with unbounded growth is the tell.
- **Logic in queries:** business rules encoded in ad-hoc SQL fragments (`WHERE status != 'cancelled' AND ...`) scattered across repos — one rule change = an archaeology dig. Rules belong in domain functions; repos execute storage, not policy.
- **Silent rule duplication:** the client, the use case, AND a DB constraint all check the same rule with subtly different logic. Decide per rule: where is it authoritative (domain), where is it defense-in-depth (DB constraint), where is it UX sugar (client hint only — never authoritative).
- **AI pattern-stuffing:** generated `AbstractOrderStateStrategyFactory` hierarchies for a two-state workflow. Complexity-control bar applies (`../01-foundation/complexity-control.md`).
