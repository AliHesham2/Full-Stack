# Common UI/UX Mistakes

Frequently occurring errors that aren't deceptive (those are dark patterns) but degrade the experience.

## Visual Mistakes

### Low-Contrast Text
Using light gray text on white backgrounds for secondary information. This fails WCAG 2.2 AA and is the #1 accessibility issue on the web.

**Fix:** Secondary text should be gray-600 to gray-700 on white. Check every color with a contrast checker.

→ See `../accessibility/contrast-readable.md`

### Inconsistent Spacing
Arbitrary spacing values (13px, 17px, 22px) instead of a base-unit system.

**Fix:** Use a 4px or 8px base unit. → See `../visual-design/spacing-composition.md`

### Too Many Fonts or Sizes
Using 3+ font families, or reaching for nearly every step of the type scale on a single screen.

**Fix:** One font family by default. Define the full scale once (8 steps is normal), then use
**4–6 of them per view**. The failure is not how many sizes the system *defines* — it is how
many appear together, because adjacent steps stop reading as distinct levels.
→ See `../visual-design/typography.md`

### Underline on Non-Links
Underlining text that isn't a link.

**Fix:** In running text, treat a solid underline as reserved for links, and use italic, bold,
or colour for emphasis. The genuine exceptions are semantic and look different on purpose:
`<abbr>` (dotted underline), `<ins>` (inserted text), and spelling/grammar indicators. If
you underline for emphasis, users will click it.

Conversely — **do not remove underlines from links in body copy.** Colour alone fails
colour-blind users, and `text-decoration: none` on inline links is the more common and more
damaging half of this mistake.

## Interaction Mistakes

### No Loading States
Content appears abruptly or the screen goes blank while loading.

**Fix:** Use skeleton screens or spinners for loads > 300ms. → See `../interaction-design/state-patterns.md`

### No Error States
Forms submit without validation, or errors appear as browser alerts.

**Fix:** Inline validation with clear messages. → See `../interaction-design/forms-input.md`

### Dead-End Empty States
"No data" with no explanation or action.

**Fix:** Explain why it's empty and provide an action. → See `../interaction-design/state-patterns.md`

### Auto-Playing Video/Audio
Video or audio starts playing without user initiation.

**Fix:** Never auto-play media with sound. Auto-playing muted video should have visible controls and pause on `prefers-reduced-motion`.

## Layout Mistakes

### Content Too Wide
Body text spanning the full width on large screens.

**Fix:** `max-width: 65–70ch` on text containers. → See `../visual-design/typography.md`

### No Clear Visual Hierarchy
All headings are the same size, all sections have equal visual weight.

**Fix:** Establish 3-level hierarchy. → See `../foundations/visual-hierarchy.md`

### Modals for Everything
Using modals for multi-step forms, lengthy content, or non-essential information.

**Fix:** Modals are for focused sub-tasks. Use pages, drawers, or inline expansion for more. → See `../components/modals-dialogs.md`

## Mobile-Specific Mistakes

### Tiny Touch Targets
Buttons and links below the WCAG floor, or merely uncomfortable.

**Fix:** **24×24 CSS px is the AA requirement** (SC 2.5.8, with spacing/inline exceptions);
44×44pt (iOS) / 48×48dp (Android) is what actually feels right under a thumb. Design for 44,
never ship below 24. The visual control may be smaller than its hit area — extend with
padding or a pseudo-element.
→ See `../accessibility/wcag-essentials.md`, `../foundations/cognitive-principles.md` (Fitts's Law)

### Hover-Only Interactions
Dropdowns, tooltips, or row actions that only appear on hover.

**Fix:** Every hover reveal must also trigger on **focus**, and be reachable by **tap**. Touch
is the obvious gap, but a keyboard user on a desktop has no hover either — "tap to toggle on
mobile" fixes only half the problem. Tooltips additionally have to be dismissible, hoverable,
and persistent (WCAG 1.4.13)
→ See `../interaction-design/progressive-disclosure.md`

### Horizontal Scroll Without Indication
Content scrolls horizontally but there's no visual cue that more content exists.

**Fix:** Show a fade/shadow at the edge, visible scrollbar, or scroll indicators (dots).

## References
- NN/g: extensive articles on each of these topics
- WebAIM annual accessibility survey (consistent findings year over year)
