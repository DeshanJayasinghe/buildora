
# Buildora AI — Sprintwise Project Plan

> **Product:** Buildora AI  
> **Type:** Multi-tenant SaaS application  
> **Development model:** One-person SaaS development assisted by Codex Plus, Claude Pro, and Antigravity Pro  
> **Primary architecture:** Next.js + TypeScript + NestJS + PostgreSQL + PixiJS + Three.js + AI Gateway
> *(Redis ~S27, Temporal ~S31, Python/FastAPI ~S31, pgvector ~S34 — all **gated** by ADR-012; not provisioned before their consumer exists.)*
> **Architecture baseline:** `docs/architecture/FINAL_SYSTEM_ARCHITECTURE.md` — ADR-001 … ADR-022
> **Planning style:** Dependency-first, vertical-slice delivery  
> **Recommended sprint length:** 1–2 focused development weeks per sprint, but completion is based on acceptance criteria rather than calendar time.

---

# 1. Purpose of This Plan

This document is the primary execution roadmap for Buildora AI.

It converts the full Buildora AI product vision into an ordered sprint plan covering:

- SaaS foundations
- multi-tenancy
- authentication
- projects
- subscriptions
- Building Model
- 2D CAD editor
- 3D workspace
- BIM/IFC
- materials
- assemblies
- quantity surveying
- BOQ
- costing
- AI Copilot
- automatic model routing
- upload-plan recognition
- document intelligence
- reporting
- billing
- launch
- production operations
- post-launch construction modules
- supplier workflows
- mobile/site capabilities
- scaling
- enterprise readiness

The plan deliberately **does not attempt to build every feature before launch**.

The product should reach commercial beta after the core value chain works:

```text
Create / Upload / Describe Building
        ↓
Building Model
        ↓
2D
        ↓
3D
        ↓
Materials
        ↓
Quantities
        ↓
BOQ
        ↓
Cost
        ↓
AI Copilot
        ↓
Reports
```

Advanced modules are scheduled after the commercial core is usable.

---

# 2. Sprint Operating Rules

Every sprint must follow these rules.

## 2.1 One sprint, one clear outcome

Do not define a sprint as:

```text
Improve 2D, AI, billing and reports.
```

Prefer:

```text
Implement reliable wall creation, selection, editing and persistence.
```

## 2.2 Every sprint must include

- implementation
- tests
- documentation update
- error states
- loading states where applicable
- permissions
- observability hooks where applicable
- architecture review
- regression verification

## 2.3 AI-assisted workflow

Recommended pattern:

```text
Claude
→ plan / challenge architecture

You
→ approve

Codex
→ implement

Codex
→ tests + pnpm verify

Claude
→ review implementation

Codex
→ fix findings

Antigravity
→ browser/regression test if UI affected

You
→ final review + merge
```

## 2.4 Definition of complete

A sprint is not complete because the UI “looks finished”.

A sprint is complete when:

- acceptance criteria pass
- tests pass
- state persists correctly
- no duplicate source of truth exists
- permissions are enforced
- architecture rules remain intact
- documentation is updated
- the branch can merge safely

---

# 3. Delivery Stages

Buildora AI is divided into six commercial stages.

| Stage | Sprints | Outcome |
|---|---:|---|
| **Stage 0** | **00** | **Architecture Lock — COMPLETE (2026-09-12)** |
| Stage A | 1–7 | SaaS + repository foundation |
| Stage B | 8–21 | Building Model + 2D + 3D |
| Stage C | 22–26 | Materials + QS + BOQ + Cost |
| Stage C+ | 26b | **Reports & exports — first sellable milestone** |
| Stage D | 27–34 | AI + Upload + BIM + Documents |
| Stage E | 35–38 | Billing + Beta + Commercial Launch |
| Stage F | 39–48 | Post-launch professional expansion |

## Two milestones, not one

| Milestone | Sprint | What it proves |
|---|---:|---|
| **Deterministic Model-to-Cost** | **26b** | Design → 2D → 3D → materials → QS → BOQ → cost → professional report. **Sellable.** No AI required. |
| Commercial paid beta | 38 | Full platform with AI, recognition, BIM, documents and billing |

Sprint numbering is **unchanged** from the original plan. Sprint 00 is prepended
and Sprint 26b is inserted; nothing is renumbered.

**AI generation must not be built before the deterministic core works.**

---

# 3.1 Contract Gates — Hard Blockers

Independent review (`CODEX_INDEPENDENT_REVIEW.md`) found three decisions that
freeze an *outcome* while leaving the *semantics that produce it* unspecified.
These are **not** open architecture questions — the decisions stand. They are
**implementation contracts that must be published before their owning sprint
ships**.

| Gate | Contract | Blocks | Why |
|---|---|---|---|
| **CG-1** | **Room/junction topology determinism** — tolerance model, junction classification (T/X/acute) and ownership (miter vs butt vs bevel), variable-thickness behaviour, open-boundary rule, transient invalid states, region matching, **split/merge/delete room identity** | **Sprint 15** (rooms), relied on from S09 | Two correct geometry libraries return different areas *and different IDs* from the same walls. ADR-006 fixes the principle, not the behaviour. Wrong here corrupts every finishes quantity downstream. |
| **CG-2** | **Session reconciliation determinism** — accepted-prefix processing, per-command preconditions, dependency/cascade metadata, rebase rules, invalid-suffix quarantine, conflict UX, **durable pending-queue persistence** | **Sprint 11–13** (persistence, optimistic editing) | ADR-008 says "reload and replay, **or** surface the conflict". That disjunction is product policy. If command 7 of 30 fails, 8–30 were computed from a state including it. |
| **CG-3** | **QS rule contract** — versioned declarative rules over a small versioned executable primitive set, both hashed; rounding/evaluation order; conformance fixtures. Plus complete `cost_estimates` lineage columns | **Sprint 23–26b** (any NRM2-branded quantity or issued estimate) | ADR-009's "rules are data" is qualified by its own admission that novel rules need code. Without the hybrid contract two engine versions interpret one ruleset hash differently. |
| **CG-4** | **MVP geometry envelope** — publish what the canonical model supports and **reject unsupported input explicitly** | **Sprint 09–16** | Wall is a straight centreline with scalar thickness; Roof is a polygon plus a plane. Curved walls, tapering thickness, vaulted ceilings and multi-plane roofs are **not** expressible. Silently segmenting them creates fake elements and fake junctions. |

**A sprint that depends on an open gate does not start.** Closing a gate is a
design deliverable — written, reviewed, and fixture-tested — not code.

Until the supported NRM2 work sections are audited with conformance fixtures,
Sprint 23 output is labelled **"Buildora QS — NRM2 subset"**, never
"NRM2-compliant".

---

# 4. High-Level Sprint Map

| Sprint | Focus | Commercial Stage |
|---:|---|---|
| **00** | **Architecture lock — COMPLETE** | **Foundation** |
| 01 | Repository & architecture bootstrap | Foundation |
| 02 | Local infrastructure & CI | Foundation |
| 03 | Design system & application shell | Foundation |
| 04 | Authentication | SaaS Core |
| 05 | Organisations & tenancy | SaaS Core |
| 06 | Projects & project dashboard | SaaS Core |
| 07 | Audit, feature flags & admin baseline | SaaS Core |
| 08 | Units & Building Model foundations | Core Engine |
| 09 | Levels, walls & command model | Core Engine |
| 10 | Doors, windows & openings | Core Engine |
| 11 | Versioning, undo/redo & persistence | Core Engine |
| 12 | 2D canvas foundation | 2D |
| 13 | Wall drawing & selection | 2D |
| 14 | Snapping, editing & dimensions | 2D |
| 15 | Doors/windows/rooms in 2D | 2D |
| 16 | Layers, floors & properties | 2D |
| 17 | 2D editor hardening | 2D |
| 18 | Three.js engine foundation | 3D |
| 19 | Building element 3D adapters | 3D |
| 20 | 3D workspace controls & inspector | 3D |
| 21 | 2D ↔ 3D synchronisation | 3D |
| **21b** | **3D walkthrough & saved views** | **3D** |
| 22 | Materials & assemblies **+ finish assignments** | QS Foundation |
| 23 | Quantity Surveying engine | QS |
| 24 | BOQ engine | QS |
| 25 | Cost engine & estimate levels | Cost |
| 26 | QS/BOQ/Cost UI + auditability | Cost |
| **26b** | **Reports & exports — FIRST SELLABLE MILESTONE** | **Cost** |
| **26c** | **FF&E catalogue & manual furnishing** | **Interior** |
| 27 | AI Gateway & model routing | AI |
| 28 | Read-only AI Copilot | AI |
| 29 | AI change proposals & approval | AI |
| 30 | AI value engineering & usage controls | AI |
| **30b** | **AI interior assistant — furnishing & style** | **AI** |
| 31 | Upload Plan pipeline | Import |
| 32 | Recognition confidence & verification | Import |
| 33 | IFC/BIM interoperability | BIM |
| 34 | Documents, RAG & project Q&A | Documents |
| 35 | Reports, exports & sharing | Commercial |
| 36 | Billing Release 1 — subscription, wallet, ledger, reserve/settle, **term model + archive entitlements** | Commercial |
| **36b** | **Billing Release 2 — top-ups, proration, reconciliation, admin UI** | **Commercial** |
| 37 | Beta hardening & onboarding | Beta |
| 38 | Production launch | Launch |
| 39 | Site/GIS/environment analysis | Growth |
| 40 | Schedule & cash flow | Growth |
| 41 | Procurement, suppliers & tender comparison | Growth |
| 42 | Actual cost, change orders & progress | Growth |
| 43 | Mobile/site companion | Growth |
| **43b** | **Home experience UX + fixed-term project passes** | **Growth** |
| 44 | Structural & MEP concept modules | Professional |
| 45 | Sustainability/compliance intelligence | Professional |
| 46 | Marketplace & professional leads | Growth |
| 47 | Enterprise collaboration & integrations | Enterprise |
| 48 | Scale, DR, performance & platform maturity | Enterprise |

