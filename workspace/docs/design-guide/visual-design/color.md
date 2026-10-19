# Color in UI Design

Color is the most misused tool in AI-generated designs. It's treated as decoration when it should be treated as a functional communication system.

## The Functional Roles of Color

Every color on screen serves one or more of these roles. If a color doesn't serve a clear role, remove it.

| Role | Purpose | Example |
|------|---------|--------|
| **Semantic** | Communicate meaning | Red = danger/delete, Green = success, Yellow = warning |
| **Interactive** | Indicate clickability | Link blue, button fill color |
| **Hierarchical** | Show importance | Primary CTA color vs. muted secondary |
| **Structural** | Group or separate | Section backgrounds, card borders, dividers |
| **Branding** | Establish identity | Logo color, brand accent |

**Rule:** Never use a semantic color for branding, and never use a brand color as a semantic signal without considering the meaning it might convey.

## Building a UI Color System

### Step 1: Neutral Palette
Start with neutrals. A good neutral palette has 8–12 steps from white to black.

```
White (#FFFFFF) → Gray 50 → Gray 100 → ... → Gray 900 → Black (#000000)
```

- **Background:** Use gray-50 or white
- **Surface (cards, modals):** White or gray-50
- **Border:** Gray-200 or gray-300
- **Secondary text:** Gray-500 or gray-600
- **Primary text:** Gray-900 or near-black

