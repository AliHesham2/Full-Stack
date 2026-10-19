# QA project context — template for an actual project

> Use only when the owner or assigned Orchestrator requests a durable project QA context and supplies an approved path. This file is a reusable template, not a claim that the starter template has a live product. Read the Roster, approved Planner Output, app config and guides first. Mark unknowns `UNKNOWN — owner input needed`; distinguish detected facts, decisions and proposals.

## Product

- Name/type/purpose: `<approved source or UNKNOWN>`
- Development, staging and production URLs: `<actual values or NOT CONFIGURED>`
- Critical user journeys and owner: `<approved Planner Output IDs or UNKNOWN>`

## Tech stack

- Frontend: `<actual framework/version/config path>`
- Backend/API: `<actual framework/version/config path>`
- Database/provider and ownership boundary: `<actual configuration; no credentials>`
- Hosting/CDN/monitoring: `<actual state or NOT CONFIGURED>`

## Test stack

- Unit/component, integration/API, contract, visual, accessibility, performance and E2E tools: `<installed tool/version/config/test path or NONE SELECTED>`
- Manual owner checks: `<approved process>`

## CI/CD

- Platform, triggers, gates, artifacts and suite duration: `<actual state or NOT CONFIGURED>`

## Environments

- Available targets, data isolation, external-service doubles and parity gaps: `<actual state>`
- Prohibited targets/actions: `<owner or host restriction>`

## Quality goals

- Approved measurable budgets/thresholds and their evidence source: `<actual targets or NOT SET>`
- Do not fabricate coverage percentages, flake limits or release gates.

## Risk areas

| Area / Planner Output ID | Impact | Likelihood | Evidence and owner |
| --- | --- | --- | --- |
| `<actual area>` | `<1–5 or UNKNOWN>` | `<1–5 or UNKNOWN>` | `<incident, requirement or decision>` |

## Team and roles

- Approved assignments: `<Roster reference; do not duplicate as authority>`
- Test ownership and escalation: `<brief/Orchestrator route>`

## Conventions

- Test folders and isolation: `<actual app configuration and assignment>`
- Selectors, fixtures, naming, reporting and evidence: `<approved guides/brief>`
- Last verified candidate/date and open unknowns: `<evidence>`
