# Design TODO for Implementer — <task ID> / design version <N>

> Designer owns this implementation specification. It is distinct from Designer's internal work TODO and Planner's software TODO. Populate actual scope; preserve one current design TODO, stable item IDs and version history. Precise specifications are design decisions, not proof of rendered measurements.

## Baseline and gate

| Field | Value |
| --- | --- |
| Designer / task / authorization / Output | `<actual references>` |
| Planner Output and software TODO versions | `<paths; requirement/AC/software item IDs>` |
| Existing kit / semantic tokens / guide sections | `<canonical references>` |
| Complete textual specification sources and approved decisions | `<exact sections, requirement/design IDs, source versions and owner decision references>` |
| Current handoff state / design Tester report | `<DRAFT / AWAITING REVIEW / REWORK / VERIFIED FOR IMPLEMENTATION / BLOCKED; actual report or NOT RUN>` |
| Orchestrator report disposition / correction evidence | `<actual references or PENDING>` |
| Design questions / scope or contract proposals | `<blocked IDs or NONE>` |

## Shared design system — constant reference for this candidate

Define shared visual decisions here once; each screen/element below references these tokens and component variants, listing only explicit approved overrides. This is a readable specification of the actual system, not a second independently editable theme/component source. Identify exact canonical source versions. Existing kit/token rules apply; new or changed values remain proposals until approved. Unresolved required values block the affected handoff rather than being chosen by Implementer.

### System baseline and authority

| Scope / themes / locales | Canonical theme, token, font and component sources / exact versions | Existing / proposed change and decision reference | Affected screens / shared decision IDs | Source gaps / readiness |
| --- | --- | --- | --- | --- |
| `<actual candidate scope>` | `<actual paths/sections and immutable version or manifest; NOT INITIALIZED where true>` | `<approved reuse versus proposed extension>` | `<DT/DS/requirement IDs>` | `<resolved or BLOCKED>` |

### Complete active color palette

| Semantic token / role and usage | Paired foreground/background or border/focus counterpart | Light value or exact canonical expression | Dark value or exact canonical expression / scope exception | Canonical definition / status | Contrast target and actual evidence state |
| --- | --- | --- | --- | --- | --- |
| `<actual token: page/surface/text/brand/status/line/focus/etc.>` | `<actual pair; no color-only meaning>` | `<source-backed value; explicitly proposed if not established>` | `<source-backed value or justified theme NOT APPLICABLE>` | `<theme source/version; approved/proposed/gap>` | `<required target; measured evidence or UNVERIFIED>` |

Inventory the complete active project palette and its applicable theme pairs, including foreground, background, surface, primary/secondary, muted/accent, status, input, border and ring roles where defined. Do not invent missing colors or create every possible status token for an unrelated feature. Displayed values are a read-only rendering of the canonical definitions; code consumes semantic tokens, with raw brand values owned by the project's theme source. Changes require reconciliation with that source/approved proposal, never silent component literals.

### Typography system

| Text role / type token | Family, source and fallback / locale | Size, line-height, weight and tracking | Viewport/container intervals and wrap/overflow rules | Canonical definition / approval and gap |
| --- | --- | --- | --- | --- |
| `<heading/body/label/helper/error/action/etc.>` | `<actual font roles/assets or unresolved source>` | `<exact scale values/expressions, not "large">` | `<defined shared behavior and content limits>` | `<source/version; proposal or actual baseline>` |

### Spacing, layout and effects system

| Shared category / token or rule | Exact value/scale/expression and units | Applicable intervals / themes / direction | Usage / constraint | Canonical source / status |
| --- | --- | --- | --- | --- |
| `<spacing step, container/gutter/grid/breakpoint, radius, border, elevation, icon size or layer>` | `<actual existing scale/definition or approved extension>` | `<deterministic conditions>` | `<where it applies; no contradictory screen defaults>` | `<exact source/version and decision>` |

### Component styles and variants

