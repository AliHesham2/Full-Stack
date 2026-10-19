# Frontend integration with owner-supplied APIs

Read [frontend-only scope](../00-navigation/frontend-only-scope.md) and [Planner contract preparation](../../../agents/planner/skillz/software-contracts.md). The filename is retained for existing frontend guide links; this is consumption guidance, not a backend implementation manual.

## Boundary and source authority

This project contains one independent Next.js app, normally `frontend/`, with its own package and lockfile. Browser → Next.js server rendering/actions/allowlisted same-origin routes → external API. No backend folder, database/ORM/provider SDK, SQL, migrations or direct storage/auth provider implementation. Do not mandate Express, Node.js, Supabase or a database engine for the external service.

The owner supplies the actual API definitions. Planner checks and maps them into the software TODO; missing required operations, fields/statuses/auth/policy return to the owner through Orchestrator. Do not invent endpoints or use guide snippets as contracts. Record exact source/version and approval before coding. External API source access, setup and live calls require separate actual authority.

## HTTP adapter responsibilities

Keep external transport knowledge in a server-only `src/lib/http.ts` adapter, with feature reads in `features/<feature>/queries.ts` and mutations in `actions.ts`. Browser interactive reads use narrowly scoped same-origin Next.js GET handlers; mutations normally use Server Actions. Server Components may use server queries directly. These are frontend transport concerns, not independent domain services or a generic proxy.

- Validate server configuration from actual approved variable names, such as `API_BASE_URL`; no default endpoint is fabricated. Optional `API_SERVICE_KEY` exists only if the owner contract requires an approved server identity. Never expose server-only credentials through NEXT_PUBLIC variables, serialized props, logs or errors.
- Construct destinations from a fixed approved origin and allowlisted operation paths. Validate/encode parameters. Never forward arbitrary client URLs, cookies, headers or bearer credentials, and do not follow unsafe redirects into a different origin. Handle upstream redirects according to an explicit safe policy.
- Supply only required approved auth/headers/cookies. External auth does not imply a specific token type. Distinguish server identity from user identity; do not grant user access through a broad service credential.
- Bound duration and resource use using approved settings. Preserve server-local cancellation and browser cancellation through allowed transport; AbortSignal is not a serialized Server Action argument. Do not silently pick quotas/timeouts or retry mutations.
- Treat response bodies as unknown. Validate against exact supplied runtime schemas before using them; distinguish optional, absent and null. Decode by declared status/media type, including NO BODY success. A 204 must not be parsed as JSON. Invalid bodies, oversized data and undeclared status/codes map to safe local failures.
- Preserve supplied success/error statuses and representations at the upstream boundary. Any browser-facing transport/action transformation is a separate approved frontend contract. Expose only required safe fields, never raw tokens or diagnostics. Do not fabricate a success response when a call failed or has not run.
- Match cache scope to sensitivity. User/session-specific responses must not enter shared public caches. Explicitly document query keys, freshness and invalidation; do not cache merely because an example does.

## Contract inventory and synchronization

For each consumed operation link the owner document/version, name/method/path, request field locations/types/required/nullability/default/encoding/constraints, media types, authentication, success status/body/headers/cookies, failure cases and their exact wire representation. Include supplied pagination/filter/sort/idempotency semantics and applicable compatibility rules. Each operation maps to frontend files, DTOs, UI states, ACs, security controls and planned checks.

Generate or derive consumer types only from approved supplied definitions through the documented route. Keep generated versus authored files clear. Runtime parsing is still needed at trust boundaries. No external backend runtime imports or independently guessed mirror schema. Contradictory definitions block consumers until owner resolution. API DTO relationships are not database tables/foreign keys.

## Reads, rendering and browser queries

Server-rendered reads call feature server queries through the adapter. Interactive client queries call a fixed same-origin GET handler that validates allowed query parameters, preserves permitted request cancellation and returns the approved browser-facing representation. Do not forward arbitrary paths/headers or implement generic external proxy access.

TanStack Query or another approved cache uses the actual contract and approved keys. Distinguish initial loading, empty, success, stale refresh, cancellation, offline and recoverable/permanent failures. An empty list is not an error fallback. Gate package/API choices on actual local syntax support and compatible installed versions.

## Authentication

Use the supplied identity/session contract; no default `/auth/login`, `/auth/me`, token envelope, cookie name or session lifetime is established by this template. Planner must state each required auth operation and exact behavior before dependent work.

