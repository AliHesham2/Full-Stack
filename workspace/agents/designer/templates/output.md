# Designer Output — <task ID> / candidate <N>

> Copy to the assigned output path. Describe actual work and review evidence; do not carry approval or visual verification from another candidate.

This is a self-contained textual implementation handoff. Include complete approved specification details and only task-authorized source/decision references. Keep internal material outside the recipient's read scope in its own authorized records. Any supplied review extract must preserve the exact candidate, decisions and evidence limitations without broad source discovery or missing implementation details.

## Candidate and sources

| Field | Value |
| --- | --- |
| Task / operator / candidate | `<IDs and immutable version/artifact>` |
| Brief, TODO and approved requirements version | `<current Planner Output and role TODO paths/IDs; no old PRD required>` |
| Design-guide and frontend-guide sections used | `<paths/headings>` |
| Detailed design TODO / software TODO versions and design gate evidence | `<actual paths/versions; Tester report and Orchestrator disposition/correction checks, or NOT RUN>` |
| Artifact locations and actual viewer/tool | `<paths/links, or TEXT SPEC ONLY>` |
| Review time and reporting route | `<time/destination>` |

## Requirement and artifact coverage

| Requirement ID / user job | Design artifact and location | States/conditions covered | Review status and evidence | Gap |
| --- | --- | --- | --- | --- |
| `<UI/FL/DS/AC ID>` | `<screen/flow/component specification section>` | `<loading/empty/error/etc.; viewports/themes/RTL>` | `SPECIFIED / VISUALLY REVIEWED / UNVERIFIED` — `<evidence>` | `<NONE or exact gap>` |

## Approved decision traceability

| Requirement / owner decision ID | Confirmed textual decision | Resulting specification item and exact section | Disposition (`APPLIED`/`PROPOSED`/`BLOCKED`) and reason | Missing required product asset, state or decision |
| --- | --- | --- | --- | --- |
| `<actual ID or NONE>` | `<complete actionable decision>` | `<DT-n; canonical text specification>` | `<actual authority and evidence>` | `<specific gap or NONE>` |

## Flow and screen specifications

| Screen/flow ID | Entry and exit / navigation | Information hierarchy and exact copy/source | Actions, validation and data/API dependency | Loading, empty, no-results, error, denied, success and recovery | Phone/tablet/desktop and RTL/LTR |
| --- | --- | --- | --- | --- | --- |
| `<UI/FL ID>` | `<routes/steps>` | `<what user sees>` | `<approved API/BR IDs; never guessed>` | `<observable outcomes>` | `<specific changes>` |

## Component, token and accessibility handoff

| Component or `DS-n` | Existing kit reuse or justified addition | Anatomy/variants/states and content constraints | Semantic token references | Keyboard/focus/meaning, contrast, target size, motion | Implementer/Tester evidence needed |
| --- | --- | --- | --- | --- | --- |
| `<name/ID>` | `<reuse or PRD proposal>` | `<specific behavior>` | `<existing names or approved addition>` | `<testable requirements>` | `<observable check>` |

## Review and unresolved decisions

### UI/UX Pro Max evidence, if used

| Deliverable ID | Source commit, query and mode | Selected result and fit | Local-guide/PRD translation or rejection | Tool result/artifact |
| --- | --- | --- | --- | --- |
| `<D-01 or NONE>` | `<pinned commit; exact non-sensitive query>` | `<candidate pattern, not an approved rule>` | `<source section and decision>` | `<output path or NOT RUN reason>` |

### Vercel and Impeccable evidence, if used

| Deliverable ID | Source snapshot/current revision and method | Target and observed evidence | Decision and local-rule reconciliation | Follow-up owner |
| --- | --- | --- | --- | --- |
| `<D-01 or NONE>` | `<Vercel heading or Impeccable playbook/mode>` | `<file:line, rendered state or spec section>` | `<adopted, rejected or UNVERIFIED with reason>` | `<Designer / Implementer / Tester / Orchestrator>` |

### Design review

| Check | Result (`REVIEWED`/`UNVERIFIED`/`NOT APPLICABLE`) | Evidence or reason |
| --- | --- | --- |
| Whole flow and unhappy paths | `<result>` | `<artifact/steps>` |
| Design-guide checklist and relevant anti-patterns | `<result>` | `<sections and observations>` |
| Frontend kit/tokens and approved PRD consistency | `<result>` | `<source IDs and findings>` |
| Actual visual viewer review, if applicable | `<result>` | `<viewer, conditions, artifact>` |

| Question/change ID | Exact source conflict or missing decision | Impact and route/owner | Status |
| --- | --- | --- | --- |
| `<Q-01>` | `<quoted source locations>` | `<blocked behavior and recipient>` | `OPEN` |

## Implementation visual review — when assigned

Phase: `<NOT RUN / REVIEWED WITH MISMATCHES / REVIEWED WITHIN CHECKED SCOPE / UNVERIFIED>`.
Review round/time: `<actual values or NOT RUN>`; assignment: `<brief reference>`.
Approved baseline/version and matching intent: `<reference>`; implementation commit/build: `<identity>`.
Environment: `<browser/viewer, viewport CSS pixels, zoom, DPR if known, theme, locale/direction, fixtures/data, asset/font readiness>`.

| Case ID | Requirement/design location and expected criteria | Conditions and actual capture/region | Result (`MATCH`/`MISMATCH`/`UNVERIFIED`/`NOT APPLICABLE`) | Evidence/reason and issue ID |
| --- | --- | --- | --- | --- |
| `<VR-n if performed>` | `<approved baseline/spec location>` | `<environment/capture>` | `<observed result>` | `<path and observation or limitation>` |

| Visual issue ID | Case and requirement | Expected versus actual / user impact | Evidence locations | Requested correction and route/owner | Retest candidate/evidence and status |
| --- | --- | --- | --- | --- | --- |
| `<VD-n only for an actual mismatch>` | `<VR-n; UI/DS/AC ID>` | `<specific discrepancy>` | `<baseline and actual regions; source line only if read>` | `<observable correction; authorized recipient>` | `<OPEN / RECHECKED FIXED / REOPENED / UNVERIFIED; evidence>` |

Checked scope and excluded/unavailable conditions: `<cases/conditions or NOT RUN>`.
Previous round and affected/shared cases rechecked: `<links/IDs or NOT APPLICABLE>`.
Remaining mismatches / unverified conditions / recommendation: `<evidence-based result>`.

Keep earlier round evidence linked. An approved baseline change needs its decision/version reference; visual matching alone does not prove functional or accessibility acceptance. During design-only work, leave this section `NOT RUN` and do not populate fictional cases or findings.

## Handoff

- Implementer: `<verified design TODO/version, linked software items, precise buildable behavior and open dependencies>`
- Design Tester phase / Orchestrator correction verification: `<separate actual references; no implied retest>`
- Tester: `<AC IDs, observable states and accessibility/responsive conditions>`
- Remaining unverified: `<specific items or NONE>`
- Recommendation to Orchestrator: `<candidate ready for review / revision needed / blocked, with evidence>`

This Output records the design candidate and any explicitly performed visual review. It does not approve a PRD, certify functional behavior or accept a feature.
