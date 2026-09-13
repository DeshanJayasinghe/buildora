# Buildora AI

> **AI Building Design, BIM, Quantity Surveying, Costing & Construction Intelligence Platform**

Buildora AI turns an idea, a prompt, or an existing drawing into a **buildable,
measurable and costed digital building**.

```text
Idea / Prompt / Existing Drawing
        ↓
Canonical Building Model
        ↓
2D  →  3D / BIM
        ↓
Quantities → BOQ → Cost
        ↓
Optimisation → Construction Intelligence
```

The differentiator is not floor-plan generation. It is that a design produces
**auditable, reproducible quantities and cost** — every figure traceable to the
elements, rules, rates and assumptions that produced it, and reproducible months
later from the versions it pinned.

---

## Status

**Architecture frozen 2026-09-12. Implementation has not started.**

The repository currently contains documentation and empty workspace scaffolding.
Sprint 00 (Architecture Lock) is complete; Sprint 01 (Repository Bootstrap) is next.

---

## Documentation

**Start at [`docs/README.md`](docs/README.md)** — the documentation router. It
lists only documents that exist.

| Read this | For |
|---|---|
| [`docs/architecture/FINAL_SYSTEM_ARCHITECTURE.md`](docs/architecture/FINAL_SYSTEM_ARCHITECTURE.md) | The frozen architecture baseline |
| [`docs/architecture/34_ADR_Index.md`](docs/architecture/34_ADR_Index.md) | The 18 accepted architecture decisions |
| [`docs/architecture/05_Building_Model_Domain.md`](docs/architecture/05_Building_Model_Domain.md) | The canonical Building Model |
| [`AGENTS.md`](AGENTS.md) | Mandatory rules for all coding agents |
| [`CLAUDE.md`](CLAUDE.md) | Claude-specific architecture/review guidance |
| [`docs/setup/00_README_START_HERE.md`](docs/setup/00_README_START_HERE.md) | Machine and tooling setup |

---

## Architecture in one paragraph

A **canonical Building Model** in PostgreSQL is the single source of truth. 2D
(PixiJS), 3D (Three.js), IFC, quantities, BOQ, cost and AI context are all
**derived representations** with no independent authority. All model changes flow
through **semantic commands** with `baseVersion` optimistic concurrency, recorded
in an append-only journal with change items and periodic snapshots. A
renderer-neutral **`packages/model-session`** holds the client working copy and
feeds both renderers from one source. **AI proposes ChangeSets; deterministic
domain code commits them.** Quantities and cost are computed by deterministic
engines and pin their full lineage so any estimate is reproducible. The backend is
a **NestJS modular monolith**; separate processes exist only where the runtime
genuinely differs (Python for CV and IFC).

---

## Stack

```text
Frontend   Next.js · React · TypeScript · Tailwind · Radix · TanStack Query · Zustand
2D         PixiJS + custom CAD layer
3D         Three.js (direct, WebGL2)
Backend    NestJS modular monolith · Drizzle
Data       PostgreSQL (+ pgvector, PostGIS when gated) · Redis · Cloudflare R2
AI         TypeScript AI Gateway · provider adapters · Python/FastAPI workers
BIM        web-ifc · That Open · IfcOpenShell
Platform   Docker · AWS ECS/Fargate · Cloudflare · Temporal · Sentry · OpenTelemetry
```

Services are provisioned at their activation gate, not up front (ADR-012).

---

## Repository layout

```text
apps/         web · api · ai-worker · bim-worker
packages/     design-system · api-contracts · units · building-model
              model-session · cad-2d · engine-3d · qs-engine · cost-engine
native/       geometry-wasm (profiling-gated, not implemented)
infrastructure/
docs/
```

Dependency direction is enforced in CI:

```text
units → building-model → model-session → { cad-2d, engine-3d }
        building-model → qs-engine → cost-engine
```

Domain packages import no framework. See
[`docs/architecture/04_Monorepo_and_Module_Architecture.md`](docs/architecture/04_Monorepo_and_Module_Architecture.md).

---

## For contributors and agents

Read [`AGENTS.md`](AGENTS.md) before making any change.

The architecture is **frozen**. Decisions recorded in ADR-001 … ADR-018 may not be
changed without a new approved ADR that explicitly supersedes them. If a decision
looks wrong, stop and propose an ADR — do not encode a different decision in code.