When the approved flow requires Next.js-owned cookies, set the approved name/scope/expiry and appropriate HttpOnly/Secure/SameSite protections, consistent with the actual environment and CSRF policy. Never copy illustrative seven-day TTLs or expose a credential to Client Components. If the external service issues cookies, determine the explicit forwarding/domain/identity policy; do not blindly relay arbitrary Set-Cookie values.

Session lookup distinguishes documented unauthenticated state from network/timeout/operational failures. Do not turn outages into logged-out state or clear a valid session for generic errors. UI redirects are convenience; they do not prove external authorization. Logout/revocation/expiry and OAuth callback/one-time-handoff behavior must follow supplied contracts and safe redirect/state rules. No provider SDK or token refresh service is implemented here.

## Mutations and Error States

Use approved Server Actions for normal mutations, with serializable validated inputs and typed action state. Validate locally for UX and transport safety; the external API remains responsible for its own authorization, business rules and input enforcement. State exact user-visible success, validation, refusal, conflict, session, loading, cancellation and uncertain-outcome behavior in software/design TODOs.

Preserve original wire errors but normalize them into one approved frontend safe model. Do not impose `{ error: { code, message, details? } }`, `{ data: ... }`, a login token shape or particular HTTP codes on the service. If a supplied API uses those shapes, parse them exactly. Field details come from the actual schema; safe generic fallback handles unknown/malformed/non-JSON failures. Never leak raw upstream messages, PII, secrets, stack traces or internal URLs. Use correlation headers only when actually supplied and safe.

Expected validation/business refusals become typed UI feedback; unexpected operational failures follow the approved error-boundary strategy. Authentication, denial, validation and outages stay distinguishable. Mutations are not automatically retried: an interrupted response may hide a successful write. Follow supplied idempotency and approved reconciliation before an explicit user retry. Coordinate invalidation/revalidation with actual affected caches and server-rendered paths. Approved optimistic updates need rollback/reconciliation and stale-response handling.

## Pagination and Realtime

Use supplied cursor/offset/filter/sort shapes and preserve metadata. Validate approved bounds and encoding at the Next.js transport boundary. No assumed page size or fabricated total count. Client cancellation and older-response ordering need explicit handling.

Realtime, SSE or WebSockets require a supplied service capability and approved Next.js/host transport design, authentication, reconnect/expiry/backpressure/resource-cleanup rules and local syntax support. No provider subscription SDK or invented streaming endpoint. Mark unsupported hosting/protocol requirements as blockers rather than claim a relay works.

## Third-Party Integrations

Consume explicitly supplied API capabilities through approved frontend transport. External provider credentials and business service operations remain outside this project; no integration backend is created for convenience. A new capability/provider/auth requirement returns to the owner. Third-party scripts in the frontend need approved security/privacy/CSP and performance treatment.

## File Uploads and Downloads

Require supplied operations and actual host limits before implementing. By this template's default boundary browser bytes pass through a bounded Next.js action/route to the external API; any direct external upload/download architecture needs an explicit owner-approved change to this boundary and a complete safe contract. Do not assume provider URLs or storage credentials are permitted.

For small uploads account for total multipart overhead and configured action/platform limits. Large transfers need approved bounded streaming/chunk/resume behavior and cancellation, not unbounded request.formData() buffering. A Route Handler does not bypass platform size/time limits. Do not implement external storage writes, ownership/quota checks or orphan cleanup; identify those as service responsibilities/contract guarantees. Frontend checks improve UX and resource safety without proving external enforcement. Content-Length and client MIME are not trustworthy evidence of actual content.

Downloads use approved authenticated relays and safe content/disposition/type/length handling. Do not redirect to arbitrary upstream locations or expose storage identifiers/credentials beyond the supplied public contract. Plan partial-failure recovery and report missing service guarantees.

## Verification limits

Tester owns tests; fixtures may cover parsing, statuses, failures, cancellation, safe forwarding, sessions, caches and UI handling in an isolated permitted environment. Labeled mocks cannot prove real API integration, service authorization or persistence. Real API checks require explicit environment/data/side-effect authority. A starter with no app/service records NOT RUN, not PASS. Follow the [delivery cycle](../../../agents/orchestrator/skillz/delivery-cycle.md); Orchestrator reviews results and personally verifies corrections. Owner manual E2E remains acceptance.
