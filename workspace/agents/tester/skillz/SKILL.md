# Tester skill — plan, verify, report

Read [backend-only scope](../../../docs/backend-guide/00-navigation/backend-only-scope.md) at startup. No UI design or frontend work is in scope.

Use this skill after `../prompt.md` on every assigned Tester task. This is a task procedure, not an operator assignment or permission grant. Read [verification.md](verification.md) for the check matrix and execution rules, and [qa-skills-integration.md](qa-skills-integration.md) before using the imported QA library. Select task-relevant methods from [qa-skills-catalog.md](qa-skills-catalog.md); the complete upstream skill and its `references/` files remain available under `references/qa-skills-upstream/skills/`. Read only the selected references and task-relevant guide/syntax sections after their navigation entries.

## 1. Establish the test target

Record the brief ID, assigned operator, candidate revision/build, affected apps, changed files/functions, approved Planner Output and acceptance-criteria IDs, API/schema/security contracts, allowed environment and data, permitted test-writing folder, report destination and output paths. Inspect adjacent callers, shared code, existing tests and recent defect history. Name the CONTRACT SPECIFICATION or IMPLEMENTATION phase. If the candidate changes during the assigned initial phase, explicitly reset stale results; never combine revisions into one PASS. Later corrections follow the [delivery cycle](../../orchestrator/skillz/delivery-cycle.md): Orchestrator verifies them personally, without an automatic Tester return.

If an essential input is absent, mark that work `BLOCKED`, explain the missing input to the Orchestrator and continue independent checks that remain valid. A blank role assignment is not permission to select an operator.

## 2. Build the TODO before tests

Use [../templates/todo.md](../templates/todo.md) at the brief's task-record path. For each approved acceptance criterion and relevant project rule, list the defect class, test level or review method, expected oracle, data/environment, risk and evidence to collect. Include negative, boundary, role/authorization, failure and regression cases according to exposure. Name planned exclusions and what would make them applicable. Send the TODO to the Orchestrator before running the planned checks; incorporate feedback without expanding permissions.

Screen the stable stream IDs in [verification.md](verification.md). Record each applicable stream with its actual scope and evidence plan; grouped exclusions must name their IDs and common reason. Do not create a different agent or mandatory workflow gate for every stream. One check may support several stream results when it actually observes them; avoid mirrored tests or copying evidence. Separate API behavior from producer/consumer contract compatibility, tool gates from local syntax conformity, and visual accuracy from usability/accessibility.

## 3. Verify within the assigned phase

For CONTRACT SPECIFICATION review the versioned software TODO/Output and exact API/schema/security definitions: coverage, field semantics, statuses/headers/bodies/examples, errors, auth/tenant/resource rules, integrity/concurrency/idempotency, migrations/recovery, source support and verifiable ACs. Use [contract procedure](../../planner/skillz/software-contracts.md) as completeness criteria, not permission to author the plan. Runtime claims remain UNVERIFIED. IMPLEMENTATION then checks actual backend code/runtime within authorization.

Check every scoped operation maps to a single complete canonical contract, security controls and applicable data definition. Compare examples and generated API docs with actual source versions; unresolved policy or contradictory shapes block the dependent gate.

Use [verification.md](verification.md). Start with cheap static and automated checks, then execute behavioral, integration and specialized checks that the change warrants. Record each pass separately. Search the affected scope, not just the diff. Never turn a check into a mandatory ceremony where it cannot catch a meaningful defect; document `NOT APPLICABLE` with a reason.

Use [focused-checks.md](focused-checks.md) when usability, localization, privacy or recovery applies, retaining separate API-usability, service compatibility, localization, privacy and recovery outcomes; VISUAL/A11Y are outside API-only scope. Use [security-baseline.md](security-baseline.md) to pin actual security requirements and methods, then the existing QA methods for their task-specific techniques. These procedures extend this same TODO/Output and do not replace project guides, approve policy, install tools or authorize destructive/live checks. A missing required condition is BLOCKED/NOT RUN rather than an exclusion. Document reviews and runtime checks stay labeled separately.

When writing tests, prove meaningful behavior and failure paths. Use isolated fixtures; do not share mutable static records with another tester. Test the real contract at the right boundary. For backend persistence behavior, follow the backend guide's real-infrastructure strategy where an authorized isolated environment exists. A mock database cannot prove a migration, transaction or SQL constraint.

Browser/accessibility/visual testing methods are outside this API-only scope. Imported references are retained for provenance and selected only through the backend adapter; no UI or browser installation is implied.

Use actual permitted HTTP/service and database tooling for API/contract/integration checks; do not open a browser or install Playwright/Lighthouse to verify an API.

## 4. Verify findings and fixes

For a suspected defect, check the cited requirement/rule verbatim, inspect the actual file and line, reproduce on the recorded candidate, and distinguish observed failure from hypothesis. Report confirmed findings through the Orchestrator with expected versus actual behavior and a proposed regression check. Do not patch production source yourself.

When a failure's cause is unclear, crosses app/service boundaries, or survives a fix, use [failure-investigation.md](failure-investigation.md) to trace evidence before recommending a cause. It adds a focused diagnostic method; it does not transfer implementation ownership to the Tester.

For the initial report, provide original reproduction details, affected/shared regression scope, commands/environment and any assigned tests. Orchestrator reviews each result independently, routes formal corrections and personally verifies the new candidate. Do not reopen your phase or retest fixes automatically. A separately owner-requested later assignment must name its new scope/candidate; historical evidence remains historical. Recurring failures require causal investigation rather than local patch cycling.

## 5. Deliver an auditable Output

Use [../templates/output.md](../templates/output.md). Every acceptance criterion is `PASS`, `FAIL` or `UNVERIFIED` with evidence. Every check is `PASS`, `FAIL`, `BLOCKED`, `NOT RUN` or `NOT APPLICABLE`, with a reason when it did not run. `BLOCKED` and `NOT RUN` roll up to `UNVERIFIED`, never `PASS`. Include exact commands and exit codes, finding IDs, source locations, affected scope, residual risks and an evidence-based readiness recommendation. The Orchestrator validates findings and decides routing; the owner/authorized decision-maker accepts product risk and final completion.


Give applicable streams their own candidate/phase-specific results and link the actual check evidence. Retain the applicability/exclusion record so categories cannot silently disappear; a grouped exclusion is acceptable but an omnibus security/accessibility/performance/reliability PASS is not. A required failed check prevents its stream from passing; missing required evidence remains explicitly blocked/not run with affected ACs UNVERIFIED. For focused checks include the actual matrix/data/run conditions and limits; security mappings include exact standard versions/IDs and observed coverage. Never infer a user study, physical device, restored service or full-standard conformance from another method's success.
