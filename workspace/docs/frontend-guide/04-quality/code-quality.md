Code quality is enforced automatically, not relied upon as a discipline. Linting and type checking run in THE CHECK (`npm run check`, 01-foundation/initialization.md), which the orchestrator runs on every step and which gates all work. This file covers the tooling configuration and conventions that keep the codebase consistent as it grows.

Scope: follow [frontend-only contract authority](../00-navigation/frontend-only-scope.md). Endpoint names, payloads and session settings in examples apply only when the actual owner-supplied contract matches; examples do not define external API behavior.

## ESLint

Use flat config (`eslint.config.mjs` — what create-next-app generates; legacy `.eslintrc` is deprecated and ESLint v10 drops it). `@next/eslint-plugin-next` is flat-config-first. Run ESLint via its own CLI — the `next lint` command was **removed in Next.js 16**, and `next build` no longer lints automatically, so linting must be an explicit script and CI gate.

Extend from `eslint-config-next` (which includes the core-web-vitals rules) plus `typescript-eslint`. Add project-specific rules sparingly; every rule you add is a rule the team must learn and every CI run must check. Avoid rules that overlap with formatting (Prettier's job) or type correctness (TypeScript's job).

Biome is a legitimate alternative — create-next-app offers it at setup time, and it unifies linting + formatting in one fast Rust tool with a smaller plugin ecosystem. Pick one toolchain per project; do not run both.

## TypeScript

Strict mode is non-negotiable in 2026. Enable it in `tsconfig.json`:

```json
{
  "compilerOptions": {
    "strict": true,
    "moduleResolution": "bundler",
    "skipLibCheck": true
  }
}
```

`moduleResolution: "bundler"` matches how Next.js and Vite resolve modules. `skipLibCheck: true` skips type-checking in `node_modules` — it speeds up builds and avoids errors in third-party types you can't fix.

## Prettier

Use Prettier for formatting when you chose the ESLint toolchain. No formatting debates in code review. The config should be minimal — Prettier's defaults are good. Override only line width (100 or 120, pick one and commit to it) and whether to use trailing commas (`"all"`). If you chose Biome instead, its formatter replaces Prettier entirely.

## Type Safety Rules

Strict mode catches implicit any and nullability errors, but does NOT forbid explicit any or unsafe response.json() values. Enable @typescript-eslint/no-explicit-any and the type-aware no-unsafe-* rules. Runtime schemas validate external JSON; type assertions do not. The conventions below also require lint/review enforcement where applicable:

- **No `any`**: Use `unknown` for truly unknown types (e.g., API response bodies before validation). Narrow `unknown` with type guards before use.
- **Discriminated unions** for state modeling: `{ status: 'loading' } | { status: 'success'; data: T } | { status: 'error'; error: Error }`. This eliminates impossible states at the type level (see [state-and-data.md](../03-data/state-and-data.md)).
- **`satisfies`** for type-checking without widening. Use it when you want to verify an object matches a type but preserve its literal types.
- **`as const`** for literal types. Use it for configuration objects, route maps, and any value where the literal string/number matters.

## Naming Conventions

- **PascalCase**: Components (`UserAvatar`), types/interfaces (`UserProfile`), enum members.
- **camelCase**: Functions (`fetchUsers`), variables (`userCount`), hooks (`useAuth`).
- **UPPER_SNAKE_CASE**: Constants that are truly constant across the application (`MAX_RETRIES`, `API_BASE_URL`).
- **kebab-case**: File names (`user-avatar.tsx`, `use-auth.ts`). This avoids case-sensitivity issues across operating systems.

## Import Conventions

- **Path aliases**: Use `@/` mapped to `src/` (configured in `tsconfig.json` paths and Next.js/Vite config). No relative imports that traverse more than one level.
- **No barrel files**: Do not create `index.ts` files that re-export everything from a directory. They cause tree-shaking failures and make dependency graphs opaque. Import directly from the source file: `import { Button } from '@/components/ui/button'`, not `import { Button } from '@/components/ui'`.
- **Import order**: React imports first, then external packages, then internal aliases, then relative imports, then styles. Enforce this with `eslint-plugin-import` or the import sorting rule in your flat config.

## Conventions to Enforce

- No unused variables or imports. TypeScript's `noUnusedLocals` and `noUnusedParameters` catch these.
- No `useEffect` with missing dependencies. The exhaustive-deps ESLint rule catches this.
- Explicit return types for all exported functions. This makes refactoring safer and serves as documentation.
- Consistent component exports: use named exports for components, not default exports. This improves grep-ability and refactoring tooling.

## CI Checks

Every pull request must pass these checks before merge (wired into the pipeline described in [delivery-and-observability.md](../05-delivery/delivery-and-observability.md)):

1. **ESLint**: `eslint . --max-warnings 0`. Zero warnings allowed — warnings rot into ignored noise.
2. **TypeScript**: `tsc --noEmit`. Catches type errors without producing output.
3. **Vitest**: Full test suite (see [testing.md](./testing.md)).
4. **Accessibility**: `jest-axe` assertions inside component tests
   ([testing.md](./testing.md)); `@axe-core/playwright` against the built app only if the user
   asks for automated E2E.
   **`@axe-core/react` is a different kind of tool** and does not belong in this gate: it is a
   development-time reporter that logs violations to the browser console as your app runs. It
   asserts nothing and fails no build. Use it while developing if you like; do not list it as a
   CI check.

## Pre-commit Hooks

Husky + lint-staged runs ESLint and Prettier on staged files before commit. This is optional but recommended — it catches issues early and keeps CI fast. Configure lint-staged to run only on changed files, not the entire project.

## Production Logging

No `console.log` in production.

```typescript
// This NO-OPS the call. It does not remove it.
const log = process.env.NODE_ENV === 'development' ? console.log : () => {};
```

> The pattern above is worth understanding precisely, because it is usually described as
> "stripping logs" and it does not strip anything. The call sites, the argument expressions, and
> every string literal you passed still ship in the bundle — an empty function just swallows
> them at runtime. Anything sensitive you logged is still readable in the source, and the
> argument expressions still execute.

**To actually remove them**, do it at build time:

- **Next.js:** `compiler: { removeConsole: { exclude: ['error', 'warn'] } }` in
  `next.config.ts` — the compiler deletes the calls from the production output.
- **ESLint:** `no-console` (allowing `warn`/`error`) so they never reach `main`.

Never log secrets, tokens, or personal data even in development — development logs end up in
screenshots, CI output, and shared terminals ([delivery-and-observability](../05-delivery/delivery-and-observability.md)).

## Decision Summary

| Concern | Tool/Rule |
|---|---|
| Linting | ESLint flat config, `next/core-web-vitals` |
| Formatting | Prettier (or Biome) |
| Type checking | `strict: true`, `moduleResolution: "bundler"` |
| No `any` | no-explicit-any + type-aware no-unsafe-* lint rules; strict alone is insufficient |
| State types | Discriminated unions |
| File names | kebab-case |
| Imports | `@/` aliases, no barrel files, ordered |
| CI gates | ESLint, TypeScript, Vitest, axe-core |
| Pre-commit | Husky + lint-staged (recommended) |
| Logging | Strip in production, no raw `console.log` |