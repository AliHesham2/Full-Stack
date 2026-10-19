# Tester Output — <task ID> / round <N>

> Copy this template into the task-record path supplied in the authorized brief. Report observed results only. Do not carry PASS results from a different candidate or treat an empty table as a successful check.

## Candidate and scope

| Field | Value |
| --- | --- |
| Task / assigned operator / round | `<IDs>` |
| Phase | `<CONTRACT SPECIFICATION / IMPLEMENTATION; do not conflate document and runtime checks>` |
| Candidate revision/build and tested artifact | `<immutable identifier>` |
| Brief, TODO and Planner Output/contract versions | `<paths or IDs>` |
| Environment, fixtures and execution time | `<reproducible details; no secrets>` |
| Affected apps and checked scope | `<backend paths and functions>` |
| Authorized reporting route | `<destination>` |

## Acceptance-criteria traceability

| AC ID | Expected behavior/source | Check IDs | Result (`PASS`/`FAIL`/`UNVERIFIED`) | Evidence or reason |
| --- | --- | --- | --- | --- |
| `<AC-1>` | `<Planner Output path:line or contract ID>` | `<T-01>` | `<result>` | `<observed result, evidence path, or gap>` |

Every approved AC appears exactly once with phase applicability explicit. A document-phase PASS covers specification checks only; runtime-dependent criteria remain UNVERIFIED with the later implementation dependency linked. `BLOCKED` and `NOT RUN` checks make their affected ACs `UNVERIFIED`; `NOT APPLICABLE` needs a reason and cannot erase an AC. An AC that depends on the owner's pending manual API acceptance remains `UNVERIFIED` here.

## Verification streams

| Stream | Check IDs | Result (`PASS`/`FAIL`/`BLOCKED`/`NOT RUN`/`NOT APPLICABLE`) | Evidence or reason |
| --- | --- | --- | --- |
| Project-guide compliance — backend | `<IDs>` | `<result>` | `<guide sections and candidate locations>` |
| Technology/syntax compliance | `<IDs>` | `<result>` | `<syntax sections and candidate locations>` |
| Format/lint/typecheck/build | `<IDs>` | `<result>` | `<commands and outputs>` |
| Maintainability and duplication | `<IDs>` | `<result>` | `<specific review evidence>` |
| Functional tests | `<IDs>` | `<result>` | `<test evidence>` |
| API/contract and cross-app integration | `<IDs>` | `<result>` | `<request/response evidence>` |
| Security/data/performance/reliability | `<IDs>` | `<result>` | `<measurements or justified exclusions>` |
| Regression | `<IDs>` | `<result>` | `<baseline and current results>` |

## Checks executed

| Check ID | Cwd / command or review method | Exit code and actual result | Evidence location | Candidate |
| --- | --- | --- | --- | --- |
| `<T-01>` | `<app cwd; exact command>` | `<0/nonzero and relevant output>` | `<log/test path or source lines>` | `<revision>` |

## Imported QA methods, if used

| Skill and pinned source | Applied method and evidence | Source guidance not applied and why |
| --- | --- | --- |
| `<qa-skills skill or NONE>` | `<check IDs/evidence>` | `<scope, local-rule conflict, missing tool or NONE>` |

## API and data verification evidence

| Check / API/DB/SEC IDs | Actual method/path and request or schema/data operation | Auth/role/tenant and isolated fixture/environment | Expected and actual status/headers/body or integrity result | Evidence / limits |
| --- | --- | --- | --- | --- |
| `<actual IDs>` | `<synthetic request or permitted operation; no real credentials>` | `<identity class, isolation and authority>` | `<exact observable behavior>` | `<redacted output; real versus doubled boundary>` |

Document actual timeout/provider-failure, duplicate/concurrent write, unauthorized/other-tenant and malformed payload cases as applicable. Real constraint/transaction/migration claims need actual authorized isolated infrastructure; mocks cannot establish them.

## Findings

Use one entry per confirmed defect or source/contract gap; separate hypotheses. Cite exact project rule/AC and candidate file:line. Do not label a style preference a defect.

### `<F-01: concise title>`

- Kind/severity/user impact: `<confirmed defect or SOURCE/CONTRACT GAP; severity and effect>`
- Source: `<Planner Output/rule path:line, section and short exact quote>`
- Candidate location: `<file:line, endpoint or database definition>`
- Environment/preconditions and steps: `<reproducible sequence>`
- Expected / actual: `<observable difference>`
- Evidence: `<test output, log or request/response, redacted>`
- Failure investigation, when needed: `<first bad boundary, experiments and results; cause CONFIRMED / LIKELY / UNKNOWN>`
- Likely affected scope and suggested regression check: `<scope; test>`
- Status on this tested candidate: `<OPEN / NOT REPRODUCED WITH evidence / UNVERIFIED; later Orchestrator correction evidence linked separately>`

## Summary and handoff

Orchestrator independently reviews every result and personally verifies Planner/Implementer corrections. Preserve this original report; do not automatically start a Tester retest or overwrite old results with Orchestrator evidence.

| Item | Result |
| --- | --- |
| ACs | `<pass N / fail N / unverified N>` |
| Checks | `<pass N / fail N / blocked N / not run N / not applicable N>` |
| Defects by severity | `<counts and IDs>` |
| What changed in assigned test area | `<paths or none>` |
| Limitations and residual risks | `<specific blind spots>` |
| Phase recommendation | `CONTRACT SPEC READY FOR IMPLEMENTATION REVIEW / IMPLEMENTATION READY FOR OWNER CHECK / NOT READY / INSUFFICIENT EVIDENCE` — `<phase scope and pending runtime criteria explicit>` |
| Risk acceptance record, if applicable | `<owner/authorized decision reference; otherwise none>` |
| Next owner/action | `<Orchestrator, Implementer fix via Orchestrator, or owner manual API acceptance>` |

If a known risk is accepted, cite its actual acceptance record; the Tester cannot create one. Both phase recommendations require Orchestrator's individual evidence review. Neither accepts a feature or authorizes release; contract specification readiness does not prove runtime behavior.
