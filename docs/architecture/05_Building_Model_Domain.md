# 05 — Canonical Building Model Domain

> **Status:** FROZEN — 2026-09-12
> **Owns:** the canonical domain model, its hierarchy, geometry semantics,
> coordinate system, element definitions, invariants and commands.
> **Governed by:** ADR-001, ADR-002, ADR-005, ADR-006, ADR-007, ADR-019, ADR-020
> **Implemented in:** `packages/building-model` (framework-free)

This document is the authority on *what the Building Model is*.
Persistence detail lives in `15_Database_and_Data_Architecture.md`.
Client-side working state lives in ADR-008.

---

# 1. The Central Rule

**The Buildora AI Building Model is the single source of truth for all design
state** (ADR-001).

```text
Canonical Building Model
    ├─→ 2D representation      (PixiJS, derived)
    ├─→ 3D representation      (Three.js, derived)
    ├─→ IFC export             (interchange, derived)
    ├─→ quantities             (derived, deterministic)
    ├─→ BOQ                    (derived)
    ├─→ cost                   (derived)
    └─→ AI context             (derived)
```

None of the derived representations holds independent authority. The Three.js
scene is not the database. The PixiJS scene is not the database. IFC is not the
internal model. Meshes are not canonical geometry.

---

# 2. Domain Purity

`packages/building-model` is **framework-free** (ADR-011). It must not import:

```text
React    Next.js    PixiJS    Three.js
NestJS   Drizzle    OpenAI    Stripe    Clerk
```

The entire model — geometry, validation, commands, room topology — must be
testable in a plain Node process with no browser, no database and no GPU.
This is enforced in CI.

---

# 3. Hierarchy

```text
Project                  organization-owned; currency, unit display, datum
 └── Site                optional; real-world location and georeference
     └── Building        one or more per site
         └── Level       elevationMm from project datum; ordered
             ├── Building Elements    wall · door · window · slab
             │                        column · beam · stair · roof
             ├── Spaces / Rooms       derived-then-persisted (§7.4)
             └── FF&E Instances       furniture · fixture · equipment
                                      (ADR-019)
```

## Element classes

Every element carries a **class**, and the class governs which subsystems see it:

| Class | Examples | In room topology? | In construction quantities? |
|---|---|---|---|
| `BUILDING` | wall, door, window, slab, column, beam, stair, roof | Yes | Yes |
| `SPACE` | room | Is the output | Drives finish quantities |
| `FFE` | furniture, fixture, equipment | **No** | **No** — separate cost section |

**FF&E never bounds a room and never affects a construction quantity** (ADR-019).
Every element iteration in QS and topology code must honour the class; a missed
check puts a sofa in a wall area.

`Site` carries real-world georeference. **Building-local geometry never carries
real-world coordinates** (ADR-005 §Georeference separation).

---

# 4. Units and Coordinates

## 4.1 Canonical unit

**Millimetres.** Always. All stored geometry, all domain arithmetic.

Display units (mm, cm, m, ft/in) are a presentation concern. Conversion is
isolated in `packages/units` and must not be scattered through UI code
(`AGENTS.md` §10).

## 4.2 Datum (ADR-005)

```text
Project-local datum:  ground-floor finished floor level (FFL) = 0 mm
```

## 4.3 Level

```text
Level
- id
- buildingId
- name
- elevationMm              signed, from project datum (basements negative)
- order / index
- defaultStoreyHeightMm    OPTIONAL design aid — NOT authoritative geometry
```

`elevationMm` is the **single authoritative vertical fact** for a level.

## 4.4 Derived vertical values

```text
floorToFloor(level_n) = level_{n+1}.elevationMm − level_n.elevationMm
```

Floor-to-floor is **computed**, never stored as an independent authority.
Editing floor-to-floor in the UI writes the *next* level's `elevationMm`.

**Slab structural thickness belongs to the Slab element and its assembly** — not
to `Level`.

## 4.5 Element z

Element `z` coordinates are **level-relative**:

```text
absoluteZ(element) = level.elevationMm + (element.baseOffsetMm ?? 0)
```

Absolute coordinates are derived on read by adapters and exporters. They are
never stored on the element.

