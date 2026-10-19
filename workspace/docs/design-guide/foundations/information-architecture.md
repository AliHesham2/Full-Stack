# Information Architecture

IA is the decision about **what content exists and how it is grouped, labeled, and related**.
It precedes navigation: [../components/navigation.md](../components/navigation.md) covers the
*patterns* that expose a structure (top bar, sidebar, breadcrumbs); this file covers the
structure itself. Choosing a sidebar cannot fix a structure users cannot reason about.

**The failure this file prevents:** interfaces that look clean and organized while being
unnavigable, because the grouping reflects how the *system* is built rather than how the user
thinks. This is a signature failure of generated designs — plausible-looking sections with no
underlying model.

## Rule 1: Structure Mirrors the User's Mental Model, Not the Org Chart or the Database

The most common IA mistake is exposing internal structure as navigation:

| Structure taken from | Symptom | What users actually think |
|---|---|---|
| The org chart | Sections named after departments ("Operations", "Compliance") | "I want to report a problem" |
| The database schema | Nav items named after tables ("Entities", "Records") | "My orders", "My team" |
| The API surface | One nav item per endpoint group | Tasks, not resources |
| Feature launch order | Newest features bolted onto the end | Related things sit together |

**Test:** write down the top-level labels and ask whether a new user could predict what is
behind each one *before clicking*. If two labels could plausibly contain the same item, the
structure is ambiguous — not the wording.

→ Mental models are covered as a cognitive constraint in
[cognitive-principles.md](cognitive-principles.md); this is the structural consequence.

## Rule 2: Pick an Organizing Scheme Deliberately

There are only two families, and mixing them without noticing is what produces incoherent IA.

**Exact schemes** — objectively correct, no interpretation:

- Alphabetical (long lists of known-item lookups: country pickers, glossaries)
- Chronological (activity, history, logs, changelogs)
- Geographic (locations, regions)

Use when the user **knows what they are looking for** and only needs to find it.

**Ambiguous schemes** — require judgment, and are where all the value and all the risk live:

- **By topic** — classic content sites. Fails when items belong to several topics.
- **By task** — "Send money", "Track a package". Strong for transactional products.
- **By audience** — "For developers" / "For teams". Only works if users self-identify
  reliably and belong to exactly one group. Fails badly when they don't.
- **By lifecycle/stage** — onboarding → active → archive. Strong when the product has real
  phases.

Use when the user is **exploring** rather than retrieving.

**Rule:** one primary scheme per level. A top level that mixes task labels and audience labels
("Send money", "Track a package", "For businesses") forces the user to hold two models at once.
Secondary schemes belong at deeper levels or in filters.

## Rule 3: Prefer Breadth Over Depth

**Do not apply Miller's 7±2 to navigation.** Miller's limit describes short-term memory for
*recalled* items; navigation items are *visible*, so they are recognition, not recall — the
limit does not apply. Users are not counting your nav items.

What the evidence does support: **deep hierarchies cost more than wide ones**. Each level is a
new decision, a new chance to guess wrong, and a new page load. A flat structure with 12
scannable items beats 4 items hiding 3 sublevels each.

Practical bounds:

- **Depth:** aim for ≤3 levels from home to any content. At 4+, add search and breadcrumbs and
  treat the depth as a problem to solve, not a given.
- **Breadth:** wide is fine *if it is scannable* — grouped, with headings, in a predictable
  order. 20 ungrouped links is not breadth, it is a dump.
- The real constraint is **scannability**, not count → [visual-hierarchy.md](visual-hierarchy.md).

## Rule 4: Labels Are Part of the Structure

- **Use the user's vocabulary, not the product's.** If users say "invoices" and the schema says
  "billing_documents", the label is "Invoices".
- **Be specific over clever.** "Resources", "Solutions", "More", "Tools" carry no information.
  A label that could hold anything tells the user nothing.
- **Never create a "Misc" / "Other" bucket.** It is an admission that the scheme failed, and it
  grows without limit. Items in it are effectively unfindable.
- **Keep labels parallel in form.** All nouns or all verb phrases within one level — mixing
  ("Dashboard", "Create report", "Settings") reads as three unrelated systems.
