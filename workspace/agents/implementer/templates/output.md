# Implementer Output — <task ID> / candidate <identity>

> Copy to the assigned path. Record actual work and checks after the final edit. This report hands over a candidate; it does not accept a feature. Paths are from project root unless explicitly stated otherwise.

## Candidate and assignment

| Field | Value |
| --- | --- |
| Task / operator / approved assignment | `<IDs/reference>` |
| Approved requirements / software TODO / contracts and review | `<current Planner Output/software TODO/version, exact contract/schema/security and contract Tester/Orchestrator gate evidence>` |
| Implementation identity | `<commit or changed-file manifest with hashes/time; branch alone is insufficient>` |
| Affected apps and environment | `<actual versions and permitted data/environment>` |
| Evidence location / report time / reporting route | `<actual values>` |
| Canonical implementation map / final inspected scope | `<path; revision/time; affected paths reconciled and limitations>` |

## Changed

| Item ID / requirement IDs | Changed files and code locations | Implemented behavior and notable decision | Guide file/heading | Local syntax file/heading and version |
| --- | --- | --- | --- | --- |
| `<I-n / actual IDs>` | `<paths:lines>` | `<actual change and rationale>` | `<controlling rule>` | `<technical support; gap stays explicit>` |

### Integration impact

| Contract / schema / types / token / dependency / config | Actual change and affected consumers | Compatibility and deployment/migration ordering | Recovery/irreversible effects | What was executed versus only prepared |
| --- | --- | --- | --- | --- |
| `<identifier or NOT APPLICABLE>` | `<actual paths/impact>` | `<supported sequence or unresolved decision>` | `<evidence-based limitation>` | `<command/environment/result or NOT RUN>` |

Preserved pre-existing edits / unrelated observed issues: `<actual observations and evidence or NONE OBSERVED>`.

## Verified

| Requirement / implementation item | Candidate and actual code/source evidence | Result (`VERIFIED WITHIN CHECKED SCOPE` / `FAILED` / `UNVERIFIED`) | Checks and limitations / independent review needed |
| --- | --- | --- | --- |
| `<AC-n / I-n>` | `<paths:lines and C-n evidence>` | `<actual result>` | `<exact proven scope; remaining Tester/Orchestrator checks>` |

Self-review evidence: `<diff scope; syntax/guide conformity; contract consumers; failure paths; security; duplication; unintended edits>`.

## Checks

| Check ID | Exact command or inspection / working directory | Candidate, environment and time | Exit/result (`PASS` / `FAIL` / `NOT RUN` / `BLOCKED` / `NOT APPLICABLE`) | Retained output/evidence and specific limitation |
| --- | --- | --- | --- | --- |
| `<C-n>` | `<actual command; backend/>` | `<identity/conditions>` | `<actual status>` | `<path/summary; reason if not run>` |

State the final `npm run check` result for each changed app. Identify missing suites, scripts, tools or services. No zero-test run, stale PASS or build alone proves requirement coverage. Do not include secrets in logs.

### Repair evidence, if applicable

| Existing issue ID | Reproduction candidate/steps and expected versus actual | Cause classification and evidence | Product fix / changed locations or routed owner | New candidate, rerun and regression evidence | Remaining uncertainty |
| --- | --- | --- | --- | --- | --- |
| `<actual issue ID or NOT APPLICABLE>` | `<observed failure or not reproduced with conditions>` | `<product / test / environment / requirement / syntax gap>` | `<bounded correction or escalation>` | `<C-n and candidate>` | `<specific gap or NONE>` |

The original Tester report remains candidate-specific. Orchestrator independently reviews each result and personally verifies corrections under the delivery cycle; no automatic Tester return. A local repair does not close its issue. Link Orchestrator correction evidence and preserve earlier reports.

### Runnable handoff and resource disposition

| App | Actual startup command / working directory / ordering | Runtime/config example path, variable names and external prerequisites | Expected URL/port or health observation | Startup status and actual evidence |
| --- | --- | --- | --- | --- |
| `<app or NOT INITIALIZED>` | `<existing approved commands; no invented seed/install command>` | `<names only; no secrets>` | `<configured expectation versus observed result>` | `<OBSERVED RUNNING / FAILED / NOT RUN / BLOCKED; candidate/output/time>` |

| Task-owned resource | Ownership evidence: process/session + project/command, or resolved path | Final state and cleanup evidence/error | Authorized reason and next owner/route if retained | Safe shutdown/removal instruction |
| --- | --- | --- | --- | --- |
| `<actual resource or NONE CREATED>` | `<verified identity; never port/name alone>` | `<STOPPED / REMOVED / RETAINED / CLEANUP FAILED / OWNERSHIP UNVERIFIED>` | `<decision/reference or NOT APPLICABLE>` | `<bounded operation or unresolved gap>` |

Preserved evidence / untouched pre-existing services: `<relevant paths/identities or NONE OBSERVED>`.
Project-map update: `<structural changes reflected, inspected scope/time, links checked, or specific stale branch>`.

## Remaining and handoff

| Gap / question / risk ID | Affected requirement and precise limitation | Evidence and required resolution | Owner / authorized route | State |
| --- | --- | --- | --- | --- |
| `<actual ID or NONE FOUND WITHIN CHECKED SCOPE>` | `<not implemented, unsupported source, not verified, or blocked>` | `<next required input/check>` | `<Orchestrator and established owner>` | `<actual state>` |

- Tester handoff: `<candidate; requirements/files/contracts; failure and boundary cases; commands/data; missing coverage>`.
- API handoff: `<complete operation catalog, canonical contract/version, synthetic request/success/failure examples, unified errors and auth rules, actual startup/env names and checked limits; no UI>`.
- Requirement/architecture decisions needed: `<precise question and source evidence or NONE>`.
- Recommendation to Orchestrator: `<candidate ready for independent review / revision needed / blocked, with reasons>`.
- Next authorized action / resume point: `<concrete step and record path>`.

Code written, developer checks executed, independent verification and owner acceptance are separate states. Report each only with its actual evidence.
