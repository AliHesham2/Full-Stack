# UX/UI Knowledge System

Evidence-based UI/UX rules for AI agents generating interfaces. Every recommendation answers: *what to do, why it works, when it doesn't apply*.

## How to Use This Folder

1. **Starting any design task?** Start at [GUIDE-MAP.md](GUIDE-MAP.md), then read this INDEX and [foundations/cognitive-principles.md](foundations/cognitive-principles.md) and [foundations/visual-hierarchy.md](foundations/visual-hierarchy.md) first — everything else depends on them.
2. **Designing more than one screen?** The sequence is a deliverable of its own —
   [interaction-design/user-flows.md](interaction-design/user-flows.md). Screens that are each
   fine but don't connect is the most common structural failure.
3. **Deciding what goes where before styling anything?** →
   [foundations/information-architecture.md](foundations/information-architecture.md).
4. **Writing any text at all** — labels, errors, empty states, buttons →
   [foundations/content-and-microcopy.md](foundations/content-and-microcopy.md). Write it
   before the layout, never after.
5. **Unsure which pattern fits?** → [when-to-what.md](when-to-what.md) has decision trees.
6. **Before delivering a design**, run the QA checklist below and check [anti-patterns/common-mistakes.md](anti-patterns/common-mistakes.md).

## Map

Start at [GUIDE-MAP.md](GUIDE-MAP.md) for the visual reading flow, task routes and complete file catalog. Return here for guide usage, principles and the pre-delivery QA checklist. The catalog lives in that one map; the actual topic files remain the sources of design guidance.

## The Five Rules That Override Everything

1. **Hierarchy before decoration.** If the visual hierarchy is broken, no styling fixes it. ([foundations/visual-hierarchy.md](foundations/visual-hierarchy.md))
2. **Every interactive element needs a signifier; nothing non-interactive may look interactive.** ([foundations/signifiers-affordances.md](foundations/signifiers-affordances.md))
3. **Design all states** — 6 element states (default, hover, active, focus, disabled, loading) **and** 3 view states (loading, empty, error). AI designs show only the ideal case. ([interaction-design/state-patterns.md](interaction-design/state-patterns.md))
4. **WCAG 2.2 AA is non-negotiable.** ([accessibility/wcag-essentials.md](accessibility/wcag-essentials.md))
5. **Be unconventional in expression, never in function.** Break clichés structurally/visually, keep conventions users rely on. ([anti-patterns/design-anti-averaging.md](anti-patterns/design-anti-averaging.md))
6. **Real copy, never placeholder.** No Lorem ipsum, no "Oops! Something went wrong", no "Learn More". Every error says how to fix it. ([foundations/content-and-microcopy.md](foundations/content-and-microcopy.md))
7. **Design the unhappy paths first.** Back must work and preserve input; nothing dead-ends; nothing loses data on failure. ([interaction-design/user-flows.md](interaction-design/user-flows.md))

## Pre-Delivery QA Checklist

- [ ] One clear primary action per view; hierarchy survives a blur test
- [ ] All 6 element states designed for every control; all 3 view states designed for every data region
- [ ] Contrast: 4.5:1 text / 3:1 UI components and focus indicators
- [ ] Full keyboard path exists; visible focus (`:focus-visible`); skip link present
- [ ] `prefers-reduced-motion` handled; nothing flashes >3×/second
- [ ] Touch targets: **24×24 CSS px is the WCAG 2.2 AA floor (SC 2.5.8)**, 44×44 is the platform-HIG target — design for 44, never ship below 24; no hover-only critical interactions
- [ ] Loading <300ms = no indicator; >1s = skeleton/progress; errors have recovery actions
- [ ] Empty states explain why + offer an action
- [ ] Spacing from 4/8px base unit; one type scale, 4–6 of its steps per view; one font family by default
- [ ] No dark patterns; destructive ≠ primary-styled; confirmations for irreversible actions
- [ ] Works in RTL if the product supports Arabic/Hebrew ([accessibility/rtl-i18n.md](accessibility/rtl-i18n.md))
- [ ] All copy is real: no Lorem ipsum, no generic CTAs, every error states the fix, buttons name the action
- [ ] Every multi-step interaction passes the flow audit — back preserves input, interruption is survivable, success is not a dead end ([interaction-design/user-flows.md](interaction-design/user-flows.md))
- [ ] Nav labels are predictable and use the user's vocabulary; no "Misc"/"Other" bucket ([foundations/information-architecture.md](foundations/information-architecture.md))
- [ ] Charts: no meaning by color alone, grayscale test passes, text alternative present, data reachable as a table ([components/data-display.md](components/data-display.md))

## Freshness

- Audited against live primary sources, re-verified September 2026. Each claim below was
  checked against the issuing body, not against secondary writing:
  - **WCAG 2.2** is the current law-relevant standard (W3C Recommendation).
  - **WCAG 3.0** is a **Working Draft** — latest `WD-wcag-3.0-20260303` (3 March 2026),
    still using placeholders for contrast. Widely estimated ~2030; do not design to it.
  - **APCA** is absent from every WCAG 3 draft from 24 July 2023 onward (present in all three
    2021 drafts). Not a compliance target →
    [accessibility/contrast-readable.md](accessibility/contrast-readable.md).
  - **W3C Design Tokens** — `CG-FINAL-format-20251028`, a **Final Community Group Report**
    (28 Oct 2025). Stable and citable, but a CG report, **not** a W3C Recommendation.
  - **EU Accessibility Act** — applies from **28 June 2025**, with an Art. 32 transitional
    period to 28 June 2030 for services using pre-existing products.
  - The **35% vestibular** figure traces to Agrawal et al. 2009 and means *measured
    dysfunction in US adults 40+*, not diagnosed disorder →
    [accessibility/motion-sensitivity.md](accessibility/motion-sensitivity.md).
- Design tooling moves fast; cognitive principles and accessibility law do not. Re-verify "current standard" claims annually.
