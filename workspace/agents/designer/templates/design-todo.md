# Design TODO for Implementer — <task ID> / design version <N>

> Designer owns this implementation specification. It is distinct from Designer's internal work TODO and Planner's software TODO. Populate actual scope; preserve one current design TODO, stable item IDs and version history. Precise specifications are design decisions, not proof of rendered measurements.

## Baseline and gate

| Field | Value |
| --- | --- |
| Designer / task / authorization / Output | `<actual references>` |
| Planner Output and software TODO versions | `<paths; requirement/AC/software item IDs>` |
| Existing kit / semantic tokens / guide sections | `<canonical references>` |
| Design artifacts / input reference matching intent | `<actual paths; match versus inspiration>` |
| Current handoff state / design Tester report | `<DRAFT / AWAITING REVIEW / REWORK / VERIFIED FOR IMPLEMENTATION / BLOCKED; actual report or NOT RUN>` |
| Orchestrator report disposition / correction evidence | `<actual references or PENDING>` |
| Design questions / scope or contract proposals | `<blocked IDs or NONE>` |

## Screen and flow inventory

| Design item ID | Requirement/AC and software item IDs | Route/screen/component and user goal | Entry, action, transition, exit and recovery | Artifact / dependent API/data IDs |
| --- | --- | --- | --- | --- |
| `<DT-n>` | `<canonical references>` | `<in-scope target>` | `<observable flow>` | `<actual source/version>` |

## Responsive layout and geometry

| Design item / element | Viewport interval and representative verification size | Container/grid/anchor and alignment | Insets, gaps, padding, margin and relative distances | Width/height, min/max and overflow | Resize rule / keyboard or safe-area behavior | Tokens/units and source |
| --- | --- | --- | --- | --- | --- | --- |
| `<DT-n; heading/form/input/action/etc.>` | `<exact project breakpoints; phone/tablet/desktop where in scope>` | `<parent edge, baseline or sibling reference>` | `<exact specified values or deterministic formula>` | `<fixed/fluid/intrinsic values and bounds>` | `<wrap/stack/clamp/scroll; intermediate widths>` | `<semantic token + resolved value when known; proposed value labeled>` |

Specify the relationship, not just a screenshot coordinate: e.g. distance from viewport/container edge to title, title-to-helper, helper-to-form, field-to-field, field-to-error, form-to-submit and submit-to-secondary action. For a login screen include the complete page/card/form/button/link/logo/content relationships, not only the card width. State values for the in-scope phone/tablet/desktop intervals and behavior between them, short/narrow screens, virtual keyboard, long text, zoom and RTL where relevant. Reference shared component rules once and record screen-specific overrides explicitly.

Use exact values where they are actual design choices; use token references, relative constraints and formulas where layout is fluid. Do not invent pixel measurements from an unmeasured reference, arbitrary breakpoints or inaccessible fixed heights. Label reference measurements as observed, inferred or proposed. Unknown essential values stay unresolved, never "make it responsive".

## Typography, visual treatment and assets

| Design item / text or surface | Font family/source, size/line-height/weight/tracking per interval | Color/background/border/radius/elevation token | Alignment, wrapping, truncation and content limits | Asset/icon identity, dimensions, fit and fallback |
| --- | --- | --- | --- | --- |
| `<DT-n; actual element>` | `<exact specified scale and behavior>` | `<existing tokens or approved proposal>` | `<long/translated copy rules>` | `<actual source or asset gap>` |

## Interaction, state and accessibility matrix

| Design item / condition | Trigger and expected visible behavior / exact copy source | State-specific geometry and feedback | Keyboard, focus, semantics and announcement | Validation/API dependency / recovery | Checkable acceptance or limitation |
| --- | --- | --- | --- | --- | --- |
| `<DT-n; default/hover/focus/disabled/loading/empty/error/denied/success as applicable>` | `<observable transition>` | `<layout stability, message placement, motion/reduced motion>` | `<labels, focus order/return, errors, touch targets>` | `<approved API/BR IDs; no invented contract>` | `<design criterion; runtime verification pending>` |

## Implementer instructions and review evidence

| Software item / design items | Existing component/token reuse and composition | Exact specification location and required states/intervals | Proposed file responsibility to reconcile with software plan | Design-phase test evidence / Orchestrator correction evidence | Remaining blocker |
| --- | --- | --- | --- | --- | --- |
| `<I-n / DT-n>` | `<approved kit>` | `<sections/artifacts>` | `<Planner owns software paths; no unilateral rename>` | `<actual reference or NOT RUN>` | `<specific gap or NONE>` |

Do not duplicate API/schema definitions or prescribe unsupported implementation syntax. Conflicts with software tasks, guide constraints or business behavior go through Orchestrator to the appropriate owner. Passing this specification gate does not prove the built UI, functional accessibility or product acceptance.
