# Tailwind CSS — Knowledge Base

**Scope**: Tailwind CSS v4.x, current as of 2026-08 (latest stable: **v4.3.x**, 4.3.3 released 2026-07-16). Covers v4.0 (2025-01-22), v4.1 (2025-04-03), v4.2 (2026-02-18), v4.3 (2026-05-08). Only the latest minor line receives support.
**Audience**: AI agents and developers building with Tailwind CSS.
**Verification**: every utility, variant and directive claimed here was **compiled against tailwindcss 4.3.3** with @tailwindcss/cli and checked in the emitted CSS — not read from release notes. Three names in earlier drafts (`color-scheme-*`, `transform-style-*`, `descendant:`) turned out not to exist and were corrected.
**Baseline assumption**: New projects use v4.3+. Existing v3 projects reference `breaking-changes.md` for migration.

## File Index

| File | Content | When to read |
|------|---------|-------------|
| [architecture.md](./architecture.md) | Oxide engine, modern CSS foundations, build pipeline, performance | Understanding how TW works under the hood |
| [configuration.md](./configuration.md) | `@theme`, `@utility`, `@custom-variant`, `@source`, `@plugin` directives, design tokens | Setting up or modifying theme, tokens, custom utilities |
| [breaking-changes.md](./breaking-changes.md) | Complete v3→v4 migration reference with diffs | Upgrading an existing v3 project |
| [new-features.md](./new-features.md) | All v4.x features: container queries, 3D transforms, gradients, masks, text-shadow, logical properties, scrollbars, new variants/utilities per release | Discovering what's available in v4 |
| [integration.md](./integration.md) | Vite plugin, PostCSS, webpack, Next.js, CLI, browser build, framework-specific setup | Installing Tailwind in a project |
| [dark-mode.md](./dark-mode.md) | `prefers-color-scheme`, class-based, data-attribute, `@custom-variant dark` | Implementing dark/light theming |
| [best-practices.md](./best-practices.md) | Component extraction, `@apply` guidance, anti-patterns, performance, naming | Writing maintainable Tailwind code |
| [ecosystem.md](./ecosystem.md) | Plugin compat (`@plugin`), v3 compat layer, UnoCSS/PandaCSS comparison, shadcn/ui | Choosing plugins or evaluating alternatives |

## Key Facts at a Glance

- **Current stable**: v4.3.x (v4.0 2025-01-22, v4.1 2025-04-03, v4.2 2026-02-18, v4.3 2026-05-08)
- **Engine**: Oxide (Rust-based), uses Lightning CSS for vendor prefixing
- **Config**: CSS-first via `@theme` directive — no `tailwind.config.js`
- **Import**: `@import "tailwindcss"` replaces `@tailwind base/components/utilities`
- **Browser target**: Safari 16.4+, Chrome 111+, Firefox 128+ (graceful degradation in older browsers since v4.1)
- **Build speed**: up to 5x faster full builds, >100x faster incremental (microseconds) vs v3
- **Color space**: OKLCH (P3-aware) replaces sRGB
- **Packages**: `tailwindcss` + `@tailwindcss/postcss`, `@tailwindcss/vite`, `@tailwindcss/cli`, or `@tailwindcss/webpack` (v4.2+)
- **Upgrade tool**: `npx @tailwindcss/upgrade` (requires Node 20+)

## Deprecated / Removed — sorted by how they fail

Compiled against 4.3.3 to confirm each. **The three groups need completely different
urgency**, and mixing them is why migrations get abandoned half-done.

### 1. Removed — produce NO CSS, silently (fix first)
- `bg-opacity-*`, `text-opacity-*`, `border-opacity-*`, `divide-opacity-*`, `ring-opacity-*`,
  `placeholder-opacity-*` → use the `bg-blue-500/50` slash syntax
- `@tailwind base/components/utilities` → `@import "tailwindcss"`
- Sass/Less/Stylus preprocessing → unsupported; Tailwind is the preprocessor

### 2. Still compile, but the VALUE shifted — silent visual drift (fix second)
- `shadow-sm` now renders v3's `shadow`; `blur-sm` is 8px (was 4px); `rounded-sm` is
  0.25rem (was 0.125rem). Rename to `shadow-xs` / `blur-xs` / `rounded-xs` to keep the old look.
- `ring` is now 1px (was 3px) → `ring-3` restores it
- `outline-none` no longer carries the forced-colors fallback → `outline-hidden` does

### 3. Still compile identically — cosmetic renames (fix last, or never)
- `flex-shrink-*` / `flex-grow-*` → `shrink-*` / `grow-*`
- `overflow-ellipsis` → `text-ellipsis`
- `decoration-slice` / `-clone` → `box-decoration-*`
- `bg-gradient-to-*` → `bg-linear-to-*`
- `!flex` → `flex!`; `bg-[--var]` → `bg-(--var)`
- `start-*` / `end-*` → `inset-s-*` / `inset-e-*` (byte-identical output)
- `tailwind.config.js` → `@theme` in CSS (the `@config` bridge still works)
