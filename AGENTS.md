# Buildora AI — AGENTS.md

> **Repository:** `buildora-ai`
>
> **Purpose:** Mandatory operating instructions for all coding agents, automation agents, reviewers, and AI-assisted development tools working in this repository.
>
> **Applies to:** Codex, Claude when writing code, Antigravity when making changes, IDE agents, CI repair agents, and any future automated contributor.
>
> **Status:** Root-level agent contract. Read this file before making changes.

---

# 1. Product Identity

**Product:** Buildora AI

**Positioning:**

> AI Building Design, BIM, Quantity Surveying, Costing & Construction Intelligence Platform

Core product flow:

```text
Idea / Prompt / Existing Drawing
        ↓
Canonical Building Model
        ↓
2D Design
        ↓
3D / BIM
        ↓
Quantities
        ↓
BOQ
        ↓
Cost
        ↓
Optimisation
        ↓
Construction Intelligence
```

Buildora AI is not only a floor-plan generator.

It is intended to become a professional AEC platform covering:

- AI-assisted building design,
- editable 2D CAD-like workflows,
- synchronized 3D,
- BIM interoperability,
- quantity takeoff,
- BOQ,
- costing,
- document intelligence,
- project-aware AI,
- reporting,
- later construction/project operations.

---

# 2. Mandatory First Step

Before modifying code, configuration, database schema, infrastructure, billing, AI behavior, or architecture:

1. Read this `AGENTS.md`.
2. Read `docs/README.md` if present.
3. Identify the current project phase and sprint.
4. Read the documentation required for the task.
5. Inspect the existing implementation and tests.
6. State the intended change before introducing a new architectural pattern.

Do not begin implementation by guessing the architecture.

---

# 3. Documentation Is Part of the Architecture

The repository documentation is authoritative.

Expected documentation structure:

```text
docs/
├── README.md
├── setup/
├── architecture/
├── standards/
├── planning/
├── finance/
├── examples/
└── operations/
```

**`docs/README.md` is the documentation router.** It lists only documents that
actually exist. Start there.

Canonical references — **all of these exist**:

```text
ARCHITECTURE — frozen baseline and decisions
docs/architecture/FINAL_SYSTEM_ARCHITECTURE.md      ← read first
docs/architecture/34_ADR_Index.md
docs/architecture/adr/                              ← ADR-001 … ADR-022

ARCHITECTURE — canonical subjects
docs/architecture/01_Master_Product_Reference.md
docs/architecture/03_System_Architecture.md
docs/architecture/04_Monorepo_and_Module_Architecture.md
docs/architecture/05_Building_Model_Domain.md
docs/architecture/06_2D_CAD_Editor_Implementation.md
docs/architecture/07_3D_Engine_Implementation.md
docs/architecture/15_Database_and_Data_Architecture.md
docs/architecture/24_Billing_AI_Credits_and_Usage.md
docs/architecture/27_DevOps_Environments_and_Deployment.md

STANDARDS
docs/standards/Buildora_AI_Backend_Engineering_Standards.md
docs/standards/Buildora_AI_Frontend_Engineering_Standards.md
docs/standards/Buildora_AI_Codex_Implementation_Reference.md
docs/standards/Buildora_AI_Solo_Developer_AI_Assisted_Development_Guide.md

PLANNING
docs/planning/Buildora_AI_Phase_Wise_Development_Checklist.md
docs/planning/Buildora_AI_Sprintwise_Project_Plan.md

EXAMPLES
docs/examples/Buildora_AI_Sample_Backend_CRUD.md
docs/examples/Buildora_AI_Sample_Frontend_CRUD.md

FINANCE
docs/finance/Buildora_AI_Development_Production_Costing_Plan.md
docs/finance/Buildora_AI_Return_On_Investment_Plan.md

SETUP
docs/setup/                                         ← 46-document setup pack
```

Several documents listed in older indexes are **not yet written**. They are
recorded in `docs/README.md` §3 with the nearest existing source.
**Never treat a path in that section as a mandatory read, and never invent
architecture because a document is missing** — stop and report the gap.

Do not create duplicate architecture documents merely because an existing file
was not found immediately.

---

# 3.1 Architecture Freeze — Read This Before Changing Anything

**The foundational architecture is FROZEN as of 2026-09-12.**

Baseline: `docs/architecture/FINAL_SYSTEM_ARCHITECTURE.md`
Decisions: `docs/architecture/34_ADR_Index.md` (ADR-001 … ADR-022, all accepted)

The following may **not** be changed by an implementation agent without a new,
explicitly approved ADR that supersedes the applicable decision:

```text
canonical Building Model            persistence/versioning model
coordinate/datum model              room/opening/type semantics
editor-session ownership            renderer boundaries
database authority                  tenant boundary
deterministic QS/BOQ/cost rules     AI ChangeSet safety model
billing ledger architecture         package dependency direction
core technology stack
```

Implementation detail may evolve within these boundaries.

**Do not relitigate a locked decision.** If you believe one is wrong, stop,
state the problem, and propose an ADR. Do not encode a different decision in code.

Performance technologies (Rust/WASM, workers, replicas) remain **evidence-driven**.
Deferred technologies do **not** become approved by appearing in a roadmap document.

---

# 4. Source-of-Truth Priority

When instructions appear to conflict, use this priority:

1. Current explicit user instruction.
2. Root `AGENTS.md`.
3. Approved ADR that explicitly supersedes an older decision.
4. Canonical architecture documents.
5. Engineering standards.
6. Current phase/sprint planning documents.
7. Examples.
8. Existing implementation.
9. Personal preference or framework convention.

If there is still a contradiction:

- stop the conflicting change,
- describe the contradiction,
- recommend one resolution,
- request/record an ADR if the decision is architectural.

Never silently choose a second architecture.

---

# 5. Development Philosophy

Buildora AI is developed as:

```text
One architecture
One canonical Building Model
One small deliverable at a time
One implementation path
Independent review
Tests before merge
```

The user is the:

```text
Product Owner
CTO
Technical Lead
Final Approver
```

Typical agent roles:

```text
Codex
→ implementation

Claude
→ architecture / planning / review / debugging

Antigravity
→ browser/UI QA and regression

User
→ final approval and integration
```

An agent may perform another role when explicitly requested, but architecture rules do not change.

---

# 6. Do Not Expand Scope Silently

For every task:

- identify the requested deliverable,
- identify the phase/sprint it belongs to,
- implement only what is needed,
- do not add unrelated features,
- do not redesign adjacent systems without approval,
- do not create future-phase infrastructure “just in case”.

If an improvement is useful but out of scope:

```text
recommend it separately
```

rather than silently implementing it.

---

# 7. Monorepo Architecture

Buildora AI is a single monorepo.

Expected high-level structure (authoritative:
`docs/architecture/04_Monorepo_and_Module_Architecture.md`, ADR-011, ADR-012):