---

# 4.1 Stage 0 — Architecture Lock

---

# Sprint 00 — Architecture Lock

> **Status: COMPLETE — 2026-09-12**

## Goal

Freeze the foundational architecture so that implementation agents never
redesign fundamental decisions, and so that every documentation route resolves
to a file that exists.

## Deliverables — all complete

```text
✓ docs/README.md created as the documentation router
✓ duplicated setup pack removed from docs/architecture/
✓ all documents renamed BuildWise → Buildora AI
✓ canonical architecture documents written:
    03_System_Architecture.md
    04_Monorepo_and_Module_Architecture.md
    05_Building_Model_Domain.md
    06_2D_CAD_Editor_Implementation.md
    07_3D_Engine_Implementation.md
✓ 22 ADRs accepted (ADR-001 … ADR-022)
    ADR-019/020 added for FF&E and finish assignment so the
    3D walkthrough/interior capability needs no later redesign
✓ docs/architecture/34_ADR_Index.md
✓ docs/architecture/adr/0000-template.md
✓ docs/architecture/FINAL_SYSTEM_ARCHITECTURE.md — frozen baseline
✓ AGENTS.md and CLAUDE.md updated to the frozen architecture
✓ ID strategy corrected in CRUD examples (uuid + public_id)
✓ packages/domain removed everywhere
```

## Architecture decisions locked

| Area | Decision |
|---|---|
| Identifiers | `uuid` PK + `public_id`; never an FK target (ADR-004) |
| Datum | Project FFL = 0; level-relative z; floor-to-floor derived (ADR-005) |
| Rooms | Interior-face boundary, persisted stable identity (ADR-006) |
| Openings | A door/window **is** its opening; one QS abstraction |
| Types | `element_types` separate from cost assemblies (ADR-007) |
| Editor state | Renderer-neutral `packages/model-session` (ADR-008) |
| QS standard | Versioned rulesets; NRM2 default (ADR-009) |
| AI pricing | Versioned rate cards behind logical aliases (ADR-010) |
| Packages | No `packages/domain`; no `packages/core` yet (ADR-011) |
| Infrastructure | Provisioning gates; nothing before a consumer (ADR-012) |
| Tenancy | Scoping + tests from S05; RLS gate before external beta (ADR-016) |
| Graphics | WebGL2 baseline; WebGPU deferred (ADR-017) |

## Exit gate — met

- [x] Every path referenced by `AGENTS.md` and `CLAUDE.md` exists
- [x] No duplicate setup pack under `docs/architecture/`
- [x] No `BuildWise` references in documentation
- [x] ID strategy identical everywhere
- [x] Datum, room, opening and model-session semantics identical everywhere
- [x] 22 ADRs accepted; index consistent
- [x] Architecture freeze statement published

---

# 5. Stage A — SaaS and Engineering Foundation

---

# Sprint 01 — Repository & Architecture Bootstrap

## Goal

Create a clean, controlled Buildora AI repository that all AI tools can work in safely.

## Deliverables

- GitHub private repository
- pnpm workspace
- Turborepo
- root TypeScript config
- `.editorconfig`
- `.gitignore`
- `.env.example`
- architecture pack added under:

```text
docs/architecture/
```

- `AGENTS.md`
- `CLAUDE.md`
- basic folder structure:

```text
apps/
packages/
native/
infrastructure/
docs/
```

## Initial folders

```text
apps/web
apps/api

packages/design-system
packages/model-session
packages/building-model
packages/units
packages/api-contracts

infrastructure/
```

## Engineering tasks

- pin Node major
- pin package manager
- configure workspace
- configure root scripts
- configure dependency rules
- configure Conventional Commit conventions
- create first ADR if setup decisions differ from existing architecture

## Acceptance criteria

- clean clone works
- `pnpm install` succeeds
- `pnpm build` executes
- agents can find architecture docs
- no application secrets committed

## Testing

- repository bootstrap validation
- CI dry run locally

## AI roles

Claude:
- review repo boundaries

Codex:
- scaffold repository

Antigravity:
- not required

---

# Sprint 02 — Local Infrastructure & CI

## Goal

Create a reproducible local development and validation environment.

> **Infrastructure gating (ADR-012).** Provision a service only when a real
> consumer exists. Sprints 01–26 run on **local Docker PostgreSQL + Neon free
> tier** only.
>
> | Service | Gate | Do not provision here |
> |---|---|---|
> | Redis | AI rate limiting (~S27) | ✗ |
> | Temporal | First durable workflow (~S31) | ✗ |
> | pgvector | RAG phase (~S34) | ✗ |
> | PostGIS | Site/GIS phase (~S39) | ✗ |
> | AWS production | First external beta (~S35) | ✗ |
>
> Any asynchronous need before Temporal is met by a `processing_jobs` table and
> a simple in-process worker, behind an interface stable enough that adopting
> Temporal later is an implementation change only.

## Deliverables

Docker Compose:

- PostgreSQL

*(Redis deferred to ~Sprint 27 — ADR-012.)*

Application tooling:

- GitHub Actions
- lint
- typecheck
- unit test runner
- build
- root `pnpm verify`

Suggested:

```text
pnpm lint
pnpm typecheck
pnpm test
pnpm build
pnpm verify
```

## Database

Initial connectivity only.

Do not create the complete future schema.

## API

Create:

```text
GET /health
```

## Web

Create:

```text
/health or basic app page
```

## CI

Run on:

- PR
- main branch push

## Acceptance criteria

```bash
docker compose up -d
pnpm dev
pnpm verify
```

all work from a fresh clone after environment setup.

---

# Sprint 03 — Design System & Application Shell

## Goal

Lock the Buildora AI visual system before page-by-page implementation.

## Deliverables

Design-system components:

- typography
- colours
- spacing
- buttons
- input
- select
- checkbox
- radio
- switch
- tabs
- cards
- table
- badge
- dialog
- drawer
- tooltip
- alert
- empty state
- loading state
- error state
- AI badge
- confidence badge
- CAD toolbar button
- property row

Storybook or equivalent showcase.

## Main application shell

Create:

```text
Sidebar
Top Header
Project Switcher placeholder
Notifications placeholder
User Menu placeholder
Main Content
AI Copilot entry placeholder
```

Navigation:

```text
Dashboard
Projects

Design
Construction
Project
AI Copilot
Settings
```

## Responsive baseline

Support:

- desktop
- laptop
- tablet shell

Full mobile CAD is not required.

## Acceptance criteria

All future screens can reuse the component library.

Do not allow feature pages to create unrelated styles.

---

# 6. Stage A2 — SaaS Core

---

# Sprint 04 — Authentication

## Goal

Implement secure user identity.

## Deliverables

Using the selected auth provider:

- sign up
- sign in
- sign out
- forgot/reset
- session handling
- protected routes
- API token verification
- local user mirror

## Database

Create:

```text
users
auth_provider_links
```

