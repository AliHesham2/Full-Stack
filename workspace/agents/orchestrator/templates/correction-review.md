# Orchestrator correction verification — <task / phase / issue IDs>

> Store this evidence at the authorized path and link it from the single control record. It is not a new status authority. Complete only for actual reviews; original Tester reports remain unchanged.

| Field | Actual record |
| --- | --- |
| Phase / original Tester report and candidate | `<CONTRACT SPECIFICATION or IMPLEMENTATION; path/version>` |
| Issue / AC / exact source and original expected versus actual | `<stable IDs; cited locations and evidence>` |
| Orchestrator's independent disposition | `<CONFIRMED / DISPUTED WITH EVIDENCE / UNVERIFIED; method and evidence>` |
| Formal correction brief / responsible Planner or Implementer | `<actual reference and allowed scope>` |
| Corrected candidate and baseline versions | `<artifact identity or code revision/manifest>` |
| Responsible operator's change report | `<actual changed locations and claimed checks; still claims until reviewed>` |

## Personally performed verification

| Check ID / issue or shared regression scope | Method / exact command and cwd or artifact inspection | Candidate, environment and expected oracle | Actual result and evidence | Limitation / next action |
| --- | --- | --- | --- | --- |
| `<actual check>` | `<original reproducer, contract/schema consistency review, runtime check or existing suite>` | `<identity and conditions>` | `<PASS / FAIL / UNVERIFIED; actual output/capture>` | `<specific gap or NONE>` |

- Affected prior evidence invalidated / shared scope rechecked: `<references>`
- Gate disposition: `<VERIFIED BY ORCHESTRATOR WITHIN NAMED SCOPE / REWORK / BLOCKED>`
- Residual risk / missing evidence / owner decision: `<actual details>`
- Control record update / next authorized action: `<link and action>`

Do not infer runtime success from design documents or source inspection. Do not call this a Tester retest, edit original reports/tests, dispatch another Tester automatically, or close an issue from the correcting operator's claim alone.
