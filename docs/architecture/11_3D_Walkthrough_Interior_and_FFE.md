# 11 — 3D Walkthrough, Materials, Furnishing & Interior Interaction

> **Status:** FROZEN — 2026-09-12
> **Owns:** interactive 3D walkthrough, manual material/finish editing, FF&E
> (furniture, fixtures, equipment), and the optional AI interior layer.
> **Governed by:** ADR-001, ADR-002, ADR-006, ADR-007, ADR-013, ADR-015, **ADR-019**, **ADR-020**
> **Implemented in:** `packages/engine-3d`, `packages/model-session`, `packages/building-model`
> **Status of delivery:** post-MVP. The schema accommodates it now; the features
> are built in the staged order in §16.

---

# 1. The Governing Principle

**Deterministic product first. AI as an optional intelligence layer on top.**

Normal interior work must never require an AI call.

```text
MANUAL
  user gesture
    → deterministic semantic command
    → Canonical Building Model
    → model-session
    → renderer

AI
  user prompt
    → AI Gateway
    → typed ChangeSet
    → deterministic validation
    → preview (with cost impact)
    → user approval
    → THE SAME semantic command
    → Canonical Building Model
    → model-session
    → renderer
```

The two paths converge on **identical domain commands**. There is no AI-specific
write path — which is what makes ADR-013's safety model hold here without
additional machinery.

---

# 2. What Requires AI, and What Does Not

| Capability | AI? | Mechanism |
|---|---|---|
| Orbit, pan, zoom | No | Camera controls |
| Walk through the building | No | First-person camera + collision |
| Select any element | No | Raycast → domain element ID |
| Change wall colour / finish | No | `SET_ELEMENT_FINISH` (ADR-020) |
| Change flooring, roof, joinery finish | No | `SET_ELEMENT_FINISH` |
| Place, move, rotate, delete furniture | No | FF&E commands (ADR-019) |
| Open / close a door | No | Viewport state |
| Day / night lighting | No | Renderer environment |
| 3D measurement | No | Model-space geometry |
| Hide / show / isolate | No | Viewport state |
| Section clipping | No | Renderer clipping planes |
| Saved viewpoints | No | Presentation state |
| Real-time cost impact of a finish change | **No** | Deterministic QS + cost engine |
| *"Make these walls light grey"* | Optional | NL → the same finish command |
| *"Furnish this living room"* | Yes | AI placement ChangeSet |
| *"Make this room Scandinavian"* | Yes | AI style ChangeSet |
| *"Find a cheaper similar floor"* | Yes | AI recommendation + deterministic cost |
| *"Reduce furnishing cost by 20%"* | Yes | AI proposal + deterministic cost |

**A 45-minute session** — walking through, trying six paint colours, changing
flooring twice, placing a sofa, moving three chairs, adjusting lighting, reviewing
cost — consumes **zero LLM tokens**.

---

# 3. Interaction Modes

## 3.1 Orbit

```text
Left drag → orbit    Right drag → pan    Scroll → zoom    Click → select
```

## 3.2 Walkthrough

```text
W/S/A/D or arrows → move       Mouse → look
Shift → faster                 Esc → exit walkthrough
```

Touch devices later: virtual joystick + touch-look.

### Architecture

```text
Canonical Building Model
      ↓
packages/model-session          ← the same working copy 2D reads
      ↓
3D adapters
      ↓
Three.js scene (WebGL2 — ADR-017)
      ↓
Walkthrough controller
      ↓
Camera + collision
```

The walkthrough controller is a **renderer/application concern**. It holds camera
position, movement state and collision volumes — none of which is domain data.

**It must never become a second source of geometry.**

## 3.3 Collision

Users must not walk through walls. Collision is deterministic and derived from the
canonical model — never hand-authored.

```text
Phase 1 → wall collision
Phase 2 → floor and stair traversal
Phase 3 → door open/closed state
Phase 4 → large FF&E obstacles
```

## 3.4 Camera height

Configurable, not scattered constants. Default eye height ≈ **1600–1700 mm**.
Future presets: adult, child, wheelchair, custom.

---

# 4. Selection and Identity

```text
pointer → raycast → Three.js object → domainElementId → canonical element
```