or equivalent mapping.

## Security

API must verify identity server-side.

Do not trust browser-supplied user IDs.

## Acceptance criteria

- unauthenticated users cannot access dashboard
- authenticated user receives local Buildora AI user
- API rejects invalid identity
- logout invalidates access correctly

---

# Sprint 05 — Organisations & Multi-Tenancy

> **ADR-021 addition — binding.** Signup must auto-create a **personal
> organization** (`kind = PERSONAL`) with the user as sole owner. There is **no
> user-owned project path**.
>
> This costs almost nothing now. Adding a second ownership path later — or
> migrating user-owned projects into organizations — is expensive and is the
> likeliest source of a cross-tenant leak, because every query, RLS policy and
> permission check would need a second branch.
>
> Acceptance: signup creates exactly one personal organization; every project
> resolves to exactly one organization; `kind` can flip `PERSONAL → TEAM` with no
> project row moving.

## Goal

Make Buildora AI a proper multi-tenant SaaS.

## Deliverables

- organisation creation
- organisation membership
- invitations
- organisation switcher
- roles

Initial security roles:

```text
owner
admin
member
viewer
```

## Database

```text
organizations
organization_members
invitations
```

## Tenant rules

Every project belongs to an organisation.

Every project query is organisation-scoped.

## Acceptance criteria

User from Organisation A cannot access Organisation B data using:

- UI
- API URL manipulation
- guessed IDs

Add explicit cross-tenant tests.

---

# Sprint 06 — Projects & Dashboard

> **ADR-021 addition.** Project status includes `archived` as a first-class state
> meaning **paid term ended, data preserved** — distinct from `deleting`/`deleted`.
> Add `archived_at`, `archive_reason`, `reactivated_at`.
>
> Archive *capability* is entitlement-driven and lands with billing (Sprint 36);
> the *state* must exist here so the lifecycle is not retrofitted later.

## Goal

Create the user's core SaaS workspace.

## Deliverables

Project CRUD:

- create
- rename
- archive
- restore
- delete

Project metadata:

- name
- location
- unit system
- currency
- project type
- status

Dashboard:

- recent projects
- active projects
- create project
- estimated portfolio cost placeholder
- pending AI actions placeholder
- recent documents placeholder
- recent activity
- quick actions

## Create Project entry screen

Options:

- Generate with AI
- Upload Existing Plan
- Draw Manually
- Import BIM

## Acceptance criteria

A user can create and reopen a project reliably.

---

# Sprint 07 — Audit, Feature Flags & Admin Baseline

## Goal

Add operational foundations before domain complexity grows.

## Deliverables

Audit events:

```text
user.login
project.created
project.updated
project.archived
organization.member_added
```

Feature flags — **rollout only, not entitlements** (ADR-022):

```text
AI_CONCEPT_GENERATION
PLAN_RECOGNITION
IFC_IMPORT
PHOTOREAL_RENDERING
```

> **The distinction must be established here, before either system grows.**
>
> | | Feature flags | Entitlements (Sprint 36) |
> |---|---|---|
> | Question | "Is this code path enabled?" | "Is this customer allowed?" |
> | Purpose | Rollout, canary, kill switch | Commercial rights |
> | Scope | Global or cohort | Per organization |
>
> A flag must **never** grant paid access. Check order is always
> **flag → entitlement → credit**.
>
> `PRO_QS` was listed as a flag; it is a **commercial entitlement**
> (`feature.qs.advanced`) and moves to the entitlement catalogue.

Admin baseline:

- users
- organisations
- projects
- basic system metrics

## Acceptance criteria

Every sensitive project mutation creates an auditable event.

Feature flags can disable unfinished modules without deployment.

---

# 7. Stage B — Building Model

---

# Sprint 08 — Units & Building Model Foundations

## Goal

Create the foundation of the Buildora AI domain.

## Critical rule

Canonical geometry unit:

```text
millimetres
```

## Deliverables

`@buildora/units`:

- LengthMm
- AreaMm2 or safe conversion representation
- VolumeMm3 or domain conversion helpers
- angle types
- conversion helpers

`@buildora/building-model`:

- ProjectModel
- Building
- Level (`elevationMm` — the **single** authoritative vertical fact, ADR-005)
- element IDs (UUID, ADR-004)
- element base types

**`element_class` ships here** (ADR-019):

```text
BUILDING | SPACE | FFE
```

FF&E is not built until Sprint 26c, but the **column and the class discipline
belong in the first model migration**. Adding a class after elements exist means
reclassifying every row and re-auditing every quantity query.

Also from the first migration: `ffe_asset_version_id` (nullable),
`element_type_id` (nullable), and the `source_kind` / `source_ref` /
`confidence` / `verification_status` provenance set.

> **CG-4 — MVP geometry envelope.** Publish what the model supports before wall
> geometry is relied upon: straight-centreline walls with scalar thickness,
> polygon-plus-plane roofs. **Curved walls, thickness varying along a wall,
> vaulted ceilings and multi-plane roofs are not expressible** — reject them
> explicitly rather than silently segmenting them into fake elements.

## Important

No React.

No PixiJS.

No Three.js.

No OpenAI.

## Acceptance criteria

Domain package can run completely headlessly in tests.

---

# Sprint 09 — Levels, Walls & Command Model

## Goal

Create the first real building geometry domain.

## Deliverables

Entities:

- Level
- Wall

Commands:

```text
CreateLevel
RenameLevel
CreateWall
UpdateWall
DeleteWall
```

Wall properties:

- start
- end
- height
- thickness
- type
- assembly ID placeholder

## Validation

- zero-length wall rejected
- invalid thickness rejected
- wall assigned to valid level
- stable IDs

## Acceptance criteria

Wall command lifecycle is deterministic and tested.

---

# Sprint 10 — Doors, Windows & Openings

## Goal

Add openings as first-class building elements.

## Deliverables

- Door
- Window
- Opening relationships
- host Wall reference
- placement offset
- width
- height
- sill where relevant

Commands:

```text
CreateDoor
UpdateDoor
DeleteDoor

CreateWindow
UpdateWindow
DeleteWindow
```

## Validation

- opening must belong to valid wall
- dimensions valid
- position remains inside wall
- invalid overlaps detected or clearly flagged

## Acceptance criteria

Doors/windows survive serialization and rehydration.

---

# Sprint 11 — Versioning, Undo/Redo & Persistence

> ## ⚠ Gate CG-2 must be closed before this sprint ships
>
> Define and fixture-test, **before** optimistic persistence exists:
> accepted-prefix processing · per-command preconditions · dependency/cascade
> metadata · deterministic rebase rules · invalid-suffix quarantine · conflict UX ·
> **durable local persistence of the pending queue**.
>
> **Undo is a new command against the current `baseVersion`, with preview —
> never a blind inverse payload replayed at a later version.**

## Goal

Make model mutations safe and auditable.

## Deliverables

- model version number
- command history
- undo
- redo
- optimistic version check
- model serialization
- persistence
- reload

## Database

Store:

- elements
- model version
- commands / audit references

Exact persistence strategy follows architecture docs.

## Acceptance criteria

Create walls → save → close → reopen → same model.

Undo/redo produces deterministic model state.

---

# 8. Stage B2 — 2D CAD

---

# Sprint 12 — 2D Canvas Foundation

## Goal

Create the first real professional editor surface.

## Deliverables

PixiJS editor package:

- canvas
- camera
- model-to-screen transform
- screen-to-model transform
- pan
- zoom
- grid
- cursor coordinates
- render static walls

## Important

Model coordinates remain millimetres.

Screen coordinates remain pixels.

## Acceptance criteria

A known wall fixture appears at correct relative scale.

---

# Sprint 13 — Wall Drawing & Selection

## Goal

Allow users to create and select walls.

## Deliverables

Wall Tool:

- click start
- move preview
- click end
- command submission

Selection:

- click wall
- selected styling
- selection ID
- properties panel

Delete:

- keyboard/button
- domain command

## Acceptance criteria

User can create, select and delete wall without duplicate editor state.

---

# Sprint 14 — Snapping, Editing & Dimensions

## Goal

Make wall authoring useful rather than decorative.

## Deliverables

Initial snapping:

- endpoint
- grid
- orthogonal

Editing:

- move endpoint
- move wall
- exact numeric length
- exact position

Dimensions:

- wall length display
- manual measurement

## Acceptance criteria

Snapped points create exact model coordinates, not visually approximate coordinates.

---

