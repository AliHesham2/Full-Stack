# Planner Output — <task ID> / planning candidate <version>

> This is the canonical new requirements/engineering proposal for the current input-driven task. Do not inherit old PRDs, ROADMAPs, templates or approvals. Prepare the software TODO from this version; Designer later owns its detailed design TODO rather than defining competing specifications. Paths are from project root unless stated otherwise.

## Candidate and authority

| Field | Value |
| --- | --- |
| Task / operator / Roster and brief | `<actual references>` |
| Current owner request / assigned input inventory | `<source paths/messages and inspected coverage>` |
| This Output path/version/time and change history | `<actual identity; affected IDs on revision>` |
| Scope/decision approval state and owner reference | `<PROPOSED / APPROVED FOR NAMED SCOPE / NEEDS REVISION; actual reference or NONE>` |
| Internal Planner TODO / downstream software TODO | `<two distinct paths; one current software plan>` |
| Designer-owned design TODO / gate evidence | `<actual path/version and review references when returned; NOT RECEIVED until then>` |
| Allowed artifacts and reporting route | `<actual authority>` |

## Changed — understanding and proposed requirements

### Input interpretation and module boundaries

| Source file/section/message | Requirement/module ID | Confirmed intent versus assumption/proposal | Module responsibilities, ownership and exclusions | Contradiction or question |
| --- | --- | --- | --- | --- |
| `<actual input>` | `<RQ-n / MOD-n>` | `<clearly classified meaning>` | `<cohesive scope>` | `<Q-n or NONE>` |

### Input preservation after distillation

| Source file/section/message and decision-bearing claim | Destination requirement/decision or required artifact | Disposition (`ABSORBED` / `REQUIRED DOWNSTREAM` / `EXCLUDED WITH REASON` / `UNRESOLVED`) | Recipient TODO link when still required | Exact-value/qualification check and gap |
| --- | --- | --- | --- | --- |
| `<actual claim; source version>` | `<canonical ID/location>` | `<evidence-based disposition>` | `<role/item/path/section or NOT APPLICABLE>` | `<preserved constraint, contradiction or missing detail>` |

Reconciliation scope/version: `<actual reviewed inputs and changed artifacts>`.
Do not label material fully absorbed if a recipient still needs it to make the correct decision. Preserve originals and record exclusions rather than silently dropping inconvenient claims.

### Module understanding, non-goals and anti-goals

| Module / source IDs | Plain-language understanding of intent and why | Confirmed versus interpreted | Non-goals / unacceptable outcomes | Source or proposal/decision reference / unresolved question |
| --- | --- | --- | --- | --- |
| `<actual IDs>` | `<short playback, not a copied input dump>` | `<actual distinction>` | `<source-backed scope or proposed status>` | `<evidence; do not invent owner agreement>` |

### User behavior and acceptance

| Requirement / flow / UI / business-rule IDs | Actor, permission and trigger | Preconditions, success and state transitions | Alternate/failure/denial and validation behavior | Acceptance ID and observable Given/When/Then |
| --- | --- | --- | --- | --- |
| `<actual IDs>` | `<source-backed behavior>` | `<defined outcome>` | `<applicable conditions>` | `<AC-n; expected evidence>` |

### API and data proposal

Requirements and decision authority remain in this Output. Complete supplied API extractions and frontend-data/SEC implementation definitions must be organized by Planner through the software TODO using [software-contracts.md](../skillz/software-contracts.md), not left for Implementer to invent. New task-specific definitions live there; existing approved contract/frontend-state/policy sources remain authoritative via exact mandatory attachments. Do not keep competing independently editable definitions here.

| API/frontend-data/SEC ID and name | Purpose and requirement/AC IDs | Existing/new/changed and approval state | Exact complete software TODO section or required authoritative attachment/version | Owning module / consumers / unresolved gate |
| --- | --- | --- | --- | --- |
| `<actual ID or NOT APPLICABLE with reason>` | `<scope and traceability>` | `<proposal versus approval and source>` | `<complete contract/frontend-state/security definition, not generic folder>` | `<affected tasks and precise blockers>` |

Contract coverage: `<all supplied request locations/field rules, exact success/failure schemas, approved frontend error normalization, DTO/state/cache mappings and dedicated security plan; owner-source versions and missing decisions>`.
Do not invent APIs or tables for scope that does not require them. Narrative and the final owner explanation derive from these same definitions and must be reconciled on change; examples are not a second contract authority.

## Shared architectural constraints and incremental outcomes

| Decision ID | Source/authorization and status | Binds: modules/roles/slices | Prevents: incompatible choices or failure | Enforceable rule or exact canonical rule reference | Deferred reason and revisit trigger, if applicable |
| --- | --- | --- | --- | --- | --- |
| `<DEC-n for an actual shared decision>` | `<existing guide/owner decision; APPROVED / PROPOSED / DEFERRED / CONFLICT>` | `<actual scope>` | `<specific divergence>` | `<reference; no duplicate architecture>` | `<reason/observable trigger or NOT APPLICABLE>` |

