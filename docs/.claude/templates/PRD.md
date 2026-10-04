# PRD — <fill: feature name>

| | |
|---|---|
| Feature folder | `plans/<fill: feature-folder>/` |
| Version | 1 |
| Status | draft |
| Written by | <fill: worker> |
| Approved by the user on | — |

`Status` is `draft` until the USER approves it; then `approved`, with the date above.

## Change Log

| Version | Date | What changed | IDs affected |
|---|---|---|---|
| 1 | <fill: YYYY-MM-DD> | first draft | all |

## How to Write This PRD

1. **Every item has an ID:** `US-n` story · `OQ-n` open question · `API-n` backend endpoint this feature calls ·
   `DS-n` design-system addition · `UI-n` screen · `FL-n` flow · `BR-n` rule · `MOD-n` code unit ·
   `PKG-n` new package · `DEC-n` decision · `AC-n` acceptance check. Each ID is defined exactly once and never reused. A removed item keeps
   its heading or row, marked `(REMOVED in vN)`. Every ID that an AC covers, a MOD depends on or
   implements or a flow shows is defined in this PRD and not REMOVED (only the
   Change Log and the removed item itself may name a REMOVED ID). A REMOVED item needs no AC, no
   MOD and no roadmap step, and a REMOVED AC or MOD never counts as covering anything.
2. **Tables, not prose**, for specifications in sections 1–9. Section 10 is the required
   final explanation for the user: simple Arabic prose and concrete request/response examples
   are allowed there. It explains the specification above; it adds no requirements, fields or
   endpoints.
3. **Never guess.** Anything unknown goes into Open Questions. A PRD with an unanswered question
   cannot be approved.
4. **Never delete a section.** A section that does not apply says `None — <why>`.
5. **Follow the project's guides.** Where a guide decides something, cite it
   (e.g. "errors per frontend-guide/03-data/backend-integration.md § Mutations and Error States").
6. **Replace every `<fill…>` marker.**
7. **There is at least one user story and one AC (not REMOVED), and every API, DS, UI, FL
   and BR item is covered by at least one AC.**
8. **Every API, DS, UI and BR item is implemented by at least one MOD** (an API-n: by the
   data-layer MOD that calls it; a BR-n: by the frontend MOD that checks it or shows its error). Each MOD row has a path under `frontend/` (no path twice among the rows that are not REMOVED: one
   file, one MOD), a kind, one job, and at least one PRD ID it implements; Depends on names only
   MOD IDs; Exports and Depends on say `—` when there are none. The one exception to "one file,
   one MOD": a generator command a guide gives (e.g. `create-next-app`) is ONE MOD of kind
   `scaffold`, whose Path is the app folder and whose job names the exact command; the files it
   generates and the packages it installs need no rows of their own, but a generated file that a
   step changes (the same step included) gets its own MOD. The
   Code Structure is the
   complete list of files the build creates or changes, every package it adds is a PKG-n, and
   every choice the guides leave open that is expensive to reverse is a DEC-n that comes from an
   answered Open Question: the implementer invents none of them.
9. **The final user explanation is mandatory.** Cover every active API-n by reference,
   without defining their IDs again. Each API has its purpose, caller/auth needs, params/query/
   headers/body, a success status/body and examples for every declared failure status/code.
   Show no-body cases explicitly (e.g. 204); do not fabricate JSON or errors for them. Use
   fictional data, never credentials or real personal data. There is no database
   part: the tables belong to the backend project. If no APIs apply (e.g. setup), say None and
   explain why in Arabic.
10. **Before you report DONE, check every rule above against the WHOLE document, one by one**, and
   put each rule's result in your report, with the coverage list: every API, DS, UI, FL and
   BR ID → its AC(s) → the MOD(s) that implement it (an FL: its AC(s) only). Claude checks them
   again by hand.

---

## 1. For the User

### Goal

<fill: one or two sentences — the problem this solves, and for whom>

### User Stories

| ID | As a … | I want … | So that … |
|---|---|---|---|
| US-1 | <fill> | <fill> | <fill> |

### In Scope

- <fill>

### Out of Scope

- <fill>

