# Tailwind CSS v4 — Integration

## Quick Start: PostCSS

```bash
npm i tailwindcss @tailwindcss/postcss
```

```js
// postcss.config.mjs
export default {
  plugins: {
    "@tailwindcss/postcss": {},
  },
};
```

```css
/* app.css */
@import "tailwindcss";
```

Remove `postcss-import` and `autoprefixer` — both are handled internally by v4.

Source: https://tailwindcss.com/docs/installation/using-postcss

---

## Quick Start: Vite Plugin (Recommended for Vite)

```bash
npm i tailwindcss @tailwindcss/vite
```

```ts
// vite.config.ts
import { defineConfig } from "vite";
import tailwindcss from "@tailwindcss/vite";

export default defineConfig({
  plugins: [tailwindcss()],
});
```

```css
/* app.css */
@import "tailwindcss";
```

**Why Vite plugin over PostCSS**: Tighter integration = faster builds, better HMR. The Vite plugin bypasses PostCSS overhead entirely and integrates directly with Vite's module system.

**Frameworks that use Vite under the hood** (Laravel, SvelteKit, React Router, Nuxt, SolidJS) should use `@tailwindcss/vite`.

Source: https://tailwindcss.com/docs/installation/vite

---

## Next.js

Tailwind CSS v4 works with Next.js via PostCSS (Next.js doesn't expose its Vite config):

```bash
npm i tailwindcss @tailwindcss/postcss
```

```js
// postcss.config.mjs
export default {
  plugins: {
    "@tailwindcss/postcss": {},
  },
};
```

```css
/* app/globals.css */
@import "tailwindcss";
```

**Note**: Modern `create-next-app` scaffolds v4 correctly (`@tailwindcss/postcss` + `@import "tailwindcss"`). Only older scaffolds leave v3 config behind. The `content` array is not needed — v4 auto-detects.

Source: https://tailwindcss.com/docs/guides/nextjs

---

## Tailwind CLI

```bash
npm i tailwindcss @tailwindcss/cli
```

```bash
npx @tailwindcss/cli -i src/app.css -o dist/output.css --watch
```

Useful for non-bundler environments, static sites, or email templates.

## Webpack (v4.2+)

```bash
npm i tailwindcss @tailwindcss/webpack
```

```js
// webpack.config.js
module.exports = {
  module: {
    rules: [
      { test: /\.css$/i, use: [MiniCssExtractPlugin.loader, "css-loader", "@tailwindcss/webpack"] },
    ],
  },
};
```

First-class loader — skips PostCSS entirely and is 2x+ faster than the `@tailwindcss/postcss` route in large webpack/Turbopack builds. Works through Turbopack's webpack-loader compatibility layer.

Source: https://tailwindcss.com/blog/tailwindcss-v4-3

---

## Browser Build (Prototyping Only)

`@tailwindcss/browser` (v4.1+) compiles in the browser via a CDN script — for demos, prototypes, and CodePen-style play. Never use in production:

```html
<script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>
```

---

## NPM Workspaces / Monorepos

v4's automatic content detection respects `.gitignore` and project boundaries. For monorepos with shared UI packages:

```css
@import "tailwindcss";

/* Add shared UI package as a source */
@source "../packages/ui/src";
```

Each workspace should have its own CSS entry point. The Vite plugin supports workspace-aware setup via Nx or similar tools.

Source: https://nx.dev/blog/setup-tailwind-4-npm-workspace

---

## Framework-Specific Notes

### SvelteKit

Use the Vite plugin (`@tailwindcss/vite`) in `vite.config.ts`. No additional configuration needed.

### React (Vite)

Use the Vite plugin. Import the CSS file in your `main.tsx`.

### Vue (Vite)

Use the Vite plugin. Import in `main.ts`. If using `<style>` blocks, utilities in templates are detected automatically.

### Laravel (Vite)

Laravel uses Vite internally. Use `@tailwindcss/vite` in `vite.config.js` and `@import "tailwindcss"` in `resources/css/app.css`.

### Astro

Use the Vite plugin in `astro.config.mjs` — `@astrojs/tailwind` is **deprecated** (since Astro 5.2) and only exists for old Tailwind v3 projects:

```js
import tailwindcss from "@tailwindcss/vite";

export default defineConfig({
  vite: { plugins: [tailwindcss()] },
});
```

Source: https://tailwindcss.com/docs/installation/framework-guides/astro

### Rails / Importmap

Use the CLI approach with `@tailwindcss/cli` to build a standalone CSS file.

---

## CSS File Organization

Recommended pattern — one CSS entry point:

```css
/* app.css */
@import "tailwindcss";

@theme {
  /* Project tokens */
}

@layer base {
  /* Project resets */
}

@layer components {
  /* Shared component styles */
}

@custom-variant dark (&:where(.dark, .dark *));
```

You can split into multiple files using standard CSS `@import`:

```css
/* app.css */
@import "tailwindcss";
@import "./theme.css";
@import "./components.css";
```

No `postcss-import` needed — v4 handles this natively.

---

## Common Setup Mistakes

1. **Forgetting to remove `autoprefixer`** — It's not harmful but adds unnecessary processing.
2. **Keeping `postcss-import`** — v4 handles imports. Having both can cause double-processing.
3. **Using old `@tailwind` directives** — Must use `@import "tailwindcss"`.
4. **Creating empty `tailwind.config.js`** — Not needed. Delete it unless using `@config` bridge.
5. **Vite plugin + PostCSS plugin simultaneously** — Choose one. Don't use both.
