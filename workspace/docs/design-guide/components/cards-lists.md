# Cards and Lists

Cards and lists are the two primary containers for content. Choosing between them (and using them well) determines scannability and information density.

## Card vs. List Decision

| Factor | Use Cards | Use Lists |
|--------|-----------|-----------|
| Content has an image or rich media | Yes | No |
| Items are independent and self-contained | Yes | Items are part of a set |
| Users compare **whole items** at a glance (image + title + gist) | Yes | Users scan for one item |
| Each item has multiple distinct attributes | Yes | Each item has 1–2 attributes |
| Space is limited | No (cards need more space) | Yes |
| Items are sequential/ordered | No | Yes |

## Card Design

### Card Anatomy
```
┌─────────────────────────────┐
│  [Media/Image (optional)]    │
├─────────────────────────────┤
│  [Category/Tag]             │
│  Title (1–2 lines, truncated) │
│  Description (2–3 lines)     │
│                              │
│  [Meta: date, author, etc.]  │
│  [Action button/link]        │
└─────────────────────────────┘
```

### Card Rules
- **Padding:** 16–24px internal
- **Border radius:** 8–16px (consistency matters more than the exact value)
- **Shadow/elevation:** Use elevation to show interactivity. A clickable card should have a visible shadow that increases on hover.
- **Clickable area:** If the card is clickable, the entire card should be the click target. Don't make the user hunt for a small link inside the card.
  **But do not wrap the whole card in an `<a>`.** That approach has three real costs: the
  screen reader announces the entire card contents as one enormous link name, the user can no
  longer select text inside the card, and any button inside it becomes a nested interactive
  element (invalid, and unreachable in some AT).
  **Do this instead:** keep one real link on the title, and stretch it over the card with a
  pseudo-element —

  ```css
  .card { position: relative; }
  .card-title a::after { content: ""; position: absolute; inset: 0; }  /* card-wide hit area */
  .card :is(button, a):not(.card-title a) { position: relative; }      /* keep these clickable */
  ```

  The accessible name stays the title, text remains selectable, and secondary buttons still
  work. If a card genuinely needs several independent actions, it is not one link — give it a
  visible primary link and place the actions beside it.
- **Title:** 1–2 lines max. Use `line-clamp` for truncation.
- **Image:** 16:9 or 4:3 aspect ratio. Cover object-fit. Consistent across all cards in a grid.

### Card Grid
- **Columns:** 2–4 on desktop, 1–2 on mobile
- **Gap:** 16–24px
- **Alignment:** Cards in a grid should be the same height (use flexbox stretch or grid)
- **Don't mix card sizes** in the same grid unless there's a clear reason (featured item)

## List Design

### When to Use a List
- Navigation items
- Settings/preferences
- Search results (when items are homogeneous)
- Messages, notifications
- Any ordered/sequential content

### List Item Anatomy
```
┌──────────────────────────────────┐
│ [Icon/Avatar] Title    [Meta]   │
│               Description      │
│               [Secondary info]  │
└──────────────────────────────────┘
```

### List Rules
- **Vertical padding per item:** 12–16px
- **Divider:** Use subtle dividers (1px, light gray) between items. Don't use full-width dividers if items have left-aligned icons — inset the divider to start after the icon.
- **Hover:** Light background highlight for clickable items
- **Truncation:** Truncate descriptions to 1–2 lines. If the full text matters, it must be
  reachable **on focus as well as hover**, or on tap — hover-only reveal is unusable on touch
  and invisible to keyboard users. Better: don't truncate what the user needs
  → [../interaction-design/progressive-disclosure.md](../interaction-design/progressive-disclosure.md).
- **Actions:** Place secondary actions (edit, delete) in a "more" menu, or reveal them on
  hover **and focus**. Hover-only actions do not exist on touch devices and are invisible to
  keyboard users — if an action is only discoverable by hovering, a phone user cannot perform
  it at all. Don't show every action for every item simultaneously.

## Table vs. List vs. Card

| Need | Use |
|------|-----|
| Compare specific **attributes** across many items | Table (see `data-display.md`) — cards are for comparing items *as wholes*; the moment the user compares values field-by-field, cards force them to hold numbers in memory and a table wins |
| Browse heterogeneous items with images | Cards |
| Scan a homogeneous set for one item | List |
| Show a ranked/ordered set | List with numbers |