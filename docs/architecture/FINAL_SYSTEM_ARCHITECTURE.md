# Buildora AI — Final System Architecture

> **Status:** **FROZEN** — 2026-09-12
> **Audience:** every implementation agent and contributor
> **Authority:** this is the concise architecture baseline. Detailed subjects live
> in their dedicated documents; this document routes to them and states what is locked.
> **Router:** `docs/README.md` · **Decisions:** `docs/architecture/34_ADR_Index.md`

---

# 1. Product and System Purpose

Buildora AI is an **AI Building Design, BIM, Quantity Surveying, Costing &
Construction Intelligence Platform**.

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
elements, rules, rates and assumptions that produced it.

Detail: `01_Master_Product_Reference.md`

---

# 2. System Context

```text
                    ┌──────────────────┐
                    │      Users       │
                    │ homeowner · architect · QS
                    │ estimator · contractor
                    └────────┬─────────┘
                             ▼
              ┌──────────────────────────────┐
              │        Buildora AI           │
              └──┬────────┬────────┬─────────┘
                 ▼        ▼        ▼
           ┌─────────┐ ┌────────┐ ┌────────┐
           │ OpenAI  │ │ Stripe │ │ Clerk  │
           │ (AI)    │ │(payment)│ │(identity)│
           └─────────┘ └────────┘ └────────┘
```

External providers are **adapters at the edge**. Buildora AI owns authorization,
entitlements, credits and all domain state regardless of provider.

---

# 3. Runtime Architecture

```text
                    ┌─────────────────────────┐
                    │        BROWSER          │
                    │  React / Next.js shell  │
                    │  ┌───────────────────┐  │
                    │  │  model-session    │  │  working model
                    │  │  + baseVersion    │  │  + pending queue
                    │  └─────┬───────┬─────┘  │
                    │   2D adapter  3D adapter│
                    │   (PixiJS)    (Three.js)│
                    └────────────┬────────────┘
                                 ▼
                         CLOUDFLARE (DNS/WAF/CDN/TLS)
                                 ▼
                      Next.js  apps/web  (Server Components)
                                 ▼
      ┌──────────────────────────────────────────────────────┐
      │        NestJS Modular Monolith — apps/api            │
      │  Controller → Use Case → Domain → Repository         │
      │  projects·orgs │ model │ qs·boq·cost │ ai │ billing  │
      └───┬──────────┬──────────┬──────────┬─────────────────┘
          ▼          ▼          ▼          ▼
    ┌──────────┐ ┌────────┐ ┌─────────┐ ┌──────────┐
    │PostgreSQL│ │   R2   │ │Temporal │ │ External │
    │AUTHORITA-│ │ blobs  │ │ (gated) │ │ OpenAI   │
    │  TIVE    │ │artifacts│ └────┬────┘ │ Stripe   │
    └──────────┘ └────────┘      ▼      │ Clerk    │
    ┌──────────┐           ┌───────────┐└──────────┘
    │  Redis   │           │  WORKERS  │
    │ (gated)  │           │ ai · bim  │
    └──────────┘           │  (gated)  │
                           └───────────┘
```

**"(gated)"** = provisioned at its activation gate, not up front (ADR-012).

Detail: `03_System_Architecture.md`, `27_DevOps_Environments_and_Deployment.md`

---

# 4. Monorepo and Module Architecture

```text
buildora-ai/
├── apps/
│   ├── web/              MVP        Next.js
│   ├── api/              MVP        NestJS modular monolith
│   ├── ai-worker/        gated ~S31 Python/FastAPI
│   ├── bim-worker/       gated ~S33 Python/IfcOpenShell
│   (render-worker: NOT CREATED — post-MVP)
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
│   (ai-tools: NOT CREATED — gated ~S27)
  (native/geometry-wasm: NOT CREATED — profiling-gated)
├── infrastructure/
└── docs/
```

**`packages/domain` does not exist and must not be created.**
**`packages/core` does not exist** and is created only under the three conditions
in `04_Monorepo_and_Module_Architecture.md` §5.1 (ADR-011).

Detail: `04_Monorepo_and_Module_Architecture.md`

---

