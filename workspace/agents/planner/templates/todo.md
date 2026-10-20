# Planner TODO — <task ID> / planning candidate <version>

> Copy to the authorized task-record path before substantive planning. Record real scope and sources, not fictional approvals or completed checks. Paths below are from project root unless explicitly stated otherwise.

## Assignment and prerequisites

| Field | Value |
| --- | --- |
| Task / operator / Roster assignment and brief | `<actual references>` |
| Owner goal / confirmed scope / exclusions | `<source references>` |
| Current input scope / owner message references | `<paths and sources; no inherited old PRD scope>` |
| Relevant current code/contracts/design inputs | `<actual project evidence or NOT INITIALIZED>` |
| Allowed writes/tools/environment and reporting route | `<actual scope>` |
| TODO / Output / proposed document locations | `<authorized paths>` |
| Downstream software TODO | `<one authorized path; Planner-owned scope; Implementer execution>` |
| Designer-owned design TODO dependency | `<actual reference when returned; not authored by Planner>` |

## Input inventory and understanding

| File/folder or owner message | Type and module intent | Material actually read / source sections | Dependencies, contradiction or missing content | Fact / request / example / proposal distinction |
| --- | --- | --- | --- | --- |
| `<actual source>` | `<HTML / Markdown / text / module folder>` | `<inspection coverage>` | `<precise gap or NONE>` | `<classification and impact>` |

## Understanding playback and anti-goals

| Module / source IDs | Concise user goal, reason and intended behavior | Owner-confirmed facts versus interpretation | Non-goals / unacceptable outcomes and source or proposed status | Consequential question and resolution route |
| --- | --- | --- | --- | --- |
| `<actual module/source>` | `<what the user is trying to accomplish>` | `<classification>` | `<evidence; do not invent constraints>` | `<Q-n or NONE>` |

## Mandatory guide route and coverage

| Guide | Map and actual files/sections read | Planning constraint / affected requirement or decision | Excluded branches and scope-based reason | Missing source or conflict |
| --- | --- | --- | --- | --- |
| Frontend | `<start workspace/docs/frontend-guide/GUIDE-MAP.md; then actual entry/routed files>` | `<consequence, not just title>` | `<branch-specific reasons>` | `<exact gap or NONE>` |
| Backend | `<start workspace/docs/backend-guide/GUIDE-MAP.md; then actual entry/routed files>` | `<consequence, not just title>` | `<branch-specific reasons>` | `<exact gap or NONE>` |

Both guides are mandatory. A map-only read does not complete discovery. Use the entry route in `workspace/agents/planner/skillz/SKILL.md`.

## Supabase/Postgres source selection — when applicable

Use [planning adapter](../skillz/supabase-planning.md) and [complete catalog](../skillz/supabase-catalog.md) before affected decisions. Read full selected files, not these summaries. Unrelated tasks record a scope-based exclusion; no provider setup or DB is implied. Planner plans and reviews, never executes the upstream operations.

| Actual affected product/DB/API/SEC scope | Selected Skill/rule file and sections / pinned source version | Planned consequence, risk or justified exclusion | Approved/proposed actual engine/SDK/driver/workflow and local syntax support | Planned verification owner / environment and blocker |
| --- | --- | --- | --- | --- |
| `<actual scope or NOT APPLICABLE with reason>` | `<complete selected entry/rules; do not use a folder alone>` | `<source-backed decision, access/query/index/migration issue>` | `<exact sources/versions or precise gap; no silent upgrade>` | `<allowed/denied, tenant, concurrency, recovery cases as relevant; NOT RUN until actual evidence>` |

## Planning work

| Item ID | Owner requirement/source | Planned artifact or analysis | Needed guide/syntax/design sources | Dependencies and decision owners | Planned review evidence | Status |
| --- | --- | --- | --- | --- | --- | --- |
| `<P-n>` | `<confirmed requirement or explicit proposal>` | `<discovery, risk analysis, proposal or role TODO>` | `<paths/sections>` | `<actual prerequisites>` | `<coverage/contract/risk/dependency check>` | `TODO` |

Use `TODO`, `IN PROGRESS`, `BLOCKED`, `REPORTED`. `REPORTED` is not approval. Stable product IDs are defined once in the new Planner Output, separately from this internal checklist. Complete input/guide analysis and report engineering/security proposals and risks before finalizing downstream TODOs.

## Software TODO preparation and design dependency

