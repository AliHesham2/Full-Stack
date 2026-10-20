# Tester Output — <task ID> / round <N>

> Copy this template into the task-record path supplied in the authorized brief. Report observed results only. Do not carry PASS results from a different candidate or treat an empty table as a successful check.

## Candidate and scope

| Field | Value |
| --- | --- |
| Task / assigned operator / round | `<IDs>` |
| Phase | `<DESIGN SPECIFICATION / IMPLEMENTATION; do not conflate document and runtime checks>` |
| Candidate revision/build and tested artifact | `<immutable identifier>` |
| Brief, TODO and Planner Output/contract versions | `<paths or IDs>` |
| Environment, fixtures and execution time | `<reproducible details; no secrets>` |
| Affected apps and checked scope | `<frontend paths and functions>` |
| Authorized reporting route | `<destination>` |

## Acceptance-criteria traceability

| AC ID | Expected behavior/source | Check IDs | Result (`PASS`/`FAIL`/`UNVERIFIED`) | Evidence or reason |
| --- | --- | --- | --- | --- |
| `<AC-1>` | `<Planner Output path:line or contract ID>` | `<T-01>` | `<result>` | `<observed result, evidence path, or gap>` |

Every approved AC appears exactly once with phase applicability explicit. A document-phase PASS covers specification checks only; runtime-dependent criteria remain UNVERIFIED with the later implementation dependency linked. `BLOCKED` and `NOT RUN` checks make their affected ACs `UNVERIFIED`; `NOT APPLICABLE` needs a reason and cannot erase an AC. An AC that depends on the owner's pending manual E2E remains `UNVERIFIED` here.

## Verification streams

Use the stable IDs from [verification matrix](../skillz/verification.md). Record each applicable stream separately; shared evidence links are allowed. Keep justified excluded IDs visible, grouping exclusions with one reason when useful rather than fabricating executions. No stream is a new agent or compulsory suite. A failed required check prevents PASS; missing required evidence is BLOCKED/NOT RUN with affected ACs UNVERIFIED. A document PASS is not runtime verification; split scope/phase rows when necessary.

