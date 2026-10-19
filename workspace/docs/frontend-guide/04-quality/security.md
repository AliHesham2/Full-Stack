# Frontend security

Follow [frontend-only scope](../00-navigation/frontend-only-scope.md), [API integration](../03-data/backend-integration.md), supplied owner contracts and the approved software/security plan. These are frontend controls, not a backend implementation assignment. Primary-source checks below inform this adaptation; they do not certify the preserved syntax collection or every package version as latest.

## Foundational Principle: Browser UI Is Not Authorization

A hidden button, disabled input, client validation or redirect cannot enforce external service permissions. The external service owns its authorization/business/data integrity. Next.js actions/routes still expose server surfaces that must validate untrusted requests, protect credentials and enforce their approved transport permissions. Do not relay privileged service credentials on behalf of an unchecked caller.

Exported Server Actions are public endpoints: validate inputs and check the caller for the approved operation, rather than relying on the page that renders a form. Keep private modules server-only and return only necessary safe DTO fields to Client Components. No client prop, action argument or client-provided tenant/role flag is trusted identity. Follow the project's external-HTTP approach; generic documentation examples using a database do not authorize one here. [Next.js data security](https://nextjs.org/docs/app/guides/data-security).

## XSS Prevention: Escaping, Safe Sinks and Sanitization

Prefer React's ordinary escaped text rendering. Avoid raw HTML, dynamic script execution and unsafe URL/DOM sinks. If approved requirements need rich untrusted HTML, use a maintained, compatible sanitizer with an explicit allowlist; never treat escaped text as safe for every JavaScript/CSS/URL context. Validate URL schemes and destinations separately. CSP is additional protection, not a substitute for safe rendering. [OWASP XSS prevention](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html).

Trusted Types may be an additional defense where the supported browser/toolchain matrix permits it. Verify actual compatibility and integration before policy enforcement; do not assert universal browser support, a dated release cutoff or that a library is safe merely because its name appears in a guide. Any implementation API must also be covered by local syntax.

## Content Security Policy

Plan CSP for the actual rendering/deployment model and approved asset/script origins. A strict nonce or hash policy is preferable to permissive script execution. A nonce must be unpredictable and fresh for each response, consistently included in the header and permitted script tags; do not cache/reuse it incorrectly. Understand strict-dynamic trust propagation and avoid unapproved script loaders or broad wildcards. Do not add unsafe-inline/unsafe-eval just to hide a violation.