### Open Questions

| ID | Question | Why it matters | Answer (the user's; a technical choice: Claude's) |
|---|---|---|---|
| OQ-1 | <fill> | <fill> | |

An unanswered question blocks approval. If there are none, replace the table with `None`.

---

## 2. Data

`None — the database belongs to the backend project. This project designs no table.`

---

## 3. API

Every endpoint here already EXISTS in the backend project. This section copies the contract of
the ones this feature calls; it never designs a new one. Errors use the envelope
`{ error: { code, message, details? } }` (`details` only for a 400 validation error). An endpoint
or a field the backend's contract does not have is an Open Question.

### API-1 — <fill: METHOD /path>

| | |
|---|---|
| Purpose | <fill> |
| Who may call | <fill: anyone / a logged-in user / a role> |
| Request | <fill: params, query and body — every field with its type; validation by BR-n> |
| Success | <fill: status + response body> |
| Idempotent · rate limit | <fill> |
| Source | <fill: where this contract comes from: the backend's PRD, its OpenAPI file, or the user's material in plans/<feature>/> |

| Error status | Code | When | Message |
|---|---|---|---|
| <fill> | <fill> | <fill> | <fill> |

---

## 4. Design System

New tokens or `components/ui` pieces this feature needs. Usually `None — uses the existing kit`.

| ID | Kind | Name | Why the existing kit is not enough |
|---|---|---|---|
| DS-1 | <fill: token / component> | <fill> | <fill> |

---

## 5. Screens

### UI-1 — <fill: screen name>

| | |
|---|---|
| Route | <fill: e.g. /login> |
| Who can see it | <fill> |
| Components | <fill: kit pieces from components/ui + feature components> |
| Data | <fill: server query / GET Route Handler for reads, Server Action for mutations → which Express API-n> |
| Loading | <fill> |
| Empty | <fill> |
| Error | <fill> |
| Success | <fill> |
| No permission | <fill> |
| Phone · tablet · desktop | <fill: what differs> |
| Text | <fill: every label, button and message the user sees> |

---

## 6. Flows

One diagram per user journey, from the screen down to the backend's API, with its failure paths.

### FL-1 — <fill: flow name>

```
UI-1 <screen> → Next.js query / GET relay (read) or Server Action (mutation) → API-1 <METHOD /path> (the backend)
  └─ fails with <status / code> → UI-1 shows "<message>" (BR-n)
```

---

## 7. Business Rules and Validation

