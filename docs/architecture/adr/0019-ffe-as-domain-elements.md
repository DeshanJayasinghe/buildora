# ADR-019 — FF&E (Furniture, Fixtures & Equipment) as First-Class Domain Elements

> **Status:** Accepted
> **Date:** 2026-09-12
> **Deciders:** Product Owner / CTO
> **Supersedes:** none
> **Superseded by:** none

---

## Context

Buildora AI's 3D workspace must support interior walkthrough, manual furnishing,
and later AI-assisted interior design ("furnish this living room under £4,000").

Furniture, fixtures and equipment are not structural building elements. They are
not load-bearing, they do not participate in wall/room topology, and they are
priced from a product catalogue rather than a construction assembly.

The tempting shortcut is to treat placed furniture as renderer state — Three.js
objects held in the scene and serialized alongside a saved view. That would make
FF&E a **second authoritative store of design state**, violating ADR-001, and
would put furniture outside versioning, undo, audit, tenancy and cost lineage.

A second question: FF&E has a two-level identity — a reusable catalogue product
("Ikea KIVIK 3-seat sofa") and a specific placed instance ("the sofa at
x=2200mm, rotated 90°, in room G.04"). Conflating those makes catalogue updates
rewrite project data.

## Decision

**FF&E instances are first-class Building Model elements**, subject to every rule
that governs walls and doors.

### Element classification

The canonical model gains an explicit element **class** distinguishing structural
building fabric from interior objects:

```text
Level
├── Building Elements      wall · door · window · slab · column · beam · stair · roof
├── Spaces / Rooms         derived-then-persisted (ADR-006)
└── FF&E Instances         furniture · fixture · equipment
```

FF&E instances live in `building_elements` with an element class of `FFE`, so
they inherit — without exception:

- a stable UUID used across model, 2D, 3D, schedules, cost and audit,
- organization and project tenancy (ADR-016),
- semantic commands and `baseVersion` concurrency (ADR-002),
- version lineage, undo/redo, audit,
- provenance (`source_kind`, `source_ref`, `confidence`, `verification_status`).

**FF&E is never renderer-only state.** A Three.js object representing a sofa is a
disposable view of an FF&E element, exactly as a wall mesh is a view of a wall.

### Catalogue asset vs placed instance

Two distinct records, deliberately separated:

```text
ffe_assets                      the catalogue product (tenant or global)
  id · name · category · manufacturer · product_code
  nominal dimensions · thumbnail · model_asset_object_id
  default finishes · tags · version

ffe_instances                   a placement in a project (an element)
  element id (UUID) · asset_id · project · level · room
  position_mm · rotation_deg · scale · dimension overrides
  finish overrides · provenance
```

The instance references the asset **by version**, so a catalogue change never
silently alters an existing project's geometry or price.

### Three identity axes, not two

ADR-007 established that *what a thing is* (`element_type_id`) is separate from
*what it costs to build* (`assembly_version_id`). FF&E adds a third axis, and all
three must stay distinct (see ADR-020):

```text
element_type_id       WHAT IT IS         geometric/schedule type
assembly_version_id   WHAT IT'S BUILT OF construction assembly (fabric)
finish_assignment     WHAT IT LOOKS LIKE visual finish (ADR-020)
ffe_asset_version_id  WHICH PRODUCT      catalogue item (FF&E only)
```

### Large assets live in object storage

GLB/GLTF models and textures are stored in object storage with a database
metadata reference (`storage_objects`). **Binary 3D assets are never stored in
relational columns.** The browser loads them lazily and disposes them explicitly.

**A GLB file is not financial truth.** The asset's cost comes from its catalogue
product record, not from the mesh.

### FF&E participates in cost as a separate section

FF&E cost is real but structurally different from construction cost: it is
product unit price × count, not quantity × assembly rate. It therefore forms its
**own BOQ/cost section**, so a construction estimate can be produced with or
without furnishing.

FF&E cost pins its lineage like everything else (ADR-015): the estimate records
which catalogue version and which price applied.

### Commands

FF&E mutations are ordinary semantic commands in the same journal:

```text
PLACE_FURNITURE · MOVE_FURNITURE · ROTATE_FURNITURE
REMOVE_FURNITURE · SET_FURNITURE_FINISH
```

There is **no separate interior undo stack**. Placing a sofa and moving a wall
occupy the same version history.

### What FF&E does NOT do

- It does **not** participate in room topology derivation (ADR-006). Furniture
  never bounds a room.
- It does **not** affect wall net area, floor area, or any construction quantity.
- It does **not** block a construction-only estimate.

## Alternatives considered

| Option | Pros | Cons | Why not chosen |
|---|---|---|---|
| A — FF&E as renderer/scene state | Fast to build; no schema work | Second authoritative store (violates ADR-001); no versioning, undo, audit, tenancy or cost; lost on reload | The failure this architecture exists to prevent |
| B — FF&E in a separate table outside the model | Clean separation of concerns | Needs its own versioning, undo, tenancy and audit — duplicating the model's machinery; two histories to reconcile | Duplicate infrastructure for no benefit |
| C — FF&E as ordinary building elements with no class distinction | Simplest schema | Furniture would enter room topology and construction quantities; a sofa would affect wall area | Corrupts QS |
| D — FF&E as classed Building Model elements (chosen) | One history, one undo, one tenancy model; QS isolation via class; catalogue/instance split protects projects from catalogue drift | One more concept; class must be honoured by QS and topology code | — |

## Consequences

**Positive**

- Furniture survives reload, appears in version history, and can be undone like
  any other edit.
- Tenancy, audit and provenance come free.
- AI furnishing uses the **same** command path as manual placement (ADR-013) —
  no second write path to secure.
- Catalogue price changes do not silently re-price issued estimates.
- A construction estimate can exclude FF&E cleanly.

**Negative / accepted cost**

- Every place that iterates elements must respect the class (QS, topology, IFC
  export). A missed check means furniture in a wall quantity.
- Catalogue versioning is extra machinery.
- 3D asset loading, caching and disposal is real browser engineering.

**Neutral**

- IFC export may map FF&E to `IfcFurnishingElement` where round-tripping matters;
  otherwise FF&E may be excluded from a structural export.

## Migration / implementation impact

No migration — decided before any element exists.

Affects: `05_Building_Model_Domain.md`, `07_3D_Engine_Implementation.md`,
`15_Database_and_Data_Architecture.md`, QS/BOQ sectioning, the new
`11_3D_Walkthrough_Interior_and_FFE.md`.

Owned by: Stage 4 of the 3D roadmap (furnishing), after the deterministic
material/finish editor. **Not MVP** — but the schema must accommodate it now,
because retrofitting element classes after projects exist means reclassifying
every row and re-auditing every quantity query.

## Rollback / exit strategy

Removing FF&E later is straightforward (drop the class and its instances).
Adding it later is the expensive direction — which is why the class distinction
is decided now even though the feature is not built yet.

## Verification

- Test: an FF&E element is **excluded** from wall net area, floor area and every
  construction quantity.
- Test: an FF&E element does **not** participate in room boundary derivation.
- Test: placing, moving and deleting furniture appears in the model version
  history and can be undone.
- Test: a catalogue asset version change does not alter an existing instance's
  stored geometry or an issued estimate.
- Test: FF&E cost appears in its own section and can be excluded from a
  construction-only estimate.
- Test: cross-tenant FF&E access returns nothing.
- Review: any furniture state held only in a Three.js scene is a BLOCKER.
- Review: any GLB binary in a relational column is a BLOCKER.

## References

- ADR-001 (canonical model), ADR-002 (commands/versioning), ADR-006 (rooms),
  ADR-007 (type vs assembly), ADR-013 (AI safety), ADR-015 (cost lineage),
  ADR-020 (finish assignment)
- `docs/architecture/11_3D_Walkthrough_Interior_and_FFE.md`
