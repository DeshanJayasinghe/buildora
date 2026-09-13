# ADR-018 — IFC as Interchange, Not Canonical State

> **Status:** Accepted
> **Date:** 2026-09-12
> **Deciders:** Product Owner / CTO
> **Supersedes:** none
> **Superseded by:** none

---

## Context

Professional AEC interoperability runs on IFC. A natural-seeming shortcut is to
adopt IFC as the internal model: one format, free import and export, instant
standards compliance.

Several AEC startups have taken that path and become permanently constrained by
IFC's representation choices — its verbosity, its awkward parametric support,
its performance characteristics under interactive editing, and its
schema-version churn.

## Decision

**IFC is an interchange format. It is never Buildora AI's internal editable
source of truth** (ADR-001).

IFC is used for:

```text
import from external BIM tools
export to external BIM tools
external BIM mapping and coordination
```

### External identity mapping

When importing or exporting, maintain explicit mapping between Buildora AI
element IDs and external IFC GUIDs (`external_element_mapping`). Round-tripping
must preserve external identity so that a model exported, edited elsewhere and
re-imported can be reconciled rather than duplicated.

### The domain is not reshaped around IFC

Where IFC cannot express a Buildora AI concept, or expresses it awkwardly, the
canonical model keeps its own representation and the exporter does the
translation. **Do not degrade the internal model to match IFC limitations.**

Conversely, IFC concepts that map cleanly (storeys ↔ levels, type objects ↔
`element_types`) should map cleanly — ADR-005 and ADR-007 were both designed with
that alignment in mind, which reduces export friction without subordinating the
domain.

### Toolchain and sequencing

| Purpose | Tool | Phase |
|---|---|---|
| Browser IFC display of imported models | web-ifc / That Open, Fragments | Sprint 33 |
| Server-side IFC parse and export | IfcOpenShell (Python, `apps/bim-worker`) | Sprint 33 |
| Advanced B-rep geometry | OpenCascade | Deferred — only if parametric geometry proves insufficient |
| DWG/DXF professional support | ODA SDK | Deferred — only on demonstrated paid demand |

**Fragments is for displaying imported IFC models. It is not the internal scene
format** for Buildora AI's own model, which is generated from the canonical model
by `packages/engine-3d` adapters. These two paths stay separate and explicit.

### Schema version

IFC4 is the primary target. IFC2x3 import tolerance is added if real customer
files require it (implementation-time decision, ADR-012 gating).

## Alternatives considered

| Option | Pros | Cons | Why not chosen |
|---|---|---|---|
| A — IFC as internal model | Free interoperability; standards-native | Internal design permanently constrained; poor interactive editing performance; schema churn touches everything | The trap this ADR exists to avoid |
| B — IFC as interchange only (chosen) | Domain stays optimal for editing and QS; IFC concerns isolated in importers/exporters | Mapping work at both boundaries; some fidelity loss is inevitable | — |
| C — No IFC support | Simplest | Excludes professional BIM users entirely | Removes a core market |

## Consequences

**Positive**

- The canonical model stays optimized for interactive editing, QS and costing.
- IFC schema changes affect only the importer/exporter, not the domain.
- Round-trips preserve external identity.

**Negative / accepted cost**

- Import and export mapping is real, ongoing work.
- Some fidelity loss in both directions is unavoidable and must be surfaced
  honestly to users rather than hidden.
- Imported models carry provenance and confidence, like any non-native source.

**Neutral**

- BIM support is deliberately post-MVP (Sprint 33) — professional credibility,
  not MVP survival.

## Migration / implementation impact

No migration.

Affects: `docs/architecture/08_BIM_IFC_Interoperability.md` (written when Phase 15
begins), `15_Database_and_Data_Architecture.md` §109–§110, `apps/bim-worker`.

Owned by: Phase 15 / Sprint 33.

## Rollback / exit strategy

Reversible in principle, but adopting IFC as the internal model later would mean
rebuilding the domain. Treat as a permanent direction.

## Verification

- Test: import → export round-trip preserves external GUIDs.
- Test: an imported element carries source provenance and confidence.
- Review: any domain change justified by "IFC requires it" is a HIGH finding
  requiring explicit consideration.
- Review: using Fragments as the internal scene format for native models is a
  BLOCKER.

## References

- `AGENTS.md` §38
- `CLAUDE.md` §5.3
- ADR-001, ADR-005, ADR-007, ADR-012