```text
apps/
├── web/                MVP
├── api/                MVP
├── ai-worker/          gated ~Sprint 31
├── bim-worker/         gated ~Sprint 33
(render-worker: NOT CREATED — post-MVP)

packages/
├── design-system/      MVP
├── api-contracts/      MVP
├── units/              MVP
├── building-model/     MVP
├── model-session/      MVP — renderer-neutral working model (ADR-008)
├── cad-2d/             MVP
├── engine-3d/          MVP
├── qs-engine/          MVP
├── cost-engine/        MVP
(ai-tools: NOT CREATED — gated ~Sprint 27)

native/
(geometry-wasm: NOT CREATED — profiling-gated)

infrastructure/

docs/
```

**`packages/domain` does not exist and must not be created.** It was an undefined
boundary and was deleted (ADR-011).

**`packages/core` does not exist.** Do not create it. Keep concepts with their
owning module. It may be introduced later only under the three conditions in
`04_Monorepo_and_Module_Architecture.md` §5.1.

**Do not create an app or package before a real consumer exists** (ADR-012).
Placeholder directories may hold a README naming the gate, but no build or
runtime configuration.

## Dependency direction — to be enforced in CI (Sprint 01 acceptance criteria)

```text
units → building-model → model-session → { cad-2d, engine-3d }
        building-model → qs-engine → cost-engine
```

Dependencies point **downward only**; no lateral edges; **no cycles**.
`cad-2d` and `engine-3d` must never import each other.

Do not split frontend, backend, AI, BIM, infrastructure, contracts, or documentation into separate Git repositories without an approved ADR.

One repository does **not** mean one deployment.

Each deployable may have its own:

- Dockerfile,
- environment,
- scaling,
- release process.

---

# 8. Approved Core Technology Direction

Unless superseded by an approved ADR:

## Frontend

```text
React
Next.js App Router
TypeScript
Tailwind
Radix primitives
Storybook
React Hook Form
Zod
TanStack Query when client-side remote state is needed
Zustand only for ephemeral UI/editor state
```

## 2D

```text
PixiJS
custom CAD/editor layer
Web Workers where useful
Rust/WASM only for measured geometry hotspots
```

## 3D

```text
direct Three.js
dedicated engine package
WebGL stable path
WebGPU progressive enhancement later
```

Do not introduce React Three Fiber as the core professional editor architecture without an ADR.

## Backend

```text
NestJS
TypeScript
modular monolith
REST/OpenAPI
WebSockets where required
```

Do not introduce microservices because a module exists.

## Data

```text
PostgreSQL
PostGIS
pgvector
Redis / Valkey
R2 / S3-compatible object storage
```

## ORM / DB access

```text
Drizzle ORM
node-postgres
```

## AI / CV

```text
TypeScript AI Gateway
OpenAI provider adapter
Python
FastAPI
OpenCV
PyTorch
ONNX Runtime
```

## BIM / CAD interoperability

```text
That Open Engine / Components / Fragments
web-ifc
IfcOpenShell
OpenCascade later when justified
ODA later when commercially justified
```

## Workflows

```text
Temporal for durable workflows
```

Do not use Temporal for trivial CRUD.

## Infrastructure

```text
Docker
Cloudflare
AWS ECS/Fargate
ECR
managed PostgreSQL
managed Redis
Temporal Cloud
Sentry
OpenTelemetry
```

Kubernetes is not the default architecture.

---

# 9. Canonical Building Model — Non-Negotiable

The **Buildora AI Building Model is the single source of truth**.

The following must never become separate authoritative states:

```text
2D
3D
IFC
mesh
QS
BOQ
cost
```

Correct architecture:

```text
Canonical Building Model
    ├─→ 2D representation
    ├─→ 3D representation
    ├─→ IFC export
    ├─→ quantities
    ├─→ BOQ
    ├─→ cost
    └─→ AI context
```

The Three.js scene is **not** the database.

The PixiJS scene is **not** the database.

IFC is **not** the internal editable source of truth.

Rendered mesh geometry is **not** the canonical domain model.

---

# 10. Building Model Units

Canonical geometry uses:

```text
millimetres
```

unless an approved domain document explicitly states otherwise.

## Datum (ADR-005 — locked)

```text
Project-local datum:  ground-floor finished floor level = 0 mm
Level.elevationMm:    signed from project datum — the SINGLE
                      authoritative vertical fact for a level
Element z:            LEVEL-RELATIVE

absoluteZ    = level.elevationMm + (baseOffsetMm ?? 0)   derived, never stored
floorToFloor = next.elevationMm − current.elevationMm    derived, never stored
```

Slab structural thickness belongs to the **Slab and its assembly**, not to Level.
Real-world georeference lives on **Site**, separate from building-local coordinates.

Unit conversion belongs in:

```text
packages/units
```

or equivalent typed domain utilities.

Do not spread ad-hoc unit conversions across UI code.

---

# 11. Building Model Domain Purity

`packages/building-model`, `packages/model-session`, `packages/units`,
`packages/qs-engine` and `packages/cost-engine` must remain framework-independent.

They must not depend directly on:

- React,
- Next.js,
- PixiJS,
- Three.js,
- NestJS,
- Drizzle,
- OpenAI,
- Stripe,
- Clerk.

Domain code should be testable without web frameworks or infrastructure.

---

# 12. Stable IDs

Building elements use stable IDs.

Examples:

```text
Wall
Door
Window
Room
Slab
Column
Beam
Stair
Opening
```

A domain element ID must remain consistent across:

```text
model
2D
3D
QS
BOQ
cost references
AI tool results
audit/versioning
```

Do not generate renderer-specific replacement identities.

---

# 12.1 Room, Opening and Type Semantics — Locked

## Rooms (ADR-006)

```text
wall centreline graph → CONNECTIVITY (which regions are enclosed)
        ↓
resolve junctions → derive INTERIOR WALL FACES
        ↓
usable room boundary → persist with stable ID + BOUNDS relationships
```

The persisted boundary comes from **interior wall faces**, never from wall
centrelines. Centreline boundaries overstate floor area by half a wall thickness
on every edge and are not professionally defensible.

- **Area is always computed from the boundary.** A user-entered area is never
  stored as authoritative geometry.
- `user_defined = false` → recomputed on bounding-wall change; stable ID, name,
  number and finishes preserved; material area change flagged.
- `user_defined = true` → **never silently overwritten**.
- Room detection lives in `packages/building-model`, not in the editor.

## Openings

**A door or window IS the opening.** Its hosted geometry creates the hole. Do not
create a second `Opening` element for the same physical hole.

A standalone `opening` element exists **only** for fixture-less holes: service
penetration, arch, unfilled wall opening.

**QS sees hosted doors/windows and standalone openings through ONE
opening/deduction abstraction.** Two code paths here produce divergent net wall
area — the most important QS number.

## Three identity axes (ADR-007, ADR-020)

