# Components

Scope: follow [frontend-only contract authority](../00-navigation/frontend-only-scope.md). Endpoint names, payloads and session settings in examples apply only when the actual owner-supplied contract matches; examples do not define external API behavior.

Every component has exactly one job. If you cannot describe what it does in one sentence, split
it. This is the most important rule in this file.

## Three Levels, Two Places

| Level | Examples | Lives in |
|---|---|---|
| **1. Primitive** | Button, Input, Card, Dialog, Avatar, Badge | `components/ui/` |
| **2. Composite**: primitives combined, still generic | SearchInput, DataTable, ConfirmDialog, EmptyState | `components/ui/` |
| **3. Feature**: knows about the product | LoginForm, ProjectCard, UserTable | `features/<feature>/` |

The boundary is knowledge: a component that knows what a "project" or "user" is belongs to a
feature. One that only knows generic UI ideas ("pressed", "disabled", "variant") belongs in
`components/ui/`. `ui/` components decide *how* to show, never *when* to show. `components/ui/`
is a **portable kit** that gets copied between projects; the rules that keep it copyable are in
[portability](./portability.md).

## Portability Rules — What Keeps the Kit Copyable

A `components/ui/` file may import **only** from React, Radix / `cva` / the icon library,
`@/lib/cn`, and other `components/ui/` files. It must **never**:

- import from `features/`, `app/`, `actions/`, `lib/http.ts` or `lib/session.ts`, or read an env
  variable (ESLint blocks the imports; `scripts/check-kit.sh` blocks `process.env`);
- fetch data, call a Server Action, or use TanStack Query, a store, or the router;
- contain product text. Labels, messages and placeholders arrive as props or children, so the kit
  works in any language;
- use a hue class, a hex color, an arbitrary value, or a raw `z-*` number ([design-tokens](./design-tokens.md));
- use a physical left/right utility. Use the logical ones so the same component works in English
  (LTR) and Arabic (RTL): `ps-*` / `pe-*`, `ms-*` / `me-*`, `inset-s-*` / `inset-e-*`,
  `border-s` / `border-e`, `rounded-s` / `rounded-e`, `text-start` / `text-end`. Flip
  directional icons with `rtl:rotate-180`.

The checks that enforce these are in [portability](./portability.md).

## Where Components Come From

- **shadcn/ui first.** `npx shadcn@latest add button` copies the source into
  `components/ui/button.tsx`, built on **Radix**. It is our code from then on; edit it freely.
  There is no upstream package to update.
- **Build it yourself** only when shadcn has nothing close, and still on a Radix primitive if it is
  interactive. Don't build compound-component plumbing yourself; Radix provides it. Pick one
  headless library per project (Radix is the default with shadcn) and don't mix.
- **Add a component when a feature first needs it.** The catalog at the end is a menu, not a build
  list. Inline first; move to `components/ui/` when a third call site needs it
  ([architecture](../01-foundation/architecture.md)).
- **After `shadcn add`, check the file against the token rules:** replace any raw palette class
  (`text-white`, `bg-black/50`) with the semantic pair (`text-destructive-foreground`,
  `bg-background/80`). See [design-tokens](./design-tokens.md).

## Component API

Props make the common case easy and the advanced case possible. Every kit component accepts
`className`, `children`, `id`, `ref` and its element's native props, plus where relevant:

| Prop | Values | Notes |
|---|---|---|
| `variant` | Button: `default` `secondary` `outline` `ghost` `link` `destructive` | colors only from tokens |
| `size` | `sm` `default` `lg` `icon` | token spacing and text steps |
| `disabled` / `loading` | boolean | `loading` shows a spinner, keeps the width, sets `aria-busy` |
| `asChild` | boolean | Radix Slot: render as a link or other element with the same styles |

- **Variants use `cva`.** Never boolean flags for looks: use `variant="default"`, not `primary={true}`.
- **Merge classes with `cn()`** so a caller's `className` can extend or override. It needs
  `clsx` and `tailwind-merge`:

  ```ts
  // src/lib/cn.ts
  import { clsx, type ClassValue } from 'clsx';
  import { twMerge } from 'tailwind-merge';

  export function cn(...inputs: ClassValue[]) {
    return twMerge(clsx(inputs));
  }
  ```
- **Full width is layout, not a variant:** the caller passes `className="w-full"`.
- **`ref` is a normal prop** in React 19. No `forwardRef`.

```tsx
// components/ui/button.tsx — the shape of every variant-driven component
import { cva, type VariantProps } from 'class-variance-authority';
import { Slot } from '@radix-ui/react-slot';
import { cn } from '@/lib/cn';

const buttonVariants = cva(
  'inline-flex items-center justify-center gap-2 rounded-md text-sm font-medium transition-colors duration-150 focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring disabled:pointer-events-none disabled:opacity-50 motion-reduce:transition-none',
  {
    variants: {
      variant: {
        default: 'bg-primary text-primary-foreground hover:bg-primary/90',
        secondary: 'bg-secondary text-secondary-foreground hover:bg-secondary/80',
        outline: 'border border-input bg-background hover:bg-accent hover:text-accent-foreground',
        ghost: 'hover:bg-accent hover:text-accent-foreground',
        link: 'text-primary underline-offset-4 hover:underline',
        destructive: 'bg-destructive text-destructive-foreground hover:bg-destructive/90',
      },
      size: {
        sm: 'h-9 px-3 pointer-coarse:h-11',  // compact with a mouse, 44px on touch screens
        default: 'h-11 px-4',                 // 44px: the project's minimum touch target
        lg: 'h-12 px-6 text-base',
        icon: 'size-11',
      },
    },
    defaultVariants: { variant: 'default', size: 'default' },
  },
);

type ButtonProps = React.ComponentProps<'button'> &
  VariantProps<typeof buttonVariants> & { asChild?: boolean };

export function Button({ className, variant, size, asChild, ...props }: ButtonProps) {
  const Comp = asChild ? Slot : 'button';
  return <Comp className={cn(buttonVariants({ variant, size }), className)} {...props} />;
}
```

