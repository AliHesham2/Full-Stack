# Build and Tooling (2026)

## Build tools
| Tool | Use case |
|------|----------|
| Next.js 16 | **This project's frontend** (frontend only: data comes from Express). 16.3.6 source-checked target (2026-09-26); historical execution below used 16.3.4. Turbopack is the default for **build**, not just dev — verified: `next build` banners `▲ Next.js 16.3.4 (Turbopack)` with no flag |
| Vite 8 | Not used to build this project. It runs only inside Vitest, the test runner |

Setup of the frontend: frontend-guide/01-foundation/initialization.md.

## Linting
```bash
# ESLint + hooks plugin — current major is v7 (7.1.x); flat config default since v6
npm install -D eslint eslint-plugin-react-hooks

# Oxlint — Rust-based, fast, React Compiler-aware rules
npm install -D oxlint
```
Add `eslint-plugin-jsx-a11y` for accessibility linting of JSX.

## Environment variables

**Rule**: NEXT_PUBLIC_* values are public and build-time embedded. next.config.ts env values are also bundled, regardless of prefix; never put secrets there. Keep secrets in server-only modules, and never serialize them into props/action results or rendered HTML.
```bash
EXPRESS_API_URL=http://localhost:4000    # server-only backend URL
EXPRESS_SERVICE_KEY=                    # optional service credential; never a Supabase key
NEXT_PUBLIC_SITE_NAME=Acme               # public: never a URL or a key the browser must not see
```
There is never a database URL or a Supabase key in the frontend: those live only in Express's
environment (frontend-guide/04-quality/security.md).

## Pinned versions (Sept 2026)

```
react / react-dom           19.3.0   (stable; checked 2026-09-26)
next                        16.3.6
react-router                8.3.1    ← the package to import from
vite                        8.2.2
@vitejs/plugin-react        6.1.1
babel-plugin-react-compiler 1.0.0
eslint-plugin-react-hooks   7.1.1
@tanstack/react-query       5.102.x
tailwindcss                 4.3.3
```

## Package manager
**npm 11 or newer.** npm 10.9 (bundled with Node 22) crashes installing Vitest — verified,
frontend-guide/01-foundation/initialization.md.

## No monorepo tooling
The project is two independent apps side by side, `frontend/` and `backend/`, each with its own
`package.json`. No Turborepo, Nx or workspaces.

## Compiler wiring reminder
In Next.js the React Compiler is the `reactCompiler` option in `next.config.ts`, not a Vite plugin:
→ full config: [react-compiler.md](./react-compiler.md)

→ Sources: [vitejs.dev](https://vitejs.dev/), [nextjs.org/docs](https://nextjs.org/docs/)
