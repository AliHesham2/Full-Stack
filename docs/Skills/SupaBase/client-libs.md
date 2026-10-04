# Client Libraries

> Project contract: Browser -> Next.js -> Express on Node.js -> Supabase.
> SDKs, credentials, authorization and provider integration live in Express only.

> Source review: 2026-09-27. This is a template reference, not a running project.
> Historical execution/measurement statements below are author reports without a supplied
> reproduction harness; they are not acceptance evidence. This review ran no Supabase
> CLI, SQL, deployed function, or real provider integration.

## supabase-js (v2)

Isomorphic JavaScript/TypeScript client. Works in browsers, Node.js, and edge runtimes.

Current checked target: **2.117.2** (2026-09-26; Node >=22). Historical examples below used 2.113.0; rerun integration checks on upgrade. v1 is archived.

**The sub-packages are version-locked, not ranged.** `supabase-js@2.113.0` declares
*exact* dependencies:

```
@supabase/auth-js       2.113.0
@supabase/postgrest-js  2.113.0
@supabase/realtime-js   2.113.0
@supabase/storage-js    2.113.0
@supabase/functions-js  2.113.0
```

Consequences:
- Never add `@supabase/postgrest-js` (or any sibling) to your own `package.json` — a
  different version installs a **second copy**, and the two clients don't share auth state.
- Upgrade `@supabase/supabase-js` alone; the rest follow.
- An older sibling can legitimately belong to another dependency. Inspect the dependency
  graph before deduplicating; do not force incompatible overrides or assume corruption.

### Dead packages — reject on sight

| Package | Status | Use instead |
|---|---|---|
| `@supabase/auth-helpers-nextjs` | **Formally deprecated on npm**: *"Package no longer supported."* Last publish 0.15.0, Nov 2025 | not used: this project has no Supabase in Next.js |
| `@supabase/auth-helpers-react` / `-sveltekit` / `-remix` | same | not used |
| `@supabase/supabase-js@1` | archived | v2 |

Most Supabase + Next.js tutorials online wire Supabase into the frontend (`auth-helpers` or
`@supabase/ssr`). **This project never does that**: only Express uses Supabase
(backend-guide/03-auth/authentication.md §0.1).

Source: https://www.npmjs.com/package/@supabase/supabase-js

## Installation

```bash
cd backend && npm install @supabase/supabase-js@2.117.2   # source-checked target — in backend/ ONLY
```

`@supabase/ssr` is not installed anywhere: it exists to put Supabase into a frontend framework,
and the frontend's ESLint rule blocks every `@supabase/*` import.

## TypeScript Support

Generate types from your schema:

```bash
supabase gen types typescript --local > src/types/database.ts
# Use --linked only after explicitly checking the intended remote project.
```

Use them:

```typescript
import { createClient } from '@supabase/supabase-js'
import type { Database } from '@/types/database'

const supabase = createClient<Database>(url, key)

// Now fully typed:
const { data } = await supabase.from('todos').select('*').eq('id', 1)
// data is typed as { id: number, title: string, ... }[]
```

Source: https://supabase.com/docs/reference/javascript/typescript-support

## Client vs Admin Client

Both clients live in **Express** (backend-guide/03-auth/authentication.md §0.1):

```typescript
const serverAuth = { persistSession: false, autoRefreshToken: false, detectSessionInUrl: false }

// USER client — per request, as the logged-in user: RLS applies. The default for user data
const supabase = createClient(url, publishableKey, {
  auth: serverAuth,
  global: { headers: { Authorization: `Bearer ${userAccessToken}` } },
})

// ADMIN client — secret key: bypasses RLS. Only for work no user owns (jobs, admin tasks)
const supabaseAdmin = createClient(url, secretKey, { auth: serverAuth })
```

## Key API Patterns

### Query builder (PostgREST)

```typescript
// Select with filter, order, pagination
const { data, error } = await supabase
  .from('posts')
  .select('*, author:users(name, avatar_url)')
  .eq('published', true)
  .order('created_at', { ascending: false })
  .range(0, 9)
```

### Realtime subscription

```typescript
// Express only: REST Authorization headers do not set the WebSocket user token.
await supabase.realtime.setAuth(userAccessToken)
supabase
  .channel('changes')
  .on('postgres_changes', { event: 'INSERT', schema: 'public', table: 'posts' }, (payload) => {
    console.log('New post:', payload.new)
  })
  .subscribe()
```

### Storage

These are independent Express snippets. Provider URLs remain internal; Express streams
files through the Next.js relay instead of returning or redirecting to Supabase URLs.

```typescript
// Upload
const { data, error } = await supabase.storage
  .from('avatars')
  .upload('user-123/photo.jpg', file)

// Public URL
const { data } = supabase.storage
  .from('avatars')
  .getPublicUrl('user-123/photo.jpg')
```

### Edge Function invocation

```typescript
const { data, error } = await supabase.functions.invoke('hello', {
  body: { name: 'World' }
})
```

## Official Client Libraries

| Library | Status |
|---------|--------|
| `@supabase/supabase-js` | Official, active |
| `@supabase/ssr` | Official, for SSR frameworks — not used in this project |
| `supabase-py` (Python) | Official |
| `supabase-swift` | Official |
| `supabase-kt` (Kotlin) | Official |
| `supabase-flutter` (Dart) | Official |
| Others | Community-contributed |

Source: https://supabase.com/docs/guides/api/rest/client-libs

## `@supabase/ssr` — not used in this project

`createBrowserClient` / `createServerClient` put Supabase into a frontend framework. This project
keeps Supabase entirely inside Express, so they are never used; see [auth.md](./auth.md) for the
Express setup.