# Sprint 15 — Doors, Windows & Rooms in 2D

> ## ⚠ Gate CG-1 must be closed before rooms are persisted
>
> Publish the deterministic topology contract: numeric tolerance model · junction
> classification (T/X/acute) and ownership (miter vs butt vs bevel) ·
> variable-thickness behaviour · open-boundary / room-separator concept ·
> transient invalid-state policy · region matching · **split/merge/delete room
> identity policy**.
>
> Golden fixtures must include unequal wall thickness, acute/T/X junctions, small
> gaps, nested loops, overlap, delete, split and merge.

## Goal

Complete the first usable residential plan editor slice.

## Deliverables

Door tool:

- wall hosting
- width
- swing visualization

Window tool:

- wall hosting
- width
- sill/height properties

Room (ADR-006):

- wall centreline graph → **connectivity** (which regions are enclosed)
- junction resolution
- **interior wall face** derivation
- boundary from interior faces — **not** from centrelines
- persisted room with **stable ID** + `BOUNDS` relationships
- `user_defined` override flag — never silently overwritten
- **area computed from the boundary**; a user-entered area is never authoritative
- room name, number, label

**Golden test — required:**

```text
5000 × 4000 mm room, 200 mm walls
centreline area = 20.00 m²    ← WRONG
interior faces  = 18.24 m²    ← REQUIRED
```

Room detection lives in `packages/building-model`, not the editor.

## Acceptance criteria

A four-wall room with one door and one window renders correctly.

---

# Sprint 16 — Floors, Layers & Property Inspector

## Goal

Support multi-level plan editing and professional controls.

## Deliverables

- floor selector
- add level
- rename level
- level elevation
- layer/object visibility
- hide/show
- lock/unlock
- property inspector
- exact values

## Acceptance criteria

Elements on other floors cannot accidentally mutate the active floor unless explicitly selected.

---

# Sprint 17 — 2D Editor Hardening

## Goal

Stabilise the first serious editor before 3D integration.

## Deliverables

- keyboard shortcuts
- undo/redo UX
- context menus
- loading project state
- error recovery
- selection edge cases
- large fixture performance
- editor empty state
- autosave strategy
- dirty state

## Testing

- fixtures
- keyboard
- snapping
- persistence
- version conflicts
- regression screenshots where practical

## Exit gate

Do not proceed to deep 3D work until a simple house can be drawn reliably.

---

# 9. Stage B3 — 3D

---

# Sprint 18 — Three.js Engine Foundation

## Goal

Create a rendering engine independent of UI/domain concerns.

## Deliverables

- Three.js scene
- renderer
- camera
- resize
- lighting
- fit-to-model
- domain-ID mapping
- first wall mesh adapter

## Unit mapping

Document scene mapping such as:

```text
1000 mm = 1 world unit
```

while domain stays in mm.

## Acceptance criteria

Known wall fixture produces expected mesh dimensions.

---

# Sprint 19 — Building Element 3D Adapters

## Goal

Render the first complete simple house.

## Deliverables

3D adapters for:

- wall
- slab
- door
- window
- floor/level grouping

Optional first-pass opening approach may be simplified, but model semantics must remain correct.

## Acceptance criteria

The 2D test fixture appears recognisably and dimensionally correctly in 3D.

---

# Sprint 20 — 3D Workspace Controls & Inspector

## Goal

Deliver the designed 3D workspace.

## Deliverables

- orbit
- pan
- zoom
- fit model
- top view
- isometric
- walkthrough
- object selection
- property inspector
- floor isolation
- visibility
- section cut baseline
- daylight mode baseline
- material preview

## Acceptance criteria

Clicking a 3D wall opens the same domain Wall ID and properties as 2D.

---

# Sprint 21 — 2D ↔ 3D Synchronisation

## Goal

Prove Buildora AI's most important architectural promise.

## Deliverables

Change:

```text
Wall in 2D
```

updates:

```text
Building Model
→ 2D
→ 3D
```

Change through supported 3D property control updates same model.

## Required regression

```text
create
save
reload
edit
undo
redo
switch 2D/3D
```

## Exit gate

The application now has a stable digital building core.

---

# Sprint 21b — 3D Walkthrough & Saved Views

> **Scope split from Sprint 20.** Walkthrough was one bullet among fifteen there.
> It is a distinct capability — first-person camera, collision, eye height — and
> deserves its own sprint (`11_3D_Walkthrough_Interior_and_FFE.md` §3).

## Goal

Let a user walk through their building. **Zero AI.**

## Deliverables

- first-person camera (W/S/A/D + mouse look, Shift to move faster, Esc to exit)
- **wall collision** — users cannot walk through walls
- floor detection and stair traversal
- configurable eye height (default 1600–1700 mm; presets later)
- `saved_views` — camera, visibility, section planes, walkthrough position
- register `feature.walkthrough` in the entitlement catalogue (ADR-022) with a
  server check site on 3D session authorization
- day/night lighting presets

## Architecture constraints

- The walkthrough controller is **renderer state**. Camera position, movement and
  collision volumes are **never persisted** except inside a saved view.
- Saved views record the `model_version` they were captured against so a stale
  view can be **flagged**, never so it can override the model.
- Collision is derived from canonical geometry, never hand-authored.

## Acceptance criteria

- A user can walk through the Sprint 21 fixture without passing through walls
- A saved view restores camera, visibility and section state exactly
- Leaving and re-entering walkthrough loses no model state
- **No LLM call occurs at any point**

---

# 10. Stage C — Materials, QS, BOQ and Cost

---

# Sprint 22 — Materials & Assemblies

## Goal

Connect geometry to construction meaning.

## Deliverables

Material domain:

- category
- unit
- density optional
- manufacturer optional
- regional rate reference

Assembly domain:

Example:

```text
EXT-BLOCK-200
```

Assembly items:

- blocks
- mortar
- plaster
- render
- primer
- paint
- labour placeholder
- waste rules

## Finish assignments (ADR-020)

Finish is a **third identity axis**, separate from type and assembly:

```text
element_type_id       WHAT IT IS
assembly_version_id   WHAT IT'S BUILT OF
finish_assignment     WHAT IT LOOKS LIKE
```

- `finish_assignments` table — **surface-scoped** (`INTERIOR_A`, `INTERIOR_B`,
  `EXTERIOR`, `TOP`, `BOTTOM`, `SOFFIT`, `ALL`), optional `room_id`
- `SET_ELEMENT_FINISH` command
- register `feature.interior.finishes` (ADR-022) with a server check site on that
  command guard
- materials carry **both** visual properties (colour, texture, roughness) **and**
  construction properties (unit, coverage rate, coats, waste, labour, rate ref)

> **Open contract — close before the finish editor ships:** the per-element
> **surface taxonomy** (which named surfaces each element type exposes, and how a
> partial-coverage finish declares its area).

**Repainting a wall creates a finish assignment, never a new assembly version.**

## UI

- material library
- assembly selector
- assign assembly to wall
- **finish picker per surface**
- material/finish inspector

## Acceptance criteria

A wall can reference an assembly without embedding duplicated material calculations in the wall object.

---

# Sprint 23 — Quantity Surveying Engine

> ## ⚠ Gate CG-3 must be closed before any NRM2-branded quantity
>
> Publish the **hybrid rule contract**: validated declarative rule documents over
> a small, versioned executable primitive set — both hashed — with defined
> rounding and evaluation order, and conformance fixtures.
>
> Until the supported NRM2 work sections are audited, label output
> **"Buildora QS — NRM2 subset"**, never "NRM2-compliant".

## Goal

Implement deterministic quantity calculations.

## Deliverables

QS functions:

- count
- length
- area
- volume

Initial supported elements:

- walls
- openings
- slabs

Initial rules:

```text
gross wall area
opening area
net wall area
floor area
wall volume
```

Assembly quantities:

```text
blocks
cement
sand
plaster
paint
```

## Golden tests

Example:

```text
5000 × 2700 wall
1000 × 2100 opening
= 11.4m² net
```

## Element class filtering — mandatory (ADR-019)

```sql
WHERE element_class = 'BUILDING' AND deleted_model_version IS NULL
```

FF&E is **excluded from every construction quantity**. Rooms
(`element_class = 'SPACE'`) contribute **finish** quantities, not fabric.

A missed filter puts furniture in a wall area. This has a mandatory test.

## Acceptance criteria

No LLM participates in authoritative quantity calculation.

**Test: FF&E elements contribute zero to every construction quantity.**

---

