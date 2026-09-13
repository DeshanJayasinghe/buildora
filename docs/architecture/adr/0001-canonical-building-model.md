# ADR-001 — Canonical Building Model as Single Source of Truth

> **Status:** Accepted
> **Date:** 2026-09-12
> **Deciders:** Product Owner / CTO
> **Supersedes:** none
> **Superseded by:** none

---

## Context

Buildora AI produces many representations of the same building: a 2D plan, a 3D
scene, IFC exports, renderer meshes, quantities, a BOQ, cost estimates, and AI
context. Every AEC platform must decide which of these is authoritative.

The common failure mode is allowing two or more representations to hold
independent state — typically a 2D model and a 3D model that drift apart, or an
IFC file that becomes the de facto internal model. Once that happens, every
feature must reconcile divergent state, and quantities stop being trustworthy.

## Decision

**The Buildora AI Building Model, persisted in PostgreSQL, is the single source
of truth for all design state.**

The following are **derived representations** with no independent authority:

```text
2D plan (PixiJS)
3D scene (Three.js)
IFC import/export
renderer meshes, GLB, Fragments
quantities
BOQ
cost estimates
AI context
```

Correct direction of flow:

```text
Canonical Building Model
    ├─→ 2D representation
    ├─→ 3D representation
    ├─→ IFC export
    ├─→ quantities
    ├─→ BOQ
    ├─→ cost
    └─→ AI context
```

Specifically:

- The Three.js scene is not the database.
- The PixiJS scene is not the database.
- IFC is not the internal editable model (see ADR-018).
- Rendered mesh geometry is not the canonical domain model.
- The client model-session is a synchronized working copy, not a second
  authority (see ADR-008).

Every mutation of design state flows through a semantic domain command
(see ADR-002). No renderer, adapter, or AI component writes domain state
directly.

## Alternatives considered

| Option | Pros | Cons | Why not chosen |
|---|---|---|---|
| A — Canonical parametric model (chosen) | One authority; all outputs derivable; quantities traceable | Requires adapters for every representation | — |
| B — IFC as internal model | Free interoperability | Internal design permanently constrained by IFC representation choices; poor editing ergonomics; slow | Reshapes the product around a interchange format's limitations |
| C — Separate 2D and 3D models with sync | Each optimized for its use | Guaranteed drift; reconciliation logic in every feature; untrustworthy quantities | The failure mode this ADR exists to prevent |
| D — Mesh/BREP as canonical | Direct rendering | Loses building semantics; cannot compute quantities meaningfully; no parametric editing | Quantities and costing are the product's differentiator |

## Consequences

**Positive**

- Quantities, BOQ and cost are always consistent with the design by construction.
- A change in one place updates every representation.
- Element identity is stable across 2D, 3D, IFC, QS, BOQ, cost and audit.
- Future collaboration has one thing to synchronize.

**Negative / accepted cost**

- Every representation needs an adapter, which is real work.
- Adapter update granularity must be engineered deliberately, or performance
  suffers at large model sizes.
- Round-tripping IFC requires explicit external-GUID mapping.

**Neutral**

- The model is parametric, not mesh-based; meshes are generated on demand.

## Migration / implementation impact

No migration — this is the founding decision and no implementation exists.

Owned by: Phase 5 / Sprint 08–11 (Building Model foundations).

Affects: every architecture document, `packages/building-model`,
`packages/model-session`, `packages/cad-2d`, `packages/engine-3d`,
`packages/qs-engine`, `packages/cost-engine`.

## Rollback / exit strategy

**Effectively irreversible** once projects exist. Reversing this decision means
rebuilding the product. The cost of getting it wrong is the reason it is
recorded first.

## Verification

- Architecture review: any PR introducing a second authoritative store of
  design state is a BLOCKER.
- Test: model round-trip — create → persist → reload → deep-equal.
- Test: a wall move updates 2D, 3D, and derived quantities from one command.
- CI: renderer packages may not import persistence or repository code.

## References

- `AGENTS.md` §9, §37, §90
- `CLAUDE.md` §5.1, §18
- `docs/architecture/05_Building_Model_Domain.md`
- ADR-002 (persistence), ADR-008 (model-session), ADR-018 (IFC)
