# URL State

## Rule
State that should survive refresh, be shareable, or drive navigation belongs in the URL. Use `nuqs` for typed search params in App Router apps.

```tsx
import { useQueryState, parseAsInteger } from 'nuqs';

function ProductList() {
  const [page, setPage] = useQueryState('page', parseAsInteger.withDefault(1));

  return (
    <button onClick={() => setPage(page + 1)}>Next</button>
  );
}
```
Without a library: framework-native `useSearchParams` (Next.js / React Router) + manual parsing and serialization.

## What belongs in the URL
- Pagination, filters, sort order
- Active tab, open modal/panel identity
- Search query

**Why**: Refresh-safe, shareable, back-button correct, server-readable — free analytics on what users filter.

## What does NOT belong in the URL
- Form input values mid-edit → [forms.md](./forms.md)
- Hover/focus/transient UI state
- Sensitive data (tokens, PII — URLs end up in logs, history, Referer headers)
  → Background: [MDN — Referer header](https://developer.mozilla.org/en-US/docs/Web/Privacy/Referer_header:_privacy_and_security_concerns)

## Rules
- Keep params canonical: one param name per concept; defaults may be omitted from the URL string but must parse identically
- Batch updates when several params change together (`nuqs` throttles history entries by default)
- Server Components read search params via props/searchParams — keep parsing logic shared with the client side

→ Sources: [nuqs.47ng.com](https://nuqs.47ng.com/)
