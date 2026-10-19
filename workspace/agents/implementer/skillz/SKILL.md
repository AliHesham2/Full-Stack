# Implementer skill — inspect, implement, verify, hand off

Start at [../prompt.md](../prompt.md). This is the model-neutral implementation procedure, not a role assignment or permission grant. The [syntax policy](syntax-policy.md) is mandatory before code: personal/installed Skills and remembered or externally fetched syntax cannot replace the local technical references.

In the new input-driven route, approved Planner Output/version and the software TODO supply the requirements/task baseline wherever this procedure refers to PRD/ROADMAP; Designer's detailed design TODO supplies visual implementation requirements. No deleted document/template is required. Start technical reading at [syntax GUIDE-MAP](../../../docs/syntax/GUIDE-MAP.md), then actual indexes/pages; record the selected sources and compatible versions.

Read the Planner-owned software TODO and the Designer-owned detailed design TODO/version with actual design gate evidence. Continue the software TODO as the execution checklist; reference design items without creating a competing merged specification. Missing or conflicting dependencies block affected work.

## 1. Readiness and source route

Read the assignment and approved PRD/ROADMAP step completely, then the relevant contracts and Designer Output. Identify source IDs, deliverables, exclusions and the exact evidence required. Do not execute an unapproved draft or start the next roadmap step without an assignment. Missing product behavior goes to Planner/PRD owner through Orchestrator; missing UI behavior goes to Designer through the same route. Continue independent, fully specified work where possible.

Use the [frontend guide](../../../docs/frontend-guide/README.md) → [frontend workflow](../../../docs/frontend-guide/00-navigation/agent-workflow.md) for frontend changes. Use the [backend guide](../../../docs/backend-guide/README.md) → [backend workflow](../../../docs/backend-guide/00-navigation/agent-workflow.md) → [decision index](../../../docs/backend-guide/00-navigation/decision-index.md) for backend changes. Cross-app work needs both. Read task-relevant sections in full, not just their headings. These guides own architecture and engineering constraints; resolve syntax gaps through the mandatory policy rather than fetching an alternate implementation source.

| Affected concern | Local reading route after navigation |
| --- | --- |
| App setup or file boundaries | Applicable guide `01-foundation/initialization.md`, `architecture.md`, `project-structure.md`; actual manifests/lockfile/runtime evidence |
| UI and user-visible states | Designer Output; frontend `02-ui/`, routing, `03-data/`, security, accessibility and resilience sections |
| HTTP and shared payloads | Frontend `03-data/backend-integration.md`; backend `02-domain-and-api/`; actual contract schemas and consumers |
| Authentication/authorization | Backend `03-auth/`, frontend/backend security guidance and actual session boundary |
| Persistence or concurrent writes | Backend `05-data/` and relevant provider pages under `syntax/` |
| Reliability, dependencies or delivery files | Routed backend `07-reliability/`, `08-observability/`, `10-delivery/`; frontend `04-quality/` and `05-delivery/` |

## 2. Inspect the baseline and prepare the TODO

Locate the actual repository root, relevant apps and current changes. Inspect affected files, callers, contracts, shared helpers, configuration and existing tests. Search for equivalent functionality before adding it. Record pre-existing edits and failures without reverting them or assuming another operator has finished. If a shared file changes during your work, reread it and reconcile within your authority; stop overlapping edits when ownership is unresolved.

Confirm actual package/runtime versions and available scripts. An uninitialized starter has no runnable baseline: record that fact and follow initialization only if assigned, never fabricate a passing check or initialize an app merely to complete a report. Existing failures need their actual command/output and relevance; do not label a failure pre-existing unless observed before your change or supported by equivalent evidence.

Follow [project-map-and-handoff.md](project-map-and-handoff.md) to read, establish and maintain one canonical implementation map from actual code. Record its path and inspected scope in the TODO before code edits; keep proposed structure separate until implemented. Read that map on each new assignment and verify the affected branch rather than trusting stale navigation. Track task-created processes and temporary paths from the moment they are planned/created.

Validate and continue the Planner-prepared software TODO against [../templates/todo.md](../templates/todo.md); create one only if none was supplied within an approved assignment. Preserve approved scope and add actual baseline/execution information in that same file. Keep design-dependent items blocked until the actual design TODO/version and design gate evidence are linked. Map each `I-n` item to requirement/module/design IDs, allowed files, guide and syntax sources, dependencies, failure paths, risk and planned checks. Record any unresolved contract/version/design gap. Keep the TODO updated as work proceeds, including the candidate, blockers and next authorized action; it is the resumable plan, not a second project status system.

## 3. Implement the smallest coherent change

Follow approved dependencies. For a cross-app feature this commonly means data constraints and backend contract/behavior before frontend integration, but the approved roadmap determines ordering. A frontend-only change uses an existing verified contract; it does not create a backend or invent payloads to simulate completeness.

Keep Browser → Next.js → Express/Node.js → database intact. Define/maintain contract schemas at their approved backend location and synchronize the actual inferred types by the project's documented route, without runtime source imports between apps or independently guessed frontend shapes. Trace success/error/status and nullable/optional fields through all affected consumers. If the synchronization mechanism is missing, report that prerequisite before dependent code.

