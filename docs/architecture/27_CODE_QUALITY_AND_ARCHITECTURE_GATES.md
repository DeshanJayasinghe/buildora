# Code Quality and Architecture Gates

## 1. Root verify command

Target:

```bash
pnpm verify
```

It should eventually run:

```text
lint
typecheck
unit tests
integration tests
architecture tests
build
```

## 2. TypeScript

Use strict TypeScript.

Avoid:

```text
any
```

at domain boundaries.

External/untrusted input begins as:

```text
unknown
```

and is validated.

## 3. ESLint

Use the framework-generated ESLint baselines and add architecture-specific restrictions gradually.

## 4. Python

```bash
uv run ruff check .
uv run pyright
uv run pytest
```

## 5. Rust

```bash
cargo fmt --check
cargo clippy --all-targets --all-features -- -D warnings
cargo test
```

## 6. Architecture dependency rules

Enforce conceptually:

```text
domain            → no React, Three.js, PixiJS, DB, OpenAI
building-model    → no renderer dependency
qs-engine         → no OpenAI SDK
cost-engine       → no OpenAI SDK
cad-2d            → may depend on building-model interfaces
engine-3d         → may depend on building-model interfaces
web               → application/UI composition
api               → orchestration/infrastructure adapters
```

## 7. PR checklist

Before merge:

- architecture docs read
- no duplicate source of truth
- tests pass
- `pnpm verify` passes
- schema migration reviewed
- authorization reviewed
- secrets absent
- observability considered
- AI usage tracked if applicable.
