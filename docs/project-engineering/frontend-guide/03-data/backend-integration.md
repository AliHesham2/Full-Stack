# Backend Integration — Next.js → Express on Node.js → Database

## Fixed Boundary

Browser → Next.js server → Express.js on Node.js → database provider.
The current provider is Supabase (Postgres, Auth and Storage); it can be replaced behind Express.
Neither Next.js nor the browser imports a database/Supabase SDK, holds provider keys/tokens,
opens a DB connection or talks directly to provider Auth/Storage/Realtime.

Next.js owns rendering, same-origin transport and the HttpOnly cookie. Express owns business
rules, authorization, provider auth/session refresh, queries and file storage. Changing the DB
provider must not change frontend flows. The two apps share an HTTP contract, not source imports.

## Choose the Transport

| Operation | Path |
|---|---|
| Initial page read | Server Component → feature queries.ts → apiFetch → Express |
| Interactive read / polling | Browser GET to Next.js route.ts → feature query → apiFetch → Express |
| Mutation / form | Browser → Next.js Server Action → apiFetch → Express |
| Large upload / download | Browser → Next.js Route Handler → Express → storage; bounded streaming |
| SSE | Browser → Next.js streaming Route Handler → Express → provider |
| Bidirectional realtime | Separate transport decision: host must support a Next.js-facing WebSocket relay; otherwise use SSE + mutations/polling |

Server Actions are for mutations. Do not use them as the default TanStack Query read transport.
AbortSignal cannot be serialized through a Server Action. A browser fetch signal cancels the
browser's same-origin request; the Route Handler also forwards its request signal and applies
its own deadline. Test disconnect propagation on the chosen hosting platform.

