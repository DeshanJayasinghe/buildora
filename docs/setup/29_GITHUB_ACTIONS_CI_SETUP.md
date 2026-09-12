# GitHub Actions CI Setup

Goal: every pull request proves the repository remains healthy.

## 1. Workflow

Create:

```text
.github/workflows/ci.yml
```

## 2. Trigger

```yaml
on:
  pull_request:
  push:
    branches: [main]
```

## 3. Node

Use Node 24.x in CI.

Install pnpm according to the pinned package-manager version.

Cache pnpm store where appropriate.

## 4. Main jobs

Initial:

```text
install
lint
typecheck
test
build
```

Later:

```text
python-test
rust-test
e2e
migration-check
architecture-check
```

## 5. Playwright

Use Linux CI and install browsers/dependencies per Playwright's current CI guidance.

## 6. Service containers

For integration tests, CI can run PostgreSQL and Redis service containers.

Do not connect CI tests to production databases.

## 7. Environment

CI uses dedicated test secrets only when necessary.

Most unit tests should require no external secrets.

## 8. AI

Live AI tests are not part of mandatory PR CI.

If you add scheduled real-provider checks:

- cap spend
- isolate credentials
- do not fail ordinary development due to nondeterministic model wording.

## 9. Required status

Protect `main` so CI must pass before merge where your GitHub plan/settings allow it.
