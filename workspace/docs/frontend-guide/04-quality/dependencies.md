Every dependency is an attack surface and a bundle cost. The default answer to "should we add a dependency?" is no. This file covers how to evaluate, manage, and maintain dependencies in production frontend projects.

Scope: follow [frontend-only contract authority](../00-navigation/frontend-only-scope.md). Endpoint names, payloads and session settings in examples apply only when the actual owner-supplied contract matches; examples do not define external API behavior.

## Before Adding a Dependency

Ask four questions in order:

1. **Can we do this with built-in APIs?** Modern browsers and Node.js cover URL parsing, date formatting (with `Intl`), base64 encoding, deep cloning (`structuredClone`), event delegation, and much more. React 19 provides `use`, `useOptimistic`, `useFormStatus`, and `useActionState` — many libraries that wrapped these patterns are now unnecessary.
2. **Is the maintenance activity healthy?** A library with no commits in 12 months is a risk. A library with 500 open issues and one maintainer is a risk.
3. **What's the bundle impact?** Check the gzipped size. Anything over 10KB should justify itself clearly.
4. **How many transitive dependencies does it bring?** Run `npm ls <package>` and inspect the tree. A 5KB library that pulls in 50 transitive dependencies is more expensive than it looks.

## Evaluating a Dependency

Check these signals before adding any package:

- **npm weekly downloads**: Current numbers matter more than total. A package with 2M weekly downloads in 2026 is actively used. A package that peaked in 2021 and declined is not.
- **Last publish date**: Within the last 6 months for active projects. Security patches should come within days of a CVE.
- **Open issues count**: High open issue count relative to the codebase size suggests the maintainer is overwhelmed or unresponsive.
- **Maintainer responsiveness**: Check if recent issues get responses. Pull requests being merged is a stronger signal than issues being closed.
- **GitHub stars**: A weak signal. Popular but unmaintained packages have high stars. Use it only as a tiebreaker.
- **Security audit**: Run `npm audit` after installing. Zero critical or high vulnerabilities is the minimum bar.
- **Bundle size**: Use `bundlephobia.com` or `pkg-size.dev` to check the gzipped size and tree-shaking support. Prefer ESM packages with named exports.

## Supply Chain Security

The September 2025 Shai-Hulud worm (self-propagating npm packages that stole credentials and published them) and the November 2025 hijacking of maintained packages like `cacheable`/`keyv` made one thing clear: provenance verification is no longer optional.

- **npm provenance**: Verify packages have Sigstore attestations. Check the "provenance" badge on npm. Packages built from verified source are significantly harder to tamper with.
- **Lockfile integrity**: Always use `npm ci`, never `npm install`, in CI and production builds. `npm ci` installs exactly what's in the lockfile and fails if the lockfile doesn't match `package.json`.
- **`npm audit` in CI**: Run as a CI step. Fail the build on critical or high vulnerabilities. Auto-fix with `npm audit fix` only after reviewing the changes — auto-fixes can introduce breaking changes.
- **Minimize dependencies**: Fewer packages means a smaller attack surface. This is the single most effective supply chain defense.

## When to Remove a Dependency

Remove a dependency when any of these apply:

- The built-in API now suffices (e.g., remove `date-fns` if `Intl.DateTimeFormat` covers your needs, remove a classname dependency only after checking cn() no longer imports it; the canonical cn() uses both clsx and tailwind-merge).
- The library is unmaintained (no release in 12+ months, unmerged PRs, no response to issues).
- The bundle cost outweighs the value (a 30KB library for a utility you use in one place).
- A critical vulnerability exists with no fix available.

Removal process: delete the import, delete the package with `npm uninstall`, verify the build passes, check bundle size impact.

## Version Pinning

Use lockfiles always. They are your source of truth for exact versions. In CI, `npm ci` enforces lockfile consistency.

In `package.json`, avoid loose version ranges (`^`, `~`) for critical dependencies like the framework, build tools, and security-related packages. For application-level dependencies, the lockfile is your guardrail, but prefer exact versions or tight ranges when possible. Never commit a `package.json` change without also committing the updated lockfile.

## Upgrade Strategy

Upgrades should be regular, not reactive:

- **Cadence**: Monthly for minor/patch versions. Quarterly for major versions. Don't let dependencies drift for months — the upgrade diff grows and the risk of breaking changes increases.
- **Process**: Check the changelog before upgrading. Run the test suite after upgrading. If tests fail, investigate before committing — don't just update expected values.
- **Major version upgrades**: Isolate in their own branch. Test thoroughly. Major React, Next.js, or TypeScript upgrades may require code changes documented in their migration guides.

## Hallucinated Packages

AI coding assistants sometimes recommend non-existent packages. Always verify a package exists on npm before adding it. Run `npm info <package>` or check npmjs.com. A package that doesn't exist will cause a silent install failure or, worse, a typo-based install of a malicious lookalike.

## Decision Summary

| Decision | Rule |
|---|---|
| Add a dependency? | Default to no. Prove you need it. |
| Evaluate | Check downloads, publish date, issues, bundle size, provenance |
| Install in CI | `npm ci`, never `npm install` |
| Security | `npm audit` in CI, require provenance for new packages |
| Remove | When built-in APIs suffice, unmaintained, or too heavy |
| Upgrade | Monthly patches, quarterly majors |
| AI-recommended package | Verify on npm before installing |