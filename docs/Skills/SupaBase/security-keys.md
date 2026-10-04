# Security & API Keys

> Source review: 2026-09-27. This is a template reference, not a running project.
> Historical execution/measurement statements below are author reports without a supplied
> reproduction harness; they are not acceptance evidence. This review ran no Supabase
> CLI, SQL, deployed function, or real provider integration.

## CRITICAL: 2025-2026 API Key Migration

Supabase is transitioning from legacy JWT-based keys (anon, service_role) to a new model:

| Legacy Key | Replacement | Scope |
|------------|-------------|-------|
| `anon` (public, JWT) | **Publishable key** (`sb_publishable_...`) | Low-privilege client access |
| `service_role` (secret, JWT) | **Secret key** (`sb_secret_...`) | Full-privilege server access |

- New keys are opaque strings, not JWTs. Both models work simultaneously; creating new keys does not affect legacy ones.
- **Deadline**: legacy `anon`/`service_role` keys are deprecated **by the end of 2026** — migrate now.
- New keys can be created/rotated independently per backend component (one secret key per service is recommended).
- Secret keys refuse to run in browsers (User-Agent check → HTTP 401).
- **Function authentication**: a user JWT in Authorization is distinct from an opaque API
  key. For user endpoints retain JWT verification and use the wrapper's user auth mode.
  For secret-key calls disable the platform JWT check only with explicit in-function
  secret-key validation (for example `withSupabase` secret mode). Passing a platform
  check alone does not establish identity or authorization.

Source: https://supabase.com/docs/guides/getting-started/api-keys

## Key Rules

### Rule 1: Never expose the secret/service_role key to clients

**Why**: It bypasses RLS, but ordinary SQL grants still apply. Treat it as privileged server access; never infer permission to perform a business operation from possessing the client.

**How it leaks**: Committed to Git, embedded in client bundles, put in `NEXT_PUBLIC_` env vars.

**Fix**: Store in server-side environment only. If leaked, rotate immediately in Dashboard → scrub Git history.

### Rule 2: The publishable/anon key is safe to expose — ONLY with RLS enabled

**Why**: It is designed to be public. Without RLS, operations granted to its database role are unrestricted by row policies. SQL grants and RLS are separate controls.

**Enforcement**: Enable RLS on every table in an exposed schema. Revoke default grants from anon/authenticated roles if needed.

### Rule 3: Two Supabase clients — both in Express

In this project the browser and Next.js have no Supabase client at all. Both live in Express
(backend-guide/03-auth/authentication.md §0.1):

```typescript
import { createClient } from '@supabase/supabase-js'
const serverAuth = { persistSession: false, autoRefreshToken: false, detectSessionInUrl: false }

// USER client — per request, publishable key + that user's token: RLS applies
export const supabaseAsUser = (accessToken: string) =>
  createClient(SUPABASE_URL, SUPABASE_PUBLISHABLE_KEY, {
    auth: serverAuth,
    global: { headers: { Authorization: `Bearer ${accessToken}` } },
  })

// ADMIN client — secret key: bypasses RLS. Only for work no user owns
export const supabaseAdmin = createClient(SUPABASE_URL, SUPABASE_SECRET_KEY, { auth: serverAuth })
```

### Rule 4: Environment variable hygiene

- **Every Supabase variable lives in Express's environment only:** `SUPABASE_URL`,
  `SUPABASE_PUBLISHABLE_KEY`, `SUPABASE_SECRET_KEY`. Not even the publishable key goes into a
  `NEXT_PUBLIC_*` variable: the frontend never talks to Supabase.
- All secrets go in `.env` files that are never committed. Document required vars in `.env.example`.

## RLS as Security Foundation

RLS is not optional. A table in an exposed schema without RLS is readable and writable by any role with a grant on it.

**Minimum for every table**:
```sql
ALTER TABLE your_table ENABLE ROW LEVEL SECURITY;
```

Then add policies. See [rls-patterns.md](./rls-patterns.md) for policy templates that still need integration verification.

## 2025 Security Changes Summary

From the official security retro:
- New API key model (publishable/secret) replacing JWT-based anon/service_role
- Keys can now be scoped and rotated independently
- Improved default security for new projects
- Breaking: OpenAPI spec access removed from anon key
- Database webhooks now support authorization

Source: https://supabase.com/docs/guides/getting-started/api-keys

## Quick Checklist

- [ ] RLS enabled on every table in public schema
- [ ] Secret/service_role key only in server-side code
- [ ] `.env*` files in `.gitignore` (except `.env.example`)
- [ ] `.env.example` documents all required vars (no real values)
- [ ] No Supabase variable in the frontend at all — only in Express's environment
- [ ] Tested access with a real user JWT, not just service_role
- [ ] Key rotation plan documented (Dashboard → Settings → API Keys)
