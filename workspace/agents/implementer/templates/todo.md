# Software TODO — <task ID> / candidate <identity>

> Planner prepares this software construction plan after analysis. Implementer validates and continues the same authorized file; create one only if no plan was supplied within an approved task. No visual TODO is required; read the actual versioned API contract review and Orchestrator disposition. Preserve approved scope and add actual execution information without invented results. Paths are from project root unless stated otherwise.

## Assignment and scope

| Field | Value |
| --- | --- |
| Task / assigned operator / assignment reference | `<IDs and actual authority>` |
| Approved requirements / software / reviewed contract versions | `<current Planner Output, software TODO baseline and exact API/DB/SEC definitions; CONTRACT SPECIFICATION Tester report and Orchestrator disposition/correction evidence; IDs>` |
| Role entry / handoff state / contract-review prerequisites | `<workspace/agents/implementer/prompt.md; WAITING FOR CONTRACT REVIEW or actual review disposition>` |
| Syntax entry and selected route | `<workspace/docs/syntax/GUIDE-MAP.md; actual indexes/pages read>` |
| Goal, affected apps and exclusions | `<bounded scope>` |
| Outcome slices / inherited shared constraints / required source material | `<Planner Output SL/DEC IDs and source paths/sections; actual prerequisite states>` |
| Allowed files, tools, environment/data and reporting route | `<actual permissions>` |
| TODO / Output / evidence locations | `<authorized paths>` |
| Baseline and intended candidate identity method | `<commit or changed-file manifest; no invented revision>` |
| Canonical implementation map / inspected branch | `<existing path or authorized backend/docs/PROJECT-MAP.md; observed scope>` |

## Baseline and ownership

| Area / file | Existing behavior, edit or failure | Evidence and candidate | Owner / consequence for this task |
| --- | --- | --- | --- |
| `<app/path or NONE OBSERVED>` | `<observed fact or NOT RUN>` | `<command/output/path>` | `<preserve / blocked / task prerequisite>` |

## Implementation and source map

| Item ID | Requirement / module / API/DB/SEC IDs | Planned change and allowed files | Guide file/heading | Local syntax file/heading and applicable version | Dependencies, negative/boundary cases and risk | Planned verification | Status |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `<I-n>` | `<actual IDs>` | `<small coherent change>` | `<backend rule>` | `<workspace/docs/syntax/...; read before code>` | `<named failure classes and prerequisites>` | `<existing check/inspection; Tester gap if needed>` | `TODO` |

Use `TODO`, `IN PROGRESS`, `BLOCKED`, `REPORTED`. `REPORTED` means evidence was delivered, not accepted. Source gaps block the dependent item under `workspace/agents/implementer/skillz/syntax-policy.md`.

## Consumed and produced boundaries

| Implementation item | Consumes: interface/artifact ID, version and producer item | Produces: interface/artifact ID and consumer item | Exact names, fields/types/signatures or canonical definition | Compatibility check / missing contract review or source prerequisite |
| --- | --- | --- | --- | --- |
| `<I-n>` | `<actual requirements/contract/review evidence>` | `<required public boundary and downstream task>` | `<source-defined boundary; no duplicated implementation>` | `<match evidence or explicit blocker>` |

Use reviewable deliverables rather than arbitrary tiny steps. Supporting setup belongs with the work that needs it unless independently useful. Compare producer/consumer names and shapes, preserve approved values and local syntax support, and leave unresolved contract/interfaces blocked. Link prioritized failure cases to their owning task and Tester verification; do not write tests or production bodies in this plan.

## API contracts — prepared by Planner before execution