| Stream ID / checked surface | Check IDs | Phase and exact checked scope | Result (`PASS`/`FAIL`/`BLOCKED`/`NOT RUN`/`NOT APPLICABLE`) | Evidence, limitation or exclusion reason |
| --- | --- | --- | --- | --- |
| GUIDE / frontend | `<IDs or justified exclusion>` | `<document/runtime; actual scope>` | `<actual result>` | `<source/check/evidence or reason>` |
| SYNTAX — local reference conformity | `<IDs or justified exclusion>` | `<document/runtime; actual scope>` | `<actual result>` | `<source/check/evidence or reason>` |
| HEALTH — configured code gates | `<IDs or justified exclusion>` | `<document/runtime; actual scope>` | `<actual result>` | `<source/check/evidence or reason>` |
| STATIC — analysis and maintainability | `<IDs or justified exclusion>` | `<document/runtime; actual scope>` | `<actual result>` | `<source/check/evidence or reason>` |
| ARCH — architecture | `<IDs or justified exclusion>` | `<document/runtime; actual scope>` | `<actual result>` | `<source/check/evidence or reason>` |
| UNIT — isolated behavior | `<IDs or justified exclusion>` | `<document/runtime; actual scope>` | `<actual result>` | `<source/check/evidence or reason>` |
| LOGIC — business rules | `<IDs or justified exclusion>` | `<document/runtime; actual scope>` | `<actual result>` | `<source/check/evidence or reason>` |
| INTEGRATION — collaborating parts | `<IDs or justified exclusion>` | `<document/runtime; actual scope>` | `<actual result>` | `<source/check/evidence or reason>` |
| API — HTTP behavior | `<IDs or justified exclusion>` | `<document/runtime; actual scope>` | `<actual result>` | `<source/check/evidence or reason>` |
| CONTRACT — consumer compatibility | `<IDs or justified exclusion>` | `<document/runtime; actual scope>` | `<actual result>` | `<source/check/evidence or reason>` |
| SECURITY — frontend technical controls | `<IDs or justified exclusion>` | `<document/runtime; actual scope>` | `<actual result>` | `<source/check/evidence or reason>` |
| DATA — frontend DTO and state | `<IDs or justified exclusion>` | `<document/runtime; actual scope>` | `<actual result>` | `<source/check/evidence or reason>` |
| DEPS — dependencies and supply chain | `<IDs or justified exclusion>` | `<document/runtime; actual scope>` | `<actual result>` | `<source/check/evidence or reason>` |
| VISUAL — visual accuracy | `<IDs or justified exclusion>` | `<document/runtime; actual scope>` | `<actual result>` | `<source/check/evidence or reason>` |
| USABILITY — task usability | `<IDs or justified exclusion>` | `<document/runtime; actual scope>` | `<actual result>` | `<source/check/evidence or reason>` |
| A11Y — accessibility | `<IDs or justified exclusion>` | `<document/runtime; actual scope>` | `<actual result>` | `<source/check/evidence or reason>` |
| COMPAT — compatibility | `<IDs or justified exclusion>` | `<document/runtime; actual scope>` | `<actual result>` | `<source/check/evidence or reason>` |
| I18N — localization | `<IDs or justified exclusion>` | `<document/runtime; actual scope>` | `<actual result>` | `<source/check/evidence or reason>` |
| PERF — frontend performance | `<IDs or justified exclusion>` | `<document/runtime; actual scope>` | `<actual result>` | `<source/check/evidence or reason>` |
| RELIABILITY — frontend failures and recovery | `<IDs or justified exclusion>` | `<document/runtime; actual scope>` | `<actual result>` | `<source/check/evidence or reason>` |
| CONCURRENCY — overlapping frontend operations | `<IDs or justified exclusion>` | `<document/runtime; actual scope>` | `<actual result>` | `<source/check/evidence or reason>` |
| OBS — frontend observability | `<IDs or justified exclusion>` | `<document/runtime; actual scope>` | `<actual result>` | `<source/check/evidence or reason>` |
| PRIVACY — personal-data handling | `<IDs or justified exclusion>` | `<document/runtime; actual scope>` | `<actual result>` | `<source/check/evidence or reason>` |
| RECOVERY — frontend artifact and config recovery | `<IDs or justified exclusion>` | `<document/runtime; actual scope>` | `<actual result>` | `<source/check/evidence or reason>` |
| REGRESSION — existing behavior | `<IDs or justified exclusion>` | `<document/runtime; actual scope>` | `<actual result>` | `<source/check/evidence or reason>` |
| EXPLORATORY — bounded exploration | `<IDs or justified exclusion>` | `<document/runtime; actual scope>` | `<actual result>` | `<source/check/evidence or reason>` |
| CHAOS — controlled fault injection | `<IDs or justified exclusion>` | `<document/runtime; actual scope>` | `<actual result>` | `<source/check/evidence or reason>` |
| AI — AI-product behavior | `<IDs or justified exclusion>` | `<document/runtime; actual scope>` | `<actual result>` | `<source/check/evidence or reason>` |

## Checks executed

| Check ID | Cwd / command or review method | Exit code and actual result | Evidence location | Candidate |
| --- | --- | --- | --- | --- |
| `<T-01>` | `<app cwd; exact command>` | `<0/nonzero and relevant output>` | `<log/test path or source lines>` | `<revision>` |

## Imported QA methods, if used

| Skill and pinned source | Applied method and evidence | Source guidance not applied and why |
| --- | --- | --- |
| `<qa-skills skill or NONE>` | `<check IDs/evidence>` | `<scope, local-rule conflict, missing tool or NONE>` |

## axe-core scan coverage, if used

| Check ID / route or component / visible state | Candidate, browser or JSDOM, engine/wrapper versions | Context and rule tags | Violations / incomplete / passes / inapplicable | Unscanned areas and manual follow-up |
| --- | --- | --- | --- | --- |
| `<T-axe>` | `<revision, environment, versions>` | `<scope and enabled tags>` | `<counts, relevant rule IDs/elements>` | `<review owner/evidence or NONE>` |

## Browser inspection and Lighthouse evidence, if used

| Check ID / tool | Candidate and exact route/state | Session or audit configuration | Observed result and artifact | Coverage gap or comparison limit |
| --- | --- | --- | --- | --- |
| `<T-browser>` | `<build, URL, account/fixture, before/after state>` | `<MCP version, browser, viewport, isolated/profile ID, action sequence>` | `<snapshot, console/network evidence or screenshot path>` | `<untested states or NONE>` |
| `<T-lighthouse>` | `<build, final audited URL, content state>` | `<Lighthouse/browser versions, device, throttling, environment, run count>` | `<individual reports, values, approved budget, summary and spread>` | `<baseline, field-data or other gap; NONE if resolved>` |

## Focused verification evidence — when applicable