# Sprint 24 — BOQ Engine

## Goal

Convert quantity results into structured commercial output.

## Deliverables

BOQ hierarchy:

```text
Section
Trade
Element
Item
```

Fields:

- item number
- description
- unit
- quantity
- source elements
- revision
- rate placeholders

## UI

- BOQ table
- grouped rows
- search
- filters
- source link

## Acceptance criteria

Every BOQ quantity can trace back to Building Model elements.

---

# Sprint 25 — Cost Engine & Estimate Levels

## Goal

Calculate reliable conceptual/elemental costs.

## Deliverables

Cost components:

- material
- labour
- plant
- subcontract
- waste
- transport
- overheads
- profit
- tax
- contingency

Estimate levels:

- concept
- elemental
- detailed placeholder
- tender placeholder

Money:

- currency-aware
- decimal-safe

## Rate provider

Start with:

```text
StaticRateProvider
```

## Acceptance criteria

Same fixture + same rates = exactly same cost result.

---

# Sprint 26 — QS / BOQ / Cost UI + Auditability

## Goal

Deliver the first real commercial differentiator.

## Screens

- Quantities
- BOQ
- Cost Estimate
- Material Breakdown
- Cost Analysis

## Features

- cost by trade
- cost by element
- cost by material
- cost per m²
- quantity trace
- rate source
- waste assumption
- user rate override
- version reference

## Change propagation

Changing a wall must invalidate/recalculate:

```text
Quantity
BOQ
Cost
```

## Exit gate

Buildora AI now provides:

```text
Design → 3D → Quantity → BOQ → Cost
```

without AI.

---

# 11. Stage D — AI

---

# Sprint 26b — Reports & Exports — FIRST SELLABLE MILESTONE

> **Moved earlier** from Sprint 35. Professional users judge the product by its
> PDF/XLSX output, and reports are the natural paywall boundary. Shipping them
> here makes the deterministic core demonstrable and sellable ~10 sprints sooner.

## Goal

Complete the deterministic Model-to-Cost product so it can be demonstrated and
sold without any AI capability.

## Deliverables

- BOQ export (XLSX)
- Cost estimate report (PDF)
- Quantity takeoff report with element-level traceability
- Report artifacts stored in object storage with metadata
- `cost_estimates` carries its **complete** lineage: `quantity_run_id`,
  `cost_assumption_set_id/version`, `qs_rule_set_version`, `rate_book_version_id`,
  `boq_version_id`, `model_version`, `engine_version` (CG-3)
- Every report pins its full lineage (ADR-015): model version, quantity run,
  BOQ version, rate book version, engine versions, assumption set

## Acceptance criteria

- A report recalculated from its pinned versions is **numerically identical**
  (byte-identical PDFs are not guaranteed — template, fonts and locale affect
  bytes; **issued reports are preserved as immutable checksummed artifacts**)
- A report never recomputes historical cost with current rates
- Quantities in a report expand to their source elements
- The full flow works end to end with **no AI involved**:

```text
create project → level → walls → room → door → window
  → 3D → assign assemblies → quantities → BOQ → cost → PDF/XLSX
```

## Why this is the milestone

This is the first point at which Buildora AI does something no design-first
competitor does simply, and no construction-first competitor does from a
model-driven design. It is sellable on its own.

---

# Sprint 26c — FF&E Catalogue & Manual Furnishing

> **ADR-019.** Furniture is a **domain element**, never a Three.js object held in
> the scene. This sprint is **post-sellable-milestone** — it follows 26b, not
> precedes it.

## Goal

Place, move and cost furniture. **Zero AI.**

## Deliverables

**Catalogue**
- `ffe_assets` / `ffe_asset_versions` (tenant and global)
- GLB/GLTF assets in object storage with `storage_objects` metadata
- category browse, search, thumbnails

**Placement**
- drag from catalogue into a room
- `PLACE_FURNITURE` · `MOVE_FURNITURE` · `ROTATE_FURNITURE` · `REMOVE_FURNITURE`
- snap to floor, snap against wall, snap to room centre
- prevent placement outside a room, intersecting a wall, or blocking a door swing
- configurable circulation clearance

**Cost**
- register `feature.home.furnishing` (ADR-022) with a server check site on the
  `PLACE_FURNITURE` command guard
- FF&E as its own BOQ section (`boq_versions.include_ffe`)
- priced from `ffe_asset_versions.unit_cost`, never from the mesh

**Rendering**
- lazy load by `ffe_asset_version_id`, cache, **explicit disposal**
- placeholder while an asset loads — never block the frame

## Architecture constraints

- FF&E elements carry `element_class = FFE` and are **excluded from every
  construction quantity** and from room topology
- Instances reference an asset **version**, so catalogue updates never alter a
  placed instance or an issued estimate
- FF&E commands share the **same** version history and undo stack as wall edits

## Acceptance criteria

- A sofa placed, moved and deleted appears in model version history and undoes correctly
- **Test: FF&E contributes zero to wall, floor and volume quantities**
- A construction-only estimate excludes FF&E; `include_ffe` adds it as its own section
- A catalogue price change does not alter a previously issued estimate
- Reload restores every placement exactly
- **No LLM call occurs at any point**

---

# Sprint 27 — AI Gateway & Automatic Model Routing

## Goal

Create a controlled, provider-independent AI layer.

## Deliverables

AI Gateway:

- provider interface
- OpenAI provider
- model aliases
- request logging
- usage logging
- error normalization
- timeout
- retry
- cost metadata
- routing

Aliases:

```text
FAST_MODEL
BALANCED_MODEL
ADVANCED_MODEL
EXPERT_MODEL
```

Routing inputs:

- task
- complexity
- context
- risk
- affected objects
- credit budget

## Acceptance criteria

No application module imports AI provider SDK directly except the provider adapter.

---

# Sprint 28 — Read-Only AI Copilot

## Goal

Make AI useful without allowing model mutation yet.

## Tools

```text
get_project_summary
get_room
get_element
get_quantities
get_boq
get_cost
```

## UI

Copilot side panel available from:

- 2D
- 3D
- quantities
- BOQ
- cost

## Example

```text
Why is concrete cost high?
```

AI retrieves real project data and explains it.

## Acceptance criteria

AI cannot invent authoritative quantities or rates when a tool result exists.

---

# Sprint 29 — AI Change Proposals & Approval

## Goal

Allow AI to propose design actions safely.

## First action

```text
propose_move_wall
```

Flow:

```text
AI
→ ChangeSet
→ validation
→ draft model
→ quantity impact
→ cost impact
→ preview
→ user approval
→ commit
```

## Required protections

- stale version detection
- permission check
- geometry validation
- undo support
- audit log

## Acceptance criteria

AI never directly writes authoritative geometry.

---

# Sprint 30 — AI Value Engineering & Usage Controls

## Goal

Introduce high-value AI optimisation while protecting SaaS margins.

## Features

AI can answer:

```text
Reduce cost to £230,000.
```

System:

- identifies high-cost elements
- proposes material/design alternatives
- calculates deterministic savings
- previews impact

## Usage

Add:

- AI usage ledger
- design credit accounting
- per-user/project cost reporting
- routing metrics
- escalation metrics

## Admin

AI operations dashboard baseline.

---

# Sprint 30b — AI Interior Assistant

> **Requires Sprints 22, 26c and 27.** AI furnishing cannot precede deterministic
> furnishing — there would be nothing to validate the proposal against.

## Goal

Layer AI on top of the deterministic interior system.

## Deliverables

Typed, project-scoped tools:

```text
get_room_geometry · get_room_openings · get_room_furniture
search_furniture_catalogue · search_material_catalogue
validate_furniture_layout · calculate_furnishing_cost
propose_furniture_placement · propose_finish_change
```

Capabilities:
- *"Furnish this living room in a modern style under £4,000."*
- *"Make this room warmer."* → finish proposals
- *"Find a cheaper floor that looks similar."* → recommendation + **deterministic** cost

## Architecture constraints

- AI returns a **typed ChangeSet**. It never writes `building_elements`,
  `ffe_instances` or `finish_assignments`.
- Every proposal passes deterministic validation: bounds, collision, clearance,
  **then cost impact** — before preview.
- `baseVersion` is **re-checked at approval**, not only at proposal time.
- Commits use the **same** `PLACE_FURNITURE` / `SET_ELEMENT_FINISH` commands as
  manual placement. No second write path.
- The **recommendation** may be AI-generated; the **cost difference is always
  computed by the cost engine**.

