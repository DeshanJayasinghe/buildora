# Project Documents, Search and RAG Setup

Do this after document upload/storage and the AI Gateway are stable.

## 1. Document ingestion stages

```text
Upload
 ↓
Store original
 ↓
Virus/type/size validation
 ↓
Extract text/metadata
 ↓
Chunk
 ↓
Embed
 ↓
Store chunks + provenance
```

## 2. Database

Enable pgvector through a migration:

```sql
CREATE EXTENSION IF NOT EXISTS vector;
```

## 3. Tables

Conceptual:

```text
documents
document_revisions
document_pages
document_chunks
document_embeddings
```

Every chunk must preserve:

- document ID
- revision ID
- page/section
- source offsets when possible
- extracted text
- embedding model/version

## 4. Retrieval

Use hybrid retrieval later:

- PostgreSQL full text
- vector similarity
- metadata filters

Always filter by:

- tenant/organization
- project
- permissions

Never retrieve another tenant's chunks.

## 5. AI response provenance

The AI answer should reference:

- document title
- revision
- page/section

Do not allow unsupported project-document claims.

## 6. Prompt injection

Treat document text as untrusted data.

Documents cannot override system/tool policy.

## 7. Embedding model changes

Do not silently mix incompatible embedding dimensions/models in one index.

Store embedding model/version and plan re-embedding migrations.
