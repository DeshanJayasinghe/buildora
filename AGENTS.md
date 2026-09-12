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

Important canonical references include, where present:

```text
docs/architecture/01_Master_Product_Reference.md
docs/architecture/Buildora_AI_Database_Architecture.md
docs/architecture/Buildora_AI_Billing_Architecture.md
docs/architecture/Buildora_AI_Deployment_Production_DevOps_Plan.md

docs/standards/Buildora_AI_Backend_Engineering_Standards.md
docs/standards/Buildora_AI_Frontend_Engineering_Standards.md
docs/standards/Buildora_AI_Codex_Implementation_Reference.md
docs/standards/Buildora_AI_Solo_Developer_AI_Assisted_Development_Guide.md

docs/planning/Buildora_AI_Phase_Wise_Development_Checklist.md
docs/planning/Buildora_AI_Sprintwise_Project_Plan.md

docs/examples/Buildora_AI_Sample_Backend_CRUD.md
docs/examples/Buildora_AI_Sample_Frontend_CRUD.md

docs/finance/Buildora_AI_Development_Production_Costing_Plan.md
docs/finance/Buildora_AI_Return_On_Investment_Plan.md
```

If a filename differs slightly, use `docs/README.md` and the nearest canonical document.

Do not create duplicate architecture documents merely because an existing file was not found immediately.

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

Expected high-level structure:

```text
apps/
├── web/
├── api/
├── ai-worker/
├── bim-worker/
└── render-worker/

packages/
├── design-system/
├── api-contracts/
├── domain/
├── building-model/
├── cad-2d/
├── engine-3d/
├── qs-engine/
├── cost-engine/
├── ai-tools/
└── units/

native/
└── geometry-wasm/

infrastructure/

docs/
```

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

Unit conversion belongs in:

```text
packages/units
```

or equivalent typed domain utilities.

Do not spread ad-hoc unit conversions across UI code.

---

# 11. Building Model Domain Purity

`packages/building-model` and other core domain packages must remain framework-independent.

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
server/database → authoritative persistent state

TanStack Query → client cache of server state where useful

Zustand → ephemeral editor/UI state
```

The canonical Building Model must not become a Zustand-only authority.

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

If legacy reference docs still contain `BuildWise`, do not mass-edit them during unrelated implementation tasks unless requested.

---

# 90. Final Agent Contract

Every agent working on Buildora AI must preserve:

1. One canonical Building Model.
2. One authoritative persistence path.
3. 2D and 3D as representations, not independent truth.
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

When uncertain:

> **Prefer the simplest implementation that preserves the approved architecture and leaves a clean path for the next phase.**
