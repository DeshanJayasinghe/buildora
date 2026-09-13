# ADR-005 — Building-Local Datum and Level-Relative Element Geometry

> **Status:** Accepted
> **Date:** 2026-09-12
> **Deciders:** Product Owner / CTO
> **Supersedes:** none
> **Superseded by:** none

---

## Context

Wall geometry carries `heightMm` and a `levelId`. A `levels` table exists. No
document stated whether an element's `z` is measured from the level datum or the
project datum, where level elevation lives, or where floor-to-floor height is
stored.

Every 3D adapter, volume quantity, stair calculation and IFC export depends on
this. Changing it after projects exist invalidates every stored geometry payload.

A first draft of this decision also proposed storing `elevation_mm`,
`floor_to_floor_mm` and `structural_thickness_mm` on `Level` together. That was
corrected during review: it creates duplicate authoritative values for the same
physical fact.

## Decision

### Project-local datum

```text
Ground-floor finished floor level (FFL) = 0 mm
```

All building-local coordinates are millimetres (ADR — units are fixed by
`AGENTS.md` §10 and `packages/units`).

### Level

```text
Level
- id
- buildingId
- name
- elevationMm          signed, from project-local datum (basements negative)
- order / index
- defaultStoreyHeightMm   optional, design aid only — NOT authoritative geometry
```

`elevationMm` is the **single authoritative vertical fact** for a level.

### Derived, never stored as independent authority

```text
floorToFloor(level_n) = level_{n+1}.elevationMm − level_n.elevationMm
```

Floor-to-floor height is **computed from adjacent level elevations**. It is not
a second authoritative column.

`defaultStoreyHeightMm` may exist as a *design default* used when creating new
elements (for example, a default wall height). It is an input convenience, never
a source of truth for the built geometry.

**Slab structural thickness belongs to the Slab element and its assembly**, not
to `Level`. A level does not own the thickness of the slab at its datum.

### Element geometry

Element `z` coordinates are **level-relative**. An element's base sits at its
level's `elevationMm` unless it carries an explicit `baseOffsetMm`.

```text
absoluteZ(element) = level.elevationMm + element.baseOffsetMm (default 0)
```

Absolute coordinates are derived on read by adapters and exporters; they are
never stored on the element.

Every geometry schema version states its datum assumption explicitly.

### Georeference separation

Real-world site coordinates (latitude/longitude, national grid, site survey
datum) are **separate from building-local project coordinates**. They live on
`Site`, not on elements, and are the concern of the site/GIS phase (PostGIS,
deferred — ADR-012).

Building-local geometry never carries real-world coordinates.

## Alternatives considered

| Option | Pros | Cons | Why not chosen |
|---|---|---|---|
| A — Level-relative (chosen) | Matches IFC `IfcBuildingStorey`; moving a level moves its contents in one write; natural editing model | Absolute z requires a lookup | — |
| B — Project-relative (absolute z on every element) | Trivial rendering | Moving a level rewrites every child element; version churn; conflict-prone | Expensive and error-prone on a routine edit |
| C — Both stored | Fast reads either way | Two authorities for one fact; guaranteed drift | Violates single-source-of-truth |
| D — Level owns elevation + floor-to-floor + slab thickness | Convenient reads | Three overlapping authorities; `floor_to_floor` contradicts the next level's elevation the moment either changes | Corrected during review; duplicate authority |

## Consequences

**Positive**

- One authoritative vertical fact per level; no possible contradiction between
  elevation and floor-to-floor.
- Moving a level is a single-row update.
- Maps cleanly to IFC storey semantics for export.
- Stair riser validation has an unambiguous rise to check against
  (`nextLevel.elevationMm − level.elevationMm`).

**Negative / accepted cost**

- Adapters and exporters must resolve absolute z rather than reading it.
- Floor-to-floor is a computed value; UI must derive it for display and for
  editing (editing floor-to-floor writes the *next* level's elevation).

**Neutral**

- `defaultStoreyHeightMm` remains available as a creation-time convenience.

## Migration / implementation impact

No migration — decided before any geometry is stored.

Affects: `docs/architecture/05_Building_Model_Domain.md`,
`15_Database_and_Data_Architecture.md` §30–§31, `packages/building-model`,
`packages/engine-3d` adapters, QS volume rules, IFC export.

Owned by: Phase 5 / Sprint 08–09.

## Rollback / exit strategy

**Effectively irreversible** once geometry is stored — reversal rewrites every
element payload. This is why it is decided before Sprint 09.

## Verification

- Test: `absoluteZ` resolution for an element on a non-zero level, including a
  negative (basement) elevation.
- Test: moving a level updates rendered and exported positions of all its
  elements without rewriting element rows.
- Test: stair riser count × riser height equals the level-to-level rise within
  tolerance.
- Review: any `floor_to_floor_mm` column added as authoritative geometry is a
  BLOCKER.
- Review: any real-world coordinate on a building element is a BLOCKER.

## References

- `AGENTS.md` §10
- `docs/architecture/05_Building_Model_Domain.md`
- OD-02 in `docs/architecture/OPEN_ARCHITECTURE_DECISIONS.md`
- ADR-001, ADR-006
