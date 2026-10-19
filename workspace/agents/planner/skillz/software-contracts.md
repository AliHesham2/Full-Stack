# Prepare complete software contracts before implementation

Use from [SKILL.md](SKILL.md) before recommending the software TODO for execution. Planner resolves software behavior, interfaces, schema and security requirements using current owner input, actual project facts and applicable guides. Implementer executes the approved specification; it does not complete missing product, API, database or security decisions. Missing decisions return through Orchestrator to their actual owner, not to invented Planner defaults.

## 1. One authoritative definition, accessible from the software TODO

Planner Output remains the canonical requirements/proposal and decision record. Its API/data section is an inventory pointing to exact complete definitions, not a second independently editable contract. Put new task-specific API/data/security specifications in the software TODO's dedicated sections. If an approved schema/OpenAPI/project policy already owns a definition, preserve that authority and reference its exact version/section as a mandatory part of the handoff. A needed complete attachment must be explicitly linked, versioned and accessible; a generic folder, README or remembered example is insufficient.

The TODO must expose the endpoint/entity/security inventories and the decisions needed by each execution item. Where a readable snapshot of an existing definition is included, label it derived/read-only with its source version; mismatches block dependent work rather than letting Implementer choose between copies. Planner owns reconciliation. A later transfer of authority to an implementation schema/OpenAPI file must be explicit and version-linked; code generation or creating that file does not silently approve a different contract.

Use the [software TODO template](../../implementer/templates/todo.md), preserving one current backend software TODO. Define real API/DB/SEC IDs only for actual scope; use justified NOT APPLICABLE when no data/security surface is affected. API-only does not mean every endpoint needs a DB. Proposed contracts/schema stay proposed until authorized.

## 2. API catalog and complete operation definitions

Start at the backend map and actual [API guide](../../../docs/backend-guide/02-domain-and-api/api-design.md), auth/validation/security/data sections and local syntax. Preserve external authorized clients → Express/Node.js → persistence/provider. Specify server/client trust and contract boundaries without requiring or building Next.js/browser code.

For every in-scope operation, define before implementation:

1. Stable API ID, operation name/purpose, method and exact path, module/handler owner, callers and consumers, requirement/AC IDs and approved version. Label existing versus new/changed.
2. Authentication/service identity, permitted actors, object/tenant ownership and session/cookie/CSRF responsibilities at each relevant boundary. Link exact approved policy, not an instruction to "add auth".
3. Separate **path parameters, query parameters, request headers, cookies and body**. Give exact field names and types/formats, nesting/items, required versus optional, null versus absent semantics, defaults, enums/ranges/lengths, normalization/encoding and validation errors. Explicitly state absent sections, unknown-field handling, allowed media types and any approved payload bounds. Do not invent limits to fill a table.
4. Every intended **success outcome**: status, response headers/cookies, complete response body schema or explicitly no body, field types/optionality/nullability, serialization and synthetic example. Describe observable side effects, relevant DB entities and transaction/state behavior. A bare "returns user" or "200 OK" is not complete.
5. Every declared applicable **failure outcome**: trigger/precondition, status, stable project error code, safe message rule, field details where allowed, response headers and complete synthetic example. Cover applicable malformed input, auth/ownership denial, missing resource, business conflict, throttling and operational failure without assigning all HTTP statuses to every endpoint. Declare no-body and non-JSON exceptions explicitly where the approved contract requires them.
6. Pagination/filter/sort vocabulary and defaults, idempotency/concurrency/retry behavior, caching/security headers, timeouts and compatibility/version changes when relevant. Use settled guide/project decisions; unresolved policy remains a blocker. A retry or quota count is not chosen silently by Implementer.
7. Owning implementation item/paths, consumers, DB/SEC IDs, observable acceptance and negative/boundary cases. State what evidence Tester and Orchestrator will need. This is a specification, not production code or executable test authorship.

Inspect examples against their schemas: status/body/headers agree, fields/types/optional states match, examples contain no real secrets, and external consumers can distinguish declared outcomes without guessing. Cases not covered by a known applicable requirement remain precise questions, not manufactured response contracts.

## 3. Unified error contract and explicit success shapes

The current project already fixes one error envelope in [authentication §0](../../../docs/backend-guide/03-auth/authentication.md) and [API design §4](../../../docs/backend-guide/02-domain-and-api/api-design.md): `{ error: { code, message, details? } }`. Preserve it; do not invent a new envelope or put errors inside successful HTTP responses. `details` is a field-path-to-message-array map only for 400 validation errors, omitted otherwise. Correlation uses the `X-Request-Id` response header, not a competing body field. Errors must not leak secrets, provider internals, stack traces or SQL.

Define one task-linked dictionary of actual error codes, triggers, statuses and safe message rules, with each endpoint referencing the applicable entries. The same failure semantics must not vary across modules. Explain unexpected/internal failures through the approved generic mapping and preserve their operational versus authentication distinction; do not turn an outage into invalid credentials.

