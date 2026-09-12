# Testing Setup

BuildWise requires stronger testing than a normal dashboard because geometry and QS errors can cascade into costs.

## 1. TypeScript unit tests

Use Vitest consistently for internal packages where possible.

Install at workspace root or per package according to monorepo convention:

```bash
pnpm add -Dw vitest @vitest/coverage-v8
```

Test:

- units
- geometry functions
- Building Model commands
- QS
- cost
- versioning.

## 2. Nest API tests

Keep:

- unit tests
- integration tests against test PostgreSQL
- authorization tests
- webhook tests.

## 3. Playwright E2E

Initialize:

```bash
pnpm create playwright
```

Use TypeScript.

Recommended first E2E:

```text
sign in
create project
open project
```

Later:

```text
draw room
switch to 3D
reload
verify persistence
```

## 4. Python

```bash
cd apps/ai-worker
uv add --dev pytest
uv run pytest
```

## 5. Rust

```bash
cargo test
```

## 6. Golden fixtures

Create:

```text
test/fixtures/
├── single-room/
├── door-window/
├── two-room/
├── two-storey/
└── qs-basic-house/
```

## 7. Golden QS test

Example:

```text
Wall:
5000 × 2700

Opening:
1000 × 2100

Expected net wall area:
11.4 m²
```

The authoritative engine must return the expected value.

## 8. Visual tests

Later add screenshot regression for stable editor fixture views.

## 9. Real AI

Do not make ordinary CI rely on live LLM responses.

Use deterministic fake AI providers in CI.
