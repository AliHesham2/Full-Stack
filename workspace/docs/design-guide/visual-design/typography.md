# Typography in UI Design

Typography is the skeleton of a UI. Get it right and the design holds together even with minimal decoration. Get it wrong and no amount of color or spacing can compensate.

## Type Scale

A type scale is a set of font sizes that relate to each other by a consistent ratio. This creates visual rhythm and mathematical harmony.

### Modular Scale

The most common approach: each size is the previous size multiplied by a ratio.

| Ratio | Character | Use when |
|-------|-----------|----------|
| 1.067 (Minor Second) | Tight, controlled | Data-dense applications, dashboards |
| 1.125 (Major Second) | Calm, professional | Enterprise software, documentation |
| 1.200 (Minor Third) | Clear, readable | General web apps, SaaS |
| 1.250 (Major Third) | Expressive, spacious | Marketing sites, editorial |
| 1.333 (Perfect Fourth) | Bold, dramatic | Hero sections, landing pages |

### Practical Type Scale (1.200 ratio, base 16px)

```
xs:     11.1px  → 11px    (captions, badges)
sm:     13.3px  → 13px    (metadata, helper text)
base:   16px            (body text, default)
md:     19.2px  → 19px    (slightly emphasized body)
lg:     23px    → 23px    (subheadings)
xl:     27.7px  → 28px    (section headings)
2xl:    33.2px  → 33px    (page titles)
3xl:    39.9px  → 40px    (display, hero)
```

**The scale defines more sizes than any one screen should use.** These are the eight steps
available; a single view should draw on roughly **4–6** of them. Defining a step costs nothing
— using every step on one screen is what flattens the hierarchy, because adjacent sizes stop
reading as different levels.

**AI failure:** Using nearly every step on one screen, or picking sizes that don't relate
mathematically, creating an inconsistent visual rhythm. Also: using sizes that are too similar
(14px, 15px, 16px), which creates ambiguity about hierarchy rather than expressing it.

## Line Height

Line height (leading) has a direct impact on readability. The right line height depends on the font size and context:

| Context | Recommended Line Height |
|---------|----------------------|
| Body text (14–18px) | 1.5 – 1.6 (150–160%) |
| Headings (24px+) | 1.1 – 1.3 |
| Small text (11–13px) | 1.5 – 1.7 |
| Single-line elements (buttons, nav) | 1.0 – 1.2 |

**Common mistakes:**
- Line height too tight (< 1.4 for body text) — lines feel cramped, hard to read
- Line height too loose (> 1.8 for body text) — text feels disconnected, paragraphs lose cohesion
- Same line height for headings and body — headings should be tighter

## Font Pairing

### The Safe Approach
Use one typeface family with multiple weights. This is the recommended default for AI agents because:
- It guarantees optical harmony
- Variable fonts make this cost-free (single download)
- It avoids the #1 pairing mistake: combining fonts with similar x-heights

### When to Use Two Fonts

Pair a **display/heading font** with a **body font**. The pair should contrast in at least two dimensions:
- Serif heading + Sans body
- Geometric heading + Humanist body
- Condensed heading + Normal-width body

**Never pair:** Two similar sans-serifs, two similar serifs, or fonts with the same x-height.

### Recommended Font Categories for UI

| Category | Examples | Strengths | Weaknesses |
|----------|----------|-----------|------------|
| Grotesque/Neo-grotesque | Inter, Helvetica Neue, IBM Plex Sans | Excellent screen readability, neutral | Can feel generic |
| Humanist | Open Sans, Nunito, Source Sans | Friendly, readable | Less distinctive at display sizes |
| Geometric | Poppins, Montserrat, DM Sans | Modern, clean | Can feel cold at small sizes |
| Monospace (for code) | JetBrains Mono, Fira Code, IBM Plex Mono | Essential for code/data | Not for body text |

## Font Weight Hierarchy

Establish 3–4 weights and use them consistently:

| Weight | Value | Use for |
|--------|-------|----------|
| Regular | 400 | Body text, default |
| Medium | 500 | Slightly emphasized text, subheadings |
| Semibold | 600 | Headings, primary labels |
| Bold | 700 | Rare emphasis only |

**AI failure:** Using font-weight: 700 (bold) for body text or paragraph-level emphasis. Bold body text signals a lack of confidence in the hierarchy. Let size and spacing do the work.

## Letter Spacing (Tracking)

