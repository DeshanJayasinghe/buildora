# 04 — Monorepo and Module Architecture

> **Status:** FROZEN — 2026-09-12
> **Owns:** repository shape, package charters, dependency direction, creation gates.
> **Governed by:** ADR-003, ADR-008, ADR-011, ADR-012

---

# 1. One Repository

Buildora AI is a single monorepo managed with **pnpm workspaces + Turborepo**.

Do not split frontend, backend, AI, BIM, infrastructure, contracts or
documentation into separate repositories without an approved ADR.

**One repository does not mean one deployment.** Each deployable has its own
Dockerfile, environment, scaling and release path.

---

# 2. Repository Shape

```text
buildora-ai/
├── apps/
│   ├── web/              Next.js — MVP
│   ├── api/              NestJS  — MVP
│   ├── ai-worker/        Python/FastAPI — gated (~Sprint 31)
│   ├── bim-worker/       Python/IfcOpenShell — gated (~Sprint 33)
│   (render-worker: NOT CREATED — post-MVP)
│
├── packages/
│   ├── design-system/    MVP
│   ├── api-contracts/    MVP
│   ├── units/            MVP
│   ├── building-model/   MVP
│   ├── model-session/    MVP
│   ├── cad-2d/           MVP
│   ├── engine-3d/        MVP
│   ├── qs-engine/        MVP
│   ├── cost-engine/      MVP
│   (ai-tools: NOT CREATED — gated ~Sprint 27)
│
│   (native/geometry-wasm: NOT CREATED — profiling-gated)
│
├── infrastructure/
└── docs/
```

**`packages/domain` does not exist and must not be created** (ADR-011).
**`packages/core` does not exist.** It may be introduced later only under the
conditions in §5.

---

# 3. Package Charters

Every package has a defensible charter. A package without one does not get created.

| Package | Owns | Explicitly not |
|---|---|---|
| `units` | Millimetre canon, unit conversion, display formatting | Building semantics |
| `building-model` | Elements, geometry, validation, commands, room topology, serialization | Rendering, persistence, frameworks |
| `model-session` | Working model, `baseVersion`, optimistic commands, pending queue, reconciliation, subscriptions | Rendering; being a second database |
| `cad-2d` | 2D rendering and interaction (PixiJS) | 3D; persistence; domain rules |
| `engine-3d` | 3D scene, adapters, disposal (Three.js) | 2D; persistence; domain rules |
| `qs-engine` | Deterministic quantity calculation, ruleset interpretation | Cost; rendering |
| `cost-engine` | Deterministic cost calculation, decimal-safe arithmetic | Quantity rules; rendering |
| `api-contracts` | Shared request/response schemas (Zod) | Domain logic; frameworks |
| `design-system` | Presentation primitives, tokens, Storybook | Any domain dependency |

---

# 4. Dependency Graph

```text
                    packages/units
                           ↓
                 packages/building-model
                           ↓
                  packages/model-session
                           ↓
                    ┌──────┴──────┐
                    ↓             ↓
             packages/cad-2d   packages/engine-3d


                 packages/building-model
                           ↓
                   packages/qs-engine
                           ↓
                  packages/cost-engine


   packages/api-contracts     packages/design-system
   (application boundaries)   (presentation only)


═══════════════════════ APPS ═══════════════════════

   apps/web  may compose:              apps/api  may compose:
     design-system                       api-contracts
     api-contracts                       building-model
     building-model                      qs-engine
     model-session                       cost-engine
     cad-2d                              units
     engine-3d                           infrastructure adapters
     units                                 (Drizzle, Stripe, Clerk, OpenAI)

   apps/ai-worker  ·  apps/bim-worker
     Python. Share no TypeScript code.
     Contract = OpenAPI schema + JSON payloads.
```

## 4.1 Why `model-session` sits where it does

Both renderers depend on `model-session`; **neither renderer depends on the
other** (ADR-008). Placing the working model inside `cad-2d` would have made the
3D engine conceptually depend on the 2D package. A renderer-neutral package
keeps the dependency direction honest and gives both adapters one update source,
which is what satisfies the 2D↔3D synchronization invariant structurally rather
than by convention.

---

# 5. Dependency Rules — To Be Enforced in CI

> **Status: Sprint 01 acceptance criteria, not a present repository fact.**
> The workspace files are currently empty and no CI workflow exists. These checks
> must be made executable in Sprint 01, before any package imports another.

