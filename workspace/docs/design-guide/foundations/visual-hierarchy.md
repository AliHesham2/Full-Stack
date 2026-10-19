# Visual Hierarchy

The single most important concept in UI design. If the hierarchy is wrong, nothing else matters — color, animation, and typography cannot fix a broken hierarchy.

## Core Mechanism

Visual hierarchy is the arrangement of elements in a way that implies importance. It answers, for every element on screen: "What should I look at first, second, third, and not at all right now?"

## Hierarchy Tools (Ranked by Reliability)

### 1. Size — Strongest Signal

The largest element on a page is seen first. This is the most reliable hierarchy signal across all cultures and contexts.

- Page titles: 24–48px (or larger for hero contexts)
- Section headers: 18–32px
- Body text: **16px default** — 14px is a metadata size, not a body size
- Captions/metadata: 11–13px

These are ranges for orientation. **Do not pick values from them freely** — the ranges overlap,
and choosing 20px here and 21px there is what produces a scale with no rhythm. Derive the
actual numbers from one modular scale → [../visual-design/typography.md](../visual-design/typography.md).
The rule that matters is the *ratio between adjacent levels*, not the absolute size: two levels
one or two pixels apart read as a mistake, not a hierarchy.

**AI failure:** Using similar sizes for too many elements, creating a flat visual field where nothing clearly leads.

### 2. Contrast

Elements that contrast with their surroundings attract attention. Contrast can come from color, value (light/dark), or weight.

- High-contrast elements (bold text on light background, bright color on muted background) demand attention
- Low-contrast elements (light gray on white, muted colors) recede

**Critical rule:** Never use contrast for decoration without considering its hierarchy impact. Every high-contrast element is implicitly saying "look at me."

### 3. Spacing/Position

- Elements with more space around them feel more important (isolation = importance)
- Top-left is the natural start point for LTR readers; bottom-right is the end
- Elements closer to the top of the viewport receive attention before elements below the fold

### 4. Weight (Typographic)

Bold vs. regular, heavy vs. light. Weight contrast creates hierarchy even at the same size.

- Use weight sparingly for emphasis — bold everything is the same as bold nothing
- A common effective pattern: bold headings (600–700), regular body (400), light metadata (300)

### 5. Hue

**Read this together with #2.** "Contrast" and "color" are not two separate tools — the reason
they rank so far apart is that they are different *properties of color*, and only one of them
is reliable:

