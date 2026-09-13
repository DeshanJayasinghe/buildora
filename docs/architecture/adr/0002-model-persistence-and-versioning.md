# ADR-002 — Building Model Persistence: Current State + Commands + Change Items + Snapshots

> **Status:** Accepted
> **Date:** 2026-09-12
> **Deciders:** Product Owner / CTO
> **Supersedes:** none
> **Superseded by:** none

---

## Context

The Building Model must support undo/redo, audit, AI preview, rollback, version
comparison ("what did this revision cost?"), and eventually collaboration.

Pure mutable state cannot answer historical questions. Full event sourcing
answers them but makes ordinary reads expensive and the system harder for one
developer to operate and debug.

## Decision

Persist the Building Model as **four cooperating structures**:

```text
Current materialized state      building_elements, element_relationships
        +
Append-only command journal     model_commands
        +
Per-version change items        model_change_items
        +
Periodic snapshots              model_snapshots
        +
Version metadata                model_versions
```

**Concurrency.** Every mutation carries `baseVersion`. The server verifies the
project's current version inside the transaction, validates the command, applies
it, increments the version, and records the command and change items — atomically.
A stale write returns `MODEL_VERSION_CONFLICT`. Unrelated geometry edits are
never silently merged.

**Immutability.** Committed model versions are never mutated. History is
append-only.

**Atomicity.** One committed model action is one transaction, including its
cascade (for example: move wall + reposition hosted doors + recompute bounded
rooms + increment version + write change items).

**Snapshot policy (configurable, not a magic constant).** Default: approximately
every **50** committed versions, and always before:

- large imports,
- large AI commits,
- any unusually large model transformation.

The threshold is configuration, tuned against measured replay cost.

**Write granularity.** High-frequency interaction is previewed client-side; one
domain command is emitted per completed gesture, not per pointer move.

**AI ChangeSet commit.** `baseVersion` is re-checked at approval/commit time, not
only at ChangeSet creation (see ADR-013).

## Alternatives considered

| Option | Pros | Cons | Why not chosen |
|---|---|---|---|
| A — Current state + journal + change items + snapshots (chosen) | Fast normal reads; full audit; cheap diffs; bounded recovery cost | Four structures to maintain | — |
| B — Mutable current state only | Simplest | No undo, audit, preview, rollback, or revision comparison | Removes headline product capabilities |
| C — Full event sourcing | Complete history; strong replay | Every read needs projection; operationally heavy for one developer | Cost exceeds benefit at this scale |
| D — Full version copy per change | Trivial rollback | Storage explodes; diffs still need computing | Unbounded storage growth |

## Consequences

**Positive**

- Normal reads hit materialized state — fast.
- Undo/redo, audit, AI preview, rollback and revision comparison all fall out.
- `model_change_items` makes revision comparison cheap, which is a headline
  feature (revision cost analysis).
- Snapshots bound worst-case reconstruction cost.

**Negative / accepted cost**

- Four structures must stay consistent — enforced by writing them in one
  transaction.
- Command payloads need schema versioning to stay replayable as the domain
  evolves (`command_schema_version` is mandatory).
- Storage grows with history; journal and audit tables are partitioning
  candidates at scale.

**Neutral**

- Undo is implemented as a compensating command, not by mutating history.

## Migration / implementation impact

No migration — greenfield.

Owned by: Phase 5 / Sprint 11 (Versioning, undo/redo & persistence).

Affects: `docs/architecture/15_Database_and_Data_Architecture.md` §37–§45,
`packages/building-model`, `apps/api` model module.

## Rollback / exit strategy

**Effectively irreversible** once projects carry history. History cannot be
reconstructed retroactively — this is precisely why it is built from the first
wall rather than added later.

## Verification

- Test: two concurrent commands on the same `baseVersion` — exactly one
  succeeds, the other returns `MODEL_VERSION_CONFLICT`.
- Test: model round-trip after N commands equals replay from the last snapshot
  plus subsequent commands.
- Test: a committed version is never updated (repository exposes no update path).
- Review: any destructive mutation pattern that makes history impossible is a
  BLOCKER.

## References

- `AGENTS.md` §13, §14, §15
- `docs/architecture/15_Database_and_Data_Architecture.md` §37–§45
- ADR-001, ADR-013, ADR-015
