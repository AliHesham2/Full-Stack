# Feature Flags — Gradual Delivery, Not Permanent Architecture

> **Part of:** Specialized Concerns — Backend Project Guide
> **Read when:** Considering a flag for a rollout, kill switch, or experiment — or drowning in flag debt.
> **Related:** `../01-foundation/complexity-control.md`, `../10-delivery/deployment.md` §2, `configuration.md` §3
> **Status:** Stable discipline · Current practice: release toggles live ~40 days max; kill switches are the permanent class

---

## 1. Position

A feature flag converts a deploy (code delivery) from a release (behavior change) — that decoupling is the entire value (`../10-delivery/deployment.md` §2): risky features activate gradually, broken features turn OFF without a rollback deploy, and migrations dual-run behind flags. The cost is real and compounding: every flag is a branch in logic that must be tested in both states, and every flag past its purpose is pure entropy. Flags are **transient scaffolding with one permanent exception class** — treat their lifecycle with the same discipline as code.

## 2. The Flag Taxonomy (Different Lifespans, Different Rules)

| Class | Purpose | Lifetime | Cleanup |
|---|---|---|---|
| **Release toggle** | Ship dark → gradual activation | days–weeks (~40-day ceiling in current practice) | remove with the rollout's completion |
| **Kill switch** | Instant-disable dangerous paths (integrations, expensive features) | permanent **by design** | none — but tested, documented, ownership-assigned |
| **Experiment/A-B** | Measure behavioral difference | experiment duration | remove with the conclusion |
| **Ops/permission toggle** | Tenant-tier entitlements, ops modes | permanent-as-data | belongs in DATA (entitlements), not flag systems, when it stops being "temporary" (§5) |

The fatal confusion: a release toggle that quietly became a permanent architecture knob nobody dares remove. Name the class at creation, with the expiry expectation attached.

## 3. Design Rules

- **Flags gate code paths, not truth:** the flag chooses between two implemented behaviors; it does not hold business state. When "flag value" becomes something the business rules read as data (tier limits per tenant), it's an entitlement record — move it to the database with the rest of the domain data (`configuration.md` §3).
- **Two clean paths, not scattered checks:** the flag branches ONCE at a seam (use-case selection or edge routing); flag checks sprinkled through 15 functions make both states untestable and removal archaeology.
- **Default-safe:** the OFF state must be the safe/old behavior; a flag-system outage evaluates to OFF for release toggles (fail toward the boring path), while kill switches default ON.
- **Evaluation at the edge, decision passed down:** resolve flags where request context lives (user/tenant/route), pass the resolved choice into use cases as a parameter — domain logic reading a global flag service is ambient-state smell (`../12-anti-patterns/anti-patterns.md` § global mutable state).
- **Observability per flag:** activation rate, error-rate split by flag state (the gradual rollout's whole safety mechanism — compare cohorts in the post-activation window, `../08-observability/observability.md` §4).

## 4. Lifecycle Discipline (Where Flags Go to Die)

1. **Register with metadata at creation:** owner, class (§2), created-date, expiry expectation, linked rollout ticket.
2. **Expiry alarms:** release toggles older than ~40 days page the owner — the debt is visible before it compounds.
3. **Removal is part of rollout completion:** the rollout isn't "done" at 100% activation; it's done when the flag and its dead branch are deleted. Cleanup PR created at flag creation (title, anyway) so the debt has a scheduled funeral.
4. **Both states tested while alive** (`../09-testing/testing-strategy.md`): CI exercises ON and OFF for every flagged path — an untested OFF state is a time bomb for the eventual removal PR.
5. **Dead-code removal on cleanup:** the OFF branch deletes WITH the flag; "keep the old code commented" is hoarding.

## 5. When NOT to Flag

- The change is safe, additive, and fully covered — flags-for-everything is ceremony that buries real risk in noise.
- The "flag" is actually config data (§3 rule) or an entitlement.
- You can't test both states — a flag you can't exercise both ways is a liability without the benefit.
- Experiment flags without a measurement design: "A/B test" with no metric plan is a permanent flag with extra steps.

## 6. Failure Modes

- **Flag landfill:** 200 flags, 12 owners (gone), both-states-forever testing burden, onboarding docs reading like a liturgy of magic constants. The §4 lifecycle is the counter.
- **The hidden permanent flag:** release toggle at 100% for 8 months; removal "risky" because nobody remembers what OFF does; the flag now owns the architecture.
- **Scattered evaluation:** flag checks in 15 files including a DB migration's logic branch — removal becomes a refactor-everything project; centralize at seams (§3).
- **Ambient flag reads in domain code:** business rules reading the global flag service → tests must configure flag state for every scenario, and the domain now depends on delivery infrastructure (§3 evaluation rule).
- **Fail-open release toggles:** flag-system outage turns every dark feature ON simultaneously — the safest direction is OFF for releases, ON for kill switches (§3 default-safe).
- **Untested OFF path:** rollout completes; cleanup PR flips the flag off; the OFF path has rotted for two months; production finds in minutes what CI never knew.
