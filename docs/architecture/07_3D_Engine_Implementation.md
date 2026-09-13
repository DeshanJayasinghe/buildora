# 07 — 3D Engine Implementation

> **Status:** FROZEN — 2026-09-12
> **Owns:** Three.js engine architecture, adapters, scene lifecycle, disposal.
> **Governed by:** ADR-005, ADR-008, ADR-011, ADR-017, ADR-018, ADR-019, ADR-020
> **Implemented in:** `packages/engine-3d`

---

# 1. Architecture

```text
React workspace                (apps/web)
  ├── View controls
  ├── Inspector
  └── Section / clipping UI
          ↓
3D controller                  (packages/engine-3d)
          ↓
packages/model-session         (working model — ADR-008)
          ↓
Element adapters               model element → scene object
          ↓
Three.js scene                 (WebGL2)
```

The 3D scene is **produced from the canonical model through adapters**. It is
never an independent model.

---

# 2. Direct Three.js, Not React Three Fiber

**Decision: direct Three.js** (ADR-017). Adopting R3F for the core editor
requires a superseding ADR.

The editor needs imperative control over:

- incremental adapter diffing driven by change items,
- explicit geometry and material disposal,
- picking mapped to domain IDs,
- frame-budget management during interaction.

R3F's reconciler works against each of these. R3F remains acceptable for
non-editor, presentational 3D if a case ever arises.

**WebGL2 is the baseline. WebGPU is deferred** (ADR-017).

---

# 3. Adapters

An adapter maps one element type to scene objects:

```text
Wall element    → extruded geometry from centreline + thickness + height
                  with openings subtracted
Door / Window   → frame + panel at host position
Slab            → extruded boundary polygon less voids
Column / Beam   → extruded section along axis / path
Room            → optional floor plane, label anchor
Roof            → pitched surface from plane definition
```

## 3.0 Adapters dispatch on element class (ADR-019)

```text
element_class = BUILDING  → fabric adapters (wall, slab, column, …)
element_class = SPACE     → room floor plane, label anchor
element_class = FFE       → catalogue asset loader (GLB from object storage)
```

FF&E adapters differ fundamentally: they **load a catalogue mesh** rather than
generate geometry from parameters. That path needs lazy loading, caching by
`ffe_asset_version_id`, and explicit disposal — an FF&E instance whose asset is
not yet loaded must render a placeholder, never block the frame.

Finish assignments (ADR-020) drive **material** selection on the resulting scene
object; they never change its geometry.

## 3.1 Vertical resolution (ADR-005)

Adapters resolve absolute position at render time:

```text
absoluteZ(element) = level.elevationMm + (element.baseOffsetMm ?? 0)
```

Absolute z is **never** read from the element — it is always derived. Moving a
level therefore moves its contents with no element writes.

## 3.2 Units

Model geometry is millimetres. The adapter applies one explicit scale to scene
units. Do not scatter conversions through adapter code (`packages/units`).

---

# 4. Incremental Updates

```text
model change → change items → update only affected scene objects
```

Full scene rebuilds on every edit will drop frames. The `model_change_items`
stream exists precisely to make this cheap.

Cascades are explicit per element type (`05_Building_Model_Domain.md` §9) —
moving a wall updates its hosted openings, bounded rooms and junction
neighbours in the scene, not just the wall.

---

# 5. Identity and Selection

```text
scene object  ←→  domain element ID
```

Picking resolves to a **domain ID**, which is the same ID used in 2D, QS, BOQ,
cost and audit. Selection state is shared through `model-session`, so selecting
in 3D highlights in 2D without either renderer knowing about the other.

**Do not hide business rules inside Three.js object metadata** (`AGENTS.md` §36).
`userData` may carry the element ID and render bookkeeping — never domain logic.

---

# 6. Resource Lifecycle

Three.js does not garbage-collect GPU resources. Every adapter must dispose what
it creates:

```text
geometry.dispose()
material.dispose()
texture.dispose()
renderer.dispose()   on unmount
```

Removing an object from the scene graph is **not** disposal. Leaks here are the
most common cause of 3D memory growth during long editing sessions.

---

# 6.1 Walkthrough, Interior Editing and FF&E

Interactive walkthrough, manual material/finish editing, furniture placement and
the optional AI interior layer are specified in
**`11_3D_Walkthrough_Interior_and_FFE.md`** (ADR-019, ADR-020).

Three rules that bind this document:

1. The **walkthrough controller is renderer state** — camera, movement, collision
   volumes. It must never become a second source of geometry.
2. **FF&E instances are domain elements**, not scene objects. A sofa is adapted
   from an element exactly as a wall mesh is (ADR-019).
3. **Finish changes are domain commands**, not renderer mutations. Setting a
   material updates the model, which notifies adapters (ADR-020).

---

# 7. Capabilities

| Capability | Notes |
|---|---|
| Orbit / pan / zoom | Camera state is ephemeral UI state (Zustand), not domain |
| Selection | Maps to domain IDs; shared via model-session |
| Materials | Derived from assemblies; visual only — cost lives in `cost-engine` |
| Section planes / clipping | View state, never geometry mutation |
| Level isolation | Show/hide by level |
| Measurement | Derived from model geometry |

---

# 8. Large Models

| Elements | Expectation |
|---|---|
| 1,000 | Comfortable |
| 10,000 | Smooth with incremental updates |
| 50,000 | Needs instancing, merging, culling |

Ordered performance responses, in this sequence:

1. instancing and geometry merging,
2. level of detail, frustum and occlusion culling,
3. reduce draw calls in the adapter layer,
4. offload geometry preparation to workers.

**Only then** consider a second render path. WebGPU is not a substitute for
adapter efficiency (ADR-017).

---

# 9. Imported BIM Models — Separate Path

```text
Buildora native model  → engine-3d adapters       → Three.js scene
Imported IFC model     → web-ifc / That Open      → Fragments display
                         (Fragments)
```

These two paths stay **separate and explicit**. Fragments is for displaying
imported IFC. **It is not the internal scene format for Buildora's own model**
(ADR-018).

---

# 10. Boundaries

```text
packages/engine-3d
  MAY import:      three, packages/model-session,
                   packages/building-model (types), packages/units
  MUST NOT import: react, next, pixi.js, @nestjs/*, drizzle-orm,
                   packages/cad-2d
```

Enforced in CI (ADR-011).

---

# 11. Review Checklist

- Scene built from adapters, not from a parallel model.
- No business logic in the scene graph.
- Domain ID ↔ object mapping intact.
- Updates incremental, driven by change items.
- Geometry, materials and textures disposed.
- Selection consistent with 2D.
- No duplicate model state.
- Absolute z derived, never stored.
- Performance acceptable at realistic project size.

---

# 12. References

- ADR-005 (datum), ADR-008 (model-session), ADR-011 (boundaries),
  ADR-017 (WebGL2 / direct Three.js), ADR-018 (IFC)
- `05_Building_Model_Domain.md`, `06_2D_CAD_Editor_Implementation.md`
- `docs/standards/Buildora_AI_Frontend_Engineering_Standards.md` §36–§38
- `AGENTS.md` §36, §37
