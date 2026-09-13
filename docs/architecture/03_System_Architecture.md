# 03 — System Architecture

> **Status:** FROZEN — 2026-09-12
> **Owns:** runtime architecture, component responsibilities, request and data flow.
> **Governed by:** ADR-001, ADR-003, ADR-008, ADR-012
> **Routes to:** `27_DevOps_Environments_and_Deployment.md` for environments,
> CI/CD and cloud topology detail.

---

# 1. System Context

```text
                    ┌──────────────────┐
                    │      Users       │
                    │ homeowner · architect · QS
                    │ estimator · contractor
                    └────────┬─────────┘
                             │ HTTPS
                             ▼
              ┌──────────────────────────────┐
              │        Buildora AI           │
              │                              │
              │  design → model → quantities │
              │  → BOQ → cost → intelligence │
              └──┬────────┬────────┬─────────┘
                 │        │        │
      ┌──────────┘        │        └──────────┐
      ▼                   ▼                   ▼
┌───────────┐      ┌────────────┐      ┌────────────┐
│  OpenAI   │      │   Stripe   │      │   Clerk    │
│ AI infer. │      │  payments  │      │  identity  │
└───────────┘      └────────────┘      └────────────┘
```

External systems are **adapters at the edge**. Buildora AI owns authorization,
entitlements, credits and all domain state regardless of provider.

---

# 2. Runtime Architecture

```text
                          ┌─────────────────────────┐
                          │        BROWSER          │
                          │                         │
                          │  React / Next.js shell  │
                          │  ┌───────────────────┐  │
                          │  │ model-session     │  │
                          │  │ in-memory working │  │
                          │  │ model + baseVersion│ │
                          │  │ + pending queue   │  │
                          │  └─────┬───────┬─────┘  │
                          │        │       │        │
                          │   2D adapter  3D adapter│
                          │   (PixiJS)    (Three.js)│
                          └────────────┬────────────┘
                                       │ HTTPS / WSS
                                       ▼
                          ┌─────────────────────────┐
                          │      CLOUDFLARE         │
                          │  DNS / WAF / CDN / TLS  │
                          └────────────┬────────────┘
                                       ▼
                          ┌─────────────────────────┐
                          │  Next.js (apps/web)     │
                          │  Server Components      │
                          │  shell · auth · chrome  │
                          └────────────┬────────────┘
                                       ▼
      ┌────────────────────────────────────────────────────────────┐
      │           NestJS Modular Monolith (apps/api)               │
      │                                                            │
      │  Controllers (thin) → Use Cases → Domain → Repositories    │
      │                                                            │
      │  ┌──────────┐ ┌──────────┐ ┌────────┐ ┌──────┐ ┌────────┐  │
      │  │ projects │ │  model   │ │  qs /  │ │  ai  │ │billing │  │
      │  │ orgs·auth│ │ commands │ │ boq /  │ │gateway│ │ledger │  │
      │  │          │ │ versions │ │  cost  │ │      │ │        │  │
      │  └──────────┘ └──────────┘ └────────┘ └──────┘ └────────┘  │
      └───────┬─────────────────┬──────────────┬──────────┬────────┘
              │                 │              │          │
              ▼                 ▼              ▼          ▼
      ┌──────────────┐  ┌──────────────┐  ┌─────────┐ ┌──────────┐
      │  PostgreSQL  │  │ Cloudflare R2│  │ Temporal│ │ External │
      │  AUTHORITATIVE│ │              │  │ (gated) │ │ OpenAI   │
      │              │  │ ▸ uploads    │  └────┬────┘ │ Stripe   │
      │ ▸ elements   │  │ ▸ snapshots  │       │      │ Clerk    │
      │ ▸ versions   │  │ ▸ GLB / IFC  │       ▼      └──────────┘
      │ ▸ commands   │  │ ▸ reports    │  ┌─────────────────────┐
      │ ▸ change     │  └──────────────┘  │   WORKERS (gated)   │
      │   items      │                    │ ai-worker  Python   │
      │ ▸ quantities │  ┌──────────────┐  │  OpenCV / ONNX      │
      │ ▸ boq / cost │  │Redis (gated) │  │                     │
      │ ▸ ledger     │  │ rate limits  │  │ bim-worker Python   │
      │ ▸ pgvector   │  │ cache        │  │  IfcOpenShell       │
      │   (gated)    │  │ presence     │  └─────────────────────┘
      └──────────────┘  └──────────────┘

      AUTHORITATIVE           DERIVED / CACHE        EXTERNAL
```

**"(gated)"** means provisioned at its activation gate, not up front (ADR-012).

---

# 3. The Governing Invariant

**Every write to canonical element state passes through the model commands
module.** There is no other path.

```text
2D · 3D · IFC · QS · BOQ · cost · AI
        are all DOWNSTREAM of it
```

No renderer, adapter, worker or AI component writes domain geometry directly
(ADR-001, ADR-013).

---

# 4. Component Responsibilities

