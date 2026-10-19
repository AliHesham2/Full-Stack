# Data Display

Tables, dashboards, and data visualizations. Where functional density meets clarity.

## Tables

### When to Use a Table
- Comparing specific attributes across 5+ items
- Sorting, filtering, and selecting items
- Displaying structured, homogeneous data

### Table Design Rules

**Headers:**
- Sticky on scroll (always visible)
- Left-align text columns, right-align numeric columns
- Provide sort indicators (arrows) for sortable columns

**Rows:**
- Alternating row colors (zebra striping) are optional — they help in wide tables but add visual noise in compact ones
- Hover highlight for interactive rows
- 12–16px vertical cell padding
- Truncate long text with tooltip on hover

**Responsiveness:**
- On narrow screens, consider: horizontal scroll, card layout, or hide less-important columns
- Never let a table crush its content — horizontal scroll is preferable to unreadable cells
- Pin the first column (key identifier) on horizontal scroll

**Actions column:**
- Place on the right
- Use icon buttons with tooltips for space efficiency
- For 3+ actions, use a "more" dropdown menu

## Dashboards

### Dashboard Design Principles
1. **Lead with the most important metric or insight** — don't make users hunt for it
2. **Provide context** — comparison to previous period, target, or benchmark
3. **Support drilling down** — clicking a metric should reveal detail
4. **Limit visible metrics** — around 5–9 on the main view. The reason is *comparison and prioritisation*, not a memory limit: past that, nothing reads as important because everything does. Progressive disclosure for the rest → [../foundations/cognitive-principles.md](../foundations/cognitive-principles.md).
5. **Consistent layout** — metrics in the same position every time

### Metric Display
- **Big number:** Current value, large font
- **Trend indicator:** Arrow up/down + percentage change + time period
- **Comparison:** vs. previous period, vs. target, vs. benchmark
- **Sparkline:** Small line chart showing trend (optional but powerful)

## Data Visualization Selection

| Data Question | Chart Type | When to Use |
|---|---|---|
| Comparison across categories | Bar chart (horizontal for many categories, vertical for few) | Comparing discrete items |
| Change over time | Line chart | Continuous time-series data |
| Part of whole | Donut chart (not pie) | When showing 2–5 segments |
| Correlation | Scatter plot | Two continuous variables |
| Distribution | Histogram, box plot | Understanding spread of data |
| Single metric vs. target | Bullet chart, gauge | KPIs with targets |

### Chart Design Rules
- **Data-ink ratio:** Maximize the ink used to show data, minimize decoration. Remove chart borders, gridlines, and backgrounds that don't aid reading.
- **Direct labeling:** Label data points directly when possible, rather than relying on a legend.
- **Color:** Use a single color with varying opacity for series, or a sequential color scale. Reserve multiple colors for distinct categories.
- **Axis labels:** Always include. Always include units.
- **Interactive:** On hover, show a tooltip with exact values. Allow click-to-filter or click-to-detail.

## Accessible Data Visualization

A chart encodes meaning in **position, color, size, and shape**. Users who cannot distinguish
those encodings get nothing at all — a chart is the easiest place in a UI to lose a user
completely. This section is not optional polish; most of it is required by WCAG 2.2 AA.

### Never encode meaning in color alone (SC 1.4.1)

Roughly 1 in 12 men and 1 in 200 women have a color vision deficiency — most commonly
red–green, which is exactly the pairing charts use for up/down, good/bad, profit/loss. (Those
figures are for populations of Northern European descent, the ones the standard statistic is
drawn from; prevalence varies by population, but never to zero. Design for it regardless of
your audience.)

Redundant encoding, in order of preference:

- **Direct labels on the data** — the strongest fix, and it removes the legend entirely
- **Distinct shapes or markers** per series (circle, square, triangle) on lines and scatters
- **Distinct line styles** (solid, dashed, dotted)
- **Patterns or textures** in bars and areas
- **Text or arrow indicators** — "↑ 12%" carries the direction without color

Semantic red/green for success/failure must always be paired with an icon and a word
→ [../visual-design/color.md](../visual-design/color.md).

### Palette rules

