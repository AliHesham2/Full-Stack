# Design Tokens

Design tokens are the stored decisions of a design system — color values, spacing, typography, and other design choices in a platform-agnostic format.

## Why They Matter for AI Agents

When an AI agent generates a UI, it's implicitly making design token decisions (even if it doesn't name them). Being explicit about tokens ensures:
- Consistency across every screen and component
- The ability to change themes (light/dark) by swapping token values
- The ability to adapt to different platforms (web, iOS, Android) from the same decisions

## Token Layers (W3C Design Tokens Format)

The W3C Design Tokens Community Group published the first stable specification (v2025.10)
on 28 October 2025. It defines a standard format for exchanging tokens between tools.

Verified by fetching the document itself: it self-identifies as a **Final Community Group
Report, 28 October 2025** (`CG-FINAL-format-20251028`). That status matters when you cite it
— a CG Final Report is stable, citable and implementable under the W3C Community Final
Specification Agreement, but it is **not** a W3C Recommendation and did not go through the
Recommendation track. Call it "the DTCG format spec," never "the W3C standard."

### Three-Tier Token Model

| Tier | Name | Purpose | Example |
|------|------|---------|--------|
| 1 | **Primitive/Global** | Raw values | `gray.900: #111827` |
| 2 | **Semantic** | Intent-based | `color.text.primary: {gray.900}` |
| 3 | **Component** | Applied to components | `button.primary.background: {color.brand}` |

### Practical Token Categories

```json
{
  "color": {
    "background": "#ffffff",
    "surface": "#f9fafb",
    "border": "#e5e7eb",
    "text-primary": "#111827",
    "text-secondary": "#6b7280",
    "brand": "#3b82f6"
  },
  "spacing": {
    "xs": "4px",
    "sm": "8px",
    "md": "16px",
    "lg": "24px",
    "xl": "32px"
  },
  "typography": {
    "font-family": "Inter, system-ui, sans-serif",
    "font-size-base": "16px",
    "line-height-body": "1.5"
  },
  "radius": {
    "sm": "4px",
    "md": "8px",
    "lg": "12px",
    "full": "9999px"
  }
}
```

### When to Use Tokens vs. Hardcoded Values

- **Use tokens for:** colors, spacing, typography, border radius, shadows, transitions — anything that should be consistent or themeable
- **Hardcoded values are acceptable for:** one-off positioning values, complex calculations, animation keyframes

## References
- W3C Design Tokens Spec (v2025.10): https://www.w3.org/community/reports/design-tokens/CG-FINAL-format-20251028/
- W3C Design Tokens Community Group: https://www.w3.org/community/design-tokens
- Contentful: "Design tokens explained" (May 2024 — principles still valid)