Sources: [React Server Functions](https://react.dev/reference/rsc/use-server),
[Next Route Handlers](https://nextjs.org/docs/app/api-reference/file-conventions/route).

## One Express Transport Module

Only lib/http.ts knows EXPRESS_API_URL and any Next→Express service credential.
Feature code owns typed endpoint wrappers; no arbitrary client-controlled proxy path.
External JSON is unknown until a runtime schema validates it.

```ts
// lib/http.ts — server-only; Express is the ONLY backend destination
import 'server-only';
import { cookies } from 'next/headers';
import { z } from 'zod';

export class ApiError extends Error {
  constructor(
    public status: number,
    public code: string,
    message: string,
    public details?: Record<string, string[]>,
  ) { super(message); }
}

const ErrorEnvelope = z.object({
  error: z.object({
    code: z.string(),
    message: z.string(),
    details: z.record(z.string(), z.array(z.string())).optional(),
  }),
});
type Options = RequestInit & {
  auth?: boolean;
  timeoutMs?: number;
  next?: { revalidate?: number | false; tags?: string[] };
  duplex?: 'half'; // streaming request bodies on Node; verify host support
};

export async function apiRequest(
  path: string,
  { auth = true, timeoutMs = 10_000, ...init }: Options = {},
): Promise<Response> {
  const base = process.env.EXPRESS_API_URL;
  if (!base) throw new Error('EXPRESS_API_URL is required');
  if (!path.startsWith('/') || path.startsWith('//')) throw new Error('Invalid API path');
  const session = auth ? (await cookies()).get('session')?.value : undefined;
  const headers = new Headers(init.headers); // supports Headers, tuples and records
  if (typeof init.body === 'string' && !headers.has('Content-Type')) {
    headers.set('Content-Type', 'application/json');
  }
  if (init.body instanceof FormData) headers.delete('Content-Type'); // generated boundary
  // Never let caller-provided headers replace identity or a service credential.
  headers.delete('Authorization');
  if (session) headers.set('Authorization', 'Bearer ' + session);
  headers.delete('X-Service-Key');
  const serviceKey = process.env.EXPRESS_SERVICE_KEY;
  if (serviceKey) headers.set('X-Service-Key', serviceKey);
  const deadline = AbortSignal.timeout(timeoutMs);
  const signal = init.signal ? AbortSignal.any([init.signal, deadline]) : deadline;
  let res: Response;
  try {
    res = await fetch(base.replace(/\/$/, '') + path, {
      ...init, headers, signal,
      ...(auth ? { cache: 'no-store' as const, next: undefined } : {}),
    });
  } catch (err) {
    if (init.signal?.aborted) throw err; // caller no longer wants this response
    if (deadline.aborted) throw new ApiError(504, 'UPSTREAM_TIMEOUT', 'Service temporarily unavailable');
    if (err instanceof TypeError) {
      throw new ApiError(503, 'UPSTREAM_UNAVAILABLE', 'Service temporarily unavailable');
    }
    throw err;
  }
  if (!res.ok) {
    const body: unknown = await res.json().catch(() => null);
    const parsed = ErrorEnvelope.safeParse(body);
    const error = parsed.success ? parsed.data.error : undefined;
    // Operational errors must not leak infrastructure details to the UI.
    throw new ApiError(
      res.status, error?.code ?? 'UNKNOWN',
      res.status >= 500 ? 'Service temporarily unavailable' : (error?.message ?? 'Request failed'),
      res.status === 400 ? error?.details : undefined,
    );
  }
  return res;
}

export async function apiFetch(path: string, options: Options = {}): Promise<unknown> {
  const res = await apiRequest(path, options);
  if (res.status === 204) return null;
  const body: unknown = await res.json();
  return body;
}
```

Authenticated responses are no-store. Only explicitly public endpoints may opt into a shared
cache using auth:false; never pass cookie/private data into use cache. Supplying a cancellation
signal does not remove the server timeout. Upload/stream endpoints set a deliberate timeout,
not an infinite default. Streams keep the fetch signal alive until body consumption ends.

Express sits on a private network where possible. If it is reachable publicly, Next.js sends
EXPRESS_SERVICE_KEY as X-Service-Key and Express verifies it on every frontend-facing endpoint.
That secret is server-only in both apps, rotated with overlap; never NEXT_PUBLIC_.
This authenticates the calling service, not the end user; Express still enforces user authorization.
Provider webhooks are separate verified ingress with provider signatures, never a blanket bypass.
No CORS headers are needed; CORS is not a firewall and simple cross-origin requests can arrive.

## API Contracts and Server Queries

Generate contract types/schemas from Express OpenAPI into the frontend, or maintain a validated
HTTP contract locally. Do not import backend source or introduce a monorepo/shared workspace.
A TypeScript interface alone does not validate JSON.

Ordinary success responses use { data: T }; lists may include pagination fields specified by
their contract. Errors use { error: { code, message, details? } }. Login is the explicit exception
200 { token }; logout is 204 without a body.

```ts
// features/projects/contracts.ts — data schema, no I/O; client/server safe
import { z } from 'zod';
export const ProjectSchema = z.object({ id: z.uuid(), name: z.string() });
export const ProjectsEnvelope = z.object({ data: z.array(ProjectSchema) });
export type Project = z.infer<typeof ProjectSchema>;

// features/projects/queries.ts — NOT a Server Action
import 'server-only';
import { apiFetch } from '@/lib/http';
import { ProjectsEnvelope } from './contracts';
export async function getProjects(signal?: AbortSignal) {
  return ProjectsEnvelope.parse(await apiFetch('/projects', { signal })).data;
}
```

List endpoints must paginate: this short example shows validation/unwrapping only. Extend the
schema with the API's cursor/page metadata rather than fetching an unbounded collection.

A Server Component imports getProjects from queries.ts and renders its result. It may call
independent server queries with Promise.all or stream slow regions under Suspense.

## Interactive Reads

```ts
// app/api/projects/route.ts — fixed endpoint, not an arbitrary proxy
import { getProjects } from '@/features/projects/queries';
import { ApiError } from '@/lib/http';
export async function GET(request: Request) {
  try {
    return Response.json({ data: await getProjects(request.signal) }, {
      headers: { 'Cache-Control': 'no-store' },
    });
  } catch (err) {
    if (!(err instanceof ApiError)) throw err; // log/report unexpected failures server-side
    return Response.json(
      { error: { code: err.code, message: err.message } },
      { status: err.status, headers: { 'Cache-Control': 'no-store' } },
    );
  }
}

// features/projects/client-queries.ts — browser calls Next.js ONLY
import { ProjectsEnvelope } from './contracts';
export class QueryHttpError extends Error {
  constructor(public status: number) { super('Could not load projects'); }
}
export async function fetchProjects(signal: AbortSignal) {
  const res = await fetch('/api/projects', {
    signal, credentials: 'same-origin', cache: 'no-store',
  });
  if (!res.ok) throw new QueryHttpError(res.status);
  const body: unknown = await res.json();
  return ProjectsEnvelope.parse(body).data;
}
```

TanStack Query uses queryFn:({signal})=>fetchProjects(signal). Preserve query cancellation
and combine it with a browser deadline when needed. Add cursor/filter query parameters through
a whitelist and validation in route.ts; never forward arbitrary URLs, headers or credentials.
For cancellation all the way through, a server query accepts a server-local signal and route.ts
passes request.signal to that query/apiFetch. It is not a serialized action argument.

## Authentication

Express alone talks to Supabase Auth, creates/revokes its opaque sessions and refreshes provider
tokens. Next.js never decodes or refreshes a provider token. On successful POST /auth/login,
Express returns { token }; Next.js sets that opaque value in its own HttpOnly cookie.

```ts
// inside features/auth/actions.ts (module starts with 'use server')
const LoginResult = z.object({ token: z.string().min(1) });
const result = LoginResult.parse(await apiFetch('/auth/login', {
  method: 'POST', body: JSON.stringify({ email, password }), auth: false,
}));
(await cookies()).set('session', result.token, {
  httpOnly: true,
  secure: process.env.NODE_ENV === 'production', // localhost dev works over HTTP; prod needs HTTPS
  sameSite: 'lax',
  path: '/',
  maxAge: 60 * 60 * 24 * 7, // no longer than Express's absolute session lifetime
});
redirect('/dashboard'); // outside try/catch: redirect works by throwing
```

The complete action imports z, cookies, redirect and apiFetch/ApiError, validates the input
against its contract and returns a typed form state. Invalid credentials are 401; invalid shape
is 400; throttling is 429; dependency failure is 503/504. Operational errors do not clear a valid
session record. Never return a session token to a Client Component or include it in logs.

getSession() is a server-only query: validate and unwrap { data: user } from /auth/me, return null
only on a genuine 401 and rethrow operational failures. Server Components/layouts can redirect
on null for UX; Express always rechecks identity and object-level authorization. Logout revokes
the session through Express, then deletes the Next.js cookie. OAuth exchanges happen in Express;
Next.js relays the resulting opaque session through a validated callback/one-time handoff flow.

## Mutations and Error States

Use Server Actions with useActionState for forms or useTransition for button actions.
Validate serializable inputs; Express independently validates, authenticates and authorizes.
Return validation/business refusals as typed form state (400 details, 409/422 message).
Throw unexpected failures to the route error boundary; show loading/error/empty states.
Login's invalid credentials are form feedback. Mutations are never automatically retried:
set retry:false and require an explicit user retry. Timeout can mean the write succeeded but
the response was lost; reconcile its outcome first. Idempotency can make an explicit retry
safe, but does not authorize automatic retry under this project's policy.
After mutation, revalidate the affected RSC path and invalidate relevant TanStack Query keys.
For optimistic TanStack mutations, snapshot in onMutate and restore in onError; rollback is not
automatic. React useOptimistic follows the authoritative base state instead.

## Pagination and Realtime

For interactive pagination, useInfiniteQuery fetches the Next.js GET route with the cursor and
query signal. RSC pagination reads URL params and uses a server query. Carry pagination metadata
through the contract and validate params in Next.js for UX and again in Express for enforcement.

SSE relays use apiRequest, a Next.js streaming Route Handler and request.signal. Browser
EventSource connects only to that same-origin Next.js route. Express owns provider subscriptions,
authorization, cleanup on res.close and bounded backpressure. Test host streaming/time limits.
No browser Supabase subscriptions and no direct browser-to-Express WebSockets.

## Third-Party Integrations

Provider credentials, business integrations and provider-specific clients stay behind Express.
Next.js requests the resulting view data through apiFetch with a timeout and optional fallback;
it does not expose third-party secrets or bypass the backend's authorization.

## File Uploads and Downloads

This project uses proxied file bytes throughout:
Browser → Next.js Server Action/Route Handler → Express → Supabase Storage.
Only Express knows bucket names, object keys, provider credentials and storage API details.
Do not generate client-facing Supabase upload URLs or install its SDK in Next.js.

Small uploads fit the configured Server Action limit including multipart overhead. For larger
files, use a dedicated same-origin Next.js Route Handler that streams to a fixed Express endpoint
through apiRequest; never call request.formData() for an unbounded upload. Express streams to
the provider when its API supports it, or uses bounded temporary storage/resumable chunks.
Verify Next.js host body-size/time limits first; a Route Handler does not bypass platform limits.
If needed, use bounded chunks via Next.js→Express; a direct-storage architecture change requires
a new explicit user decision.

Enforce actual byte limits at Next.js ingress, Express parser/stream and storage configuration.
Content-Length and client MIME are hints, not proof. Express verifies content, assigns keys,
checks quota/ownership, quarantines where required and records metadata only after confirmed
storage success. On abort/failure, clean partial objects and reconcile orphans.
Downloads also use authenticated Next.js→Express relays; never blindly redirect to a provider URL.
The frontend uses stable provider-independent file IDs and same-origin /api/files/<id> URLs.