Every geometry schema version states its datum assumption explicitly.

---

# 5. Element — Common Structure

Every element, without exception, carries:

```text
id                          UUID, stable across model/2D/3D/IFC/QS/BOQ/cost/audit
organization_id             tenant ownership (ADR-016)
project_id
element_class               BUILDING | SPACE | FFE          (ADR-019)
element_type                WALL | DOOR | WINDOW | ROOM | SLAB | COLUMN |
                            FURNITURE | FIXTURE | EQUIPMENT | ...
level_id
host_element_id             nullable — e.g. a door's wall
parent_element_id           nullable
element_type_id             nullable — type/instance identity (ADR-007)
assembly_version_id         nullable — cost expansion

geometry_schema_version     + geometry_jsonb
properties_schema_version   + properties_jsonb

source_kind                 provenance: USER | AI | IMPORT | RECOGNITION
source_ref                  nullable
confidence                  nullable
verification_status

created_model_version
updated_model_version
deleted_model_version       nullable
```

## 5.1 Stable IDs

An element's ID is the **same identifier** in the model, 2D scene, 3D scene, IFC
mapping, quantities, BOQ lines, cost items, AI tool results and audit records.

Renderers must never generate replacement identities (`AGENTS.md` §12).

## 5.2 JSONB discipline

JSONB is permitted for parametric geometry and flexible properties, but never as
an untyped dumping ground (`AGENTS.md` §26). Every `element_type` has:

```text
a geometry schema     a properties schema
a schema version      runtime validation before persistence
```

## 5.3 Parametric, not mesh

**Never persist vertices, normals, indices or triangles as canonical geometry.**
Store the parametric definition; generate Pixi representations, Three.js meshes,
GLB, Fragments and IFC from it.

## 5.4 Three identity axes (ADR-007, ADR-020)

```text
element_type_id       → WHAT IT IS          geometric type, schedules, IFC type
assembly_version_id   → WHAT IT'S BUILT OF  construction build-up, waste, labour
finish_assignment     → WHAT IT LOOKS LIKE  surface finish (ADR-020)
```

These are three different facts and must never be conflated. Repainting a wall
creates a **new finish assignment**, never a new assembly version.

For FF&E a fourth reference applies — `ffe_asset_version_id`, the catalogue
product (ADR-019).

## 5.5 Finish assignments are surface-scoped (ADR-020)

A wall has two faces and they are routinely finished differently. A finish binds
to an element **surface**, not the whole element:

```text
finish_assignments
  element_id · surface (INTERIOR_A | INTERIOR_B | EXTERIOR |
                        TOP | BOTTOM | SOFFIT | ALL)
  room_id (nullable) · material_version_id · coverage
  provenance · created_model_version · deleted_model_version
```

This expresses a feature wall in one room without splitting the wall element.

A material carries **both** visual properties (colour, texture, roughness — read
by the renderer) **and** construction properties (unit, coverage rate, coats,
waste, labour, rate reference — read by QS and cost). Neither side invents the
other's data.

**Whether a small opening is deducted from a finish area is decided by the QS
ruleset** (ADR-009), never by the finish assignment or a domain constant.

---

# 6. Relationships

Host and parent are columns. Everything else is a first-class row:

```text
element_relationships
- source_element_id
- target_element_id
- relationship_type      CONNECTS_TO | BOUNDS | SUPPORTED_BY | SERVES | RELATED_TO
- properties_jsonb
- created_model_version
- deleted_model_version
```

Do not store queryable graph relationships inside element JSON.

---

# 7. Element Definitions and Invariants

Architecture-level structures and their invariants. Full field lists belong to
implementation.

## 7.1 Wall

```text
start, end        centreline (mm, level-relative)
thicknessMm
heightMm
baseOffsetMm      optional
assemblyId
```

**Invariants**
- `thicknessMm > 0`; `heightMm > 0`; `start ≠ end`.
- The **centreline is authoritative**. Faces are derived, never stored.
- Net area = gross area − Σ hosted opening areas.
- Junction resolution is derived at render/QS time, never persisted as separate
  geometry.

## 7.2 Door / Window

