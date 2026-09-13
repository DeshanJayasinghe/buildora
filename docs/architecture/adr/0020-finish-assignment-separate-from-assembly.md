# ADR-020 — Finish Assignment Is Separate From Construction Assembly

> **Status:** Accepted
> **Date:** 2026-09-12
> **Deciders:** Product Owner / CTO
> **Supersedes:** none
> **Superseded by:** none
> **Extends:** ADR-007 (type vs assembly)

---

## Context

A user in the 3D walkthrough selects a wall and changes its colour from white to
warm grey. That must be instant, must not require AI, and must not rebuild the
wall.

But a wall's paint is also a **construction finish** with a measurable area and a
real cost. The wall build-up is:

```text
Construction assembly        102.5mm brick · 100mm cavity · 100mm block · 12.5mm plasterboard
Interior finish              skim + primer + 2 coats warm grey emulsion
```

These are different facts that change independently and at different rates. A
client repaints a room; the wall construction is untouched. A specification
changes from blockwork to timber frame; the paint colour is unaffected.

ADR-007 already separated *what a thing is* from *what it costs to build*. The
question this ADR settles is whether **visual finish** is a third axis, or a
property of the assembly.

Treating finish as merely `{ "color": "#ffffff" }` on the renderer would be the
easy path — and would make every paint change invisible to QS, so finishes would
silently vanish from cost estimates. Treating finish as part of the construction
assembly would mean a colour change forces a new assembly version, re-pricing the
entire wall build-up.

## Decision

**Finish assignment is a third, independent identity axis.**

```text
element_type_id       WHAT IT IS          "1200×1200 casement window, type W-02"
assembly_version_id   WHAT IT'S BUILT OF  "102.5 brick + 100 cavity + 100 block + 12.5 board"
finish_assignment     WHAT IT LOOKS LIKE  "warm grey emulsion, 2 coats"
```

### Finish assignments are surface-scoped

A wall has two faces, and they are routinely finished differently. A finish is
therefore assigned to an element **surface**, not to the element as a whole:

```text
finish_assignments
  id · organization_id · project_id
  element_id                    the host element
  surface                       INTERIOR_A | INTERIOR_B | EXTERIOR |
                                TOP | BOTTOM | SOFFIT | ALL
  room_id                       nullable — which room this face serves
  material_version_id           the finish material
  coverage                      FULL | PARTIAL (with area override)
  source_kind · source_ref      provenance (MANUAL | AI | CATALOGUE | IMPORT)
  created_model_version
  deleted_model_version
```

This is what makes "paint the feature wall in this room only" expressible without
splitting the wall element.

### Materials carry both visual and construction data

A material is **not** a colour. One record serves the renderer and the estimator:

```text
Visual (consumed by Three.js)        Construction (consumed by QS/cost)
  base_colour                          unit of measure (m², m, nr)
  texture / normal / roughness         coverage rate (e.g. m² per litre)
  metallic · opacity                   number of coats / layers
  texture_scale                        waste profile
                                       labour constant
                                       rate reference
                                       manufacturer · product code
```

**The renderer reads the visual properties. QS and cost read the construction
properties. Neither invents the other's data.**

### Finish changes propagate deterministically to cost

A finish change is an ordinary semantic command, and its cost consequence is
computed by the deterministic engines — **no AI, no approximation**:

```text
SET_ELEMENT_FINISH
  → model command (baseVersion checked)
  → new model version
  → change items identify affected surfaces
  → QS recomputes finish areas for those surfaces only
  → cost engine reprices against the pinned rate book
  → UI shows the delta
```

Worked example, fully deterministic:

```text
Floor finish: basic laminate → engineered oak
Room floor area (interior faces, ADR-006): 42.5 m²

  42.5 m² × £25.00/m²  =  £1,062.50      before
  42.5 m² × £55.00/m²  =  £2,337.50      after
                           ───────────
                           +£1,275.00
```

### The QS measurement rule still governs deduction

