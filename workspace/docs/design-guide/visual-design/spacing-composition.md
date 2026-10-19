# Spacing and Composition

Whitespace (negative space) is the most underused and most powerful design tool. AI-generated designs consistently fail here: they fill every pixel, creating dense, exhausting interfaces.

## The Function of Whitespace

Whitespace is not empty — it's active. It performs these functions:

| Function | How It Works |
|----------|-------------|
| **Grouping** | Elements closer together are perceived as related (Gestalt proximity) |
| **Separation** | Space between groups creates visual sections |
| **Emphasis** | Isolated elements feel more important |
| **Breathing room** | Reduces cognitive load, prevents overwhelm |
| **Visual rhythm** | Consistent spacing creates a predictable, comfortable structure |

## Spacing System

### Use a Base Unit

Pick a base unit (4px or 8px are standard) and derive all spacing from it:

```
4px base:
  xs:  4px   (tight padding within components)
  sm:  8px   (related items within a group)
  md:  16px  (between items in a list, between label and input)
  lg:  24px  (between sections)
  xl:  32px  (between major sections)
  2xl: 48px  (page-level breathing room)
  3xl: 64px  (major section separation)

8px base:
  xs:  8px
  sm:  16px
  md:  24px
  lg:  32px
  xl:  48px
  2xl:  64px
  3xl:  96px
```

**Why a base unit?** Mathematical consistency creates visual harmony without conscious awareness. When spacing is arbitrary (13px, 17px, 22px), the design feels "off" even if users can't articulate why.

### The 2x Rule for Section Spacing

The space *between* sections should be roughly 2x the space *within* sections. This is Gestalt
proximity applied as arithmetic: what is closer together reads as belonging together.

- Internal card padding: 16–24px
- Space between cards: 16–24px
- Space between card groups (sections): 32–48px
- Space between major page sections: 48–64px

> ⚠️ **Note the second and third lines and pick deliberately.** Internal padding and the gap
> between cards are listed at the same range — so if you take 24px for both, the space *inside*
> a card equals the space *between* cards, and proximity stops signalling anything. A card
> survives this because its border or background draws the boundary instead.
>
> **The rule to carry away:** whenever grouping is expressed by *space alone*, the outer gap
> must be visibly larger than the inner gap — not equal, and never smaller. If you cannot make
> the outer gap bigger, draw the boundary another way (a background, a border, a divider) and
> stop relying on proximity.
>
> The classic failure is a form where the gap between a label and its input equals the gap
> between fields: every label then appears to belong to the field above it.

### Padding Within Components

- Buttons: 8–12px vertical, 16–24px horizontal
- Cards: 16–24px
- Form fields: 10–14px vertical, 12–16px horizontal
- Modal padding: 24–32px
- Table cells: 12–16px vertical, 16px horizontal

## Composition Principles

### The Rule of Thirds — use with caution

Place important elements at the intersection points of a 3×3 grid.

This is a **photography and fine-art** heuristic, and it transfers poorly to UI. Interfaces are
scanned in reading order, resized continuously, and judged by task completion — not composed
within a fixed frame. It can be useful for a hero section, a marketing splash, or an
illustration, where the layout genuinely is a single fixed composition.

For application UI, **alignment, proximity, and the type scale do the work the rule of thirds
does in a photograph.** If a rule-of-thirds placement conflicts with reading order or hierarchy,
reading order wins → [../foundations/visual-hierarchy.md](../foundations/visual-hierarchy.md).

### Visual Balance

- **Symmetrical balance:** Formal, stable. Good for headers, footers, centered content.
- **Asymmetrical balance:** Dynamic, engaging. Good for content pages, marketing layouts.
- **Rule:** If the layout feels "top-heavy" or "left-heavy," add visual weight to the opposite side through color, size, or density.

### Alignment

Every element should align with something else. Misaligned elements create visual noise.

- **Align body text to the start of the reading direction.** Write `text-align: start`, not
  `text-align: left` — the logical keyword is correct in Arabic and Hebrew without a second
  rule → [../accessibility/rtl-i18n.md](../accessibility/rtl-i18n.md). Centered body text is
  harder to read because the start point shifts on every line; reserve centring for single
  lines and short headings.
- **Align labels with their inputs** — left-align labels above inputs, or right-align labels to the left of inputs in forms with short labels
- **Use a consistent grid** for multi-column layouts

## Density Trade-offs

Density is not inherently good or bad. It's a choice based on context.

### High Density Is Appropriate When:
- Users are experts who use the tool daily
- The primary goal is information comparison or scanning
- Screen real estate is limited
- Users have expressed a preference for density (e.g., "compact view" option)

### Low Density Is Appropriate When:
- Users are new or infrequent
- The primary goal is reading or comprehension
- The content is emotionally heavy or requires careful consideration
- The context is mobile or casual browsing

**The AI mistake:** Defaulting to medium-high density because it looks more "professional" or "complete." Default to comfortable density and add density controls when appropriate.

## References
- Gestalt proximity principle → `../foundations/cognitive-principles.md`
- 8-point grid system: widely adopted across design systems (Material, Ant Design, etc.)