```text
host_element_id   a wall (required)
offsetMm          along host centreline
widthMm, heightMm
sillHeightMm      windows
swing / handing   doors
element_type_id   optional
```

**Invariants**
- Host must exist and be a wall.
- The opening footprint lies entirely within the host's extent and height.
- Deleting a host deletes or explicitly orphans hosted elements — never silently.
- Moving a host moves hosted elements (§9 cascade).
- **The hosted element is the single representation of its opening** (§7.3).

## 7.3 Opening — single representation rule

**A door or window IS the opening.** Its hosted geometry creates the hole. There
is no second `Opening` element for the same physical hole.

A standalone `opening` element exists **only** for fixture-less holes:

```text
service penetration    arch    unfilled wall opening
```

**QS sees hosted doors/windows and standalone openings through one single
opening/deduction abstraction.** Two code paths for opening deduction would
produce divergent net areas — the most important QS number.

## 7.4 Room / Space (ADR-006)

```text
id                stable UUID, preserved across recomputation
levelId
boundary          derived, persisted polygon — INTERIOR WALL FACES
BOUNDS            relationships to bounding walls
name, number
properties / finishes
user_defined      override flag
provenance        where appropriate
```

### Derivation pipeline

```text
wall connectivity / centreline graph
        ↓  (centrelines used for CONNECTIVITY)
identify enclosed regions
        ↓
resolve wall junctions
        ↓
derive interior wall faces
        ↓  (interior faces used for the BOUNDARY)
derive usable room boundary
        ↓
persist Room / Space with stable ID
        ↓
BOUNDS relationships to bounding walls
```

The centreline graph finds *which* regions are enclosed. The persisted boundary
is generated from **interior wall faces**, offset inward from each bounding
wall's centreline by that wall's half-thickness, with junctions resolved.

This distinction is what makes floor finish area, ceiling finish area, skirting
perimeter, internal wall finish area and stated room dimensions professionally
correct.

**Invariants**
- Boundary is a closed, non-self-intersecting polygon.
- **Area is calculated from the boundary.** A user-entered area is never stored
  as authoritative geometry.
- `user_defined = false` → wall change triggers recomputation; stable ID, name,
  number and finishes are preserved; material area change is flagged.
- `user_defined = true` → the boundary is **never silently overwritten**; the
  room may be flagged as potentially stale.
- Rooms on one level must not overlap.

Room detection lives in `packages/building-model` — headless and testable, not
editor code. The algorithm (half-edge planar subdivision vs polygon/face boolean)
is an implementation choice. **Do not prematurely reach for Rust/WASM** (ADR-012).

## 7.5 Slab

```text
boundary polygon
thicknessMm
baseOffsetMm       from level
assemblyId
voids              e.g. stair openings
```

**Invariants**
- Polygon closed and non-self-intersecting; `thicknessMm > 0`.
- Voids lie within the boundary.
- Volume = (area − void area) × thickness.

## 7.6 Column

```text
insertion point
widthMm / depthMm, or radius
heightMm
baseOffsetMm
assemblyId
```

**Invariants**
- Positive section; `heightMm > 0`.
- Spans multiple levels only via an explicit top-level reference — never by
  implicit extension.

## 7.7 Beam

```text
start, end        centreline
section dimensions
baseOffsetMm      typically to underside of slab
assemblyId
```

**Invariants**
- `start ≠ end`; positive section.
- `SUPPORTED_BY` relationships to columns/walls are explicit, never inferred at
  QS time.

## 7.8 Stair

```text
baseLevelId, topLevelId
run path
treadDepthMm, riserHeightMm, widthMm
riser count
```

**Invariants**
- `riserCount × riserHeightMm` equals the level-to-level rise within tolerance —
  where the rise is `topLevel.elevationMm − baseLevel.elevationMm` (ADR-005).
- Spans exactly two levels.
- Requires a matching slab void.

## 7.9 FF&E Instance (ADR-019)

```text
element_class         FFE
element_type          FURNITURE | FIXTURE | EQUIPMENT
ffe_asset_version_id  which catalogue product version
room_id               which room it sits in (nullable)
geometry              positionMm · rotationDeg · scale · baseOffsetMm
                      optional dimension override
```