Follow [focused checks](../skillz/focused-checks.md). Use this table as a scoped evidence index; detailed cases may remain in their linked check artifacts. Remove a row only when its ID has a recorded exclusion above. Describe actual observations, not template examples presented as successful checks.

| Stream / check IDs | Actual baseline, case matrix and policy/version | Method, conditions and evidence | Scope/coverage limits and unresolved prerequisite |
| --- | --- | --- | --- |
| VISUAL / USABILITY / COMPAT | `<flow/AC/design versions; browser/viewport/theme/content/input and actual device matrix>` | `<comparable renders, heuristic walkthrough or actual authorized user study; distinguish these; expected versus observed>` | `<unobserved states/devices; no invented user measures or visual tolerance>` |
| I18N | `<approved locale/direction/timezone/currency; boundary inputs and expected formats>` | `<actual render/API results, language/fallback/mixed-text and relevant time/rounding cases>` | `<translation review limits; untested locales; no changed wire contract>` |
| PRIVACY | `<approved data inventory/policy; synthetic fixture, actor/tenant, allowed frontend surfaces and lifecycle requests>` | `<actual exposure/access/retention/deletion/export observations, time and redacted evidence>` | `<external service/backup enforcement outside scope; technical evidence is not legal sign-off>` |
| RECOVERY | `<approved frontend recovery plan/build/config artifact and isolated target; no DB/PITR>` | `<actual frontend artifact/config recovery steps/time and startup/adapter checks; isolation/cleanup evidence>` | `<measured versus estimated; frontend artifact recovery versus external service recovery; unavailable dependencies or NOT RUN>` |

## Security requirement coverage — when applicable

Use [security baseline](../skillz/security-baseline.md). Every selected requirement links to actual checks and evidence; no whole-standard claim follows from partial coverage or a clean scan. Keep approved scope and excluded/unverified requirements explicit.

| SEC/AC / exact standard version and requirement ID | Candidate/surface and method, including actual WSTG version/ID when used | Check IDs and actual result/evidence | Untested conditions / policy, environment or assurance limitation |
| --- | --- | --- | --- |
| `<actual approved project control and verified source identifier>` | `<document/runtime; exact method and conditions>` | `<linked observed results; no invented mappings>` | `<explicit partial coverage or NONE WITHIN CHECKED SCOPE>` |

## Findings

Use one entry per confirmed defect or source/design gap; separate hypotheses. Cite exact project rule/AC and candidate file:line. Do not label a style preference a defect.

### `<F-01: concise title>`

- Kind/severity/user impact: `<confirmed defect or SOURCE/DESIGN GAP; severity and effect>`
- Source: `<Planner Output/rule path:line, section and short exact quote>`
- Candidate location: `<file:line, endpoint or screen>`
- Environment/preconditions and steps: `<reproducible sequence>`
- Expected / actual: `<observable difference>`
- Evidence: `<test output, log, screenshot or request/response, redacted>`
- Failure investigation, when needed: `<first bad boundary, experiments and results; cause CONFIRMED / LIKELY / UNKNOWN>`
- Likely affected scope and suggested regression check: `<scope; test>`
- Status on this tested candidate: `<OPEN / NOT REPRODUCED WITH evidence / UNVERIFIED; later Orchestrator correction evidence linked separately>`

## Summary and handoff

Orchestrator independently reviews every result and personally verifies Designer/Implementer corrections. Preserve this original report; do not automatically start a Tester retest or overwrite old results with Orchestrator evidence.

| Item | Result |
| --- | --- |
| ACs | `<pass N / fail N / unverified N>` |
| Checks | `<pass N / fail N / blocked N / not run N / not applicable N>` |
| Defects by severity | `<counts and IDs>` |
| What changed in assigned test area | `<paths or none>` |
| Limitations and residual risks | `<specific blind spots>` |
| Phase recommendation | `DESIGN SPEC READY FOR IMPLEMENTATION REVIEW / IMPLEMENTATION READY FOR OWNER CHECK / NOT READY / INSUFFICIENT EVIDENCE` — `<phase scope and pending runtime criteria explicit>` |
| Risk acceptance record, if applicable | `<owner/authorized decision reference; otherwise none>` |
| Next owner/action | `<Orchestrator, Implementer fix via Orchestrator, or owner manual E2E>` |

If a known risk is accepted, cite its actual acceptance record; the Tester cannot create one. Both phase recommendations require Orchestrator's individual evidence review. Neither accepts a feature or authorizes release; design specification readiness does not prove runtime behavior.
