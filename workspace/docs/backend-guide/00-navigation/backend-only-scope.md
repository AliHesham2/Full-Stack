# Backend-only scope and API delivery authority

## Mandatory scope for every role

This reusable starter produces APIs only. Its sole app is `backend/`: Express on Node.js with its own package, lockfile, tests and deploy boundary. External callers are other projects. No frontend app, Next.js code, screens, UI components, design tokens, images, Storybook, browser automation or UI design package is included. API/domain architecture and data modeling belong to Planner, not a UI Designer.

An approved API capability may require database access, authentication, validation, service logic or a bounded integration behind the HTTP surface. These support the API; they do not authorize unrelated services, queues, jobs or infrastructure. Add only what the task requires. Database-free endpoints do not need invented tables or migrations. SQL/migrations can be prepared only in approved scope; writing them never authorizes applying them to a live service.

The current local reference covers Express, JavaScript, Node.js and Supabase. Supabase is the documented provider baseline, not proof of a connected database, selected deployment or configured credentials. Planner records the actual owner-approved provider/engine/version and applicable source support for each real task. Provider changes need actual approval and compatible sources; do not silently replace the stack or assume all Supabase references apply to another provider.

## Planning authority and complete delivery

Unlike the frontend-only template, the owner need not supply already-designed APIs. Planner reads current requirements/material under its input or explicitly assigned sources, identifies gaps and proposes complete backend contracts, DB schema when needed and a dedicated Security plan. Product choices and material policy remain proposals until the actual owner decision. Do not inherit deleted PRDs, old feature assumptions or fixed client identities.

Planner's own TODO plans discovery; Output is the canonical requirements/DEC/CHG proposal and approval record. One downstream software TODO specifies exact files/responsibilities, construction order, API names/methods/paths, all request fields, success/failure statuses/headers/bodies/examples, unified error dictionary, data/access/transaction/migration requirements and security enforcement/negative checks. Complete definitions may live in exact mandatory approved source attachments; no competing editable copies. Implementer validates/continues that same software TODO and must not invent missing contract, schema, business or security decisions.

Final API handoff includes one complete operation catalog, exact versioned contract/schema documentation (OpenAPI where approved and locally supported), synthetic request/success/failure examples, auth/permissions, pagination/filter/sort/retry/idempotency semantics, actual startup/config names and candidate-specific checks/limitations. API documentation is an artifact, not a frontend website. Keep docs synchronized with actual implementation; do not claim generation or conformance without checking it.

Planner Output ends with a simplified Arabic owner explanation of the APIs, request/success/failure examples, and applicable tables/keys/relationships with a simple diagram. Mark absent data scope NOT APPLICABLE. Follow Arabic Markdown bidi rules without modifying identifiers, JSON or diagram syntax.

## Contract rules and clients

Existing approved contracts constrain changes. For new APIs this template uses the backend guide's unified error envelope `{ error: { code, message, details? } }`, with details only for 400 validation errors and correlation in `X-Request-Id`. Define exact success shapes per operation, including documented auth exceptions and 204 NO BODY; never return failures as HTTP success or leak secrets, SQL, stack traces or provider diagnostics. An existing incompatible contract is an explicit change/compatibility decision, not permission to break consumers.

The current documented transport is authorized server callers → Express → persistence/provider; client implementations live elsewhere. Do not require the caller to be Next.js or create its cookies/components/routes. Service identity and end-user identity are separate; authenticate and authorize each applicable operation/resource/tenant. Private networking/CORS alone is not user authorization. A browser-facing contract or cookie-based Express auth is an architecture decision requiring explicit owner approval and complete security/source review; no permissive CORS workaround.

## Workflow and evidence

Read [Roster](../../../rostar/roster.md) before selecting an operator. Active roles are Orchestrator, Planner, Implementer, Tester and optional Advisor; assignments remain blank until approved. There is no UI Designer assignment or design TODO dependency. Follow [delivery cycle](../../../agents/orchestrator/skillz/delivery-cycle.md): planning → focused Orchestrator review → Tester CONTRACT SPECIFICATION review → Orchestrator review/personal verification of Planner corrections → implementation → Tester IMPLEMENTATION verification → Orchestrator review/personal verification of Implementer corrections → owner manual API acceptance. Do not automatically return fixes to Tester.

Contract review checks documents, not runtime correctness. Meaningful API tests cover actual status/header/body behavior, auth/resource/tenant denial, validation, failure/degraded dependencies, idempotency/concurrency and regression as applicable. Persistence/constraint/migration claims need actual authorized isolated infrastructure; mocks cannot prove them. Distinguish contract fixtures/provider doubles from real integration. A schema plan is not a deployed DB; a clean build is not a tested API.

Implementer uses only exact compatible local syntax and maintains a real backend application map, never an agents map or an imagined frontend. Tester owns tests. Orchestrator executes authorized existing checks for corrections without editing Tester assertions. Missing tools/data/permissions leave affected work UNVERIFIED/BLOCKED; owner manual API acceptance remains separate. Automated full-system E2E, scans/load, live changes, installations and deployment require actual authority; ordinary assigned API integration tests are not prohibited by the E2E rule.

All authorized agent communication uses [Bridge entry](../../../bridge.md), its external guide-only restriction and two-stage role-read confirmation before task delivery. This policy authorizes no communication, operator assignment or external access. Advisor is optional/nonbinding; no independent review claim without actual separate evidence. A reusable starter is not a running app or a reason to fabricate TODO results, PASS, sessions or API responses.