```text
element_type_id       → WHAT IT IS          schedules, IFC type, type-level edits
assembly_version_id   → WHAT IT'S BUILT OF  materials, labour, plant, waste
finish_assignment     → WHAT IT LOOKS LIKE  surface finish (ADR-020)
```

Never conflate these. Repainting a wall creates a **new finish assignment**,
never a new assembly version.

## FF&E — furniture, fixtures, equipment (ADR-019)

FF&E instances are **domain elements** carrying `element_class = FFE`. They get
stable IDs, tenancy, commands, versioning, undo, audit and provenance like any
other element.

```text
✗ furniture held only in a Three.js scene       ← release-blocking
✗ GLB binaries in relational columns            ← release-blocking
✓ ffe_assets (catalogue, versioned) + ffe_instances (placed elements)
```

**FF&E never bounds a room and never affects a construction quantity.** Every
element iteration in QS and topology code must honour the class.

## Manual interior work requires no AI

Walkthrough, material/finish changes and furniture placement are **deterministic
commands**. Manual and AI paths converge on the **same commands** — there is no
AI-specific write path. Ordinary interaction consumes **zero LLM tokens**.

---

# 13. Command Model

Meaningful model changes use semantic commands.

Examples:

```text
ADD_WALL
MOVE_WALL
RESIZE_OPENING
CHANGE_MATERIAL
DELETE_ELEMENT
```

A command should carry enough information for:

- validation,
- version control,
- audit,
- undo/redo,
- AI proposal,
- collaboration.

Do not treat arbitrary scene mutation as a domain operation.

---

# 14. Optimistic Concurrency

Model mutations must eventually use:

```text
baseVersion
```

or the canonical equivalent.

Server flow:

```text
client base version
→ server verifies current version
→ command validated
→ transaction
→ new model version
```

Stale write:

```text
MODEL_VERSION_CONFLICT
```

Do not silently overwrite newer model changes.

---

# 15. Versioning & Audit

Building Model architecture supports:

```text
current materialized state
+
append-oriented commands/change items
+
model versions
+
periodic snapshots
```

This enables:

- undo/redo,
- audit,
- AI preview,
- rollback,
- version compare,
- future collaboration.

Do not introduce destructive mutation patterns that make history impossible.

---

# 16. AI Must Never Directly Mutate Live Geometry

AI modification flow:

```text
User Request
→ AI Gateway
→ Structured ChangeSet
→ Draft/Sandbox
→ Geometry Validation
→ QS/Cost Impact
→ Preview
→ User Approval
→ Transactional Commit
```

Forbidden:

```text
LLM
→ direct UPDATE building_elements
```

AI proposes.

Domain validates.

User/system policy approves.

Deterministic code commits.

---

# 17. AI Tooling Rules

AI tools must be typed.

Examples:

```text
get_project_summary
get_room
get_element
get_quantities
get_boq
get_cost
search_documents
propose_move_wall
```

Tool inputs and outputs must use runtime-validated schemas.

Do not expose raw database access to the LLM.

Do not allow arbitrary SQL generated by the LLM to run against production.

---

# 18. AI Model Routing

Use logical aliases such as:

```text
FAST
BALANCED
ADVANCED
EXPERT
```

Provider model IDs belong in configuration.

Do not embed provider-specific model names throughout domain/business code.

AI routing should consider:

- task,
- complexity,
- cost,
- latency,
- user entitlement,
- risk.

---

# 19. Deterministic Calculations

The LLM is not authoritative for:

```text
geometry
quantities
BOQ totals
financial arithmetic
tax arithmetic
billing balances
credit balances
```

AI may:

- explain,
- summarize,
- recommend,
- classify,
- propose.

Deterministic engines calculate authoritative results.

---

# 20. QS Golden Rule

QS calculations must be reproducible from model + rules.

Canonical golden test example:

```text
Wall:
5000 mm × 2700 mm

Opening:
1000 mm × 2100 mm

Gross:
13.5 m²

Opening:
2.1 m²

Net:
11.4 m²
```

This or equivalent golden tests must remain part of QS verification.

---

# 21. Costing Rules

Financial calculations use exact decimal-safe arithmetic.

Prefer PostgreSQL:

```text
NUMERIC
```

for persisted money/rates.

Do not use floating-point arithmetic for authoritative financial totals.

Every estimate should be traceable to:

```text
model version
quantity run
BOQ version
rate-book version
assumptions
cost-engine version
```

---

# 22. Materials & Assemblies

Geometry does not directly equal cost.

Use assemblies to expand building elements into:

- materials,
- labour,
- plant,
- waste,
- other cost components.

Waste must be configurable.

Do not hardcode material quantities into UI components.

---

# 23. Database Architecture

PostgreSQL is authoritative for relational/domain state.

Use:

```text
PostgreSQL → authoritative domain state
Redis      → ephemeral/cache/presence/rate limits
R2/S3      → large blobs/artifacts/files
PostGIS    → real-world geography
pgvector   → semantic embeddings
Temporal   → durable workflow state/orchestration
```

Redis must not contain irreplaceable project state.

Object storage must not replace domain metadata.

---

# 24. Multi-Tenancy

Organization is the primary tenant boundary.

Tenant-owned repositories/queries must carry tenant scope explicitly.

Never fetch:

```text
project by projectId only
```

when tenant context is required.

Cross-tenant access is a release-blocking defect.

Tests must cover tenant isolation.

---

# 25. Database Access

Use module-owned repositories.

Avoid:

```text
global generic database service
```

that allows arbitrary access from every module.

Repositories must express business scope.

Example:

```text
findProjectById(organizationId, projectId)
```

not:

```text
findById(projectId)
```

where tenancy matters.

---

# 26. JSONB Rules

JSONB is permitted for flexible typed properties and geometry payloads.

Every important JSONB structure requires:

- TypeScript type/schema,
- runtime validation,
- schema version where evolution matters.

Do not create untyped dumping-ground JSON blobs.

---

# 27. Large Data

Do not store:

- large mesh arrays,
- render files,
- PDFs,
- IFC binaries,
- user uploads,

inside ordinary relational columns.

Use object storage with database metadata.

---

# 28. Database Migrations

Production migrations are committed artifacts.

Do not:

- use production `schema push`,
- modify old applied migration files,
- run migrations automatically at API startup.

Use:

```text
expand
→ migrate/backfill
→ switch
→ contract later
```

for non-trivial changes.

---

# 29. Transactions

Keep DB transactions short.

Never hold a database transaction while waiting for:

- OpenAI,
- Stripe,
- R2,
- external APIs,
- rendering,
- long worker processing.

---

# 30. Backend Standards

Controllers are thin.

Preferred flow:

```text
Controller
→ validation/auth
→ use case/application service
→ domain
→ repository/infrastructure
```

Controllers should not contain business logic.

---

# 31. API Contracts

Shared API schemas live in:

```text
packages/api-contracts
```

Use Zod or the canonical runtime schema system.

The frontend must not maintain a manually duplicated version of backend request/response types.

---

# 32. API Errors

Use structured problem responses.

