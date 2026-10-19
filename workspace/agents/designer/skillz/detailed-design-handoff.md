# Detailed design TODO — implementation specification

Use after discovery in [SKILL.md](SKILL.md). Designer creates its own internal work TODO before design and authors the separate [design TODO](../templates/design-todo.md) for Implementer. Planner owns software construction tasks, file responsibilities, contracts and requirement definitions; Designer owns the visual/interaction specification within that approved scope.

## Specify the full surface

Inventory every in-scope route, screen, reusable component and meaningful condition. Link each `DT-n` design item to requirement/AC IDs, relevant software tasks, actual artifacts and canonical API/data constraints. Reference reusable definitions rather than repeating conflicting component measurements across screens.

For each screen specify hierarchy, navigation, exact content or approved source, alignment, grid, container bounds, width/height policy, padding/gaps, edge/sibling distances, typography, token usage, image/icon geometry, overflow, and composition. Include nested elements: a login form requires page/card placement, logo, heading/subtitle, labels/fields, helper/error rows, password action, checkbox/link if required, primary action and secondary navigation if in scope. This is a completeness example, not permission to add those controls to a product that does not require them.

State the origin of each distance: viewport edge, container inset, sibling edge or text baseline. Use exact units/token references and resolved values where known. If matching an image, distinguish actual measured values from inferred/proposed values and state source size/scale limitations. Do not claim pixel measurements or render checks you did not perform. If existing kit/tokens define a value, link that source and name any justified override rather than inventing a second scale.

## Specify responsive rules rather than only device screenshots

Use actual project breakpoints; if missing, propose explicit intervals for approval. Define representative in-scope phone/tablet/desktop sizes and deterministic behavior across intermediate widths: fixed/fluid/intrinsic sizing, min/max, container gutters, grid changes, stacking, wrapping, alignment, visibility, scrolling and sticky/fixed behavior. Address short screens, safe areas and virtual keyboards when relevant. Do not leave tablet unspecified or use arbitrary device names without intervals.

For text include font/source/fallback, size, line height, weight, tracking and wrapping/truncation. Account for long or translated strings, RTL/LTR, zoom and text resizing. Prefer constraints and token/formula-based rules to fragile absolute coordinates or fixed heights that clip content. Precise responsive intent may be a formula or min/max bound; precision does not mean every coordinate must be a constant.

## Specify behavior and all relevant states

For each interactive element define default/hover/focus/active/disabled/loading/error/success and any applicable empty, no-results, denied, stale or recovery state. Include trigger, exact visible result, copy/source, geometry changes, layout stability and API/business dependencies. Record keyboard interactions, semantic role/label, focus order and return, error association/announcement, touch targets and reduced-motion behavior. Do not invent permissions, validation rules or responses missing from Planner's approved baseline.

Every value or rule needed for implementation must be specified, linked to an actual shared definition or explicitly blocked. Avoid vague directions such as "modern", "space nicely" or "responsive" without checkable rules. Define justified visual comparison criteria where needed; do not invent a global pixel tolerance or numerical similarity promise.

## Review and deliver

Self-check requirement coverage, software/API compatibility, guides, kit reuse, internal consistency and complete responsive/state matrices. Inspect actual visual artifacts with available authorized tools; label a text-only specification honestly. Missing source/assets/decisions are explicit gaps, not fabricated examples.

Deliver the versioned design TODO, Output and artifacts to Orchestrator. Under [delivery cycle](../../orchestrator/skillz/delivery-cycle.md), Tester reviews this specification first; Orchestrator independently checks each result. Correct confirmed issues through a formal brief, preserve stable IDs/history and return exact changed sections and new candidate evidence. Orchestrator personally verifies the corrected design and affected scope; no automatic return to Tester. Design-gate success permits the approved handoff, not a claim that a built interface has been tested.
