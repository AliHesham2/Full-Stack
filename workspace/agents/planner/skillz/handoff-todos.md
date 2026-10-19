# Planner — software TODO and design dependency handoff

Use after input/guide discovery and engineering/security risk analysis in [SKILL.md](SKILL.md). The versioned Planner Output is canonical requirements/engineering scope, without historical PRD/ROADMAP dependencies. Follow the shared [delivery cycle](../../orchestrator/skillz/delivery-cycle.md).

## One downstream software TODO

Prepare one current file, normally `software.todo.md`, at the authorized task-record location using the [Implementer TODO template](../../implementer/templates/todo.md). This is separate from Planner's internal discovery TODO. Do not prepare Designer's internal work TODO or detailed design TODO. Never write generated handoffs into owner's input folders or overwrite another assignment's records.

Specify task/requirement/AC IDs, canonical Output/version, explicit [Implementer starting prompt](../../implementer/prompt.md), goal/exclusions, proposed file paths/names/responsibilities, module dependencies, construction sequence, public interfaces, data/security constraints, local syntax support, allowed files/tools/environment and reporting/evidence paths. Record actual Roster assignment or UNASSIGNED; blank permissions are not unrestricted access. Proposed files remain proposed until actual code/project-map evidence exists.

Every item links source-backed consumed/produced boundaries, defined names/fields/types/signatures and their canonical versions. Avoid duplicate contract definitions, speculative code, arbitrary tiny tasks or minute quotas. Group coherent work into outcome slices with satisfied prerequisites; justify repeated shared-file churn. Carry negative/failure/authorization cases into planned verification, with Tester owning assigned tests.

## Mandatory software definitions before release to Implementer

Follow [software-contracts.md](software-contracts.md) and fill the template's supplied API, frontend error/DTO/state, Security and readiness sections. The owner supplies wire definitions; Planner inventories and checks exact requests, successes and failures, then prepares approved frontend mappings and file responsibilities. No backend/schema/migration work is required. Missing external policy/definitions returns to the owner, not Implementer.

Maintain one authoritative source per definition: read-only external contract extraction plus mandatory exact original versions, and separately approved frontend decisions in the software TODO. Output inventories and explains these sources. Blank fields, generic links or "Implementer decides" fail the gate. Orchestrator reviews readiness before dispatch.

## Designer owns the visual implementation TODO

Orchestrator reviews the planning result and briefs Designer from it. Designer creates its own internal TODO and a separate [design TODO for Implementer](../../designer/templates/design-todo.md), with detailed geometry, responsive constraints, typography, states, content, components and accessibility behavior. Planner supplies software constraints and source references, not Designer's screen-level execution checklist or invented visual values.

Tester reviews the actual design specification before implementation. Orchestrator independently reviews every result; Designer corrects confirmed issues and Orchestrator personally verifies corrections, without another Tester round. The actual design TODO/version and gate evidence become Implementer's required inputs alongside the software TODO. UI-dependent software items remain WAITING FOR DESIGN until that gate; record UI-free exceptions with rationale.

Routine handoff does not require Planner to reproduce or absorb all design details. Orchestrator links the actual verified design and dependencies; Planner revises the software plan only for real software scope/interface conflicts or authorized changes. Designer updates its own document as affected. No role silently rewrites the other's canonical content.

## Readiness, execution and changes

Before recommending readiness, check frontend-guide/owner-contract source coverage, input preservation, decisions, interfaces, dependencies/cycles, role boundaries and actual permissions. Perform a recipient check on Output/software TODO without relying on chat history. Rank consequential missing answers, record self-check versus actual independent review and preserve source/decision references. A readiness label is not permission to dispatch.

Implementer validates and continues the same software TODO using its role template, adds execution evidence, and reads both software and verified design TODOs. Do not create a competing merged requirements checklist. Scope changes return to Planner through Orchestrator; visual specification changes return to Designer. Preserve previous versions, stable IDs and historical evidence, and use canonical CHG/DEC analysis for consequential changes.

After implementation, Tester verifies the actual code candidate once. Orchestrator independently reviews every result, formally routes product corrections to Implementer and personally verifies each correction plus affected/regression scope; no automatic return to Tester. Missing verification remains UNVERIFIED/BLOCKED rather than an invented pass. Owner manual E2E and acceptance remain separate.

Before any authorized communication follow [bridge entry](../../../bridge.md). This planning procedure grants no dispatch, source/test editing, runtime access, automatic role appointment or live changes.