# 5. Allowed Dependency Graph

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


   packages/api-contracts      packages/design-system
   (application boundaries)    (presentation only)

   apps/web   → design-system, api-contracts, building-model,
                model-session, cad-2d, engine-3d, units
   apps/api   → api-contracts, building-model, qs-engine,
                cost-engine, units, infrastructure adapters
   workers    → Python; no TypeScript packages; OpenAPI + JSON contract
```

## Rules — to be enforced in CI (Sprint 01 acceptance criteria)

1. Dependencies point **downward only**. No lateral edges. **No cycles.**
2. Domain packages (`units`, `building-model`, `model-session`, `qs-engine`,
   `cost-engine`) import **no framework**: not `react`, `next`, `pixi.js`,
   `three`, `@nestjs/*`, `drizzle-orm`, `openai`, `stripe`, `@clerk/*`.
3. `cad-2d` may import PixiJS; `engine-3d` may import Three.js. **Neither imports
   the other.** Neither imports React.
4. `design-system` never imports domain packages.
5. `api-contracts` depends only on `units` (and Zod).
6. Apps import packages. **Packages never import apps.**
7. `cost-engine` may import `qs-engine` types; never the reverse.
8. Python workers share no TypeScript code.

---

# 6. Canonical Building Model Principles

**The Building Model in PostgreSQL is the single source of truth** (ADR-001).

```text
Canonical Building Model
    ├─→ 2D  ├─→ 3D  ├─→ IFC  ├─→ quantities
    ├─→ BOQ ├─→ cost └─→ AI context
```

Derived, never authoritative: the Pixi scene, the Three scene, IFC, meshes, the
model-session, quantities, BOQ, cost.

```text
Project → Site → Building → Level → Element
```

Every element carries a stable UUID used identically across model, 2D, 3D, IFC,
QS, BOQ, cost, AI results and audit. Geometry is **parametric, never mesh**.
JSONB payloads are schema-versioned and runtime-validated.

Detail: `05_Building_Model_Domain.md`

---

# 7. Coordinates, Units and Datum

```text
Canonical unit:        millimetres (packages/units)
Project-local datum:   ground-floor FFL = 0 mm
Level.elevationMm:     signed from project datum — SINGLE authoritative vertical fact
Element z:             level-relative
absoluteZ = level.elevationMm + (baseOffsetMm ?? 0)    — derived, never stored
floorToFloor = next.elevationMm − current.elevationMm  — derived, never stored
```

Slab structural thickness belongs to the **Slab and its assembly**, not to Level.
`Level.defaultStoreyHeightMm` is an optional creation-time design aid only.

Real-world georeference lives on **Site**, separate from building-local
coordinates (ADR-005).

---

# 8. Room, Opening and Type Semantics

## Rooms (ADR-006)

```text
wall centreline graph  →  CONNECTIVITY (which regions are enclosed)
        ↓
resolve junctions → derive INTERIOR WALL FACES
        ↓
usable room boundary  →  persisted with stable ID + BOUNDS relationships
```

The boundary is derived from **interior wall faces**, not centrelines. Area is
**always computed from the boundary**; a user-entered area is never authoritative.
`user_defined = false` → recomputed on wall change (ID, name, finishes preserved).
`user_defined = true` → never silently overwritten.

## Openings

**A door or window IS the opening.** There is no second `Opening` element for the
same hole. Standalone `opening` exists only for fixture-less holes (service
penetration, arch, unfilled opening). **QS sees all openings through one
deduction abstraction.**

## Element types (ADR-007)

```text
element_type_id      → WHAT IT IS    (schedules, IFC type, type-level edits)
assembly_version_id  → WHAT IT COSTS (materials, labour, plant, waste)
```

Never conflated.

---

# 9. Command and Versioning Architecture

```text
Current materialized state  +  model_commands (append-only journal)
+  model_change_items       +  model_versions  +  model_snapshots
```

- Every mutation carries **`baseVersion`**; stale writes return
  **`MODEL_VERSION_CONFLICT`**. Geometry is never silently merged.
- Committed versions are **immutable**.
- One command = one atomic transaction **including its cascade**.
- One completed gesture = one command. Never one per pointer move.
- Snapshots ~every 50 versions (configurable) and before large imports or AI commits.
- Undo is a **compensating command**, never a mutation of history.

Detail: ADR-002, `15_Database_and_Data_Architecture.md` §37–§45

---

# 10. Client Editor / Session Architecture

**`packages/model-session`** — renderer-neutral (ADR-008) — owns:

```text
in-memory working model · baseVersion · optimistic command application
pending command queue · server reconciliation · MODEL_VERSION_CONFLICT handling
subscriptions · cascade propagation
```

It imports **no framework** and is **not a second database**. PostgreSQL remains
authoritative; the session is a synchronized working copy.

| State | Owner |
|---|---|
| Persisted model | PostgreSQL |
| Working model, `baseVersion`, pending commands | `model-session` |
| Project metadata, lists, QS/BOQ/cost read models | TanStack Query |
| Active tool, selection, panels, camera, temporary interaction | Zustand (ephemeral only) |

---

# 11. 2D Architecture

```text
React shell → editor controller → model-session → PixiJS renderer
```

React owns chrome; PixiJS owns the canvas. Pixi objects are **disposable views**
carrying domain IDs. One explicit **mm ↔ px** transform; pixel values never reach
the domain. Snapping is ephemeral, never hidden domain state. Room detection lives
in `building-model`, not the editor.

Detail: `06_2D_CAD_Editor_Implementation.md`

---

# 12. 3D Architecture

```text
React workspace → 3D controller → model-session → adapters → Three.js (WebGL2)
```

**Direct Three.js, not React Three Fiber. WebGL2 baseline; WebGPU deferred**
(ADR-017). Adapters resolve absolute z at render time. Explicit disposal of
geometry, materials and textures. Selection maps to domain IDs. No business logic
in scene metadata.

Imported IFC displays via web-ifc / Fragments on a **separate path** — Fragments
is never the internal scene format (ADR-018).

Detail: `07_3D_Engine_Implementation.md`

---

# 13. 2D ↔ 3D Synchronization

```text
domain command → model-session → subscribers → 2D adapter AND 3D adapter
```

Both renderers consume **one source**, so divergence is structurally prevented.

```text
✗ 2D patches 3D          ✗ 3D writes the database
✗ separate 2D/3D models  ✗ renderer-generated identities
```

Adapters apply **incremental updates from change items**, with an explicit
cascade set per element type (`05_Building_Model_Domain.md` §9).

---

# 14. Database Architecture

```text
PostgreSQL   → authoritative domain state
Redis        → cache, rate limits, presence (never irreplaceable state)
R2 / S3      → blobs, snapshots, derived artifacts
pgvector     → embeddings (gated)
PostGIS      → real-world geography (gated)
Temporal     → durable workflow state (gated)
```

- One database, one application schema.
- **`uuid` primary keys** (UUIDv7 preferred); `public_id` for display — **never an
  FK target** (ADR-004).
- `BIGINT` for monotonic counters. **`NUMERIC` for all money and rates.**
- `TIMESTAMPTZ` everywhere; UTC at API boundaries.
- Hybrid relational + **schema-versioned, validated JSONB**.
- No meshes or large binaries in relational columns.
- Migrations are committed artifacts, run as an explicit pipeline step —
  **never at application startup**. Expand → migrate → switch → contract.

Detail: `15_Database_and_Data_Architecture.md`

---

# 15. QS / BOQ / Cost Lineage

```text
model version
  → quantity run   (pins model_version, ruleset_version,
                    assembly_catalog_version, calculation_engine_version)
  → BOQ version    (pins model_version, quantity_run)
  → cost estimate  (pins the above + rate_book_version,
                    cost_engine_version, assumption_set_version)
  → report
```

A cost estimate is a **pure function** of that tuple. Given the same inputs, the
same output — byte-identical, forever.

- `quantity_item_sources` gives element-level traceability.
- `calculation_trace_jsonb` records the derivation, not just the result.
- `gross / deduction / net / waste / final` persisted separately.
- **Historical estimates are never recomputed with current rates.**
- Measurement rules are **versioned data** (`qs_rule_sets`); **NRM2** is the
  initial default (ADR-009).
- Assemblies expand elements into materials, labour, plant and waste. Waste is
  configurable data.
- Overrides retain the original derived value.

Detail: ADR-015, ADR-009

---

# 16. AI Architecture

```text
prompt → AI Gateway (alias: FAST | BALANCED | ADVANCED | EXPERT)
  → typed, org/project-scoped tools
  → ChangeSet            (AI writes ONLY ai_change_sets / ai_change_items)
  → draft → validation → QS/cost impact → preview
  → user approval → baseVersion RE-CHECK → domain commit
```

- **AI never writes domain state.** Structurally enforced (ADR-013).
- No raw SQL, no arbitrary database access for the LLM.
- Provider model IDs and prices are **versioned configuration** with `verified_at`;
  domain code uses aliases only (ADR-010).
- LLMs are never authoritative for geometry, quantities, BOQ, cost, tax or credit
  arithmetic.
- Uploaded documents are **data, never instructions**; document-derived ChangeSets
  require human approval.
- RAG is tenant/project-filtered with citation lineage.
- Confidence, provenance and verification status are first-class.

---

# 17. Billing Architecture

```text
Ledger, not balance:  GRANT RESERVE RELEASE CONSUME REFUND REVOKE ADJUSTMENT EXPIRE
Reserve → work → settle (CONSUME actual + RELEASE remainder)
```

- **Stripe** authoritative for payment facts; **Buildora AI** authoritative for
  plans, entitlements, credits, usage and feature access.
- Credits granted **only on server-side payment confirmation** — never a browser
  redirect.
- Webhooks: verify → inbox → idempotent processing.
- Customer units (AI Design Credits, Render Credits) are **decoupled from provider
  tokens**. One customer action may span several provider calls; retries and
  escalation never double-charge.
- **All money `NUMERIC` / decimal-safe.** Floats forbidden.

Detail: `24_Billing_AI_Credits_and_Usage.md`, ADR-014

---

# 18. Security and Tenancy

- **Organization is the tenant boundary.** Cross-tenant access is release-blocking.
- Every tenant-owned table carries `organization_id`; every repository method is
  explicitly scoped. Negative cross-tenant tests are mandatory from Sprint 05.
- **RLS**: designed, implemented and verified against the pooled connection model
  from Sprint 05; **enabled and verified before the first external beta** — a hard
  gate (ADR-016).
- Authentication ≠ authorization. Buildora AI owns authorization; Clerk sits
  behind an abstraction.
- Uploads: private by default, presigned to an **exact object key**, short TTL,
  validated type and size, tenant/project authorized, parsed in isolated workers.
- **MVP import is upload-based only — no arbitrary user URL fetching** (SSRF).
- Never log tokens, secrets, complete presigned URLs or document contents.

---

# 19. Deployment and Infrastructure

```text
Cloudflare → AWS ALB → ECS Fargate (web, api) → Neon · R2 · Redis · Temporal
GitHub Actions · ECR · Sentry · OpenTelemetry
```

- Build once; promote the **same immutable digest**. No `latest`.
- Migrations are a separate pipeline step from app boot.
- OIDC to AWS; no long-lived cloud keys.
- Health and readiness separated — liveness must not fail because a provider is down.

## Provisioning gates (ADR-012)

| Service | Gate |
|---|---|
| PostgreSQL (local + Neon), Sentry, CI | Immediately |
| R2 | First upload feature (~S31) |
| Redis | AI rate limiting (~S27) |
| Temporal | First durable workflow (~S31) |
| pgvector | RAG phase (~S34) |
| PostGIS | Site/GIS phase (~S39) |
| AWS production | First external beta (~S35) |
| Rust/WASM | Profiling evidence only |

Sprints 01–26 run on **local Docker + Neon**.

---

# 20. Deferred Architecture

**Not part of the planned architecture** (require a superseding ADR):

```text
Kubernetes · Kafka · service mesh · database sharding
multi-region · per-tenant databases · Go services · microservices
```

**Deferred, gated on demonstrated need:**

```text
realtime collaboration / Yjs      ODA SDK (DWG)
OpenCascade                       GPU render workers
read replicas                     table partitioning
render-worker                     native/geometry-wasm
packages/ai-tools                 Elasticsearch
```

**Mentioning a technology in a roadmap document does not approve it.**

---

# 21. ADR Index

22 accepted ADRs. Full index: `docs/architecture/34_ADR_Index.md`

| ADR | Title |
|---|---|
| 001 | Canonical Building Model as single source of truth |
| 002 | Model persistence: current state + commands + change items + snapshots |
| 003 | NestJS modular monolith with separate language workers |
| 004 | UUID primary keys with separate public identifiers |
| 005 | Building-local datum and level-relative element geometry |
| 006 | Room/Space topology: interior-face boundary, persisted identity |
| 007 | Element type / instance identity |
| 008 | Renderer-neutral model-session architecture |
| 009 | Versioned QS measurement rulesets; NRM2 initial default |
| 010 | Provider models and pricing as versioned configuration |
| 011 | Monorepo package boundaries and dependency direction |
| 012 | Infrastructure provisioning gates |
| 013 | AI ChangeSet safety architecture |
| 014 | Billing ledger and credit reservation architecture |
| 015 | QS / BOQ / cost reproducibility lineage |
| 016 | Tenant isolation and the RLS production gate |
| 017 | WebGL2 graphics baseline; WebGPU deferred |
| 018 | IFC as interchange, not canonical state |
| 019 | FF&E (furniture/fixtures/equipment) as first-class domain elements |
| 020 | Finish assignment separate from construction assembly |
| 021 | Two customer lifecycles on one platform |
| 022 | Runtime feature gating and admin-managed catalogue |

---

# 22. Architecture Invariants

Violating any of these is a **release-blocking defect**.

1. One canonical Building Model. 2D, 3D, IFC, QS, BOQ and cost are derived.
2. All model writes flow through semantic domain commands.
3. `baseVersion` optimistic concurrency; committed versions immutable.
4. Millimetres canonical; level-relative z; one authoritative level elevation.
5. Room boundaries from interior wall faces; area computed, never entered.
6. A door or window **is** its opening; one QS deduction abstraction.
7. Element type ≠ construction assembly ≠ visual finish — **three distinct axes** (ADR-007, ADR-020).
7a. FF&E are domain elements, excluded from room topology and construction quantities (ADR-019).
7b. Manual interior work requires **no AI**; manual and AI paths share the same commands.
8. `model-session` is a working copy, never a second authority.
9. Renderers hold no domain state and generate no identities.
10. `uuid` primary keys; `public_id` never an FK target.
11. `NUMERIC` / decimal-safe money. **No floats.**
12. Organization-scoped every tenant query; RLS before external beta.
13. AI proposes ChangeSets; the domain validates and commits.
14. LLMs never compute authoritative geometry, quantities or money.
15. Uploaded documents are data, never instructions.
16. Credits are a ledger with reservations; grants only on server-side confirmation.
16a. Plans may be **recurring or fixed-term**; an expired term is success, not churn (ADR-021).
16b. **Every user has an organization** — personal, auto-created. No user-owned project path.
16c. Archive preserves project data; capability is entitlement-driven, never deletion.
16d. Feature code reads **entitlements**, never plan names.
16f. **Gated capabilities are enforced server-side.** Hiding a UI control is not
     access control. Check order: flag → entitlement → credit (ADR-022).
16g. **Feature flags ≠ entitlements.** Flags are rollout; entitlements are
     commercial rights. A flag must never grant paid access.
16e. **One set of engines for all segments.** No Home/Professional QS or cost duplication.
17. Webhooks are idempotent.
18. Cost estimates pin their full lineage; history is never recomputed with current rates.
19. Domain packages import no framework; dependencies point downward; no cycles.
20. Managed services before distributed complexity; no service without a consumer.

---

# Architecture Freeze

**Status: ACCEPTED**

Buildora AI's foundational architecture is frozen as of the date of this update.

The following may not be changed by an implementation agent without an explicitly
approved ADR that supersedes the applicable decision:

- canonical Building Model
- persistence/versioning model
- coordinate/datum model
- room/opening/type semantics
- FF&E element class and finish-assignment separation
- editor-session ownership
- renderer boundaries
- database authority
- tenant boundary (including the universal-organization rule)
- deterministic QS/BOQ/cost rules
- AI ChangeSet safety model
- billing ledger architecture
- monorepo/package dependency direction
- core technology stack

Implementation details may evolve within these boundaries.

Performance-related technologies remain evidence-driven.

Deferred technologies do not become approved merely because they are mentioned in
roadmap documents.
