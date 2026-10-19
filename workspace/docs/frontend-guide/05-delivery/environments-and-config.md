# Environments and Configuration

Configuration determines what code runs where, with which values. Misconfigured environments leak secrets, ship dev settings to production, or break builds that worked locally. This file defines how to separate environments, where configuration lives, and when to use feature flags.

## Environment Separation

Three environments minimum for any real project:

| Environment | Purpose | Data | Deployed from |
|---|---|---|---|
| **Local** (dev) | Feature development | Seeded/mock data, never production data | n/a |
| **Preview/Staging** | Validate branches and releases; mirrors production configuration | Production-like, isolated | Every pull request / main |
| **Production** | Real users | Real data | Main after verification |

**Rules:**

- Staging must differ from production in *data only* — same environment variables structure, same services, same build process. A staging environment that behaves differently from production validates nothing.
- Never point staging at production databases or APIs, even read-only. "Just checking something" against production data is how incidents happen.
- Local development uses `.env.local` (gitignored) with values documented in `.env.example` (committed). If a new variable is required and missing locally, the app should fail loudly at startup, not silently at runtime.

See [delivery-and-observability.md](./delivery-and-observability.md) for the pipeline that promotes builds between these environments.

## Configuration Boundaries: Build-Time vs Runtime

**Build-time configuration** is embedded into the bundle. In Next.js, NEXT_PUBLIC_* and next.config.ts env values are embedded at build time. Importing a server-only module does not inline every env read; static prerendering can still capture a value during build. Read deployment-varying values on a dynamic server request path and verify the promoted artifact with changed runtime env.

**Runtime configuration** is read on the server per request/process: server-only env vars (`process.env.SECRET` inside Server Actions/Components), the Express API URL, third-party API keys. Never a database URL or Supabase key — those live only in Express's environment ([security.md](../04-quality/security.md)).

Decision rule:

- Does the value need to change without a rebuild? → Runtime (server-side).
- Is it needed in the browser? → It will be public. `NEXT_PUBLIC_` is a promise to the world, not a setting.
- Could it cause harm if exposed? → Server-side only, referenced only from Server Components, Actions, or route handlers.

**Never** put secrets behind public prefixes. See [security.md](../04-quality/security.md) — if a value must reach the client, it is not a secret by definition.

A useful pattern: validate required server env vars at startup with one schema so a misconfigured deploy fails immediately:

```ts
// lib/env.ts — server-only
import { z } from 'zod';

const schema = z.object({
  EXPRESS_API_URL: z.string().url(),   // the only backend Next.js knows — no DATABASE_URL, no Supabase keys
});

export const env = schema.parse(process.env);
```

Importing this module in server code makes every deploy self-checking: missing variables crash the boot, not the first user request.

## Environment Variables Hygiene

- One source of truth: `.env.example` lists every required variable with a placeholder and a comment explaining what it does. Update it in the same PR that introduces the variable.
- Prefix conventions are enforced by the framework (`NEXT_PUBLIC_`, `VITE_`) — treat them as security boundaries, not naming style.
- Never commit real values. If a secret lands in git history, rotate it; deleting the file is not enough.
- CI reads secrets from the platform's secret store (GitHub Actions secrets, Vercel project env vars), never from committed files.

## Feature Flags

Feature flags decouple deployment from release: code ships dark, then activates without a redeploy.

**Use a flag when:** enabling a risky feature gradually (percentage rollout), keeping an unfinished feature mergeable behind a flag, or needing an emergency kill switch.

**Do not use flags when:** a plain config value suffices (API base URL), the feature is simply incomplete — finish it on a branch instead, or the flag would outlive its purpose indefinitely. Flag debt is real debt: schedule removal when the feature is fully rolled out.

**Implementation tiers:**

1. **Server-side boolean check** (`process.env.FEATURE_X === 'true'` read in a Server Component) — sufficient for most projects; changes require redeploy but no client cost.
2. **Database/hosted flag service** (Flagsmith, PostHog, LaunchDarkly) — when non-developers control rollouts or you need percentage targeting.
3. Client-side flags add bundle size and expose your roadmap — avoid unless personalization requires them.

Prefer server-evaluated flags evaluated in Server Components or proxy.ts ([initialization](../01-foundation/initialization.md)) so flagged UI never flashes on the client.

## What NOT to Do

- **Config sprawl**: dozens of one-off env vars for values that are actually constants. If it doesn't change between environments, it's code, not config.
- **Runtime reads of build-time values**: reading `NEXT_PUBLIC_*` in a Server Action and expecting a different staging value than the one baked into the build.
- **Environment detection by hostname hacks** (`window.location.includes('staging')`). Use explicit env vars.
- **Secrets in preview deployments shared across teams** without access control — preview envs inherit real env vars by default on most platforms; audit this.
