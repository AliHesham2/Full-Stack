# Interpreting axe-core evidence

Use this procedure only when the assigned Tester task includes an accessibility scan and the project already has an approved, installed `axe-core` integration or explicitly authorizes adding one. [axe-core](https://github.com/dequelabs/axe-core) is a Web UI accessibility engine, **not an agent skill**. Its upstream license is [MPL-2.0](https://github.com/dequelabs/axe-core/blob/develop/LICENSE); this procedure does not vendor its code. It complements the existing [accessibility-testing skill](references/qa-skills-upstream/skills/accessibility-testing/SKILL.md) and the [frontend accessibility guide](../../../docs/frontend-guide/02-ui/accessibility.md), without replacing manual keyboard and assistive-technology checks.

## Establish what was scanned

Record the candidate revision, app/build, test environment, browser or JSDOM, axe engine and wrapper versions, route/component, rendered state, scan context, rule/tag configuration, relevant frames and evidence artifact. Scan the actual UI state under review: a closed dialog scan says nothing about the open dialog; an error page or redirect is not proof about the intended screen. Re-run after making hidden menus, dialogs, validation errors or asynchronous content visible when those states are in scope. Do not add Playwright E2E to this template unless the owner explicitly requests automated E2E.

If a frame, shadow-root region, authenticated state or other part of the intended UI was not reached, record the coverage gap. Do not count that area as scanned. Use the project's approved WCAG target and enabled-rule set; if filtering rules by tags, record the exact tags and confirm they cover every required level/version. A narrowly filtered run does not prove full WCAG coverage. See the official [axe API context and options](https://github.com/dequelabs/axe-core/blob/v4.14.0/doc/API.md).

## Interpret all four result categories

The official [results object](https://github.com/dequelabs/axe-core/blob/v4.14.0/doc/API.md#results-object) has `violations`, `incomplete`, `passes` and `inapplicable` arrays. Record counts and relevant rule IDs, nodes and help links, with sensitive content redacted.

| Result | Tester interpretation |
| --- | --- |
| `violations` | A rule found a problem in the scanned context. Reproduce and report the affected element/state and user impact. A zero count is only a pass for the rules and states actually evaluated. |
| `incomplete` | axe could not decide pass or fail for these nodes; assign targeted manual or browser follow-up. Until resolved, affected acceptance criteria remain `UNVERIFIED`, even if `violations` is empty. |
| `passes` | Specific rule/element checks passed in this scan. They do not certify the full page, all interactive states, or complete WCAG conformance. |
| `inapplicable` | No matching elements for those rules in this context; this is not a pass for content that may appear in another state. |

An execution error or a scan of the wrong page/state is `BLOCKED` or `NOT RUN`, not zero violations. The Tester Output should say exactly which rules/states were evaluated and which need human follow-up. Do not suppress a rule or exclude a region merely to make the scan green; any justified exception needs an explicit source, scope and owner decision.

## Environment-specific limits

- The local component-test guide uses `jest-axe` with JSDOM. [jest-axe documents](https://github.com/NickColley/jest-axe#checks-that-do-not-work-in-jest-axe) that color-contrast checks are disabled in JSDOM. Record contrast as **not verified by that check**; use the approved browser/manual method when contrast matters.
- axe analyzes rendered content and does not automatically cover inactive menus or closed modals. Its [API notes](https://github.com/dequelabs/axe-core/blob/v4.14.0/doc/API.md#api-notes) require exposing those states and re-running analysis.
- An automated scan cannot verify keyboard operability, meaningful focus order, screen-reader experience or all WCAG requirements. Route those to the existing manual accessibility procedure and record outstanding evidence.
- On a future real project, choose package versions from current stable releases compatible with the installed test runner and lockfile; verify official package data at install time. As checked on 2026-10-18, npm listed `axe-core` **4.14.0**, `@axe-core/playwright` **4.14.0**, and `jest-axe` **11.0.0**. These are observations, not dependencies installed by this template. A new axe minor may surface new findings; compare the same UI and rule set when upgrading. See [axe-core releases](https://github.com/dequelabs/axe-core/releases) and [release guidance](https://github.com/dequelabs/axe-core/blob/develop/doc/release-and-support.md).

## Output evidence

In the [Tester Output](../templates/output.md), record one scan row per route/component **and visible state**: candidate, environment, engine/wrapper versions, context and rule tags, counts of all four categories, violation IDs/elements, incomplete items and their review owner, unscanned regions, and manual follow-up. An accessibility AC is `PASS` only when its actual required automated and manual evidence is present; otherwise use `FAIL` or `UNVERIFIED` with the precise reason.
