# Setup Troubleshooting

## Port already in use

Check:

```bash
lsof -nP -iTCP:3000 -sTCP:LISTEN
lsof -nP -iTCP:4000 -sTCP:LISTEN
lsof -nP -iTCP:5432 -sTCP:LISTEN
lsof -nP -iTCP:6379 -sTCP:LISTEN
lsof -nP -iTCP:7233 -sTCP:LISTEN
```

Stop the correct process only after identifying it.

## Docker does not start

Verify Docker Desktop is running:

```bash
docker info
```

## PostgreSQL connection refused

```bash
docker compose ps
docker compose logs postgres
```

Verify `DATABASE_URL`.

## Redis connection refused

```bash
docker compose exec redis redis-cli ping
```

## pnpm workspace package not found

Check:

```text
pnpm-workspace.yaml
package name
workspace:* dependency
```

Then:

```bash
pnpm install
```

## Next.js build works locally but not CI

Check:

- Node major
- pnpm version
- case-sensitive file paths
- uncommitted generated files
- missing environment variables.

## Python import errors

Use:

```bash
uv sync
uv run python -V
uv run pytest
```

Do not install random dependencies globally.

## Rust/WASM not found

Check:

```bash
rustc --version
cargo --version
rustup target list --installed
wasm-pack --version
```

## Temporal history disappears

Start dev server with a DB filename:

```bash
temporal server start-dev --db-filename .temporal/buildora.db
```

## Auth works in web but API rejects

Check:

- token audience/issuer
- server verification
- environment mismatch
- local user/org mirror
- project authorization.

## AI works in a script but not application

Check:

- API key is server-only and loaded
- provider timeout
- model alias config
- usage/credit guard
- tool schema validation
- outbound network.

## Never solve setup problems by

- disabling TypeScript strict mode,
- committing secrets,
- making every route public,
- using `any` everywhere,
- bypassing tenant authorization,
- duplicating Building Model state.
