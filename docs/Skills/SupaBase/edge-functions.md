# Edge Functions

> Source review: 2026-09-27. This is a template reference, not a running project.
> Historical execution/measurement statements below are author reports without a supplied
> reproduction harness; they are not acceptance evidence. This review ran no Supabase
> CLI, SQL, deployed function, or real provider integration.

> Project boundary: these are provider capabilities, not frontend permissions. All provider
> SDK calls, DB/Auth/Storage/Realtime integrations run in Express on Node.js for this project.
> Browser → Next.js → Express → Supabase. Provider-specific URLs/keys/tokens stay behind Express.
> The frontend uses same-origin relays; generic direct-client examples are not this project's flow.


Server-side TypeScript functions running on Deno, distributed globally at the edge.

Source: https://supabase.com/docs/guides/functions

## Runtime

- **Deno-based** (supabase `edge-runtime`, a Deno fork; Deno 2.x since Apr 2025).
- Standard Web APIs (Request, Response, Headers, fetch).
- ESM only — `import`, never `require`.
- Built on Supabase's `edge-runtime` (open source): https://github.com/supabase/edge-runtime

### Dependencies: use specifiers in `deno.json`, not CDN URLs

All three specifiers are supported — **including npm packages and Node built-ins**:

```json
// supabase/functions/<name>/deno.json  — per function, not global
{
  "imports": {
    "@supabase/supabase-js": "npm:@supabase/supabase-js@2",
    "@supabase/server": "npm:@supabase/server@^1",
    "@std/path": "jsr:@std/path@^1"
  }
}
```
```ts
import { createClient } from '@supabase/supabase-js';
import process from 'node:process';        // Node built-ins work
```

Prefer reproducible per-function import maps and dependency locks. URL imports can
include versions and editor types; they are not inherently untyped. Shared import maps
require deliberate dependency resolution rather than being inherently broken.

Source: https://supabase.com/docs/guides/functions/quickstart

## When to Use

These are general provider capabilities. In this template webhooks, OAuth, CRUD and
application logic belong in Express. An Edge Function needs an explicit architecture
decision and stays behind Express; it does not replace the Node.js backend.

| Use case | Edge Function? |
|----------|---------------|
| Webhook receiver (Stripe, etc.) | Optional provider adjunct |
| OAuth token exchange | Optional provider adjunct |
| Data transformation before DB write | Optional provider adjunct |
| Heavy CPU work (video processing, ML inference) | ✗ Use a dedicated worker |
| Long-running jobs | ✗ Wall clock: 150s free / 400s paid |
| Simple CRUD | Use Express, which may call PostgREST |

## Basic Structure

Adapted historical example reported from CLI **2.116.0**, not newly generated — note the two
imports, which most write-ups omit:

```typescript
// supabase/functions/hello-world/index.ts
import "@supabase/functions-js/edge-runtime.d.ts";
import { withSupabase } from "@supabase/server";

export default {
  fetch: withSupabase({ auth: ["publishable", "secret"] }, async (req, ctx) => {
    // ctx.authMode      → "publishable" | "secret" — branch on who called you
    // ctx.supabaseAdmin -> privileged client; availability is NOT authorization.
    // It can exist in user contexts too. This handler uses it only after secret auth.
    if (ctx.authMode === "secret") {
      const { user_id } = await req.json();
      const { data } = await ctx.supabaseAdmin.auth.admin.getUserById(user_id);
      return Response.json({ email: data?.user?.email });
    }
    const { name } = await req.json();
    return Response.json({ message: `Hello ${name}!` });
  }),
};
```

The CLI also writes a **`deno.json` import map** next to the function — without it neither
import resolves:

```json
{
  "imports": {
    "@supabase/functions-js": "jsr:@supabase/functions-js@^2",
    "@supabase/server": "npm:@supabase/server@^1"
  }
}
```

`@supabase/server` (1.5.x) is a separate npm package that handles auth, client creation and
context injection. Callers pass the key in an **`apiKey`** header:

```bash
curl -X POST 'http://127.0.0.1:54321/functions/v1/hello-world' \
  --header 'apiKey: sb_publishable_…' --data '{"name":"Functions"}'
```

Historically reported CLI 2.116.0 `--auth` values (not checked by executing 2.118.0):
`none`, `apikey` (default), `user`.

Older tutorials using `serve()` from `deno.land/std/http` are **outdated**.

Source: https://supabase.com/docs/guides/functions/quickstart

## Key Features

- **Background Tasks**: `EdgeRuntime.waitUntil()` extends worker work, not durable delivery.
  Persist required work before acknowledging and use a durable queue/outbox with retries.
- **Ephemeral Storage**: Temporary file storage per invocation (not persisted).
- **WebSocket support**: Can accept WebSocket connections.
- **Dashboard deployment**: Create, edit, and deploy directly from the Supabase Dashboard.

Source: https://supabase.com/docs/guides/functions/quickstart-dashboard

## Accessing Supabase from Edge Functions

Prefer the authenticated wrapper context rather than an unauthenticated handler that
returns table data. User endpoints use `withSupabase` user mode and its user-scoped
`ctx.supabase` client; privileged service endpoints use secret mode and explicitly
authorize each business operation before `ctx.supabaseAdmin`.

New automatically injected secret keys use the JSON dictionary
`SUPABASE_SECRET_KEYS`. Do not assume a singular `SUPABASE_SECRET_KEY` environment
variable is automatically present; a manually configured variable is a separate choice.
Never return these keys to callers. User JWTs and opaque API keys are different credentials.

Sources: https://supabase.com/docs/guides/functions/auth
and https://supabase.com/docs/guides/getting-started/api-keys

## Hard Limits

**Runtime** — these kill a running invocation:

| Limit | Value |
|-------|-------|
| Memory | 256 MB |
| CPU time per request | **2s** (actual CPU only; async I/O waits don't count) |
| Worker lifetime | 150s (Free) / 400s (paid); may serve multiple requests/background work |
| Request idle timeout | 150s → 504 |

**Platform** — these block a deploy or throttle you, and are the ones people hit first:

| Limit | Value |
|-------|-------|
| Function size | 20 MB (local bundling) / **5 MB** (server-side bundling) |
| Functions per project | 100 Free · 1,000 Pro · 2,000 Team · unlimited Enterprise |
| Secrets | 100 per project; up to 48 KiB (24,576 characters) per secret; name up to 256 characters; `SUPABASE_` prefix reserved |
| Log message length | 10,000 characters |
| Log events | 100 per 10 seconds |
| Recursive self-calls | 30 requests per trace in 60 seconds |

**Blocked or unavailable**: outgoing ports **25 and 587** (so no direct SMTP — use an email
API), serving HTML requires a custom domain, no Web Worker API, no Node `vm`, no
multithreading libraries.

Source: https://supabase.com/docs/guides/functions/limits

## Caveats

- **Cold start**: Functions spin down after inactivity. First request may be slower.
- **Local parity**: Local execution via CLI can differ from production. Verify in staging.
- **Memory limit**: 256 MB. Not suitable for large in-memory operations — stream instead.
- **No persistent filesystem**: Use Supabase Storage or the database for persistence.
