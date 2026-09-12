# Cloudflare R2 / Object Storage Setup

Local development may initially use a filesystem adapter.

Production should use S3-compatible object storage such as Cloudflare R2 or AWS S3.

## 1. Create storage abstraction

Conceptual interface:

```text
putObject
getSignedDownloadUrl
deleteObject
headObject
```

Implement:

```text
LocalObjectStorage
R2ObjectStorage
```

## 2. R2 bucket

Create separate buckets or logical prefixes for environments:

```text
buildwise-dev
buildwise-staging
buildwise-production
```

Do not mix production files with development.

## 3. Credentials

Server-only:

```text
R2_ACCOUNT_ID=
R2_ACCESS_KEY_ID=
R2_SECRET_ACCESS_KEY=
R2_BUCKET=
R2_ENDPOINT=
```

Never expose secret keys to the browser.

## 4. Upload pattern

Preferred:

```text
Browser asks API for upload authorization
 ↓
API verifies project permission
 ↓
signed upload / controlled upload
 ↓
object stored
 ↓
API records object metadata
```

Do not trust a client-provided object key to determine tenancy.

## 5. Object key convention

Example:

```text
org/{orgId}/project/{projectId}/documents/{documentId}/{revisionId}/original.pdf
```

Use generated IDs, not raw user filenames as authoritative paths.

## 6. Store metadata in Postgres

- bucket/key
- content type
- size
- hash
- original filename
- uploader
- project
- created_at
- status

## 7. Security

Use private buckets for project data.

Serve through short-lived signed URLs or controlled proxy endpoints.

## 8. Local adapter

Before R2 integration, implement local storage so upload workflows can be tested without cloud dependency.