| Property | Strength | Why |
|---|---|---|
| **Lightness / value contrast** (#2) | Strong | Survives color blindness, greyscale, glare, and cheap screens. This is what makes contrast rank second. |
| **Hue alone** (#5) | Weak | Two colors of the same lightness but different hue may be indistinguishable to some users and vanish entirely in greyscale. |

So: *a dark element on a light background* is a strong hierarchy signal. *A green element next
to a red one of the same lightness* is nearly no signal at all.

Hue is the weakest and most dangerous tool because:
- Color vision deficiency affects roughly 8% of men and 0.5% of women (populations of Northern
  European descent, where the standard figure comes from; it varies by population but never to
  zero)
- Cultural color associations vary
- Screen conditions (brightness, ambient light) affect color perception

**Use hue as a supporting signal, never the only signal.** A red button must still read as the
destructive action without its color — through label, position, and treatment.

**The greyscale test:** desaturate the screen. If the hierarchy survives, it was built on
lightness. If it collapses, it was built on hue.

## The Three-Level Hierarchy Model

Every screen should have a clear three-level hierarchy:

| Level | Purpose | Typical Treatment |
|-------|---------|------------------|
| **Primary** | The one thing the user should do or understand | Largest, highest contrast, boldest weight |
| **Secondary** | Supporting content or alternative actions | Medium size, moderate contrast |
| **Tertiary** | Context, metadata, supplementary info | Smallest, lower contrast, lighter weight |

> ⚠️ **Do not stack all three reductions at once.** "Smallest + lowest contrast + lightest
> weight" compounds into text that is 11px, weight 300, and grey-on-white — which fails
> **WCAG 1.4.3 (4.5:1)** and is unreadable for a large share of users. Demotion is *relative*,
> and it has a floor:
>
> - **Contrast is not a hierarchy tool below 4.5:1.** Tertiary text still needs 4.5:1 (3:1 only
>   if it is genuinely ≥24px, or ≥18.66px bold). Demote with **size and weight**, and keep
>   contrast legal → [../accessibility/contrast-readable.md](../accessibility/contrast-readable.md).
> - **Pick two of the three.** Smaller *and* lighter weight at full contrast reads as
>   secondary without becoming invisible.
> - Weight 300 below ~16px is a legibility problem in Latin script and a worse one in Arabic
>   → [../accessibility/rtl-i18n.md](../accessibility/rtl-i18n.md).
>
> The hierarchy must be built so that **the least important thing is still readable**. If
> tertiary content isn't worth making readable, it isn't worth putting on the screen.

**Test:** Blur your eyes (or apply a Gaussian blur to a screenshot). Can you still identify the primary action? If not, the hierarchy is broken.

## Common Hierarchy Failures (Especially in AI-Generated Designs)

### The "Everything is Important" Problem
Every section has a bold heading, a colored accent, and similar spacing. Nothing stands out because nothing recedes.

**Fix:** Ruthlessly demote secondary and tertiary content. If a heading is just a label, make it small and light. Save bold and large for what truly matters.

### The "Decoration as Hierarchy" Problem
Using color, gradients, borders, or icons to create visual interest without considering what they communicate about importance.

**Fix:** Every visual decision should answer: "Does this make it clearer what to look at first?" If not, remove it.

### The "Missing Exit" Problem
Designing a page where the primary action is clear but there's no obvious way to go back, cancel, or do something else.

**Fix:** Always provide a clear secondary path. Not every page needs a CTA — some pages need a "Back" link more than a "Submit" button.

## Hierarchy in Practice

### For Content Pages
1. Page title (H1) — what is this page about?
2. Primary content — the main value
3. Secondary content — supporting details
4. Tertiary content — related links, metadata
5. Navigation — how to go elsewhere

### For Dashboards
1. Most important metric or insight
2. Primary actions (filters, date ranges)
3. Supporting charts/tables
4. Secondary metrics and details

### For Forms
1. Form title and brief instruction
2. Current field/question
3. Progress indicator (for multi-step)
4. Submit/cancel actions

## Reading Patterns

### F-Pattern (for text-heavy content)
Users scan in an F-shaped pattern: horizontal across the top, then shorter horizontal scans, then vertical down the left.

**Implication:** Place important content at the top and along the left edge. Use headings and bold text at the start of paragraphs.

### Z-Pattern (for less text, more visual layouts)
Users scan in a Z: top-left to top-right, diagonally to bottom-left, then bottom-left to bottom-right.

**Implication:** Place the primary CTA at the end of the Z (bottom-right). Place the value proposition at the start (top-left).

These are tendencies, not rules. The actual scan pattern depends on the content, the user's goal, and the layout. But they provide a useful starting framework.

### Both patterns mirror in RTL

The F and Z patterns are consequences of **reading direction**, not of human vision. In Arabic
or Hebrew the scan starts **top-right**, the F's stem runs down the **right** edge, and the Z
reverses — so the CTA at "the end of the Z" belongs **bottom-left**.

This is not a detail to handle later. "Top-left is the start, bottom-right is the end" is stated
as universal in *Spacing/Position* above; it is true only for LTR. Applying LTR scan-pattern
placement to an Arabic interface puts the primary action where the eye finishes *last*.

Use logical properties (`margin-inline-start`, `text-align: start`) so placement follows
direction automatically → [../accessibility/rtl-i18n.md](../accessibility/rtl-i18n.md).

## References

- IxDF Visual Hierarchy (updated 2026): https://ixdf.org/literature/topics/visual-hierarchy
- NN/g on visual hierarchy: multiple articles at nngroup.com
- "Refining the F-pattern" (NN/g) — updates on original F-pattern research
