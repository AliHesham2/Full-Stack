# Verification matrix and execution rules

Read [backend-only scope](../../../docs/backend-guide/00-navigation/backend-only-scope.md). Only API/backend work is assigned. VISUAL and A11Y are out of scope; usability/compatibility/localization refer to API consumers and service behavior, not screens.

Select checks based on the changed surface, defect likelihood and user impact. These are evidence streams within the assigned Tester role, not separate agents, compulsory tool installations or consecutive workflow stages. Screen applicability before execution, using the stable IDs below in TODO and Output. Give each applicable stream its own result and evidence; a passing build or security scan cannot stand in for another stream. Related checks may share evidence without duplicating tests.

Keep this proportional: justify scope-based exclusions, grouping exclusions with the same reason when useful, and record the excluded IDs. Missing access, tools, requirements or implementation is `BLOCKED`/`NOT RUN`, not `NOT APPLICABLE`. In CONTRACT SPECIFICATION, label document checks separately; a document PASS cannot satisfy runtime criteria. No stream changes the shared delivery cycle or authorizes another Tester round after corrections.

| ID / stream | Target and evidence | Primary reading route |
| --- | --- | --- |
| GUIDE — project rules | Read actual backend guide sections and approved task sources; cite exact rule/candidate. No frontend/design guide is required. | backend map, actual sections and approved requirements |
| SYNTAX — local reference conformity | Compare technical constructs with exact compatible local syntax sections. Successful parsing/typing does not prove this conformity. | `workspace/docs/syntax/`; Implementer's syntax policy |
| HEALTH — configured code gates | Run actual format/lint/typecheck/compile/build gates and `npm run check` when defined; record command, cwd, candidate and exit/output. | Actual package scripts; initialization/code-quality guides |
| STATIC — analysis and maintainability | Review likely defects, dead code, duplication, names, complexity and misplaced responsibility. Distinguish tool findings from manual judgment and investigate false positives. | Code-quality, complexity-control and anti-pattern guides |
| ARCH — architecture | Verify approved backend layers, module ownership and dependency direction; do not impose another architecture style. | backend architecture/project structure and approved decisions |
| UNIT — isolated behavior | Exercise meaningful function/service/domain behavior and negative/boundary cases; no per-function quota. | Actual backend test strategy; unit-testing reference |
| LOGIC — business rules | Verify requirement-derived decisions, invariants, permissions and state transitions. This is core functional coverage whenever business rules exist, not a large-project extra. | Approved Planner Output/ACs and actual domain rules |
| INTEGRATION — collaborating parts | Exercise actual routes/services/persistence and allowed provider boundaries; distinguish real infrastructure from doubles. | Approved contracts, actual backend test strategy and permitted fixtures |
| API — HTTP behavior | Assert actual endpoint methods/paths, validation, auth, status/headers and complete success/error serialization. | Exact approved API definitions and backend auth/validation guides |
| CONTRACT — consumer compatibility | Compare published schemas/handlers/docs and external consumer compatibility, including optional/null/status/errors/pagination/version evolution; do not edit client projects. | Canonical contract versions and actual permitted consumers; contract-testing |
| SECURITY — backend technical controls | Check actual service/user/resource/tenant authorization, validation/injection/secrets/sessions/abuse controls at backend trust boundaries with versioned requirements. | Security baseline; backend security/auth/API rules |
| DATA — persistence and migrations | Verify actual approved constraints/relations, transactions, access, migrations/backfills and integrity in authorized isolated infrastructure; mocks cannot prove DB behavior. | Backend data/migration guides; database-testing |
| DEPS — dependencies and supply chain | Check actual manifests/lockfiles, support/advisories, relevant vulnerable usage and approved script/provenance policies. Age or a newer release alone is not a vulnerability or upgrade authorization. | Dependency/security guides and actual audit evidence |
| VISUAL — visual accuracy — OUT OF SCOPE | No UI or visual baseline is part of this API-only starter. Record scope exclusion; do not install browser tools. | Backend-only scope |
| USABILITY — API consumer usability | Check scoped API task discoverability, documentation/examples, error clarity and recovery against approved requirements. No UI walkthrough, research participants or client app. | Focused checks; approved API docs and ACs |
| A11Y — UI accessibility — OUT OF SCOPE | No UI/DOM/WCAG audit is part of this API-only starter. Do not turn API documentation into a website assignment. | Backend-only scope |
| COMPAT — service and protocol compatibility | Check actual approved Node/Express/provider, HTTP/media/encoding and consumer contract support; emulating a browser or building client UI is not needed. | Focused checks; actual approved support/contract versions |
| I18N — API localization | Check approved Unicode/locale/fallback and canonical date/number/currency representations; no RTL layout, font or screen audit. | Focused checks; approved API/time-and-money policy |
| PERF — API performance | Measure authorized latency/throughput/resource use against agreed conditions/budgets; load/stress/spike/soak require explicit safe target and authority. | Backend performance/scaling rules; performance-testing; actual plan |
| RELIABILITY — failures and recovery | Exercise bounded timeout/retry/cancellation, partial failure, degraded dependency and recovery behavior; unknown write outcome is not safe automatic retry. | Reliability/error-handling guides; actual operation policy |
| CONCURRENCY — overlapping operations | Check races, simultaneous changes, duplicate delivery, idempotency and state ordering using controlled independent requests and approved data. | Concurrency/idempotency/transaction guides; actual invariants |
| OBS — observability | Check required safe logs/metrics/traces, correlation and error visibility; validate actual alert delivery only through an explicitly authorized test route. | Observability guide; `observability-driven-testing` |
| PRIVACY — personal-data handling | Verify permitted data inventory, minimization, tenant access, retention/deletion/export, logs/files/third parties and restoration behavior against approved policy. | Focused checks; approved backend data/privacy/security requirements |
| RECOVERY — backup and restoration | Demonstrate authorized isolated restore/PITR, integrity and service smoke checks against approved RPO/RTO; backup job success is insufficient. | Focused checks; actual permitted backend disaster-recovery runbook |
| REGRESSION — existing behavior | Recheck previously working affected/shared paths on the actual candidate. Later correction checks remain with Orchestrator under the cycle. | Baseline, existing tests, original failure and task scope |
| EXPLORATORY — bounded exploration | Use a risk-based charter, time/scope boundary, recorded actions and observations; discovery does not replace required AC coverage. | `exploratory-testing`; approved brief |
| CHAOS — controlled fault injection | Use only a specific justified failure hypothesis, safe target, abort conditions and recovery plan; never infer permission from project size or skill availability. | `chaos-engineering`; actual environment permission |
| AI — AI-product behavior | Only for actual AI features: verify approved task quality, unsupported claims, injection/data exposure, tool permissions and failure/cost limits using repeatable datasets and declared nondeterminism. | `ai-system-testing`; actual product requirements |