Canonical direction:

```text
RFC 9457
application/problem+json
```

Stable error codes include examples such as:

```text
PROJECT_NOT_FOUND
MODEL_VERSION_CONFLICT
FORBIDDEN
VALIDATION_FAILED
INSUFFICIENT_CREDITS
```

Do not return internal stack traces to clients.

---

# 33. Frontend Architecture

Next.js App Router rules:

- Server Components by default.
- Client Components only where interaction/browser APIs require them.
- Keep data fetching close to the appropriate server/client boundary.
- Do not mark entire route trees `"use client"` for convenience.

---

# 34. Frontend State

Do not create one giant application Zustand store.

Use:

```text
server/database   → authoritative persistent state
model-session     → in-memory WORKING COPY of the canonical model
                    (baseVersion, optimistic commands, pending queue,
                     reconciliation) — packages/model-session, ADR-008
TanStack Query    → project metadata, lists, QS/BOQ/cost read models
Zustand           → ephemeral UI state ONLY
                    (active tool, selection, panel visibility,
                     camera/UI preferences, temporary interaction state)
```

The canonical Building Model must not become a Zustand-only authority.

**`packages/model-session` is a synchronized working copy, not a second
authoritative database.** PostgreSQL remains authoritative.

**TanStack Query must not own working geometry.** It is wrong for CAD interaction
rates and its refetch semantics do not fit a working document.

---

# 35. 2D Editor Performance

React controls editor chrome and panels.

PixiJS handles high-frequency canvas rendering.

Avoid React-rendering thousands of CAD elements directly.

High-frequency interactions should not cause the whole application tree to rerender.

Use workers/optimized geometry where justified.

---

# 36. 3D Engine Rules

Three.js is the rendering engine.

The 3D scene is produced from the canonical model through adapters.

Selection IDs should map to domain IDs.

Do not hide business rules inside Three.js object metadata.

---

# 37. 2D ↔ 3D Synchronization

Correct:

```text
Building Model changed
→ 2D adapter updates
→ 3D adapter updates
```

Incorrect:

```text
2D scene changes
→ patch 3D directly
```

or:

```text
3D scene changes
→ separately update database
```

All model edits must converge through domain commands.

---

# 38. BIM / IFC

IFC is:

```text
interchange
import
export
external BIM mapping
```

It is not Buildora AI's internal source of truth.

Maintain external GUID mapping when importing/exporting.

Do not reshape core domain architecture around IFC limitations.

---

# 39. Plan / Drawing Import

Preferred flow:

```text
Upload
→ detect type
→ vector/raster path
→ deterministic/vector extraction
→ CV if required
→ AI semantics
→ confidence
→ human verification
→ Building Model commands
```

Do not rasterize a good vector/CAD source and ask an LLM to rediscover precise geometry.

AI handles ambiguity and semantics, not authoritative raw geometry when precise vector data exists.

---

# 40. Confidence Is First-Class

Recognition results should support:

- confidence score,
- source/provenance,
- verification status.

Low-confidence recognition requires user review.

Do not silently convert uncertain AI recognition into authoritative model geometry.

---

# 41. Documents / RAG

Project document Q&A must preserve traceability.

Desired lineage:

```text
AI Answer
→ Retrieval Run
→ Chunk
→ Document Revision
→ File
```

Project answers should cite source material in the UI where appropriate.

Tenant/project filtering must occur before or together with semantic ranking.

---

# 42. Embeddings

Embedding profile/model/dimension must be explicit.

Do not mix incompatible embedding dimensions in the same search profile.

Start simple.

Do not introduce Elasticsearch before PostgreSQL FTS/trigram/pgvector is demonstrably insufficient.

---

# 43. Authentication & Authorization

Authentication provider may be Clerk initially, behind Buildora-owned abstraction.

Buildora AI owns authorization.

Never assume:

```text
authenticated = authorized
```

Every sensitive operation checks:

- user,
- organization,
- membership,
- role/policy,
- resource scope.

---

# 44. Billing Architecture

Customer-facing credits are **not raw AI tokens**.

Plans may be **RECURRING** or **FIXED_TERM** (ADR-021). A project pass ends at its
term with **no cancellation event** — that is a successful outcome, not churn.
Expiry archives the project (data preserved); reactivation restores capability.

Feature code reads **entitlements** (`feature.qs.advanced`, `limit.active_projects`),
never plan names.

The entitlement catalogue is **runtime-editable by a platform admin** (ADR-022).
Moving a capability between tiers is a data change, not a deployment:

```text
one customer, now      → plan_entitlement_overrides   (immediate)
everyone, going forward → publish a new plan version  (existing subscribers
                                                       keep their version)
```

Customer-facing concepts:

```text
Plan
Entitlements
AI Design Credits
Render Credits
Project limits
Team limits
Storage limits
```

Internal provider usage tracks:

- model,
- provider,
- input tokens,
- cached input,
- output/reasoning,
- actual provider cost.

Keep these models separate.

---

# 45. Billing Ledger

Do not store billing as:

```text
organization.aiCredits = 27
```

only.

Use wallets/ledger/reservation concepts.

Credit operations include:

```text
GRANT
RESERVE
RELEASE
CONSUME
REFUND
REVOKE
ADJUSTMENT
EXPIRE
```

Financial/credit mutation must be idempotent and auditable.

---

# 46. Billing Jobs

One customer-visible AI job may cause multiple provider calls.

Customer charge:

```text
job/action level
```

Internal cost:

```text
sum of provider usage calls
```

Retries and escalation must not accidentally charge the customer twice.

---

# 47. Stripe

Stripe is authoritative for payment-processing facts such as:

- payment status,
- Stripe invoice,
- Stripe subscription object,
- refunds/disputes.

Buildora AI is authoritative for:

- plan catalog/version,
- entitlements,
- credit wallet,
- usage,
- feature access,
- AI/render accounting.

Never grant purchased credits only because the browser reached a success URL.

Grant after authoritative server-side payment confirmation.

---

# 48. Webhooks

Webhook handling:

```text
verify signature
→ store unique event/inbox record
→ acknowledge/queue
→ process idempotently
```

Duplicate webhook delivery must be safe.

---

# 49. Temporal

Use Temporal for durable multi-step processes such as:

- plan recognition,
- BIM processing,
- report generation,
- long AI jobs,
- rendering,
- deletion workflows,
- reconciliation.

Do not use Temporal as an expensive replacement for ordinary synchronous CRUD.

Workflow code must remain replay-compatible.

---

# 50. Background Work

Long-running work must not stay inside an HTTP request.

Preferred:

```text
request
→ create job
→ return accepted/status
→ durable/background workflow
→ update status
```

---

# 51. Object Storage

User files are private by default.

Preferred upload:

```text
client asks API
→ API validates tenant/project/entitlement
→ short-lived presigned upload
→ direct upload
→ finalize/verify metadata
```

Do not proxy every large upload through the API unless necessary.

---

