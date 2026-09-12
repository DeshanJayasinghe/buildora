# Codex, Claude and Antigravity Setup

BuildWise uses AI coding tools as separate engineering roles.

## 1. Canonical architecture folder

Before substantial coding, place the BuildWise architecture pack at:

```text
docs/architecture/
```

All agents must reference the same files.

## 2. Root AGENTS.md

Copy the template in this pack:

```text
templates/AGENTS.md
```

to:

```text
AGENTS.md
```

Codex must be told to read it before implementing.

## 3. Root CLAUDE.md

Copy:

```text
templates/CLAUDE.md
```

to:

```text
CLAUDE.md
```

Claude should default to planning/review rather than silently redesigning the repository.

## 4. Antigravity guidance

Create a saved workspace instruction that says:

```text
BuildWise architecture is canonical under docs/architecture.
Do not redesign domain boundaries while performing browser/UI verification.
For UI changes, report failures first. Make code changes only when specifically assigned.
```

## 5. Agent responsibility

```text
Claude      → plan / architecture review / debugging
Codex       → implementation / tests / refactors
Antigravity → browser / UI / regression verification
You         → final decisions and merge approval
```

## 6. One feature, one owner

Never have Codex and Claude independently implement the same feature in the same branch.

For difficult features:

```text
Claude: plan only
You: approve
Codex: implement
Claude: review diff
Codex: fix
Antigravity: browser test if UI changed
You: merge
```

## 7. Standard agent task preamble

Use:

```text
Before changing code:
1. Read AGENTS.md.
2. Read docs/architecture/00_README.md.
3. Read docs relevant to this feature.
4. Read applicable ADRs.
5. Inspect current code before proposing new packages.
6. Preserve the BuildWise Building Model as the source of truth.
7. Do not introduce a new architectural dependency without explaining why.
```

## 8. Agent task completion rule

Every implementation prompt should end with:

```text
Before declaring the task complete:
- run the relevant tests,
- run typecheck,
- run lint,
- run build if applicable,
- report files changed,
- report architectural assumptions,
- report anything not completed.
```
