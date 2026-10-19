# Forms

## The Split: Display, Behavior, Validation, Data

| Layer | Job | Tool | Lives in |
|---|---|---|---|
| **Display** | fields, layout, labels, errors | the form component + kit inputs + `FormField` | `features/<f>/<name>-form.tsx` |
| **Behavior** | submit, pending, errors, reset | React 19 `useActionState` | same form component |
| **Validation** | rules and messages | a Zod schema | `features/<f>/schemas.ts` |
| **Data** | validate, then send to the backend | Server Action → `apiFetch` → Express | `features/<f>/actions.ts` |

There is **no client-side "service" layer and no `fetch` in forms**. The browser never calls
Express ([backend-integration](../03-data/backend-integration.md)). React Hook Form is not the
default: use it only for complex cross-field or per-keystroke validation, or forms with 100+
fields.

## The Modern Form Pattern

In React 19 with Next.js, the standard form pattern uses Server Actions directly on the `<form>` element. This replaces `onSubmit` handlers, form state libraries for submission, and manual fetch calls:

```tsx
// features/settings/profile-form.tsx
'use client';   // REQUIRED: useActionState is a client hook
import { useActionState } from 'react';
import { updateProfile } from '@/features/settings/actions';

export function ProfileForm() {
  const [state, formAction, isPending] = useActionState(updateProfile, null);

  return (
    <form action={formAction}>
      {state?.message && <p role="alert">{state.message}</p>}  {/* a refusal: above the fields */}
      <label htmlFor="name">Name</label>
      <input id="name" name="name" defaultValue={state?.values?.name} />
      {state?.errors?.name && (
        <p id="name-error" role="alert">{state.errors.name}</p>
      )}
      <button type="submit" disabled={isPending}>
        {isPending ? 'Saving...' : 'Save'}
      </button>
    </form>
  );
}
```

> ⚠️ **`useActionState` is a client hook.** A file under `app/` is a Server Component by
> default, so calling it there throws at render. Keep `app/settings/page.tsx` a thin Server
> Component that renders this client form:
>
> ```tsx
> // app/settings/page.tsx — Server Component
> import { ProfileForm } from '@/features/settings/profile-form';
> export default function SettingsPage() { return <ProfileForm />; }
> ```
>
> This is the `'use client'` boundary rule from [components](./components.md): the page
> stays on the server, the interactive leaf opts in.

The Server Action receives a `prevState` and `formData`. It validates, mutates, and returns the next state:

```tsx
// features/settings/actions.ts
'use server';
import { z } from 'zod';
import { apiFetch, ApiError } from '@/lib/http';

const schema = z.object({ name: z.string().min(1, 'Name is required') });

export async function updateProfile(prevState: FormState | null, formData: FormData) {
  const raw = Object.fromEntries(formData);
  const result = schema.safeParse(raw);
  if (!result.success) {
    return { errors: result.error.flatten().fieldErrors, values: raw };
  }
  try {
    await apiFetch('/users/me', { method: 'PATCH', body: JSON.stringify(result.data) });
  } catch (err) {
    if (err instanceof ApiError && err.code === 'VALIDATION_ERROR') {
      return { errors: err.details, values: raw };   // Express's field errors, same shape
    }
    if (err instanceof ApiError && (err.status === 409 || err.status === 422)) {
      return { message: err.message, values: raw };  // a business refusal: show its message
    }
    throw err;   // 401/403, an outage or a bug → error.tsx (backend-integration.md § Mutations and Error States)
  }
  return { success: true };
}
```

**Decision:** Use this pattern for all forms that submit to your own backend. The Server Action validates the input shape so field errors come back fast, then hands the validated data to `apiFetch`. Express still re-validates: the Server Action is a convenience layer, not the trust boundary. The action never writes to a database itself (see [backend-integration](../03-data/backend-integration.md)). Reach for client-side fetch only when calling external APIs that cannot be Server Actions.

## Validation Strategy

Express is the real validation — it re-checks everything, because any client (or a hand-written request) can reach it. Validating in the Server Action too is for fast, well-mapped field errors. Client-side validation is UX only. Neither the Server Action nor the browser is a security boundary.

Layer client validation with native HTML attributes first: `required`, `pattern`, `type`, `min`, `max`. These prevent obvious mistakes without JavaScript. For complex rules (e.g., password strength, cross-field confirmation), add custom validation in `onChange` or `onBlur` handlers.

**Decision:** Do not add client-side validation rules that duplicate the server schema unless they materially improve UX. Async client validation (username availability, format checking) is the exception where client-side logic adds real value — run it on `onBlur` with debounce.

## Fields and Errors: `FormField`

One kit component wires the label, control, hint and error together with the right ids and
ARIA, so no form repeats that plumbing:

```tsx
// components/ui/form-field.tsx
import { useId } from 'react';
import { cn } from '@/lib/cn';

type FormFieldProps = {
  label: string;
  hint?: string;
  error?: string | string[];
  className?: string;
  children: (field: { id: string; 'aria-describedby'?: string; 'aria-invalid'?: boolean }) => React.ReactNode;
};

export function FormField({ label, hint, error, className, children }: FormFieldProps) {
  const id = useId();
  const message = Array.isArray(error) ? error[0] : error;
  const describedBy =
    [hint && `${id}-hint`, message && `${id}-error`].filter(Boolean).join(' ') || undefined;

  return (
    <div className={cn('grid gap-2', className)}>
      <label htmlFor={id} className="text-sm font-medium">{label}</label>
      {children({ id, 'aria-describedby': describedBy, 'aria-invalid': message ? true : undefined })}
      {hint && <p id={`${id}-hint`} className="text-sm text-muted-foreground">{hint}</p>}
      {message && <p id={`${id}-error`} role="alert" className="text-sm text-destructive">{message}</p>}
    </div>
  );
}
```

```tsx
// features/users/user-form.tsx
<form action={formAction} className="grid gap-6 md:grid-cols-2">
  <FormField label="Email" error={state?.errors?.email}>
    {(field) => <Input {...field} name="email" type="email" defaultValue={values?.email} required />}
  </FormField>
  <Button type="submit" disabled={isPending} className="w-full md:col-span-2 md:w-auto">
    {isPending ? 'Saving…' : 'Save'}
  </Button>
</form>
```

