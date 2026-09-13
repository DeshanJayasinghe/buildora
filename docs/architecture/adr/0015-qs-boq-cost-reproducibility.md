# ADR-015 — QS / BOQ / Cost Reproducibility Lineage

> **Status:** Accepted
> **Date:** 2026-09-12
> **Deciders:** Product Owner / CTO
> **Supersedes:** none
> **Superseded by:** none

---

## Context

Buildora AI's differentiator is not that it produces a cost figure. It is that
the figure can be **defended** — traced back to the elements, rules, rates and
assumptions that produced it, and reproduced identically months later.

Professional quantity surveying requires this. A cost estimate that silently
changes when a rate book is updated is worse than useless; it destroys trust and
makes revision comparison meaningless.

## Decision

### Every derived commercial result pins its inputs

```text
quantity_runs
  → model_version
  → ruleset_version           (ADR-009)
  → assembly_catalog_version
  → calculation_engine_version

boq_versions
  → model_version
  → quantity_run_id

cost_estimates
  → model_version
  → quantity_run_id
  → boq_version_id
  → rate_book_version_id
  → cost_engine_version
  → cost_assumption_set_version
```

A cost estimate is a **pure function** of that tuple. Given the same inputs, the
engine must produce byte-identical outputs.

### Element-level traceability

`quantity_item_sources` links each quantity item to the elements that produced
it, with each element's contribution. A grouped quantity can always be expanded
to show which walls, slabs or rooms contributed what.

Each quantity item carries `calculation_trace_jsonb` — the derivation, not just
the result.

Quantities persist `gross`, `deduction`, `net`, `waste` and `final` as separate
`NUMERIC` columns. Intermediate values are never discarded.

### Historical estimates are never silently recomputed

**A report must never recompute a historical cost using today's rate book and
present it as the same version.** Reproducing an old estimate uses the versions
it pinned. Producing an updated estimate creates a new version with new pins.

This is the single most important rule in the costing design.

### Determinism

Geometry, quantities, BOQ totals and cost arithmetic are computed by
deterministic engines in `packages/qs-engine` and `packages/cost-engine`. No LLM
participates in authoritative calculation (ADR-013).

All monetary values use `NUMERIC` / decimal-safe arithmetic (ADR-014).

### Assemblies separate geometry from cost

Geometry does not equal cost. Assemblies expand elements into materials, labour,
plant and waste. Waste rules are configurable data, not constants in code, and
are versioned like everything else in the lineage.

### Overrides are auditable

User overrides of quantities, BOQ items or rates are recorded as overrides with
their original derived value retained — never as silent replacements of the
derived result.

## Alternatives considered

| Option | Pros | Cons | Why not chosen |
|---|---|---|---|
| A — Recompute on demand from current data | Always "fresh"; less storage | Historical estimates change silently; revision comparison meaningless; professionally indefensible | Destroys the product's core claim |
| B — Snapshot final numbers only | Simple; stable | Cannot explain *why* a number is what it is; no element traceability | Fails professional audit |
| C — Full pinned lineage with traces (chosen) | Reproducible; explainable to element level; revision comparison is exact | More storage; more version plumbing | — |

## Consequences

**Positive**

- Any estimate can be reproduced exactly, at any later date.
- Any quantity can be expanded to its contributing elements and formula.
- Revision cost comparison ("what did this design change cost?") is exact rather
  than approximate — a headline professional feature.
- Rate book updates do not corrupt issued estimates.

**Negative / accepted cost**

- More version tables and foreign keys than a naive design.
- Calculation traces consume storage; large models produce large traces.
- Engine versions must be bumped deliberately when calculation behaviour changes.

**Neutral**

- Rebuildable derived artifacts (GLB, thumbnails) are not part of this lineage;
  they may be regenerated freely.

## Migration / implementation impact

No migration.

Affects: `docs/architecture/15_Database_and_Data_Architecture.md` §58–§72,
`packages/qs-engine`, `packages/cost-engine`, reports.

Owned by: Phase 10–11 / Sprints 23–26.

## Rollback / exit strategy

**Effectively irreversible** — lineage cannot be reconstructed retroactively for
estimates produced without it.

## Verification

- Golden test: net wall area (ADR-009), pinned to ruleset version.
- Golden test: room area from interior faces (ADR-006).
- Test: re-running a historical estimate with its pinned versions reproduces
  byte-identical totals.
- Test: changing the rate book does not alter any previously issued estimate.
- Test: every quantity item expands to its source elements summing to its total.
- Review: any report path that recomputes history with current rates is a BLOCKER.
- Review: any float in authoritative cost arithmetic is a BLOCKER.

## References

- `AGENTS.md` §19, §20, §21, §22
- `CLAUDE.md` §19, §20
- `docs/architecture/15_Database_and_Data_Architecture.md` §58–§72
- ADR-006, ADR-009, ADR-013, ADR-014
