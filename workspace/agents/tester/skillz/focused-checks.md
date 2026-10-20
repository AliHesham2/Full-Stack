# Focused verification — usability, localization, privacy and recovery

Use from [SKILL.md](SKILL.md) and [verification matrix](verification.md) only for applicable assigned scope. These are procedures within Tester, not new roles, parallel approval gates or permission to change product policy. Use existing TODO/Output/check IDs and link evidence once. Missing required policy goes through Orchestrator to its owner; missing tools/data/access makes dependent execution BLOCKED/NOT RUN, not inapplicable or passed. Follow the existing phase and correction cycle.

## 1. Visual accuracy, usability and compatibility

Read approved Planner flows/ACs, the actual Designer TODO/shared Design System and relevant frontend guide sections. Use existing `visual-testing`, `cross-browser-testing` or `exploratory-testing` methods through the QA adapter rather than creating another visual baseline or tool setup.

For VISUAL, compare actual renders with the approved baseline at matching content/data, viewport, theme, locale, fonts and interaction state. Inspect spacing/alignment, hierarchy, typography, colors, assets, overflow, state and authorized motion. Record baseline/candidate identities, viewing conditions and comparable captures; investigate environment mismatch before blaming code. Only an approved tolerance or source rule can define an unacceptable difference. A screenshot cannot prove motion, keyboard use or an unseen state.

For USABILITY, name the actor, goal, starting state and approved expected completion. Walk the permitted task: discoverability of the next action, understandable labels/content, navigation/orientation, feedback/progress, validation, cancellation, destructive-action safeguards, error recovery and preservation of appropriate user input. Include relevant empty, loading, denial and failure states. A visually matching screen can fail these checks.

Separate a requirement violation from an evidence-backed usability concern and an optional preference. Do not invent success-rate/time targets, new business behavior or research participants. A Tester/AI walkthrough is heuristic inspection; a user-study claim needs actual authorized participant method, observations and agreed measures. Do not contact users or collect participant data from this procedure. Document review checks the specified behavior; observed interaction is needed for runtime claims.

For COMPAT, use the actual approved support matrix: browser/engine/version, OS when relevant, viewport/zoom, theme/direction, pointer/keyboard/touch and required device classes. Include boundaries around relevant layout changes, long content and critical states, not just one desktop screenshot. Distinguish actual hardware, device emulation and source inspection. A missing supported browser/device remains unverified, not silently unsupported. Never invent a support matrix or install a browser from a catalog entry.

Evidence: flow/AC and baseline IDs; actual comparison conditions or tested matrix; expected versus observed behavior; method (document/heuristic/user-study/actual render/device); redacted capture/check references and untested conditions. VISUAL, USABILITY, A11Y and COMPAT retain distinct results even if they share a capture.

## 2. Localization and internationalization

Use actual approved languages/regions, translations, fallback policy, direction, timezone and currency rules; missing required behavior is a source gap, not permission to write translations or choose a financial policy. Read relevant [layout guidance](../../../docs/frontend-guide/02-ui/layout-and-responsive.md), [accessibility guidance](../../../docs/frontend-guide/02-ui/accessibility.md) and [time/money rules](../../../docs/backend-guide/11-specialized-concerns/time-and-money.md).

Select relevant cases: language switching and retained state; missing-key/fallback behavior; plural/zero cases; long strings, truncation and wrapping; Arabic/Latin mixed text and unaltered identifiers; RTL reading/focus order, directional icons and logical layout; localized validation/errors and accessible labels; date/number/currency formatting and approved rounding; timezone/day-boundary/DST behavior where applicable; locale-specific input parsing. Verify code/IDs/URLs are not accidentally translated or reordered. Keep backend machine values and wire formats separate from presentation; formatted text is not a replacement contract.

Record the actual locale/direction/timezone/currency matrix and synthetic boundary inputs with approved expected results. Compare UI and API behavior where relevant. A viewport in RTL alone does not prove translated content or correct timezone handling. Check Arabic Markdown artifacts in their actual target viewer under project bidi rules; do not modify app UI styles or claim CSS alignment from Unicode controls.

Evidence: policy/source version, locales and input states, actual environment/renderer, cases/expected/actual, observed output and missing coverage. A native-language judgment requires a supported reviewer or reliable source; machine output alone is not proof of idiomatic translation.

## 3. Privacy and personal-data lifecycle

Read approved data inventory/retention/access/consent requirements and [privacy guidance](../../../docs/backend-guide/11-specialized-concerns/privacy-compliance-audit.md), relevant data/security/frontend rules and applicable `compliance-testing` methods through the adapter. A technical check does not decide legal obligations, lawful basis or certify compliance. Unresolved jurisdiction/policy questions return to the actual owner and qualified reviewer; no legal rule is inferred from an imported example.

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

Keep one TODO and one phase Output at their actual brief paths, with the selected stream/check IDs and evidence locations. Requirements/design/policy changes belong to their established owners. Report product defects through Orchestrator; do not edit another role's artifacts. Orchestrator personally verifies subsequent corrections under the shared cycle, using exact new candidates and these evidence requirements where relevant, without an automatic Tester redispatch.
