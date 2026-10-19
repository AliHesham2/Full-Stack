# Prepare frontend consumption contracts before implementation

Use from [SKILL.md](SKILL.md) and the [frontend-only scope](../../../docs/frontend-guide/00-navigation/frontend-only-scope.md). Planner organizes the owner's supplied external API contracts; it does not invent, design or change them. Implementer executes approved frontend construction and reports gaps rather than choosing missing behavior.

## 1. Source authority and intake

Inventory each assigned owner document/message, exact path/section, version/date or content hash, API environment/base URL source, responsible contract owner and approval/status. Preserve originals under assigned input or their approved location. Treat HTML/React examples as illustrative until the owner establishes a wire contract. Inventory unreadable, incomplete, ambiguous or contradictory material with affected operations and precise questions.

Planner Output owns current requirements/proposals/DEC/CHG and links this inventory. The single software TODO contains a labeled read-only contract extraction or exact mandatory complete attachments; originals remain authoritative. Every extracted field/example links its source/version. Differences block affected work for owner resolution through Orchestrator, never Implementer choice. A generic link, assumed endpoint, remembered response or framework convention is not a complete source.

Prepare one [software TODO](../../implementer/templates/todo.md), distinct from Planner's internal checklist and Designer's detailed TODO. Register actual API/DATA/SEC IDs only for current scope. DATA means frontend DTO/state, never database structure. API-free work is NOT APPLICABLE with a reason; it does not need invented services or API documents.

## 2. Complete supplied operation catalog

Read [frontend integration](../../../docs/frontend-guide/03-data/backend-integration.md), relevant security/state/resilience sections and the actual supplied definitions. Record every consumed operation before dependent code:

1. API ID, purpose/module, exact method/path, external contract owner/version/approval, frontend consumer/implementation item, requirement and AC IDs. Distinguish the external operation from a proposed browser-facing Next.js route/action; proposed local transport routes do not add external backend capabilities.
2. Authentication and identity transport, permitted frontend-visible roles/states, ownership rules as supplied, required headers/cookies/CSRF, and responsibility at Browser/Next.js/external service boundaries. No assumed bearer tokens, cookie names, TTL or login endpoint. Missing necessary policy is a question, not a default.
3. Separate path/query/header/cookie/body tables: exact names/types/formats, nesting/items, required/optional/null/absent distinctions, defaults/enums/length/range, encoding/normalization, allowed media types, unknown-field and payload-limit behavior where relevant. Explicitly mark absent sections. Missing required values stay BLOCKED.
4. Each declared success: trigger, exact status, response headers/cookies, complete schema or NO BODY, types/optional/nullability, synthetic schema-consistent example, frontend parsing/state/cache effects, external side effects only as documented. Do not infer DB writes or transaction behavior.
5. Each declared failure: trigger, exact status, original body/envelope/code/message/details/header semantics and synthetic example; source-defined exceptions such as no body/non-JSON; frontend field/global/session/denial/offline/error-boundary mapping. Cover applicable invalid input, auth/denial, missing data, conflicts, throttling, dependency and unexpected failures without manufacturing server status guarantees.
6. Supplied pagination/filter/sort, retry/idempotency/concurrency, cache/version rules and relevant limits. Propose frontend timeouts/cancellation/reconciliation explicitly, with source and approval; never infer upstream retry safety. Unknown write outcomes are not permission to resubmit automatically.
7. Consumers, exact approved frontend paths/DTO/state/source support, dependencies, acceptance and planned negative/boundary checks. Check every example against its source schema/status/headers. Synthetic examples never contain real personal data, credentials or tokens.

## 3. Original wire errors versus unified frontend errors

The owner-supplied API owns its response envelopes and statuses, including any differences across services. Do not impose the FullStack envelope, wrap a success differently, turn a failure into upstream success, or demand the external service change. A frontend adapter may normalize errors only through a named, approved separate model.

Record the local safe error fields/types (including optional/absent semantics), operation/status/wire-code mapping, permitted field validation errors, safe message policy, request/correlation handling where supplied, and distinctions between authentication, denial, validation, business, network, timeout, cancellation, malformed response and operational failure. Handle unrecognized statuses/codes and invalid/non-JSON bodies as safe generic failures; do not invent external codes. Preserve retryability/uncertain outcomes rather than disguising outages as bad credentials. Never leak stack traces, raw upstream bodies, secrets or PII.

Every API/Next.js/action consumer uses the same approved frontend mapping for the same semantics. UI errors need not expose upstream internal diagnostics. Success shapes remain exact per operation; 204 has no JSON body. Keep wire extraction and frontend decisions distinguishable, versioned and linked; no independently editable competing contract in Output.

## 4. Frontend data, state and cache plan

Define actual consumed DTOs from the supplied schema: field/source/type/optionality/nullability, runtime parsing/validation, approved view-model transformation, sensitive/server-only fields and serialization boundary. Do not describe DTOs as database tables or infer storage keys, foreign keys, indexes or migrations.

Assign frontend state ownership (server rendering, URL, component/form state or approved query cache), query keys, approved freshness/cache policy, cancellation, stale/empty/loading/error/success behavior, mutation invalidation and optimistic rollback/reconciliation as relevant. Prevent cross-user cache leakage, duplicated state and stale responses overwriting newer state. Unsettled behavior returns through Orchestrator to its decision owner.

A simple UI-data flow diagram may show Browser → Next.js → external operation and DTO/view consumers. A database diagram is not required or authored here; owner-provided storage context is read-only reference, not implementation scope.

## 5. Dedicated frontend security plan

Every software TODO includes a Security section with actual SEC IDs, threats, affected operations/files, concrete controls/enforcement, approved settings and negative/boundary verification. Read [frontend security](../../../docs/frontend-guide/04-quality/security.md) and supplied auth/permission rules.

Address applicable XSS/output safety, input and untrusted-response parsing, safe redirects/URLs, allowlisted fixed outbound destinations and forwarding rules, session/cookie/CSRF transport, server-only credentials, private/public environment variables, serialization/log/PII/error exposure, user-specific caches, upload/download limits and sensitive UI flows. External service authorization remains its responsibility: hiding a button or passing a frontend test does not prove enforcement. Note required external guarantees and unverified evidence honestly.

Use actual policy values or explicitly proposed owner decisions. Never ask Implementer to select unknown security policy or weaken a control to unblock integration. Record justified exclusions. No production calls, security scan, external project access or setup is authorized by writing the plan.

## 6. Per-item readiness, changes and explanation

Each executable item must link complete owner-provided API inputs/successes/failures, approved frontend error/DTO/state/security mappings, file responsibilities, compatible local syntax support, ACs and actual design prerequisite evidence. Blank fields, generic links, TBD, fake success handlers or instructions to invent missing API behavior fail the gate. Block only dependent work; unrelated fully specified work may proceed when assigned.

After an owner contract update, preserve originals/version history and record CHG/DEC impact on consumers, requirements, software/design TODOs, security, tests and stale approvals. Route revised visual specs to Designer. Implementer cannot silently update upstream definitions or choose between contradictory versions.

End Planner Output with the simplified Arabic owner explanation of supplied API names/methods/inputs/success/failure examples, safe error behavior and frontend data flow, generated from the same sources. State database schema/backend implementation is outside scope. Explanation is not a second editable contract or evidence of working integration.
