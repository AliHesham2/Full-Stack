# Grid Systems

The structural foundation of layout. A grid creates alignment, consistency, and visual rhythm.

## The 12-Column Grid

The de facto standard for web UI. 12 columns divide evenly into 2, 3, 4, 6, and 12 — covering most common layout needs.

```
| 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 |
|         6         |         6         |    Full width (12)
|    4    |    4    |    4    |          Three columns
|       8       |       4       |       Content + sidebar
```

### Grid Implementation

- **Gutter width:** 16–32px between columns (24px is common)
- **Max content width:** 1200–1440px for most applications. Marketing sites may go wider.
- **Margins:** Auto-center the grid with equal margins on both sides
- **On mobile:** Stack to 4-column or single-column grid

## When to Break the Grid

- Hero sections with full-bleed images or backgrounds
- Feature sections that need to feel different from the grid
- Edge-to-edge colored sections

**Rule:** Break the grid intentionally, not accidentally. If elements are misaligned, it should look like a design choice, not a mistake.

## CSS Grid vs. Flexbox

| Need | Use |
|------|-----|
| Two-dimensional layout (rows AND columns) | CSS Grid |
| One-dimensional layout (row OR column) | Flexbox |
| Overlapping elements | CSS Grid |
| Equal-width columns | CSS Grid — `grid-template-columns: repeat(3, 1fr)` |
| Responsive columns that wrap on their own | CSS Grid — `repeat(auto-fit, minmax(16rem, 1fr))` |
| Centering a single item | Flexbox (`justify-content: center; align-items: center`) |
| Responsive card grid | CSS Grid with `auto-fit`/`auto-fill` |

**Modern approach (2025+):** Use CSS Grid as the primary layout tool, Flexbox for component-internal alignment.

## Container Queries

Container queries let components respond to their *container's* size rather than the viewport
— a significant improvement for component-based design systems.

**Support (checked against the `web-features` Baseline dataset, v3.36.0):** container queries
reached Baseline **newly available 2023-02-14** and **widely available 2025-08-14**. An earlier
revision said "widely supported since 2023," which conflated the two dates by two and a half
years. They are safe to use now; that was not true in 2023.

```css
.card-container {
  container-type: inline-size;   /* the PARENT is the container */
}

/* Mobile-first: one rule, one direction. */
.card { flex-direction: column; }

@container (min-width: 400px) {
  .card { flex-direction: row; }
}
```

> **Don't pair `min-width: 400px` with `max-width: 399px`.** CSS lengths are fractional —
> at 399.5px (browser zoom, fractional device pixels, a scrollbar's width) *neither* query
> matches and the component falls back to unstyled defaults. Write one query and let the
> default carry the other case. The same trap applies to media queries.

**Three things that catch people out:**

1. **A container cannot query itself.** `container-type` goes on the *parent*; the rules
   inside apply to its descendants. Putting `container-type` on `.card` and then querying
   for `.card` does nothing.
2. **`container-type: inline-size` applies size containment on the inline axis.** The element
   no longer sizes itself from its children's width. That is the point, but it also means an
   element that relied on shrink-to-fit will change size when you make it a container.
3. **Containment interacts badly with `position: sticky` descendants** and can clip overflow
   in ways `overflow: visible` won't undo. If a sticky child stops sticking after you add
   `container-type`, that's the cause.

**When to use container queries:** for components that may appear in different-width containers
(sidebar, main content, modal) and need to adapt. Prefer them over media queries for
component-level responsiveness → [responsive-strategy.md](responsive-strategy.md).

## References
- CSS Grid specification: MDN
- Container queries: MDN. Baseline newly available 2023-02-14, widely available 2025-08-14 (`web-features` v3.36.0)
