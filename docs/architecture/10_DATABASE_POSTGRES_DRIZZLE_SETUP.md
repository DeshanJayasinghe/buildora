# PostgreSQL + Drizzle Setup

BuildWise uses PostgreSQL as the main transactional database.

For the TypeScript data layer, this setup pack recommends **Drizzle ORM + node-postgres** because it remains close to SQL and has useful PostgreSQL extension support.

## 1. Run local PostgreSQL

Use Docker Compose from the local infrastructure guide.

Initial database:

```text
host: localhost
port: 5432
database: buildwise
user: buildwise
password: local-development-only
```

Never use the local password in production.

## 2. Add Drizzle dependencies to API

```bash
pnpm --filter api add drizzle-orm pg
pnpm --filter api add -D drizzle-kit @types/pg
```

## 3. Create DB folder

```text
apps/api/src/db/
├── client.ts
├── schema/
└── migrations/
```

A better long-term option is a dedicated internal data-access package if several deployables require database access. Do not create it until needed.

## 4. Connection

Use one PostgreSQL Pool per API process.

Conceptual:

```ts
import { Pool } from "pg";
import { drizzle } from "drizzle-orm/node-postgres";

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
});

export const db = drizzle({ client: pool });
```

## 5. Drizzle config

Create `apps/api/drizzle.config.ts` with schema and migration paths appropriate to the app.

Example:

```ts
import "dotenv/config";
import { defineConfig } from "drizzle-kit";

export default defineConfig({
  dialect: "postgresql",
  schema: "./src/db/schema/**/*.ts",
  out: "./drizzle",
  dbCredentials: {
    url: process.env.DATABASE_URL!,
  },
});
```

## 6. First schema

Start with only:

- users mirror
- organizations
- organization_members
- projects
- audit_events

Do not create all future BIM/QS tables on day one.

## 7. Migration scripts

Add package scripts:

```json
{
  "db:generate": "drizzle-kit generate",
  "db:migrate": "drizzle-kit migrate",
  "db:studio": "drizzle-kit studio"
}
```

## 8. Migration rule

Never use schema push as the normal production deployment mechanism.

Production schema changes go through committed migrations.

## 9. Financial data

Use PostgreSQL `numeric` for money/rates where exact decimal semantics matter.

## 10. Verify DB

From container:

```bash
docker compose exec postgres psql -U buildwise -d buildwise -c "SELECT 1;"
```

Then run a simple API integration test that queries the DB.

## 11. Production option

For a solo developer, Neon is a practical first managed PostgreSQL target.

Keep the API on ordinary PostgreSQL semantics so migration to RDS/Aurora remains feasible.
