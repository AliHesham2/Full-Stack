# Software TODO — <task ID> / candidate <identity>

> Planner prepares this software construction plan after analysis. Implementer validates and continues the same authorized file; create one only if no plan was supplied within an approved task. Designer owns the separate detailed design TODO, which must also be read for UI work. Preserve approved scope and add actual execution information without invented results. Paths are from project root unless stated otherwise.

## Assignment and scope

| Field | Value |
| --- | --- |
| Task / assigned operator / assignment reference | `<IDs and actual authority>` |
| Approved requirements / task / actual design versions | `<current Planner Output, software TODO baseline, actual design TODO and Designer Output; design Tester report and Orchestrator disposition/correction evidence; IDs>` |
| Role entry / handoff state / design prerequisites | `<workspace/agents/implementer/prompt.md; WAITING FOR DESIGN or actual review disposition>` |
| Syntax entry and selected route | `<workspace/docs/syntax/GUIDE-MAP.md; actual indexes/pages read>` |
| Goal, affected apps and exclusions | `<bounded scope>` |
| Outcome slices / inherited shared constraints / required source material | `<Planner Output SL/DEC IDs and source paths/sections; actual prerequisite states>` |
| Allowed files, tools, environment/data and reporting route | `<actual permissions>` |
| TODO / Output / evidence locations | `<authorized paths>` |
| Baseline and intended candidate identity method | `<commit or changed-file manifest; no invented revision>` |
| Canonical implementation map / inspected branch | `<existing path or authorized docs/PROJECT-MAP.md; observed scope>` |

## Baseline and ownership

| Area / file | Existing behavior, edit or failure | Evidence and candidate | Owner / consequence for this task |
| --- | --- | --- | --- |
| `<app/path or NONE OBSERVED>` | `<observed fact or NOT RUN>` | `<command/output/path>` | `<preserve / blocked / task prerequisite>` |

## Implementation and source map

| Item ID | Requirement / module / design IDs | Planned change and allowed files | Guide file/heading | Local syntax file/heading and applicable version | Dependencies, negative/boundary cases and risk | Planned verification | Status |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `<I-n>` | `<actual IDs>` | `<small coherent change>` | `<frontend/backend rule>` | `<workspace/docs/syntax/...; read before code>` | `<named failure classes and prerequisites>` | `<existing check/inspection; Tester gap if needed>` | `TODO` |

Use `TODO`, `IN PROGRESS`, `BLOCKED`, `REPORTED`. `REPORTED` means evidence was delivered, not accepted. Source gaps block the dependent item under `workspace/agents/implementer/skillz/syntax-policy.md`.

## Consumed and produced boundaries

| Implementation item | Consumes: interface/artifact ID, version and producer item | Produces: interface/artifact ID and consumer item | Exact names, fields/types/signatures or canonical definition | Compatibility check / missing design or source prerequisite |
| --- | --- | --- | --- | --- |
| `<I-n>` | `<actual requirements/contract/reviewed design>` | `<required public boundary and downstream task>` | `<source-defined boundary; no duplicated implementation>` | `<match evidence or explicit blocker>` |

Use reviewable deliverables rather than arbitrary tiny steps. Supporting setup belongs with the work that needs it unless independently useful. Compare producer/consumer names and shapes, preserve approved values and local syntax support, and leave unresolved design/interfaces blocked. Link prioritized failure cases to their owning task and Tester verification; do not write tests or production bodies in this plan.

## Supplied external API contracts — organized by Planner before execution