**Composition: parts over configuration props.** Use `<Dialog><DialogTrigger/><DialogContent>
<DialogHeader><DialogTitle/></DialogHeader><DialogFooter/></DialogContent></Dialog>`, never
`<Dialog title footer />`. Named slots (`CardHeader`, `CardAction`) give predictable layouts;
prefer them over render props.

**Controlled and uncontrolled:** stateful components support both, Radix-style: `open` +
`onOpenChange`, or `defaultOpen`; `value` + `onValueChange`, or `defaultValue`.

## Interaction States

Every interactive element implements all five:

- **Default:** token-defined.
- **Hover:** `hover:`. If the design doesn't define one, don't invent it.
- **Focus:** `focus-visible:` (not `focus:`), with the `ring` token. Never skip it; it is an
  accessibility requirement.
- **Active:** `active:`, usually a slight color shift.
- **Disabled:** `disabled:` / `data-disabled:`. Fade enough to read as inactive, not absent (WCAG
  exempts disabled text from contrast, but an unreadable label still hurts). Prefer
  `aria-disabled` when users might try the control: `disabled` removes it from the tab order, so
  they can never reach it to learn why.

Every kit component also has: keyboard support and correct ARIA ([accessibility](./accessibility.md)),
a **44×44px touch target** (WCAG AA requires 24×24; 44 is the project standard), and correct
rendering in light, dark, LTR and RTL.

## Server and Client

Push `'use client'` to the deepest leaf that needs it: an event handler, state, an effect, or a
browser API. A list that only maps data to cards stays a Server Component; only the card with
the delete button is a client component. A Server Component can pass rendered children into a
client component. See [architecture](../01-foundation/architecture.md).

**State:** local UI state (`useState`) stays inside; props are never mutated; shareable state
(filters, tabs, page) goes in the URL ([state-and-data](../03-data/state-and-data.md)). **A
component never fetches data it doesn't own.** Its parent (usually a Server Component) fetches and
passes it down.

## Native `<dialog>` for Simple Modals

For confirmations and alerts, `showModal()` gives focus trapping, focus return, Escape to close,
top-layer rendering and an inert background for free:

```tsx
function Dialog({ open, onOpenChange, children }) {
  const ref = useRef<HTMLDialogElement>(null);
  useEffect(() => {
    const el = ref.current;
    if (!el) return;
    if (open && !el.open) el.showModal();   // showModal() on an open dialog throws
    if (!open && el.open) el.close();
  }, [open]);
  return (
    <dialog ref={ref} onClose={() => onOpenChange(false)} aria-labelledby="dialog-title">
      {children}
    </dialog>
  );
}
```

It does **not** lock body scroll (`globals.css` handles that) and does not name itself (point
`aria-labelledby` at the heading). For animated or complex dialogs, use Radix Dialog.

## Size

200 lines is a prompt to ask "is this still one job?", not a law. A cohesive 300-line component is
better than three 100-line pieces that only make sense together. Split by job, never by line
count or imagined reuse.

## File Conventions

- One component per file, **named export**, kebab-case file: `project-card.tsx` exports `ProjectCard`.
- **No `index.ts` barrels.** Import from the file: `import { Button } from '@/components/ui/button'`.
- A multi-part composite may get a folder, with each part imported directly:
  `components/ui/data-table/data-table.tsx`, `data-table-toolbar.tsx`, and so on.
- Tests live in each tester's folder, not next to the file (`tests/<assigned-tester>/components/ui/button.test.tsx`;
  see [testing](../04-quality/testing.md)).
  Storybook is deferred until the project decides it needs it.

## Catalog

Add from here when a feature needs one; shadcn provides most of them.

- **Actions:** Button, IconButton (`size="icon"` + `aria-label`), DropdownMenu, ContextMenu
- **Inputs:** Input, Textarea, Select, Combobox, Checkbox, RadioGroup, Switch, Slider, InputOTP,
  DatePicker, FileUpload. Form wiring is in [forms](./forms.md)
- **Layout:** Container, Card, Separator, AspectRatio, ScrollArea. Stack and Grid are plain
  Tailwind `flex` / `grid`, not components ([layout-and-responsive](./layout-and-responsive.md))
- **Navigation:** Breadcrumb, Tabs, Pagination, Stepper, NavigationMenu, Sidebar
- **Overlays:** Dialog, AlertDialog, Sheet/Drawer, Popover, Tooltip, HoverCard
- **Data and feedback:** [data-display-and-feedback](./data-display-and-feedback.md)
- **Media:** Avatar, Image (`next/image`), Icon (the icon library), Carousel
