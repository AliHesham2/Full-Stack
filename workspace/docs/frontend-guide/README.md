# Frontend Project Guide

The frontend engineering rules for this project: how a frontend project is initialized,
structured, built phase-by-phase, validated, secured, tested, optimized, and delivered.
Specialized technology guides explain individual technologies; this guide owns the cross-cutting
engineering concerns no single technology guide covers. Routing between guides is not part of
this guide.

**Every path in this guide is relative to `frontend/`**, the Next.js app's folder in the project
root (`src/app/` means `frontend/src/app/`). Run its commands inside `frontend/`.

## Frontend-only contract boundary

Read [frontend-only scope](00-navigation/frontend-only-scope.md). The owner supplies external API definitions; Planner checks and maps them, never authors a backend or database schema. Preserved examples apply only to matching approved contracts.

## How to Use This Guide

Start at [GUIDE-MAP.md](GUIDE-MAP.md) for the reading flowchart and a short description of every file. It routes to this guide's actual procedures and rules; it does not replace them.

Read [agent-workflow.md](./00-navigation/agent-workflow.md) first — it defines the task workflow and the decision tree for which sections apply to your current task. Then read only what the task requires:

```
Task → 00-navigation/agent-workflow (which files) → only those files → decision-framework if stuck
```

**Stack:** Next.js (App Router) · TypeScript · Tailwind CSS v4 · shadcn/ui on Radix · `next-themes`
· external API backend reached only through `lib/http.ts`.

## File Map

| Folder | File | Owns |
|---|---|---|
| **00-navigation** | [agent-workflow](./00-navigation/agent-workflow.md) | Task workflow, which files to read, key rules. **Read first, every task** |
| | [decision-framework](./00-navigation/decision-framework.md) | Answer trees for "where does this go?" |
| **01-foundation** | [initialization](./01-foundation/initialization.md) | Phase 0 setup, technology defaults, verification |
| | [architecture](./01-foundation/architecture.md) | Rendering strategy, dependency direction, state ownership |
| | [project-structure](./01-foundation/project-structure.md) | Canonical folder tree, where every file goes |
| | [routing](./01-foundation/routing.md) | Route organization, loading/error boundaries, protected routes |
| **02-ui** | [design-tokens](./02-ui/design-tokens.md) | Semantic colors, every scale, applying the design |
| | [theme-and-dark-mode](./02-ui/theme-and-dark-mode.md) | The three CSS files, fonts, `next-themes` |
| | [components](./02-ui/components.md) | The `components/ui/` kit: API, variants, states, catalog |
| | [forms](./02-ui/forms.md) | Server Action forms, `FormField`, validation, uploads |
| | [layout-and-responsive](./02-ui/layout-and-responsive.md) | Breakpoints, container queries, layout per screen size |
| | [data-display-and-feedback](./02-ui/data-display-and-feedback.md) | Tables, cards, lists, states, toasts, dialogs |
| | [accessibility](./02-ui/accessibility.md) | WCAG 2.2 AA, ARIA, keyboard/focus, a11y testing |
| | [portability](./02-ui/portability.md) | Copying the kit to another project; purity checks |
| **03-data** | [state-and-data](./03-data/state-and-data.md) | Five state categories, caching, derived state |
| | [backend-integration](./03-data/backend-integration.md) | `apiFetch`, auth boundary, API contracts, uploads |
| **04-quality** | [security](./04-quality/security.md) · [performance](./04-quality/performance.md) · [resilience](./04-quality/resilience.md) · [testing](./04-quality/testing.md) · [dependencies](./04-quality/dependencies.md) · [code-quality](./04-quality/code-quality.md) · [anti-patterns](./04-quality/anti-patterns.md) | One quality dimension per file |
| **05-delivery** | [environments-and-config](./05-delivery/environments-and-config.md) · [delivery-and-observability](./05-delivery/delivery-and-observability.md) | Env/config, CI/CD, releases, monitoring |

## Priorities

**Non-negotiable on every task:** [accessibility](./02-ui/accessibility.md),
[security](./04-quality/security.md), [resilience](./04-quality/resilience.md) states.

**Load-bearing:** the `01-foundation/` files, and the rule that the browser and Next.js never
touch the database ([backend-integration](./03-data/backend-integration.md)).

## Freshness

Re-verified against primary sources (npm registry, official docs, W3C, `web-features`
Baseline v3.36.0). Other rows are prior snapshots, not a claim that all packages/standards were re-executed on that date:

| Package / standard | Verified |
|---|---|
| `next` | **16.3.6** — Turbopack stable, `middleware.ts` → `proxy.ts`, `next lint` removed (`next build` no longer lints), `cacheComponents`, Node.js **20.9+**, TypeScript 5.1+ minimum |
| `react` / `react-dom` | **19.3.0** |
| `babel-plugin-react-compiler` | **1.0.0** — `reactCompiler` promoted to a stable top-level flag, **not enabled by default** |
| `tailwindcss` | **4.3.3** — CSS-first `@theme`, no JS config |
| `@tanstack/react-query` | **5.102.8** · `zustand` **5.0.15** · `nuqs` **2.10.1** |
| `vitest` **4.1.11** · `@playwright/test` **1.62.1** · `msw` **2.15.0** · `axe-core` **4.13.0** | |
| `eslint` | **10.11.0** — flat config only; `.eslintrc` support removed |
| `typescript` | **7.0.2** — see the note below |
| WCAG 2.2 | **ISO/IEC 40500:2025** (per W3C WAI). That ISO edition corresponds to the **October 2023** text of WCAG 2.2; the December 2024 text is expected as ISO/IEC 40500:2026 |
| Trusted Types | Baseline **Newly available, 2026-02-24** |
| AVIF | Baseline **Widely available, 2026-07-25** |
| `scheduler.yield()` | **Not Baseline** — Chrome and Firefox only; feature-detect ([performance](./04-quality/performance.md)) |

**TypeScript 7 is out and this guide's defaults do not assume it.** TS 7 is the native port of
the compiler (order-of-magnitude faster `tsc`), and Next.js 16's stated minimum is only TS
5.1. Treat TS 7 as an opt-in performance upgrade, not the baseline: verify your
`tsconfig.json`, ESLint/typescript-eslint versions, and editor tooling support it before
adopting, and record its actual approval/reason in Planner Output (DEC-n) and the software TODO. The rules in
[code-quality](./04-quality/code-quality.md) (`strict`, `moduleResolution: "bundler"`) are
unchanged either way.

Re-verify framework majors before each project start — this table ages.
