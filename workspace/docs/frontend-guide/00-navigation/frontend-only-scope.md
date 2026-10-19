# Frontend-only scope and contract authority

## Mandatory for every role

This starter contains the Next.js frontend and its rendering/transport layer only. Read this policy at role startup, then the frontend GUIDE-MAP and actual task-relevant pages. There is no backend app, backend guide, database implementation or provider assignment here. An external API may use any owner-selected backend; do not impose Express, Node.js, Supabase or a database vendor. Node.js used to run Next.js tooling is not an assignment to build a separate service.

Use Browser → Next.js → owner-supplied external API for application data. Server Components, Server Actions and narrowly scoped same-origin Route Handlers remain frontend infrastructure, not permission to recreate domain services, persistence or authentication services. No database/ORM/provider clients, database credentials, SQL, migrations or invented endpoints in this project. Do not inspect the external backend project without separate permission.

## The owner supplies external API contracts

Planner receives detailed API specifications from the owner in its input folder or explicitly assigned sources. Preserve originals and their versions. Planner checks completeness, detects contradictions, maps approved operations to frontend files, data/state, security and acceptance criteria, and prepares one software TODO. It does not author or approve an external backend contract. Missing required endpoints, fields, success/failure outcomes, authentication or policy go back through Orchestrator to the owner; dependent work stays BLOCKED. Independent fully specified work may continue. API DTOs describe wire data, not database tables.

The external contract owns method/path, request parameters/headers/cookies/body, auth, statuses, response schemas, errors and external side effects. The software TODO contains a labeled read-only extraction and exact mandatory source/version, plus the proposed/approved frontend adapter and UI mappings. Do not silently change upstream envelopes, token transport, TTL, IDs, retry limits or pagination to match a guide example. UI-facing error normalization is a separate frontend definition and must preserve meaningful distinctions and safe messaging. No raw secrets or provider internals reach UI/logs.

Engineering guides own frontend architecture; design sources own approved visuals; local syntax owns implementation mechanics. Provider-specific examples in preserved syntax or imported reference snapshots are technical reference only, never permission to add a backend or override the supplied API. Guide examples using example endpoints, response envelopes, session cookies or numerical settings apply only where the actual approved contract/policy matches. A material conflict is reported with both sources, not silently resolved by changing an API.

## Role-specific expectations

- Planner: inventory owner API documents, check every consumed operation, define proposed frontend file responsibilities and concrete security controls, and block missing decisions. No database-schema or backend implementation deliverable. Its final owner explanation shows supplied requests/successes/failures and frontend data relationships; database schema is outside scope.
- Designer: specify all observable UI states against supplied API behavior. Do not infer business permissions, wire shapes or backend validation from a picture. Keep the full shared Design System and detailed design TODO.
- Implementer: execute approved software and verified design TODOs using only supported local syntax. Validate the supplied contracts; do not invent missing behavior, alter upstream contracts, create fake success handlers or edit Tester-owned tests. Its real project map describes the actual frontend app, not the agents system or an imagined backend.
- Tester: verify frontend behavior and supplied-contract conformance at the permitted boundary. Contract fixtures prove frontend handling only; no fixture, stub, snapshot or successful UI proves external authorization, database integrity, migrations or real integration. Backend-only methods in imported QA references are out of scope. Real API calls need the actual assigned environment/data and permissions; otherwise mark integration NOT RUN/BLOCKED.
- Orchestrator: review contract completeness and scope, enforce the shared delivery cycle, and personally verify corrections. Communicate through Bridge only when actually authorized and after its two-stage role startup. Do not appoint operators or dispatch from blank Roster rows.
- Advisor: optional, nonbinding frontend/integration advice. Backend/schema concerns become contract questions for the owner, not a backend implementation assignment or mandatory gate.

## Evidence and lifecycle

This is a reusable starter, not a running product. Do not initialize apps, install tools, contact external services, claim a passing build or create fake task results merely to fill templates. Mocks/prototypes require explicit scoped authorization, must be labeled, and never count as real integration evidence. Owner input remains untouched. Keep role TODOs and Outputs in task locations authorized by the brief, not input or syntax folders.

The current workflow starts at [Roster](../../../rostar/roster.md). [Delivery cycle](../../../agents/orchestrator/skillz/delivery-cycle.md) remains the phase authority. No actual assignments are approved by this migration. Imported source snapshots retain their upstream content/licenses, interpreted through the active role adapters and this scope policy.
