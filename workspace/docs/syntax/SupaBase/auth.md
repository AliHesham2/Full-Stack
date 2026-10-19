# Authentication

> Source review: 2026-09-27. This is a template reference, not a running project.
> Historical execution/measurement statements below are author reports without a supplied
> reproduction harness; they are not acceptance evidence. This review ran no Supabase
> CLI, SQL, deployed function, or real provider integration.

## How It Works

Supabase Auth (GoTrue) manages users and issues JWTs. The JWT contains claims including `sub` (user ID) and `role`. RLS policies consume these via `auth.uid()` and `auth.jwt()`.

Source: https://supabase.com/docs/guides/auth

## Supported Methods

| Method | Notes |
|--------|-------|
| Email/password | Standard. Enable email confirmation in Dashboard. |
| Magic link (OTP) | Passwordless. Link sent to email; verify via `verifyOtp()` (token hash) in SSR flows too. |
| OAuth providers | Google, GitHub, Apple, Facebook, etc. Configure in Dashboard → Auth → Providers. |
| Phone (OTP) | Requires an SMS provider: Twilio / Twilio Verify / Vonage / MessageBird / Textlocal (community-supported). WhatsApp requires Twilio or Twilio Verify. |
| Anonymous | Temporary users, can convert to permanent. |
| SAML SSO | Available on Pro and above; confirm current SSO usage charges on the pricing page. |
| Third-party auth | Auth0, Firebase, Cognito JWTs work with Supabase Data APIs. |
| PKCE flow | For mobile apps and custom auth flows. |

Source: https://supabase.com/docs/guides/auth/phone-login

## JWT Structure

- Tokens are short-lived. Supabase Auth continuously issues new JWTs (refresh token rotation).
- Default expiry: 1 hour access token, configurable.
- Custom claims are added via the **Custom Access Token Hook** — there is no `auth.jwts_claims` table.

```sql
-- Custom Access Token Hook: adds claims to every issued JWT
create or replace function public.custom_access_token_hook(event jsonb)
returns jsonb
language plpgsql
stable
security invoker
set search_path = ''
as $$
declare
  claims jsonb;
  user_role text;
begin
  select role into user_role from public.profiles where id = (event->>'user_id')::uuid;

  claims := coalesce(event->'claims', '{}'::jsonb);
  if user_role is not null then
    claims := jsonb_set(claims, '{user_role}', to_jsonb(user_role));
  end if;
  event := jsonb_set(event, '{claims}', claims);   -- path must be text[]: '{claims}', not 'claims'

  return event;
end;
$$;

-- Assumes public.profiles(id uuid, role text); enable RLS in its schema migration.
grant usage on schema public to supabase_auth_admin;
grant select (id, role) on public.profiles to supabase_auth_admin;
create policy auth_hook_read_role on public.profiles
  for select to supabase_auth_admin using (true);
grant execute on function public.custom_access_token_hook(jsonb) to supabase_auth_admin;
revoke execute on function public.custom_access_token_hook(jsonb) from authenticated, anon, public;
```

> **`jsonb_set` takes a `text[]` path.** Writing `jsonb_set(event, 'claims', claims)` — as
> many copies of this snippet do — fails at runtime with
> `ERROR: malformed array literal: "claims"`, and because the hook runs on **every token
> issue**, that breaks sign-in entirely. Verified on PostgreSQL 18: the braced form
> `'{claims}'` returns
> `{"claims":{"sub":"x","user_role":"admin"},"user_id":"…"}`; the unbraced form errors.
> Test the function with a literal `select` before registering it.

```sql
-- Smoke-test before wiring it up
select public.custom_access_token_hook(
  '{"user_id":"<a real uuid>","claims":{"sub":"x"}}'::jsonb
);
```

Register it under **Dashboard → Authentication → Hooks → Custom Access Token** (available on Free and Pro). Read claims in RLS via `auth.jwt()->'user_role'`. Legacy alternative: triggers on `auth.users` updating `raw_app_meta_data` (merged into the JWT).

Source: https://supabase.com/docs/guides/auth/auth-hooks/custom-access-token-hook

## Server-Side Setup (Express) — the only place this project uses Supabase Auth

In this project **only Express talks to Supabase**, auth included. There is no Supabase client in
the browser and none in Next.js (no `@supabase/ssr`, no `NEXT_PUBLIC_SUPABASE_*` variable). Express
verifies the credentials, keeps the Supabase tokens in its own session record, and gives the
frontend only an opaque session id. The full design (keys, the two clients, login, every request,
refresh, logout): backend-guide/03-auth/authentication.md §0 and §0.1.

```typescript
// backend — a client for ONE request. A server has no browser storage, and Express refreshes
// tokens itself, per session
import { createClient } from '@supabase/supabase-js'

const serverAuth = { persistSession: false, autoRefreshToken: false, detectSessionInUrl: false }

export function supabaseForLogin() {
  return createClient(process.env.SUPABASE_URL!, process.env.SUPABASE_PUBLISHABLE_KEY!, {
    auth: serverAuth,
  })
}

export function supabaseAsUser(accessToken: string) {   // RLS applies as this user
  return createClient(process.env.SUPABASE_URL!, process.env.SUPABASE_PUBLISHABLE_KEY!, {
    auth: serverAuth,
    global: { headers: { Authorization: `Bearer ${accessToken}` } },
  })
}

// login (POST /auth/login): verify, then store the tokens in Express's own session record
const { data, error } = await supabaseForLogin().auth.signInWithPassword({ email, password })
// Invalid credentials -> 401. Provider/network outage -> 503; deadline -> 504;
// rate limit -> 429. Classify the provider error rather than mapping every error to 401.
// Success requires data.session; keep its tokens only in the Express session store.

// the access token expired → refresh it (one refresh at a time per session)
const { data: refreshed, error: refreshError } = await supabaseForLogin().auth.refreshSession({ refresh_token })
// Invalid/revoked refresh token -> end session, 401. Operational failures retain the
// session and return 429/503/504. Coordinate rotation across Express instances atomically.
```

**Identity verification**: `getSession()` reads local session state; it is not proof of
identity. `getClaims(jwt)` validates JWT claims, while `getUser(jwt)` queries Auth for a
fresh user record. Express chooses the check required by the operation, then separately
checks resource authorization. Handle verification outages as operational failures.

Protect `profiles.role` from self-promotion: permission to update a user's profile must
not permit changing privileged role fields. Never authorize using editable
`user_metadata`. JWT role claims remain stale until refresh; check current membership in
Express where immediate revocation is required.

Sources: https://supabase.com/docs/guides/auth/auth-hooks
and https://supabase.com/docs/reference/javascript/auth-getclaims
and https://supabase.com/docs/guides/auth/debugging/error-codes

## Auth ≠ Authorization

Authentication proves identity. Authorization decides what each user can do. Supabase Auth handles identity; RLS handles authorization. Do not conflate them.

A logged-in user is not automatically authorized to access all data. Every table needs its own RLS policies.

See [rls-patterns.md](./rls-patterns.md) for authorization patterns.
