# Data Display & Feedback

Components that **show** data and **report** status. The generic ones are kit (`components/ui/`);
anything that knows a product's data is a feature component.

## The Business Rule

```
components/ui/card.tsx                   ✅ generic surface
components/ui/data-table/data-table.tsx  ✅ generic table engine
features/products/product-card.tsx       ✅ product-specific, composes Card
components/ui/product-card.tsx           ❌ never — the kit does not know "products"
```

**Kit components only present.** They receive data as props and never fetch it. Fetching happens
in the page (Server Component) or the feature, per
[state-and-data](../03-data/state-and-data.md). A table in
"server mode" still doesn't fetch: it reports sort, filter and page changes through callbacks or
URL state (`nuqs`), and the page refetches.

## Every Data Component Handles Every State

No exceptions, from the first version:

| State | Shown as | Kit piece |
|---|---|---|
| Loading | a Skeleton in the final layout's shape (no layout jump) | `Skeleton` |
| Empty | "nothing here yet" plus the next action | `EmptyState` |
| No results | "nothing matches your search" plus clear filters, **different from empty** | `EmptyState` with `variant="no-results"` |
| Error | what failed plus a retry | `ErrorState` |
| Content | the data | — |

These are also responsive and correct in both themes. The patterns behind them are in
[resilience](../04-quality/resilience.md).

## Display Catalog

- **DataTable:** sorting, filtering, search, pagination, row selection, expandable rows, sticky
  header and first column, row actions; client mode or server mode. Built on the shadcn Table with
  TanStack Table. A multi-file composite in `components/ui/data-table/`: `data-table.tsx`,
  `data-table-toolbar.tsx`, `data-table-pagination.tsx`, `data-table-skeleton.tsx`, and so on,
  each imported directly.
- **Card** (with header / content / footer / action slots) · **StatCard** / **KPI** (label, value,
  trend with an accessible "up 12%" text, not just an arrow).
- **List** (plus a virtualized list for very long data) · **DescriptionList** (label → value
  details; use it instead of a two-column table).
- **Timeline** (ordered steps such as order created → paid → shipped) · **TreeView** (folders,
  categories, permissions).
- **Badge** (status: its color comes from the semantic status tokens, **and** it always has text;
  color never carries meaning alone) · **Tag** · **Avatar** / **AvatarGroup** (`+4` overflow) ·
  **Progress** (linear / circular) · **Rating**.

## Feedback Catalog

| Need | Component | Rule |
|---|---|---|
| Brief confirmation ("Saved") | **Toast** (shadcn's Sonner) | auto-dismisses; never the only place an error appears |
| Message tied to a section | **Alert** (`info`, `success`, `warning`, `destructive`) | stays until resolved |
| Confirm a destructive action | **AlertDialog** (ConfirmDialog) | says what will happen; the confirm button uses `variant="destructive"`; focus starts on Cancel |
| Work in progress | **Progress**, **Spinner** inside the button (`loading`) | prefer an inline pending state over a full-screen overlay |
| Blocking work (rare) | **LoadingOverlay** | only when the user truly must not interact |

## Rules

- Composable: build big components from small parts (DataTable = toolbar + table + pagination),
  never one giant configurable component.
- All text arrives as props. The kit has no built-in English or Arabic strings.
- Status colors come from tokens (`bg-success/10 text-success`), never hues.
- Mobile behavior of tables (cards on small screens) is in
  [layout-and-responsive](./layout-and-responsive.md).