| Component / canonical source | Variant and intended usage | Size, min target, padding, gaps and icon geometry | Typography / fill, foreground, border, radius, shadow tokens | Responsive, content and composition rules | State definition / approval and coverage |
| --- | --- | --- | --- | --- | --- |
| `<Button/Input/FormField/Card/etc. actually used>` | `<exact existing variant or justified approved extension>` | `<exact definitions; shared sizes referenced once>` | `<full style references, not "primary style">` | `<full-width/intrinsic, wrapping, RTL and applicable intervals>` | `<state rows below or exact complete canonical section; source version>` |

### Shared component state styles

| Component / variant / size scope | State and trigger | Fill/text/border/focus/icon treatment | Geometry/content/motion effect | Semantics, keyboard and announcement | Canonical source / specification and evidence status |
| --- | --- | --- | --- | --- | --- |
| `<actual component scope>` | `<default/hover/active/focus-visible/disabled/loading/error/success as applicable>` | `<exact tokens and style changes or explicit unchanged>` | `<precise change or none; layout stability>` | `<observable behavior and labels>` | `<source/version; specified versus actually verified>` |

Cover each applicable state for every specified variant; a complete existing state definition can be referenced by exact version/section rather than copied. Clearly justify non-applicable states. Reuse the actual kit before adding components; this inventory does not require building an unused catalog. Screen-specific business messages and transitions stay in the per-screen interaction matrix below.

### Shared motion — when authorized and applicable

| Motion / affected component or transition | Trigger / purpose | Property, start/end, duration, delay and easing | Interruption/reversal / viewport rules | Reduced-motion alternative | Canonical source / actual approval and evidence |
| --- | --- | --- | --- | --- | --- |
| `<actual requested/approved motion or NOT APPLICABLE with reason>` | `<user-visible purpose>` | `<exact specification>` | `<deterministic behavior>` | `<specific non-motion/limited-motion behavior>` | `<source; SPECIFIED / observed / UNVERIFIED>` |

Do not add animation just to fill this table. No asset, rendered result, contrast or interaction is certified merely by documenting its tokens. Preserve one source of truth and update affected screen references/review evidence when the baseline changes.

## Screen and flow inventory

| Design item ID | Requirement/AC and software item IDs | Route/screen/component and user goal | Entry, action, transition, exit and recovery | Text specification / dependent API/data IDs |
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
| `<DT-n; actual element>` | `<shared typography role/section; exact approved override if any>` | `<shared component/token definitions; exact approved override if any>` | `<long/translated copy rules>` | `<actual required product asset or asset gap>` |

## Interaction, state and accessibility matrix

| Design item / condition | Trigger and expected visible behavior / exact copy source | State-specific geometry and feedback | Keyboard, focus, semantics and announcement | Validation/API dependency / recovery | Checkable acceptance or limitation |
| --- | --- | --- | --- | --- | --- |
| `<DT-n; default/hover/focus/disabled/loading/empty/error/denied/success as applicable>` | `<observable transition>` | `<layout stability, message placement, motion/reduced motion>` | `<labels, focus order/return, errors, touch targets>` | `<approved API/BR IDs; no invented contract>` | `<design criterion; runtime verification pending>` |

## Implementer instructions and review evidence

| Software item / design items | Existing component/token reuse and composition | Exact specification location and required states/intervals | Proposed file responsibility to reconcile with software plan | Design-phase test evidence / Orchestrator correction evidence | Remaining blocker |
| --- | --- | --- | --- | --- | --- |
| `<I-n / DT-n>` | `<approved kit>` | `<sections/artifacts>` | `<Planner owns software paths; no unilateral rename>` | `<actual reference or NOT RUN>` | `<specific gap or NONE>` |

Do not duplicate API/schema definitions or prescribe unsupported implementation syntax. Conflicts with software tasks, guide constraints or business behavior go through Orchestrator to the appropriate owner. Passing this specification gate does not prove the built UI, functional accessibility or product acceptance.