## Acceptance criteria

- An AI furnishing proposal shows affected elements and cost delta before commit
- Rejecting a proposal changes nothing
- A proposal approved against a stale version returns `MODEL_VERSION_CONFLICT`
- AI has **no database write path** to model tables (privilege test, ADR-013)

---

# 12. Stage D2 — Upload, Recognition, BIM and Documents

---

# Sprint 31 — Upload Plan Pipeline

## Goal

Accept real customer plan files safely.

## Initial formats

- PDF
- PNG
- JPG

Optional vector PDF handling where feasible.

## Flow

```text
upload
→ object storage/local adapter
→ file record
→ workflow
→ extraction
→ recognition
```

## Security

- file type validation
- file size limit
- untrusted-input treatment
- signed/private storage

## Temporal

Introduce durable workflow if not already active.

## Acceptance criteria

Failed processing can retry/resume without requiring re-upload where possible.

---

# Sprint 32 — Recognition Confidence & Verification

## Goal

Convert uploaded drawings into user-verifiable Building Model drafts.

## Detect initial subset

- walls
- doors
- windows
- rooms
- dimensions/scale where possible

## Confidence

Example:

```text
Wall 98%
Window 76%
Door 93%
```

## UI

- overlay
- low-confidence list
- confirm
- reject
- edit
- redraw
- convert

## Acceptance criteria

Recognition output never becomes authoritative without validation/conversion workflow.

---

# Sprint 33 — IFC / BIM Interoperability

## Goal

Begin professional BIM support.

## Browser

- view IFC
- select IFC element
- inspect properties

## Server

IfcOpenShell worker.

## Initial mapping

- IfcBuildingStorey
- IfcWall
- IfcDoor
- IfcWindow
- IfcSlab
- IfcSpace

## Flow

```text
IFC
→ parse
→ review
→ map supported elements
→ Buildora AI Model
```

## Export

Basic supported IFC export can be a stretch goal.

## Acceptance criteria

IFC remains an interchange format, not Buildora AI's internal source of truth.

---

# Sprint 34 — Documents, RAG & Project Q&A

## Goal

Allow the Copilot to answer questions from real project documentation.

## Documents

- upload
- revision
- metadata
- extracted text
- chunking
- embedding
- retrieval

## pgvector

Enable.

## Search filters

Must scope:

- tenant
- project
- permission

## AI examples

```text
What concrete grade is specified?
Which revision changed the drainage?
```

## Provenance

Responses must reference:

- document
- revision
- page/section where possible

## Acceptance criteria

Project documents are treated as untrusted content and cannot override AI system/tool policy.

---

# 13. Stage E — Commercial SaaS

---

# Sprint 35 — Reports, Exports & Sharing

## Goal

Make output useful outside Buildora AI.

## Reports

- project summary
- quantities
- BOQ
- cost estimate
- material schedule
- value engineering
- revision comparison

## Exports

- PDF
- XLSX
- CSV

## Project sharing

- view-only link or invited viewer
- secure permissions
- expiry where applicable

## Acceptance criteria

Reports are generated from a specific model/BOQ/cost version so they are reproducible.

---

# Sprint 36 — Billing Release 1

> **Scope reduced** (ADR-014). Release 1 covers subscription, entitlements,
> wallet, ledger and reserve/settle. Top-ups, proration, reconciliation and the
> admin UI move to Sprint 36b.
>
> **Release 1 scope:** plan catalog, entitlements, Stripe Checkout subscription,
> webhook inbox, wallet + ledger + monthly grants, reserve/settle/release,
> provider usage records, customer usage UI, cancel/reactivate. Ship with a
> single paid plan.
>
> **ADR-021 additions (Release 1):**
> - `term_model` (`RECURRING | FIXED_TERM`), `term_ends_at`, `auto_renew` on
>   `billing_subscriptions`.
> - **Archive entitlement set** — read-only capabilities for an expired term.
> - Term expiry moves the project to `archived`; **no cancellation event** is
>   recorded, and analytics must not count it as churn.
> - Reactivation restores full capability against the existing model.
>
> Fixed-term (project pass) products themselves may ship in Release 2 — but the
> **schema and the archive entitlement set belong in Release 1**, because
> retrofitting the term model after subscriptions exist is a migration.

## Goal

Turn Buildora AI into a revenue-generating SaaS.

## Stripe

Implement:

- customer
- subscription
- webhook
- internal subscription state
- plan mapping

## Initial plans

```text
Free
Home
Pro
Business
```

## Pro baseline

Conceptually:

```text
$49/month
30 AI Design Credits
20 Render Credits
```

Keep all values configurable.

## Credit ledger

Implement append-style ledger.

Support:

```text
reserve
settle
release
adjust
```

## Feature entitlement & runtime gating (ADR-022)

**Catalogue — runtime-editable by a platform admin, no deployment:**

```text
feature_definitions          code · name · category · value_type
                             default_value · is_active
plan_entitlement_overrides   per-organization, IMMEDIATE, with reason
```

**Resolution order:** enterprise override → organization override →
add-on/promotion → plan version → default.

**Enforcement contract:**

```text
SERVER decides   every gated capability checked in apps/api BEFORE the work
                 denial → FEATURE_NOT_ENTITLED (402/403) with the code
CLIENT reflects  hide / disable / badge — usability only, never security
```

Check order: **feature flag → entitlement → credit reservation.** An unentitled
request consumes **no** credit.

**Admin screens:**

- feature catalogue (register/retire; **ORPHANED** badge where no check site exists)
- plan editor, with an explicit warning that publishing affects **new subscribers
  only** — existing subscribers keep their version (ADR-021 grandfathering)
- organization view showing each entitlement's **resolved source**
  (override / add-on / plan / default)
- audit of every entitlement change, with reason

Example gated capabilities:

- project limits · AI credits · exports · IFC · professional QS
- `feature.walkthrough` · `feature.home.furnishing` ·
  `feature.interior.finishes` · `feature.ai.interior`

## Acceptance criteria

- Client cannot unlock paid capabilities by modifying browser state
- **A direct API call bypassing the UI is refused for an unentitled organization**
- An admin override grants access **immediately**, with no deployment
- Publishing a plan version does **not** alter existing subscribers
- An unentitled request consumes **no** credit
- A globally disabled feature flag overrides an entitlement that grants it
- Every entitlement change writes an audit event with a reason

---

# Sprint 36b — Billing Release 2

> **Split from Sprint 36** (ADR-014). Compressing the full billing scope into one
> sprint risks under-tested financial code — the least forgiving code in the
> product.

## Goal

Complete commercial billing after Release 1's core is proven.

## Deliverables

- Credit top-up purchase
- Upgrade / downgrade with proration policy
- Payment-failure grace behaviour
- Reconciliation jobs (subscription, invoice, credit grant, top-up)
- Admin billing UI and margin analytics
- Credit expiry job

## Acceptance criteria

- Duplicate webhook delivery grants credits exactly once
- Parallel reservations never drive a wallet negative
- A failed AI job releases its full reservation
- An escalated AI job charges the customer once while recording every provider call
- Reconciliation detects and reports drift against Stripe

---

# Sprint 37 — Beta Hardening & Onboarding

## Goal

Prepare Buildora AI for real external users.

## User onboarding

- welcome
- create first project
- example/demo project
- empty states
- guided onboarding
- help/tooltips
- error recovery

## Product analytics

Events:

- signup
- project created
- 2D opened
- 3D opened
- quantity calculated
- BOQ generated
- AI used
- subscription started

## Security

- tenant penetration tests
- permissions review
- rate limiting
- signed storage URLs
- webhook replay/idempotency
- audit checks

## Performance

Test representative residential fixture.

## Acceptance criteria

A new user can complete the core workflow without developer assistance.

---

# Sprint 38 — Production Launch

## Goal

Release the first paid commercial Buildora AI version.

## Production setup

- production DB
- backups
- production Redis
- object storage
- production auth
- Stripe live
- AI production project/key
- Temporal production
- monitoring
- Sentry
- OpenTelemetry
- alerts

## Launch scope

Core:

```text
Projects
2D
3D
Materials
Quantities
BOQ
Cost
AI Copilot
Upload Plans
Basic IFC
Documents
Reports
Billing
```

## Operational requirements

- backup verified
- restore procedure
- AI spend alert
- DB alert
- billing webhook alert
- error monitoring
- support email
- privacy/terms
- basic status/incident procedure

## Release gate

Do not launch paid subscriptions if:

