# Signifiers and Affordances

How users know what they can do. This is the most frequently violated principle in AI-generated UIs — designs that look polished but leave users unsure what's interactive, what's information, and what's decoration.

## Core Concepts (from Don Norman)

### Affordance
The *possible actions* an object provides. A button affords clicking. A slider affords dragging. A text field affords typing.

In digital UI, affordances are mostly conventional rather than physical. A flat rectangle doesn't physically "afford" clicking the way a physical button does — users know it's clickable because they've learned the convention.

### Signifier
A *visual cue* that indicates where and how to act. Signifiers tell users about affordances.

- A button's raised shadow says "click me"
- A hand cursor on hover says "this is interactive"
- Underlined blue text says "this is a link"
- A hamburger icon says "there's a menu here"

**The #1 rule:** If an element is interactive, it must have a clear signifier. If an element is not interactive, it must not look like it is.

## Why AI Agents Fail at This

AI models learn from screenshots where signifiers are present, but they don't understand the *relationship* between visual treatment and interactivity. This produces:

1. **Interactive elements that look static** — flat buttons with no hover state, no cursor change, no visual differentiation from text
2. **Static elements that look interactive** — underlined text that isn't a link, colored text that isn't clickable, headers that look like tabs
3. **Ambiguous elements** — elements where the user genuinely cannot tell if they can interact

## Signifier Toolkit

### For Clickable Elements
- **Buttons:** Distinct background color or border, padding, cursor:pointer, hover state change
- **Links:** Underline, link color, cursor:pointer. Color alone is insufficient.
- **Cards as links:** Entire card has cursor:pointer, hover shows elevation change or border highlight
- **Icon buttons:** A visible label wherever space allows. Icon-only buttons **always** need
  an accessible name (`aria-label` or visually-hidden text) — a tooltip is not an accessible
  name and does not reach touch users. Hit area ≥24×24 CSS px (WCAG floor), 44×44 by design
  → [../accessibility/wcag-essentials.md](../accessibility/wcag-essentials.md)

### For Non-Interactive Elements
- Text: No underline, no link color, default cursor
- Images: No border that suggests a frame/link, no hover effect
- Sections/dividers: No shadow, no hover state, default cursor

### For Draggable Elements
- Drag handle (grip dots icon)
- Cursor:grab on hover, cursor:grabbing while dragging
- Visual feedback on pickup (slight scale, shadow increase)

## Signifier Consistency

Within a single product, the same type of interaction must always have the same signifier:

| If this is interactive... | It must always look like... |
|---|---|
| Primary action | Filled button with brand color |
| Secondary action | Outlined button or text button with clear hover |
| Destructive action | Red-colored button or outlined red button |
| Link | Underlined text with link color — the underline is the signifier; color alone fails color-blind users |
| Expandable section | Chevron icon that rotates |
| Dismissible item | X icon in corner |

**Violating this consistency breaks user trust.** If a red button means "delete" on one page and "important" on another, users will hesitate on both.

## The Hover Problem

Hover states are a critical signifier on desktop, but they don't exist on touch devices. Design for both:

- **Desktop:** Use hover to *confirm* interactivity (subtle color shift, underline appearance, shadow change)
- **Touch:** Interactive elements must be identifiable without hover — through color, position, iconography, and context
- **Never rely on hover to reveal critical information** (tooltips that contain essential info, menus that only appear on hover)

## Clickability Cues (Priority Order)

Ranked by **how many users the cue actually reaches**, which is the only ranking that matters.
A cue that requires a mouse reaches nobody on a phone.

1. **Visual treatment** — button shape, fill, border, link underline, card elevation. The only
   cue that works before any interaction, on every input type. This is where the burden sits.
2. **Text label** — a clear verb tells the user what will happen even if the styling is
   ambiguous. Screen-reader users often hear labels with no styling at all.
3. **Position/context** — in a toolbar, a nav bar, a form's action row. Strong, but it is an
   inference, not a statement.
4. **Iconography** — chevrons for expand, X for close, + for add. Conventional, but an icon
   alone is never sufficient → it needs an accessible name.
5. **Cursor change** (`cursor: pointer`) — a *confirmation*, not a signal. It appears only
   after the user has already guessed correctly and moved the mouse there, and it does not
   exist on touch at all. Include it; never count on it.

> **An earlier revision ranked `cursor: pointer` first as "the most universal signal."** That
> is backwards, and it contradicted this file's own "Hover Problem" section: a cue invisible
> until hover, on a device class that has no hover, is the *least* universal cue available.
> The test for any signifier is: **does it work on a touchscreen, before contact?**

**Corollary — the static test.** Take a screenshot. With no cursor, no hover, and no motion,
can you tell what is interactive? If not, the design depends on cues most users never receive.

## References

- Don Norman, *The Design of Everyday Things* — origin of affordance/signifier concepts
- NN/g: Signifiers in UX (multiple articles)