# 52. User Uploads Are Untrusted

Apply:

- file-size limits,
- type validation,
- timeout,
- parser isolation,
- resource limits,
- safe filenames/object keys.

Heavy BIM/CAD parsing belongs in dedicated workers, not the public API process.

---

# 53. Security

Never commit:

- passwords,
- API keys,
- private certificates,
- production environment files.

Never log:

- bearer tokens,
- database passwords,
- Stripe secrets,
- OpenAI keys,
- complete presigned URLs,
- raw private documents by default.

---

# 54. Logging

Use structured logs.

No production `console.log()` as the logging architecture.

Log useful identifiers such as:

```text
service
environment
release
traceId
organizationId
projectId
jobId
modelVersion
```

Do not put sensitive document/prompt contents in logs.

---

# 55. Observability

Production services require:

- health/readiness,
- structured logs,
- error monitoring,
- release metadata,
- metrics/traces where useful.

Preferred direction:

```text
Sentry
OpenTelemetry
AWS/runtime metrics
```

Observability is part of Definition of Done for production features.

---

# 56. DevOps Rules

Initial production architecture:

```text
Cloudflare
AWS ECS/Fargate
ECR
managed PostgreSQL
managed Redis
R2
Temporal Cloud
GitHub Actions
```

Do not introduce Kubernetes without an approved ADR and measured reason.

---

# 57. CI/CD Rules

Build once and promote the same immutable artifact.

Use:

```text
Git SHA
image digest
release version
```

Do not use mutable `latest` as the production deployment authority.

GitHub Actions should use OIDC/short-lived cloud credentials where supported.

---

# 58. Deployment

Expected flow:

```text
PR
→ CI
→ merge
→ build image
→ staging
→ migration
→ smoke/E2E
→ production approval
→ same image digest
→ production migration
→ deploy
→ observe
```

Do not bypass staging for normal production releases.

---

# 59. Health Checks

Liveness:

```text
process is alive
```

Readiness:

```text
service can safely accept traffic
```

Do not fail liveness merely because OpenAI or Stripe is down.

---

# 60. Graceful Shutdown

Services must handle termination safely.

Workers should stop polling and complete/abandon work safely.

APIs should stop accepting new work and close resources cleanly.

---

# 61. Testing Expectations

Every meaningful change should include the lowest appropriate level of test:

```text
unit
integration
E2E
```

Critical areas require strong automated coverage:

- tenant isolation,
- Building Model,
- version conflicts,
- geometry invariants,
- QS,
- cost,
- billing/credits,
- webhook idempotency,
- AI ChangeSets,
- migrations.

---

# 62. Do Not Skip Tests to Make CI Green

If a test fails:

- understand why,
- fix code or correct the test if the test is genuinely invalid,
- document intentional behavior changes.

Do not:

```text
.skip
xit
disable test
loosen assertion
```

merely to pass CI.

---

# 63. TypeScript Quality

Do not use:

```text
any
@ts-ignore
```

without an isolated, documented reason.

Prefer:

- strict types,
- discriminated unions,
- branded/domain types where helpful,
- exhaustive matching,
- runtime validation at trust boundaries.

---

# 64. Python Quality

Python projects should use the repository's chosen `uv` workflow.

Use:

- typed interfaces where practical,
- deterministic dependency lock,
- lint/type/test tools defined by the project.

Do not create ad-hoc virtual environment instructions that conflict with repo setup docs.

---

# 65. Rust / WASM

Rust belongs only where profiling or geometry complexity justifies it.

Do not move ordinary product logic into Rust because it appears faster.

Good candidates:

- intersections,
- clipping,
- polygon operations,
- offsets,
- room detection,
- geometric constraints,
- triangulation,
- spatial indexes.

---

# 66. Dependency Policy

Before adding a dependency:

1. Check if existing dependency solves it.
2. Check maintenance/security.
3. Check bundle/runtime impact.
4. Check license.
5. Explain why it is needed.

Do not upgrade unrelated major dependencies during a small feature task.

---

# 67. Package Managers

Use repository-standard tools:

```text
TypeScript/JS → pnpm
Python        → uv
Rust          → cargo
```

Do not introduce npm/yarn/pipenv/poetry without an ADR/reason.

Commit lockfiles.

---

# 68. Code Placement

Code belongs in the bounded context that owns it.

Examples:

```text
API transport schema
→ packages/api-contracts

core geometry/model behavior
→ packages/building-model

2D rendering/editor behavior
→ packages/cad-2d

3D rendering
→ packages/engine-3d

quantity rules
→ packages/qs-engine

cost calculations
→ packages/cost-engine
```

Do not build giant `utils/` dumping grounds.

---

# 69. Reuse vs Abstraction

Do not create abstractions before a real repeated pattern exists.

Prefer:

```text
clear duplication twice
```

over:

```text
premature generic framework
```

when the domains are not actually the same.

But once a canonical shared concept exists, do not duplicate it across apps.

---

# 70. Examples Are Patterns, Not Blind Copy Targets

`docs/examples/` shows preferred patterns for normal SaaS CRUD.

Use them for:

- projects,
- settings,
- conventional entities.

Do **not** use simple CRUD architecture for:

```text
Building Model mutations
credit ledgers
versioned financial history
workflow orchestration
```

where domain architecture requires commands/ledgers/versioning.

---

# 71. Git Safety

Never:

- delete unrelated user changes,
- run destructive reset/clean commands without explicit approval,
- rewrite history without approval,
- force-push without approval,
- commit secrets,
- commit large generated model files.

Do not commit or push automatically unless the user explicitly asks.

---

# 72. Branches

Preferred:

```text
main
feature/*
fix/*
```

Keep branches narrow.

Do not create long-lived architecture branches unnecessarily.

---

# 73. Commit Scope

Small, coherent commits are preferred.

Example:

```text
feat(building-model): add wall aggregate
test(qs): add net wall area golden case
chore(ci): add typecheck job
```

Do not mix unrelated formatting, dependency upgrades, and feature code in one change unless necessary.

---

# 74. Repository Hygiene

Do not commit:

- node_modules,
- build outputs,
- `.env`,
- Python virtual environments,
- customer uploads,
- AI model weights,
- render outputs,
- Terraform state,
- local DB dumps.

Follow root `.gitignore`.

---

# 75. Documentation Updates

If behavior or architecture changes, update documentation in the same change.

Examples:

- API contract changed → API docs/standards where necessary.
- architecture decision changed → ADR.
- operational procedure changed → `docs/operations`.
- phase checklist completed → update planning status only if user expects it.

Do not allow docs to describe an obsolete architecture.

---

# 76. ADR Requirement

Create/propose an ADR for changes such as:

- framework replacement,
- database technology change,
- canonical model storage redesign,
- new source of truth,
- Kubernetes adoption,
- microservice extraction,
- alternative billing architecture,
- different AI mutation architecture,
- different tenancy boundary.

Do not silently encode architectural decisions in code.

---

