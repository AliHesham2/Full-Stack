# Verification matrix and execution rules

Read [frontend-only scope](../../../docs/frontend-guide/00-navigation/frontend-only-scope.md). External APIs are supplied by the owner. No backend source, DB tests or provider restoration is assigned here.

Select checks based on the changed surface, defect likelihood and user impact. These are evidence streams within the assigned Tester role, not separate agents, compulsory tool installations or consecutive workflow stages. Screen applicability before execution, using the stable IDs below in TODO and Output. Give each applicable stream its own result and evidence; a passing build or security scan cannot stand in for another stream. Related checks may share evidence without duplicating tests.

Keep this proportional: justify scope-based exclusions, grouping exclusions with the same reason when useful, and record the excluded IDs. Missing access, tools, requirements or implementation is `BLOCKED`/`NOT RUN`, not `NOT APPLICABLE`. In DESIGN SPECIFICATION, label document checks separately; a document PASS cannot satisfy runtime criteria. No stream changes the shared delivery cycle or authorizes another Tester round after corrections.

| ID / stream | Target and evidence | Primary reading route |
| --- | --- | --- |
| GUIDE — project rules | Read actual frontend guide sections and approved task sources; cite exact rule/candidate. Owner API definitions govern consumed behavior; no backend guide is required. | frontend map, actual sections and approved requirements |
| SYNTAX — local reference conformity | Compare technical constructs with exact compatible local syntax sections. Successful parsing/typing does not prove this conformity. | `workspace/docs/syntax/`; Implementer's syntax policy |
| HEALTH — configured code gates | Run actual format/lint/typecheck/compile/build gates and `npm run check` when defined; record command, cwd, candidate and exit/output. | Actual package scripts; initialization/code-quality guides |
| STATIC — analysis and maintainability | Review likely defects, dead code, duplication, names, complexity and misplaced responsibility. Distinguish tool findings from manual judgment and investigate false positives. | Code-quality, complexity-control and anti-pattern guides |
| ARCH — architecture | Verify approved frontend layers, module ownership and dependency direction; do not impose another architecture style. | frontend architecture/project structure and approved decisions |
| UNIT — isolated behavior | Exercise meaningful function/component/server-adapter behavior and negative/boundary cases; no per-function quota. | Actual frontend test strategy; unit-testing reference |
| LOGIC — business rules | Verify requirement-derived decisions, invariants, permissions and state transitions. This is core functional coverage whenever business rules exist, not a large-project extra. | Approved Planner Output/ACs and actual domain rules |
| INTEGRATION — collaborating parts | Check actual frontend routes/actions/adapters and supplied external operation boundaries. Fixtures prove handling, not live service behavior; actual calls need a permitted API environment. | Approved contracts, actual frontend test strategy and permitted fixtures |
| API — HTTP behavior | Verify frontend requests, safe forwarding and handling of supplied statuses/headers/bodies/auth/errors; do not author endpoints or prove external enforcement. | Exact owner-supplied APIs and approved frontend mappings |
| CONTRACT — consumer compatibility | Compare frontend requests/parsing/normalization and UI states against exact supplied versions; missing upstream definitions return to the owner. | Canonical contract versions and actual permitted consumers; contract-testing |
| SECURITY — frontend technical controls | Check XSS/CSRF, caller/session handling, safe outbound destinations, response parsing, server-only secrets and private caches at the frontend boundary; UI denial does not prove upstream authorization. | Security baseline; frontend security/integration; supplied auth contract |
| DATA — frontend DTO and state | Verify supplied DTO parsing, optional/null semantics, frontend state/cache ownership, invalidation and sensitive field boundaries. No SQL, migrations, DB constraints or provider tests. | Supplied schemas, frontend data/security rules and software TODO |
| DEPS — dependencies and supply chain | Check actual manifests/lockfiles, support/advisories, relevant vulnerable usage and approved script/provenance policies. Age or a newer release alone is not a vulnerability or upgrade authorization. | Dependency/security guides and actual audit evidence |
| VISUAL — visual accuracy | Compare observed UI with approved design/tokens at equivalent content, viewport, theme and state; use approved tolerances, not an invented percentage score. | Actual design TODO; `visual-testing`; [focused checks](focused-checks.md) |
| USABILITY — task usability | Inspect task clarity, navigation, feedback, errors and recovery separately from visual match. Disclose heuristic inspection versus actual user-study evidence. | Approved flows/ACs; [focused checks](focused-checks.md) |
| A11Y — accessibility | Check applicable semantics, keyboard/focus, reading order, contrast, resizing and announcements against the actual approved WCAG target; automated results do not prove full conformance. | Accessibility guide; [axe procedure](axe-core-results.md) |
| COMPAT — compatibility | Record actual browser/engine, viewport, theme, input and device coverage; responsive emulation is not physical-device testing. | Approved support matrix; `cross-browser-testing`; [focused checks](focused-checks.md) |
| I18N — localization | Check relevant languages, RTL/mixed content, plural/long text, formatting, timezone/currency and locale transitions without changing canonical wire formats. | [Focused checks](focused-checks.md); approved locale policy |
| PERF — frontend performance | Measure relevant render/bundle/network/interaction behavior with approved budgets; no load/stress on external APIs without a separate permitted target. | Frontend performance; Lighthouse procedure; approved measurement plan |
| RELIABILITY — frontend failures and recovery | Check cancellation, offline/dependency failures, stale responses and approved retry/reconciliation; never invent external idempotency or automatic mutation retries. | Frontend resilience, actual supplied operation policy |
| CONCURRENCY — overlapping frontend operations | Check duplicate submits, racing requests, stale results and optimistic rollback within approved frontend scope; no DB race/transaction enforcement claim. | Frontend state/resilience; exact supplied concurrency/idempotency semantics |
| OBS — frontend observability | Check required redacted frontend/Next.js events, correlation and errors within approved access; upstream traces/alerts are not assumed available. | Frontend delivery/security guide; actual test telemetry access |
| PRIVACY — personal-data handling | Check allowed frontend inputs/outputs, browser/Next.js caches/logs, third-party scripts and consent/lifecycle UI against approved policy. Do not infer upstream deletion or inspect external stores. | Focused checks; approved frontend data/privacy/security requirements |
| RECOVERY — frontend artifact and config recovery | Only approved frontend build/config/state restoration in an isolated target; external DB/provider backup/restore is OUT OF SCOPE, never a prerequisite. Otherwise justify inapplicability. | Focused checks; actual permitted frontend recovery plan |
| REGRESSION — existing behavior | Recheck previously working affected/shared paths on the actual candidate. Later correction checks remain with Orchestrator under the cycle. | Baseline, existing tests, original failure and task scope |
| EXPLORATORY — bounded exploration | Use a risk-based charter, time/scope boundary, recorded actions and observations; discovery does not replace required AC coverage. | `exploratory-testing`; approved brief |
| CHAOS — controlled fault injection | Use only a specific justified failure hypothesis, safe target, abort conditions and recovery plan; never infer permission from project size or skill availability. | `chaos-engineering`; actual environment permission |
| AI — AI-product behavior | Only for actual AI features: verify approved task quality, unsupported claims, injection/data exposure, tool permissions and failure/cost limits using repeatable datasets and declared nondeterminism. | `ai-system-testing`; actual product requirements |

