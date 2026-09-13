# ADR-007 — Element Type / Instance Identity

> **Status:** Accepted
> **Date:** 2026-09-12
> **Deciders:** Product Owner / CTO
> **Supersedes:** none
> **Superseded by:** none

---

## Context

The Building Model had no concept of a repeated type instance — one window type
used thirty times, a standard internal door, a repeated structural column.

Without it:

- window and door schedules (a core professional deliverable) have nothing to
  group by,
- type-level edits ("make every W-02 window 1500 mm wide") require touching
  every instance,
- IFC round-trips lose type definitions (`IfcWindowType`, `IfcDoorType`),
- QS cannot aggregate meaningfully by type.

Retrofitting means migrating every element and reworking schedules and QS grouping.

## Decision

Introduce **`element_types`** — tenant-scoped, versioned type definitions — and
an **optional `element_type_id`** on `building_elements`.

```text
element_types
- id
- organization_id
- code                  e.g. "W-02", "D-01"
- category              WINDOW | DOOR | COLUMN | ...
- name
- default_geometry_jsonb    + schema version
- default_properties_jsonb  + schema version
- version
- created_at / updated_at
```

**Instances inherit type defaults and may override individually.** An instance
with no `element_type_id` is a one-off, which is legitimate.

### Type identity is separate from assembly identity

This separation is deliberate and must be preserved:

```text
element_type_id     → WHAT IT IS
                      geometric type, schedule grouping, IFC type,
                      "a 1200×1200 casement window, type W-02"

assembly_version_id → WHAT IT COSTS
                      material/labour/plant expansion, waste rules,
                      "200mm blockwork + render + paint"
```

A window type W-02 and the assembly that prices its construction are different
facts about the element. Conflating them would make it impossible to re-price a
type without redefining it, or to use one assembly across several types.

### Used for

- standard doors and windows,
- repeated element types,
- door/window/element schedules,
- type-level property updates,
- BIM interoperability (IFC type export and import).

## Alternatives considered

| Option | Pros | Cons | Why not chosen |
|---|---|---|---|
| A — No types; every element independent | Simplest model | No schedules; no bulk type edit; poor IFC fidelity | Loses a core professional deliverable |
| B — `element_types` + optional FK (chosen) | Matches BIM convention; small cost now; enables schedules and type edits | One more table and inheritance resolution | — |
| C — Reuse `assemblies` as type identity | One fewer concept | Conflates geometric type with cost expansion; cannot re-price without redefining the type | Wrong boundary |
| D — Types as a JSONB property on elements | No schema change | Not queryable; no referential integrity; no versioning | Defeats the purpose |

## Consequences

**Positive**

- Schedules group by type with referential integrity.
- Type-level edits touch one row and propagate to instances that have not overridden.
- IFC export can emit proper type definitions; IFC import can map them back.
- QS may aggregate by type where the ruleset calls for it.

**Negative / accepted cost**

- Property resolution has two layers (type default, instance override) — must be
  explicit and testable.
- Type versioning interacts with model versioning; changing a type is a model
  change affecting every non-overriding instance, and must go through a command.

**Neutral**

- Optional. One-off elements carry no type and behave exactly as before.

## Migration / implementation impact

No migration — decided before doors and windows ship.

Affects: `docs/architecture/05_Building_Model_Domain.md`,
`15_Database_and_Data_Architecture.md` §31, QS/BOQ grouping, IFC mapping.

Owned by: Phase 5 / Sprint 10 (doors, windows & openings).

## Rollback / exit strategy

Moderately reversible — dropping types means flattening defaults onto instances,
which is a mechanical migration. Adding types later is the expensive direction,
which is why it is decided now.

## Verification

- Test: instance inherits type defaults; instance override wins over type default.
- Test: editing a type propagates to non-overriding instances and creates one
  model version.
- Test: schedule groups instances by type correctly.
- Review: any code path that treats `element_type_id` and `assembly_version_id`
  as interchangeable is a BLOCKER.

## References

- `docs/architecture/05_Building_Model_Domain.md`
- OD-05 in `docs/architecture/OPEN_ARCHITECTURE_DECISIONS.md`
- ADR-001, ADR-018
