# Verification matrix and execution rules

Select checks based on the changed surface, defect likelihood and user impact. These are evidence streams within the assigned Tester role, not separate agents, compulsory tool installations or consecutive workflow stages. Screen applicability before execution, using the stable IDs below in TODO and Output. Give each applicable stream its own result and evidence; a passing build or security scan cannot stand in for another stream. Related checks may share evidence without duplicating tests.

Keep this proportional: justify scope-based exclusions, grouping exclusions with the same reason when useful, and record the excluded IDs. Missing access, tools, requirements or implementation is `BLOCKED`/`NOT RUN`, not `NOT APPLICABLE`. In DESIGN SPECIFICATION, label document checks separately; a document PASS cannot satisfy runtime criteria. No stream changes the shared delivery cycle or authorizes another Tester round after corrections.

| ID / stream | Target and evidence | Primary reading route |
| --- | --- | --- |
| GUIDE — project rules | Check actual frontend/backend guide constraints; cite violated rule and candidate line. A cross-app change needs both applicable guides. | Both engineering maps and actual relevant sections |
| SYNTAX — local reference conformity | Compare technical constructs with exact compatible local syntax sections. Successful parsing/typing does not prove this conformity. | `workspace/docs/syntax/`; Implementer's syntax policy |
| HEALTH — configured code gates | Run actual format/lint/typecheck/compile/build gates and `npm run check` when defined; record command, cwd, candidate and exit/output. | Actual package scripts; initialization/code-quality guides |
| STATIC — analysis and maintainability | Review likely defects, dead code, duplication, names, complexity and misplaced responsibility. Distinguish tool findings from manual judgment and investigate false positives. | Code-quality, complexity-control and anti-pattern guides |
| ARCH — architecture | Verify dependency direction, module ownership, separation of concerns and approved boundaries. Do not require a named architecture style the project did not adopt. | Actual architecture/project-structure guides and approved decisions |
| UNIT — isolated behavior | Exercise meaningful function/service/component behavior, negative and boundary cases; no test-per-function quota or tests of library internals. | Actual test strategy and `unit-testing` reference |
| LOGIC — business rules | Verify requirement-derived decisions, invariants, permissions and state transitions. This is core functional coverage whenever business rules exist, not a large-project extra. | Approved Planner Output/ACs and actual domain rules |
| INTEGRATION — collaborating parts | Exercise real routing/service/data interactions at the intended boundary; distinguish real dependencies from doubles. | Backend test strategy; `service-virtualization` reference where needed |
| API — HTTP behavior | Assert actual method/path, validation, status, headers, serialized success/error and negative outcomes for scoped operations. | Approved API definitions; backend API/validation/auth guides |
| CONTRACT — consumer compatibility | Compare producer and consumer fields/types/nullability, status/error semantics, pagination and version evolution; types alone are insufficient. | Exact canonical contracts and actual consumers; `contract-testing` |
| SECURITY — technical controls | Verify relevant identity, object/tenant authorization, injection, secrets, sessions, abuse and trust-boundary controls with versioned requirements and methods. | [Security baseline](security-baseline.md); actual security/auth guides |
| DATA — persistence and migrations | Verify constraints, relationships, transactions, access, migrations/backfills, concurrency-related integrity and recovery assumptions against approved schema. Mocks cannot prove DB behavior. | Data/migration guides; `database-testing` |
| DEPS — dependencies and supply chain | Check actual manifests/lockfiles, support/advisories, relevant vulnerable usage and approved script/provenance policies. Age or a newer release alone is not a vulnerability or upgrade authorization. | Dependency/security guides and actual audit evidence |
| VISUAL — visual accuracy | Compare observed UI with approved design/tokens at equivalent content, viewport, theme and state; use approved tolerances, not an invented percentage score. | Actual design TODO; `visual-testing`; [focused checks](focused-checks.md) |
| USABILITY — task usability | Inspect task clarity, navigation, feedback, errors and recovery separately from visual match. Disclose heuristic inspection versus actual user-study evidence. | Approved flows/ACs; [focused checks](focused-checks.md) |
| A11Y — accessibility | Check applicable semantics, keyboard/focus, reading order, contrast, resizing and announcements against the actual approved WCAG target; automated results do not prove full conformance. | Accessibility guide; [axe procedure](axe-core-results.md) |
| COMPAT — compatibility | Record actual browser/engine, viewport, theme, input and device coverage; responsive emulation is not physical-device testing. | Approved support matrix; `cross-browser-testing`; [focused checks](focused-checks.md) |
| I18N — localization | Check relevant languages, RTL/mixed content, plural/long text, formatting, timezone/currency and locale transitions without changing canonical wire formats. | [Focused checks](focused-checks.md); approved locale policy |
| PERF — performance | Measure relevant latency/throughput/resource or UI metrics against approved budgets and comparable conditions. Load/stress/spike/soak require actual target and authority. | Performance guides; `performance-testing`; Lighthouse procedure when applicable |
| RELIABILITY — failures and recovery | Exercise bounded timeout/retry/cancellation, partial failure, degraded dependency and recovery behavior; unknown write outcome is not safe automatic retry. | Reliability/error-handling guides; actual operation policy |
| CONCURRENCY — overlapping operations | Check races, simultaneous changes, duplicate delivery, idempotency and state ordering using controlled independent requests and approved data. | Concurrency/idempotency/transaction guides; actual invariants |
| OBS — observability | Check required safe logs/metrics/traces, correlation and error visibility; validate actual alert delivery only through an explicitly authorized test route. | Observability guide; `observability-driven-testing` |
| PRIVACY — personal-data handling | Verify minimization, access, retention/deletion/export, logs/caches/third parties and restoration effects against approved policy; no legal-compliance certification. | [Focused checks](focused-checks.md); privacy/data guides |
| RECOVERY — backup and restoration | Demonstrate permitted isolated restore, integrity and service usability against approved RPO/RTO. Backup-job success or a runbook is not restoration evidence. | [Focused checks](focused-checks.md); disaster-recovery guide |
| REGRESSION — existing behavior | Recheck previously working affected/shared paths on the actual candidate. Later correction checks remain with Orchestrator under the cycle. | Baseline, existing tests, original failure and task scope |
| EXPLORATORY — bounded exploration | Use a risk-based charter, time/scope boundary, recorded actions and observations; discovery does not replace required AC coverage. | `exploratory-testing`; approved brief |
| CHAOS — controlled fault injection | Use only a specific justified failure hypothesis, safe target, abort conditions and recovery plan; never infer permission from project size or skill availability. | `chaos-engineering`; actual environment permission |
| AI — AI-product behavior | Only for actual AI features: verify approved task quality, unsupported claims, injection/data exposure, tool permissions and failure/cost limits using repeatable datasets and declared nondeterminism. | `ai-system-testing`; actual product requirements |

