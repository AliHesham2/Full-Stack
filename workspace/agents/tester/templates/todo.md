# Tester TODO — <task ID> / round <N>

> Copy this template into the task-record path supplied in the authorized brief. Replace placeholders; remove illustrative rows. Write it before executing tests and send it to the assigned Orchestrator for review. A TODO is a plan, not evidence of a PASS.

## Target and authority

| Field | Value |
| --- | --- |
| Feature/task and operator | `<ID>` / `<approved operator>` |
| Brief and assignment reference | `<paths or IDs>` |
| Phase | `<DESIGN SPECIFICATION / IMPLEMENTATION; do not conflate document and runtime checks>` |
| Candidate | `<commit, build ID or immutable artifact>` |
| Approved Planner Output/ACs and contracts | `<paths, versions, IDs>` |
| Affected apps/files/functions | `<specific scope>` |
| Allowed environment/data and prohibited actions | `<explicit limits>` |
| Test-writing area and output path | `<paths from brief>` |
| Reporting route | `<authorized destination>` |

## Source selection

| Source | Relevant section/rule | Why applicable |
| --- | --- | --- |
| `<frontend-guide, supplied API source, syntax, Planner Output, design or contract path>` | `<heading/ID>` | `<changed behavior or file>` |

## Stream applicability and scope

Screen every ID in [verification matrix](../skillz/verification.md). Instantiate actual scoped rows, or group IDs sharing a justified exclusion; do not require an executed test suite for every stream. Missing tooling/access/policy is a blocker for required checks, not an exclusion. Preserve phase distinction: reviewing a specified control does not prove its runtime enforcement.

| Stream ID / affected surface and phase | Applicable risk/requirement, or exclusion reason | Guide/contract/policy version and method | Planned check IDs / required evidence | Decision, access or tool gap / revisit trigger |
| --- | --- | --- | --- | --- |
| `<actual ID(s); frontend/document/runtime>` | `<named defect class or justified NOT APPLICABLE>` | `<exact source/section; no invented targets>` | `<check references or exclusion>` | `<precise blocker/owner or NONE>` |

For USABILITY, I18N, PRIVACY and RECOVERY follow [focused checks](../skillz/focused-checks.md). Plan the actual flow/support/locale/data/restore matrix, comparison baseline, expected outcomes and permitted environment; link it here or include it below without a parallel status record. No permission to contact participants, read production PII, restore a backup, send an alert or create infrastructure follows from selecting a stream.

## Security requirement and method selection — when applicable

Use [versioned security baseline](../skillz/security-baseline.md). Pin actual approved sources/assurance scope; reading a standard does not select an ASVS level or change existing policy. Remove only with a justified SECURITY exclusion recorded above.

| Project SEC/AC and affected surface | Actual standard/version, exact requirement ID and source | Selected method/version/section and expected observable outcome | Planned check/environment/permission and limitation |
| --- | --- | --- | --- |
| `<actual IDs>` | `<verified version-qualified ASVS ID; project-specific control with reason if no mapping>` | `<actual WSTG version/ID when relevant; no invented crosswalk>` | `<check IDs; document/runtime distinction; precise gaps>` |

## Imported QA methods, if used

| Skill and pinned source path | Relevant sections/references | Planned use | Omitted or translated guidance and reason |
| --- | --- | --- | --- |
| `<qa-skills skill or NONE>` | `<sections read>` | `<specific check/design method>` | `<e.g. provider-specific setup, unauthorized E2E, or NONE>` |

## Planned verification

| Check ID | AC/rule ID | Risk and defect class | Stream / method | Expected oracle | Environment/data | Planned evidence | State |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `T-01` | `<AC-1 or rule>` | `<impact × likelihood; defect>` | `<unit, integration, review, etc.>` | `<observable expected result>` | `<safe fixture/environment>` | `<command, output, trace or source comparison>` | `TODO` |

Include at least one route to verify each applicable approved AC. Cover meaningful negative, boundary, failure and regression cases. Choose project-guide and syntax sections by the actual changed stack; API integration checks require the frontend guide and exact owner-supplied contract versions; no backend guide or backend implementation is required. Record any dependency between checks.

## Exclusions and blockers

| Stream/check | `NOT APPLICABLE`, `BLOCKED` or deferred | Reason and trigger to revisit |
| --- | --- | --- |
| `<item>` | `<state>` | `<specific explanation>` |

## Questions or source conflicts

| ID | Exact conflicting/missing source and counterexample | Sent to | State |
| --- | --- | --- | --- |
| `<Q-1>` | `<path:line, quote and impact>` | `<Orchestrator>` | `OPEN` |
