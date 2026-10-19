# Migrating Express 4 → 5

Express 4 is in **MAINTENANCE** with a TC target EOL no sooner than **2026-10-01**. Any codebase still on 4.x needs this migration, not a patch.

## Start with the codemods

```bash
npx codemod@latest @expressjs/v5-migration-recipe
npm install express@5 @types/express@5
```

The recipe covers the mechanical renames below. It does **not** understand your route strings, so path syntax and the behavioural changes still need review by hand.

Source: [Official migration guide](https://expressjs.com/en/guide/migrating-5.html)

## Removed — will throw or silently misbehave

| Express 4 | Express 5 |
|-----------|-----------|
| `app.del()` | `app.delete()` |
| `req.param(name)` | `req.params.x` / `req.body.x` / `req.query.x` |
| `res.send(body, status)` | `res.status(status).send(body)` |
| `res.send(status)` | `res.sendStatus(status)` |
| `res.json(obj, status)` | `res.status(status).json(obj)` |
| `res.redirect(url, status)` | `res.redirect(status, url)` — **argument order reversed** |
| `res.redirect('back')`, `res.location('back')` | `res.redirect(req.get('Referrer') \|\| '/')` |
| `res.sendfile()` | `res.sendFile()` |
| `req.acceptsCharset/Encoding/Language()` | pluralised: `acceptsCharsets/Encodings/Languages()` |
| `app.param(fn)`, `router.param(fn)` | Removed entirely — no replacement |
| `app.param(':name', fn)` | Leading colon is ignored; write `app.param('name', fn)` |
| `express.static.mime` | Use the `mime-types` package |
| `hidden` / `from` options | `dotfiles` / `root` (in `res.sendFile` and `express.static`) |

> **`res.redirect(url, status)` is the dangerous one** — the reversed order does not throw. `res.redirect('/login', 302)` in v5 tries to use `'/login'` as the status code.

## Behaviour changes that break working code silently

| Area | v4 | v5 | Fix |
|------|----|----|-----|
| `req.body` unparsed | `{}` | `undefined` | `const b = req.body ?? {}` |
| `req.query` | writable | read-only getter | write to `req.validatedQuery` |
| Query parser default | `extended` (nested) | `simple` (flat) | `app.set('query parser', 'extended')` if clients send `a[b]=c` |
| `express.urlencoded` `extended` | `true` | `false` | pass `{ extended: true }` if you need nested form data |
| `req.host` | port stripped | **includes port** | use `req.hostname` |
| `req.params` | normal object | null prototype | `Object.hasOwn(p, k)`, not `p.hasOwnProperty(k)` |
| Wildcard capture | `req.params[0]` string | named **array** | `/*splat` → `req.params.splat` is `['a','b']` |
| Unmatched params | `undefined` / `''` | key omitted | check with `in` / `??` |
| `express.static` dotfiles | served | `"ignore"` (404) | mount `/.well-known` explicitly with `dotfiles: 'allow'` |
| `res.clearCookie` | honoured `maxAge`/`expires` | ignores them | must match `path`/`domain`/`secure`/`httpOnly` instead |
| `res.status` | any value | integers 100–999 only, else throws | validate before passing through |
| `res.vary()` | no-op without field | throws | always pass a field |
| `app.listen` errors | thrown | passed to the callback | `app.listen(port, (err) => { if (err) throw err })` |

## Path syntax (path-to-regexp v8) — the big one

Every route string must be re-read. Full rules in [routing.md](./routing.md).

```js
// v4                          →  v5
app.get('/*', h)               →  app.get('/{*splat}', h)
app.get('/files/*', h)         →  app.get('/files/*splat', h)
app.get('/:file.:ext?', h)     →  app.get('/:file{.:ext}', h)
app.get('/user(s)?', h)        →  app.get(['/user', '/users'], h)
app.get(/^\/\d+$/, h)          →  app.get('/:id', validateNumeric, h)
```

`( ) [ ] ? + !` are reserved — a v4 route containing them throws `PathError` at registration, so **startup fails loudly**. Regex sub-expressions were removed to make routing ReDoS-proof, not as a style choice; do not reintroduce them via `RegExp` route objects.

## What you gain

- **Async errors auto-forward** — delete every `asyncHandler`/`express-async-errors` wrapper and every `try { } catch (e) { next(e) }` whose only job was plumbing (see [error-handling.md](./error-handling.md)).
- **Brotli request bodies** — `express.json/urlencoded/text/raw` decompress `Content-Encoding: br`.
- `res.send()` accepts `Uint8Array`; `res.sendFile()` accepts an `ETag` option.
- `res.render()` is asynchronous for every view engine.

## Migration order that actually works

1. Bump Node first to a supported LTS (24) — v5 needs `>= 18` but 18/20 are EOL. See [overview.md](./overview.md).
2. Run the codemod recipe; commit that alone so the diff stays reviewable.
3. Grep for the silent-behaviour list: `req.body`, `req.query =`, `req.host`, `res.redirect(`, `hasOwnProperty`, `params[0]`.
4. Boot the app — `PathError` on startup tells you which routes still use v4 syntax.
5. Run integration tests ([testing.md](./testing.md)); route-matching regressions surface as 404s, not exceptions.
6. Only then strip the async wrappers — one behaviour change at a time.

## Not migrating yet?

Express 4 keeps receiving security patches until EOL, but nothing else. If you must stay, at minimum pin `express@^4` (never `*`), run `npm audit` in CI, and track [expressjs.com/blog](https://expressjs.com/en/blog/) for the security releases. Commercial post-EOL support for 4.x exists (HeroDevs), which is a budget decision, not an engineering one.
