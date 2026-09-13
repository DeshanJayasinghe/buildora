# ADR-011 — Monorepo Package Boundaries and Dependency Direction

> **Status:** Accepted
> **Date:** 2026-09-12
> **Deciders:** Product Owner / CTO
> **Supersedes:** none
> **Superseded by:** none

---

## Context

`packages/domain` appeared in five documents — the root README, the Master
Product Reference, the Codex Implementation Reference, the Backend Standards,
and a Dockerfile `COPY` line — and was **never defined in any of them**.

An undefined package sitting beside well-defined ones is not a boundary. It is
an invitation for every implementation agent to place ambiguous code in
`domain/`, which within a few sprints becomes the `utils/` dumping ground that
`AGENTS.md` §68 forbids.

A first draft of this decision proposed replacing it with `packages/core`. That
was corrected during review: pre-creating a generic "shared primitives" package
before a real shared need exists simply relocates the same problem.

## Decision

### `packages/domain` is deleted

It is removed from every document, diagram and configuration.

### `packages/core` is NOT created now

Concepts stay with their owning module until a genuine, demonstrated need for
sharing exists:

| Concept | Home |
|---|---|
| Units, conversion, millimetre canon | `packages/units` |
| Building semantics, geometry, commands, room topology | `packages/building-model` |
| API transport schemas | `packages/api-contracts` |
| Quantity rules | `packages/qs-engine` |
| Cost calculation | `packages/cost-engine` |
| Working model, reconciliation | `packages/model-session` |
| Billing domain | `apps/api` billing module |

`packages/core` may be introduced **later**, and only when **all** of the
following hold:

1. multiple real packages need the same genuinely domain-neutral primitive,
2. duplication has actually appeared (not been predicted),
3. a precise charter can be written that excludes building semantics.

Until then, `AGENTS.md` §69 applies: clear duplication twice is preferable to a
premature generic framework.

### Dependency graph

```text
packages/units
       ↓
packages/building-model
       ↓
packages/model-session
       ↓
 ┌─────┴─────┐
 ↓           ↓
cad-2d    engine-3d

packages/building-model
       ↓
packages/qs-engine
       ↓
packages/cost-engine

packages/api-contracts   (used at application boundaries)
packages/design-system   (presentation only; no domain dependency)
```

### Rules — enforced in CI

1. **Dependencies point downward only.** No upward or lateral edges. No cycles.
2. **Domain packages import no framework.** `units`, `building-model`,
   `model-session`, `qs-engine`, `cost-engine` must not import:
   `react`, `next`, `pixi.js`, `three`, `@nestjs/*`, `drizzle-orm`, `openai`,
   `stripe`, `@clerk/*`.
3. **`cad-2d` may import PixiJS. `engine-3d` may import Three.js.** Neither imports
   the other. Neither imports React.
4. **`design-system` never imports domain packages.**
5. **`api-contracts` depends only on `units`** (and Zod). It must stay importable
   by both `apps/web` and `apps/api`.
6. **Apps may import packages. Packages never import apps.**
7. **`cost-engine` may import `qs-engine` types**; not the reverse.
8. **Python workers share no TypeScript code.** Their contract is the OpenAPI
   schema plus JSON payloads.

### Composition at the app layer

```text
apps/web   may compose:  design-system, api-contracts,
                         building-model, model-session,
                         cad-2d, engine-3d, units

apps/api   may compose:  api-contracts, building-model,
                         qs-engine, cost-engine, units,
                         infrastructure adapters (Drizzle, Stripe, Clerk, OpenAI)
```

Framework and infrastructure dependencies belong at the app and adapter layer,
never inside domain packages.

### Package creation timing

Built for the MVP: `design-system`, `api-contracts`, `units`, `building-model`,
`model-session`, `cad-2d`, `engine-3d`, `qs-engine`, `cost-engine`.

Not built yet: `packages/ai-tools` (no consumer until Sprint 27; merge into
`api-contracts` if it stays a thin schema layer), `native/geometry-wasm`
(profiling-gated — ADR-012).

## Alternatives considered

| Option | Pros | Cons | Why not chosen |
|---|---|---|---|
| A — Keep `packages/domain` undefined | No change | Becomes a dumping ground; boundary with no meaning | The problem being solved |
| B — Rename to `packages/core` now | Feels tidier | Same dumping ground under a better name, created before a real need | Corrected during review |
| C — Delete; keep concepts with owners; defer `core` (chosen) | No undefined boundaries; packages earn their existence | Some duplication may appear before extraction is justified | — |

## Consequences

**Positive**

- Every package has a defensible charter.
- No catch-all destination for ambiguous code.
- Dependency direction is machine-checkable, so drift is caught by CI rather
  than review.

**Negative / accepted cost**

- Genuinely shared primitives may be duplicated briefly before `core` is justified.
- Requires discipline to resist creating `core` at the first minor duplication.

**Neutral**

- Directories for deferred packages may exist as placeholders with a README
  stating the gate, but must carry no build or runtime configuration.

## Migration / implementation impact

No migration — no code exists. `packages/domain` is removed from documentation
before Sprint 01 creates it.

Affects: root `README.md`, `AGENTS.md` §7, `CLAUDE.md`,
`docs/architecture/04_Monorepo_and_Module_Architecture.md`,
`27_DevOps_Environments_and_Deployment.md` (Dockerfile `COPY` lines),
`docs/standards/Buildora_AI_Codex_Implementation_Reference.md`.

Owned by: Sprint 00 (documentation), Sprint 01 (workspace creation).

## Rollback / exit strategy

Fully reversible. Introducing `packages/core` later is a normal refactor with a
written charter — which is the intended path, not a failure.

## Verification

- CI: dependency-cruiser (or equivalent) enforces rules 1–8 and fails on cycles.
- CI: forbidden-import check on domain packages.
- Review: creating a new package without a written charter is a HIGH finding.
- Grep gate: `packages/domain` must not appear in any document.

## References

- `AGENTS.md` §7, §11, §68, §69
- OD in `docs/architecture/OPEN_ARCHITECTURE_DECISIONS.md`
- ADR-003, ADR-008, ADR-012