**Invariants**
- References a catalogue asset **by version**, so catalogue changes never alter a
  placed instance.
- **Never bounds a room. Never contributes to a construction quantity.**
- Placement should respect room bounds, wall intersection and door-swing
  clearance — these are deterministic constraints, not AI.
- Priced from the catalogue product record, never from its mesh.

## 7.10 Roof

```text
boundary polygon
pitch / plane definition
thicknessMm
assemblyId
```

**Invariant**
- Area is measured **on the sloping plane**, not the plan projection. A classic
  QS error; covered by an explicit test.

---

# 7.11 Saved Views — presentation, not geometry

A saved view captures camera, visibility, section planes and (for walkthrough)
position. It records the `model_version` it was taken against so a stale view can
be **flagged**, never so it can override the model.

**Saved views are presentation state. They never mutate canonical geometry.**

---

# 8. Commands (ADR-002)

Meaningful model changes are **semantic commands**, never arbitrary scene mutation.

```text
CREATE_WALL      MOVE_WALL        UPDATE_WALL_PROPERTIES
CREATE_DOOR      RESIZE_OPENING   DELETE_ELEMENT
CREATE_LEVEL     ...

SET_ELEMENT_FINISH                                          (ADR-020)

PLACE_FURNITURE  MOVE_FURNITURE   ROTATE_FURNITURE          (ADR-019)
REMOVE_FURNITURE SET_FURNITURE_FINISH
```

A command carries enough for validation, versioning, audit, undo/redo, AI
proposal and collaboration. Every command carries `command_schema_version` so the
journal stays replayable as the domain evolves.

## 8.1 Concurrency

Every mutation carries `baseVersion`. The server verifies the current version
inside the transaction; a stale write returns `MODEL_VERSION_CONFLICT`. Unrelated
geometry edits are never silently merged.

## 8.2 Write granularity

```text
drag preview in browser  (no command)
        ↓
drag end
        ↓
one domain command → one model version
```

Never one command per pointer move.

---

# 9. Cascades

One command is one atomic transaction **including its cascade**. The cascade set
is explicit per element type — it is not "whatever the renderer noticed".

**Moving a wall cascades to:**

```text
hosted openings (doors, windows, standalone openings)
BOUNDS-related rooms (recompute boundary unless user_defined)
connected walls at junctions
```

Adapters consume `model_change_items` and apply incremental updates — they do not
rebuild the whole scene, and they do not guess the cascade.

---

# 10. Provenance and Confidence

Every element carries `source_kind`, `source_ref`, `confidence` and
`verification_status` from creation — not retrofitted when recognition ships.

Low-confidence recognition results require user review. **Never silently convert
uncertain AI or recognition output into authoritative geometry**
(`AGENTS.md` §40).

---

# 11. What the Model Must Never Become

```text
✗ a second authoritative model in 2D or 3D
✗ a mesh store
✗ an IFC document
✗ a Zustand store
✗ directly writable by an LLM
✗ mutable without a version
✗ dependent on React, Pixi, Three, Nest or Drizzle
```

---

# 12. Golden Tests

These must exist and must pass.

| Test | Expected |
|---|---|
| Net wall area: 5000 × 2700 wall, 1000 × 2100 opening | gross 13.5 m², deduction 2.1 m², **net 11.4 m²** |
| Room area: 5000 × 4000 room bounded by 200 mm walls | **18.24 m²** (interior faces: 4800 × 3800), *not* 20.00 m² |
| Model round-trip | create → persist → reload → deep-equal |
| Concurrency | two commands on one `baseVersion` → exactly one succeeds |
| Cascade | wall move updates hosted doors, bounded rooms, 2D and 3D |
| Stair validation | riser count × riser height = level-to-level rise |
| Roof area | measured on slope, not plan projection |

---

# 13. References

- ADR-001 (canonical model), ADR-002 (persistence), ADR-005 (datum),
  ADR-006 (rooms), ADR-007 (types), ADR-008 (model-session),
  ADR-009 (QS rulesets), ADR-015 (lineage)
- `docs/architecture/15_Database_and_Data_Architecture.md` §29–§47
- `AGENTS.md` §9–§15, §37, §40
- `CLAUDE.md` §14, §15
