# Tester skill — plan, verify, report

Use this skill after `../prompt.md` on every assigned Tester task. This is a task procedure, not an operator assignment or permission grant. Read [verification.md](verification.md) for the check matrix and execution rules, and [qa-skills-integration.md](qa-skills-integration.md) before using the imported QA library. Select task-relevant methods from [qa-skills-catalog.md](qa-skills-catalog.md); the complete upstream skill and its `references/` files remain available under `references/qa-skills-upstream/skills/`. Read only the selected references and task-relevant guide/syntax sections after their navigation entries.

## 1. Establish the test target

Record the brief ID, assigned operator, candidate revision/build, affected apps, changed files/functions, approved PRD and acceptance-criteria IDs, design/API contracts, allowed environment and data, permitted test-writing folder, report destination and output paths. Inspect adjacent callers, shared code, existing tests and recent defect history. Name the DESIGN SPECIFICATION or IMPLEMENTATION phase. If the candidate changes during the assigned initial phase, explicitly reset stale results; never combine revisions into one PASS. Later corrections follow the [delivery cycle](../../orchestrator/skillz/delivery-cycle.md): Orchestrator verifies them personally, without an automatic Tester return.

If an essential input is absent, mark that work `BLOCKED`, explain the missing input to the Orchestrator and continue independent checks that remain valid. A blank role assignment is not permission to select an operator.

## 2. Build the TODO before tests

Use [../templates/todo.md](../templates/todo.md) at the brief's task-record path. For each approved acceptance criterion and relevant project rule, list the defect class, test level or review method, expected oracle, data/environment, risk and evidence to collect. Include negative, boundary, role/authorization, failure and regression cases according to exposure. Name planned exclusions and what would make them applicable. Send the TODO to the Orchestrator before running the planned checks; incorporate feedback without expanding permissions.

## 3. Verify within the assigned phase

For DESIGN SPECIFICATION, inspect actual design TODO/Output, requirement coverage, software/API/data constraints, kit/tokens, geometry, typography, responsive intervals and resize rules, all relevant states, focus/keyboard/semantics and unresolved decisions. Use the [design TODO template](../../designer/templates/design-todo.md) to identify required specification dimensions, not to author the design. Document checks can pass while runtime-dependent behavior remains unverified. Do not run code tests/builds against a nonexistent app or infer visual accessibility from prose. For IMPLEMENTATION, use the risk-aware passes below against actual code/runtime.

Use [verification.md](verification.md). Start with cheap static and automated checks, then execute behavioral, integration and specialized checks that the change warrants. Record each pass separately. Search the affected scope, not just the diff. Never turn a check into a mandatory ceremony where it cannot catch a meaningful defect; document `NOT APPLICABLE` with a reason.

When writing tests, prove meaningful behavior and failure paths. Use isolated fixtures; do not share mutable static records with another tester. Test the real contract at the right boundary. For backend persistence behavior, follow the backend guide's real-infrastructure strategy where an authorized isolated environment exists. A mock database cannot prove a migration, transaction or SQL constraint.

When an authorized accessibility check uses `axe-core` or a wrapper around it, follow [axe-core-results.md](axe-core-results.md) to record what was actually scanned and what remains unverified. The existing accessibility-testing skill supplies broader keyboard, assistive-technology and WCAG methods; this page adds only axe result interpretation and environment limits.

For authorized browser inspection through Playwright MCP, follow [playwright-mcp-inspection.md](playwright-mcp-inspection.md). For an authorized Lighthouse audit, follow [lighthouse-evidence.md](lighthouse-evidence.md). These are tool-specific evidence procedures; neither authorizes installing tools, automated E2E or new CI gates.

## 4. Verify findings and fixes

For a suspected defect, check the cited requirement/rule verbatim, inspect the actual file and line, reproduce on the recorded candidate, and distinguish observed failure from hypothesis. Report confirmed findings through the Orchestrator with expected versus actual behavior and a proposed regression check. Do not patch production source yourself.

When a failure's cause is unclear, crosses app/service boundaries, or survives a fix, use [failure-investigation.md](failure-investigation.md) to trace evidence before recommending a cause. It adds a focused diagnostic method; it does not transfer implementation ownership to the Tester.

For the initial report, provide original reproduction details, affected/shared regression scope, commands/environment and any assigned tests. Orchestrator reviews each result independently, routes formal corrections and personally verifies the new candidate. Do not reopen your phase or retest fixes automatically. A separately owner-requested later assignment must name its new scope/candidate; historical evidence remains historical. Recurring failures require causal investigation rather than local patch cycling.

## 5. Deliver an auditable Output

Use [../templates/output.md](../templates/output.md). Every acceptance criterion is `PASS`, `FAIL` or `UNVERIFIED` with evidence. Every check is `PASS`, `FAIL`, `BLOCKED`, `NOT RUN` or `NOT APPLICABLE`, with a reason when it did not run. `BLOCKED` and `NOT RUN` roll up to `UNVERIFIED`, never `PASS`. Include exact commands and exit codes, finding IDs, source locations, affected scope, residual risks and an evidence-based readiness recommendation. The Orchestrator validates findings and decides routing; the owner/authorized decision-maker accepts product risk and final completion.
