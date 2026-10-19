# Complexity Control — Justifying Every Ounce of Architecture

> **Part of:** Foundation — Backend Project Guide
> **Read when:** About to introduce a queue, cache, service split, abstraction, event bus, or ANY structural complexity — or auditing an AI-generated design that did.
> **Related:** `architecture.md`, `../06-async-and-scale/*`, `../12-anti-patterns/anti-patterns.md`
> **Status:** Stable principles · Deliberately conservative

---

## 1. Position

Complexity is not free at any scale, and AI-generated code has a documented tendency toward **cargo-cult architecture**: confidently generating microservices, event buses, and cache layers for problems that do not exist. This file is the brake. Every structural addition must pass a justification bar — this protects the project more than any positive pattern in this guide.

The core economic truth: **complexity is paid continuously** (debugging, onboarding, testing, upgrading) while its benefit is often hypothetical. A component that "might help at 100× scale" is a tax collected every single day until then.

## 2. The Justification Bar

Before introducing structural complexity, answer in writing, in Planner Output and the approved software TODO before any code (a DEC-n row: `../10-delivery/documentation-and-decisions.md` §4):

1. **What measured problem does this solve?** (latency number, incident, team friction — not "scale")
2. **What is the simplest alternative that solves 80% of it?**
3. **What failure modes does this ADD?** (every distributed component adds several — see the failure-mode sections of each guide)
4. **What does this make irreversible or hard to reverse?**
5. **Who/what will operate it at 3 AM?** (If the answer is "nobody", it is an outage waiting to happen)

If (1) is vague or (2) hasn't been seriously considered, **stop**. Default to boring.

## 3. Defaults That Require Evidence to Override

| Concern | Default | Evidence required to override |
|---|---|---|
| Architecture | Modular monolith | Multiple decision criteria from `architecture.md` §3 |
| Consistency | Strong/transactional | Measured write contention or genuine scale; then targeted eventual consistency, not global |
| Execution model | Synchronous request handling | Operation >~1–2s, or retry-ability need → background job; still not a distributed mesh |
| Caching | None | Profiling shows hot reads with measurable latency/cost impact (`../06-async-and-scale/caching.md`) |
| Messaging | Direct calls | Genuine decoupling need + outbox plan (`../06-async-and-scale/events-and-messaging.md`) |
| Horizontal scaling | Single instance + vertical headroom | Actual saturation evidence (`../06-async-and-scale/scalability.md`) |
| Abstraction | Concrete code, called directly | ≥2 real implementations or a proven seam (`architecture.md` §7) |
| Custom infrastructure code | Managed/established library | Non-standard requirement no library covers |

## 4. Complexity Budget Heuristic

Treat complexity as a budget, not an entitlement. A project can afford roughly **one** structurally sophisticated subsystem at a time (OR a message pipeline, OR a custom caching layer, OR a service extraction — not all three in the same quarter). Choose the one the evidence most demands, pay it down (stabilize, document, test), then consider the next. This is why "add it when needed" is genuinely different advice from "add it now to avoid later work": later work arrives with better information about what was actually needed.

## 5. Reversibility Ladder

Prefer reversible moves. Roughly ordered most→least reversible:

```
config change < code change < schema addition < cache introduction
< background job pipeline < API contract change < data model split
< service extraction < database engine change < event schema lock-in
```

Rules:

- Before an irreversible-ish move, write the migration path OUT as seriously as the path IN.
- Irreversible moves deserve the heaviest review and the most evidence (this is why sharding is last-resort — see `../06-async-and-scale/scalability.md`).
- Never bundle two irreversible moves in one release.

## 6. AI-Agent-Specific Discipline

Because the consumer of this guide is an AI agent writing code:

1. **Do not generate infrastructure you weren't asked for.** A request for "add an endpoint" does not authorize a repository pattern, a cache, a queue, a service layer, or a factory. Match the blast radius of the request.
2. **Do not mimic tutorial architecture.** Tutorials demonstrate maximum machinery for teaching; production minimizes it.
3. **Do not name things "Enterprise/Base/Abstract/Manager/HandlerFactory"** unless the abstraction passed §2's bar. Long class taxonomies are a cargo-cult signature.
4. **When extending existing code, copy its level of abstraction** — if the codebase is plain functions, your feature is plain functions. Consistency beats sophistication.
5. **If a requested feature genuinely needs more structure** (e.g., it's the first async work), that's an explicit decision: it is recorded in Planner Output first (DEC-n) and reflected in the software TODO, not invented while writing code. Implementer builds approved scope; a need the current plan does not cover → report BLOCKED with the triggering evidence. Keep the introduction minimal.

## 7. Complexity Acceptance Checklist

Introducing structural complexity is acceptable when:

- [ ] The measured problem is written down with numbers
- [ ] The simplest alternative was considered and rejected for a stated reason
- [ ] New failure modes are listed AND handled (timeouts, retries, observability)
- [ ] The rollback path is written and tested
- [ ] Someone/something will operate it (alerts exist — `../08-observability/observability.md`)
- [ ] Tests cover the new component's failure paths (`../09-testing/testing-strategy.md`)
- [ ] The ADR records assumption + revisit trigger (`../10-delivery/documentation-and-decisions.md`)

## 8. Failure Modes

- **Resume-driven / demo-driven architecture:** the stack diagram is impressive; the p50 latency is unchanged; the on-call rotation is drowning.
- **"Just in case" middleware:** every request passes through 6 cross-cutting layers, 5 of which no feature uses. Audit middleware stacks quarterly; delete what has no consumer.
- **Distributed single-server:** all the services deploy together on one box, share one database, and fail together — maximum complexity, zero isolation benefit.
- **Reflexive reversal:** teams burned by complexity sometimes ban ALL structure ("we'll keep it simple") and end up with a 15,000-line `handlers.ts`. Complexity control is evidence-based restraint, not absence of design. The rule is *justify*, not *prohibit*.
