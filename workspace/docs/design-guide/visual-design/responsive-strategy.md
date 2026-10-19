# Responsive Strategy

Responsive design is not "making things smaller on mobile." It's designing distinct layouts optimized for different contexts while maintaining design consistency.

## Breakpoint Strategy

### Standard Breakpoints (Based on Common Device Widths)

| Name | Width | Target Devices |
|------|-------|---------------|
| Mobile (small) | 0–374px | Small phones |
| Mobile | 375–767px | Most phones (design for 375px first) |
| Tablet | 768–1023px | iPads, small tablets |
| Desktop | 1024–1279px | Laptops, small desktops |
| Desktop (large) | 1280–1439px | Standard desktops |
| Desktop (wide) | 1440px+ | Large monitors |

### Content Breakpoints vs. Device Breakpoints

**Prefer content breakpoints.** Set breakpoints where the layout *breaks*, not at arbitrary device widths.

```css
/* Bad: arbitrary device breakpoint */
@media (max-width: 768px) { ... }

/* Good: content-driven breakpoint */
@media (max-width: 600px) { /* Layout naturally breaks at 600px */ }
```

## Mobile-First vs. Desktop-First

**Mobile-first is the recommended default.** Write base styles for mobile, add complexity for larger screens.

```css
/* Mobile-first: base = mobile, add for larger */
.grid { display: grid; grid-template-columns: 1fr; }
@media (min-width: 768px) { .grid { grid-template-columns: 1fr 1fr; } }

/* Desktop-first: base = desktop, override for smaller */
.grid { display: grid; grid-template-columns: 1fr 1fr; }
@media (max-width: 767px) { .grid { grid-template-columns: 1fr; } }
```

Mobile-first is preferred because:
- Base styles are simpler and faster to render
- It forces you to prioritize content (what's essential on mobile?)
- `min-width` media queries are easier to reason about (additive complexity)

## Key Responsive Patterns

### Navigation Collapse
- Desktop: Horizontal top nav or sidebar
- Tablet: May collapse to icon-only sidebar
- Mobile: Bottom tab bar + hamburger menu for overflow

### Grid Adaptation
- Desktop: 3–4 columns
- Tablet: 2 columns
- Mobile: 1 column

### Table Response
- Desktop: Full table
- Tablet: Horizontal scroll or hide columns
- Mobile: Card layout (each row becomes a card)

### Modal/Drawer
- Desktop: Centered modal (max-width ~480px)
- Mobile: Full-width bottom sheet or full-screen

### Typography Scaling
- Use `clamp()` for fluid type scaling:
```css
h1 { font-size: clamp(1.75rem, 1rem + 2vw, 2.5rem); }
```
- This avoids jarring size changes between breakpoints

## Viewport Breakpoints Are the Fallback, Not the Default

Everything above describes **viewport** breakpoints. For anything that is a *component* rather
than a *page*, that is the wrong axis: a card does not care how wide the window is, it cares
how much room it was given. A card in a sidebar and the same card in a main column need
different layouts at the *same* viewport width, and a media query cannot express that.

**Decision rule:**

| Adapting… | Use |
|---|---|
| Page-level layout, navigation shell, global density | Viewport media queries |
| A reusable component appearing in more than one slot | **Container queries** |
| A single value that scales smoothly (type, gap, padding) | `clamp()` — no breakpoint at all |

Container queries are covered in [grid-systems.md](grid-systems.md). Reach for them by default
for components: a component styled with viewport breakpoints is only correct in the one slot
it was designed for.

`clamp()` deserves the same emphasis. Many "breakpoints" exist only to step a number, and a
single `clamp()` removes both the breakpoint and the visible jump at its threshold — see
fluid type in [typography.md](typography.md).

## Touch Considerations for Mobile
- Minimum touch target: 44×44pt (iOS), 48×48dp (Android)
- Space between tappable elements: at least 8px
- No hover-dependent interactions (tooltips, hover menus)
- Avoid horizontal scrolling (except in carousels with visible scroll indicators)
- Make sure form labels are visible (not inside inputs as placeholders)

## References
- Brad Frost, *Atomic Design* — component-based responsive design
- Every Layout (every-layout.dev) — modern CSS layout patterns