- **Categorical:** use a colorblind-safe qualitative palette (Okabe–Ito's 8-color set and
  ColorBrewer's qualitative sets are the standard, empirically-derived choices). Do not invent
  one by picking evenly spaced hues.
- **Sequential:** vary **lightness**, not hue. A ramp that only changes hue is unreadable in
  grayscale and to many users. OKLCH makes this mechanical — hold hue and chroma, step
  lightness → [../visual-design/color.md](../visual-design/color.md).
- **Diverging:** use two hues that differ in lightness as well, with a light neutral midpoint.
  Avoid red–green; blue–orange and blue–red are safe.
- **Cap categorical series at ~6.** Beyond that, no palette is reliably distinguishable — group
  the tail into "Other" and offer drill-down.
- **The grayscale test:** desaturate the chart. If series become indistinguishable, the encoding
  fails. This one check catches most palette problems.

### Contrast in charts

- **Adjacent data segments need 3:1** against each other, and **all data marks need 3:1**
  against the background — SC 1.4.11 covers graphical objects, not just controls.
- Axis labels, tick labels, and legends are **text**: 4.5:1.
- Gridlines are decorative and are exempt — but if a gridline is the only way to read a value,
  it isn't decorative.
- Thin lines and small scatter points need *more* contrast than the minimum to actually resolve.

### Every chart needs a text equivalent

A chart is a non-text element and requires an alternative (SC 1.1.1). "Chart" or "Bar graph" is
a failure — it names the format and omits the content.

- **Short description** on the image/SVG: what it shows and the *conclusion*, e.g. "Monthly
  revenue, Jan–Dec 2026: steady growth from SAR 40k to SAR 120k, with a dip in August."
- **The underlying data must be reachable** — an adjacent (optionally collapsed) data table, or
  a download. The table is the real accessible alternative; the summary is the convenience.
- For inline SVG, use `role="img"` with an `aria-label`, or `<title>` referenced by
  `aria-labelledby`. Mark purely decorative SVG `aria-hidden="true"`.
- **Never make a tooltip the only way to read a value.** Tooltips are hover-only unless you
  build keyboard support, so hover-only tooltips fail keyboard and touch users
  → [../accessibility/keyboard-focus.md](../accessibility/keyboard-focus.md).

### Keyboard and screen reader

- Interactive charts need a keyboard path: focus the chart, arrow between data points, and
  announce each point's label and value.
- If that is too costly, **the data table is the accessible path** — provide it and make it
  easy to find. A well-marked-up table is a complete, legitimate alternative.
- Live-updating charts must not announce every tick. Debounce, or mark the region
  `aria-live="off"` and provide an explicit "updated" summary
  → [../interaction-design/feedback-systems.md](../interaction-design/feedback-systems.md).

### Accessible tables

The table rules above are about layout; these are about semantics:

- Real `<table>` markup with `<th>` and `scope="col"`/`scope="row"`. A grid of
  `<div>`s conveys nothing to a screen reader.
- Give the table a `<caption>`.
- **Sticky headers are visual only** — `<th>` is what makes a header a header.
- Sort controls must expose state via `aria-sort`, and announce the change after sorting.
- Horizontally scrolling tables need `tabindex="0"` on the scroll container so keyboard users
  can scroll them, plus an accessible name.
- **Truncation with a hover-only tooltip loses data** for keyboard and touch users. Expose the
  full value on focus too.

### Honest charts

Misleading charts are a design defect, and in a product they shade into dark patterns
→ [../anti-patterns/dark-patterns.md](../anti-patterns/dark-patterns.md).

- **Bar charts start at zero.** Always. Truncating the axis exaggerates differences.
- Line charts may use a non-zero baseline, but the axis must be labeled clearly.
- Never use dual y-axes to imply a correlation — the relationship is an artifact of the chosen
  scales.
- No 3D, no perspective, no shadows on data — they distort area and length judgments.
- Donut/pie only for 2–5 segments summing to a meaningful whole, and label the values.

## Data Display Checklist

- [ ] No meaning carried by color alone; grayscale test passes
- [ ] Colorblind-safe categorical palette; sequential ramps vary lightness
- [ ] Data marks 3:1 against background and against adjacent segments; label text 4.5:1
- [ ] Every chart has a meaningful text alternative stating the conclusion
- [ ] Underlying data reachable as a table or download
- [ ] No value readable only via hover
- [ ] Tables use real `<table>`/`<th>`/`scope`/`<caption>`; sorting exposes `aria-sort`
- [ ] Scrollable table containers are keyboard-focusable
- [ ] Bar charts start at zero; no 3D; no dual-axis correlation claims
- [ ] Loading, empty, and error states designed for every chart and table
      → [../interaction-design/state-patterns.md](../interaction-design/state-patterns.md)

## References
- Edward Tufte, *The Visual Display of Quantitative Information*
- NN/g data visualization articles
- Okabe & Ito, colorblind-safe qualitative palette: https://jfly.uni-koeln.de/color/
- ColorBrewer (sequential/diverging/qualitative scales): https://colorbrewer2.org
- WAI, "Complex Images" (text alternatives for charts): https://www.w3.org/WAI/tutorials/images/complex/