- cross-tenant access test fails
- financial calculations fail
- model save/reload fails
- billing webhooks are unreliable
- backups are unverified

---

# 14. Stage F — Post-Launch Growth

These sprints should be prioritised based on actual customer feedback.

They represent the full product vision, not a commitment to build every item immediately.

---

# Sprint 39 — Site / GIS / Environmental Analysis

## Goal

Add site context to design decisions.

## Features

- address/geolocation
- plot boundary
- orientation
- PostGIS storage
- sun path
- basic shadow analysis
- site metrics

Later:

- terrain
- slope
- solar
- wind
- noise

## AI

Copilot can explain site metrics using deterministic analysis tools.

---

# Sprint 40 — Construction Schedule & Cash Flow

## Goal

Connect scope/cost to project time.

## Schedule

- tasks
- dependencies
- duration
- milestones
- responsible party
- Gantt

Generate initial schedule from:

- BOQ
- assemblies
- productivity assumptions

## Cash flow

- monthly planned spend
- cumulative spend
- cash-flow chart

## Acceptance criteria

Schedule calculations remain separate from LLM reasoning.

---

# Sprint 41 — Procurement, Suppliers & Tender Comparison

## Goal

Extend BOQ into buying and contracting workflows.

## Features

- RFQ
- supplier records
- quote upload
- quote comparison
- supplier selection
- purchase order
- delivery status

## Tender analysis

AI can identify:

- missing lines
- exclusions
- unusually high rates
- inconsistent allowances

using deterministic data extraction/comparison.

---

# Sprint 42 — Actual Cost, Change Orders & Progress

## Goal

Move from pre-construction estimating into live construction tracking.

## Features

Cost states:

```text
Estimated
Committed
Actual
Forecast
Variance
```

Change orders:

- request
- quantity impact
- cost impact
- approval
- revised budget

Progress:

- planned %
- actual %
- milestone
- site photos
- delay notes

---

# Sprint 43 — Mobile / Field Companion

## Goal

Create field-friendly Buildora AI access.

## Recommended scope

- project list
- drawings
- 3D viewer
- AI Copilot
- site photos
- notes
- issues
- approvals
- progress
- deliveries

Do not attempt complete CAD authoring on mobile first.

Later:

- LiDAR
- AR measurement
- native camera workflows

---

# Sprint 43b — Home Experience & Fixed-Term Project Passes

> **ADR-021.** The two customer lifecycles share one platform and **one set of
> engines**. This sprint adds a second *experience*, never a second product.

## Goal

Serve the project-lifecycle customer (homeowner, self-builder, renovator) whose
usage is intense for 6–18 months and then correctly ends.

## Home experience — UX only

- guided home-project onboarding
- simplified cost view: estimated total, **room-by-room cost**, material choices,
  cost impact of a change
- client-friendly reports
- "invite your architect / QS / contractor" flow

## Fixed-term products

- 6 / 12 / 18-month project passes (`term_model = FIXED_TERM`)
- term expiry → project `archived`, **no cancellation event**
- reactivation restores full capability against the existing model

## Architecture constraints — binding

- **No second QS, BOQ, cost or Building Model engine.** A homeowner's "estimated
  total" is a *presentation* of the same authoritative calculation a QS sees.
  Two engines would mean two sets of golden tests and eventually two different
  answers to the same question.
- Differentiate through **UX · onboarding · entitlements · reporting depth ·
  terminology** only.
- Persona affects presentation; **entitlements control authorization**.
- Analytics must distinguish the two cohorts — a blended churn number is
  meaningless when one segment is *expected* to end.

## Acceptance criteria

- A home user's cost figure and a professional's, for the same model and pinned
  versions, are **identical**
- An expired term archives the project with data intact and records no cancellation
- Reactivation resumes rather than restarts
- `home user → professional invitation` is instrumented (the growth-loop metric)

---

# Sprint 44 — Structural & MEP Concept Modules

## Goal

Extend project intelligence without pretending to replace licensed engineering professionals.

## Structural concept

- column layout metadata
- beam layout metadata
- slab types
- foundation concept
- indicative quantities

## MEP

Initial:

- sockets/lights
- plumbing fixtures
- pipe/cable quantity concepts

## Safety

All engineering-sensitive output clearly communicates:

```text
Concept / planning assistance
Requires qualified professional review
```

---

# Sprint 45 — Sustainability & Compliance Intelligence

## Goal

Support sustainability and regional compliance assistance.

## Sustainability

- material carbon data
- embodied carbon estimate
- material alternatives
- solar potential
- water/waste metrics

## Compliance

- regional rule knowledge base
- project checks
- warnings
- cited evidence

Do not represent AI output as statutory approval.

---

# Sprint 46 — Marketplace & Professional Leads

## Goal

Create additional revenue streams beyond subscriptions.

## Marketplace

Connect users to:

- architects
- QS
- builders
- engineers
- electricians
- plumbers
- suppliers

## Commercial models

- listing fee
- subscription
- lead fee
- commission
- promoted listing

## Trust

- verified profiles
- reviews
- quote history
- fraud controls

---

# Sprint 47 — Enterprise Collaboration & Integrations

## Goal

Prepare Buildora AI for larger construction organisations.

## Features

- SSO
- SCIM later
- advanced RBAC
- company rate libraries
- shared assembly libraries
- project templates
- approvals
- team dashboards
- company branding

## Integrations

Potential:

- Autodesk
- Revit
- AutoCAD
- Xero
- QuickBooks
- Google Drive
- OneDrive
- supplier APIs

Only build integrations that customers actually request.

---

# Sprint 48 — Platform Scale, DR & Performance Maturity

## Goal

Evolve Buildora AI from startup SaaS into a mature platform.

## Performance

- large-model loading
- spatial indices
- geometry worker profiling
- WASM hotspots
- binary payloads
- incremental sync
- model streaming
- caching

## Infrastructure

Evaluate only when necessary:

- additional workers
- Go services
- Kubernetes
- multi-region
- read replicas
- object/CDN tuning

## Reliability

- formal SLOs
- restore drills
- capacity tests
- chaos/failure tests
- incident runbooks
- dependency outage tests

## Enterprise readiness

- security review
- audit export
- retention policies
- privacy controls
- enhanced observability

---

# 15. Commercial Release Milestones

## Alpha 1 — After Sprint 17

Core:

```text
Projects
Building Model
2D Editor
```

Internal only.

## Alpha 2 — After Sprint 21

Core:

```text
2D
↔
3D
```

Internal and trusted testers.

## Alpha 3 — After Sprint 26

Core:

```text
Design
→ Quantities
→ BOQ
→ Cost
```

This is the first version with clear commercial value.

## AI Alpha — After Sprint 30

Adds:

```text
AI Copilot
AI Change Proposal
Value Engineering
```

## Private Beta — After Sprint 34

Adds:

```text
Plan Upload
Recognition
IFC
Documents/RAG
```

## Paid Beta — After Sprint 37

Adds:

```text
Reports
Billing
Onboarding
Analytics
Security Hardening
```

## Production V1 — Sprint 38

First commercial launch.

---

# 16. SaaS-Specific Work That Must Not Be Forgotten

Buildora AI is not only an engineering application.

It is a SaaS business.

The following must exist before paid launch.

## Account lifecycle

- sign up
- sign in
- password/recovery
- account settings
- deactivate/delete workflow

## Tenant lifecycle

- create organisation
- invite
- remove
- role changes
- ownership transfer later

## Subscription lifecycle

- trial
- active
- payment failure
- cancelled
- grace period
- downgrade
- upgrade

## Usage lifecycle

- AI credit grant
- AI reservation
- AI settlement
- render credits
- limits
- usage dashboard

## Project lifecycle

- create
- archive
- restore
- delete
- retention
- ownership

## Commercial communication

- welcome email
- invoice/payment notification
- subscription changes
- usage warning
- failed payment
- major AI job completion

## Support

- support contact
- bug reporting
- project/job correlation ID
- admin audit view

---

# 17. Sprint Metrics

Track per sprint:

```text
planned issues
completed issues
carry-over
tests added
bugs introduced
bugs escaped
CI failures
architecture violations
AI development usage
```

Do not optimise for:

```text
lines of code
```

Optimise for:

```text
working vertical value
correctness
maintainability
```

---

# 18. Product Metrics After Beta

Track:

## Acquisition

- signups
- source
- conversion

## Activation

- project created
- first wall drawn
- 3D opened
- plan uploaded
- first BOQ
- first AI request

