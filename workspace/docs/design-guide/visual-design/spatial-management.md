# Spatial Management: Z-Index, Overflow, and Layers

Managing what's on top, what scrolls, and how layers relate. This is a common source of bugs in production.

## Z-Index Management

### The Problem
Z-index values like 9999, 99999, and 100000 are a code smell. They indicate that the stacking context isn't being managed intentionally.

### Z-Index Scale

Assign z-index values from a predefined scale. Every layer has a named level:

| Level | Z-Index | Content |
|-------|---------|---------|
| base | 0 | Normal page content |
| dropdown | 100 | Dropdown menus, popovers |
| sticky | 200 | Sticky headers, toolbars |
| overlay | 300 | Modal overlays, drawer backdrops |
| modal | 400 | Modal dialogs |
| toast | 500 | Toast notifications |
| tooltip | 600 | Tooltips (must be above everything) |

### Stacking Context Rules

1. **Every element with z-index creates a stacking context** if it also has position != static, opacity < 1, transform, filter, or certain other properties.
2. **Children cannot escape their parent's stacking context.** A z-index: 9999 inside a z-index: 0 parent is still below a z-index: 1 element outside that parent.
3. **Use CSS custom properties for z-index** so the scale is visible and maintainable:

```css
:root {
  --z-dropdown: 100;
  --z-sticky: 200;
  --z-overlay: 300;
  --z-modal: 400;
  --z-toast: 500;
}
```

## The Top Layer Beats Every Z-Index

The scale above governs **normal-flow content**. It does not govern modern overlays, and
assuming it does is the fastest way to a stacking bug you cannot fix by raising a number.

`dialog.showModal()` and any element with the `popover` attribute render in the browser's
**top layer** — a layer above the entire document, outside every stacking context. Consequences:

- **z-index does nothing to them.** A top-layer dialog sits above `z-index: 999999`.
- **A parent's stacking context cannot trap them.** This is the point: it fixes the
  `overflow: hidden` / `transform` parent problem that forces people into portals.
- **Order within the top layer is order of opening**, not z-index. The most recently shown
  element is on top.
- **`::backdrop`** styles the layer behind each one.

**The rule this changes:** if a dropdown, tooltip, or menu is escaping its container or being
clipped by a parent, the fix is `popover`, not a bigger z-index
→ [../components/modals-dialogs.md](../components/modals-dialogs.md).

### The gap in every z-index scale

The scale above assigns `dropdown: 100` and `modal: 400`. So what is the z-index of a
**dropdown inside a modal**? By the table it is 100 — beneath the modal that contains it.

Every flat z-index scale has this hole, because "dropdown" is not a global altitude; it is
*relative to whatever it opens within*. Two ways out:

1. **Preferred:** make the dropdown a `popover`. It goes to the top layer and the question
   disappears.
2. **Otherwise:** treat the scale as *per stacking context*, not global — inside the modal's
   context, the dropdown only needs to beat the modal's own children, so a small local value
   works. This is why rule 2 below matters more than the table.

## Overflow and Scrolling

### When to Scroll
- **Content area:** Scroll within the main content area, not the page. Keep headers/footers fixed.
- **Tables:** Horizontal scroll within the table container.
- **Long lists:** Virtual scrolling for 100+ items (only render visible items). **It has real
  costs:** browser find-in-page (Ctrl+F) can't see unrendered rows, Ctrl+End and screen-reader
  "read all" break, and `aria-setsize`/`aria-posinset` become mandatory so AT can report
  "item 12 of 900". Paginate or filter before virtualising — virtualisation is a rendering
  optimisation, not an information-architecture answer.
- **Sidebars:** If nav items exceed viewport height, scroll within the sidebar.

### When NOT to Scroll
- **Modals:** If modal content is long, scroll within the modal body, not the page.
- **Horizontal page scroll:** Almost never acceptable. Content should reflow, not require horizontal scroll.

### Scroll Behavior
- `scroll-behavior: smooth` for in-page anchor links (but respect `prefers-reduced-motion`)
- Provide scroll shadows or fade indicators when content is scrollable but the scrollbar isn't visible (macOS)
- Consider `overflow-y: auto` (show scrollbar only when needed) over `overflow-y: scroll`

## Fixed Positioning Pitfalls

- **`position: fixed`** is relative to the viewport, NOT the containing element — *unless* an
  ancestor establishes a containing block. That happens with `transform`, `filter`,
  `backdrop-filter`, `perspective`, `contain: paint` (or `layout`/`strict`), and
  `will-change` naming any of those. A fixed header that mysteriously scrolls away almost
  always has an ancestor with one of these. **`backdrop-filter` is the usual culprit** in
  frosted-glass designs, where it is applied without anyone expecting it to change positioning
  → [../anti-patterns/ai-design-cliches.md](../anti-patterns/ai-design-cliches.md).
- **`position: sticky`** is often a better choice for headers/footers within a scrollable area.
- **On mobile, `position: fixed`** can cause issues with the virtual keyboard pushing content up unpredictably.

## References
- MDN: Stacking context, z-index
- Josh W Comeau: "A Complete Guide to z-index"