- **Body text:** Use the font's default tracking. Don't add letter-spacing to body text — it slows reading.
- **Small caps / Uppercase labels:** Slight positive tracking (0.02–0.05em) improves readability of all-caps text
- **Large headings:** Slight negative tracking (-0.01 to -0.02em) can improve visual cohesion at large sizes

## Fluid Type with `clamp()`

A fixed scale plus breakpoints produces a visible jump in heading size at each threshold.
`clamp()` removes both the jump and the breakpoints:

```css
h1 { font-size: clamp(1.75rem, 1.2rem + 2.5vw, 3rem); }
```

**Two rules that are easy to get wrong here:**

- Always include a `rem` term in the middle (preferred) value. A pure-`vw` preferred value
  does not respond to the user's browser font-size setting and can fail
  **WCAG 1.4.4 Resize Text**.
- Keep the `min` at a genuinely readable size — `clamp()` makes it easy to ship 11px body
  text on phones without noticing.

Use fluid type for headings and display sizes. Body text is usually better at a fixed,
deliberate size; smoothly-scaling body copy mostly produces sizes nobody chose.

## Modern Text Rendering

Two properties that separate typography that looks considered from typography that looks
defaulted:

```css
h1, h2, h3, .card-title { text-wrap: balance; }   /* even lines in short headings */
p                       { text-wrap: pretty; }    /* prevents orphans in body copy */
```

- **`text-wrap: balance`** — evens out line lengths. Intended for **short** blocks; engines
  cap how many lines they will balance, so apply it to headings, blockquotes, and card titles,
  not to paragraphs. It fixes the classic ragged two-line heading with one orphaned word.
- **`text-wrap: pretty`** — for body copy; optimises the last lines to avoid orphans, at
  lower cost than `balance`.
- Both degrade silently to normal wrapping where unsupported, so there is no reason to gate
  them behind `@supports`.

**Support, checked (`web-features` Baseline v3.36.0):** `text-wrap: balance` is **newly
available (2024-05-13)**; `text-wrap: pretty` is **not Baseline yet** — Chromium and Safari
have it, Firefox does not. Use both anyway: the failure mode is ordinary wrapping. Just do not
design a layout that only works if `pretty` applies, and do not describe `pretty` as
supported everywhere.

## Font Loading — a Typography Decision, Not a Build Detail

The type scale is irrelevant if text is invisible for two seconds or reflows after paint.

```css
@font-face {
  font-family: "Brand";
  src: url("/fonts/brand.woff2") format("woff2");
  font-display: swap;      /* show fallback immediately, swap when ready */
  size-adjust: 104%;       /* match fallback metrics to kill the reflow */
  ascent-override: 90%;
}
```

| `font-display` | Behaviour | Use when |
|---|---|---|
| `swap` | Fallback shown instantly, swapped later (FOUT) | Body text — content must be readable |
| `optional` | Very short block, no swap if slow | Best for layout stability; font may not appear at all |
| `block` | Invisible text up to ~3s (FOIT) | Almost never — this hides content |

- **Preload only the fonts used above the fold**, woff2 only:
  `<link rel="preload" as="font" type="font/woff2" crossorigin href="…">`. The
  `crossorigin` attribute is required *even same-origin*, or the file downloads twice.
- **Subset.** A Latin-only subset is a fraction of the full file. For Arabic this matters even
  more → [../accessibility/rtl-i18n.md](../accessibility/rtl-i18n.md).
- **Match fallback metrics** with `size-adjust` / `ascent-override` so the swap does not
  shift layout. This is the difference between FOUT nobody notices and FOUT that jumps.
  *Support:* `font-display` is widely available (since 2022), but **font metric overrides
  (`size-adjust`, `ascent-override`, `descent-override`) are not Baseline** — unsupported
  engines ignore the descriptor and you get the ordinary reflow, so including them is a
  free win where they land, not a guarantee everywhere.
- **Variable fonts** collapse many weights into one file — which is what makes the weight
  hierarchy above affordable. One variable file usually beats three static weights.

## Text Width (Measure)

Optimal line length for readability: **45–75 characters** (including spaces). Lines that are too long cause the eye to lose its place when scanning to the next line. Lines that are too short cause excessive line breaks that fragment reading.

For responsive design:
- Set `max-width: 65–70ch` on text containers (where `ch` = the width of the "0" character)
- Don't let body text span the full width on large screens

## References

- Modular Scale calculator: https://typescale.com
- Material Design typography guidelines (reference for scale approach)
- Butterick's Practical Typography: practicaltypography.com
