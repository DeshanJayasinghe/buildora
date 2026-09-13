# 06 — 2D CAD Editor Implementation

> **Status:** FROZEN — 2026-09-12
> **Owns:** 2D editor architecture, PixiJS boundaries, interaction model.
> **Governed by:** ADR-006, ADR-008, ADR-011, ADR-017
> **Implemented in:** `packages/cad-2d`

---

# 1. Architecture

```text
React shell                    (apps/web)
  ├── Toolbar
  ├── Properties panel
  ├── Level selector
  └── Status / feedback
          ↓  (commands in, notifications out)
Editor controller              (packages/cad-2d)
          ↓
packages/model-session         (working model — ADR-008)
          ↓
PixiJS renderer                (packages/cad-2d)
          ↓
Derived render objects         (disposable views)
```

**React controls chrome. PixiJS controls the canvas.** React must never render
thousands of CAD elements; high-frequency interaction must never re-render the
application tree.

---

# 2. Source of Truth

`packages/model-session` holds the working model. The Pixi scene is a **derived,
disposable view** of it.

```text
✗ Pixi display objects as domain state
✗ selection or snapping state as hidden domain state
✗ editor state duplicating the model
✓ Pixi objects carrying domain element IDs
```

Domain element IDs are preserved on render objects. Renderers never generate
replacement identities (`AGENTS.md` §12).

---

# 3. Coordinate Transform

Canvas pixels are **not** construction coordinates.

```text
model millimetres  ←→  screen pixels
```

Maintain one explicit, testable transform. Every hit-test, snap and dimension
resolves through it. Never let a pixel value reach the domain.

---

# 4. Mutation Path

```text
user gesture
  → draft interaction          local preview only, NO command
  → gesture completes
  → semantic domain command
  → model-session (optimistic apply)
  → subscribers notified       2D and 3D update from ONE source
  → API with baseVersion
  → server commits
  → reconciliation             ack | MODEL_VERSION_CONFLICT
```

**One completed gesture → one command → one model version.** Never one command
per pointer move (ADR-002 §write granularity).

---

# 5. PixiJS Lifecycle

```text
mount
  → initialize renderer (once)
  → subscribe to model-session
  → render model
  → interaction loop
  → unsubscribe
  → destroy renderer, free GPU resources
```

Do **not** recreate the Pixi application on React re-render. Initialize once,
tear down once.

WebGL2 baseline (ADR-017).

---

# 6. Incremental Updates

Adapters consume **change items**, not full rebuilds.

```text
model change → change items → update affected display objects only
```

A full scene rebuild on every edit will drop frames at professional model sizes.

## 6.1 The cascade set

Cascades are explicit per element type — not "whatever the renderer noticed"
(`05_Building_Model_Domain.md` §9). Moving a wall updates:

```text
hosted openings (doors, windows, standalone openings)
BOUNDS-related rooms (boundary recompute unless user_defined)
connected walls at junctions
```

An integration test must assert that after a wall move, the model, 2D and 3D all
agree.

---

# 7. Editor Capabilities

| Capability | Notes |
|---|---|
| Selection | Single, multi, marquee. Maps to domain IDs. |
| Snapping | Endpoint, midpoint, intersection, perpendicular, grid, alignment. **Ephemeral** — never domain state. |
| Dimensions | Derived from model geometry, not drawn independently. |
| Wall drawing | Chained; junctions resolved by the domain. |
| Openings | Placed on a host wall; position validated against host extent. |
| Rooms | Derived per ADR-006; user may name, merge, split, override. |
| Levels | Switch active level; elements are level-scoped. |
| Layers / visibility | Presentation filter only. |
| Undo / redo | Compensating commands via model-session, never a local scene stack. |

## 7.1 Room detection ownership

Room detection lives in **`packages/building-model`**, not here. It is
deterministic domain geometry, testable headlessly without a renderer (ADR-006).

The editor triggers recomputation and renders the result; it does not own the
algorithm.

---

# 8. Performance

## 8.1 Targets

| Model size | Expectation |
|---|---|
| 1,000 elements | Comfortable; no special handling |
| 10,000 elements | Smooth with incremental updates and controlled React scope |
| 50,000 elements | Requires spatial indexing and possibly worker offload |

## 8.2 Worker boundary

Move to a Web Worker **only when measured**: room detection or hit-testing
exceeding ~16 ms at 5,000 elements is the trigger.

## 8.3 Rust/WASM

**Profiling-gated** (ADR-012). Implement geometry in TypeScript first. Do not
prematurely reach for Rust because geometry "feels" like it needs it. Likely
candidates *if* evidence appears: polygon boolean operations, offsets, room
detection, spatial indexing.

## 8.4 React scope

High-frequency interaction must not re-render panels. Keep selection and hover
feedback inside the renderer; push only coarse state (selected element summary)
to React.

---

# 9. Accessibility

Professional editor shortcuts must not make the rest of the UI inaccessible
(`AGENTS.md` §78). Keyboard interaction, focus management, labels and contrast
apply to editor chrome. Canvas interaction should have keyboard equivalents for
core operations where practical.

---

# 10. Boundaries

```text
packages/cad-2d
  MAY import:     pixi.js, packages/model-session,
                  packages/building-model (types), packages/units
  MUST NOT import: react, next, three, @nestjs/*, drizzle-orm,
                   packages/engine-3d
```

Enforced in CI (ADR-011).

---

# 11. Review Checklist

- Model is the source of truth; editor state is ephemeral.
- Snapping has not become hidden domain state.
- Domain IDs preserved on render objects.
- React re-render scope controlled.
- Geometry math testable outside the renderer.
- Commands are semantic, one per completed gesture.
- Pixi resources disposed on unmount.
- Transform explicit; no pixel values in domain calls.

---

# 12. References

- ADR-006 (rooms), ADR-008 (model-session), ADR-011 (boundaries), ADR-017 (WebGL2)
- `05_Building_Model_Domain.md`, `07_3D_Engine_Implementation.md`
- `docs/standards/Buildora_AI_Frontend_Engineering_Standards.md` §34–§35
- `AGENTS.md` §35, §37