Use [Planner's contract preparation procedure](../../planner/skillz/software-contracts.md). This section contains complete new task-specific contracts, or exact mandatory linked versions/sections of an existing authoritative contract. A derived snapshot must be labeled read-only with its source/version. Planner Output inventories and explains these same definitions; do not maintain independently editable duplicates. Use `NOT APPLICABLE` with a scope reason if no API is affected.

### Operation inventory

| API ID / name and purpose | Method / exact path / boundary | Owning module / implementation item and paths | Callers / requirement and AC IDs | Existing/new/changed; authoritative definition/version and approval | Linked DB/SEC IDs / readiness gap |
| --- | --- | --- | --- | --- | --- |
| `<actual API-n or NOT APPLICABLE>` | `<Express operation and authorized external consumer boundary>` | `<I-n; exact planned files>` | `<consumers and IDs>` | `<this section or complete versioned attachment; decision reference>` | `<resolved prerequisites or BLOCKED>` |

### Complete operation definition — repeat for each actual API ID

Operation/version: `<API ID and approved definition>`; identity/roles/object/tenant rules: `<exact policy and enforcement>`; request media type/unknown-field handling/payload bounds: `<approved rules or explicit gap>`.

| Request location | Exact field/header/cookie name | Type/format, nested schema or allowed values | Required/optional, nullable/absent and default behavior | Validation/normalization/encoding and bounds | Invalid/missing outcome / source |
| --- | --- | --- | --- | --- | --- |
| `<PATH / QUERY / HEADER / COOKIE / BODY; state NONE for absent locations>` | `<exact name>` | `<complete canonical type/schema>` | `<precise semantics>` | `<approved rule, not guessed limits>` | `<declared error mapping>` |

| Outcome / triggering condition | HTTP status | Response headers/cookies and boundary-specific handling | Complete body schema / fields, types, required/nullability or NO BODY | Synthetic request/response example or exact complete attachment | Observable side effects / API-DB-SEC-AC references |
| --- | --- | --- | --- | --- | --- |
| `<each intended SUCCESS case>` | `<exact status>` | `<defined headers/cookies or NONE>` | `<exact representation; 204 has no body>` | `<schema-consistent synthetic example>` | `<writes/transaction/state and downstream handling>` |
| `<each declared applicable FAILURE case>` | `<exact status>` | `<correlation/retry headers when applicable>` | `<shared error envelope + defined code/message/details rules>` | `<complete example with no internals/secrets>` | `<trigger/denial/operational behavior and ACs>` |

Pagination/filter/sort vocabulary/defaults: `<exact definitions or NOT APPLICABLE>`.
Idempotency/concurrency/retry/timeout and cache/security-header policy: `<settled applicable rules, source and responsibility at each boundary>`.
Compatibility/version/dependent consumer changes: `<approved change and synchronization route or NONE>`.
Do not use prose such as "returns user", "handle failures" or a bare status as a complete contract. Missing fields/statuses/policy are Planner blockers, not Implementer choices. Keep browser-facing behavior distinct from the Express response to authorized external clients.

### Unified error system

Canonical envelope: `{ error: { code, message, details? } }`, per backend API-design §4 and authentication §0. `details` is a field-path-to-message-array map only for 400 validation errors; otherwise omit it. Correlation is in `X-Request-Id`, not a new body field. Preserve applicable project error/status semantics and safe messages; never leak provider internals, secrets, SQL or stack traces, and never disguise an error as HTTP success.

| Stable error code | Trigger / applicable API IDs | HTTP status / safe message rule | Validation details schema or OMITTED | Required response headers / next-layer handling | Source/approval and verification case |
| --- | --- | --- | --- | --- | --- |
| `<actual project code, not an invented per-handler variant>` | `<explicit cases>` | `<auth versus denial versus business versus operational distinction>` | `<field rules or OMITTED>` | `<approved contract>` | `<source and AC/check IDs>` |

Success shapes are explicit per operation above; a shared error envelope does not permit changing an approved success envelope or existing login/no-body exceptions. No error dictionary or API is fabricated for API-free scope.

## Database schema and access plan — prepared by Planner

Read the contract procedure's DB section and actual data guides. Use `NOT APPLICABLE` with a reason for scope without DB changes/access; a picture alone or "create users table" is insufficient.

Database engine/approved version and actual baseline: `<evidence or precise blocker>`.

| DB ID / schema-qualified table/entity | Owning module and purpose | Existing/new/changed and authoritative schema version | Related API/I/SEC IDs | Required read/write patterns and allowed owner/tenant scope |
| --- | --- | --- | --- | --- |
| `<actual DB-n>` | `<one owner>` | `<exact source/approval>` | `<IDs>` | `<query and access behavior; no frontend DB access>` |

| Table / exact column | Engine-supported type / enum or referenced definition | Nullable / default / generation / absent-value semantics | PK/FK target / unique/check constraint | Sensitive/server-owned classification and write rule | Source/approval |
| --- | --- | --- | --- | --- | --- |
| `<actual column>` | `<complete type and source/version>` | `<explicit choice>` | `<exact definitions including composite constraints>` | `<derived versus accepted input>` | `<canonical reference>` |

Relationships and ER diagram: `<simple Mermaid or text diagram of the actual proposed/approved entities, keys and cardinalities; label status; no fictional existing schema>`.
Relationship update/delete behavior: `<exact FK actions and ownership rules>`.
Indexes and supported queries/invariants: `<columns/order/uniqueness; rationale or NOT APPLICABLE>`.
Transaction/concurrency/idempotency and data lifecycle: `<API-to-DB reads/writes, atomic boundaries, races, timestamps, deletion/retention/audit as applicable>`.
Migration/backfill/compatibility/order and recovery: `<existing-data impact, consumers, supported recovery and environment permission; preparing is not applying>`.

## Security plan — prepared by Planner

Assets, actors and trust boundaries: `<actual affected surfaces; no assumed infrastructure>`.
Relevant threat/abuse scenarios and source checklist: `<concrete scenarios plus task-relevant guide sections>`.

| Security ID / protected API/DB/software items | Threat/denial scenario and source policy | Required concrete control / exact enforcement location | Approved settings/limits and secret/data handling | Observable negative/boundary verification / owner | Status / missing decision or justified exclusion |
| --- | --- | --- | --- | --- | --- |
| `<actual SEC-n>` | `<source-backed risk, not a presumed vulnerability>` | `<what must be enforced and where; not "secure it">` | `<settled values or BLOCKED; no secret values>` | `<expected outcome and AC/check IDs>` | `<actual readiness>` |

Address applicable authentication, object/tenant authorization, server-owned inputs, validation/injection/output safety, session/cookie/CSRF/service transport, secret/PII/log/error exposure, abuse/resource bounds and outbound/file/dependency risks. Reference existing mandatory policy rather than invent controls or defaults; record exclusions with reasons. Missing security decisions block affected work. This plan is not evidence of compliance, a scan or an accepted security exception.

## Planner contract-readiness gate

| Execution item | Complete approved API definitions / shared errors and successes | Complete DB/access/migration definitions | Concrete security controls | File responsibilities / local syntax and actual contract-review prerequisites | Decision evidence / missing detail and owner | Disposition |
| --- | --- | --- | --- | --- | --- | --- |
| `<I-n>` | `<API IDs/version or justified NOT APPLICABLE>` | `<DB IDs/version or justified NOT APPLICABLE>` | `<SEC IDs or justified exclusion>` | `<exact satisfied dependencies/review or gaps>` | `<actual references, not presumed approval>` | `<READY FOR ORCHESTRATOR REVIEW / BLOCKED / WAITING FOR CONTRACT REVIEW>` |

No ready item contains unresolved contract/schema/security choices, placeholder fields or unexplained `TBD`. Orchestrator reviews this gate before dispatch. Implementer validates and executes the approved plan, reports inconsistencies and blocks dependent work; it does not invent endpoints, request/response shapes, DB structure, security policy or change requirements to make implementation easier. Internal code mechanics still require the approved guides/local syntax and self-verification.

## API catalog and contract publication plan

| Contract/catalog artifact and exact path | Canonical definition/version and generation or synchronization direction | Operation/request/success/error/auth coverage | Compatibility consumers and approval | Validation/documentation evidence and gaps |
| --- | --- | --- | --- | --- |
| `<versioned OpenAPI or actual approved complete contract format; no unsupported syntax>` | `<one authority, no competing manual definitions>` | `<all scoped API IDs and exact complete definitions/examples>` | `<external callers; change/approval references>` | `<existing permitted tooling and checks or BLOCKED>` |

No documentation website/UI is implied. Use synthetic values, actual configured server paths and no secret/token values. Do not claim a generated contract matches implementation without checking it.

## Integration and side-effect plan

| Concern | Existing owner and consumers | Approved change / compatibility and ordering | Recovery or irreversible effect | Execution permission / open decision |
| --- | --- | --- | --- | --- |
| `<API/schema/types/tokens/dependency/config or NOT APPLICABLE>` | `<actual paths>` | `<scope and synchronization route>` | `<supported recovery, or unknown>` | `<prepare only / permitted environment / BLOCKED>` |

## Check plan

| Check ID | Item / requirement | Exact existing command and working directory or inspection | Candidate/environment/data required | Expected evidence | Dependency / owner |
| --- | --- | --- | --- | --- | --- |
| `<C-n>` | `<I-n / AC-n>` | `<command; app directory>` | `<runtime/fixture or missing prerequisite>` | `<exit/output or observation>` | `<Implementer execution / Tester-owned suite>` |

Include `npm run check` in every changed app when available and record missing scripts/environments honestly. Tests stay with Tester; automated E2E is not authorized by this plan.

## Navigation, startup and owned-resource plan

Project map: `<existing map verified / creation planned before code>`; structural changes to reflect: `<actual or proposed paths, kept distinct>`.

| App / prerequisite | Existing command and working directory / config source | Required variable names and external services | Planned readiness observation / permission gap |
| --- | --- | --- | --- |
| `<app or NOT INITIALIZED>` | `<approved startup script/config path; no secret values>` | `<names/prerequisites only>` | `<expected URL/health/state; NOT RUN until observed>` |

| Task-created resource / purpose | Planned location and cleanup boundary | Actual process/session identity + project/command, or resolved file path after creation | Lifecycle / handoff need | Current state |
| --- | --- | --- | --- | --- |
| `<server/helper/temp artifact or NONE PLANNED>` | `<authorized location; evidence retained separately>` | `<actual evidence or NOT CREATED>` | `<stop/remove when no longer needed or authorized continuation>` | `<actual state>` |

Use `workspace/agents/implementer/skillz/project-map-and-handoff.md` (project-root path). Do not stop pre-existing processes or delete unknown files. Keep evidence required for review.

## Questions and blockers

| ID | Exact missing/conflicting requirement, design, source or permission | Affected item and impact | Evidence / route / resolution required | State |
| --- | --- | --- | --- | --- |
| `<Q-n only when needed>` | `<specific gap>` | `<dependent work>` | `<sources and Orchestrator route>` | `<OPEN / RESOLVED with decision reference>` |

## Resume point

- Last update / actual candidate / current item: `<values>`
- Completed work and evidence: `<item IDs and paths or NONE>`
- Remaining work / blockers: `<IDs and actual state>`
- Next authorized action: `<one concrete step>`
- Output / earlier candidate and issue records: `<links/paths when they exist>`
