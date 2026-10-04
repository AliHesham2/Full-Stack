# Forms and Validation (2026)

## Recommendation: Server Actions + `useActionState` + Zod

The default for every form in this project: a Server Action on the `<form>`, `useActionState`
for pending state and errors (see below), and a Zod schema. The action never touches a database:
it validates, then calls Express through `apiFetch` (frontend-guide/02-ui/forms.md). The browser
never calls the backend itself.

```tsx
'use server';

import { z } from 'zod';
import { revalidatePath } from 'next/cache';
import { apiFetch, ApiError } from '@/lib/http';

// Zod 4: string formats are top-level functions; z.string().email() is deprecated
const schema = z.object({ email: z.email() });

// The one pattern (frontend-guide/03-data/backend-integration.md § Mutations and Error States):
// what the user can fix comes back as state (with what they typed); the rest goes to error.tsx.
export async function createUser(prevState: State, formData: FormData): Promise<State> {
  const values = { email: String(formData.get('email') ?? '') };
  const result = schema.safeParse(values);
  if (!result.success) return { errors: z.flattenError(result.error).fieldErrors, values };
  try {
    await apiFetch('/users', { method: 'POST', body: JSON.stringify(result.data) });
  } catch (err) {
    if (err instanceof ApiError && err.code === 'VALIDATION_ERROR') {
      return { errors: err.details, values };
    }
    if (err instanceof ApiError && (err.status === 409 || err.status === 422)) {
      return { message: err.message, values };
    }
    throw err;   // 401/403, an outage or a bug → error.tsx
  }
  revalidatePath('/users');
  return { success: true };
}
```

## React Hook Form — only for complex forms

Use it only for cross-field or per-keystroke validation, or forms with 100+ fields. It still
submits through a Server Action (in a transition), never with a browser `fetch` to the backend:

```tsx
'use client';
import { useState, useTransition } from 'react';
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { profileSchema, type ProfileInput } from '@/features/settings/schemas';
import { saveProfile } from '@/features/settings/actions'; // Server Action

function ProfileForm() {
  const [isPending, startTransition] = useTransition();
  const [message, setMessage] = useState<string | null>(null);
  const { register, handleSubmit, setError, formState: { errors } } =
    useForm<ProfileInput>({ resolver: zodResolver(profileSchema) });

  // The action returns what the user can fix as state (the same pattern as useActionState):
  // Express's field errors go onto their fields, a refusal's message above them.
  const onSubmit = (data: ProfileInput) => startTransition(async () => {
    const result = await saveProfile(data);
    setMessage(result.message ?? null);
    for (const [field, messages] of Object.entries(result.errors ?? {})) {
      setError(field as keyof ProfileInput, { message: messages[0] });
    }
  });

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      {message && <p role="alert">{message}</p>}
      <input {...register('email')} type="email" />
      {errors.email && <span role="alert">{errors.email.message}</span>}
      <button disabled={isPending}>Save</button>
    </form>
  );
}
```
**Why RHF here**: uncontrolled inputs (fewer re-renders) and one Zod schema shared between the
client check and the Server Action.
### Server Functions are public HTTP endpoints

This is the single most-missed thing about Actions, and react.dev's Server Functions page
does not mention it at all. Every `'use server'` function compiles to a **callable POST
endpoint with a stable ID**.

**Demonstrated against a real Next.js 16.3.4 production build.** The action ID is public in
two places — the rendered HTML and the build manifest:

```html
<!-- what the browser receives for <form action={deleteProject}> -->
<form action="" encType="multipart/form-data" method="POST">
  <input type="hidden" name="$ACTION_ID_40e505443c7f6f20b450492de5f103001514925329"/>
```
```
.next/server/server-reference-manifest.json → { "node": { "40e5054…": … } }
```

Replaying that as a plain POST — no browser, no session, no JS — ran the action:

```
POST /admin   Origin: <same-origin>
  $ACTION_ID_40e5054… = ""
  id = "OWNED-BY-ATTACKER"
→ 200
[SERVER] deleteProject EXECUTED with id=OWNED-BY-ATTACKER
```

**Exporting the function is what publishes it.** An action exported from a `'use server'`
file but rendered by **no component at all** was still in the manifest and still executed
when called directly. Deleting the UI does not unpublish the endpoint — deleting the export
does.

**What the framework does and does not do for you** (verified on Next 16.3.4):

| Attack | Next.js built-in | Your job |
|---|---|---|
| Cross-site POST from `evil.test` | **Blocked** — Origin/Host check: *"does not match origin header … Aborting the action"* | — |
| Same-origin POST by any user, authenticated or not | **Not blocked** — 200, action runs | the action forwards the caller's session; **Express** authenticates every call |
| Malformed / hostile arguments | not validated | parse with a schema, in the action |
| A logged-in user acting on someone else's row | not checked | **Express** checks this user may act on this row |

