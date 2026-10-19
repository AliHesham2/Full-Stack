# Tailwind CSS v4 — Ecosystem

## Plugin Compatibility

### Loading v3 Plugins in v4

Use the `@plugin` directive as a compatibility bridge:

```css
@import "tailwindcss";
@plugin "@tailwindcss/typography";
@plugin "daisyui";
```

The `@plugin` directive loads JavaScript plugins that use the v3 API (`addUtilities`, `addComponents`, `plugin()`). This is a **compatibility feature** — the JS plugin API is deprecated.

### Plugin Status (as of 2026-08)

| Plugin | v4 Status | Notes |
|--------|-----------|-------|
| `@tailwindcss/typography` | ✅ Updated for v4 | Load via `@plugin "@tailwindcss/typography"` |
| `@tailwindcss/forms` | ✅ Updated | Load via `@plugin` |
| `@tailwindcss/aspect-ratio` | ⚠️ Absorbed into core | Native `aspect-*` utilities |
| `@tailwindcss/container-queries` | ⚠️ Absorbed into core | Native `@container` support |
| daisyUI | ✅ v4 compatible (v5+) | Load via `@plugin "daisyui"` |
| tailwindcss-animate | ⚠️ Check version | May need `@plugin` compat |

**Rule**: Before upgrading, check each plugin's npm page or GitHub for v4 compatibility. Most popular plugins updated within weeks of v4's release.

### Writing Native v4 Plugins

v4 plugins are CSS-only, using `@utility`, `@custom-variant`, and `@theme`:

```css
/* my-plugin.css */
@theme {
  --color-accent-500: oklch(0.7 0.15 200);
}

@utility text-glow {
  text-shadow: 0 0 10px var(--color-accent-500);
}

@custom-variant tablet (@media (width >= 768px) and (width < 1024px));
```

Users import it:

```css
@import "tailwindcss";
@import "./my-plugin.css";
```

This is the recommended path for new plugins. No JavaScript, no build step.

---

## shadcn/ui

shadcn/ui is the dominant component library for Tailwind CSS. It copies component source code into your project (not an npm dependency).

### v4 Compatibility

shadcn/ui added v4 support in early 2025. When initializing in a v4 project:

```bash
npx shadcn@latest init
```

It detects v4 and generates compatible code (uses `@import "tailwindcss"`, CSS variables in `@theme`, etc.).

### Pattern

shadcn/ui defines design tokens as CSS variables, then uses them via Tailwind utilities. This aligns well with v4's CSS-first approach.

---

## Alternatives Comparison

### Tailwind CSS v4 vs UnoCSS

| Dimension | Tailwind v4 | UnoCSS |
|-----------|------------|--------|
| Build speed | 5x v3, ~100x incremental | An order of magnitude faster than TW |
| Output size | Larger (full framework) | Smaller (on-demand engine) |
| Ecosystem | Largest (shadcn, plugins, tutorials) | Smaller but growing |
| Config | CSS-first (`@theme`) | JS/TS config |
| Presets | Not applicable | Powerful preset system |
| Browser compat | Modern only (16.4+) | Configurable |
| Maturity | Production-proven, v4 stable | Production-proven |

**When to prefer UnoCSS**: Performance-critical apps needing sub-millisecond builds, teams comfortable with less ecosystem support, projects already using UnoCSS presets.

**When to prefer Tailwind v4**: Largest ecosystem, best component library support (shadcn/ui), CSS-first config is more maintainable for large teams, first-party framework integrations.

Source: https://www.pkgpulse.com/guides/tailwind-v4-vs-unocss-vs-pandacss-2026

### Tailwind CSS v4 vs PandaCSS

| Dimension | Tailwind v4 | PandaCSS |
|-----------|------------|----------|
| Approach | HTML utility classes | CSS-in-JS (extracted to CSS at build) |
| Type safety | None (string classes) | Full TypeScript type inference |
| Config | CSS-first | TypeScript/JS config |
| Runtime | Zero | Zero (extracted) |
| Recipes | No formal concept | Recipe system for component patterns |
| DX with LLMs | Excellent (string-based, widely trained) | Good but less training data |

**When to prefer PandaCSS**: TypeScript-heavy teams wanting type-safe styles, projects that benefit from recipe abstraction.

**When to prefer Tailwind v4**: Faster iteration, larger community, better LLM tooling support, more hiring familiarity.

---

## When Tailwind Is NOT the Right Choice

Based on community discussion and production experience:

1. **Large-scale design systems** with many non-utility patterns — consider CSS Modules or a dedicated design system framework
2. **Email HTML** — Tailwind's output can be large and email clients have limited CSS support. Use the CLI with aggressive purging, or consider dedicated email frameworks
3. **Teams that strongly prefer semantic CSS** and find utility classes reduce readability for their specific workflow
4. **Performance-critical contexts** where every byte matters — UnoCSS or hand-written CSS may produce smaller output

---

## Version Strategy

- **New projects**: Start with v4.3.x directly
- **Existing v3 projects**: Use `npx @tailwindcss/upgrade` on a branch. Most plugins have v4 support. Plan 1-2 days for testing and tweaks
- **v3 LTS**: the last v3 release is **3.4.19** (2025-12-10); security fixes only, no new features. npm carries a dedicated tag — install it explicitly with `npm i tailwindcss@v3-lts` rather than pinning `^3.4` and drifting
- **Staying on v3**: Only if you need older browser support (pre-Safari 16.4 / pre-Chrome 111)
- **On v4.0–v4.2**: Upgrade to the latest 4.3.x — only the newest minor line is actively supported