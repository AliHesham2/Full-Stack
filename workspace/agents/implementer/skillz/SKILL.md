# Implementer skill — inspect, implement, verify, hand off

Read [backend-only scope](../../../docs/backend-guide/00-navigation/backend-only-scope.md) at startup. No UI design or frontend work is in scope.

Start at [../prompt.md](../prompt.md). This is the model-neutral implementation procedure, not a role assignment or permission grant. The [syntax policy](syntax-policy.md) is mandatory before code: personal/installed Skills and remembered or externally fetched syntax cannot replace the local technical references.

Approved Planner Output/software TODO and exact reviewed API/schema/security sources define current scope. No historical template or visual TODO is required. Start at [syntax map](../../../docs/syntax/GUIDE-MAP.md), then actual relevant indexes/pages with compatible versions.

Read and continue the one Planner-owned software TODO with actual contract gate evidence and exact complete attachments. No competing checklist or merged contract is created. Missing/conflicting definitions block dependent work.

## 1. Readiness and source route

Read assignment, approved requirements, exact software items and linked contracts/schema/security completely. Identify source IDs, outputs/exclusions and evidence. Do not execute an unapproved draft or invent missing business/auth/data behavior; raise gaps through Orchestrator to Planner/the owner.

Use [backend map](../../../docs/backend-guide/GUIDE-MAP.md) → [README](../../../docs/backend-guide/README.md) → [workflow](../../../docs/backend-guide/00-navigation/agent-workflow.md) → [decision index](../../../docs/backend-guide/00-navigation/decision-index.md). Read exact relevant sections, not headings. No frontend/design guide is required. Architecture comes from these guides; local syntax exclusively supports mechanics.

| Affected concern | Local reading route after navigation |
| --- | --- |
| App setup or file boundaries | Applicable guide `01-foundation/initialization.md`, `architecture.md`, `project-structure.md`; actual manifests/lockfile/runtime evidence |
| Endpoint and client-observable behavior | Approved API/status/error/serialization contracts; backend domain/API, validation, auth and reliability sections |
| HTTP and shared payloads | Backend domain/API guidance; actual approved contract schemas and external consumer compatibility |
| Authentication/authorization | Backend auth/security guidance; actual service/user/resource/tenant boundaries |
| Persistence or concurrent writes | Backend `05-data/` and relevant provider pages under `syntax/` |
| Reliability, dependencies or delivery files | Backend reliability, observability, delivery/configuration and actual scripts; no frontend guide |

## 2. Inspect the baseline and prepare the TODO

Locate the actual repository root, relevant apps and current changes. Inspect affected files, callers, contracts, shared helpers, configuration and existing tests. Search for equivalent functionality before adding it. Record pre-existing edits and failures without reverting them or assuming another operator has finished. If a shared file changes during your work, reread it and reconcile within your authority; stop overlapping edits when ownership is unresolved.

Confirm actual package/runtime versions and available scripts. An uninitialized starter has no runnable baseline: record that fact and follow initialization only if assigned, never fabricate a passing check or initialize an app merely to complete a report. Existing failures need their actual command/output and relevance; do not label a failure pre-existing unless observed before your change or supported by equivalent evidence.

Follow [project-map-and-handoff.md](project-map-and-handoff.md) to read, establish and maintain one canonical implementation map from actual code. Record its path and inspected scope in the TODO before code edits; keep proposed structure separate until implemented. Read that map on each new assignment and verify the affected branch rather than trusting stale navigation. Track task-created processes and temporary paths from the moment they are planned/created.

Validate and continue the Planner-prepared software TODO against [../templates/todo.md](../templates/todo.md); create one only if none was supplied within an approved assignment. Preserve approved scope and add actual baseline/execution information in that same file. Keep items blocked until exact contract review and Orchestrator disposition are linked. Map each `I-n` item to requirement/module/API/DB/SEC IDs, allowed files, guide and syntax sources, dependencies, failure paths, risk and planned checks. Record any unresolved contract/version/review gap. Keep the TODO updated as work proceeds, including the candidate, blockers and next authorized action; it is the resumable plan, not a second project status system.

Check the Planner-prepared API, database and security sections under [software-contracts.md](../../planner/skillz/software-contracts.md). Each executable item must link complete settled definitions: exact request locations/field validation, declared success/error schemas/statuses, DB fields/constraints/relationships/access and specific security enforcement/verification. Verify required source versions and attachments are accessible. No unresolved product, contract, schema or security setting is delegated to you. Stop affected work and report gaps through Orchestrator; do not invent values, endpoint shapes or policy, create a rival definition or silently repair Planner-owned scope.

## 3. Implement the smallest coherent change

Follow the approved backend construction order: settle data/contract/security prerequisites before consumers. No frontend code or fake endpoint success is created to simulate completeness.

Keep authorized server client → Express/Node.js → persistence/provider intact. Maintain schemas/types at their approved backend locations, with documented contract publication and external consumer compatibility. No runtime source imports from another project. Trace request/success/error/status and nullable/optional fields through actual backend handlers/services/persistence.