**Every rendered object preserves its Buildora domain ID** (ADR-001). Renderers
never generate replacement identities.

The property inspector reads **canonical domain data**, never inferring business
facts from mesh geometry. It may show identity, type, dimensions, finish,
assembly, quantities, cost references, provenance, revision and BOQ links.

`userData` may carry the element ID and render bookkeeping. **It must never carry
business rules.**

---

# 5. Material and Finish Editing (ADR-020)

## The three identity axes

```text
element_type_id       WHAT IT IS          "casement window type W-02"
assembly_version_id   WHAT IT'S BUILT OF  "brick + cavity + block + board"
finish_assignment     WHAT IT LOOKS LIKE  "warm grey emulsion, 2 coats"
```

A user repaints a wall. The **assembly is untouched** — no new assembly version,
no re-pricing of the build-up. Only a new finish assignment.

## Finish is surface-scoped

A wall has two faces and they are routinely finished differently. Finishes attach
to a **surface**, not the whole element — which is how a feature wall in one room
is expressed without splitting the wall.

```text
finish_assignments
  element_id · surface (INTERIOR_A | INTERIOR_B | EXTERIOR | TOP | BOTTOM | SOFFIT | ALL)
  room_id (nullable) · material_version_id · coverage
  provenance · version lineage
```

> **Open contract — must close before the finish editor ships.** The per-element
> **surface taxonomy**: which named surfaces each element type exposes, and how a
> partial-coverage finish declares its area.

## A material is not a colour

```text
Visual (Three.js reads)              Construction (QS/cost reads)
  base colour                          unit of measure (m², m, nr)
  texture · normal · roughness         coverage rate
  metallic · opacity                   coats / layers
  texture scale                        waste profile
                                       labour constant · rate reference
                                       manufacturer · product code
```

**Neither side invents the other's data.** A material with visual properties but
no unit, coverage or waste cannot be costed — that is a defect, not a shortcut.

## The editing flow

```text
select element surface
  → finish panel
  → choose material
  → SET_ELEMENT_FINISH command (baseVersion checked)
  → new model version
  → model-session notifies adapters
  → Three.js material updates
  → QS recomputes affected surfaces
  → cost delta shown
```

## Cost impact is deterministic

```text
Floor finish: basic laminate → engineered oak
Room floor area (interior faces, ADR-006): 42.5 m²

  42.5 × £25.00 = £1,062.50
  42.5 × £55.00 = £2,337.50
                  ───────────
                  +£1,275.00
```

No AI. A permanent golden test asserts this figure.

**Deduction rules are the QS ruleset's decision** (ADR-009), not the finish
assignment's and not a domain constant.

---

# 6. FF&E — Furniture, Fixtures & Equipment (ADR-019)

## FF&E are domain elements, not scene objects

```text
Level
├── Building Elements    wall · door · window · slab · column · beam · stair · roof
├── Spaces / Rooms       derived-then-persisted (ADR-006)
└── FF&E Instances       furniture · fixture · equipment
```

FF&E instances carry an element **class** of `FFE` and inherit every model rule:
stable UUID, tenancy, semantic commands, `baseVersion`, version lineage, undo,
audit, provenance.

**A sofa held only in a Three.js scene is a release-blocking defect.**

## Catalogue asset vs placed instance

```text
ffe_assets          the product          id · name · category · manufacturer
                                         product_code · nominal dimensions
                                         thumbnail · model_asset_object_id
                                         default finishes · tags · VERSION

ffe_instances       the placement        element id · asset_version_id
                                         project · level · room
                                         position_mm · rotation_deg · scale
                                         dimension + finish overrides · provenance
```

Instances reference the asset **by version**. A catalogue update never silently
changes an existing project's geometry or price.

## What FF&E does NOT do

- **Does not** participate in room topology. Furniture never bounds a room.
- **Does not** affect wall net area, floor area, or any construction quantity.
- **Does not** block a construction-only estimate.

Every element iteration in QS and topology code must honour the class. A missed
check puts a sofa in a wall quantity.

## Assets live in object storage

GLB/GLTF models and textures go to R2 with a database metadata reference. **Never
binaries in relational columns.** Lazy load, cache, dispose explicitly; never load
the whole catalogue into the browser.

**A GLB is not financial truth.** Price comes from the catalogue product record.

