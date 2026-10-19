# Project Structure

> **In this project the layout is backend-guide/01-foundation/project-structure.md:** modules by
> domain (`src/modules/<domain>/…`), not folders by layer. The approved software TODO specifies each
> path from that guide. The layer-based layout below is general Express knowledge; its app
> factory, middleware order and error handler still apply inside the guide's layout.

## Recommended Layout (TypeScript)

```
my-express-app/
  src/
    index.ts              # Entry point: creates app, listens
    app.ts                # Express app factory (exported for testing)
    config/
      env.ts              # Validated env vars (dotenv + zod)
      db.ts               # Database connection
    routes/
      users.ts             # User router
      auth.ts              # Auth router
      health.ts            # Health check routes
    middleware/
      auth.ts              # Auth middleware
      validate.ts          # Zod validation wrapper
      errorHandler.ts      # Centralized error handler
    services/
      user.service.ts      # Business logic
    repositories/
      user.repository.ts   # Data access
    types/
      express.d.ts         # Augment Express types
      errors.ts            # Custom error classes
  tests/
    tester-a/            # one folder per tester, each mirroring src/
    tester-b/            #   (backend-guide/01-foundation/project-structure.md)
  package.json
  tsconfig.json
  vitest.config.ts
  .env.example
  .gitignore
```

## Entry Point Pattern

```ts
// src/app.ts — export for supertest
import express from 'express';
import helmet from 'helmet';
import { errorHandler, notFound } from './middleware/errorHandler.js';
import userRoutes from './routes/users.js';
import healthRoutes from './routes/health.js';

export function createApp() {
  const app = express();
  app.disable('x-powered-by');
  app.use(helmet());
  // no cors(): only the Next.js server calls this API
  app.use(express.json({ limit: '10kb' }));
  app.use('/health', healthRoutes);
  app.use('/api/users', userRoutes);
  app.use(notFound);
  app.use(errorHandler);
  return app;
}
```

```ts
// src/index.ts — only starts server
import { createApp } from './app.js';
import { config } from './config/env.js';

const app = createApp();
app.listen(config.port, () => {
  console.log(`Listening on :${config.port}`);
});
```

**Why separate `app.ts` and `index.ts`**: Allows importing `app` in tests without starting the server or listening on a port.

## Environment Config

```ts
// src/config/env.ts
import 'dotenv/config';
import { z } from 'zod';

// Zod 4.x — string formats are top-level (`z.url()`, not `z.string().url()`)
const envSchema = z.object({
  NODE_ENV: z.enum(['development', 'production', 'test']).default('development'),
  PORT: z.coerce.number().int().default(3000),
  DATABASE_URL: z.url(),
  SESSION_SECRET: z.string().min(32),
  LOG_LEVEL: z.enum(['fatal', 'error', 'warn', 'info', 'debug', 'trace']).default('info'),
});

export const config = envSchema.parse(process.env);
```

Fails fast on startup if env vars are missing or invalid.

> Node 20.6+ can load env files natively (`node --env-file=.env`), so `dotenv` is optional — but keep the zod schema either way. Loading is not validating.

## Type Augmentation

```ts
// src/types/express.d.ts
import 'express';

declare global {
  namespace Express {
    interface Request {
      user?: {
        id: string;
        role: string;
      };
      validated?: Record<string, unknown>;   // filled by validate() — see security.md
    }
  }
}
```

## Typing handlers (TypeScript)

`RequestHandler` takes four generics in a non-obvious order:

```ts
import type { RequestHandler } from 'express';
//                       Params      ResBody        ReqBody      Query
type CreateUser = RequestHandler<{ id: string }, UserResponse, CreateUserBody, { page?: string }>;

export const createUser: CreateUser = async (req, res) => {
  req.params.id;      // string
  req.body.email;     // typed
  res.json(user);     // must match UserResponse
};
```

Derive the body type from the zod schema rather than declaring it twice:

```ts
const createUserSchema = z.object({ name: z.string(), email: z.email() });
type CreateUserBody = z.infer<typeof createUserSchema>;
```

**Gotchas**:
- An `async` handler returns `Promise<void>`, which does not satisfy some
  `RequestHandler` overloads if you `return res.json(...)`. Write `res.json(...); return;`
  or type the handler explicitly.
- `@types/express@5.x` is required — the 4.x types describe different `req.params`
  semantics → [overview.md](./overview.md).
- Error handlers are `ErrorRequestHandler`; the 4-argument arity is what Express keys on
  at runtime, so never let a linter "clean up" an unused `next`.

## package.json Scripts

```json
{
  "engines": { "node": ">=22" },
  "scripts": {
    "dev": "tsx watch src/index.ts",
    "build": "tsc",
    "start": "node dist/index.js",
    "test": "vitest run",
    "test:watch": "vitest",
    "lint": "eslint src/",
    "audit": "npm audit --audit-level=high"
  }
}
```

> **`tsx` vs native**: Node 22.18+/24 strips TypeScript types natively, so `node --watch src/index.ts` works without a loader — but only for *erasable* syntax. `enum`, `namespace`, parameter properties, and path aliases still need `tsx`. Keep `tsx` unless you have verified the codebase is erasable-only.

## Anti-patterns

1. **Everything in one file** — split into routes, services, repositories
2. **Requiring the app to listen for tests** — export the app factory
3. **Config scattered across files** — centralize in `config/env.ts`
4. **No `.env.example`** — new devs don't know what env vars to set
5. **Using `require()` in a TypeScript project** — use ESM imports throughout

Test directory names in this example are placeholders. Roster and the authorized brief select actual Tester operators, count and isolated write scopes; they do not require a second operator. Follow the shared delivery cycle for Orchestrator's personal verification of corrections, without automatic Tester redispatch.