Use guide-owned layers and existing abstractions; each new helper, module or dependency needs a current requirement. Follow prescribed generation/setup commands only when assigned and syntax-supported, then inspect generated files and the complete diff. Do not replace the portable component kit or generate a second token system. Match the approved design's real content, viewports, themes, RTL, focus and loading/empty/error/success states. Do not replace unfinished integrations with hardcoded success, swallowed errors, no-op handlers or undisclosed mock data.

For backend changes, explicitly examine relevant trust boundaries: input validation, identity, resource/tenant ownership, secret/PII exposure, consistent error mapping and limits. For writes, examine applicable duplicate requests, partial failures, concurrent updates and transactional boundaries using the relevant guides. Observability must help diagnose failure without exposing secrets. Avoid adding retries, caching, queues or new layers without the required justification.

For approved contract/schema/dependency changes, list consumers and apply compatible changes within scope. For migrations, inspect current schema and existing data assumptions, ordering and constraints; explain recovery/forward-fix options and irreversible effects. Do not promise rollback unless supported. Preparing files does not authorize live execution. Do not install or upgrade packages, regenerate lockfiles, apply a migration or alter configuration outside the approved task merely to make a check pass.

## 4. Verify the candidate

Inspect the actual diff and final affected code against the original criteria, not just the intended change. Check source coverage under [syntax-policy.md](syntax-policy.md), module ownership, contract synchronization, security/failure paths, UI/design consistency, duplication and accidental edits. Every changed app must run its existing `npm run check` from that app's directory when available; also run any applicable prescribed build or focused check not already included. Confirm what a script actually executes before treating it as evidence. Never claim a build proves all behavior or a green command proves missing test coverage.

Use existing Tester-owned suites without editing them. Read the [frontend testing ownership rules](../../../docs/frontend-guide/04-quality/testing.md) and task-relevant [backend test strategy](../../../docs/backend-guide/09-testing/testing-strategy.md). Generic guide advice to write tests routes to Tester under this role's ownership boundary. Missing negative/boundary/regression coverage is a handoff gap for Orchestrator and Tester, not permission to write in their folders. The owner's manual E2E remains acceptance; do not add or run automated E2E without its explicit authorization. Any assigned developer smoke check uses only permitted environments/data and is labeled with its actual narrow scope.

Record exact command, working directory, candidate, relevant environment, exit/result and output evidence for each check. Redact secrets from retained logs. On missing tooling/access or external-service dependency, record `NOT RUN`/`BLOCKED` and the precise gap; do not install a replacement, fake a service or downgrade the check. A source-map gap is blocking under the syntax policy even when compilation passes.

## 5. Investigate and repair failures

For each reported failure, read the original requirement and full relevant error output. Reproduce it on the identified candidate with the permitted environment/data; retain actual versus expected evidence. If it cannot be reproduced, record conditions and uncertainty rather than dismissing the report. Distinguish a product defect, test defect, environment failure, missing requirement and source-policy conflict before deciding a correction.

Fix product code within approved scope and record the root cause. A test defect goes to its Tester through Orchestrator with exact evidence; do not modify assertions, snapshots, fixtures, test discovery or quality thresholds. A requirement/design/contract change goes through change control. If the same issue recurs or a fix causes repeated regressions, stop speculative patching and report the unresolved cause and affected scope for coordinated investigation.

After a code fix, identify a new candidate, rerun the relevant reproducer and affected checks, plus `npm run check` in each changed app. Include shared consumers and neighboring behavior in regression scope. Under the [delivery cycle](../../orchestrator/skillz/delivery-cycle.md), Orchestrator reviews every Tester result and personally reruns the reproducer and affected/regression checks after your corrections. Do not send fixes back to Tester automatically. Your repair evidence does not close an issue; Orchestrator records its own candidate-specific verification or an explicit gap. Preserve previous issue IDs and evidence; no silent carryover of earlier PASS results to changed code.

## 6. Self-review and deliver

Reconcile the TODO with actual changed files, source decisions and evidence. Remove only your own task-generated debris where authorized; preserve unrelated work and open concerns. Verify that no requirement was silently dropped, no unsupported technical choice entered the diff, and no contract consumer was omitted. Any unresolved check, prerequisite or design state remains visible in the Output.

Reconcile affected project-map paths/responsibilities and links with the final filesystem/code. Apply the [runnable handoff and owned-resource procedure](project-map-and-handoff.md): include real startup commands/configuration prerequisites and observed state, then stop/remove verified task-owned disposable resources or document an authorized continuation. Retained evidence and other operators' processes are preserved. Record failed or ambiguous cleanup explicitly.

Copy [../templates/output.md](../templates/output.md) to its assigned path. Use Changed / Verified / Checks / Remaining as the report structure. Identify the candidate by commit when present, or a manifest of changed file identities/hashes and evidence time when uncommitted; branch name alone is insufficient. Include checks after the final code edit and label later changes as requiring revalidation. Do not create a commit merely to obtain an identity.

Give Tester affected requirements, files/contracts, relevant failure cases, commands/environment and unresolved coverage. Give Designer built routes/states, implementation identity, approved design reference and known visual gaps. Route both through Orchestrator; direct peer communication requires actual authorization. Before any authorized bridge operation, read [workspace/bridge.md](../../../bridge.md) and its permitted guide. An optional Advisor is not a mandatory gate or permanent named model. Report a candidate for review with precise blockers; do not update project acceptance/status records, release, or dispatch workers yourself.