**Finish area is a measured quantity, not a geometric constant.** Whether a small
opening is deducted from a painted wall area is decided by the **QS ruleset**
(ADR-009), not by the finish assignment and not by a domain invariant. This is the
same rule already established for wall net area.

### Changing finish does not version the assembly

Repainting a wall creates a new **finish assignment**, not a new assembly version.
The construction build-up, its rates and its lineage are untouched.

### AI and manual use the identical path

```text
Manual:  select wall → colour picker  ┐
                                       ├→ SET_ELEMENT_FINISH → same command path
AI:      "make this room warmer"      ┘   (AI proposes; ADR-013 governs commit)
```

There is no AI-specific finish mechanism to secure or test separately.

## Alternatives considered

| Option | Pros | Cons | Why not chosen |
|---|---|---|---|
| A — Finish as a renderer-only colour | Instant; trivial | Invisible to QS, so finishes silently drop out of estimates; lost on reload; a second store of design state | Breaks the product's core claim |
| B — Finish as part of the construction assembly | One concept | Every colour change forces a new assembly version and re-prices the whole build-up; cannot paint two faces differently | Wrong granularity |
| C — Finish as a plain element property | Simple schema | Cannot express per-surface or per-room finishes without splitting elements | Feature walls are ordinary, not exotic |
| D — Surface-scoped finish assignment (chosen) | Per-face and per-room finishes; independent versioning; deterministic cost; one command path for manual and AI | A third concept to hold; surface taxonomy must be defined per element type | — |

## Consequences

**Positive**

- A colour change is instant visually **and** correctly priced.
- Feature walls, per-room finishes and differing internal/external treatment are
  expressible without splitting geometry.
- Assembly and finish version independently, so neither churns the other.
- Finish history is auditable — "who changed this and when" is answerable.
- AI interior styling needs no new write path.

**Negative / accepted cost**

- A third identity axis is genuine conceptual load; the three must not blur.
- The surface taxonomy (which faces a wall, slab or roof exposes) must be defined
  per element type before the finish editor ships.
- Finish area measurement interacts with room topology (ADR-006) and the QS
  ruleset (ADR-009) — a change in either affects finish quantities.

**Neutral**

- FF&E finish overrides (a sofa in a different fabric) use the same mechanism
  against the FF&E instance (ADR-019).

## Migration / implementation impact

No migration — decided before any finish exists.

Affects: `05_Building_Model_Domain.md`, `15_Database_and_Data_Architecture.md`
(materials §51–§57), QS finish rules, `07_3D_Engine_Implementation.md`,
`11_3D_Walkthrough_Interior_and_FFE.md`.

Owned by: Stage 3 of the 3D roadmap (material/finish editor), and the QS finishes
work at Sprint 23.

**Open contract (must close before the finish editor ships):** the per-element
surface taxonomy — which named surfaces each element type exposes, and how a
partial-coverage finish declares its area.

## Rollback / exit strategy

Collapsing finish into assembly later would require re-versioning every assembly
that carries a finish and would lose per-surface granularity. Treat as effectively
irreversible once finishes are priced in issued estimates.

## Verification

- Golden test: 42.5 m² floor, laminate → engineered oak, produces exactly
  **+£1,275.00** against the pinned rate book.
- Test: two faces of one wall carry different finishes without splitting the wall.
- Test: changing a finish creates a new model version and does **not** create a
  new assembly version.
- Test: finish deduction for small openings follows the **ruleset**, not a
  hard-coded rule.
- Test: an AI-proposed finish change commits through `SET_ELEMENT_FINISH`, not a
  bespoke path.
- Test: a finish change reprices only the affected surfaces, not the whole project.
- Review: any finish stored only as a renderer colour is a BLOCKER.
- Review: any material record carrying visual data but no construction data (unit,
  coverage, waste) is a HIGH finding — it cannot be costed.

## References

- ADR-006 (room topology — finish areas), ADR-007 (type vs assembly),
  ADR-009 (measurement rulesets), ADR-013 (AI safety), ADR-015 (cost lineage),
  ADR-019 (FF&E)
- `docs/architecture/11_3D_Walkthrough_Interior_and_FFE.md`
