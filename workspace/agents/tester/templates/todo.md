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

## Imported QA methods, if used

| Skill and pinned source path | Relevant sections/references | Planned use | Omitted or translated guidance and reason |
| --- | --- | --- | --- |
| `<qa-skills skill or NONE>` | `<sections read>` | `<specific check/design method>` | `<e.g. provider-specific setup, unauthorized E2E, or NONE>` |

## Planned verification

| Check ID | AC/rule ID | Risk and defect class | Stream / method | Expected oracle | Environment/data | Planned evidence | State |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `T-01` | `<AC-1 or rule>` | `<impact × likelihood; defect>` | `<unit, integration, review, etc.>` | `<observable expected result>` | `<safe fixture/environment>` | `<command, output, trace or source comparison>` | `TODO` |

Include at least one route to verify each applicable approved AC. Cover meaningful negative, boundary, failure and regression cases. Choose project-guide and syntax sections by the actual changed stack; API/cross-app changes require both project guides. Record any dependency between checks.

## Exclusions and blockers

| Stream/check | `NOT APPLICABLE`, `BLOCKED` or deferred | Reason and trigger to revisit |
| --- | --- | --- |
| `<item>` | `<state>` | `<specific explanation>` |

## Questions or source conflicts

| ID | Exact conflicting/missing source and counterexample | Sent to | State |
| --- | --- | --- | --- |
| `<Q-1>` | `<path:line, quote and impact>` | `<Orchestrator>` | `OPEN` |