| Software TODO / Implementer template | Planner Output/version and requirement IDs | Proposed paths, responsibilities and construction order | Handoff state / blocking decisions | Actual design TODO/version and gate evidence |
| --- | --- | --- | --- | --- |
| `<single path; existing Implementer template>` | `<source version/IDs>` | `<software plan, no production bodies>` | `<WAITING FOR DESIGN where applicable>` | `<NOT RECEIVED or actual Designer-owned artifact and review>` |

Designer creates its internal TODO and detailed design handoff from Orchestrator's brief. Planner plans software scope, interfaces and dependencies; do not prepare a Designer checklist or a second downstream plan.

## Preservation, shared constraints and outcome plan

| Planned review | Current sources/scope | Required evidence in canonical Output | Blocking gap / next action |
| --- | --- | --- | --- |
| Decision-bearing source claims survive distillation | `<input modules/sections and changed versions>` | `<source-to-destination map; absorbed versus required downstream material; justified exclusions>` | `<actual gap or NONE>` |
| Shared architectural invariants remain consistent | `<existing guide rules/owner decisions and affected modules>` | `<DEC-n source/status, binds/prevents/rule; deferred-choice revisit trigger>` | `<unresolved proposal or NONE>` |
| Incremental outcome slices use satisfied prerequisites | `<SL-n / requirement and AC IDs>` | `<useful outcome; minimal UI/API/data; enabled foundation work; shared-file churn rationale>` | `<actual missing predecessor>` |
| Readiness rests on actual decisions/evidence | `<recipient/slice and needed inputs>` | `<source/version and supported/pending/conflicting status; precise blocked scope>` | `<actual decision owner>` |

Consequential change, when one occurs: `<CHG-n; trigger/evidence; current/completed/remaining scope; options; required decision; TODO versions to reconcile>`.
Do not populate fictional changes or architecture decisions. Canonical definitions and actual review results belong in Planner Output; this is the plan for obtaining them.

## Software contract preparation — mandatory when applicable

| Area | Required deliverable in the software TODO or exact mandatory authoritative attachment | Planned validation | State / missing decision owner |
| --- | --- | --- | --- |
| APIs | Names/methods/paths, path/query/header/cookie/body field rules, auth, explicit success bodies/headers, applicable shared-error outcomes and examples | Compare every outcome/example with its canonical schema and consumer/AC; separate Next.js and Express boundaries | `<actual state>` |
| DB | Exact tables/columns/types, keys/constraints/index rationale, relationships/ER diagram, access/transactions/lifecycle and migration/recovery | Compare API-to-DB behavior and actual engine/schema/version; no invented existing DB | `<actual state>` |
| Security | Separate SEC-linked plan with trust boundaries, concrete controls/enforcement locations, approved policy values and negative verification | Apply relevant guide threat checklist; no vague "make secure" or Implementer-chosen policy | `<actual state>` |
| Per-item readiness | Every software item links settled definitions/decisions and actual prerequisites | Missing details remain BLOCKED; Orchestrator reviews before dispatch | `<actual state>` |

Follow `workspace/agents/planner/skillz/software-contracts.md` (project-root path). Use justified NOT APPLICABLE without manufacturing infrastructure. Planner resolves requirements/policy with the actual decision owner through Orchestrator; Implementer executes and checks, not completes missing contract/schema/security design.

## Planned recipient and interface checks

| Check item | Target Output/TODO version | Expected review evidence | Responsible route / status |
| --- | --- | --- | --- |
| Recipient can work without chat/author access | `<versions>` | `<consequential questions ranked and disposition; self-check or actual independent review identified>` | `<authorized route; actual state>` |
| Consumed/produced boundaries match | `<task/interface IDs and versions>` | `<producer-consumer names/shapes and unresolved prerequisites>` | `<actual state>` |
| Task size and plan proportion | `<role task IDs>` | `<independently reviewable deliverables; no vague instructions or unnecessary code bodies>` | `<actual state>` |
| Uncovered failure cases prioritized | `<requirement/risk IDs>` | `<owner task and planned verification, or reviewed none with scope>` | `<actual state>` |

## Open questions, assumptions and risks

| ID | Fact / assumption / proposal / question | Source and impact | Required decision or evidence / owner | Blocking scope and state |
| --- | --- | --- | --- | --- |
| `<actual ID>` | `<clearly labeled statement>` | `<source and affected items>` | `<authorized route>` | `<precise dependency; OPEN / RESOLVED with reference>` |

## Resume point

- Last update / current source and artifact versions: `<actual values>`
- Completed items and evidence: `<IDs/paths or NONE>`
- Remaining items / blockers: `<IDs and scope>`
- Next authorized action: `<one step>`
- Output / prior review references: `<existing records>`