Use guide-owned layers and existing abstractions; each new helper, module or dependency needs a current requirement. Follow prescribed generation/setup commands only when assigned and syntax-supported, then inspect generated files and the complete diff. Match exact approved endpoint/auth/validation/success/error and data integrity behavior. Do not implement frontend routes, components or token systems. Do not replace unfinished integrations with hardcoded success, swallowed errors, no-op handlers or undisclosed mock data.

For backend changes, explicitly examine relevant trust boundaries: input validation, identity, resource/tenant ownership, secret/PII exposure, consistent error mapping and limits. For writes, examine applicable duplicate requests, partial failures, concurrent updates and transactional boundaries using the relevant guides. Observability must help diagnose failure without exposing secrets. Avoid adding retries, caching, queues or new layers without the required justification.

For approved contract/schema/dependency changes, list consumers and apply compatible changes within scope. For migrations, inspect current schema and existing data assumptions, ordering and constraints; explain recovery/forward-fix options and irreversible effects. Do not promise rollback unless supported. Preparing files does not authorize live execution. Do not install or upgrade packages, regenerate lockfiles, apply a migration or alter configuration outside the approved task merely to make a check pass.

## 4. Verify the candidate

Inspect the actual diff and final affected code against the original criteria, not just the intended change. Check source coverage under [syntax-policy.md](syntax-policy.md), module ownership, contract synchronization, security/failure paths, API/schema/documentation consistency, duplication and accidental edits. Every changed app must run its existing `npm run check` from that app's directory when available; also run any applicable prescribed build or focused check not already included. Confirm what a script actually executes before treating it as evidence. Never claim a build proves all behavior or a green command proves missing test coverage.

Use existing Tester-owned suites without editing them. Read [backend testing strategy](../../../docs/backend-guide/09-testing/testing-strategy.md). Missing tests go through Orchestrator to their actual Tester owner; no weakening assertions or discovery. Assigned backend integration/contract tests are valid verification, while automated full-system E2E needs separate explicit authorization. Owner manual API acceptance remains distinct.

Record exact command, working directory, candidate, relevant environment, exit/result and output evidence for each check. Redact secrets from retained logs. On missing tooling/access or external-service dependency, record `NOT RUN`/`BLOCKED` and the precise gap; do not install a replacement, fake a service or downgrade the check. A source-map gap is blocking under the syntax policy even when compilation passes.

## 5. Investigate and repair failures

For each reported failure, read the original requirement and full relevant error output. Reproduce it on the identified candidate with the permitted environment/data; retain actual versus expected evidence. If it cannot be reproduced, record conditions and uncertainty rather than dismissing the report. Distinguish a product defect, test defect, environment failure, missing requirement and source-policy conflict before deciding a correction.

Fix product code within approved scope and record the root cause. A test defect goes to its Tester through Orchestrator with exact evidence; do not modify assertions, snapshots, fixtures, test discovery or quality thresholds. A requirement/contract/schema change goes through change control. If the same issue recurs or a fix causes repeated regressions, stop speculative patching and report the unresolved cause and affected scope for coordinated investigation.

After a code fix, identify a new candidate, rerun the relevant reproducer and affected checks, plus `npm run check` in each changed app. Include shared consumers and neighboring behavior in regression scope. Under the [delivery cycle](../../orchestrator/skillz/delivery-cycle.md), Orchestrator reviews every Tester result and personally reruns the reproducer and affected/regression checks after your corrections. Do not send fixes back to Tester automatically. Your repair evidence does not close an issue; Orchestrator records its own candidate-specific verification or an explicit gap. Preserve previous issue IDs and evidence; no silent carryover of earlier PASS results to changed code.

## 6. Self-review and deliver

Reconcile the TODO with actual changed files, source decisions and evidence. Remove only your own task-generated debris where authorized; preserve unrelated work and open concerns. Verify that no requirement was silently dropped, no unsupported technical choice entered the diff, and no contract consumer was omitted. Any unresolved check, prerequisite or contract-review state remains visible in the Output.

Reconcile affected project-map paths/responsibilities and links with the final filesystem/code. Apply the [runnable handoff and owned-resource procedure](project-map-and-handoff.md): include real startup commands/configuration prerequisites and observed state, then stop/remove verified task-owned disposable resources or document an authorized continuation. Retained evidence and other operators' processes are preserved. Record failed or ambiguous cleanup explicitly.

Copy [../templates/output.md](../templates/output.md) to its assigned path. Use Changed / Verified / Checks / Remaining as the report structure. Identify the candidate by commit when present, or a manifest of changed file identities/hashes and evidence time when uncommitted; branch name alone is insufficient. Include checks after the final code edit and label later changes as requiring revalidation. Do not create a commit merely to obtain an identity.

Give Orchestrator/Tester the actual candidate, affected requirements/files/endpoints/schema, failure cases, exact commands/environments, API docs/examples and remaining coverage. No external client code or visual handoff is required. Before actually authorized bridge use read [bridge entry](../../../bridge.md). Optional Advisor is not a gate; do not dispatch or claim acceptance/release.