So the framework gives you CSRF protection, not authorization. The UI that "only admins can
see" is not a control. In this project the action is never the authorization boundary: it
validates, then forwards to Express, which authenticates and authorizes every request
(backend-guide/03-auth/).

```ts
'use server';
import { z } from 'zod';
import { apiFetch, ApiError } from '@/lib/http';

const DeleteInput = z.object({ id: z.uuid() });

export async function deleteProject(formData: FormData) {
  // 1. VALIDATE — arguments are attacker-controlled, whatever the TS types say
  const { id } = DeleteInput.parse({ id: formData.get('id') });

  // 2. FORWARD — apiFetch sends the caller's session. Express authenticates it AND checks this
  //    user may delete THIS project: a stranger gets 401, the wrong user gets 403
  try {
    await apiFetch(`/projects/${id}`, { method: 'DELETE' });
  } catch (err) {
    if (err instanceof ApiError && (err.status === 409 || err.status === 422)) {
      return { message: err.message };   // a refusal the user can act on: shown inline
    }
    throw err;   // 401/403, an outage or a bug → error.tsx
  }
  return { success: true };
}
```

**Rules**:
- Auth checks in a layout, page, or middleware do **not** protect an action — the action is
  reachable without ever rendering them (verified above).
- An unused exported action is still live. Audit with
  `node -p "Object.keys(require('./.next/server/server-reference-manifest.json').node).length"`
  and make sure that count matches the actions you meant to publish.
- TypeScript parameter types are erased. Re-validate every argument server-side.
- Never close over a secret and never return one; return values are serialized to the client.
- An action imported into a Client Component is **exposed by definition**; deleting the UI
  that calls it does not unpublish the endpoint. Delete the export.
- Client validation is UX; server validation is security. Express re-validates everything, and
  its check is the one that counts.
**Caveat**: `<form action={serverAction}>` bypasses React Hook Form — for progressive enhancement use native form actions; for the complex forms that need RHF, keep its `handleSubmit` calling the action via `startTransition` (above).

## Actions with state: `useActionState`
Wraps a Server Action (or async function) with pending/result handling — no manual loading state:
```tsx
// features/auth/actions.ts
'use server';
import { apiFetch, ApiError } from '@/lib/http';

// The one pattern (frontend-guide/03-data/backend-integration.md § Mutations and Error States):
// what the user can fix comes back as state; anything else is rethrown to error.tsx.
export async function signupAction(prevState: State, formData: FormData): Promise<State> {
  const email = String(formData.get('email'));
  if (!email.includes('@')) return { errors: { email: ['Invalid email'] }, values: { email } };
  try {
    await apiFetch('/auth/signup', { method: 'POST', body: JSON.stringify({ email }) });
  } catch (err) {
    if (err instanceof ApiError && err.code === 'VALIDATION_ERROR') {
      return { errors: err.details, values: { email } };
    }
    if (err instanceof ApiError && (err.status === 409 || err.status === 422)) {
      return { message: err.message, values: { email } };
    }
    throw err;   // 401/403, an outage or a bug → error.tsx
  }
  return { success: true };
}
```

```tsx
// features/auth/signup-form.tsx
'use client';
import { useActionState } from 'react';
import { signupAction } from './actions';

function Signup() {
  const [state, formAction, isPending] = useActionState(signupAction, {});
  return (
    <form action={formAction}>
      {state.message && <p role="alert">{state.message}</p>}
      <input name="email" type="email" required defaultValue={state.values?.email} />
      {state.errors?.email && <span role="alert">{state.errors.email}</span>}
      <button disabled={isPending}>{isPending ? 'Signing up…' : 'Sign up'}</button>
    </form>
  );
}
```

## Optimistic updates: `useOptimistic`
Show the expected result immediately, roll back automatically on failure:
```tsx
function SendMessages({ messages }: { messages: Message[] }) {
  const [optimistic, addOptimistic] = useOptimistic(
    messages,
    (current, text: string) => [...current, { id: 'temp', text }],
  );

  async function send(formData: FormData) {
    addOptimistic(String(formData.get('text')));   // instant UI
    await deliver(formData);                        // reverts on throw
  }

  return (
    <>
      {optimistic.map(m => <p key={m.id}>{m.text}</p>)}
      <form action={send}>...</form>
    </>
  );
}
```

## Avoid
- Formik — unmaintained
- React Final Form — unmaintained
- Manual `useState` for multi-field forms
- Trusting an action's TypeScript signature as an input contract → see the security note above

→ Sources: [react-hook-form.com](https://react-hook-form.com/), [zod.dev](https://zod.dev/), [useActionState](https://react.dev/reference/react/useActionState)