Proposals are not binding. Deferred choices cannot leave the next executable work unsafe or unspecified. Actual code/project-map owns evolving implementation inventory.

| Slice ID | Useful user outcome and requirement/AC IDs | Minimal UI/API/data and role task references | Existing/earlier prerequisites and blocked work | Necessary technical enablers | Shared-file churn / consolidation decision |
| --- | --- | --- | --- | --- | --- |
| `<SL-n>` | `<observable result>` | `<software items and linked Designer-owned items when available>` | `<source/earlier completion evidence or explicit blocker>` | `<foundation task and outcome enabled, or NONE>` | `<cohesive grouping or justified separation>` |

No hidden forward dependencies, all-tables-up-front milestone or extra TODO per slice. Necessary technical tasks remain allowed and must be tied to the outcome they support.

## Engineering proposals and risks — before downstream TODOs

| Proposal/risk/question ID and category | Evidence/source and observed-versus-potential classification | Affected IDs, impact and likelihood rationale | Options/tradeoffs and recommended mitigation | Required verification / decision owner / blocking scope | State and actual decision reference |
| --- | --- | --- | --- | --- | --- |
| `<PROP-n / RISK-n / Q-n; engineering / coding / integration / data / security>` | `<actual source or explicit uncertainty>` | `<concrete consequence>` | `<bounded option, not silently approved scope>` | `<needed evidence/authorization>` | `<PROPOSED / OPEN / RESOLVED; reference>` |

Security review coverage: `<relevant trust boundaries, auth/ownership, input handling, secret exposure and resource abuse; justified exclusions>`.
Recommendations are proposals; planning risks are not confirmed vulnerabilities or proof of a completed security audit.

## Verified — mandatory guide and traceability review

| Guide | Map and actual entry/routed files/sections read | Concrete planning constraint and affected IDs | Excluded branches with scope-based reason | Gap or conflict |
| --- | --- | --- | --- | --- |
| Frontend | `<actual sources>` | `<consequence>` | `<reason>` | `<NONE or exact gap>` |
| Backend | `<actual sources>` | `<consequence>` | `<reason>` | `<NONE or exact gap>` |

| Owner requirement/source | Canonical IDs in this Output | Required design input or actual Designer item | Software TODO item | Planned Tester evidence / missing coverage |
| --- | --- | --- | --- | --- |
| `<actual source>` | `<RQ/MOD/API/DB/AC IDs>` | `<item or justified NOT APPLICABLE>` | `<item or blocker>` | `<observable checks and environment needed>` |

Dependency/conflict plan: `<hard prerequisites, contract dependencies, shared-file/resource conflicts, checked cycles and proposed role ordering>`.

## Software TODO and downstream design dependencies

| Current software TODO / starting Implementer prompt | Scope/requirements version and assignment state | Readiness state, decisions and blockers | Expected implementation Output |
| --- | --- | --- | --- |
| `<one current file; Implementer prompt>` | `<IDs/version; actual assignment or UNASSIGNED>` | `<DRAFT / AWAITING DECISION / WAITING FOR DESIGN / READY FOR ORCHESTRATOR REVIEW>` | `<authorized path>` |

Designer input: `<requirement IDs, known software/API/data constraints and raw source references; not a Planner-authored Designer checklist>`.
Designer-owned design TODO/version and design gate evidence: `<actual references or NOT RECEIVED>`.
Software conflicts requiring Planner revision: `<exact impact and decision, or NONE; routine design linking is not a mandatory replan>`.
Do not release design-dependent code from anticipated artifacts. Orchestrator owns gate review/routing; readiness grants no dispatch authority.

## Recipient check and focused verification gaps

Review mode and actor: `<SELF-CHECK / ACTUAL INDEPENDENT REVIEW; actual operator or NOT RUN>`.
Reviewed Output and software TODO versions, plus actual design references if available: `<exact versions>`.

| Question / issue | Why a wrong guess matters / priority | Existing answer/source or required decision | Affected task/requirement | Disposition and evidence |
| --- | --- | --- | --- | --- |
| `<actual question or NO CONSEQUENTIAL GAPS FOUND WITHIN CHECKED SCOPE>` | `<concrete consequence>` | `<actual authority; minor call only within delegation>` | `<IDs>` | `<resolved / blocked / unverified; exact evidence>` |

| Priority / case ID | Plausible uncovered input or failure condition | Expected observable behavior and source/risk | Owning role/task and planned verification | Remaining decision or disposition |
| --- | --- | --- | --- | --- |
| `<actual case; no fixed quota>` | `<specific condition>` | `<source-backed outcome or open question>` | `<task ID and verification owner; Tester writes tests>` | `<linked coverage, blocked, or precise gap>` |

