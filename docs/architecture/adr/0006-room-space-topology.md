# ADR-006 — Room / Space Topology: Derived from Interior Wall Faces, Persisted with Stable Identity

> **Status:** Accepted
> **Date:** 2026-09-12
> **Deciders:** Product Owner / CTO
> **Supersedes:** none
> **Superseded by:** none

---

## Context

Rooms were listed as an element type with no boundary semantics. Nothing stated
whether a room is user-drawn or wall-derived, what happens when a bounding wall
moves, whether rooms may overlap, or whether area is entered or computed.

Room area drives floor finishes, ceiling finishes, skirting perimeter and
internal wall finishes — a material share of every cost estimate. The choice
affects the data model, the 2D editor, the QS engine and the update cascade.

A first draft proposed deriving the room boundary from the **wall centreline**
polygon. That was corrected during review: centrelines are correct for
*connectivity*, but wrong for *professional area*. A room bounded by 200 mm walls
measured to centrelines overstates floor area by roughly 100 mm on every edge.

## Decision

Rooms are **derived from wall topology, bounded by interior wall faces, and
persisted with stable identity.**

### Derivation pipeline

```text
wall connectivity / centreline graph
        ↓
identify enclosed regions
        ↓
resolve wall junctions
        ↓
derive interior wall faces
        ↓
derive usable room boundary
        ↓
persist Room / Space with stable ID
        ↓
BOUNDS relationships to bounding walls
```

The **centreline graph is used for connectivity** — finding which regions are
enclosed. The **final persisted boundary is generated from the appropriate
interior wall faces**, offset inward from each bounding wall's centreline by
that wall's half-thickness, with junctions resolved.

This distinction is the substance of this ADR. It is what makes the resulting
areas professionally defensible for:

- floor finish area,
- ceiling finish area,
- skirting perimeter,
- internal wall finish area,
- stated room dimensions,
- QS generally.

### Room / Space structure

```text
Room / Space
- id                    stable UUID, preserved across recomputation
- levelId
- boundary              derived, persisted polygon (interior faces)
- BOUNDS relationships  to bounding walls
- name
- number
- properties / finishes
- user_defined          override flag
- provenance            where appropriate (source, confidence, verification)
```

### Recomputation rules

**`user_defined = false`** — a wall change triggers topology and boundary
recomputation. The room keeps its stable ID, name, number and finishes. If area
changes materially, the room is flagged for user attention.

**`user_defined = true`** — the user's boundary is **never silently overwritten**.
Wall changes may flag the room as potentially stale, but the system does not
replace a boundary the user has taken ownership of.

### Area

Area is **calculated from the boundary**. A user-entered room area is never
stored as authoritative geometry. If a user needs to state a target area, that
is a design input or a property, not the geometric truth.

### Ownership

Room detection lives in **`packages/building-model`** — headless, deterministic,
unit-testable without a renderer. It is not editor code.

### Deliberately left to implementation

The detection algorithm — half-edge planar subdivision versus polygon/face
boolean operations — is an implementation-time choice. Both satisfy this
architecture. Choose on measured performance at realistic model sizes.

**Do not prematurely reach for Rust/WASM.** Implement in TypeScript first;
Rust is profiling-gated (ADR-012).

## Alternatives considered

| Option | Pros | Cons | Why not chosen |
|---|---|---|---|
| A — User-drawn polygons only | Simple | Diverges from walls on the first edit; areas silently wrong | Untrustworthy quantities |
| B — Fully derived, never persisted | Always consistent | No stable ID, name, number or finishes; no QS lineage | Rooms must be nameable and costable |
| C — Derived from centrelines, persisted | Simple derivation | Overstates floor area by half the wall thickness on every edge; not professionally defensible | Corrected during review |
| D — Derived from interior faces, persisted, with override (chosen) | Correct professional areas; stable identity; stays consistent; user can take control | Junction resolution is real geometry work | — |

## Consequences

**Positive**

- Floor, ceiling and skirting quantities are professionally correct.
- Rooms keep stable identity across wall edits, so QS lineage and finishes survive.
- Users can override where the automatic result is wrong, without fighting the system.

**Negative / accepted cost**

- Junction resolution (how two walls of different thickness meet at a corner) is
  genuine geometric complexity and the most likely source of subtle bugs.
- The recompute cascade must run on every bounding-wall change.
- Overridden rooms can drift from geometry — accepted, and surfaced by flagging.

**Neutral**

- Rooms on a level must not overlap; enforced as a domain invariant.

## Migration / implementation impact

No migration — decided before rooms exist.

Affects: `docs/architecture/05_Building_Model_Domain.md`,
`06_2D_CAD_Editor_Implementation.md`, `15_Database_and_Data_Architecture.md`,
`packages/building-model`, QS finishes rules.

Owned by: Phase 6 / Sprint 15 (rooms in 2D), with the domain algorithm landing in
`packages/building-model` alongside Sprint 09–11 wall work.

## Rollback / exit strategy

Changing from interior-face to centreline boundaries (or back) after projects
exist would silently change every finishes quantity and therefore every cost
estimate containing finishes. Treat as irreversible once quantities are issued
to customers.

## Verification

- **Golden test:** a 5000 × 4000 mm room bounded by 200 mm walls measured to
  centrelines yields 20.00 m²; measured to interior faces yields 4800 × 3800 =
  18.24 m². The engine must produce **18.24 m²**.
- Test: room retains its stable ID, name and finishes after a bounding wall moves.
- Test: `user_defined = true` room boundary is unchanged after a bounding wall moves.
- Test: rooms on one level do not overlap.
- Test: junction resolution for walls of differing thickness at a corner.
- Review: any user-entered area stored as authoritative geometry is a BLOCKER.

## References

- `docs/architecture/05_Building_Model_Domain.md`
- OD-03 in `docs/architecture/OPEN_ARCHITECTURE_DECISIONS.md`
- ADR-001, ADR-005, ADR-009, ADR-015