**Common AI failure:** Using pure black (#000) for body text. Near-black (like #1a1a2e, #111827) is more comfortable to read, especially on light backgrounds.

### Step 2: Semantic Colors
Define exactly 4–5 semantic colors with clear usage rules:

- **Primary:** The brand color, used for primary actions and key interactive elements
- **Success:** Positive outcomes (confirmations, completions, online status)
- **Warning:** Cautions (unsaved changes, approaching limits)
- **Danger/Destructive:** Irreversible or harmful actions (delete, remove, critical errors)
- **Info:** Informational (tips, external links, help text)

Each semantic color needs a full scale (50–900) for backgrounds, text, borders, and hover states.

### Step 3: 60:30:10 Distribution Rule

- **60%** — Dominant color (usually neutrals/white for backgrounds)
- **30%** — Secondary color (surface colors, secondary text, supporting elements)
- **10%** — Accent color (primary CTA, key highlights)

**AI failure:** Using bright accent colors on 30–40% of the surface, creating visual noise and diminishing the accent's impact.

## Work in OKLCH, Not Hex

Hex and HSL are how colors get *written down*; they are a poor space to *think* in. sRGB-HSL
lightness does not match perceived lightness — `hsl(60 100% 50%)` (yellow) and
`hsl(240 100% 50%)` (blue) claim identical 50% lightness and look nothing alike. This is
precisely why generated palettes tend to have muddy mid-tones and uneven ramps: the steps are
numerically even and perceptually uneven.

**OKLCH is perceptually uniform.** `oklch(L C H)` — lightness 0–1, chroma, hue 0–360.

```css
:root {
  /* one hue, an even perceptual ramp — L actually tracks what the eye sees */
  --brand-100: oklch(0.95 0.03 265);
  --brand-500: oklch(0.62 0.19 265);
  --brand-900: oklch(0.28 0.10 265);
}
```

Why this matters when generating a palette:

- **Ramps become mechanical.** Hold C and H, step L evenly, and the scale is even *by
  construction* — no eyeballing.
- **Hue shifts stop being accidental.** Changing lightness in hex drifts the hue; in OKLCH it
  does not.
- **Wide gamut is reachable.** OKLCH can express colors outside sRGB (Display P3), so a brand
  color is not silently clipped on modern screens.
- **Theming gets cheap.** `color-mix(in oklch, …)` and relative color syntax
  (`oklch(from var(--brand) calc(l - 0.1) c h)`) derive hover/active/disabled variants from
  one token instead of hand-picking a dozen hexes.

**Support, checked:** the `web-features` Baseline dataset (v3.36.0) lists Oklab/OkLCh as
**Widely available since 2025-11-09** — this is a safe default, not an experiment.
`color-mix()` is also widely available (2025-11-09); **relative color syntax** is only
*newly* available (2024-09-16), so treat `oklch(from …)` as the newer of the two techniques.

**Remaining constraints, honestly:** not every design tool round-trips OKLCH, and gamut
mapping still differs between engines at the extremes. Contrast
rules below are unaffected: **WCAG 2.2 ratios are defined on sRGB regardless of the notation
you author in**, so a P3 color must still pass as its sRGB equivalent — a vivid P3 accent can
look brighter than it legally counts as.

> The CSS-level syntax and gamut-mapping details live in the sibling `css/` folder,
> `06-color-and-theming.md`. This file is about choosing colors; that one is about
> expressing them.

## Contrast Requirements (Non-Negotiable)

### Current Standard: WCAG 2.2 AA

| Element | Minimum Contrast Ratio |
|---------|----------------------|
| Normal text (< 18pt / 14pt bold) | 4.5:1 |
| Large text (≥ 18pt / ≥ 14pt bold) | 3:1 |
| UI components and graphical objects | 3:1 |
| Focus indicators | 3:1 against adjacent colors |

### WCAG 3.0 — Do Not Design For It Yet

WCAG 3.0 is a W3C Working Draft, years from completion (~2030 at the earliest). Its final contrast algorithm is **undetermined** — the often-mentioned APCA was removed from the draft in 2023 and continues only as independent draft guidance. Ignore APCA for compliance; design for WCAG 2.2 AA.

Full status and details → [../accessibility/contrast-readable.md](../accessibility/contrast-readable.md)

### Practical Contrast Rules

- **Never put light gray text on white backgrounds.** Gray-400 on white typically fails WCAG.
- **Check your neutrals.** Many design systems' "secondary text" colors fail contrast requirements.
- **Don't use color as the only indicator.** Always pair color with text, icons, or patterns.
- **Dark mode isn't just inverting.** Dark backgrounds with light text require their own contrast testing.

## Dark Mode Considerations

Dark mode is not achieved by inverting colors. It requires a separate, carefully calibrated palette:

- **Background:** Not pure black (#000). Use very dark gray (#121212, #1a1a2e) to reduce glare
- **Surface:** Slightly lighter than background (#1e1e2e, #252540)
- **Text:** Not pure white. Use off-white (#e5e5e5, #f0f0f0) to reduce eye strain
- **Elevation:** Lighter surfaces = higher elevation (opposite of light mode)
- **Desaturate colors slightly** — bright colors on dark backgrounds create excessive contrast

## Forced Colors and User Contrast Preferences

Contrast ratios are not the whole accessibility story for color. Three media queries let the
user tell you what they need — and they are absent from almost all generated CSS:

```css
@media (prefers-contrast: more) { :root { --border: oklch(0 0 0); } }

@media (prefers-reduced-transparency: reduce) {
  .glass { backdrop-filter: none; background: var(--surface); }
}

@media (forced-colors: active) {
  /* Windows High Contrast Mode: the OS replaces your palette entirely. */
  .button { border: 1px solid ButtonText; }   /* system color keywords still apply */
}
```

- **`forced-colors: active`** — the OS overrides color, background, and border. Anything you
  communicated *only* through a background color or image disappears. Check that focus rings,
  selected states, and icon-only buttons survive. Use `forced-color-adjust` sparingly and
  never to defeat the user's setting.
- **`prefers-contrast: more`** — strengthen borders and separators; this is where hairline
  `#eee` dividers vanish entirely.
- **`prefers-reduced-transparency`** — the frosted-glass look is exactly what this setting
  exists to turn off. If a design leans on it, it needs a solid fallback →
  [../anti-patterns/ai-design-cliches.md](../anti-patterns/ai-design-cliches.md).

**Support, checked (`web-features` v3.36.0):** `forced-colors` and `prefers-contrast` are
both **widely available** (2025-03-12 and 2024-11-30) — use them freely.
`prefers-reduced-transparency` is **not Baseline**: it works in Chromium, not everywhere. It
still costs nothing to include, because an unsupported media query is simply ignored, but do
not rely on it as the *only* protection — if heavy transparency would make text unreadable,
fix the base design rather than deferring to a query that may never match.

## Color and Accessibility Beyond Contrast

- **Color blindness:** ~8% of men, 0.5% of women. Never use red/green as the only differentiator
- **`prefers-color-scheme`:** Respect the user's OS-level preference as a default
- **High contrast mode:** Test with Windows High Contrast Mode — many "modern" designs break completely

## References

- WCAG 2.2 contrast requirements: https://www.w3.org/WAI/WCAG21/Understanding/contrast-minimum.html
- APCA (independent draft guidance): https://git.apcacontrast.com
- Material Design color system (reference for scale approach)
- Adrian Roselli, "WCAG3 Contrast as of April 2026": http://adrianroselli.com/2026/04/wcag3-contrast-as-of-april-2026.html
