# Project implementation map

> Copy to the project's existing canonical map, or `docs/PROJECT-MAP.md` when none exists and that write is authorized. Replace placeholders with inspected facts. This template itself is not an inventory of a running app. Paths below are relative to project root.

## Authority and freshness

| Field | Value |
| --- | --- |
| Canonical map path / maintainer | `<one actual path; Implementer under assigned authority>` |
| Knowledge entry maps | `workspace/docs/frontend-guide/GUIDE-MAP.md`, `workspace/docs/backend-guide/GUIDE-MAP.md`, `workspace/docs/syntax/GUIDE-MAP.md` |
| Approved structure sources | `<frontend/backend project-structure guides; current Planner Output/version and role TODO>` |
| Code revision or inspected working-tree identity / time | `<actual evidence; never a fabricated commit>` |
| Scope verified / known stale or unavailable branches | `<what was inspected; explicit limitations>` |

## Existing project tree

```text
<replace with a compact tree of inspected application paths and responsibilities>
```

List only existing paths. Record missing apps as `NOT INITIALIZED` outside the tree. Exclude dependency/VCS/build/cache/coverage internals and secret-bearing files. Keep proposed additions in the task TODO until created. Summarize large modules and route to their sections rather than listing every function.

## Module and entry-point navigation

| App / area | Actual path or existing module-document link | Responsibility and main entry points | Public interface / relevant consumers | Ownership boundary |
| --- | --- | --- | --- | --- |
| `<observed area>` | `<actual source path>` | `<inspected purpose and entry file>` | `<source-backed interface or NONE>` | `<Implementer source / Tester tests / other established owner>` |

## Cross-app and data boundaries

| Boundary | Producer / definition path | Consumer / entry path | Interface and synchronization route | Inspected source evidence / unresolved gap |
| --- | --- | --- | --- | --- |
| `<actual boundary or NOT INITIALIZED>` | `<path>` | `<path>` | `<HTTP contract, approved type sync, persistence boundary>` | `<path:line or precise missing prerequisite>` |

Describe the actual Next.js → Express/Node.js → database route when present. Do not invent an endpoint, import relationship or synchronization mechanism from a canonical example.

## Run and verification entry locations

| App | Script/config entry locations | Test locations and owners | Detailed instructions / evidence location |
| --- | --- | --- | --- |
| `<observed app or NOT INITIALIZED>` | `<manifest/config/example paths; no secret values>` | `<existing paths and established owners>` | `<existing README/runbook or relevant Output path>` |

## Current structural gaps

| Gap | Actual evidence and affected navigation | Owner / resolution route |
| --- | --- | --- |
| `<observed gap or NONE OBSERVED WITHIN INSPECTED SCOPE>` | `<precise missing/stale reference>` | `<established owner through Orchestrator>` |

Update this canonical document with structural changes. Task history, status, checks and decisions stay in their existing records; do not duplicate them here.
