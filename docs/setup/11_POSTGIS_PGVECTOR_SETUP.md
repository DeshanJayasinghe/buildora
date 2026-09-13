# PostGIS and pgvector Setup

Do not enable these until a feature needs them.

- PostGIS → site/GIS/spatial functionality
- pgvector → RAG/semantic retrieval

## 1. PostGIS

Enable on a database where the extension is installed:

```sql
CREATE EXTENSION IF NOT EXISTS postgis;
```

Verify:

```sql
SELECT PostGIS_Version();
```

With Drizzle, spatial columns can be represented using its PostgreSQL geometry types where suitable, and custom SQL can be used for more advanced geometry functions.

Use SRID:

```text
4326
```

for ordinary longitude/latitude storage unless a different projected coordinate system is deliberately selected.

## 2. pgvector

Enable:

```sql
CREATE EXTENSION IF NOT EXISTS vector;
```

Verify:

```sql
SELECT extversion FROM pg_extension WHERE extname = 'vector';
```

## 3. Drizzle extension migrations

Extensions should be enabled through explicit migration SQL.

Example:

```bash
pnpm --filter api exec drizzle-kit generate --custom
```

Edit the generated migration to include:

```sql
CREATE EXTENSION IF NOT EXISTS vector;
CREATE EXTENSION IF NOT EXISTS postgis;
```

## 4. Do not confuse spatial geometry

The Buildora AI Building Model's internal construction geometry is not the same as PostGIS site geography.

Use:

- domain geometry → building model / geometry engine
- PostGIS → real-world site/location data

Do not store every wall as a PostGIS object merely because PostGIS exists.

## 5. Vector dimensions

Do not hardcode an embedding vector dimension across the domain layer.

The selected embedding provider/model controls dimensions.

Keep it in infrastructure/configuration and migrations.

## 6. Managed DB

Before enabling an extension on a managed provider, verify that the chosen PostgreSQL version/provider supports it.