The imported methods above are reached through [QA catalog](qa-skills-catalog.md) and its adapter, not auto-invoked. Business logic is checked even in small projects; privacy and recovery applicability follows data and loss impact, not project size. AI testing applies to an AI feature in the product, not merely because an AI agent wrote its code.

## How to choose test techniques

- Use equivalence classes, boundary values, decision tables or state transitions where the input or behavior warrants them. Use property-based tests, fuzzing or mutation testing only when a specific invariant/risk and suitable tooling justify the cost.
- A useful test names the defect it would catch. Do not chase a universal coverage percentage or duplicate trivial assertions.
- Backend persistence, migrations, constraints, transactions and concurrency need an isolated real database or other authorized realistic infrastructure. Mock or stub external provider boundaries to avoid side effects; simulate timeout, error and malformed responses as relevant.
- Component/accessibility tools supplement manual inspection where that inspection is in scope. A passing automated accessibility check does not prove keyboard, reading order or visual usability.
- Performance claims need a stated baseline, budget, comparable environment and measurement method. Without these, record a risk or measurement plan, not a performance PASS.
- Keep architecture and maintainability findings tied to adopted rules or concrete defect/change cost; do not treat personal style, another architecture pattern or a tool warning as a confirmed defect.
- For OBS, distinguish inspecting logging configuration, observing an emitted event/trace and proving an alert reached its permitted test destination. Redact credentials/PII; do not page people or contact external recipients without actual authorization.
- The owner's manual E2E is separate from worker verification. Automated E2E is conditional on an explicit request; the frontend guide does not install it by default.

## Evidence and safety

For every execution, capture the candidate, environment, fixture identity, command/tool, exit code, observed output and timestamp when useful. For a source review, cite both the source rule and actual candidate file/line. Redact secrets and personal data. Do not infer test success from code appearance, stale CI or another candidate's run.

Before any destructive, load-heavy or externally visible check, confirm the brief expressly allows that environment and action. If it does not, write an executable safe test plan and mark execution `BLOCKED` or `NOT RUN`; do not simulate a PASS. Isolate and clean up test data, including across parallel tester operators.

If a requirement is absent, contradictory or untestable, report a `SOURCE/DESIGN GAP` with a concrete counterexample. Do not rewrite the Planner Output or silently invent expected behavior. If two local sources disagree, quote both and send the conflict to the Orchestrator for resolution.