**Decision:** Errors belong to their field and appear after the first submit, not on every
keystroke. `role="alert"` makes screen readers announce them. A form-level problem the user can act
on (a business refusal: the action's `state.message`) shows once, in an `Alert` above the fields.
A provider outage uses the route error boundary; session expiry follows the defined authentication flow. Login 401 is expected form feedback. Unexpected failures are rethrown to
`error.tsx` ([backend-integration](../03-data/backend-integration.md) § Mutations and Error States).
Show a top summary only for multi-step or very long forms.

## Optimistic Updates with `useOptimistic`

For actions where the user expects immediate feedback (liking a post, toggling a setting), use `useOptimistic`:

```tsx
'use client';
import { useOptimistic } from 'react';
import { toggleLike } from './actions';

export function LikeButton({ postId, liked, count }) {
  // Track both values together — the count has to move with the flag.
  const [optimistic, setOptimistic] = useOptimistic(
    { liked, count },
    (_state, next: { liked: boolean; count: number }) => next,
  );

  return (
    <form
      action={async () => {
        setOptimistic({
          liked: !optimistic.liked,
          count: optimistic.count + (optimistic.liked ? -1 : 1),
        });
        await toggleLike(postId);
      }}
    >
      <button type="submit" aria-pressed={optimistic.liked}>
        {optimistic.liked ? 'Liked' : 'Like'} ({optimistic.count})
      </button>
    </form>
  );
}
```

**Three things an earlier revision of this example got wrong**, all of which break at runtime
or produce visibly wrong output:

- **`formAction` needs a `<form>`.** A bare `<button formAction={...}>` with no enclosing
  form does nothing. Put the action on the `<form>` (or keep `formAction` on a button that
  is genuinely inside one).
- **`useOptimistic` updates must run inside an action or transition.** Calling the setter from
  a plain `onClick` throws *"An optimistic state update occurred outside a transition or
  action"*. The form `action` above provides that scope.
- **Derive the count from the same optimistic value.** The earlier form
  (`count + (optimisticLiked && !liked ? 1 : 0)`) only ever adds — un-liking left the count
  unchanged, so the UI contradicted itself. Optimistic state that disagrees with the thing it
  is optimistic about is worse than no optimism at all.

**Decision:** Use `useOptimistic` sparingly. It adds complexity around rollback. For forms with validation (most forms), `useActionState` is the right tool. Reserve `useOptimistic` for toggle-like actions where validation is not a concern.

## Pending State with `useFormStatus`

`useFormStatus` reads the pending state of a parent `<form>`. It must be used in a child component — it cannot be called in the same component that renders the `<form>`:

```tsx
function SubmitButton() {
  const { pending } = useFormStatus();
  return <button disabled={pending}>{pending ? 'Saving...' : 'Save'}</button>;
}
```

**Decision:** Prefer `useActionState`'s `isPending` return value when you have it available. Use `useFormStatus` only when the button component is extracted separately (e.g., shared across forms) and cannot access the action state.

## Dirty State and Navigation Warning

Track whether the form has been modified to warn users before navigating away. Use a `useFormDirty` hook that compares current values against initial values:

```tsx
const isDirty = useFormDirty(formRef);

useEffect(() => {
  if (!isDirty) return;
  const handler = (e: BeforeUnloadEvent) => {
    e.preventDefault();
  };
  window.addEventListener('beforeunload', handler);
  return () => window.removeEventListener('beforeunload', handler);
}, [isDirty]);
```

For in-app navigation, use `next/navigation`'s `useRouter` events or a route change blocker if available.

## Accessible Forms

Every form must meet these requirements:

- Every `<input>`, `<select>`, `<textarea>` has an associated `<label>` via `htmlFor` and `id`. Never use placeholder text as a substitute for a label.
- Related fields are grouped in `<fieldset>` with `<legend>`. This is essential for radio groups and checkbox groups.
- Hints and errors are linked with `aria-describedby`; invalid fields set `aria-invalid="true"`. `FormField` does both.
- **Every kit input submits natively through `FormData`:** it accepts `name` and needs no React state. Radix Select, Checkbox and Switch do this when given `name`. This is what makes Server Actions and progressive enhancement work.
- **Layout:** one column on mobile, two from `md` for wide forms; the submit button is full width on mobile ([layout-and-responsive](./layout-and-responsive.md)).
- **Default values come in as props:** the page (a Server Component) fetches the record and passes it down. Never hard-code them in the form.
- **Text comes from the feature**, never from the kit: `FormField` takes a `label`, and the feature decides the words.

## File Uploads

Use Server Actions with FormData. Files arrive as `File` objects in the FormData entry:

```tsx
'use server';
import { apiFetch, ApiError } from '@/lib/http';

export async function uploadAvatar(prevState: FormState, formData: FormData) {
  const file = formData.get('avatar');
  if (!(file instanceof File) || file.size === 0) {
    return { errors: { avatar: ['Please select a file'] } };
  }
  if (file.size > 512 * 1024) {            // must stay under the configured body limit
    return { errors: { avatar: ['File must be at most 512KB'] } };
  }
  if (!['image/png', 'image/jpeg', 'image/webp'].includes(file.type)) {
    return { errors: { avatar: ['Upload a PNG, JPEG, or WebP image'] } };
  }
  // Forward to Express, which uploads to storage and records the URL — see
  // 03-data/backend-integration.md for the Next.js→Express streaming pattern on larger files.
  try {
    await apiFetch('/users/me/avatar', { method: 'POST', body: formData });
  } catch (err) {
    if (err instanceof ApiError && err.code === 'VALIDATION_ERROR') return { errors: err.details };
    if (err instanceof ApiError && (err.status === 409 || err.status === 422)) {
      return { message: err.message };
    }
    throw err;   // 401/403, an outage or a bug → error.tsx
  }
  return { success: true };
}
```

> ⚠️ **Next.js caps a Server Action request body at 1 MB by default.** Verified against the
> `serverActions` config docs: *"By default, the maximum size of the request body sent to a
> Server Action is 1MB."* This has a consequence people discover in production:
>
> **A size check above the limit never runs.** An earlier revision of this example rejected
> files over 5 MB — but a 5 MB upload is refused by the framework *before your action is
> invoked*, so the user gets an opaque server error instead of "File must be under 5MB". Your
> in-action check only ever fires for files **below** the configured limit.
>
> **Match the check to the limit.** Either keep uploads under 1 MB, or raise it deliberately:
>
> ```ts
> // next.config.ts
> const nextConfig: NextConfig = {
>   experimental: { serverActions: { bodySizeLimit: '5mb' } },
> };
> ```
>
> Raising it costs server memory on every parse and widens a DoS surface — which is why the
> default is low. **Above a few MB, stop using Server Actions for the bytes** and switch to a
> same-origin Next.js Route Handler streaming through Express to storage → [backend-integration](../03-data/backend-integration.md).
>
> Always validate `file.type` as well as size — and never trust it as proof of content;
> the backend must sniff the real type before storing.

**Decision:** Server Actions for small uploads below the body limit including multipart overhead; bounded streaming/chunks through Next.js Route Handler → Express → storage for larger files. No direct browser-to-Supabase upload.

## Reusable vs Feature-Specific Form Logic

Keep simple forms inline with `useActionState`. Extract custom hooks only when validation logic is reused across multiple forms (e.g., `useFormValidation(schema)` that wraps zod schemas with client-side error messages matching the server schema). Do not abstract forms into a generic "Form" component — the `<form>` element is already the abstraction. See [components](./components.md) for component extraction guidelines.

Reusable field groups (address, password + confirm, contact info): a generic one goes in `components/ui/`, and one that knows the product's fields stays in its feature.

**Input catalog** (add when a form first needs one): Input, Textarea, Select, Combobox, Checkbox, RadioGroup, Switch, DatePicker, InputOTP, FileUpload, PasswordInput (show / hide), SearchInput, NumberInput, PhoneInput.

**Naming:** `LoginForm`, not `Login` · `UserForm`, not `UserCreate` · `PasswordInput`, not `InputPassword`. Files are kebab-case: `login-form.tsx`.
