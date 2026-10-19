# Designer — review implementation against the approved design

Use this after [SKILL.md](SKILL.md) when the Orchestrator assigns a review of an implemented UI. This is a later review phase, not an instruction to start an app or trial the template now. Keep the same task's TODO and Output, with a dated review round and explicit candidate identity; do not create a second acceptance record.

## 1. Establish the comparison

Read the approved design artifact/version, Planner Output IDs, recorded design decisions and Implementer's handoff. Identify the actual implementation commit/build and the routes/components in scope. A screenshot in [input/](../input/README.md) is the baseline only when the owner or approved brief designates it as such. Record whether each reference requires close visual matching, specific attributes, or inspiration; do not assume pixel equality from an inspiration image. If no visual baseline exists, review observable criteria from the approved textual specification and state that limitation.

Plan the comparison cases in the [TODO](../templates/todo.md). Each `VR-n` case identifies a screen/component, state, viewport in CSS pixels, theme, language/direction, expected result and baseline location. Include the conditions required by the design/Planner Output: relevant phone/tablet/desktop layouts, loading/empty/error/disabled states, long content, zoom and reduced motion. Link functional checks already owned by Tester rather than duplicating its suite. Missing implementation or baseline means `UNVERIFIED`, not a passed review or an invented result.

## 2. Inspect comparable renders

Use the authorized existing browser/viewer and artifact paths. Follow the actual app's startup instructions only when the assignment authorizes local execution. Use supplied fixtures or a permitted non-destructive state; ask for missing access rather than changing accounts, production data or contracts to obtain a screenshot. Do not install a browser, add visual-test dependencies, write automated E2E tests, or change source code through this review procedure. The [frontend testing guide](../../../docs/frontend-guide/04-quality/testing.md) remains authoritative for test ownership and automation.

Record browser/viewer, viewport, zoom, device-pixel ratio when available, theme, locale/direction, data state and capture time. Wait for required fonts/assets and the intended stable UI state before comparing; preserve intentional loading cases separately. Mark unavailable conditions explicitly. Compare at corresponding scale and inspect the whole page, then important regions: hierarchy and layout; alignment, spacing and density; typography and wrapping; semantic colors and contrast; images, crop and icons; component states; overflow and responsive/RTL adaptation. Review motion only from an observed interaction or recording; a still image does not prove it.

Use side-by-side images or an authorized overlay/diff viewer when useful. Store baseline references and actual captures at the task's approved evidence path, clearly labeled with case ID and implementation candidate. Do not modify owner inputs. Differences caused by mismatched fonts, content, viewport or rendering conditions must be investigated before reporting a design defect. Any tolerance must come from the approved criteria or a recorded decision; never invent a percentage score or require exact pixels without an agreed basis.

## 3. Report differences and route fixes

Record each case as `MATCH`, `MISMATCH`, `UNVERIFIED` or `NOT APPLICABLE`, with evidence or a reason. `MATCH` covers only the named visual criteria under observed conditions. For each mismatch, use a stable `VD-n` ID, reference the `VR-n` case and requirement/design location, show expected versus actual with capture paths/region, describe user impact, and state the observable correction. Cite source `file:line` only when actually inspected; do not guess the code cause from a screenshot.

Send findings through the brief's authorized route to Orchestrator. Implementer owns code corrections; a requirement/design conflict needs a decision from its established owner. Designer does not appoint workers, accept a feature, or waive missing Tester evidence. Keyboard, screen-reader, API, security and performance claims remain with their required verification routes; visual observations may identify concerns but cannot establish compliance.

## 4. Recheck after correction

For each new implementation candidate, rerun failed cases and affected neighboring/shared-component cases under the recorded comparison conditions. Record the new candidate, new evidence and per-issue result; keep earlier findings and captures linked. Close a `VD-n` only after observing its correction. If the baseline must change, obtain an approved design/requirement decision and retain the old version with the reason; never update a reference merely to make the implementation match.

Repeated or reopened mismatches require root-cause investigation through Orchestrator and Implementer, not blind visual patching. Finish with checked scope, remaining mismatches, unverified conditions and a recommendation in the [Output](../templates/output.md). A design candidate can be handed off before implementation exists, but its later implementation review remains explicitly `NOT RUN` until performed. Visual review is one input to the Orchestrator's decision and the owner's acceptance.
