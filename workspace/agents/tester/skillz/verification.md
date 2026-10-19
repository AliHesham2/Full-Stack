# Verification matrix and execution rules

Select checks based on the changed surface, defect likelihood and user impact. These are distinct evidence streams, not one linear test stage. Record a result for every applicable stream in the TODO and Output. For low-risk changes, a concise justified exclusion is better than theatrical testing.

| Stream | Target and evidence | Primary local source |
| --- | --- | --- |
| Project-guide compliance | Check affected frontend code against actual frontend rules; cite exact path/line and supplied API version. | `workspace/docs/frontend-guide/README.md`; owner API documents; approved Planner Output |
| Technology/syntax compliance | Check relevant `syntax` pages for technology behavior and misuse. A technology reference does not silently override a project decision. | `workspace/docs/syntax/<technology>/` |
| Automated code health | Run configured format/lint/typecheck/test/build, including npm run check where defined. Record actual command/cwd/candidate/exit/output. | Frontend initialization and code-quality guides; actual app scripts |
| Maintainability review | Review duplication, misplaced responsibilities, names, complexity, dead code and anti-patterns; identify concrete maintenance/defect impact, not taste. | Frontend code-quality and anti-patterns guides |
| Functional behavior | Cover each approved AC with unit/component/API/integration checks at the level able to observe it. Include success, negative, boundary and state transitions that matter. | Approved Planner Output; each app's testing guide |
| Supplied API and frontend integration | Check exact supplied requests, successes/failures, auth/serialization, safe forwarding and approved frontend normalization. External service internals are outside scope; no DB/provider clients in Next.js. | Frontend integration/security guides; exact owner API sources; software TODO |
| Specialized risk | Check affected frontend auth/permission states, XSS/CSRF/secrets, keyboard/RTL/accessibility, duplicate actions/cancellation, failures and measured performance. Do not claim external authorization/persistence enforcement. | Applicable frontend/design guides and approved requirements |
| Regression | Re-run relevant existing suites and previously working paths touched by the change; after a fix, both the original failure and surrounding scope. | Baseline evidence, existing tests, task brief |

## How to choose test techniques

- Use equivalence classes, boundary values, decision tables or state transitions where the input or behavior warrants them. Use property-based tests, fuzzing or mutation testing only when a specific invariant/risk and suitable tooling justify the cost.
- A useful test names the defect it would catch. Do not chase a universal coverage percentage or duplicate trivial assertions.
- Use fixtures for frontend handling of supplied success/error/timeout/malformed responses only. Real API integration requires an approved external environment and permitted data/actions. Backend persistence, migrations, constraints and transaction tests are outside this template; a mock proves none of them.
- Component/accessibility tools supplement manual inspection where that inspection is in scope. A passing automated accessibility check does not prove keyboard, reading order or visual usability.
- Performance claims need a stated baseline, budget, comparable environment and measurement method. Without these, record a risk or measurement plan, not a performance PASS.
- The owner's manual E2E is separate from worker verification. Automated E2E is conditional on an explicit request; the frontend guide does not install it by default.

## Evidence and safety

For every execution, capture the candidate, environment, fixture identity, command/tool, exit code, observed output and timestamp when useful. For a source review, cite both the source rule and actual candidate file/line. Redact secrets and personal data. Do not infer test success from code appearance, stale CI or another candidate's run.

Before any destructive, load-heavy or externally visible check, confirm the brief expressly allows that environment and action. If it does not, write an executable safe test plan and mark execution `BLOCKED` or `NOT RUN`; do not simulate a PASS. Isolate and clean up test data, including across parallel tester operators.

If a requirement is absent, contradictory or untestable, report a `SOURCE/DESIGN GAP` with a concrete counterexample. Do not rewrite the Planner Output or silently invent expected behavior. If two local sources disagree, quote both and send the conflict to the Orchestrator for resolution.