# 77. Performance

Measure before introducing complexity.

Do not add:

- Rust,
- microservices,
- Kafka,
- Redis clustering,
- Kubernetes,
- sharding,

based only on hypothetical future scale.

Use profiling and production evidence.

---

# 78. Accessibility

Frontend work should consider:

- keyboard interaction,
- focus,
- labels,
- contrast,
- semantic structure.

Professional editor shortcuts must not make the rest of the UI inaccessible.

---

# 79. Error UX

Errors should be:

- actionable,
- non-sensitive,
- mapped from stable backend codes.

Examples:

```text
MODEL_VERSION_CONFLICT
→ explain stale model and recovery

INSUFFICIENT_CREDITS
→ explain required action

PROJECT_NOT_FOUND
→ safe not-found experience
```

Do not expose raw server error text.

---

# 80. Cost Awareness

Buildora AI is cost-sensitive.

Before adding expensive services/features, read:

```text
docs/finance/Buildora_AI_Development_Production_Costing_Plan.md
docs/finance/Buildora_AI_Return_On_Investment_Plan.md
```

Expensive operations should be:

- measurable,
- metered,
- bounded,
- linked to customer value.

---

# 81. Do Not Move Forward If

Stop and resolve before continuing if:

- canonical Building Model direction is unclear,
- tenant isolation fails,
- tests are being skipped,
- financial arithmetic uses floats,
- AI directly mutates geometry,
- 2D and 3D have separate authority,
- QS depends on LLM arithmetic,
- billing is only a mutable balance,
- webhook handling is non-idempotent,
- migration safety is unclear,
- secrets are committed,
- uploads are public unintentionally,
- production backup strategy is missing for paid launch.

---

# 82. Phase Discipline

Before implementing a major feature, locate it in:

```text
docs/planning/Buildora_AI_Phase_Wise_Development_Checklist.md
docs/planning/Buildora_AI_Sprintwise_Project_Plan.md
```

Do not implement later-phase architecture before preceding exit gates unless the user explicitly approves it.

---

# 83. First Vertical Slice Principle

Do not start Buildora AI by generating whole buildings with AI.

The first meaningful editor/model proof remains:

```text
Create Project
→ Ground Floor
→ Four Walls
→ Room
→ Door
→ Window
→ 3D
→ modify wall
→ 2D updates
→ 3D updates
→ save
→ reload
→ identical
```

Then:

```text
model
→ QS
→ BOQ
→ cost
```

Then:

```text
AI proposal
→ validated ChangeSet
→ preview
→ approval
→ same deterministic pipeline
```

---

# 84. Agent Task Workflow

For each task:

## Before coding

Report briefly:

```text
Current phase/sprint:
Goal:
Relevant docs read:
Files/modules likely affected:
Risks/assumptions:
```

For a trivial task, keep this compact.

## During coding

- keep scope narrow,
- follow existing patterns,
- preserve unrelated work,
- add tests,
- run appropriate validation.

## After coding

Report:

```text
What changed
Files changed
Tests/commands run
Results
Known limitations
Follow-up only if necessary
```

Do not claim tests passed if they were not run.

---

# 85. Review Before Completion

Before declaring a task complete, check:

### Architecture
- Does it preserve the canonical model?
- Does it introduce duplicate state?
- Does it fit the bounded context?

### Backend
- Thin controller?
- Typed contract?
- Tenant scope?
- Stable errors?

### Frontend
- Server/client boundary correct?
- Persistent state not hidden in Zustand?
- Render engine isolated?

### Database
- Correct types?
- Constraints?
- Migration?
- Tenant index?
- No large blob misuse?

### AI
- Typed tools?
- No direct mutation?
- Cost/usage visible?

### Billing
- Idempotent?
- Ledger/reservation?
- Exact money?

### Security
- Authorization?
- Secrets?
- Upload safety?
- Logs safe?

### Tests
- Happy path?
- failure path?
- tenant/concurrency where relevant?

---

# 86. Expected Commands

Use scripts that actually exist in `package.json`.

Likely project-wide checks may eventually include:

```text
pnpm lint
pnpm typecheck
pnpm test
pnpm build
pnpm verify
```

Do not invent or report a command as successful without checking the repository.

For Python/Rust, use the commands defined by those projects.

---

# 87. No False Completion Claims

Never say:

```text
done
fully implemented
production-ready
tests all pass
```

unless the work and verification actually support that statement.

Be explicit when:

- a dependency is not installed,
- tests cannot run,
- configuration is missing,
- implementation is partial,
- a later phase is intentionally deferred.

---

# 88. Protect User Decisions

Do not replace an approved technology simply because another tool is fashionable.

Examples:

- do not replace NestJS with another backend framework,
- do not replace PixiJS with a new canvas library,
- do not replace direct Three.js with R3F,
- do not replace Drizzle with another ORM,
- do not replace Temporal casually,
- do not split the monorepo.

If there is a serious reason to reconsider:

```text
propose
compare
ADR
user approval
```

first.

---

# 89. Buildora AI Naming

Use:

```text
Buildora AI
```

for the product.

Use:

```text
buildora-ai
```

for repository/package scope where appropriate.

Do not introduce the old project name `BuildWise` into new production code or documentation.

**The rename is complete as of 2026-09-12.** No `BuildWise` reference remains in
any document, template, or configuration file. If one reappears, it is a
regression — fix it rather than treating it as legacy.

---

# 90. Official Documentation & Dependency Freshness Agent Instructions

> **Purpose:** Mandatory dependency, integration, SDK, API, framework, and external-service verification rules for all AI agents working on Buildora AI.
>
> **Applies to:** Codex, Claude, Antigravity, IDE agents, CI repair agents, and any future automated contributor.

---

## 90.0 Policy

Buildora AI must be built against current, supported, stable technology.

AI agents must **never rely only on remembered framework/package knowledge** when implementing an integration, installing a dependency, configuring a service, or using an external API.

**Current official documentation is the primary technical source.**

---

## 90.1 Mandatory Documentation Check Before Implementation

Before starting implementation involving any:

- framework,
- package,
- SDK,
- API,
- cloud service,
- database extension,
- AI provider,
- CAD/BIM integration,
- CLI,
- build tool,
- deployment service,
- authentication provider,
- payment provider,
- monitoring/observability integration,

the agent MUST:

1. Identify the exact technology involved.
2. Read the relevant **current official documentation**.
3. Confirm the currently supported stable version.
4. Confirm that the package/API/module being used is not deprecated.
5. Check whether the repository already pins a compatible version.
6. Review migration notes or breaking changes when relevant.
7. Confirm compatibility with the Buildora AI architecture.
8. Only then begin implementation.

Examples include:

```text
Next.js
React
NestJS
Drizzle ORM
PostgreSQL
PostGIS
pgvector
Redis / Valkey
Temporal
Stripe
Clerk
OpenAI
Cloudflare
AWS
Three.js
PixiJS
That Open
web-ifc
IfcOpenShell
FastAPI
PyTorch
ONNX Runtime
Rust
WASM tooling
Docker
GitHub Actions
Sentry
OpenTelemetry
```

