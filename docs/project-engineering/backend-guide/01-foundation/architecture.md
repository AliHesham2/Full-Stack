# Architecture — Decision Framework for Backend Structure

> **Part of:** Foundation — Backend Project Guide
> **Read when:** Choosing or revising the overall architecture; deciding boundaries, dependency direction, or layering.
> **Related:** `project-structure.md`, `complexity-control.md`, `../02-domain-and-api/domain-logic.md`, `../05-data/data-architecture.md`
> **Status:** Stable principles · Current as of 2026

---

## 1. Position

This guide does not impose one architecture. It teaches the decision process. The same reasoning that justifies a modular monolith for a 2-person product justifies services for a 20-team platform. What is NOT acceptable is choosing an architecture because it is fashionable, because tutorials use it, or because an AI agent generated its boilerplate confidently.

## 2. The Candidate Architectures

| Architecture | Best fit when | Cost / risk |
|---|---|---|
| **Modular monolith** (default) | One team, one deployable, domains still separable | Discipline required to keep module boundaries honest |
| **Layered monolith** (routes → services → data) | Small projects, few domains | Tends to rot into a "service layer dumping ground" without domain boundaries |
| **Service-oriented / microservices** | Independent teams needing independent deploys; hard scaling isolation per domain | Distributed failure modes, network security, operational tooling, eventual consistency everywhere |
| **Serverless functions-first** | Spiky, event-driven, low-traffic workloads | Cold starts, execution limits, stateless constraints, vendor coupling |

**Default recommendation:** modular monolith with enforced internal boundaries. It preserves the *option* to extract services later (boundaries already exist) without paying distributed-systems costs now. Extraction is an irreversible-ish step; deferring it is cheap.

## 3. Decision Criteria (Not Vibes)

Choose services over monolith only when **multiple** of these are true — measured, not anticipated:

1. ≥2 teams need independent deploy cadence for different domains
2. A specific domain has fundamentally different scaling/resource profile (verified by numbers)
3. Hard isolation requirement (compliance, blast radius) that code boundaries cannot satisfy
4. Different runtime/technology requirements per domain

If fewer than two apply: monolith. This guide decides that, so the PRD cites this section and asks nothing. Only when two or more apply is the choice left open: it is then an Open Question in the PRD, and its answer a DEC-n row (an ADR: `../10-delivery/documentation-and-decisions.md` §4).

## 4. Boundaries

Boundaries are the load-bearing concept — everything else is arranging furniture.

### 4.1 What a boundary is
A module/domain boundary is a line across which only **explicit contracts** pass. Inside: private implementation. Across: interfaces, DTOs, events. If another module can reach your tables directly, you do not have a boundary — you have a folder.

### 4.2 How to draw boundaries
- **By domain capability** ("orders", "billing", "identity"), not by technical layer ("controllers", "utils", "helpers"). Technical partitioning scatters one business rule across the whole codebase.
- **By data ownership:** each boundary owns its tables. No other module writes to them; reads go through the owning module's contract (or a documented read model exception, see `../05-data/data-architecture.md`).
- **By change frequency:** things that change together belong together. If every feature touches two "separate" modules, the boundary is wrong.

### 4.3 Dependency direction (inviolable)

```
edge (routes/HTTP, event consumers, job handlers)
   ↓ depends on
application (use cases / orchestration)
   ↓ depends on
domain (entities, invariants, policies)
   ↑ implements
infrastructure (DB repos, external APIs, cache, queue clients)
```

- **Domain depends on nothing** (no framework imports, no DB, no HTTP). If domain files import infrastructure, the architecture is already dead — fix immediately.
- Infrastructure depends on domain interfaces (dependency inversion), never the reverse.
- The edge layer is a translation shell: parse/serialize, authenticate, delegate. If the edge contains `if` statements about business rules, business logic has leaked (`../02-domain-and-api/domain-logic.md`).

## 5. Layering Decisions

**When is a layer justified?** A layer earns existence by answering: "what varies independently here?" Layers that merely forward calls ("pass-through services") add indirection without isolating change — delete them and let the caller talk to the next meaningful layer.

**Minimum viable layering for a domain module:**

```
edge/        → HTTP handlers, request parsing, response mapping
application/ → use cases: orchestration, transactions, authorization calls
domain/      → entities, invariants, domain policies (pure)
infra/       → repositories, external clients, cache implementations
```

Rules of thumb:

- A CRUD-only module may skip `domain/` (there is no invariant worth modeling). Admit it; don't fabricate anemic entity classes.
- A complex domain may need more structure inside `domain/` (aggregates, value objects). Add when the invariant complexity demands it, not for aesthetic completeness.
- **Never** create layers "for future flexibility" — that future arrives with different requirements than the layer anticipated (`complexity-control.md`).

## 6. Coupling & Cohesion

- **Cohesion:** everything inside a boundary works toward one capability. Symptom of low cohesion: module names like `common`, `utils`, `helpers`, `core`, `misc`.
- **Coupling:** minimize what crosses boundaries. Prefer passing DTOs over entities; prefer narrow interfaces (2–4 methods) over wide ones; prefer events (with an explicit contract) for notification, synchronous calls for queries you need answers to.
- **Transitive coupling is still coupling:** if module A imports B's types which import C's ORM models, A depends on C. Enforce with lint boundaries or import rules where the ecosystem allows.

## 7. Extensibility Decisions

| Extension mechanism | Use when | Warning |
|---|---|---|
| Plain function/module call | Default. Always. | — |
| Interface + implementations | ≥2 real implementations exist | "Future-proofing" with 1 implementation is speculative generality |
| Plugin/registry pattern | Third parties extend behavior | Real complexity: ordering, versioning, isolation |
| Event subscription | Reaction is genuinely optional/decoupled | Don't use events to fake control flow you actually need synchronously |

## 8. Revisiting Architecture

Architecture decisions are revisited when their **stated assumptions break**, not when novelty appears. Each architectural ADR should name: (a) assumptions, (b) revisit triggers, (c) migration direction if triggered. Example: "Assume single-region. Revisit when p95 latency outside region >X for ≥Y% of traffic. Direction: read replicas before full multi-region."

## 9. Failure Modes

- **Distributed monolith:** services deployed together, calling each other synchronously for every operation — all the latency and failure of distribution with none of the independence. Caused by drawing service boundaries along technical, not domain, lines.
- **Anemic everything:** "domain" layers that are bags of getters/setters with logic living in services — the boundary then protects nothing. Invariants belong in the domain (`../02-domain-and-api/domain-logic.md`).
- **Boundary theater:** folders named after domains, but shared tables, shared entities, and cross-imports everywhere. Auditable symptom: one schema change forces edits in N modules.
- **Architecture by inertia:** a 2026-era codebase still carrying 2015-era patterns (e.g., XML config ceremony, three-layer forwarding for CRUD) because "that's the template". Audit yearly.
