# Backend-only starter workspace

Start at [Roster](rostar/roster.md), the assigned role prompt and Skill, then mandatory [backend-only scope](docs/backend-guide/00-navigation/backend-only-scope.md). No operator is assigned by this migration.

| Entry | Purpose |
| --- | --- |
| [Backend GUIDE-MAP](docs/backend-guide/GUIDE-MAP.md) | Engineering reading flow and complete file catalog. |
| [Syntax GUIDE-MAP](docs/syntax/GUIDE-MAP.md) | Express, Node.js, JavaScript and Supabase reference routes; exclusive local implementation policy. |
| [Delivery cycle](agents/orchestrator/skillz/delivery-cycle.md) | Planning, contract specification review, implementation, API verification and owner acceptance. |
| [Bridge entry](bridge.md) | Required only before actually authorized communication; guide-only external access boundary. |

## Product and artifacts

The sole future app is backend/: APIs with necessary domain/data/auth/validation logic, not a frontend. Planner receives owner requirements through [input](agents/planner/input/README.md), proposes complete API contracts/schema/security and prepares one software TODO plus its own internal TODO and Output. Owner approvals remain explicit. Implementer validates/continues the approved software TODO using local syntax; its project map describes actual backend code. There is no Designer, visual TODO, design guide or UI test gate.

Tester independently reviews contracts before code and verifies real API behavior afterward. Orchestrator reviews each result and personally verifies Planner/Implementer corrections; no automatic Tester return. Advisor is optional/nonbinding. Every active role has TODO/Output templates, exact candidate evidence and resume rules. Imported QA references retain their full source/licenses, while browser/UI/mobile techniques are excluded by the active adapter.

Final delivery includes the API catalog, complete versioned contract (OpenAPI when approved/source-supported), request/success/failure examples, unified errors, applicable schema/diagram/migration/recovery, security/auth rules, runnable setup/env names and real verification/limitations. Planner Output ends with the simplified Arabic owner explanation. No Swagger website, browser client or frontend is implied by API documentation.

The existing backend syntax and learning records were retained; obsolete client-specific workflow scaffolding was removed without a legacy archive. Supabase remains a documented provider baseline, not an assumed live database. No application, credentials, provider setup, role dispatch or runtime PASS is fabricated. Actual supported versions/source coverage must be established for real work; migration is not a blanket latest-release certification.
