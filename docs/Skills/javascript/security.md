# JavaScript Security

## DOM XSS

### Dangerous sinks — never pass untrusted data
```js
element.innerHTML = userInput;        // ✗
element.outerHTML = userInput;        // ✗
document.write(userInput);            // ✗
element.insertAdjacentHTML('...', userInput); // ✗
```

### Safe alternatives
```js
element.textContent = userInput;      // ✓
element.appendChild(document.createTextNode(userInput)); // ✓
element.innerHTML = DOMPurify.sanitize(userInput);       // ✓ if HTML needed
```

### Dangerous sources
URL parameters, `document.referrer`, `document.cookie`, `localStorage`, `postMessage`, form inputs, third-party API responses.

### Script injection via `src`/`href`
```js
img.src = userInput;  // ✗ javascript: URI
a.href = userInput;   // ✗ javascript: URI

// ✓ Validate
function isSafeUrl(url) {
  try {
    const p = new URL(url);
    return p.protocol === 'https:' || p.protocol === 'http:';
  } catch {
    return false; // relative URL — decide per context
  }
}
```

## Prototype Pollution

### How it works — precisely

`JSON.parse` creates `__proto__` as an ordinary **own data property**; it does not set
a prototype. Nothing is polluted yet. The damage happens when that value is copied
somewhere using `[[Set]]`, which triggers the `__proto__` accessor. Verified in Node:

```js
const parsed = JSON.parse('{"__proto__": {"isAdmin": true}}');
Object.hasOwn(parsed, '__proto__');            // true — a plain key, inert so far

const a = Object.assign({}, parsed);
Object.getPrototypeOf(a) !== Object.prototype; // true  — a's prototype was replaced
Object.prototype.isAdmin;                      // undefined — global prototype is intact

const s = { ...parsed };
Object.getPrototypeOf(s) === Object.prototype;  // true — spread uses [[DefineOwnProperty]]
Object.hasOwn(s, '__proto__');                  // true — the key survives as data
```

**Two distinct failures, don't conflate them:**

1. `Object.assign` / manual `target[key] = value` on one level → replaces **that object's**
   prototype. Bad, but contained.
2. A **recursive** deep merge that walks into `__proto__` → reaches `Object.prototype`
   and poisons every object in the process. This is the real prototype-pollution CVE class.

Note that spread is safe only for the object it produces — the `__proto__` key rides
along as data and becomes dangerous again if that object is later deep-merged.

### Dangerous merge operations
```js
Object.assign(target, source);      // ✗ single-level prototype swap
_.merge(target, source);            // ✗ recursive — the classic vector (Lodash <4.17.12)
_.defaultsDeep(target, source);     // ✗ same
```

**Structural fix**: parse untrusted input into a schema (zod/valibot) instead of merging
it. A validated object has only the keys you declared, so there is no `__proto__` to walk.

### Safe alternatives
```js
structuredClone(source);           // ✓ No merge, but safe
{ ...source };                     // ✓
Object.fromEntries(Object.entries(source)); // ✓
```

### Safe deep merge
```js
function safeMerge(target, source) {
  for (const [key, value] of Object.entries(source)) {
    if (key === '__proto__' || key === 'constructor' || key === 'prototype') continue;
    if (value && typeof value === 'object' && !Array.isArray(value)) {
      target[key] = target[key] || {};
      safeMerge(target[key], value);
    } else {
      target[key] = value;
    }
  }
  return target;
}
```

## ReDoS (Regular Expression DoS)

### Vulnerable patterns
```js
/(a+)+$/        // ✗ Nested quantifiers
/(a|a)+$/       // ✗ Overlapping alternatives
/(a*)*b/        // ✗ Nested with nothing after
```

### Detection rules
- Nested quantifiers: `(x+)+`, `(x*)*`, `(x+)*`
- Overlapping alternations with quantifiers: `(a|ab)+`
- Quantifier on same char twice: `a+a+`

### Mitigation
```js
/^[a-z]+$/          // ✓ Linear time
/^(\w+\s?)+$/       // ✗ Still nested — restructure
```
**Tools:** `redos-detector` (actively maintained, 2026) or `recheck` in CI.
`safe-regex` is still widely cited but was last published in **2019** — do not add it
to a new project.

**Structural fix:** for user-supplied patterns, don't use `RegExp` at all. Use RE2
(`node-re2`), which has no backtracking and therefore no ReDoS by construction. Where
you build patterns from user *values* (not user patterns), `RegExp.escape()` is the
right tool → [modern-features.md](./modern-features.md).

## `postMessage` security

```js
// ✗ No origin check
window.addEventListener('message', (e) => { processData(e.data); });

// ✓ Always verify origin
window.addEventListener('message', (e) => {
  if (e.origin !== 'https://trusted-domain.com') return;
  processData(e.data);
});
```

## Defense in depth (browser)

Sanitizing at each sink is necessary but not sufficient — one missed sink is an XSS.

```js
// Trusted Types: make DOM sinks refuse raw strings entirely (Chromium)
// Content-Security-Policy: require-trusted-types-for 'script'
const policy = trustedTypes.createPolicy('app', {
  createHTML: (s) => DOMPurify.sanitize(s),
});
el.innerHTML = policy.createHTML(userInput);   // a bare string now throws
```

```js
// Element.setHTML() — sanitizer built into the platform, no library
el.setHTML(userInput);            // MDN: "Limited availability" — NOT Baseline
```

- `setHTML()` is the right long-term answer but is not yet cross-browser — keep
  DOMPurify as the portable path and feature-detect (`typeof el.setHTML === 'function'`).
- **CSP** with `script-src` (no `unsafe-inline`/`unsafe-eval`) is the backstop that turns
  a missed sink from "account takeover" into "blocked script".
- Trusted Types converts every unsanitized `innerHTML` into a **runtime error you can
  see in CI**, rather than a silent vulnerability.

## Deserialization

`JSON.parse` alone is safe — it produces only primitives, arrays, and plain objects. Cannot instantiate classes or execute code.

Reviver functions can be dangerous if they call constructors with attacker-controlled values. Avoid complex revivers on untrusted input.

## Supply chain

- **Commit the lockfile; install with `npm ci`** — `npm install` can silently resolve
  to a newer, compromised patch version.
- Run `npm audit` / `pnpm audit` in CI and fail the build on high/critical.
- `postinstall` scripts run arbitrary code at install time and are the main vector for
  the 2024–2026 npm account-takeover attacks. Audit with `--ignore-scripts`, and prefer
  pnpm's default of blocking lifecycle scripts until explicitly allowed.
- Pin exact versions for anything security-sensitive; use Renovate/Dependabot to move
  deliberately rather than by floating range.
- Prefer platform built-ins over a dependency. Every removed package is one fewer
  maintainer account that can be phished → [modern-features.md](./modern-features.md).