Uniform errors do not imply all successes have the same body. Follow the actual approved success envelope per operation and existing guide exceptions. For example, the fixed Express login contract differs from ordinary data responses, and 204 has no response body. Planner must specify the exact outcome; Implementer may not wrap/unwrap fields, change statuses or expose internal responses for convenience. Specify exact client-visible behavior; external client implementations remain outside this project.

## 4. Database schema and access behavior

Read actual [data architecture](../../../docs/backend-guide/05-data/data-architecture.md), relevant access/transaction/concurrency/migration/auth guidance and applicable local syntax sources. Inspect existing schema/version before proposing changes; a starter template is not a deployed database. Provider/engine/version choices must be established or remain questions, never inferred from generic examples.

For each affected entity/table specify its exact schema-qualified name, owning module, purpose, existing/new/changed state and DB ID. Give complete columns: exact type supported by the actual engine/version, nullability, default/generation, PK/FK target, unique/check constraints, enum vocabulary and sensitive/server-owned classification. Include composite keys and constraint names/definitions where relevant. Do not invent database functions or values absent from the approved source/version.

Document relationship cardinality/optionality and deliberate delete/update behavior; add a simple ER diagram consistent with the actual field definitions. Use a text diagram if Mermaid is unavailable. The picture does not replace the table/constraint definitions. Indexes must name the actual query/filter/sort or correctness invariant they support, rather than indexing every field.

Map each operation to allowed reads/writes, ownership predicates, derived values, transaction boundaries, uniqueness/race handling and idempotency where applicable. Specify timestamps, deletion/retention/audit behavior only as required by the existing policy/approved scope. Preserve table ownership and trust boundaries; data remains accessible through Express, never through external client database/provider credentials.

For a change to existing data, specify compatibility, migration/backfill/order, dependent consumers, data-loss risks and supported recovery/forward-fix requirements. Distinguish preparing migration artifacts from permission to apply them. Unknown actual schema/version, ownership, destructive behavior or required policy blocks dependent work. No schema execution or live migration follows from a completed planning table.

## 5. Dedicated security plan

Every software TODO has a Security section. Identify actual assets, actors, entry points/trust boundaries and plausible failure/abuse scenarios. Read [backend feature threat checklist](../../../docs/backend-guide/04-security/security-foundations.md#8-feature-threat-checklist-run-per-task) and applicable auth/authorization/secrets guidance. "Secure it" is not an implementation plan.

For each applicable `SEC-n`, specify the source rule/decision, concrete enforcement location, protected API/DB/software items, required control and observable verification/denial cases. Address authentication and object/tenant authorization separately; server-derived ownership/permissions, input validation and safe query/output handling, session/CSRF/cookie transport, secret/PII exposure, abuse/resource bounds, outbound/file/dependency risks and failure/logging behavior as applicable. Carry existing controls forward and name exclusions with reasons. No new security product or infrastructure is added merely because a risk category exists.

Exact settings/limits and retention must come from applicable approved policy or a recorded owner decision; unresolved values cannot be selected by Implementer. Document implementation controls and planned verification separately from actual security assurance. A checklist or threat model does not establish compliance or a verified vulnerability. Security exceptions need the actual authorized decision and cannot waive higher-priority rules.

## 6. Readiness gate and changes

Before recommending any software item as executable, verify complete applicable API request/success/error definitions, DB schema/access/migration decisions, security controls, file responsibilities, source/version support, contract-review prerequisites and acceptance mapping. Link each item to exact API/DB/SEC definitions. Neither `TBD`, a blank cell, "handle errors", "set up DB" nor "implement secure auth" is a completed handoff. Mark the affected item `BLOCKED` with the exact gap/decision owner; unrelated fully specified work may continue within authority.

Have Orchestrator review readiness and the versioned Tester CONTRACT SPECIFICATION results before dispatch. Planner resolves plan/contract/schema gaps, and the actual owner decides missing product/material policy choices. Implementer inspects actual code/source compatibility, reports contradictions and executes approved details without becoming product/architecture decision owner.

After approval, contract/schema/security changes require the applicable decision and Planner's versioned CHG/DEC analysis, updated definitions/consumer mappings, software TODO dependencies and renewed checks. Update Output's inventory and final simplified owner explanation from those same definitions. Preserve prior versions; do not allow drift between narrative, TODO, authoritative schema and examples.

## 7. Published API contract and final catalog

Plan one complete API catalog and versioned machine-readable contract, normally OpenAPI when approved and locally supported, with exact source path/version/ownership and publication route. Name each operation, auth/security scheme, path/query/header/cookie/body schema, all success/error statuses/headers/bodies, examples and applicable pagination/idempotency rules. Avoid independently editable schemas: specify canonical ownership and generation/synchronization direction. Tester checks supported conformance and coverage; missing schema/tool syntax is a blocker, not permission for Implementer to guess. Docs/examples/collections contain synthetic values and no credentials; serving docs publicly or adding a docs UI requires explicit scope. A plain versioned contract file is sufficient as an API artifact.
