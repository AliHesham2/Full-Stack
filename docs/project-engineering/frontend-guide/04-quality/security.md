## Foundational Principle: Frontend Is Not a Security Boundary

Frontend security is defense in depth, not enforcement. Authorization checks, rate limiting, server-side input validation, and business logic guards all belong on the backend. A compromised or modified client can bypass any frontend restriction. Build frontend security measures to reduce attack surface and raise the cost of exploitation — never as the sole control. This distinction matters because AI-generated code ships with security flaws at a high rate,
and the most common class of them is exactly this mistake: treating client-side logic as
authoritative. Various 2025–2026 studies put the flawed-sample rate somewhere around a third to
a half depending on language and task; the number moves with methodology, so **do not quote a
specific percentage** — the architectural conclusion does not depend on it and a precise-looking
figure invites false confidence.

## XSS Prevention: Three Layers

**Layer 1 — Trusted Types (Baseline Newly Available, February 2026).** Deploy `require-trusted-types 'script'` via Content Security Policy. Supported in Chrome/Edge (since 83), Firefox 148+, and Safari 26+; older browsers ignore the enforcement directive, so deployment degrades gracefully. Trusted Types prevents DOM-based XSS by requiring all dangerous sink assignments (innerHTML, outerHTML, eval, document.write) to go through an approved policy. This is the single most impactful XSS mitigation available in 2026.

**Layer 2 — DOMPurify as the Trusted Types policy.** DOMPurify implements the sanitization policy that Trusted Types requires. Keep it current — CVEs in 2025 (mXSS via namespace confusion) and 2026 (mutation XSS in SVG attributes) demonstrate that sanitizers are not set-and-forget. Pin a version range with automated Dependabot/Socket alerts, and audit upgrades.

**Layer 3 — Framework auto-escaping.** React, Vue, and Svelte all escape interpolated values by default. This is necessary but not sufficient — it only covers template rendering, not DOM API calls. Never use `dangerouslySetInnerHTML` (React), `v-html` (Vue), or equivalent with user-controlled content. If you must render rich content, route it through DOMPurify's Trusted Types policy, never raw.

## Content Security Policy

Use nonce-based CSP for script loading:

```
script-src 'nonce-{random}' 'strict-dynamic'
```

Generate a cryptographically random nonce per request on the server and inject it into both the CSP header and script tags. `strict-dynamic` allows scripts loaded by trusted (nonced) scripts to execute without listing every third-party domain.

Never use `unsafe-inline` in `script-src`. Large-scale CSP crawls have consistently found
that roughly half of deployed policies still include it — a misconfiguration that renders the
whole policy ineffective against XSS, because the attacker's injected inline script is then
explicitly permitted. Similarly, never use `unsafe-eval`.

Note that `'strict-dynamic'` **causes browsers that support it to ignore** `unsafe-inline`
and host/scheme allowlists — which is why the nonce + `strict-dynamic` combination above is
the recommended shape: older browsers fall back to the allowlist, modern ones enforce the
nonce.

For reporting, use the `report-to` directive with the Reporting API (the legacy `report-uri` directive is deprecated). Start in report-only mode (`Content-Security-Policy-Report-Only` header, no enforcement) to catch violations, then move to enforcement after a burn-in period. See [backend-integration.md](../03-data/backend-integration.md) for CSP header implementation patterns.

## Token Storage and Authentication

**HttpOnly + Secure + SameSite cookies.** This is settled in 2026. Never store tokens in localStorage or sessionStorage — they are accessible to any JavaScript on the page, making XSS trivially exploitable for token theft. Cookies with HttpOnly prevent JS access, Secure prevents plaintext transmission, and SameSite=Lax or Strict mitigates CSRF.

**OAuth 2.1: Authorization Code + PKCE.** The Implicit flow is dead (removed from OAuth 2.1). Use the authorization code flow with PKCE for all client types, including single-page apps. The backend should exchange the code for tokens and set the HttpOnly cookie.

## No Database or Supabase SDK in Next.js — Ever, Not Even Server-Side

This project's architecture ([backend-integration](../03-data/backend-integration.md)) puts Express
between Next.js and Supabase. That boundary only holds if it is also a secrets boundary:

- **Never install or import a Supabase (or any database) client in the Next.js codebase.** Not
  `@supabase/supabase-js`, not `@supabase/ssr`, not an ORM pointed at the same Postgres instance.
  If it can open a connection to the database, it does not belong here — including in a Route
  Handler, a Server Action, or a Server Component. Only Express holds that dependency.
- **No Supabase key — anon or service role — exists anywhere in the Next.js environment.** Not
  as `NEXT_PUBLIC_*` (obviously client-exposed) and not as a server-only variable either: a
  server-only Supabase key in the Next.js process is still a second place the credential lives,
  which is exactly the "one point of change" guarantee this architecture exists to protect. The
  only backend URL Next.js knows is `EXPRESS_API_URL`.
- **This includes login.** The session cookie Next.js sets on the browser is issued by Next.js
  itself after Express confirms the credentials — see the Authentication flow in
  [backend-integration](../03-data/backend-integration.md). Next.js never calls
  `supabase.auth.signInWithPassword()` or any Supabase Auth method directly.
- If a future feature seems to need direct Supabase access from Next.js (realtime subscriptions,
  storage downloads), use an Express endpoint relayed through Next.js; do not return a provider URL — never an exception
  to this rule.

## CSRF Protection

