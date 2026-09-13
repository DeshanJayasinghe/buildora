# NestJS API Setup

Target:

```text
apps/api
```

Buildora AI uses NestJS as the main API/application backend.

## 1. Scaffold with the CLI

Install or run Nest CLI.

One-time global option:

```bash
npm i -g @nestjs/cli
```

Then:

```bash
nest new apps/api --strict
```

Choose:

- TypeScript
- ESM if the current CLI asks and the workspace/tooling is compatible
- pnpm

If the generator creates a nested `.git`, remove only that nested `.git`.

## 2. Verify

```bash
pnpm --filter api start:dev
```

Default Nest port is commonly:

```text
http://localhost:3000
```

but the web app also uses 3000.

Set API port to:

```text
4000
```

through configuration.

## 3. Add configuration

```bash
pnpm --filter api add @nestjs/config
```

Create configuration validation rather than reading arbitrary environment variables throughout modules.

## 4. Add health endpoint

Add:

```text
GET /health
```

Response:

```json
{
  "status": "ok"
}
```

Later include:

- DB
- Redis
- Temporal connectivity

but keep liveness simple.

## 5. API prefix

Use:

```text
/api/v1
```

or configure versioning through Nest.

## 6. OpenAPI

When endpoint development starts:

```bash
pnpm --filter api add @nestjs/swagger
```

Expose Swagger only in development/staging or protect it appropriately in production.

## 7. Domain boundaries

Nest modules coordinate use cases. They do not own core mathematical/domain truth.

Examples:

```text
ProjectsModule
BuildingModelModule
QuantitiesModule
BoqModule
CostModule
AiModule
DocumentsModule
BillingModule
```

Core deterministic logic belongs in packages such as:

```text
@buildora/building-model
@buildora/qs-engine
@buildora/cost-engine
```

## 8. Verify

```bash
pnpm --filter api lint
pnpm --filter api test
pnpm --filter api build
```
