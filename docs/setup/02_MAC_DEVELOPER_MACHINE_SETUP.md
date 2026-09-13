# macOS Developer Machine Setup

This is the primary workstation guide for Buildora AI.

## 1. Check macOS and architecture

```bash
sw_vers
uname -m
```

Typical CPU output:

```text
arm64
```

for Apple Silicon, or:

```text
x86_64
```

for Intel.

## 2. Install Apple command-line tools

```bash
xcode-select --install
```

Verify:

```bash
xcode-select -p
clang --version
```

These tools will also be useful later for Rust/native compilation.

## 3. Install Homebrew

If Homebrew is not already installed, use the installer from:

```text
https://brew.sh/
```

After installation:

```bash
brew update
brew doctor
```

## 4. Install Git

macOS command-line tools include Git, but Homebrew can provide a newer version:

```bash
brew install git
```

Verify:

```bash
git --version
```

## 5. Install Node 24 LTS

Buildora AI pins Node 24 LTS for the initial development baseline.

You may use the official Node installer or a version manager.

After installation:

```bash
node --version
npm --version
```

Expected Node major:

```text
v24.x.x
```

Create a repository `.nvmrc` later containing:

```text
24
```

even if you personally do not use nvm. It communicates the intended major version.

## 6. Install pnpm

Use one install method only.

Homebrew:

```bash
brew install pnpm
```

or pnpm's official install script:

```bash
curl -fsSL https://get.pnpm.io/install.sh | sh -
```

Restart the terminal if required.

Verify:

```bash
pnpm --version
```

## 7. Install Docker Desktop

Install Docker Desktop for your Mac architecture.

Verify after Docker Desktop is running:

```bash
docker version
docker compose version
docker run --rm hello-world
```

## 8. Useful CLI tools

Recommended:

```bash
brew install jq
brew install ripgrep
brew install tree
brew install gh
```

Optional:

```bash
brew install watch
brew install shellcheck
```

Verify:

```bash
jq --version
rg --version
gh --version
```

## 9. GitHub login

```bash
gh auth login
gh auth status
```

SSH is still recommended for repository Git operations; see the Git setup guide.

## 10. IDEs and agents

Keep one primary editor/workspace at a time when an autonomous agent is actively modifying files.

Recommended rule:

- Codex: main implementation
- Claude: plan/review
- Antigravity: browser/UI QA

Avoid simultaneous edits to the same branch.

## 11. Final machine verification

```bash
git --version
node --version
pnpm --version
docker --version
docker compose version
gh --version
clang --version
```

Do not proceed until all required commands work without shell errors.
