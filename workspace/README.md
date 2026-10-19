# Frontend-only starter workspace

Start at [Roster](rostar/roster.md), then the assigned role prompt and Skill. Assignments remain blank until the owner approves them; no model has a permanent responsibility. This is reusable workflow/reference material, not a running frontend application.

## Required reading routes

| Entry | Purpose |
| --- | --- |
| [Frontend-only scope](docs/frontend-guide/00-navigation/frontend-only-scope.md) | Mandatory for every role: external API contract authority and frontend limits. |
| [Frontend map](docs/frontend-guide/GUIDE-MAP.md) | Engineering flowchart and complete file catalog. |
| [Design map](docs/design-guide/GUIDE-MAP.md) | Design foundations, components, visual and accessibility methods. |
| [Syntax map](docs/syntax/GUIDE-MAP.md) | React/Next.js, JavaScript and Tailwind references; exclusive implementation source policy. |
| [Delivery cycle](agents/orchestrator/skillz/delivery-cycle.md) | Planner → design/spec verification → implementation verification → owner acceptance; Orchestrator verifies corrections personally. |
| [Bridge entry](bridge.md) | Read only before actually authorized communication; external guide-only access restriction. |

## API input and artifacts

The owner supplies detailed external APIs to [Planner input](agents/planner/input/README.md) or an explicitly assigned location. Planner checks/maps the supplied contract and prepares one software TODO; it does not design backend endpoints or DB schemas. Missing operations/fields/policy return to the owner. Designer authors the separate detailed design TODO with the shared Design System. Implementer consumes both approved textual TODOs using local syntax; its project map describes actual frontend application code. Tester supplies candidate-specific evidence; its fixtures never prove real external service behavior.

Every role starts with its prescribed TODO and reports through its Output template. Create actual task records only for authorized work; do not populate fictional examples/approvals. Optional Designer concept images remain owner-only and are excluded from the implementation package without mention. Advisor is optional and nonbinding.

## Preserved material

Existing frontend syntax and learning records were preserved. Imported role reference snapshots and licenses were copied unchanged; active adapters constrain selection to frontend work. The owner requested removal of the obsolete template archive after migration; only the active workflow remains. No runtime trial, installation, real API call, role assignment or Git push follows from migration. Preserved reference contents are not a fresh audit of every upstream stable version; a source/version gap must be resolved before dependent implementation.

See [migration evidence and limits](MIGRATION-REPORT.md) for the source revision, preservation checks and current local state.
