# Testing

## Recommended Stack (2026)

| Tool | Version (Sept 2026) | Role |
|------|---------------------|------|
| Vitest | 4.x | Test runner — native ESM/TS, fast watch mode |
| supertest | 7.x | HTTP assertions against the app without binding a port |
| msw | 2.x | Mock external HTTP services |

**Vitest 4 requires Node ^20 \|\| ^22 \|\| >=24.** Pair it with the Node version from [overview.md](./overview.md) — Node 24.

> Node's built-in `node:test` + `node --test` is a legitimate zero-dependency alternative and needs no config. Choose it when you want no test-runner dependency at all; choose Vitest when you want watch mode, coverage config, and `vi.mock`. **In this project the runner is Vitest:** THE CHECK runs `vitest run` (backend-guide/01-foundation/initialization.md), so a `node:test` test would never run.

## Setup

```ts
// vitest.config.ts
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    environment: 'node',
    globals: true,          // optional — omit it and import { describe, it, expect } from 'vitest'
    coverage: { provider: 'v8' }, // inspect gaps; no arbitrary global percentage gate
  },
});
```

## Testing Routes with Supertest

```ts
import { describe, it, expect } from 'vitest';
import request from 'supertest';
import { createApp } from '../../../../src/app.js';  // from tests/<tester>/modules/<domain>/ — adjust to the test's depth

const app = createApp();  // factory pattern (see project-structure.md) — no port bound

describe('POST /users', () => {
  it('creates a user with valid data', async () => {
    const res = await request(app)
      .post('/users')
      .send({ name: 'Alice', email: 'alice@example.com' })
      .expect(201)
      .expect('Content-Type', /json/);

    expect(res.body.data).toHaveProperty('id');
    expect(res.body.data.name).toBe('Alice');
  });

  it('rejects invalid email', async () => {
    const res = await request(app)
      .post('/users')
      .send({ name: 'Alice', email: 'not-email' })
      .expect(400);

    expect(res.body.error.code).toBe('VALIDATION_ERROR');
    expect(res.body.error.details).toHaveProperty('email');
  });
});
```

## Testing Middleware in Isolation

```ts
import { describe, it, expect, vi } from 'vitest';
import type { Request, Response, NextFunction } from 'express';

function createMocks() {
  const req = { headers: {}, log: { info: vi.fn(), error: vi.fn() } } as unknown as Request;
  const res = { status: vi.fn().mockReturnThis(), json: vi.fn().mockReturnThis() } as unknown as Response;
  const next = vi.fn();
  return { req, res, next };
}

describe('requireAuth middleware', () => {
  it('calls next() when token is valid', async () => {
    const { req, res, next } = createMocks();
    req.headers.authorization = 'Bearer valid-token';
    await requireAuth(req, res, next);
    expect(next).toHaveBeenCalledWith();
  });

  it('forwards a 401 error when no token', async () => {
    const { req, res, next } = createMocks();
    await requireAuth(req, res, next);
    expect(next).toHaveBeenCalledWith(expect.objectContaining({
      statusCode: 401, code: 'UNAUTHENTICATED',
    }));
    expect(res.status).not.toHaveBeenCalled(); // central error handler owns the response
  });
});
```

## Test File Organization

One folder per tester, outside `src/`, each mirroring the modules and endpoints it proves; each
keeps its own fixtures, and the two suites share nothing
(backend-guide/01-foundation/project-structure.md):

```
tests/
  tester-a/
    modules/
      users/
        users.routes.test.ts      # the HTTP contract, through supertest
        users.use-cases.test.ts
      auth/
        auth.routes.test.ts
    shared/
      http/
        validate.test.ts          # edge helpers (src/shared/http/)
    fixtures/
      users.json                  # this tester's own, never shared
  tester-b/                       # only if a second Tester is actually assigned
```

## Key Rules

1. **Don't bind to a port in tests** — supertest handles this via `request(app)`
2. **Use a test database** — never test against production or shared dev databases
3. **Only bind a port when you genuinely need one** (e.g. testing graceful shutdown) — then pass port `0` and read `server.address().port`, so parallel runs never collide
4. **Isolate middleware tests** — don't need a full app to test a middleware function
5. **Mock external services with msw** — don't make real HTTP calls in tests
6. **Test error paths** — not just happy path: 400, 401, 403, 404, 422, 500

## Vitest vs Jest for Express

Vitest is recommended for new projects: same API as Jest but native ESM, dramatically faster, and better watch mode. Migration from Jest is straightforward — [migration guide](https://vitest.dev/guide/comparisons).

Source: [Vitest comparisons](https://vitest.dev/guide/comparisons), [Node.js test runner](https://nodejs.org/api/test.html)

Test directory names in this example are placeholders. Roster and the authorized brief select actual Tester operators, count and isolated write scopes; they do not require a second operator. Follow the shared delivery cycle for Orchestrator's personal verification of corrections, without automatic Tester redispatch.