- **One concept, one label, everywhere.** The nav item, the page title, and the breadcrumb must
  use the same word. Synonyms across surfaces break the sense of location.

→ Wording rules for labels and all other UI text:
[content-and-microcopy.md](content-and-microcopy.md).

## Rule 5: Browse and Search Are Both Required Above a Threshold

- **Small, stable content (< ~50 items):** browse only. Search adds a control nobody needs.
- **Large or user-generated content:** search is the primary path and must be prominent.
  Browsing becomes the fallback for users who don't know the term.
- Search that returns nothing must return **something useful** — spelling correction, related
  results, or an explicit path back to browsing. An empty search result is an empty state and
  obeys the empty-state rules → [../interaction-design/state-patterns.md](../interaction-design/state-patterns.md).
- **Filters are IA too.** Facets expose the structure sideways; their categories should come
  from the same scheme, not be invented separately.

## Rule 6: URLs Are the Most Durable Expression of the IA

For anything on the web, the URL is structure the user can see, edit, share, and bookmark.

- Readable and predictable: `/projects/atlas/settings`, not `/p?id=8321&v=2`.
- Hierarchy in the path should match hierarchy in the breadcrumbs. If they disagree, one of
  them is lying about the structure.
- **Truncating the URL should work.** If `/projects/atlas/settings` is valid, `/projects/atlas`
  and `/projects` should be too. Users do this.
- Changing IA breaks links — plan redirects. IA changes are migrations, not restyles.

## Validating an IA Without Users

Full card sorting and tree testing need participants. When designing without them, these
substitutes catch most structural errors:

1. **The prediction test.** For each top-level label, write the three things you expect to find
   behind it. Then list the actual contents. Divergence = ambiguous label or wrong grouping.
2. **The item-placement test.** Take 15 real content items and place each in exactly one
   category. Any item that fits two categories equally well marks an overlap; any item with no
   home marks a missing category — *not* a reason to create "Other".
3. **The stranger test.** Show only the nav labels, no content. Can someone state what the
   product does? If not, the structure is describing the system, not the offering.
4. **The one-item-per-section check.** A category with a single item is not a category. Merge it.
5. **The growth check.** Where does next quarter's feature go? A structure with no obvious slot
   for the next thing will accumulate a "More" menu within a year.

When real users are available, **tree testing** (navigate a text-only hierarchy to find a given
item) is the highest-value method: it isolates structure from visual design, which is exactly
the thing this file is about.

## IA Anti-Patterns

| Anti-pattern | Why it fails |
|---|---|
| Org-chart or schema navigation | Users don't know or care about your internals |
| "Misc" / "Other" / "More" as a real category | Unbounded, unfindable, hides the scheme's failure |
| Mixing organizing schemes at one level | Forces two mental models at once |
| Deep nesting to keep menus "clean" | Each level is a guess; clean and unusable |
| Vague top-level labels ("Solutions", "Resources") | Zero information scent |
| Same content under multiple labels | Users can't tell if they're in the right place |
| Nav that changes between sections | Destroys the sense of a stable structure |
| Breadcrumbs that don't match the URL | Two contradictory claims about where the user is |

## Cross-References

- Navigation patterns that expose this structure → [../components/navigation.md](../components/navigation.md)
- Why depth costs the user → [cognitive-principles.md](cognitive-principles.md)
- Making a wide structure scannable → [visual-hierarchy.md](visual-hierarchy.md)
- Hiding complexity without hiding it from findability → [../interaction-design/progressive-disclosure.md](../interaction-design/progressive-disclosure.md)
- Moving *through* the structure → [../interaction-design/user-flows.md](../interaction-design/user-flows.md)
- Label wording → [content-and-microcopy.md](content-and-microcopy.md)

## References

- Rosenfeld, Morville & Arango, *Information Architecture* (4th ed.) — the organizing-scheme
  taxonomy used above
- NN/g on tree testing and IA: https://www.nngroup.com/topic/information-architecture/
- NN/g, "Flat vs. Deep Website Hierarchies" — the breadth-over-depth evidence
