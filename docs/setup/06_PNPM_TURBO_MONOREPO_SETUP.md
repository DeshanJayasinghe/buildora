# pnpm + Turborepo Monorepo Setup

## 1. Create pnpm workspace

Create `pnpm-workspace.yaml`:

```yaml
packages:
  - "apps/*"
  - "packages/*"
```

Rust/Python folders do not need to be pnpm packages unless they contain a JS wrapper.

## 2. Install Turborepo

From the root:

```bash
pnpm add -Dw turbo
```

## 3. Create turbo.json

```json
{
  "$schema": "https://turbo.build/schema.json",
  "tasks": {
    "dev": {
      "cache": false,
      "persistent": true
    },
    "build": {
      "dependsOn": ["^build"],
      "outputs": [".next/**", "dist/**", "!**/.next/cache/**"]
    },
    "lint": {
      "dependsOn": ["^lint"]
    },
    "typecheck": {
      "dependsOn": ["^typecheck"]
    },
    "test": {
      "dependsOn": ["^build"],
      "outputs": ["coverage/**"]
    }
  }
}
```

## 4. Shared package naming

Use internal names such as:

```text
@buildora/domain
@buildora/building-model
@buildora/units
@buildora/api-contracts
@buildora/design-system
@buildora/cad-2d
@buildora/engine-3d
@buildora/qs-engine
@buildora/cost-engine
@buildora/ai-tools
```

## 5. Internal package versioning

For private workspace packages:

```json
{
  "name": "@buildora/domain",
  "private": true,
  "version": "0.0.0"
}
```

Consumers can use:

```json
"@buildora/domain": "workspace:*"
```

## 6. Verify workspace

```bash
pnpm install
pnpm exec turbo --version
pnpm exec turbo run build
```

It is acceptable for the build graph to be mostly empty at this stage.

## 7. Rule for packages

A package exists because it owns a cohesive responsibility, not merely to make the tree look modular.

Do not create micro-packages for trivial helpers.