The imported methods above are reached through [QA catalog](qa-skills-catalog.md) and its adapter, not auto-invoked. Business logic is checked even in small projects; privacy and recovery applicability follows data and loss impact, not project size. AI testing applies to an AI feature in the product, not merely because an AI agent wrote its code.

## How to choose test techniques

- Use equivalence classes, boundary values, decision tables or state transitions where the input or behavior warrants them. Use property-based tests, fuzzing or mutation testing only when a specific invariant/risk and suitable tooling justify the cost.
- A useful test names the defect it would catch. Do not chase a universal coverage percentage or duplicate trivial assertions.
- Backend DB/provider verification and restore drills are outside scope. Supplied-contract fixtures prove frontend handling only; real API calls need the approved environment, data and permissions. Missing required API access is BLOCKED/NOT RUN.
- Component/accessibility tools supplement manual inspection where that inspection is in scope. A passing automated accessibility check does not prove keyboard, reading order or visual usability.
- Performance claims need a stated baseline, budget, comparable environment and measurement method. Without these, record a risk or measurement plan, not a performance PASS.
- Keep architecture and maintainability findings tied to adopted rules or concrete defect/change cost; do not treat personal style, another architecture pattern or a tool warning as a confirmed defect.
- For OBS, distinguish inspecting logging configuration, observing an emitted event/trace and proving an alert reached its permitted test destination. Redact credentials/PII; do not page people or contact external recipients without actual authorization.
- The owner's manual E2E is separate from worker verification. Automated E2E is conditional on an explicit request; the frontend guide does not install it by default.

## Evidence and safety

For every execution, capture the candidate, environment, fixture identity, command/tool, exit code, observed output and timestamp when useful. For a source review, cite both the source rule and actual candidate file/line. Redact secrets and personal data. Do not infer test success from code appearance, stale CI or another candidate's run.

Before any destructive, load-heavy or externally visible check, confirm the brief expressly allows that environment and action. If it does not, write an executable safe test plan and mark execution `BLOCKED` or `NOT RUN`; do not simulate a PASS. Isolate and clean up test data, including across parallel tester operators.

If a requirement is absent, contradictory or untestable, report a `SOURCE/DESIGN GAP` with a concrete counterexample. Do not rewrite the Planner Output or silently invent expected behavior. If two local sources disagree, quote both and send the conflict to the Orchestrator for resolution.