The imported methods above are reached through [QA catalog](qa-skills-catalog.md) and its adapter, not auto-invoked. Business logic is checked even in small projects; privacy and recovery applicability follows data and loss impact, not project size. AI testing applies to an AI feature in the product, not merely because an AI agent wrote its code.

## How to choose test techniques

- Use equivalence classes, boundary values, decision tables or state transitions where the input or behavior warrants them. Use property-based tests, fuzzing or mutation testing only when a specific invariant/risk and suitable tooling justify the cost.
- A useful test names the defect it would catch. Do not chase a universal coverage percentage or duplicate trivial assertions.
- Backend persistence, migrations, constraints, transactions and concurrency need an isolated real database or other authorized realistic infrastructure. Mock or stub external provider boundaries to avoid side effects; simulate timeout, error and malformed responses as relevant.
- Browser/component/visual/accessibility auditing is outside API-only scope. Use actual permitted HTTP/service/data checks.
- Performance claims need a stated baseline, budget, comparable environment and measurement method. Without these, record a risk or measurement plan, not a performance PASS.
- Keep architecture and maintainability findings tied to adopted rules or concrete defect/change cost; do not treat personal style, another architecture pattern or a tool warning as a confirmed defect.
- For OBS, distinguish inspecting logging configuration, observing an emitted event/trace and proving an alert reached its permitted test destination. Redact credentials/PII; do not page people or contact external recipients without actual authorization.
- Owner manual API acceptance is separate; normal assigned unit/API/integration/contract tests remain valid. Automated full-system E2E requires explicit separate authority.

## Evidence and safety

For every execution, capture the candidate, environment, fixture identity, command/tool, exit code, observed output and timestamp when useful. For a source review, cite both the source rule and actual candidate file/line. Redact secrets and personal data. Do not infer test success from code appearance, stale CI or another candidate's run.

Before any destructive, load-heavy or externally visible check, confirm the brief expressly allows that environment and action. If it does not, write an executable safe test plan and mark execution `BLOCKED` or `NOT RUN`; do not simulate a PASS. Isolate and clean up test data, including across parallel tester operators.

If a requirement is absent, contradictory or untestable, report a `SOURCE/CONTRACT GAP` with a concrete counterexample. Do not rewrite the Planner Output or silently invent expected behavior. If two local sources disagree, quote both and send the conflict to the Orchestrator for resolution.