Use [Planner's contract preparation procedure](../../planner/skillz/software-contracts.md). The owner supplies external contracts. This section holds labeled read-only extractions with exact source/version or mandatory complete attachments; Planner does not author upstream definitions. Frontend transport/adapter proposals are separate and need actual approval. A derived snapshot must be labeled read-only with its source/version. Planner Output inventories and explains these same definitions; do not maintain independently editable duplicates. Use `NOT APPLICABLE` with a scope reason if no API is affected.

### Operation inventory

| API ID / name and purpose | Method / exact path / boundary | Owning module / implementation item and paths | Callers / requirement and AC IDs | Owner source/version and approval; extraction verification | Linked frontend-data/SEC IDs / readiness gap |
| --- | --- | --- | --- | --- | --- |
| `<actual API-n or NOT APPLICABLE>` | `<external operation versus proposed Next.js relay/action made explicit>` | `<I-n; exact planned files>` | `<consumers and IDs>` | `<owner source/section/version; read-only extraction check>` | `<resolved prerequisites or BLOCKED>` |

### Complete operation definition — repeat for each actual API ID

Operation/version: `<API ID and approved definition>`; identity/roles/object/tenant rules: `<exact policy and enforcement>`; request media type/unknown-field handling/payload bounds: `<approved rules or explicit gap>`.

| Request location | Exact field/header/cookie name | Type/format, nested schema or allowed values | Required/optional, nullable/absent and default behavior | Validation/normalization/encoding and bounds | Invalid/missing outcome / source |
| --- | --- | --- | --- | --- | --- |
| `<PATH / QUERY / HEADER / COOKIE / BODY; state NONE for absent locations>` | `<exact name>` | `<complete canonical type/schema>` | `<precise semantics>` | `<approved rule, not guessed limits>` | `<declared error mapping>` |

| Outcome / triggering condition | HTTP status | Response headers/cookies and boundary-specific handling | Complete body schema / fields, types, required/nullability or NO BODY | Synthetic request/response example or exact complete attachment | Observable side effects / API-data-SEC-AC references |
| --- | --- | --- | --- | --- | --- |
| `<each intended SUCCESS case>` | `<exact status>` | `<defined headers/cookies or NONE>` | `<exact representation; 204 has no body>` | `<schema-consistent synthetic example>` | `<supplied side effects; frontend state/cache and UI handling>` |
| `<each declared applicable FAILURE case>` | `<exact status>` | `<correlation/retry headers when applicable>` | `<original supplied wire envelope + approved frontend mapping>` | `<complete example with no internals/secrets>` | `<trigger/denial/operational behavior and ACs>` |

Pagination/filter/sort vocabulary/defaults: `<exact definitions or NOT APPLICABLE>`.
Idempotency/concurrency/retry/timeout and cache/security-header policy: `<settled applicable rules, source and responsibility at each boundary>`.
Compatibility/version/dependent consumer changes: `<approved change and synchronization route or NONE>`.
Do not use prose such as "returns user", "handle failures" or a bare status as a complete contract. Missing fields/statuses/policy are Planner blockers, not Implementer choices. Keep browser-facing behavior distinct from the Next.js → external API response.

### Unified frontend error mapping — distinct from supplied wire format

Original error/status source per API: `<exact owner definitions; no imposed envelope>`.
Approved frontend error model: `<exact fields/types/optionality, safe messages and source/decision; BLOCKED if unsettled>`.

| API / wire status, code and trigger | Original body/details/header semantics and source version | Frontend safe category / code / message / field mapping | UI/session/cache/error-boundary behavior | Correlation/retry/uncertain-outcome treatment | AC/check and approval |
| --- | --- | --- | --- | --- | --- |
| `<each applicable supplied outcome>` | `<original wire format; no altered upstream contract>` | `<one consistent local mapping>` | `<precise behavior>` | `<approved policy; no tokens/raw bodies>` | `<IDs>` |
| `<network/timeout/cancellation/malformed or unknown response>` | `<local failure versus supplied status distinguished>` | `<safe generic policy; no invented server code>` | `<defined recovery and accessible feedback>` | `<no blind mutation retry>` | `<IDs>` |

Success schemas remain per operation, including NO BODY responses. Never disguise an outage as invalid credentials or expose internal diagnostics to the browser. API-free scope needs no manufactured dictionary.

## Frontend data and state plan — prepared by Planner

No database tables, SQL, keys/indexes, migrations or backend code are planned here. DATA IDs identify supplied wire DTOs and frontend state only. Use NOT APPLICABLE with a reason for purely presentational scope.

| DATA ID / supplied DTO and source version | Exact fields/types/required/nullability or mandatory schema attachment | Parsing / approved view transformation / server-only exclusions | Frontend consumer and state owner | API/I/SEC/AC references and gaps |
| --- | --- | --- | --- | --- |
| `<actual DATA-n>` | `<complete read-only source extraction>` | `<exact supported rules>` | `<server/URL/form/component/query cache; paths>` | `<settled definitions or BLOCKED>` |

| State/query owner and key | Loading/empty/error/success/stale/cancellation behavior | Cache scope/freshness and sensitive-data policy | Mutation invalidation / optimistic rollback / uncertain outcome | Sources/approval and verification |
| --- | --- | --- | --- | --- |
| `<actual state and consumers>` | `<explicit states>` | `<approved settings or gap>` | `<approved behavior or NOT APPLICABLE>` | `<IDs and exact source>` |

Frontend data flow: `<actual Browser/Next.js/external operation/DTO consumers; no inferred database schema>`.

## Security plan — prepared by Planner

Assets, actors and trust boundaries: `<actual affected surfaces; no assumed infrastructure>`.
Relevant threat/abuse scenarios and source checklist: `<concrete scenarios plus task-relevant guide sections>`.

| Security ID / protected API/DATA/software items | Threat/denial scenario and source policy | Required concrete control / exact enforcement location | Approved settings/limits and secret/data handling | Observable negative/boundary verification / owner | Status / missing decision or justified exclusion |
| --- | --- | --- | --- | --- | --- |
| `<actual SEC-n>` | `<source-backed risk, not a presumed vulnerability>` | `<what must be enforced and where; not "secure it">` | `<settled values or BLOCKED; no secret values>` | `<expected outcome and AC/check IDs>` | `<actual readiness>` |

Address applicable authentication, object/tenant authorization, server-owned inputs, validation/injection/output safety, session/cookie/CSRF/service transport, secret/PII/log/error exposure, abuse/resource bounds and outbound/file/dependency risks. Reference existing mandatory policy rather than invent controls or defaults; record exclusions with reasons. Missing security decisions block affected work. This plan is not evidence of compliance, a scan or an accepted security exception.

## Planner contract-readiness gate

| Execution item | Complete approved API definitions / shared errors and successes | Complete frontend DTO/state/error mappings | Concrete security controls | File responsibilities / local syntax and actual design prerequisites | Decision evidence / missing detail and owner | Disposition |
| --- | --- | --- | --- | --- | --- | --- |
| `<I-n>` | `<API IDs/version or justified NOT APPLICABLE>` | `<DATA IDs/version or justified NOT APPLICABLE>` | `<SEC IDs or justified exclusion>` | `<exact satisfied dependencies or gaps>` | `<actual references, not presumed approval>` | `<READY FOR ORCHESTRATOR REVIEW / BLOCKED / WAITING FOR DESIGN>` |

No ready item contains unresolved contract/frontend-state/security choices, placeholder fields or unexplained `TBD`. Orchestrator reviews this gate before dispatch. Implementer validates and executes the approved plan, reports inconsistencies and blocks dependent work; it does not invent endpoints, request/response shapes, external contracts or database structure, security policy or change requirements to make implementation easier. Internal code mechanics still require the approved guides/local syntax and self-verification.

## Integration and side-effect plan

| Concern | Existing owner and consumers | Approved change / compatibility and ordering | Recovery or irreversible effect | Execution permission / open decision |
| --- | --- | --- | --- | --- |
| `<frontend transport/DTO/types/tokens/dependency/config or NOT APPLICABLE>` | `<actual paths>` | `<scope and synchronization route>` | `<supported recovery, or unknown>` | `<prepare only / permitted environment / BLOCKED>` |

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
