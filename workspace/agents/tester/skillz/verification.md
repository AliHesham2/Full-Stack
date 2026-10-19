# Verification matrix and execution rules

Select checks based on the changed surface, defect likelihood and user impact. These are distinct evidence streams, not one linear test stage. Record a result for every applicable stream in the TODO and Output. For low-risk changes, a concise justified exclusion is better than theatrical testing.

| Stream | Target and evidence | Primary local source |
| --- | --- | --- |
| Project-guide compliance | Check affected code against the relevant frontend/backend guide sections; quote each violated rule, path and line. A cross-app API change requires both guides. | `workspace/docs/frontend-guide/README.md`; `workspace/docs/backend-guide/README.md` |
| Technology/syntax compliance | Check relevant `syntax` pages for technology behavior and misuse. A technology reference does not silently override a project decision. | `workspace/docs/syntax/<technology>/` |
| Automated code health | Run the affected app's configured format/lint/typecheck/test/build gate, including `npm run check` where the app defines it. Record cwd, command, exit code and output. | Each app's `01-foundation/initialization.md`; frontend `04-quality/code-quality.md`; backend `09-testing/testing-strategy.md` |
| Maintainability review | Look for duplicated logic, misplaced responsibility, unclear names, unnecessary complexity, dead code and relevant anti-patterns. Explain the concrete maintenance or defect risk; do not report style preference as a bug. | Frontend `04-quality/code-quality.md` and `anti-patterns.md`; backend `01-foundation/complexity-control.md` and `12-anti-patterns/anti-patterns.md` |
| Functional behavior | Cover each approved AC with unit/component/API/integration checks at the level able to observe it. Include success, negative, boundary and state transitions that matter. | Approved Planner Output; each app's testing guide |
| Contract and cross-app integration | Verify frontend HTTP usage against backend-owned request/response schema, status/error envelope, auth, serialization and compatibility. Check that only Express accesses the database/provider credentials. | Frontend `03-data/backend-integration.md`; backend `02-domain-and-api/api-design.md`; approved Planner Output |
| Specialized risk | When affected: authorization and tenant isolation, injection and secrets, accessibility/keyboard/RTL, concurrency/idempotency, dependency failure, performance against agreed budgets, observability, migration and rollback. Use safe test environments. | Applicable guide sections and approved requirements |
| Regression | Re-run relevant existing suites and previously working paths touched by the change; after a fix, both the original failure and surrounding scope. | Baseline evidence, existing tests, task brief |

## How to choose test techniques

- Use equivalence classes, boundary values, decision tables or state transitions where the input or behavior warrants them. Use property-based tests, fuzzing or mutation testing only when a specific invariant/risk and suitable tooling justify the cost.
- A useful test names the defect it would catch. Do not chase a universal coverage percentage or duplicate trivial assertions.
- Backend persistence, migrations, constraints, transactions and concurrency need an isolated real database or other authorized realistic infrastructure. Mock or stub external provider boundaries to avoid side effects; simulate timeout, error and malformed responses as relevant.
- Component/accessibility tools supplement manual inspection where that inspection is in scope. A passing automated accessibility check does not prove keyboard, reading order or visual usability.
- Performance claims need a stated baseline, budget, comparable environment and measurement method. Without these, record a risk or measurement plan, not a performance PASS.
- The owner's manual E2E is separate from worker verification. Automated E2E is conditional on an explicit request; the frontend guide does not install it by default.

## Evidence and safety

For every execution, capture the candidate, environment, fixture identity, command/tool, exit code, observed output and timestamp when useful. For a source review, cite both the source rule and actual candidate file/line. Redact secrets and personal data. Do not infer test success from code appearance, stale CI or another candidate's run.

Before any destructive, load-heavy or externally visible check, confirm the brief expressly allows that environment and action. If it does not, write an executable safe test plan and mark execution `BLOCKED` or `NOT RUN`; do not simulate a PASS. Isolate and clean up test data, including across parallel tester operators.

If a requirement is absent, contradictory or untestable, report a `SOURCE/DESIGN GAP` with a concrete counterexample. Do not rewrite the Planner Output or silently invent expected behavior. If two local sources disagree, quote both and send the conflict to the Orchestrator for resolution.
