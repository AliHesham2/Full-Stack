# Planner — software TODO and design dependency handoff

Use after input/guide discovery and engineering/security risk analysis in [SKILL.md](SKILL.md). The versioned Planner Output is canonical requirements/engineering scope, without historical PRD/ROADMAP dependencies. Follow the shared [delivery cycle](../../orchestrator/skillz/delivery-cycle.md).

## One downstream software TODO

Prepare one current file, normally `software.todo.md`, at the authorized task-record location using the [Implementer TODO template](../../implementer/templates/todo.md). This is separate from Planner's internal discovery TODO. No visual design TODO is required. Never write generated handoffs into owner's input folders or overwrite another assignment's records.

Specify task/requirement/AC IDs, canonical Output/version, explicit [Implementer starting prompt](../../implementer/prompt.md), goal/exclusions, proposed file paths/names/responsibilities, module dependencies, construction sequence, public interfaces, data/security constraints, local syntax support, allowed files/tools/environment and reporting/evidence paths. Record actual Roster assignment or UNASSIGNED; blank permissions are not unrestricted access. Proposed files remain proposed until actual code/project-map evidence exists.

Every item links source-backed consumed/produced boundaries, defined names/fields/types/signatures and their canonical versions. Avoid duplicate contract definitions, speculative code, arbitrary tiny tasks or minute quotas. Group coherent work into outcome slices with satisfied prerequisites; justify repeated shared-file churn. Carry negative/failure/authorization cases into planned verification, with Tester owning assigned tests.

## Mandatory software definitions before release to Implementer

Follow [software-contracts.md](software-contracts.md) and fill the software template's dedicated **API contracts**, **Database schema and access plan**, **Security plan** and **Planner contract-readiness gate** sections. Planner specifies the exact software behavior before execution: endpoint names/methods/paths, all input locations and field rules, every intended success output, applicable unified errors, schema/keys/relationships, access/transaction/migration behavior and concrete security enforcement/verification.

Keep one authoritative definition for each API/DB/SEC contract. New task-specific definitions live in the software TODO; existing approved schema/OpenAPI/policy sources remain authoritative through exact versioned mandatory attachments. Output inventories and explains those same definitions. A summary, generic reference, blank field or "Implementer decides" does not satisfy the gate. Planner resolves source/requirement gaps through Orchestrator and the actual decision owner; affected work stays BLOCKED until settled, and Orchestrator reviews the evidence before dispatch.

## Contract review before implementation

Orchestrator reviews the planning result, obtains actual owner decisions and gives Tester the versioned software TODO, Output and complete contracts/schema/security for CONTRACT SPECIFICATION review. Orchestrator individually reviews every result, returns confirmed issues to Planner and personally verifies its corrections without automatic Tester return. Link actual reviewed versions and Orchestrator disposition; unresolved items stay WAITING FOR CONTRACT REVIEW. There is one software TODO and no Designer or visual TODO.

## Readiness, execution and changes

Before recommending readiness, check backend-guide/source coverage, input preservation, decisions, interfaces, dependencies/cycles, role boundaries and actual permissions. Perform a recipient check on Output/software TODO without relying on chat history. Rank consequential missing answers, record self-check versus actual independent review and preserve source/decision references. A readiness label is not permission to dispatch.

Implementer validates and continues the same software TODO using its role template, adds execution evidence, and reads the exact reviewed contract/schema/security sources. Do not create a competing merged requirements checklist. Scope changes return to Planner through Orchestrator; contract/schema/security changes use the same canonical approval and impact procedure. Preserve previous versions, stable IDs and historical evidence, and use canonical CHG/DEC analysis for consequential changes.

After implementation, Tester verifies the actual code candidate once. Orchestrator independently reviews every result, formally routes product corrections to Implementer and personally verifies each correction plus affected/regression scope; no automatic return to Tester. Missing verification remains UNVERIFIED/BLOCKED rather than an invented pass. Owner manual API acceptance remain separate.

Before any authorized communication follow [bridge entry](../../../bridge.md). This planning procedure grants no dispatch, source/test editing, runtime access, automatic role appointment or live changes.
