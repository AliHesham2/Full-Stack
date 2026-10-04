# Regular Expressions

ReDoS and untrusted patterns → [security.md](./security.md). This file is about the
correctness traps that produce silently wrong results.

## `lastIndex` — the `/g` and `/y` state bug

A regex with `g` or `y` **carries a cursor between calls**. `test()` and `exec()` both
advance it. Verified in Node:

```js
const re = /a/g;
re.test('a');   // true
re.test('a');   // false  ← same input, opposite answer
re.test('a');   // true   ← and it alternates forever
```

The failure is worse across different inputs, because it looks like a data problem:

```js
const HAS_DIGIT = /\d+/g;          // module-level constant — the classic setup
['a1b', 'a1b', 'a1b'].map(s => HAS_DIGIT.test(s));   // [true, false, true]
```

**Why it only sometimes bites**: a regex *literal* evaluates to a fresh object each time
the expression runs, so a literal written inside a function is safe. It is the hoisted
constant, the cached regex, and the one built once with `new RegExp` that break.

```js
const f = () => /x/g;
f() === f();     // false — new object per call
```

**Rules**:
- For a boolean answer, **don't use `g` at all**: `/\d+/.test(s)`.
- If a shared regex must keep `g`, reset before use: `re.lastIndex = 0`.
- Prefer `String.prototype.match` / `matchAll` / `replace`, which manage the cursor
  themselves, over hand-driven `exec` loops.

## `match` with `/g` throws away capture groups

```js
'a1b2'.match(/(\d)/g);                                   // ['1', '2']  — groups lost
[...'a1b2'.matchAll(/(?<d>\d)/g)].map(m => m.groups.d);  // ['1', '2']  — groups kept
```

`match` with `g` returns only full-match strings; without `g` it returns one result
*with* groups. `matchAll` (ES2020) is the only form that gives every match **and** its
groups — and it requires the `g` flag (`TypeError` otherwise).

**Rule:** any time you need groups from multiple matches, use `matchAll`.

## `replaceAll` rejects a non-global regex

```js
'aa'.replaceAll('a', 'b');      // 'bb'  — string pattern, fine
'aa'.replaceAll(/a/g, 'b');     // 'bb'
'aa'.replaceAll(/a/, 'b');      // ✗ TypeError: called with a non-global RegExp
```
Unlike `replace`, which silently replaces only the first match. If you pass a variable
regex to `replaceAll`, assert the flag.

## Named groups and readability

```js
const m = '2026-09-02'.match(/(?<y>\d{4})-(?<m>\d{2})-(?<d>\d{2})/);
const { y, m: month, d } = m.groups;
```
- `groups` is `undefined` when the regex has no named groups — destructure defensively.
- Duplicate names in *different alternatives* are legal since ES2025:
  `/(?<val>\d+)px|(?<val>\d+)em/`.

## Dynamic patterns

```js
new RegExp(`^${RegExp.escape(userValue)}$`);   // ES2025 — escapes every metacharacter
```
Interpolating a raw user value produces either a broken regex or an unintended pattern.
Before `RegExp.escape` was available the manual form was
`s.replace(/[.*+?^${}()|[\]\\]/g, '\\$&')`. Never build a regex from a user-supplied
**pattern** — that is a ReDoS vector → [security.md](./security.md).

## Flags worth knowing

| Flag | Meaning | Note |
|---|---|---|
| `g` | all matches | carries `lastIndex` state — see above |
| `i` | case-insensitive | ASCII-naive without `u`/`v` |
| `m` | `^`/`$` match line boundaries | does **not** make `.` match newlines |
| `s` | `.` matches newline | this is the one people want when reaching for `m` |
| `u` | Unicode mode | required for `\p{...}` |
| `v` | Unicode sets (ES2024) | supersedes `u`; adds set subtraction `--` and intersection `&&` |
| `y` | sticky | matches only at `lastIndex`; same state caveat |
| `d` | indices | populates `match.indices` with start/end offsets |

```js
/./s.test('\n');            // true
/\p{Script=Arabic}/u.test('ب');   // true — needs u or v
```

## Anti-patterns

1. **Parsing HTML/JSON/URLs with regex** — use `DOMParser`, `JSON.parse`, `new URL()`.
2. **`.` expecting it to match newlines** — add `s`, not `m`.
3. **Validating emails with a regex** — accept anything with an `@`, then verify by
   sending mail. Every "perfect" email regex is wrong and several are ReDoS-prone.
4. **A shared `/g` regex in a hot path** — see `lastIndex` above.
5. **`new RegExp` inside a loop** — compile once outside (but then mind `lastIndex`).