| Component | Owns | Must not |
|---|---|---|
| `apps/web` | Shell, routing, auth UI, Server Components, editor hosting | Hold authoritative domain state |
| `packages/model-session` | Working model, `baseVersion`, optimistic commands, reconciliation | Import any framework; be a second database |
| `packages/cad-2d` | 2D rendering and interaction (PixiJS) | Import `engine-3d`; write to the database |
| `packages/engine-3d` | 3D rendering and adapters (Three.js) | Import `cad-2d`; hold business rules in scene metadata |
| `apps/api` | Authorization, use cases, domain, persistence, external adapters | Contain business logic in controllers |
| `packages/building-model` | Domain model, geometry, commands, room topology | Import any framework |
| `packages/qs-engine` / `cost-engine` | Deterministic calculation | Depend on an LLM for arithmetic |
| PostgreSQL | **Authoritative state** | Store meshes or large binaries |
| Redis | Cache, rate limits, presence | Hold irreplaceable state |
| R2 | Blobs and derived artifacts | Replace domain metadata |
| Temporal | Durable multi-step workflows | Replace ordinary CRUD |
| Workers | CV, ML, IFC parsing | Write domain state directly |

---

# 5. Primary Flows

## 5.1 Editing a wall

```text
user drags a wall handle
  → cad-2d renders a local preview      (no command)
  → gesture ends
  → MOVE_WALL command applied optimistically to model-session
  → model-session notifies subscribers  → 2D and 3D update from ONE source
  → command POSTed with baseVersion
  → API: authorize → validate → transaction:
        verify current version
        apply command + cascade (hosted openings, bounded rooms, junctions)
        increment version
        write model_commands + model_change_items
  → response: new version + change items
  → model-session reconciles
        ack      → confirm, advance baseVersion
        conflict → MODEL_VERSION_CONFLICT → reload/replay or surface to user
```

## 5.1a Interior editing (no AI)

```text
select wall surface → choose finish
  → SET_ELEMENT_FINISH command (baseVersion checked)
  → new model version
  → model-session notifies BOTH adapters
  → 3D material updates · QS recomputes affected surfaces · cost delta shown
```

```text
drag furniture from catalogue
  → PLACE_FURNITURE command
  → FF&E element created (element_class = FFE)
  → 3D adapter lazy-loads the GLB from object storage by asset version
```

**Zero LLM tokens.** Both paths are ordinary domain commands (ADR-019, ADR-020).

---

## 5.2 Quantities to cost

```text
model version N
  → quantity run   (pins model_version, ruleset_version,
                    assembly_catalog_version, calculation_engine_version)
  → BOQ version    (pins model_version, quantity_run)
  → cost estimate  (pins the above + rate_book_version,
                    cost_engine_version, assumption_set_version)
  → report
```

Deterministic throughout. Reproducible from the pinned tuple, forever (ADR-015).

## 5.3 AI change proposal

```text
user prompt
  → AI Gateway (logical alias: FAST | BALANCED | ADVANCED | EXPERT)
  → typed, org/project-scoped tools
  → structured ChangeSet         (AI writes ONLY change sets)
  → draft / sandbox
  → geometry validation
  → QS + cost impact
  → preview
  → user approval
  → baseVersion RE-CHECK
  → domain commands commit
  → new model version
```

AI proposes. The domain validates. Deterministic code commits (ADR-013).

## 5.4 Long-running work

```text
request → create job → return 202 + job id
        → durable workflow (Temporal, once gated in)
        → status updates → client polls or subscribes
```

Long work never sits inside an HTTP request. Before Temporal's gate, a
`processing_jobs` table plus an in-process worker satisfies the same interface
(ADR-012).

---

# 6. Trust and Failure Boundaries

## 6.1 Trust

```text
UNTRUSTED: browser input, uploaded files, document contents,
           LLM output, external webhooks
TRUSTED:   server-side domain validation, deterministic engines,
           PostgreSQL constraints
```

Uploaded documents are **data, never instructions** (ADR-013). Webhooks are
signature-verified and processed idempotently through an inbox (ADR-014).

## 6.2 Degradation

| Outage | Behaviour |
|---|---|
| OpenAI | AI actions fail or queue; reservations release; **deterministic design, QS and cost remain fully available** |
| Stripe | Existing entitlements honoured; new purchases deferred; reconcile later |
| Redis | Degraded rate limiting and presence; **never reconstruct project state from Redis** |
| R2 | Uploads and artifact generation fail; model editing continues |
| Temporal | Long workflows pause and resume; synchronous work unaffected |
| PostgreSQL | Hard outage — the one dependency with no graceful degradation |

Liveness must not fail because an external provider is down. Readiness may.

---

# 7. Scale Path

No fundamental rewrite is required across the modelled range.

| Stage | Change |
|---|---|
| 10 → 1,000 users | Nothing. Single API task, single database. |
| 1,000 → 10,000 | Horizontal API tasks; connection pooling; AI budget controls. |
| 10,000 → 100,000 | Read replica; partition `audit_events`, `model_commands`, `ai_provider_calls` by time; consider extracting a worker-heavy module. |

The seams that make this work — modular monolith with bounded contexts, stateless
API tasks, derived artifacts already in object storage, time-partitionable
journals — are present from the start by design, not added later.

**Not part of the planned architecture:** Kubernetes, Kafka, service mesh,
sharding, multi-region, per-tenant databases (ADR-012).

---

# 8. References

- ADR-001, ADR-003, ADR-008, ADR-012, ADR-013, ADR-015
- `04_Monorepo_and_Module_Architecture.md` — package boundaries
- `05_Building_Model_Domain.md` — the domain itself
- `27_DevOps_Environments_and_Deployment.md` — environments and cloud detail
- `15_Database_and_Data_Architecture.md` — persistence detail