Do not implement based solely on:

- remembered APIs,
- old tutorials,
- outdated Stack Overflow answers,
- blog posts,
- obsolete examples,
- code copied from older major versions.

---

## 90.2 Official Sources First

Use information sources in this order:

1. Official documentation.
2. Official API reference.
3. Official migration/upgrade guide.
4. Official GitHub repository.
5. Official release notes/changelog.
6. Official package registry metadata.
7. Maintainer-provided examples.

Community tutorials, blog posts, forum answers, videos and third-party examples may be used only as supplementary material.

They must never override current official documentation.

---

## 90.3 Stable Versions Only

Unless the user explicitly approves otherwise, use:

> **the latest stable, supported version that is compatible with Buildora AI and its existing dependency graph.**

Do not automatically use:

- alpha versions,
- beta versions,
- release candidates,
- nightly builds,
- experimental releases,
- deprecated versions,
- unsupported versions,
- abandoned packages,
- unmaintained forks.

Example:

```text
latest stable
✓ acceptable

latest beta
✗ do not use unless explicitly approved
```

"Latest" does not mean "highest published version number."

Always verify the stable release channel.

---

## 90.4 Never Install Deprecated Packages

Before adding a dependency, verify:

- the package is actively maintained,
- the package is not deprecated,
- the intended API is not deprecated,
- official documentation still recommends/supports it,
- the package is compatible with the current runtime/framework,
- the licence is acceptable,
- there is no officially recommended replacement that should be used instead.

If official documentation says:

```text
deprecated-package
→ replacement-package
```

use the supported replacement unless there is an explicitly approved architectural reason not to.

Do not knowingly introduce deprecated architecture into new code.

---

## 90.5 Check APIs, Not Only Package Versions

A package can be current while a method, class, config key, CLI command or endpoint inside it is deprecated.

Before implementation, verify that the exact:

- method,
- class,
- hook,
- API endpoint,
- environment variable,
- config option,
- command,
- integration pattern,

is still current and supported.

Example:

```text
Package:
current

Method:
deprecated

Result:
DO NOT USE
```

Use the officially documented replacement.

---

## 90.6 External Integration Documentation Is Mandatory

Before implementing any external integration, read the provider's current official integration documentation.

Examples:

```text
OpenAI
Stripe
Clerk
Temporal
Cloudflare
AWS
Sentry
GitHub
```

Verify at minimum:

- current SDK,
- current API version,
- authentication method,
- recommended integration flow,
- retries,
- idempotency requirements,
- webhook behavior,
- rate limits,
- timeout guidance,
- security recommendations,
- versioning/deprecation policy.

Do not copy an old integration pattern merely because it still compiles.

---

## 90.7 OpenAI / AI Provider Rule

Before implementing or modifying AI-provider functionality:

1. Read the current official provider API documentation.
2. Verify current model IDs.
3. Verify currently supported API interfaces.
4. Verify tool/function-calling format.
5. Verify structured-output capabilities.
6. Verify usage/token metadata returned by the API.
7. Verify current pricing separately from product logic.
8. Check deprecation notices.
9. Verify current SDK version.

Buildora domain code must use logical aliases such as:

```text
FAST
BALANCED
ADVANCED
EXPERT
```

Provider-specific model IDs belong in configuration.

Do not hard-code AI SDK assumptions from old tutorials.

---

## 90.8 Database / ORM Rule

Before implementing database or ORM functionality, verify the current official documentation for relevant technologies such as:

```text
PostgreSQL
Drizzle ORM
node-postgres
PostGIS
pgvector
```

Particularly verify current guidance for:

- migrations,
- transactions,
- connection pooling,
- prepared statements,
- JSONB,
- UUID,
- NUMERIC,
- extensions,
- indexes,
- RLS,
- serverless/pooled connection behavior.

Do not generate production database behavior from outdated ORM examples.

---

## 90.9 Frontend Framework Rule

Before implementing framework-specific frontend behavior, verify current documentation for:

```text
Next.js
React
TanStack Query
Zustand
React Hook Form
Zod
Tailwind
Radix
```

Pay special attention to frequently changing framework features:

- Next.js App Router,
- Server Components,
- Client Components,
- caching,
- route handlers,
- server actions,
- metadata,
- rendering behavior,
- React APIs.

Do not apply patterns from older Next.js/React generations without verification.

---

## 90.10 Graphics / CAD / BIM Rule

Before implementing graphics or BIM functionality, verify current official documentation for:

```text
PixiJS
Three.js
That Open
web-ifc
IfcOpenShell
```

Do not assume examples written for older major versions remain valid.

Graphics/BIM libraries frequently change:

- initialization APIs,
- event APIs,
- render loops,
- loaders,
- disposal APIs,
- WebGL/WebGPU behavior,
- IFC APIs,
- scene/fragment APIs.

Implementation must match the installed version's documentation.

---

## 90.11 Dependency Installation Procedure

Before installing a new dependency, determine:

```text
Package:
Purpose:
Current stable version:
Existing repository version:
Official documentation:
Maintenance status:
Deprecation status:
Licence:
Compatibility:
Reason required:
```

Then install it using the repository-standard package manager:

```text
TypeScript / JavaScript → pnpm
Python                  → uv
Rust                    → cargo
```

Do not introduce another package manager without an approved ADR.

---

## 90.12 Do Not Blindly Use `latest`

Do not run:

```bash
pnpm add some-package@latest
```

without first checking what `latest` resolves to.

Confirm that it:

- is stable,
- is supported,
- is compatible,
- does not force an unrelated framework/runtime upgrade,
- does not introduce unreviewed breaking changes.

The target is:

```text
latest stable compatible version
```

not:

```text
highest published version
```

---

## 90.13 Existing Repository Versions Take Priority

If the repository already uses a stable supported version:

Do not upgrade it merely because a newer version exists.

Upgrade only when:

- required by the requested feature,
- required for security,
- required for compatibility,
- current version is deprecated/EOL,
- the user explicitly asks for an upgrade,
- the task is a dependency-maintenance task.

Avoid unrelated dependency upgrades in feature work.

---

## 90.14 Major-Version Upgrade Rule

A major-version upgrade must never be hidden inside an unrelated task.

Example:

```text
Task:
Add project search

Unacceptable side effect:
Upgrade Next.js major version
```

Major upgrades require:

- official migration guide review,
- breaking-change analysis,
- compatibility review,
- tests,
- explicit mention in the implementation plan,
- user approval where architecture/runtime behavior may change.

---

## 90.15 Lockfiles Are Mandatory

Commit and preserve deterministic lockfiles:

```text
pnpm-lock.yaml
uv.lock
Cargo.lock
```

Do not delete or regenerate lockfiles unnecessarily.

CI and production must resolve deterministic dependency versions.

---

