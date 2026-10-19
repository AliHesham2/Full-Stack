# File & Object Storage — Uploads, Downloads, and Boundaries

> **Part of:** Async & Scale — Backend Project Guide
> **Read when:** Accepting user files, storing generated artifacts, serving media, or designing signed access.
> **Related:** `../04-security/security-foundations.md`, `../02-domain-and-api/validation.md`, `../06-async-and-scale/background-jobs.md` (scanning/processing)
> **Status:** Stable principles · Storage service mechanics in the storage/platform guide

---

## 1. Rule Zero

**Object storage (S3-class / platform equivalent) is the system of record for files; the database stores references and metadata.** Storing blobs in the DB (bytea columns) multiplies DB size, breaks backups, poisons caches/buffers, and makes the DB the bottleneck for media traffic. Exceptions are tiny, rare, and deliberate (e.g., 2KB avatars — and even then, usually not).

## 2. Upload Flows

**When file APIs are actually in scope, the current boundary is proxied access:** external authorized client → Express → approved storage. Clients receive provider-independent file IDs and authenticated API routes; only backend owns bucket/key mapping and credentials. Bounded streaming/chunks, cancellation and actual ingress/time limits must be specified. Direct grants below remain alternatives requiring an explicit approved contract/architecture change; no client app or provider setup is created by this guide.

| Flow | Shape | Use when |
|---|---|---|
| **Proxied upload** | client → your API → storage | Small files, validation must precede storage, no direct-grant capability |
| **Direct upload (presigned URL)** | your API issues scoped signed URL → client uploads straight to storage | Large files, high volume, mobile/flaky networks — a general option only when the project explicitly permits direct storage access |

Direct-upload sequence: (1) client requests upload grant with file intent (name, size, type) → (2) API validates policy (is this type/size allowed for this user/quota?) → (3) issues short-TTL, single-purpose signed URL with constrained content-type/key prefix → (4) client uploads → (5) client notifies completion → (6) backend verifies existence/metadata and records it. Step 6 matters: never trust the client's "I uploaded it" as validation.

## 3. Upload Validation (Defense in Depth)

- **Size:** enforced at THREE layers — API body limit, signed-URL policy (content-length-range), storage-side caps. One alone always leaks.
- **Type: verify content, not claims.** Client MIME and file extension are user input. Validate **magic bytes** / sniff actual content (or run a real type-detection pass), restrict to an explicit allowlist, and reject mismatches between claimed and detected.
- **Filename: never trusted, never used as storage path.** Generate storage keys server-side (UUID-based, prefix-structured: `tenants/{tid}/uploads/{date}/{uuid}`); store the original name as display metadata only (and sanitize it for any later rendering — `../04-security/security-foundations.md` §3).
- **Path traversal:** storage keys are server-generated (above), so client-supplied path segments never reach path concatenation (`../04-security/security-foundations.md` §3).
- **Malware scanning** for user-facing platforms: async scan-on-upload (job or storage-event-triggered scanner — `background-jobs.md`), file quarantined (unscanned = unlisted/unservable) until clean. Not every product needs it; marketplaces and shared-content products do.
- **Re-processing:** images/videos get normalized on ingest (strip EXIF for privacy-sensitive products — GPS coordinates in photos are a real PII leak —, re-encode to safe formats, resize variants via jobs).

## 4. Access Control

- **Private by default.** Public buckets are the exception, chosen per artifact class (truly public assets: marketing media), never the default posture.
- **Access = authorization check + controlled delivery** (`../03-auth/authorization.md`): the issuing endpoint runs the object-level authorization decision; the signed URL is short-TTL (minutes) and scoped to GET that key. Authorization is evaluated at issuance time — if permissions change mid-TTL, the window is bounded by TTL; for sensitive content, keep TTLs small.
- Under the current proxied contract Express consumes provider grants internally and relays permitted bytes to the authorized caller. Return provider-independent API identifiers/locations, not secret provider credentials or grants. Client-origin URLs and client proxy implementation belong to another project.
- Signed URLs are **bearer capabilities**: don't log them, don't make them long-lived, don't reuse one URL as a stable public identifier.
- Public objects (when genuinely public) still go through an allowlist path — a "private bucket with everything presigned forever" is a public bucket with worse observability.

## 5. Serving & Downloads

- Serve via CDN in front of storage for hot media; `Cache-Control` per artifact class (immutable content-addressed objects can cache forever; permissioned short-TTL URLs cache per their TTL).
- Downloads of user-supplied files: `Content-Disposition: attachment` + correct `Content-Type` + `X-Content-Type-Options: nosniff` — prevents stored-XSS via "HTML" uploads rendered inline (`../04-security/security-foundations.md` §4).
- Range requests/streaming for large media — the storage/CDN layer provides; your API shouldn't proxy bytes it doesn't have to (every proxied MB is your CPU and latency budget).

## 6. Lifecycle & Cost

- Define lifecycle per artifact class at design time: retention (user deletes account ⇒ their files go within N days — `../11-specialized-concerns/privacy-compliance-audit.md`), orphan cleanup (uploaded-but-never-recorded objects — the abandoned-upload sweeper job), archival tiers, expiration for transient artifacts (exports, caches).
- Orphans are guaranteed: crashed flows, abandoned uploads. A scheduled reconciliation job (storage list vs DB references) is part of the design, not an afterthought.
- Quotas per identity/tenant — enforcement at issuance (§2 step 2) plus storage-side caps.
- Metadata in DB: key, size, detected type, owner, tenant, hash (dedup + integrity), scan status, lifecycle state.

## 7. Failure Modes

- **Trust-the-client metadata:** size/type/name from the request body accepted as truth; "avatar.jpg" is a 40MB HTML polyglot. Verify content (§3).
- **The eternal presigned URL:** TTL of days "so links don't break" — leaked URLs = permanent unauthorized access. Short TTL + a fresh-URL endpoint instead.
- **Upload-then-trust:** recording the file as available because the client said so, without existence/metadata verification (§2 step 6).
- **Unbounded uploads:** no quota, no size cap at every layer — a weekend script fills the storage bill.
- **The orphan field:** deletions remove DB rows but not objects (or vice versa); storage grows monotonically, costs creep, and "deleted" PII survives in objects — reconciliation job per §6.
- **Inline user content:** serving user uploads inline as `text/html` (or sniffed types) — stored XSS with a file-manager UI. nosniff + attachment + re-encoding.