## Commands

```text
PLACE_FURNITURE · MOVE_FURNITURE · ROTATE_FURNITURE
REMOVE_FURNITURE · SET_FURNITURE_FINISH
```

Same journal, same undo, same history as moving a wall. **There is no separate
interior undo stack.**

## Deterministic placement intelligence

These need no AI — they are geometry and constraints:

```text
snap to floor · snap against wall · snap to room centre
prevent placement outside a room
prevent intersection with walls
prevent blocking a door swing
maintain configured circulation clearance
align · distribute · duplicate · mirror
```

## FF&E cost is its own section

FF&E is priced as product unit price × count, not quantity × assembly rate. It
forms a **separate BOQ/cost section**, so a construction estimate can be issued
with or without furnishing. Lineage is pinned like everything else (ADR-015).

---

# 7. Viewport State vs Model State

Presentation state. **Never mutates canonical geometry.**

| Capability | Notes |
|---|---|
| Hide / show / isolate / hide level / hide category | Viewport filter |
| Section planes, clipping | Renderer clipping — canonical geometry untouched |
| Door open/closed animation | Viewport unless explicitly persisted |
| Lighting: morning / midday / sunset / night | Environment settings |
| Render quality: performance / balanced / high | Renderer settings |
| Saved views | Presentation state, versioned separately from the model |

A **saved view** may hold camera position and target, projection, visible levels,
hidden categories, section planes and walkthrough position. It is not geometry
authority.

---

# 8. 3D Measurement

Point-to-point, horizontal, vertical, angle, clearance.

Measurements use **canonical model-space geometry in millimetres**, never screen
distances or mesh vertices.

---

# 9. The AI Interior Layer

## Flow

```text
"Furnish this living room in modern Scandinavian style, under £4,000."
      ↓
AI Gateway (logical alias — ADR-010)
      ↓
narrow typed tools read: room geometry · openings · existing furniture
                         catalogue subset · style · budget
      ↓
AI returns a typed FF&E ChangeSet
      ↓
DETERMINISTIC validation:
  schema · authorization · baseVersion · room exists · asset exists
  geometry bounds · collision · clearance · cost impact
      ↓
preview (what changes, and what it costs)
      ↓
user approval
      ↓
baseVersion RE-CHECK (ADR-013)
      ↓
the SAME commands manual placement uses
```

## Example ChangeSet

```json
{
  "operations": [
    {
      "type": "PLACE_FURNITURE",
      "assetVersionId": "019a3f2c-...",
      "roomId": "019a3f2c-...",
      "positionMm": { "x": 2200, "y": 800 },
      "rotationDeg": 90
    }
  ]
}
```

**The LLM proposes placement. Buildora performs geometry, collision, validation,
costing, persistence and rendering.**

## Forbidden

```text
✗ LLM → arbitrary Three.js scene → saved as project truth
✗ LLM → direct write to ffe_instances or finish_assignments
✗ LLM → authoritative quantity or cost arithmetic
```

## AI recommendations, deterministic numbers

> *"Find a cheaper alternative that looks similar to this oak floor."*

```text
Current:      Engineered Oak      £55.00/m²
Alternative:  Oak-effect LVT      £29.00/m²
Difference:   −£1,430.00
```

The **recommendation** may be AI-generated. The **cost difference is always
computed by the cost engine** against the pinned rate book.

## Partial approval

Interior design is the strongest case for per-item approval:

```text
✓ Sofa    ✓ Rug    ✗ Coffee table    ✓ Wall colour    ✗ Floor change
```

`ai_change_items` already supports per-item status. **MVP policy remains
all-or-nothing** (OD-14); this is where it should first be revisited.

---

# 10. Provenance

FF&E and finish changes carry provenance like every other element:

```text
source_kind:  MANUAL | AI | CATALOGUE | IMPORT
source_ref:   AI job id · catalogue id · import id
```

This is what lets a user later ask *"which of this furniture did the AI place?"* —
and is impossible to reconstruct if not captured at creation.

---

# 11. Authorization

Interior capability is distinct from structural edit capability. A client may walk
through a model without being able to change it.

```text
model.read · model.edit
interior.read · interior.edit
materials.read · materials.assign
furniture.read · furniture.place
cost.read
ai.use · ai.modify · ai.approve_changes
```