## 90.16 Security Advisories

For new or upgraded dependencies, check for known critical security issues where practical.

Never knowingly introduce a version with a serious unresolved vulnerability when a supported fixed version exists.

Security fixes may justify an upgrade even when unrelated dependency upgrades are normally avoided.

---

## 90.17 Avoid Abandoned Packages

Before adopting a new library, consider:

- recent release activity,
- active maintenance,
- unresolved security issues,
- issue/PR activity,
- ecosystem maturity,
- compatibility,
- licence.

For foundational Buildora architecture, prefer mature, maintained, well-documented technologies.

Do not add a dependency for trivial functionality that can be safely implemented with a small amount of code.

---

## 90.18 Integration Versioning

When an external service supports explicit API versions, record the selected version where appropriate.

Examples:

```text
Stripe API version
IFC schema version
AI provider model/rate version
Embedding profile
Workflow schema version
```

Do not allow provider-default changes to silently change production behavior.

---

## 90.19 Record Version-Sensitive Decisions

If an implementation materially depends on a specific framework/API behavior, document it in the appropriate location:

```text
ADR
architecture document
integration README
code comment near non-obvious compatibility logic
```

Do not document obvious code.

Document assumptions that would matter during a future upgrade.

---

## 90.20 Documentation Version Awareness

When reading official documentation, ensure it applies to the version actually installed or planned.

Do not combine:

```text
v5 package
+
v3 documentation
```

If documentation has a version selector, use the correct version.

---

## 90.21 When Documentation and Existing Code Conflict

If official documentation conflicts with existing Buildora code:

Do not silently rewrite the architecture.

Determine whether:

1. existing code uses an older but still supported version,
2. existing code is incorrect,
3. the official documentation refers to a newer major version,
4. architecture documentation explicitly requires the existing behavior.

Then report the conflict.

If resolving it requires an architectural change or major upgrade, request approval first.

---

## 90.22 When Official Documentation Cannot Be Verified

If a task materially depends on an external API/package behavior and current official documentation cannot be verified:

Do not invent the API.

Do not guess package versions.

Do not present remembered syntax as verified current behavior.

Report clearly:

```text
Official documentation could not be verified.
This integration should not proceed until the current API/version
is confirmed.
```

This should not block unrelated version-independent work.

---

## 90.23 Implementation Start Checklist

Before implementation involving any external dependency or service, confirm:

```text
[ ] Relevant Buildora architecture docs read
[ ] AGENTS.md rules read
[ ] Current official documentation checked
[ ] Stable version confirmed
[ ] Deprecation status checked
[ ] Existing repository version checked
[ ] Breaking changes reviewed when relevant
[ ] Compatibility verified
[ ] Correct package manager selected
[ ] Lockfile impact understood
```

Only then begin implementation.

---

## 90.24 Agent Task Workflow Addition

Under the normal "Before Coding" workflow, include:

```text
For every external package, SDK, service, API, framework or
integration involved in the task, read its current official
documentation and verify the stable supported version/API before
implementation.
```

---

## 90.25 Review Before Completion

For every task that touches dependencies or integrations, check:

### Dependency / Integration Stability

- Current official documentation checked?
- Stable supported version used?
- Deprecated package/API avoided?
- Existing repository version respected?
- Lockfile preserved?
- No unrelated major upgrades?
- Integration follows the provider's current recommended pattern?
- Version-sensitive behavior documented where needed?
- Security/deprecation notices considered?

---

## 90.26 Completion Report

When a task introduces or changes a dependency/integration, report:

```text
Dependencies added/changed:
- package
- version
- reason

Official documentation verified:
- technology
- documentation/version checked

Deprecated APIs avoided/replaced:
- if applicable

Compatibility notes:
- if applicable
```

Do not claim "latest stable" unless it was actually verified.

---

## 90.27 Core Stability Rule

Buildora AI prioritizes stability over novelty.

The default decision order is:

```text
Supported
    ↓
Stable
    ↓
Maintained
    ↓
Well documented
    ↓
Compatible
    ↓
Necessary
    ↓
Then install/use it
```

Not:

```text
Newest
    ↓
Install immediately
```

---

## 90.28 Final Mandatory Rule

> **Before implementing any external framework, package, SDK, service, API, database extension, AI provider, cloud integration, CAD/BIM integration, or infrastructure technology, the agent MUST consult its current official documentation and verify the latest stable compatible approach. Deprecated, unsupported, experimental, or remembered legacy patterns must not be introduced into Buildora AI without explicit approval.**

When stability and novelty conflict:

> **Choose the stable, officially supported approach.**

---

# 91. Final Agent Contract

Every agent working on Buildora AI must preserve:

1. One canonical Building Model.
2. One authoritative persistence path.
3. 2D and 3D as representations, not independent truth, both fed by
   `packages/model-session` as a synchronized working copy — never a second
   authority.
4. IFC as interchange, not internal authority.
5. Deterministic geometry/QS/costing.
6. Typed AI tools and ChangeSets.
7. No direct LLM mutation of live geometry.
8. Exact financial arithmetic.
9. Auditable/versioned model and billing changes.
10. Strong tenant isolation.
11. Explicit database migrations.
12. Private user files.
13. Idempotent external-event processing.
14. Small, testable tasks.
15. No silent scope expansion.
16. No silent architecture replacement.
17. Managed services before premature infrastructure complexity.
18. Cost-aware AI/rendering.
19. Documentation and tests as part of implementation.
20. User approval for significant architectural changes.
21. Level-relative geometry with one authoritative level elevation (ADR-005).
22. Room boundaries from interior wall faces; area computed, never entered (ADR-006).
23. A door or window IS its opening; one QS deduction abstraction.
23a. FF&E are classed domain elements, never renderer state (ADR-019).
23b. Finish, assembly and element type are three distinct axes (ADR-020).
23c. Manual interior work never requires AI; both paths share one command set.
23d. Every user has a personal organization; there is **no user-owned project path** (ADR-021).
23e. Plans may be recurring or fixed-term; archive preserves data (ADR-021).
23f. Feature code reads entitlements, never plan names.
23g. **Never duplicate QS/BOQ/cost/model engines per customer segment.**
23h. **Gated capabilities are enforced server-side** before the work happens.
     Hiding a button is not access control (ADR-022).
23i. **Check order: feature flag → entitlement → credit reservation.** An
     unentitled request consumes no credit.
23j. **Feature flags are rollout; entitlements are commercial rights.** A flag
     must never grant paid access.
24. `uuid` primary keys; `public_id` never a foreign-key target (ADR-004).
25. Package dependencies point downward only; domain packages import no
    framework; no cycles (ADR-011).
26. No app, package or managed service before a real consumer exists (ADR-012).
27. The frozen decisions in `FINAL_SYSTEM_ARCHITECTURE.md` are not relitigated
    without a superseding ADR.

When uncertain:

> **Prefer the simplest implementation that preserves the approved architecture and leaves a clean path for the next phase.**
