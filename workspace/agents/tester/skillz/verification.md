# Verification matrix and execution rules

Select checks based on the changed surface, defect likelihood and user impact. These are distinct evidence streams, not one linear test stage. Record a result for every applicable stream in the TODO and Output. For low-risk changes, a concise justified exclusion is better than theatrical testing.

| Stream | Target and evidence | Primary local source |
| --- | --- | --- |
| Project-guide compliance | Check affected backend code/definitions against actual backend rules; cite exact source and candidate lines. | Backend README/map and relevant actual sections |
| Technology/syntax compliance | Check relevant `syntax` pages for technology behavior and misuse. A technology reference does not silently override a project decision. | `workspace/docs/syntax/<technology>/` |
| Automated code health | Run configured lint/format/typecheck/test/build including npm run check when available; record candidate/cwd/command/exit/output. | Backend initialization, actual package scripts and test strategy |
| Maintainability review | Review duplication, misplaced responsibilities, names, complexity, dead code and actual defect/maintenance impact. | Backend complexity-control, project structure and anti-patterns |
| Functional behavior | Cover every applicable AC with unit/service/API/integration checks able to observe it, including negative/boundary/state cases. | Approved Planner Output/software TODO and backend test strategy |
| API contracts and external integration | Check actual HTTP methods/paths/parameters/body/auth/status/headers/success/errors against exact canonical definitions; verify no API/doc drift. No external client implementation is implied. | Backend API/auth/validation guides; exact contracts and approved requirements |
| Specialized backend risk | Check auth/tenant/object isolation, injection/secrets, concurrency/idempotency, provider failures/timeouts, agreed performance, logs, migrations and recovery as applicable, using safe isolated environments. | Relevant backend guides and approved API/DB/SEC definitions |
| Regression | Re-run relevant existing suites and previously working paths touched by the change; after a fix, both the original failure and surrounding scope. | Baseline evidence, existing tests, task brief |

## How to choose test techniques

- Use equivalence classes, boundary values, decision tables or state transitions where the input or behavior warrants them. Use property-based tests, fuzzing or mutation testing only when a specific invariant/risk and suitable tooling justify the cost.
- A useful test names the defect it would catch. Do not chase a universal coverage percentage or duplicate trivial assertions.
- Backend persistence, migrations, constraints, transactions and concurrency need an isolated real database or other authorized realistic infrastructure. Mock or stub external provider boundaries to avoid side effects; simulate timeout, error and malformed responses as relevant.
- Browser UI, visual/accessibility auditing and component tests are outside this API-only scope. HTTP assertions prove the service contract without browser automation.
- Performance claims need a stated baseline, budget, comparable environment and measurement method. Without these, record a risk or measurement plan, not a performance PASS.
- Owner manual API acceptance is separate from worker verification. Ordinary assigned API integration/contract tests remain valid; automated full-system E2E needs explicit separate authorization.

## Evidence and safety

For every execution, capture the candidate, environment, fixture identity, command/tool, exit code, observed output and timestamp when useful. For a source review, cite both the source rule and actual candidate file/line. Redact secrets and personal data. Do not infer test success from code appearance, stale CI or another candidate's run.

Before any destructive, load-heavy or externally visible check, confirm the brief expressly allows that environment and action. If it does not, write an executable safe test plan and mark execution `BLOCKED` or `NOT RUN`; do not simulate a PASS. Isolate and clean up test data, including across parallel tester operators.

If a requirement is absent, contradictory or untestable, report a `SOURCE/CONTRACT GAP` with a concrete counterexample. Do not rewrite the Planner Output or silently invent expected behavior. If two local sources disagree, quote both and send the conflict to the Orchestrator for resolution.