1. **Dependencies point downward only.** No upward edges, no lateral edges, **no cycles**.
2. **Domain packages import no framework.** `units`, `building-model`,
   `model-session`, `qs-engine`, `cost-engine` must not import:
   ```text
   react   next   pixi.js   three
   @nestjs/*   drizzle-orm   openai   stripe   @clerk/*
   ```
3. **`cad-2d` may import PixiJS. `engine-3d` may import Three.js.**
   Neither imports the other. Neither imports React.
4. **`design-system` never imports domain packages.**
5. **`api-contracts` depends only on `units`** (and Zod). It must stay importable
   by both `apps/web` and `apps/api`.
6. **Apps may import packages. Packages never import apps.**
7. **`cost-engine` may import `qs-engine` types**; never the reverse.
8. **Python workers share no TypeScript code.**

Framework and infrastructure dependencies belong at the **app and adapter layer**.

Enforce with dependency-cruiser (or equivalent) plus a forbidden-import check.
A violation fails CI.

## 5.1 When `packages/core` may be created

Only when **all three** hold:

1. multiple real packages need the same genuinely domain-neutral primitive,
2. duplication has actually appeared — not been predicted,
3. a precise charter can be written that excludes building semantics.

Until then, `AGENTS.md` §69 applies: clear duplication twice beats a premature
generic framework. Creating `core` speculatively simply relocates the
`packages/domain` problem.

---

# 6. Creation Gates

A package or app is created when its first real consumer exists (ADR-012).

| Artifact | Gate |
|---|---|
| `apps/web`, `apps/api` | Sprint 01 |
| `design-system`, `api-contracts`, `units` | Sprint 01–03 |
| `building-model`, `model-session` | Sprint 08–12 |
| `cad-2d` | Sprint 12 |
| `engine-3d` | Sprint 18 |
| `qs-engine` | Sprint 23 |
| `cost-engine` | Sprint 25 |
| `apps/ai-worker` | ~Sprint 31 |
| `apps/bim-worker` | ~Sprint 33 |
| `packages/ai-tools` | ~Sprint 27 — merge into `api-contracts` if it stays thin |
| `apps/render-worker` | Post-MVP. Not implemented. |
| `native/geometry-wasm` | Profiling-gated. Not implemented. |

A deferred directory may exist **only if it contains a `README.md` naming its
gate**, and must carry **no build, CI or runtime configuration**. A directory
with neither a consumer nor a gate README must be deleted — an unexplained empty
folder is read by agents as an instruction to fill it. An empty directory costs
nothing; a configured-but-unused deployable costs on every CI run and deploy.

---

# 7. Module Structure Inside `apps/api`

The API is a modular monolith (ADR-003), organized by bounded context:

```text
apps/api/src/modules/
├── organizations/
├── projects/
├── model/            commands, versions, elements
├── materials/
├── qs/
├── boq/
├── cost/
├── ai/
├── documents/
├── billing/
└── admin/
```

Each module owns its repositories. There is **no global database service** that
lets any module read any table.

Repository signatures express tenant scope:

```text
✓ findProjectById(organizationId, projectId)
✗ findById(projectId)
```

Modules that stay internally isolated regardless of deployment shape:
`billing` (financial invariants), `model` (version integrity), `ai`
(cost-control surface).

---

# 8. Code Placement

```text
API transport schema        → packages/api-contracts
core geometry / model       → packages/building-model
working client model        → packages/model-session
2D rendering / editing      → packages/cad-2d
3D rendering                → packages/engine-3d
quantity rules              → packages/qs-engine
cost calculation            → packages/cost-engine
unit conversion             → packages/units
UI primitives               → packages/design-system
application orchestration   → apps/api modules
```

**Do not create `utils/` dumping grounds** (`AGENTS.md` §68). If code has no
obvious home, that usually means the boundary is wrong — raise it rather than
inventing a catch-all.

---

# 9. Tooling

```text
TypeScript / JS  → pnpm
Python           → uv
Rust             → cargo
Task orchestration → Turborepo
```

Commit lockfiles. Do not introduce npm/yarn/pipenv/poetry without an ADR.

---

# 10. References

- ADR-003 (modular monolith), ADR-008 (model-session), ADR-011 (boundaries),
  ADR-012 (gates)
- `AGENTS.md` §7, §11, §67, §68, §69
- `03_System_Architecture.md`, `05_Building_Model_Domain.md`