SameSite=Lax (or Strict) cookies provide baseline CSRF protection. For additional defense, require a custom header (e.g., `X-Requested-With: XMLHttpRequest`) on
all API mutations. The mechanism is worth stating precisely, because "browsers strip it" is not
what happens: an HTML form submission **cannot set custom headers at all**, and a cross-origin
`fetch`/XHR that sets one is no longer a simple request — it triggers a **CORS preflight**,
which your server refuses. So the header cannot be forged from another origin. This is simpler
and more reliable than CSRF tokens for SPAs, and it only works if the server actually **rejects
requests missing the header** — the check is the protection, not the header.

## Third-Party Script Risk

Third-party scripts (analytics, chat widgets, ad pixels) are the largest real-world XSS vector. Mitigate with three controls:

1. **CSP** — nonced scripts only; third-party scripts must be loaded via a trusted script loader or iframe.
2. **Subresource Integrity (SRI)** — pin hashes for any third-party script tags you include directly.
3. **Permissions Policy** — restrict access to sensitive APIs (`geolocation`, `camera`, `microphone`, `payment`).

For untrusted widgets, isolate in iframes with the `sandbox` attribute. Use `sandbox="allow-scripts"` and nothing else unless explicitly needed. This prevents the widget from accessing the parent page's cookies, DOM, and storage.

## Supply Chain Security

The Shai-Hulud and keyv/cacheable attacks demonstrated self-propagating worms in the npm ecosystem. Countermeasures:

- **Require npm provenance** for all direct dependencies. Reject packages without provenance in CI.
- **Lockfile integrity** — always commit `package-lock.json` (or equivalent). Use `npm ci` in CI, never `npm install`.
- **Minimize dependencies** — every package is an attack surface. Audit regularly with `npm audit` and automated tools (Socket, Snyk).
- **Pin transitive critical dependencies** using npm overrides when necessary.
- **Never run postinstall scripts from untrusted packages** — use `--ignore-scripts` in CI and selectively enable only for known-safe packages.

## Security Headers

Deploy these headers on every response:

- **Content-Security-Policy** — nonce-based, as described above
- **Strict-Transport-Security** — `max-age=63072000; includeSubDomains; preload`
- **X-Content-Type-Options: nosniff** — prevents MIME type sniffing
- **X-Frame-Options: DENY** — prevents clickjacking (superseded by CSP `frame-ancestors`, but keep both for compatibility)
- **Referrer-Policy: strict-origin-when-cross-origin** — limits data leaked via the Referer header to other sites
- **Permissions-Policy** — restrict browser feature access
- **Cross-Origin-Opener-Policy: same-origin** — isolates your browsing context from windows
  you open. Safe to deploy broadly; the one thing to check is OAuth popups that rely on
  `window.opener` (use `same-origin-allow-popups` if you have them).
- **Cross-Origin-Resource-Policy: same-origin** — on *your own* resources, stops other sites
  embedding them. Safe.

### Do not add `Cross-Origin-Embedder-Policy: require-corp` by default

It belongs in a different category from every other header on this list. The rest harden your
site; **COEP changes what your site is allowed to load.** Under `require-corp`, every
cross-origin resource — images, fonts, scripts, iframes, embeds — must send
`Cross-Origin-Resource-Policy` or proper CORS headers, or the browser blocks it outright. Most
third-party assets do not, so enabling it blanket-style typically breaks analytics, embedded
video, map tiles and CDN images all at once.

COOP + COEP together enable **cross-origin isolation**, which you need only for
`SharedArrayBuffer`, `performance.measureUserAgentSpecificMemory()`, and unthrottled
high-resolution timers. **If you do not use those, you do not need COEP.**

If you do need it: roll out with `Cross-Origin-Embedder-Policy-Report-Only` first, fix or
proxy every resource it reports, then enforce. Verify with `self.crossOriginIsolated`.
See [performance.md](./performance.md).

## Common AI-Generated Security Mistakes

When generating or reviewing code, flag these patterns immediately:

- **JWT in localStorage** — always use HttpOnly cookies instead
- **Missing HttpOnly, Secure, or SameSite attributes** on auth cookies
- **Importing a Supabase (or any database) client SDK anywhere in the Next.js codebase** — even
  server-side, even for "just reading" — see [No Database or Supabase SDK in Next.js](#no-database-or-supabase-sdk-in-nextjs--ever-not-even-server-side) above
- **A Client Component calling Express directly** — every request goes through a Next.js Server Action or Route Handler; the browser only ever talks to Next.js
- **`dangerouslySetInnerHTML` without DOMPurify sanitization** — or any unsanitized HTML rendering
- **`Access-Control-Allow-Origin: *`** with credentials — browsers reject this anyway, but it signals misunderstanding of CORS
- **Hardcoded API keys or secrets** in client code — never acceptable
- **Missing security headers** — CSP, HSTS, and nosniff are non-negotiable
- **Client-side-only authorization checks** — the backend must enforce all access control
- **Using `eval()`, `new Function()`, or `setTimeout(string)`** — these are blocked by Trusted Types for good reason

## Secrets Management

No secrets in client code. Environment variables prefixed with `NEXT_PUBLIC_`, `VITE_`, or `REACT_APP_` are embedded in the production bundle. Use server-side environment variables or a secret management service (Vault, AWS Secrets Manager, Cloudflare Secrets) for any sensitive value. If a value must reach the client, it is not a secret — design your architecture accordingly.

**Supabase keys are not a Next.js secret — they are not a Next.js value at all.** Server-side
`.env` variables are still a place a credential lives; this project's rule is that only Express's
environment holds Supabase keys. Next.js's entire backend-facing environment surface is
`EXPRESS_API_URL` and, only when needed, the server-only `EXPRESS_SERVICE_KEY` for service authentication. See [No Database or Supabase SDK in Next.js](#no-database-or-supabase-sdk-in-nextjs--ever-not-even-server-side).