Cross-task boundary check: `<canonical interface IDs, matching producer/consumer names, fields/types/signatures and versions; gaps>`.
Task-sizing/proportion check: `<reviewable deliverables; supporting setup folded in; vague instructions and unnecessary code removed>`.
If the focus list is empty, record reviewed scope and why no additional gaps were found. A self-check is not independent review. Recheck affected interfaces/coverage after changes and link the new evidence.

## Decision-based readiness

| Recipient / slice / task | Needed decision/input | Actual source/version and status (`SUPPORTED` / `PENDING` / `CONFLICTING` / `NOT APPLICABLE`) | Exact work blocked / independent work still valid | Readiness recommendation and required resolution |
| --- | --- | --- | --- | --- |
| `<role / SL-n / item>` | `<real prerequisite, not a prescribed document name>` | `<recorded evidence and reason>` | `<affected scope>` | `<existing handoff state; decision owner>` |

A missing document type matters only when a needed decision is absent. Security, contract, local syntax and actual design prerequisites remain mandatory where applicable. Readiness does not authorize dispatch, waive a risk or establish runtime behavior.

## Checks and remaining decisions

| Planning check | Method and candidate/version | Result (`REVIEWED` / `FAILED` / `NOT RUN` / `BLOCKED` / `NOT APPLICABLE`) | Evidence and limitation |
| --- | --- | --- | --- |
| Assigned inputs and the frontend guide and owner-supplied API contracts understood | `<coverage and specific consequences>` | `<result>` | `<actual evidence>` |
| Requirement/task sources in both directions | `<IDs and the software TODO; actual design items if available>` | `<result>` | `<missing/orphaned items>` |
| Engineering/security risks surfaced before handoff | `<risk/decision records>` | `<result>` | `<blocked scope>` |
| Modules, contracts, data and syntax feasibility | `<source/consumer inspection>` | `<result>` | `<gap>` |
| Complete supplied API success/error, frontend DTO/state and dedicated security handoff | `<software TODO readiness gate; API/frontend-data/SEC source versions and per-item evidence>` | `<result>` | `<missing choices block dependent implementation>` |
| Dependencies, design readiness and file ownership | `<graph and actual design inputs>` | `<result>` | `<conflicts>` |
| Recipient questions, exact interfaces and task proportion | `<reviewed versions; actual review mode; boundary comparison>` | `<result>` | `<gaps and dispositions>` |
| Prioritized uncovered failure cases | `<owning task and verification links; reviewed scope>` | `<result>` | `<remaining cases/decisions>` |
| Source preservation, inherited constraints and outcome slices | `<source claim destinations; DEC/SL IDs; prerequisites and repeated-file review>` | `<result>` | `<omissions/conflicts>` |
| Decision-based readiness and consequential change impacts | `<needed decisions/source evidence; CHG-n when applicable>` | `<result>` | `<blocked work and unresolved options>` |
| Owner explanation matches canonical definitions | `<IDs/examples/relationships>` | `<result>` | `<gap>` |

Remaining: `<precise questions, decision owners, blockers and next authorized action; or NONE FOUND WITHIN REVIEWED SCOPE>`.
Review does not establish implemented behavior, security compliance, message delivery or feature acceptance. Preserve Planner's resume point and previous version evidence; only the authorized Orchestrator route schedules recipients.

## Consequential replanning — only when triggered

| Change ID / trigger and evidence | Current/completed/remaining slices and requirement/task IDs affected | Invalidated constraints, dependencies, approvals or evidence | Alternatives and tradeoffs | Authorized decision and updated artifact/TODO versions | Work blocked / independent valid work |
| --- | --- | --- | --- | --- | --- |
| `<CHG-n for an actual change, otherwise NOT APPLICABLE>` | `<precise impact; historical evidence preserved>` | `<what no longer holds>` | `<local fix / resequence / scope revision / supported recovery; risk and uncertainty, no invented dates>` | `<reference or AWAITING DECISION>` | `<bounded disposition>` |

Keep earlier versions and retired IDs linked; do not erase completed evidence or execute recovery through this planning record. Reconcile software tasks and dependencies as the decision permits; route affected design changes to Designer, which owns its design TODO.

## Owner explanation — simplified Arabic, last section

`<Explain the current approved/proposed scope plainly, with status made clear. For every specified API show its purpose, request, success and declared-failure examples, referencing canonical IDs. Explain frontend DTO/state relationships and the Browser/Next.js/external API flow. State database schema/backend implementation is outside scope; do not invent tables or an ER diagram. Keep identifiers, JSON and diagram source exact, use synthetic values, and apply project Arabic Markdown direction rules to Arabic prose. Update this section when canonical definitions change.>`
