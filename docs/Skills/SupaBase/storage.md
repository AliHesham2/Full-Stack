# Storage

> Source review: 2026-09-27. This is a template reference, not a running project.
> Historical execution/measurement statements below are author reports without a supplied
> reproduction harness; they are not acceptance evidence. This review ran no Supabase
> CLI, SQL, deployed function, or real provider integration.

> Project boundary: these are provider capabilities, not frontend permissions. All provider
> SDK calls, DB/Auth/Storage/Realtime integrations run in Express on Node.js for this project.
> Browser → Next.js → Express → Supabase. Provider-specific URLs/keys/tokens stay behind Express.
> The frontend uses same-origin relays; generic direct-client examples are not this project's flow.


S3-compatible file storage with CDN, image transformations, and access control via Postgres RLS.

Source: https://supabase.com/storage

## Bucket Design

Buckets are containers for files. Two types:

| Type | Public access | RLS | Use case |
|------|--------------|-----|----------|
| Public | Yes (anyone with URL can read) | Optional on read, required on write | Avatars, public assets |
| Private | No (signed URL or auth required) | Required for all operations | User uploads, documents |

### Common pattern

```
avatars/     (public bucket)  — profile images
documents/  (private bucket) — user-uploaded files
```

## Storage RLS

Storage is just a Postgres table (`storage.objects`) with RLS, so the same policy rules
apply — including everything in [rls-patterns.md](./rls-patterns.md). The only addition is
`storage.foldername(name)`, which splits the object path into an array so you can gate on
folders. You still use `auth.uid()` alongside it:

```sql
-- Users can only upload to their own folder
CREATE POLICY "users_own_uploads"
  ON storage.objects FOR INSERT
  TO authenticated
  WITH CHECK (
    bucket_id = 'documents'
    AND (storage.foldername(name))[1] = auth.uid()::text
  );

-- Users can read own files
CREATE POLICY "users_read_own"
  ON storage.objects FOR SELECT
  TO authenticated
  USING (
    bucket_id = 'documents'
    AND (storage.foldername(name))[1] = auth.uid()::text
  );
```

**Path convention**: Store files at `{user_id}/{filename}` so folder-level policies work.

## Image Transformations

On-the-fly image resizing and optimization. Available on Pro plan and above.

```typescript
// No transformation — original file
const { data } = supabase.storage
  .from('avatars')
  .getPublicUrl('user-123/photo.jpg')

// With transformation
const { data } = supabase.storage
  .from('avatars')
  .getPublicUrl('user-123/photo.jpg', {
    transform: { width: 200, height: 200, resize: 'cover' }
  })
```

Transformed images are **cached**. First request generates the variant; subsequent requests serve from CDN cache.

**Cost optimization**: Transforming on every request can increase bandwidth. Use specific sizes rather than arbitrary dimensions, and cache aggressively.

Source: https://supabase.com/docs/guides/storage/serving/image-transformations

## Signed URLs

For private files, generate a time-limited URL:

```typescript
const { data, error } = await supabase.storage
  .from('documents')
  .createSignedUrl('user-123/contract.pdf', 3600) // 1 hour expiry
```

## File Size Limits

| Plan | Max file upload size | CDN |
|------|---------------------|-----|
| Free | 50 MB | Basic CDN |
| Pro | 500 GB | Smart CDN |
| Team | 500 GB | Smart CDN |

Source: https://supabase.com/pricing (Storage section)
## Template transport and size limits

Provider public/signed URLs are internal to Express. Stream uploads/downloads through
Next.js and Express; do not redirect browsers to Supabase URLs. Enforce ownership,
bounded streaming, file type and byte limits.

Free 50 MB / paid 500 GB are global file-size configuration ceilings, not promises that
standard uploads or Next/Express hosting support that size. Buckets can set lower limits.
Large files need an appropriate resumable/S3 strategy behind the relay, respecting
deployment limits.

Source: https://supabase.com/docs/guides/storage/uploads/file-limits
