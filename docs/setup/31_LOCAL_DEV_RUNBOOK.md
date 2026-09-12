# Local Development Runbook

This is the daily start/stop guide.

## Start

### Terminal 1 — infrastructure

```bash
docker compose up -d
docker compose ps
```

### Terminal 2 — Temporal, only when needed

```bash
temporal server start-dev --db-filename .temporal/buildwise.db
```

### Terminal 3 — TypeScript apps

From root:

```bash
pnpm dev
```

### Terminal 4 — Python worker, only when needed

```bash
cd apps/ai-worker
uv run fastapi dev src/buildwise_ai/main.py
```

## Expected local URLs

Suggested:

```text
Web       http://localhost:3000
API       http://localhost:4000
Temporal  http://localhost:8233
```

Other dev UIs are optional.

## First diagnostic commands

```bash
docker compose ps
docker compose logs --tail=100 postgres
docker compose logs --tail=100 redis
pnpm typecheck
```

## Database migration

After pulling a branch with migrations:

```bash
pnpm --filter api db:migrate
```

Use the exact script defined in the API package.

## Stop

Stop app dev servers with Ctrl+C.

Then:

```bash
docker compose down
```

Temporal can be stopped with Ctrl+C.

## Reset local database

Destructive:

```bash
docker compose down -v
docker compose up -d
```

Then:

```bash
pnpm --filter api db:migrate
pnpm --filter api db:seed
```

when a seed script exists.

## Before pushing

```bash
pnpm verify
git status
```

Inspect the diff before pushing agent-generated code.
