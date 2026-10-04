# Portability: Copying the Kit to Another Project

The shared frontend is built to be **copied into a new project and work immediately**. The new
project changes its look by editing **values**, never components. This file is the contract.

## What Never Changes (Copy As-Is)

**UI kit:**

| Path | What |
|---|---|
| `src/components/ui/**` | every shared component, including `theme-toggle`, `form-field`, `container` |
| `src/components/theme-provider.tsx` | `next-themes` setup |
| `src/lib/cn.ts` | class merging |
| `src/styles/tokens.css` | semantic names → Tailwind, z-index layers, dark variant |
| `src/app/globals.css` | imports and base styles |
| `components.json` | shadcn config, so `npx shadcn add` keeps working |

**Platform layer** (portable whenever the new project's backend follows the same Express contract
in [backend-guide/03-auth/authentication.md §0](../../backend-guide/03-auth/authentication.md)):

| Path | What |
|---|---|
| `src/lib/http.ts` | `apiFetch`, the only file that knows Express exists |
| `src/lib/session.ts` | `getSession()` |

## What a Project Changes

| Change | Where | Nothing else is touched |
|---|---|---|
| Brand colors, radius | `src/styles/theme.css` (values only, never names) | ✅ |
| Fonts, language, direction | `src/app/layout.tsx` (`next/font`, `lang`, `dir="rtl"`) | ✅ |
| Logo, favicon, images | `public/` | ✅ |
| Backend address | `EXPRESS_API_URL` env variable | ✅ |
| All product text | the project's `features/` and `app/` (the kit has none) | ✅ |

**The test of the whole system:** if a screen cannot be rebranded by editing `theme.css` alone,
something is hard-coded. Fix the component, never work around it in the theme.

## What Is Never Copied

`src/features/**`, `src/app/**` routes and pages, `src/actions/**`: that is the product, not the
kit.

## New-Project Checklist

1. Copy the kit files (and the platform layer, if the backend contract matches).
2. Install the kit's dependencies: `next-themes`, `class-variance-authority`, `clsx`,
   `tailwind-merge`, `tw-animate-css`, the Radix packages the copied components import, and the
   icon library. Vet them per [dependencies](../04-quality/dependencies.md).
3. Replace the values in `theme.css`. Check **every** color pair for 4.5:1 contrast in **both**
   light and dark.
4. Set the fonts, `lang` and `dir` in `app/layout.tsx`.
5. Replace `public/` assets. Set `EXPRESS_API_URL`.
6. Copy `scripts/check-kit.sh` and the ESLint architecture rules too, then run `npm run check`. It
   must pass.
7. Open one page that uses every kit component and check it in light / dark / system, and in LTR
   and RTL.

## Kit-Purity Checks

`scripts/check-kit.sh` runs every check, and `npm run check` runs the script
([initialization](../01-foundation/initialization.md)). Any finding is a component that will not
survive a rebrand, and it fails the check. Imports that reach into the product or a database are
blocked by ESLint, so they are not repeated here.

```bash
#!/usr/bin/env bash
# Kit purity: fails if any check finds something in the portable kit. Run by `npm run check`.
# Imports (features/, app/, lib/http, databases) are enforced by ESLint, not here.
DIR=src/components/ui
FAIL=0
command -v grep >/dev/null 2>&1 || { echo "KIT ERROR: grep missing" >&2; exit 2; }
[ -d "$DIR" ] || { echo "KIT ERROR: missing $DIR (create it during setup)" >&2; exit 2; }
check() {
  local name="$1" pattern="$2" out status
  if out=$(grep -rnE -- "$pattern" "$DIR" 2>&1); then
    echo "KIT FAIL — $name"; echo "$out"; FAIL=1
  else
    status=$?
    if [ "$status" -ne 1 ]; then
      echo "KIT ERROR — $name: $out" >&2; FAIL=1
    fi
  fi
}
# variant selectors like data-[state=open]: are allowed — they end in ':'
check "raw color or arbitrary value"  '#[0-9a-fA-F]{3,8}\b|-\[[^]]+\]([^:]|$)'
check "hue palette class"             '\b(bg|text|border|ring|fill|stroke)-(slate|gray|zinc|neutral|stone|red|orange|amber|yellow|lime|green|emerald|teal|cyan|sky|blue|indigo|violet|purple|fuchsia|pink|rose|black|white)\b'
check "dark: color override"          'dark:(bg|text|border)-'
check "physical left/right (RTL)"     '\b(pl|pr|ml|mr|left|right|rounded-l|rounded-r)-|\b(border-l|border-r|rounded-l|rounded-r|text-left|text-right)\b'
check "raw z-index number"            '\bz-[0-9]+'
check "env variable in the kit"       'process\.env'
[ "$FAIL" -eq 0 ] && echo "kit purity: OK"
exit "$FAIL"
```

**Centering is the one physical-direction trap.** `left-1/2 -translate-x-1/2` centers, but its
logical twin `inset-s-1/2` moves the wrong way in RTL. Center without a side instead:
`fixed inset-0 m-auto h-fit` for overlays, `mx-auto` or `grid place-items-center` elsewhere.
Then the RTL check stays clean.

## Keeping Kits From Drifting

The template project is the kit's **single source**. A fix or a new component made while building
a product is **copied back to the template** in the same week, following
[components](./components.md). Otherwise every project ends up with its own slightly
different Button, and the kit stops being portable.
