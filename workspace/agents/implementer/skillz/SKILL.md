# Implementer skill — inspect, implement, verify, hand off

Read [frontend-only scope](../../../docs/frontend-guide/00-navigation/frontend-only-scope.md) at startup. This procedure operates only within that boundary.

Start at [../prompt.md](../prompt.md). This is the model-neutral implementation procedure, not a role assignment or permission grant. The [syntax policy](syntax-policy.md) is mandatory before code: personal/installed Skills and remembered or externally fetched syntax cannot replace the local technical references.

In the new input-driven route, approved Planner Output/version and the software TODO supply the requirements/task baseline; Designer's detailed design TODO supplies visual implementation requirements. No deleted document/template is required. Start technical reading at [syntax GUIDE-MAP](../../../docs/syntax/GUIDE-MAP.md), then actual indexes/pages; record the selected sources and compatible versions.

Read the Planner-owned software TODO and the Designer-owned detailed design TODO/version with actual design gate evidence. Continue the software TODO as the execution checklist; reference design items without creating a competing merged specification. Missing or conflicting dependencies block affected work.

## 1. Readiness and source route

Read the assignment, approved Planner Output and assigned software TODO item completely, then the relevant contracts and Designer Output. Identify source IDs, deliverables, exclusions and the exact evidence required. Do not execute an unapproved draft or start the next software task without an assignment. Missing product behavior goes to Planner/requirements owner through Orchestrator; missing UI behavior goes to Designer through the same route. Continue independent, fully specified work where possible.

Use the [frontend guide](../../../docs/frontend-guide/README.md) → [frontend workflow](../../../docs/frontend-guide/00-navigation/agent-workflow.md), relevant actual pages and owner-supplied API sources. There is no backend-guide route or backend implementation in scope. Resolve syntax gaps through the exclusive local policy, never an alternate implementation source.

| Affected concern | Local reading route after navigation |
| --- | --- |
| App setup or file boundaries | Applicable guide `01-foundation/initialization.md`, `architecture.md`, `project-structure.md`; actual manifests/lockfile/runtime evidence |
| UI and user-visible states | Designer Output; frontend `02-ui/`, routing, `03-data/`, security, accessibility and resilience sections |
| HTTP and shared payloads | Frontend `03-data/backend-integration.md`; exact owner API documents, approved frontend DTOs and consumers |
| Authentication/authorization | Owner-supplied auth/permission contract; frontend security guide; actual Next.js session/transport boundary |
| External state changes | Supplied operation semantics; approved frontend retry/reconciliation/cache plan; no database implementation |
| Reliability, dependencies or delivery files | Frontend `04-quality/` and `05-delivery/`; actual app scripts/config |

For React/Next.js changes, use [react-best-practices.md](react-best-practices.md) to select and read relevant performance rules within the existing guide/syntax route. Apply it during implementation and final self-review; it creates no additional workflow or artifact.

## 2. Inspect the baseline and prepare the TODO

Locate the actual repository root, relevant apps and current changes. Inspect affected files, callers, contracts, shared helpers, configuration and existing tests. Search for equivalent functionality before adding it. Record pre-existing edits and failures without reverting them or assuming another operator has finished. If a shared file changes during your work, reread it and reconcile within your authority; stop overlapping edits when ownership is unresolved.

Confirm actual package/runtime versions and available scripts. An uninitialized starter has no runnable baseline: record that fact and follow initialization only if assigned, never fabricate a passing check or initialize an app merely to complete a report. Existing failures need their actual command/output and relevance; do not label a failure pre-existing unless observed before your change or supported by equivalent evidence.

Follow [project-map-and-handoff.md](project-map-and-handoff.md) to read, establish and maintain one canonical implementation map from actual code. Record its path and inspected scope in the TODO before code edits; keep proposed structure separate until implemented. Read that map on each new assignment and verify the affected branch rather than trusting stale navigation. Track task-created processes and temporary paths from the moment they are planned/created.

Validate and continue the Planner-prepared software TODO against [../templates/todo.md](../templates/todo.md); create one only if none was supplied within an approved assignment. Preserve approved scope and add actual baseline/execution information in that same file. Keep design-dependent items blocked until the actual design TODO/version and design gate evidence are linked. Map each `I-n` item to requirement/module/design IDs, allowed files, guide and syntax sources, dependencies, failure paths, risk and planned checks. Record any unresolved contract/version/design gap. Keep the TODO updated as work proceeds, including the candidate, blockers and next authorized action; it is the resumable plan, not a second project status system.

Check supplied API and frontend data/security definitions under [software-contracts.md](../../planner/skillz/software-contracts.md). Each executable item links exact settled requests, success/failure outcomes, DTO/state mappings and security enforcement/verification. Check required attachments/version access. No missing upstream contract or frontend policy is delegated to you; block and report gaps.

## 3. Implement the smallest coherent change

Follow approved frontend dependencies. Consumed operations must have complete owner-supplied contracts before integration. Do not create a backend or invent payloads to simulate completeness.

Keep Browser → Next.js → supplied external API intact. Derive consumer DTOs from exact approved contract versions through the documented synchronization route, without external backend runtime imports or guessed shapes. Trace every success/error/status and nullable/optional field through frontend consumers. Missing synchronization is a prerequisite gap.

Use guide-owned layers and existing abstractions; each new helper, module or dependency needs a current requirement. Follow prescribed generation/setup commands only when assigned and syntax-supported, then inspect generated files and the complete diff. Do not replace the portable component kit or generate a second token system. Match the approved design's real content, viewports, themes, RTL, focus and loading/empty/error/success states. Do not replace unfinished integrations with hardcoded success, swallowed errors, no-op handlers or undisclosed mock data.

For affected frontend trust boundaries inspect input/response validation, output safety, session/CSRF integration, safe URLs and redirects, secrets/PII, error messages, resource bounds and permitted outbound calls. Respect supplied external concurrency/idempotency rules and reconcile uncertain write outcomes. Do not implement or claim external authorization/persistence enforcement. Avoid unapproved retries, caching or layers.

For approved frontend DTO/dependency/config changes list consumers and compatibility/recovery effects. External API changes require new owner-supplied definitions before dependent work. Do not prepare/apply migrations or change an external project; do not install/upgrade packages or alter lockfiles/config outside approved scope just to pass a check.

## 4. Verify the candidate

Inspect the actual diff and final affected code against the original criteria, not just the intended change. Check source coverage under [syntax-policy.md](syntax-policy.md), module ownership, contract synchronization, security/failure paths, UI/design consistency, duplication and accidental edits. Every changed app must run its existing `npm run check` from that app's directory when available; also run any applicable prescribed build or focused check not already included. Confirm what a script actually executes before treating it as evidence. Never claim a build proves all behavior or a green command proves missing test coverage.

Use existing Tester-owned suites without editing them. Read [frontend testing ownership rules](../../../docs/frontend-guide/04-quality/testing.md). Missing regression/negative coverage is a gap for Orchestrator/Tester. Manual E2E remains owner acceptance; automated E2E needs explicit authorization. A permitted smoke check records its actual limited scope.

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