## Engagement

- active projects
- AI requests/user
- exports
- design revisions
- BOQs generated

## Revenue

- free → paid
- MRR
- ARPU
- churn
- expansion
- credit purchases

## Cost

- AI cost/user
- AI cost/project
- render cost
- infrastructure cost
- storage cost

## Reliability

- request error rate
- AI failure
- workflow failure
- upload failure
- editor crash
- save/reload failure

---

# 19. Technical KPIs

Recommended eventual targets should be set from real measurements.

Measure:

- dashboard load
- project open time
- 2D canvas FPS
- 3D FPS
- model hydration time
- save latency
- quantity calculation time
- BOQ generation time
- AI first-response latency
- AI tool completion latency
- upload processing duration
- IFC import duration
- report generation time

Do not prematurely invent hard production SLOs before baseline measurements exist.

---

# 20. Solo Developer Scope-Control Rules

Because Buildora AI is a one-person SaaS build, apply these rules strictly.

## Rule 1

Do not work on more than one major engine at the same time.

## Rule 2

Do not add a new technology because it is interesting.

Add it only because a current sprint requires it.

## Rule 3

Do not add an advanced feature when the lower-level engine is still unstable.

Examples:

Do not add AI wall modification before reliable wall editing.

Do not add AI QS before deterministic QS.

Do not add BIM before Building Model is stable.

## Rule 4

Do not build for every country initially.

Start with one pricing/measurement configuration.

## Rule 5

Do not support every construction method initially.

Start with a small verified residential catalogue.

## Rule 6

Do not support every BIM element initially.

Start with:

- level
- wall
- door
- window
- slab
- space

## Rule 7

Do not create 50 AI tools.

Start with 6–8 reliable tools.

## Rule 8

Do not build advanced enterprise infrastructure until paying usage requires it.

---

# 21. Recommended Initial Feature Cut for V1

Must have:

```text
Authentication
Organisations
Projects
Dashboard
2D Floor Plan
3D Building
Materials
Basic Assemblies
Quantities
BOQ
Cost Estimate
AI Copilot
Plan Upload
Recognition Review
Basic IFC
Documents
Reports
Billing
Usage Credits
Admin
Monitoring
```

Can wait:

```text
Photoreal Video
Full MEP
Full Structural Design
Procurement
Marketplace
GIS/Wind/Noise
Mobile CAD
Full Revit Roundtrip
Enterprise SSO
Global Cost Database
```

---

# 22. Release Gate Checklist

Before any production release:

## Code

- [ ] lint passes
- [ ] typecheck passes
- [ ] tests pass
- [ ] build passes

## Domain

- [ ] no duplicate Building Model state
- [ ] version migration reviewed
- [ ] geometry validation passes
- [ ] deterministic calculations pass

## SaaS

- [ ] tenant isolation passes
- [ ] entitlement checks pass
- [ ] audit events pass

## Security

- [ ] no secrets committed
- [ ] upload validation
- [ ] signed file access
- [ ] webhook signatures
- [ ] authorization

## AI

- [ ] model aliases configured
- [ ] usage ledger
- [ ] error handling
- [ ] tool validation
- [ ] no direct geometry mutation
- [ ] deterministic calculations remain authoritative

## Operations

- [ ] staging smoke test
- [ ] migration plan
- [ ] backup health
- [ ] monitoring
- [ ] rollback procedure

---

# 23. AI Agent Responsibility by Sprint Type

| Sprint Type | Claude | Codex | Antigravity |
|---|---|---|---|
| Architecture | Lead review | spike/prototype | optional |
| Backend/domain | plan/review | lead implementation | no |
| 2D/3D | algorithm review | lead implementation | browser verification |
| QS/cost | calculation review | lead implementation | UI verification |
| AI | tool/policy review | gateway implementation | Copilot UX testing |
| SaaS/billing | security review | implementation | checkout testing |
| Import | pipeline review | implementation | upload workflow testing |
| Launch | risk review | fixes/automation | regression testing |

---

# 24. Suggested Sprint Branch Naming

```text
sprint/01-repo-bootstrap
sprint/02-local-infra-ci
sprint/03-design-system
sprint/04-auth
...
```

For smaller work inside the sprint:

```text
feature/wall-selection
fix/window-hosting
test/qs-wall-fixture
```

Do not force one giant sprint branch if multiple PRs are safer.

---

# 25. Documentation Per Sprint

Every sprint should update relevant docs.

Examples:

Building Model sprint:

```text
docs/architecture/05_Building_Model_Domain.md
```

AI sprint — *(not yet written; write at the start of Phase 12. Until then use
ADR-010 and ADR-013.)*

```text
docs/architecture/10_AI_Gateway_and_Model_Routing.md
docs/architecture/11_AI_Copilot_Tool_Architecture.md
```

QS sprint — *(not yet written; write at the start of Phase 10. Until then use
ADR-009, ADR-015 and `15_Database_and_Data_Architecture.md` §58–§72.)*

```text
docs/architecture/12_QS_Engine.md
docs/architecture/13_BOQ_and_Cost_Engine.md
```

Architecture changes require:

```text
ADR
```

Do not let implementation become the only documentation.

---

# 26. Sprint Review Template

At the end of each sprint, record:

```text
Sprint:
Goal:

Completed:
- ...

Not completed:
- ...

Architecture decisions:
- ...

New ADRs:
- ...

Tests:
- ...

Known issues:
- ...

Performance observations:
- ...

Security observations:
- ...

Next sprint prerequisites:
- ...

Deployment status:
- local / staging / production
```

---

# 27. Sprint Planning Template

Before starting each sprint:

```text
SPRINT XX — NAME

GOAL
One measurable outcome.

USER VALUE
Why this sprint matters.

ARCHITECTURE DOCS
Relevant references.

IN SCOPE
- ...

OUT OF SCOPE
- ...

ISSUES
- ...

ACCEPTANCE CRITERIA
- ...

TEST PLAN
- ...

MIGRATIONS
- ...

SECURITY
- ...

OBSERVABILITY
- ...

AI COST IMPACT
- ...

ROLLBACK PLAN
- ...
```

---

# 28. Backlog Priority System

Use:

```text
P0 — blocks release / security / data correctness
P1 — core user value / architectural requirement
P2 — important improvement
P3 — later optimisation
```

Do not let visual polish P2/P3 work delay core P0/P1 domain correctness.

---

# 29. Bug Severity

```text
S0
Security/data-loss/cross-tenant breach

S1
Core model/QS/cost incorrect

S2
Major workflow blocked

S3
Minor workflow or UX defect

S4
Cosmetic
```

S0/S1 must be fixed before release.

---

# 30. Long-Term Product Roadmap After Sprint 48

Sprint 48 should not be considered “the end”.

Potential later directions:

- contractor ERP
- AI construction scheduling optimisation
- richer BIM roundtrip
- Revit plugins
- AutoCAD plugins
- point cloud
- LiDAR
- digital twin
- site IoT
- advanced structural analysis integrations
- supplier ordering network
- insurance/lender integrations
- tender marketplace
- developer feasibility analysis
- planning permission workflows
- generative site layouts
- regional regulations
- enterprise private AI deployments

These should be prioritised using actual paying-customer evidence.

---

# 31. Final Execution Sequence

The sequence to remember is:

```text
FOUNDATION
    ↓
SAAS CORE
    ↓
BUILDING MODEL
    ↓
2D
    ↓
3D
    ↓
MATERIALS
    ↓
QS
    ↓
BOQ
    ↓
COST
    ↓
AI
    ↓
UPLOAD
    ↓
BIM
    ↓
DOCUMENTS
    ↓
REPORTS
    ↓
BILLING
    ↓
BETA
    ↓
LAUNCH
    ↓
PROFESSIONAL EXPANSION
```

Do not reverse this order merely because later features appear more impressive.

---

# 32. Final Principle

For Buildora AI as a solo-developed SaaS:

> **Build the smallest reliable vertical slice, prove it, test it, merge it, and only then expand.**

The product will succeed technically if these six engines remain reliable:

```text
1. Building Model
2. 2D Engine
3. 3D Engine
4. QS Engine
5. Cost Engine
6. AI Orchestration Engine
```

The SaaS business will succeed only if those engines are wrapped with equally reliable:

```text
Authentication
Tenancy
Billing
Usage Control
Security
Monitoring
Support
Onboarding
```

This sprint plan should therefore be treated as the primary execution order unless a later ADR or real customer evidence justifies changing the sequence.