Cost visibility remains a **distinct permission** (OD-20) — a client may furnish a
room without seeing contractor margin.

---

# 12. Performance

## Incremental updates only

```text
model command → change items → update ONLY affected objects
```

Full scene rebuild is for initialisation and exceptional cases only.

## Asset discipline

Lazy load · cache · LOD when measured · **explicit disposal of geometry,
materials and textures**. Three.js does not free GPU memory automatically, and
removing an object from the scene is not disposal — that distinction is the most
common cause of slow memory growth in a long walkthrough session.

## Realtime is not photoreal

```text
Realtime walkthrough    browser · Three.js · interactive · WebGL2
Photoreal rendering     queued job → image/video artifact (post-MVP)
```

**Photoreal rendering is never a prerequisite for the interactive editor.**

---

# 13. Degradation

When AI providers are unavailable, the user can still:

```text
open projects · view 2D · view 3D · walk through · select elements
change materials · place and move furniture · measure
review quantities · review cost · save · reload · export
```

Only the AI *suggestions* are unavailable. This is the practical expression of the
principle in §1.

---

# 14. Future Channels

VR walkthrough, AR furniture placement, mobile AR and LiDAR capture are
**presentation and input channels over the same Building Model**. They require no
change to the domain.

---

# 15. Architecture Invariants

1. Walkthrough, material editing and manual furnishing **never require AI**.
2. Manual and AI actions use the **same domain commands**.
3. Three.js objects are derived views, never source of truth.
4. FF&E instances are **domain elements with stable identity**, never scene state.
5. FF&E is **excluded** from room topology and construction quantities.
6. Large 3D assets live in object storage; a GLB is not financial truth.
7. Finish assignment, construction assembly and element type are **three distinct axes**.
8. A material carries both visual and construction data; neither side invents the other's.
9. Finish deduction rules come from the **QS ruleset**, not from geometry.
10. QS and cost remain deterministic; AI never computes authoritative numbers.
11. AI interior changes are typed ChangeSets, validated and previewed before commit.
12. `baseVersion` is re-checked at commit.
13. Viewport state never silently mutates canonical geometry.
14. Interior actions use the **single** model version history and undo stack.
15. Renderer resources are explicitly disposed.
16. Ordinary UI interaction consumes **zero LLM tokens**.
17. Buildora remains fully usable when AI providers are down.

---

# 16. Delivery Stages

**None of this is MVP.** The MVP is the deterministic Model-to-Cost core. These
stages follow it — but the schema accommodates them now (ADR-019, ADR-020),
because element classes and finish assignments are expensive to retrofit.

| Stage | Scope | Depends on |
|---|---|---|
| **1 — Core 3D** | Orbit, pan, zoom, select, highlight, hide/show/isolate, property inspector | 3D engine (Sprint 18–20) |
| **2 — Walkthrough** | First-person camera, wall collision, floor navigation, camera height, saved views | Stage 1 |
| **3 — Material & Finish** | Wall/floor/roof/joinery finishes, texture support, material library, **deterministic cost impact** | Stage 1 + QS/cost (Sprint 23–25) + surface taxonomy contract |
| **4 — Furnishing** | FF&E catalogue, drag/drop, move/rotate, snapping, collision, room-aware placement, persistence | Stage 3 + ADR-019 schema |
| **5 — AI Interior** | "Furnish this room", style changes, material recommendations, cost optimisation | Stage 4 + AI Gateway (Sprint 27+) |

**Do not delay the deterministic core to build AI interior generation first.** An
AI that furnishes a room before there is a deterministic way to price and validate
the result produces output nobody can trust.

---

# 17. References

- ADR-001 (canonical model) · ADR-002 (commands/versioning) · ADR-006 (rooms)
- ADR-007 (type vs assembly) · ADR-009 (QS rulesets) · ADR-010 (AI routing)
- ADR-013 (AI ChangeSet safety) · ADR-015 (cost lineage) · ADR-017 (WebGL2)
- **ADR-019 (FF&E as domain elements)** · **ADR-020 (finish assignment)**
- `05_Building_Model_Domain.md` · `07_3D_Engine_Implementation.md`
- `15_Database_and_Data_Architecture.md`