The backend always enforces a rule (it is not this project's code). The frontend checks it only
for quicker feedback. (In the feature
`setup`, a rule that a check enforces, e.g. an ESLint rule: Checked in = that check; Error code =
`—`; Message = the check's own message. A runtime rule, such as an error shape, a body limit or
a timeout, gets its real status and code like any other rule.)

| ID | Rule | Checked in | Error code | Message |
|---|---|---|---|---|
| BR-1 | <fill> | <fill: the backend · the frontend MOD that also checks it> | <fill> | <fill> |

---

## 8. Code Structure

Every file the build creates or changes, decided here, before any code exists. Paths follow the
guide's project structure (frontend-guide/01-foundation/project-structure.md). One row per file; each file has ONE job.

| ID | Path | Kind | Its one job | Exports (with types) | Depends on | Implements |
|---|---|---|---|---|---|---|
| MOD-1 | <fill: frontend/src/…> | <fill: action · query · component · page · …> | <fill: one sentence> | <fill: e.g. `login(input: LoginInput): Promise<SessionId>`> | <fill: MOD-n, or —> | <fill: PRD IDs> |

A change to a file's job or exports later is a DESIGN change: it comes back here first, never
straight into the code.

### New Packages

Every package the build adds (frontend-guide/04-quality/dependencies.md). If there are none, `None`.

| ID | Package | App | Why it is needed | Source (the guide that allows it, or the OQ-n that decided it) |
|---|---|---|---|---|
| PKG-1 | <fill> | frontend | <fill> | <fill> |

### Decisions

Every choice the guides leave open that is expensive to reverse: architecture shape, the rendering
strategy, the state-management approach, buy vs build, a deviation from a guide default. The PRD writer never decides one
alone: it is first an Open Question with its options; once answered, it becomes a row here whose
source is that OQ. The OQ keeps only the short answer; this row is the decision's full record.
If there are none, `None`.

| ID | Decision | Context | Alternatives, and why each lost | Consequences | Assumptions, what would re-open it, and the direction then | Source |
|---|---|---|---|---|---|---|
| DEC-1 | <fill> | <fill> | <fill> | <fill> | <fill> | <fill: OQ-n> |

---

## 9. Acceptance Criteria

The testers turn every row into a test, and report results by these IDs.

| ID | Covers | Given | When | Then |
|---|---|---|---|---|
| AC-1 | <fill: IDs, e.g. UI-1, BR-1> | <fill> | <fill> | <fill> |

---

## 10. شرح مبسط ليك — الـAPIs اللي بنستخدمها

هذا آخر قسم في الـPRD، مخصص لصاحب المشروع. اكتب بالعربية البسيطة، وعرّف أي مصطلح
تقني عند أول استخدام. المعلومات والأمثلة تشرح الأقسام السابقة ولا تضيف قرارات جديدة.
حدّث هذا القسم مع كل تغيير في عقود الـAPI.

### الفكرة في خطوات بسيطة

<fill: ماذا سيحدث من ضغط المستخدم على الزر إلى ظهور النتيجة، وكيف يمر الطلب من
المتصفح إلى Next.js ثم إلى الـbackend (مشروع تاني)>

### الـAPIs التي سنستخدمها (موجودة في مشروع الـbackend)

كرّر البلوك التالي لكل endpoint فعّال في قسم 3، واستخدم الـAPI ID كمرجع فقط.
إذا لم توجد APIs فعّالة، اكتب «مفيش APIs للميزة دي» مع السبب، ولا تنشئ أمثلة وهمية.

#### شرح endpoint: <fill: API ID من قسم 3>

| السؤال | الإجابة المبسطة |
|---|---|
| بتعمل إيه؟ | <fill: الغرض ومتى يستخدمها المستخدم> |
| العنوان والطريقة | <fill: METHOD /path في Express؛ اشرح GET/POST/PATCH/DELETE المستخدمة> |
| مين يقدر يستخدمها؟ | <fill: تسجيل الدخول والصلاحية المطلوبة؛ Next.js يمرّر هوية المستخدم إلى Express> |
| هنبعت إيه؟ | <fill: path params، query، headers، body؛ معنى كل حقل مطلوب/اختياري> |
| هيرجع إيه لو نجحت؟ | <fill: status ومعنى الحقول؛ اذكر عدم وجود body إذا كان العقد بلا body> |
| ممكن تفشل ليه؟ | <fill: كل failure status/code المعلن في قسم 3، ومعناه وما يمكن للمستخدم فعله> |

**مثال request**

اعرض العنوان والـheaders في بلوك `http`، والـbody إن وُجد في بلوك `json` صالح.

<fill: METHOD والعنوان مع قيم params/query خيالية، والـheaders المطلوبة فقط.
إذا كان الطلب يحتوي على body، أضف مثال JSON صالحًا مطابقًا للعقد.
إذا لم يكن له body، اكتب «الطلب ده مفيهوش body».
أي session/service credential يظهر كـREDACTED فقط؛ لا تعرض أي مفتاح حقيقي.>

**مثال response عند النجاح**

اعرض HTTP status بوضوح، والـJSON الفعلي في بلوك `json` عند وجوده.

<fill: HTTP status ثم body صالح مطابق لقسم 3، واشرح معناه بجملة بسيطة.
في 204 أو download/stream، وضّح شكل النتيجة الحقيقي بدل اختراع JSON.>

**أمثلة response عند الفشل**

لكل حالة فشل معلنة، اعرض HTTP status والـerror JSON في بلوك منفصل، ثم شرح السبب.

<fill: مثال لكل failure status/code معلن في قسم 3: الحالة، body مطابق للعقد،
السبب بكلام بسيط، والتصرف المناسب. لا تضف 401/403/400 أو غيرها ما لم يعلنها العقد.>

