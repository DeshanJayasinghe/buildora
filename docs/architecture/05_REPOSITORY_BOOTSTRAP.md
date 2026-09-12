# Repository Bootstrap

Run after machine and GitHub setup.

## 1. Create folders

From the repository root:

```bash
mkdir -p apps packages native infrastructure docs/architecture
```

## 2. Copy architecture docs

Place the previously generated architecture pack under:

```text
docs/architecture/
```

The folder should contain the master product reference, implementation guides, and ADRs.

## 3. Copy agent files

Copy from this setup pack:

```text
templates/AGENTS.md  → AGENTS.md
templates/CLAUDE.md  → CLAUDE.md
```

## 4. Initialize package.json

```bash
pnpm init
```

Update root `package.json` to be private and monorepo-oriented.

Recommended baseline:

```json
{
  "name": "buildwise",
  "private": true,
  "version": "0.0.0",
  "engines": {
    "node": ">=24 <25"
  },
  "scripts": {
    "dev": "turbo dev",
    "build": "turbo build",
    "lint": "turbo lint",
    "typecheck": "turbo typecheck",
    "test": "turbo test",
    "verify": "pnpm lint && pnpm typecheck && pnpm test && pnpm build"
  }
}
```

Do not hardcode a pnpm version manually from this document. After installing the current chosen pnpm version, record it:

```bash
pnpm --version
```

and add:

```json
"packageManager": "pnpm@YOUR_INSTALLED_VERSION"
```

## 5. Pin Node major

```bash
echo "24" > .nvmrc
```

Optionally also add:

```text
.node-version
```

with:

```text
24
```

if your local tooling recognizes it.

## 6. Add editor config

Create `.editorconfig`:

```ini
root = true

[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
indent_style = space
indent_size = 2
trim_trailing_whitespace = true

[*.py]
indent_size = 4

[*.md]
trim_trailing_whitespace = false
```

## 7. Create gitignore

Include:

```text
node_modules/
.next/
dist/
coverage/
.env
.env.*
!.env.example
.DS_Store
.turbo/
.venv/
__pycache__/
.pytest_cache/
target/
pkg/
playwright-report/
test-results/
```

## 8. First commit

```bash
git add .
git commit -m "chore(repo): bootstrap BuildWise repository"
git push -u origin main
```

## 9. Do not scaffold every app yet

Proceed to the monorepo guide first so all apps are created inside the intended workspace.
