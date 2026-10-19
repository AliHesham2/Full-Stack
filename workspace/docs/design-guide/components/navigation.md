# Navigation

Navigation is how users find their way. It's the skeleton of the information architecture made visible.

## Navigation Pattern Decision Guide

The item count alone does not decide this — the ranges below overlap on purpose, because at
5–7 items *both* a top nav and a sidebar work. **Depth and switching frequency break the tie:**

- **Flat structure, users switch sections occasionally** → top nav
- **Nested structure, or users switch constantly while working** → sidebar (it stays visible,
  and it has room for a second level)

| Sections | Pattern | When to Use |
|---|---|---|
| 2–4 | **Tab bar** | Simple apps, mobile, sub-navigation within a page |
| 3–7, flat | **Horizontal top nav** | Most web apps of moderate scope |
| 5–15, or nested | **Sidebar nav** | Complex apps with deep hierarchies |
| 15+ | **Sidebar with collapsible groups** | Enterprise tools with many modules — but first check whether the IA is too deep → [../foundations/information-architecture.md](../foundations/information-architecture.md) |
| Mobile | **Bottom tab bar** (3–5 items) + a menu for the rest | Mobile apps, responsive web |

## Top Navigation

- **Logo:** At the **start** of the bar (left in LTR, **right in RTL**), linking to
  home/dashboard. Use logical properties so this follows direction automatically
  → [../accessibility/rtl-i18n.md](../accessibility/rtl-i18n.md)
- **Nav items:** Left-aligned after logo (or centered for marketing sites)
- **Actions:** Right side (search, notifications, profile)
- **Active state:** Underline, background highlight, or bold — must be clearly visible, and
  must not be conveyed by colour alone. Mark it programmatically too: `aria-current="page"`
  on the active link. Without it, a screen-reader user has no way to know where they are.
- **Hover state:** Subtle underline or background change
- **Dropdowns: open on click, not on hover.** Hover-opened menus are unreachable for keyboard
  users, non-existent on touch, and fire accidentally when the pointer crosses them on the way
  somewhere else. They also have to satisfy WCAG 1.4.13 (dismissible / hoverable / persistent),
  which a naive `mouseleave` handler fails. This is the same rule as
  [../anti-patterns/common-mistakes.md](../anti-patterns/common-mistakes.md) → "Hover-Only
  Interactions"; an earlier revision of this file contradicted it.
  If you add hover-open as a desktop enhancement, keep click working, add a 100–200ms open
  delay and a close delay, and make Escape close it. A `popover` gives you most of this for
  free → [modals-dialogs.md](modals-dialogs.md).

## Sidebar Navigation

- **Width:** 220–280px (collapsible to 56–72px icon-only)
- **Active state:** Background highlight, bold text, or left border indicator
- **Grouping:** Group related items with section headers (not clickable)
- **Collapse:** Support collapsible sidebar for more content area. Show tooltips on icon-only items.
- **Scroll:** If nav items exceed viewport, allow scroll within the nav. Indicate scrollability.

## Breadcrumbs

Show the user's location within the hierarchy. Required for any site with 3+ levels.

- **Format:** Home > Category > Subcategory > Current Page
- **Current page:** Not a link, visually distinct (bold or different color)
- **When to show:** Always on interior pages, even if the user arrived via search or direct link. Breadcrumbs serve as escape routes.
- **Separator:** Use `>` or `/`. Not arrows (→), which imply direction of action.
  **Generate separators in CSS** (`::before { content: "/" }`), not as text in the markup —
  a screen reader reads literal text separators aloud between every item ("Home greater-than
  Products greater-than…").
- **Markup:** wrap in `<nav aria-label="Breadcrumb">` with an ordered list, and mark the last
  item `aria-current="page"`. The `aria-label` matters because a page usually has more than
  one `<nav>`.

## Mobile Navigation

- **Bottom tab bar:** For 3–5 primary destinations. Each tab = a top-level section.
- **Hamburger menu:** For secondary items. The icon (☰) is widely recognised but hides
  content — use it only for items that aren't primary tasks. It is a **button**, not a link,
  and needs an accessible name ("Menu") plus `aria-expanded` reflecting open/closed state.
  An unlabelled icon-only hamburger announces as "button" and nothing else.
- **Don't put primary CTAs in a hamburger menu.** If the most important action is hidden, users can't find it.

## Wayfinding

Users should always be able to answer:
1. Where am I? → Active nav state, breadcrumbs, page title
2. Where can I go? → Navigation links, CTAs
3. How do I get back? → Breadcrumbs, back button, clear nav hierarchy

If any of these are unclear, the navigation is failing.

## References
- IxDF Navigation (updated 2026): https://ixdf.org/literature/topics/navigation
- NN/g: Navigation articles
