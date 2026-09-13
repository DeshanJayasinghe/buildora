# Git, GitHub and SSH Setup

## 1. Configure Git identity

```bash
git config --global user.name "YOUR NAME"
git config --global user.email "YOUR_GITHUB_EMAIL"
git config --global init.defaultBranch main
git config --global pull.rebase false
```

Check:

```bash
git config --global --list
```

## 2. Generate SSH key

Recommended:

```bash
ssh-keygen -t ed25519 -C "YOUR_GITHUB_EMAIL"
```

Accept the default file unless you already manage multiple keys.

Start agent:

```bash
eval "$(ssh-agent -s)"
```

On macOS, add the key:

```bash
ssh-add --apple-use-keychain ~/.ssh/id_ed25519
```

## 3. Copy public key

```bash
pbcopy < ~/.ssh/id_ed25519.pub
```

Add it to GitHub:

```text
GitHub → Settings → SSH and GPG keys → New SSH key
```

## 4. Test

```bash
ssh -T git@github.com
```

## 5. Create repository

Recommended:

- Private
- Name: `buildora`
- Do not generate unrelated starter code if creating locally first.

Using GitHub CLI:

```bash
mkdir buildora
cd buildora
git init
gh repo create buildora --private --source=. --remote=origin
```

## 6. Branch model

For a solo developer, keep this simple:

```text
main
└── feature/*
```

A permanent `develop` branch is optional. Do not add process without value.

Use branches such as:

```text
feature/project-domain
feature/building-model-wall
feature/2d-wall-tool
feature/3d-wall-renderer
fix/wall-undo
```

## 7. Worktrees for AI agents

Example:

```bash
mkdir -p ../buildora-worktrees

git worktree add ../buildora-worktrees/wall feature/building-model-wall
git worktree add ../buildora-worktrees/ui feature/dashboard-ui
```

List:

```bash
git worktree list
```

Remove completed worktree:

```bash
git worktree remove ../buildora-worktrees/wall
```

## 8. Commit convention

Use Conventional Commit-style messages:

```text
feat(model): add wall entity
feat(editor): add wall selection
fix(qs): subtract openings from wall area
test(cost): add decimal rounding cases
docs(adr): define canonical geometry unit
```

## 9. Protect main

On GitHub, enable at least:

- pull request before merge where practical,
- CI status checks,
- prevent accidental force push,
- secret scanning if available.

Even as one person, use PRs for larger architectural changes.
