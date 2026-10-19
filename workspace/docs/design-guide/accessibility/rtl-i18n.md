# RTL and Internationalization

Most UI guidance silently assumes left-to-right, English. Interfaces for Arabic, Hebrew, and other RTL languages need mirrored layout logic — not just translated strings. Directly relevant for Arabic-first products (e.g., Saudi/GCC e-commerce themes).

## The Core Rule

**Mirror the layout, never the content.** Directional *layout* (position, flow, icons that imply direction) flips in RTL. Media, logos, clocks, and text content do not flip.

## What Mirrors in RTL

| Element | LTR | RTL |
|---------|-----|-----|
| Reading flow / scan pattern | Top-left start | Top-right start |
| Text alignment | Left | Right |
| Sidebar navigation | Left side | Right side |
| Breadcrumb separator direction | Home > Page | Home < Page (visually reversed order) |
| Back/forward arrows | ← back, → forward | → back, ← forward |
| Chevrons on expandable items | `>` points right | `<` points left |
| Progress/timeline direction | Left to right | Right to left |
| Icon+label button ordering | [icon][label] | [label][icon] |

## What Does NOT Mirror

- **Media**: photos, videos, illustrations (a photo of a person is still a photo)
- **Logos and brand marks**
- **Music players, clock faces, calculators** (physical metaphors)
- **Numbers**: Arabic-Indic digits are optional per locale; Western digits stay LTR either way
- **Phone numbers, code snippets, URLs**: always render LTR with `dir="auto"` or explicit wrapping
- **Mixed-direction text**: use `<bdi>` or `unicode-bidi: isolate` around user-generated content so a Latin brand name inside Arabic text doesn't scramble punctuation

## Implementation Rules (CSS)

Use **logical properties**, not physical ones. This makes one stylesheet serve both directions:

```css
/* BAD: physical — breaks in RTL */
.card { padding-left: 16px; text-align: left; margin-left: auto; }

/* GOOD: logical — flips automatically */
.card { padding-inline-start: 16px; text-align: start; margin-inline-start: auto; }
```

| Physical (avoid) | Logical (use) |
|------------------|---------------|
| `margin-left/right` | `margin-inline-start/end` |
| `padding-left/right` | `padding-inline-start/end` |
| `text-align: left/right` | `text-align: start/end` |
| `left` / `right` offsets | `inset-inline-start/end` |
| `border-left` | `border-inline-start` |

Set direction at the document level and let inheritance work:

```html
<html lang="ar" dir="rtl">
```

- `float`, flexbox `row`, and grid column order all follow the element's `direction` automatically — no manual flipping needed if you didn't hardcode physical values.
- Tailwind v4 note: prefer `ps-`/`pe-`/`ms-`/`me-`/`start-`/`end-` utilities over `pl-`/`pr-`/`ml-`/`mr-`/`left-`/`right-`.

## Typography Adjustments for Arabic Script

- **Line height needs more room**: Arabic ascenders/descenders and diacritics clip at 1.4 — use ≥1.6 for body text.
- **Choose an actual Arabic typeface**, don't rely on fallbacks: IBM Plex Sans Arabic, Noto Sans Arabic, Cairo, Tajawal, GE SS. A Latin font's "Arabic range" fallback renders inconsistently.
- **Font size**: Arabic script reads ~1–2px smaller at equal pixel size than Latin; consider +1px base or test with native readers.
- **No letter-spacing on Arabic**: tracking breaks connected script (it disconnects letters). Never apply `letter-spacing` to Arabic text.
- **No fake bold**: some Arabic webfonts lack real bold weights and browsers synthesize them, destroying letterforms. Load explicit weights.

## Forms, Numbers, Dates

- Input fields for names/addresses in RTL locales: align text `start`, keep numeric/email inputs LTR (`dir="ltr"` on those inputs).
- Validate against locale expectations: phone formats, postal codes, and name structures differ (no "first/last name" assumption — use "full name" or given/family per locale).
- Dates: use Intl APIs (`Intl.DateTimeFormat(locale)`), never hardcoded `MM/DD/YYYY`.
- Currency: SAR displays as "ر.س" or "SAR" depending on locale — position follows locale rules via `Intl.NumberFormat(locale, { style: 'currency', currency: 'SAR' })`.

## Testing Checklist

- [ ] Toggle `dir="rtl"` — does the layout mirror without manual overrides?
- [ ] No horizontal scrollbars from unmirrored fixed widths
- [ ] Icons implying direction (arrows, chevrons) point the correct way
- [ ] Mixed Latin/Arabic text doesn't scramble punctuation (`<bdi>`, `dir="auto"`)
- [ ] Arabic typography: real Arabic font loaded, line-height ≥1.6, no letter-spacing
- [ ] Form inputs behave correctly (numeric LTR, text RTL)
- [ ] Carousels/sliders advance in the correct visual direction

## References

- MDN: CSS logical properties — https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_logical_properties_and_values
- MDN: HTML `dir` attribute and `bdi` — https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/dir
- **W3C, _Arabic & Persian Layout Requirements_ — https://www.w3.org/TR/alreq/**
  The authoritative reference for Arabic script layout: justification, kashida, line breaking,
  baselines, numerals, and mixed-direction runs. Consult this before inventing an approach.
- W3C: structural markup and right-to-left text in HTML — https://www.w3.org/International/questions/qa-html-dir
- W3C i18n article index — https://www.w3.org/International/articlelist

*(An earlier revision cited `w3.org/International/questions/qa-arabic-script`; that URL now
404s. Link-check references before trusting them — a dead citation in a knowledge base is worse
than no citation, because it looks authoritative.)*
- Google Fonts: Arabic webfont pairing guidance (Noto Sans Arabic specimen)