Use report-only observation to discover integration failures before enforcement, with safe reporting/privacy settings. Report-only is not protection against blocked execution; record actual rollout state. Choose compatible reporting directives rather than claiming one works in every target browser. [OWASP CSP](https://cheatsheetseries.owasp.org/cheatsheets/Content_Security_Policy_Cheat_Sheet.html).

## Token Storage and Authentication

Follow the owner's complete authentication/session contract; do not invent login paths, token shape, lifetime or provider refresh. Credentials remain in the approved server-only transport. Do not store session credentials in localStorage/sessionStorage or return them to Client Components/logs. When the approved flow uses Next.js cookies, specify HttpOnly, production HTTPS/Secure, appropriate SameSite, path/domain and expiry consistent with that contract. Missing or incompatible requirements block the affected flow for owner resolution.

OAuth code/PKCE, callback state, safe redirects and one-time handoffs require an actual supplied flow and supported local syntax. No auth-provider SDK or token service is added here. Distinguish unauthorized/expired sessions from operational failures; do not erase a valid session because the API timed out.

## No Database or Provider SDK in Next.js

No database/ORM, SQL, migrations or direct external storage/auth provider implementation. Database URLs and provider credentials never belong in this frontend, even server-only. This boundary does not select a vendor for the external API. A needed missing capability returns to the owner; it does not justify adding a second backend. See [integration](../03-data/backend-integration.md).

## CSRF Protection

Select controls for the actual cookie/identity and deployment boundaries. SameSite cookies are defense in depth, not a universal complete CSRF solution; same-site is broader than same-origin. Do not perform state changes on GET. Protect custom mutation Route Handlers as well as actions, with an explicit origin/token or appropriately constrained custom-header policy.

A custom header protects only when the server requires it and untrusted origins cannot obtain a permissive credentialed CORS response. Wildcard/suffix trust in subdomains can undermine that assumption; a header alone is no guarantee. Validate token/origin checks on real allowed/denied scenarios. [OWASP CSRF prevention](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html).

Next.js has Server Action origin/host protections, but they do not replace input/caller checks. Verify reverse-proxy forwarded-host/origin behavior and any allowedOrigins configuration for the actual approved version/deployment; never loosen it broadly to make a form work. [Next.js Server Actions configuration](https://nextjs.org/docs/app/api-reference/config/next-config-js/serverActions).

## Outbound Requests, Uploads and Downloads

Only approved fixed API origins/operation paths are reachable through the server adapter. Validate parameters, avoid arbitrary URL/header/cookie forwarding and assess redirects before following them. Credentials must not follow an unapproved destination. Restricting the initial URL alone is insufficient if redirects can escape the approved boundary. [OWASP SSRF prevention](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html).

Apply approved ingress/body/time limits and bounded streaming rules; a browser MIME or Content-Length assertion is untrusted. External content/ownership/storage enforcement remains the service's responsibility. Do not expose provider credentials or arbitrary upstream content as executable same-origin HTML. Follow supplied contracts and host limitations; missing guarantees remain questions.

## Third-Party Script Risk

Inventory approved analytics/widgets/fonts/assets, purposes, recipients and privacy requirements. Keep unnecessary scripts out. Apply CSP, appropriate integrity controls for fixed assets and restrictive Permissions Policy. Isolate untrusted widgets where feasible; sandbox permissions must be individually justified. SRI and sandboxing do not replace reviewing what an allowed script can access. No new script/provider is approved by this guide.

## Supply Chain Security

Use approved compatible dependencies and committed lockfiles; inspect actual package/tool scripts before execution. Reproducible CI uses the actual frozen-lock install command. Review advisories and provenance where available, dependency necessity, update compatibility and install scripts. Missing provenance is a risk signal requiring project policy, not a fabricated universal guarantee of compromise. Never blindly disable scripts needed by approved build tooling or install replacements to silence a failure. Dependency changes need actual task authorization and syntax support.

## Security Headers

Define and verify applicable CSP, nosniff, framing protection through CSP frame-ancestors and compatible X-Frame-Options, Referrer-Policy, appropriate Permissions-Policy and HTTPS/HSTS for the actual deployment. HSTS subdomain/preload scope has operational consequences: do not copy a max-age/includeSubDomains/preload policy without confirming ownership and readiness. Authenticated/private responses need explicit cache handling. Inspect actual response headers; configuration text alone is not proof of deployment. [OWASP HTTP headers](https://cheatsheetseries.owasp.org/cheatsheets/HTTP_Headers_Cheat_Sheet.html).

## Secrets, Errors, Logs and Personal Data

Only approved API credentials remain server-only, with environment names and .env.example placeholders documented; no real values in artifacts. NEXT_PUBLIC values are public. Review props, action outputs, HTML, caches, logs, source maps and error reporting for unintended disclosure. Raw upstream errors/bodies are not safe UI messages. Use the approved normalized frontend errors with redacted diagnostics/correlation where permitted.

User-specific caches cannot be shared across users/tenants. Log only necessary redacted events, not full requests, tokens, personal data or sensitive URLs. Consent/privacy requirements depend on the actual product, recipients and jurisdiction; do not claim legal compliance from a template.

## Verification and Escalation

Planner's Security section names threats, exact controls/locations/settings, ownership and observable negative cases. Tester verifies the frontend on its actual candidate and permitted environment; mocks cannot prove service authorization, real integration or storage correctness. Missing data/access/tools remain UNVERIFIED/BLOCKED. Orchestrator reviews every result and personally verifies corrections under the [delivery cycle](../../../agents/orchestrator/skillz/delivery-cycle.md). No scan, installation, external project access, production call or deployment is authorized merely by this guide.
