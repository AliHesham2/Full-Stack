# Focused verification — usability, localization, privacy and recovery

Use from [SKILL.md](SKILL.md) and [verification matrix](verification.md) only for applicable assigned scope. These are procedures within Tester, not new roles, parallel approval gates or permission to change product policy. Use existing TODO/Output/check IDs and link evidence once. Missing required policy goes through Orchestrator to its owner; missing tools/data/access makes dependent execution BLOCKED/NOT RUN, not inapplicable or passed. Follow the existing phase and correction cycle.

Read [backend-only scope](../../../docs/backend-guide/00-navigation/backend-only-scope.md). No UI Designer, frontend guide, browser test or visual artifact is introduced.

## 1. API consumer usability and service compatibility

USABILITY here means the developer/caller experience of approved APIs, not screen design. Read exact requirements, contracts/docs/examples and expected caller tasks. Check scoped operations can be understood and exercised from supplied documentation: names/auth/preconditions, required fields, success/error feedback, pagination/idempotency/recovery and complete safe examples. Separate a contract/documentation defect from an optional style preference. Do not invent business behavior, client code, research participants or usability measures. A document/heuristic review is not an actual user study or runtime API test.

For COMPAT use actual approved runtime/library/provider and consumer-contract support: Node.js/Express versions, declared HTTP/media/encoding behavior, serializers and relevant backend dependencies. Record actual versions/targets tested; unsupported combinations are not silently added, and missing required environments stay unverified. No UI/browser/device matrix is required. Consumer application code remains external unless separately authorized.

Evidence: actual API task/AC and exact contract/docs versions, synthetic request/example, approved support versions, method/expected/actual behavior and redacted output. Keep VISUAL/A11Y excluded by backend-only scope; neither API docs nor an HTTP test creates a UI/WCAG assignment.

## 2. API localization and canonical data

Read approved locale/language/fallback and wire representations, plus [time/money guidance](../../../docs/backend-guide/11-specialized-concerns/time-and-money.md). Verify applicable Unicode/mixed-text round trips, accepted locale hints, supplied translated-content/fallback rules, machine-readable error codes, date/timezone/day-boundary/DST and decimal/currency/rounding invariants. Do not localize identifiers, translate stable codes or replace canonical numbers/timestamps with UI formatting. Never choose a financial or translation policy to fill a gap.

Record exact locale/timezone/currency cases and synthetic boundary inputs, source-defined expected status/headers/body/schema or data invariants, actual results and coverage limits. No RTL layout, typography, keyboard or screen comparison is assigned. A verified wire representation is not proof of idiomatic translations or client rendering. Missing required rules return to Planner/the owner; a database-free endpoint needs no invented tables.

## 3. Privacy and personal-data lifecycle

Read approved data inventory/retention/access/consent requirements and [privacy guidance](../../../docs/backend-guide/11-specialized-concerns/privacy-compliance-audit.md), relevant backend data/security rules and applicable `compliance-testing` methods through the adapter. A technical check does not decide legal obligations, lawful basis or certify compliance. Unresolved jurisdiction/policy questions return to the actual owner and qualified reviewer; no legal rule is inferred from an imported example.

Trace only affected data classes through allowed request/response fields, DB/files, caches, logs/traces, analytics/third parties, exports and backups. Use synthetic or explicitly approved data; never fetch production PII to populate a checklist. Check minimization and server/client field exposure, cross-user/tenant denial, safe errors/logs, and approved consent/withdrawal behavior where applicable.

For scoped lifecycle operations, verify authorized export/correction/deletion and retention expiry across the actual approved stores/derived copies, with documented exceptions and async timing. Test denied and repeated requests where relevant. Do not infer deletion from a vanished UI row or a 204 response. Explain what was actually checked in external services versus doubled/unavailable boundaries. Preserve legitimate audit/retention exceptions only as the approved policy defines them; do not invent erasure deadlines or remove protected evidence.

Coordinate privacy and recovery checks: an approved restoration process must account for previously erased data so it is not silently reintroduced into service. Check the required tombstone/re-erasure/restriction behavior in the authorized isolated target, not by deleting an original backup. Missing policy/implementation remains a named gap.

Evidence: synthetic data class/fixture, operation/actor/tenant, store inventory/version and retention rule, expected exposure/deletion outcome, actual observed locations/timing, redacted evidence and uninspected copies. Do not retain real personal data or secret-bearing exports as proof.

## 4. Backup and recovery

Read the actual approved runbook, affected data/service inventory, RPO/RTO targets and [disaster-recovery guidance](../../../docs/backend-guide/11-specialized-concerns/disaster-recovery.md). A startup template or database-free task need not have a restore drill. A real critical persistence change cannot waive recovery coverage merely because the project is small. Targets come from the actual owner; examples in the guide are not approved objectives.

Before executing, require explicit authority for the exact backup/source classification, isolated target, restore/PITR actions, resource/cost bounds and safe cleanup. Confirm environment identities, source preservation and target isolation; never overwrite the source, production or another operator's environment. Restore may import credentials, schedules, webhooks and jobs: prevent unapproved egress and external effects before starting the restored service, and use approved isolated configuration. Missing isolation/permission blocks the drill; do not improvise infrastructure or load a production snapshot without authorization.

Within that authority: identify the actual backup and recovery point, restore to the isolated target, verify relevant schema/constraints/relationships and meaningful data invariants, then exercise approved startup/auth/API smoke paths. A matching row count alone is insufficient. Measure elapsed recovery against the defined start/end conditions and RTO; estimate the actual data-loss window against RPO using known fixture/checkpoint times. Distinguish snapshot restore from PITR, DB recovery from complete service recovery, and measured results from estimates. Include approved deletion-on-restore/privacy checks where required.

Evidence: safe backup identity/recovery point, target and isolation controls, runbook/candidate versions, actual steps/commands/time observations, integrity and service smoke results, measured RPO/RTO comparison, unavailable dependencies and remaining configuration gaps. A successful backup job, runbook review, mock or unexecuted command is not a verified restore. Stop/clean only verified task-owned resources within actual authority and retain redacted evidence; document authorized resources left for handoff.

## 5. Reporting and changes

Keep one TODO and one phase Output at their actual brief paths, with the selected stream/check IDs and evidence locations. Requirements/contract/schema/policy changes belong to their established owners. Report product defects through Orchestrator; do not edit another role's artifacts. Orchestrator personally verifies subsequent corrections under the shared cycle, using exact new candidates and these evidence requirements where relevant, without an automatic Tester redispatch.
