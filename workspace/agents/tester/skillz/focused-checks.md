# Focused verification — usability, localization, privacy and recovery

Use from [SKILL.md](SKILL.md) and [verification matrix](verification.md) only for applicable assigned scope. These are procedures within Tester, not new roles, parallel approval gates or permission to change product policy. Use existing TODO/Output/check IDs and link evidence once. Missing required policy goes through Orchestrator to its owner; missing tools/data/access makes dependent execution BLOCKED/NOT RUN, not inapplicable or passed. Follow the existing phase and correction cycle.

Read [frontend-only scope](../../../docs/frontend-guide/00-navigation/frontend-only-scope.md). This project has no backend guide or database implementation.

## 1. Visual accuracy, usability and compatibility

Read approved Planner flows/ACs, the actual Designer TODO/shared Design System and relevant frontend guide sections. Use existing `visual-testing`, `cross-browser-testing` or `exploratory-testing` methods through the QA adapter rather than creating another visual baseline or tool setup.

For VISUAL, compare actual renders with the approved baseline at matching content/data, viewport, theme, locale, fonts and interaction state. Inspect spacing/alignment, hierarchy, typography, colors, assets, overflow, state and authorized motion. Record baseline/candidate identities, viewing conditions and comparable captures; investigate environment mismatch before blaming code. Only an approved tolerance or source rule can define an unacceptable difference. A screenshot cannot prove motion, keyboard use or an unseen state.

For USABILITY, name the actor, goal, starting state and approved expected completion. Walk the permitted task: discoverability of the next action, understandable labels/content, navigation/orientation, feedback/progress, validation, cancellation, destructive-action safeguards, error recovery and preservation of appropriate user input. Include relevant empty, loading, denial and failure states. A visually matching screen can fail these checks.

Separate a requirement violation from an evidence-backed usability concern and an optional preference. Do not invent success-rate/time targets, new business behavior or research participants. A Tester/AI walkthrough is heuristic inspection; a user-study claim needs actual authorized participant method, observations and agreed measures. Do not contact users or collect participant data from this procedure. Document review checks the specified behavior; observed interaction is needed for runtime claims.

For COMPAT, use the actual approved support matrix: browser/engine/version, OS when relevant, viewport/zoom, theme/direction, pointer/keyboard/touch and required device classes. Include boundaries around relevant layout changes, long content and critical states, not just one desktop screenshot. Distinguish actual hardware, device emulation and source inspection. A missing supported browser/device remains unverified, not silently unsupported. Never invent a support matrix or install a browser from a catalog entry.

Evidence: flow/AC and baseline IDs; actual comparison conditions or tested matrix; expected versus observed behavior; method (document/heuristic/user-study/actual render/device); redacted capture/check references and untested conditions. VISUAL, USABILITY, A11Y and COMPAT retain distinct results even if they share a capture.

## 2. Localization and internationalization

Use actual approved languages/regions, translations, fallback policy, direction, timezone and currency rules; missing required behavior is a source gap, not permission to write translations or choose a financial policy. Read relevant [layout guidance](../../../docs/frontend-guide/02-ui/layout-and-responsive.md), [accessibility guidance](../../../docs/frontend-guide/02-ui/accessibility.md) and exact owner-supplied time/money representations and approved frontend formatting rules.

Select relevant cases: language switching and retained state; missing-key/fallback behavior; plural/zero cases; long strings, truncation and wrapping; Arabic/Latin mixed text and unaltered identifiers; RTL reading/focus order, directional icons and logical layout; localized validation/errors and accessible labels; date/number/currency formatting and approved rounding; timezone/day-boundary/DST behavior where applicable; locale-specific input parsing. Verify code/IDs/URLs are not accidentally translated or reordered. Keep backend machine values and wire formats separate from presentation; formatted text is not a replacement contract.

Record the actual locale/direction/timezone/currency matrix and synthetic boundary inputs with approved expected results. Compare UI and API behavior where relevant. A viewport in RTL alone does not prove translated content or correct timezone handling. Check Arabic Markdown artifacts in their actual target viewer under project bidi rules; do not modify app UI styles or claim CSS alignment from Unicode controls.

Evidence: policy/source version, locales and input states, actual environment/renderer, cases/expected/actual, observed output and missing coverage. A native-language judgment requires a supported reviewer or reliable source; machine output alone is not proof of idiomatic translation.

## 3. Frontend privacy and personal data

Read approved frontend data/retention/consent/access requirements, [security](../../../docs/frontend-guide/04-quality/security.md), [integration](../../../docs/frontend-guide/03-data/backend-integration.md) and applicable compliance-testing methods through the adapter. Technical checks do not select jurisdiction/legal obligations or certify compliance; missing policy returns through Orchestrator to the owner.

Trace synthetic/approved affected data through forms, client/Server Component props, API/action responses, URL/query strings, browser storage/cache, Next.js caches/logs and actual authorized third-party scripts. Check minimization, safe serialization/redaction, cross-user cache leakage, consent/withdrawal behavior and sensitive failure states. No production PII or real credentials in evidence. No backend code, DB/files/backups or external provider account is inspected by this procedure.

For supplied deletion/export/correction operations verify the approved frontend request/auth, user feedback, local state/cache invalidation and denied/repeated/uncertain outcomes. A 204 or vanished screen does not prove deletion in the service, backups or other external stores. Report those as external contract guarantees/evidence limits or questions to the owner; do not create backend tests or claim full lifecycle coverage. Retention/client cleanup expectations need actual approved policy and verifiable timing, not invented deadlines.

Evidence: supplied contract/policy version, synthetic fixture/actor, actual allowed frontend surfaces, expected/observed exposure or cleanup, timing/redacted captures and inaccessible external boundaries. Do not keep sensitive exports as proof.

## 4. Frontend recovery boundary

External DB/provider backups, PITR, transaction recovery and whole-system RPO/RTO drills are OUT OF SCOPE for this frontend-only starter. Their absence does not block an otherwise valid frontend task. Do not import the backend restore procedure, require backend guides or claim a mock proves external recovery.

Only when actually assigned, verify restoring an approved frontend build/config artifact or relevant frontend state in an authorized isolated target. Require exact artifact/candidate identity, supported API contract compatibility, permitted variables/config sources, recovery expectations and resource/cleanup authority. No production overwrite, backend mutation or infrastructure creation follows from this procedure. Ensure the restored frontend uses permitted API targets/accounts and cannot trigger unapproved jobs, analytics or outbound calls before startup.

Record actual restore/redeploy steps and timings, startup/render/API-adapter smoke checks within authority, preserved or cleared local state/cache as specified, and compatibility/failure limits. RTO-like frontend recovery targets apply only if actually approved; do not invent a system-wide RPO. An artifact/config review is document evidence, not an observed restoration or deployed release. Other scope is justified NOT APPLICABLE; missing required frontend tools/authority stays BLOCKED/NOT RUN. Stop/remove only verified task-owned disposable resources and retain redacted evidence.

## 5. Reporting and changes

Keep one TODO and one phase Output at their actual brief paths, with the selected stream/check IDs and evidence locations. Requirements/design/policy changes belong to their established owners. Report product defects through Orchestrator; do not edit another role's artifacts. Orchestrator personally verifies subsequent corrections under the shared cycle, using exact new candidates and these evidence requirements where relevant, without an automatic Tester redispatch.
