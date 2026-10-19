# Testing

Test-runner choice and usage on modern Node. Cross-ref: [typescript.md](./typescript.md) (running TS tests without a build step).

## Runner selection

| Runner | Pick when | Notes |
|---|---|---|
| `node:test` | zero-dependency libs, small services, CI-friendliness | built-in, Stable; runs `.ts` directly on ≥ 22.18 |
| **Vitest** | apps, TS projects, component/integration suites | fastest-moving ecosystem; Jest-compatible API; mock/spy/snapshot/coverage built in |
| Jest | existing brownfield suites | still maintained; do not start new projects on it |
| Playwright/Supertest | E2E / HTTP-level tests | framework-agnostic |

Consensus 2026: Vitest for modern projects, Jest kept for legacy, `node:test` when dependencies must stay zero. **In this project the runner is Vitest** (THE CHECK runs `vitest run`: backend-guide/01-foundation/initialization.md); the `node:test` sections below are general knowledge, never for the project's suites. Sources: [vitest.dev/guide/comparisons](https://vitest.dev/guide/comparisons), community surveys 2026 (pkgpulse, testdino).

## `node:test` essentials [Stable]

```js
// math.test.ts — general node:test knowledge; this project's tests are Vitest, in tests/<tester>/
import { test, describe, beforeEach, mock } from 'node:test';
import assert from 'node:assert/strict';

describe('math', () => {
  let calc;
  beforeEach(() => { calc = createCalc(); });

  test('adds', () => {
    assert.equal(calc.add(1, 2), 3);
  });

  test('async with subtests', async (t) => {
    const fn = mock.fn();
    await t.test('subtest auto-awaited', async () => {   // Node ≥ 24: no manual await needed
      await Promise.resolve(); fn();
      assert.equal(fn.mock.callCount(), 1);
    });
  });
});
```
```bash
node --test                         # see the discovery table below
node --test --test-reporter=dot
node --test --experimental-test-coverage
node --test "**/*.spec.ts"          # explicit glob when your names don't match the defaults
```

**Default discovery is narrower than Jest/Vitest — verified by running it:**

| Filename | Picked up by bare `node --test`? |
|---|---|
| `a.test.js` | ✅ |
| `c-test.js` / `d_test.js` / `test-e.js` / `test.js` | ✅ |
| any file inside a `test/` directory | ✅ |
| **`b.spec.js`** | ❌ |
| `f.tests.js` | ❌ |
| `sub/h.js` (not under `test/`) | ❌ |

> **This is the dangerous one.** A team migrating from Jest/Vitest keeps its `*.spec.ts`
> names, runs `node --test`, and gets `# pass 0 / # fail 0` — a **green CI that ran
> nothing**. Either rename to `*.test.ts` or pass an explicit glob. Assert a minimum test
> count in CI if you want a hard guard.
Key behaviors: subtest promises are awaited automatically since Node 24 ([v24 release notes](https://nodejs.org/en/blog/release/v24.0.0)); `assert/strict` should be the default import; snapshot files belong in source control (official docs guidance).

## Rules

**R1 — One runner per project; do not mix. [Stable]**
Mixed runners double config, CI time, and mock semantics confusion. Migrating Jest→Vitest is deliberately low-friction (API-compatible) — do it wholesale, not incrementally.

**R2 — Tests import the same module format the app ships. [Stable]**
ESM app → ESM tests (`import`). Thanks to `require(esm)` ([module-system.md](./module-system.md) R2) and native TS ([typescript.md](./typescript.md) R1), the "test files must be CJS" era is over — remove those `babel-register`/`ts-jest` shims when modernizing.

**R3 — Mock external boundaries, not internals. [Stable]**
Mock network (`mock.method(globalThis, 'fetch')` in node:test; `vi.spyOn`/`msw` in Vitest)
and time (`mock.timers`); don't mock modules you own — refactor for testability instead.
Excessive internal mocking produces tests that verify implementation details, not behavior.

> **`mock.module()` is still flag-gated.** Verified: `require('node:test').mock.module` is
> `undefined` without `--experimental-test-module-mocks`, and a function with it. If your
> design needs module mocking to be testable, that is usually a signal to inject the
> dependency instead — or to use Vitest, where `vi.mock` is a first-class feature.

**R4 — Deterministic tests: control time and randomness. [Stable]**
`node:test` `mock.timers.enable({ now })` / Vitest `vi.useFakeTimers()`. Never assert on real sleep durations. Seed anything random. Flaky tests erode trust faster than no tests.

**R5 — HTTP endpoint tests: drive the real server on a random port. [Stable]**
```js
const server = app.listen(0);                     // port 0 = OS-assigned, parallel-safe
const res = await fetch(`http://127.0.0.1:${server.address().port}/health`);
assert.equal(res.status, 200);
```
Works with every framework (fastify.inject is a faster in-process alternative). Avoid mocking `res`/`req` objects by hand.

**R6 — CI ordering: lint → types (`tsc --noEmit`) → unit → integration. [Stable]**
Types are a test suite; fail them before runtime tests for fast feedback. Keep unit tests under seconds; move slow integration/E2E to a later stage or separate job.
