
# BuildWise — Sprintwise Project Plan

> **Product:** BuildWise  
> **Type:** Multi-tenant SaaS application  
> **Development model:** One-person SaaS development assisted by Codex Plus, Claude Pro, and Antigravity Pro  
> **Primary architecture:** Next.js + TypeScript + NestJS + PostgreSQL + Redis + PixiJS + Three.js + Python/FastAPI + Temporal + AI Gateway  
> **Planning style:** Dependency-first, vertical-slice delivery  
> **Recommended sprint length:** 1–2 focused development weeks per sprint, but completion is based on acceptance criteria rather than calendar time.

---

# 1. Purpose of This Plan

This document is the primary execution roadmap for BuildWise.

It converts the full BuildWise product vision into an ordered sprint plan covering:

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

BuildWise is divided into six commercial stages.

| Stage | Sprints | Outcome |
|---|---:|---|
| Stage A | 1–7 | SaaS + repository foundation |
| Stage B | 8–21 | Building Model + 2D + 3D |
| Stage C | 22–26 | Materials + QS + BOQ + Cost |
| Stage D | 27–34 | AI + Upload + BIM + Documents |
| Stage E | 35–38 | Billing + Beta + Commercial Launch |
| Stage F | 39–48 | Post-launch professional expansion |

Recommended **commercial MVP / paid beta target: Sprint 38**.

---

# 4. High-Level Sprint Map

| Sprint | Focus | Commercial Stage |
|---:|---|---|
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
| 22 | Materials & assemblies | QS Foundation |
| 23 | Quantity Surveying engine | QS |
| 24 | BOQ engine | QS |
| 25 | Cost engine & estimate levels | Cost |
| 26 | QS/BOQ/Cost UI + auditability | Cost |
| 27 | AI Gateway & model routing | AI |
| 28 | Read-only AI Copilot | AI |
| 29 | AI change proposals & approval | AI |
| 30 | AI value engineering & usage controls | AI |
| 31 | Upload Plan pipeline | Import |
| 32 | Recognition confidence & verification | Import |
| 33 | IFC/BIM interoperability | BIM |
| 34 | Documents, RAG & project Q&A | Documents |
| 35 | Reports, exports & sharing | Commercial |
| 36 | Billing, plans & credits | Commercial |
| 37 | Beta hardening & onboarding | Beta |
| 38 | Production launch | Launch |
| 39 | Site/GIS/environment analysis | Growth |
| 40 | Schedule & cash flow | Growth |
| 41 | Procurement, suppliers & tender comparison | Growth |
| 42 | Actual cost, change orders & progress | Growth |
| 43 | Mobile/site companion | Growth |
| 44 | Structural & MEP concept modules | Professional |
| 45 | Sustainability/compliance intelligence | Professional |
| 46 | Marketplace & professional leads | Growth |
| 47 | Enterprise collaboration & integrations | Enterprise |
| 48 | Scale, DR, performance & platform maturity | Enterprise |

---

# 5. Stage A — SaaS and Engineering Foundation

---

# Sprint 01 — Repository & Architecture Bootstrap

## Goal

Create a clean, controlled BuildWise repository that all AI tools can work in safely.

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
packages/domain
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

## Deliverables

Docker Compose:

- PostgreSQL
- Redis

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

Lock the BuildWise visual system before page-by-page implementation.

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
- authenticated user receives local BuildWise user
- API rejects invalid identity
- logout invalidates access correctly

---

# Sprint 05 — Organisations & Multi-Tenancy

## Goal

Make BuildWise a proper multi-tenant SaaS.

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

Feature flags:

```text
AI_CONCEPT_GENERATION
PLAN_RECOGNITION
IFC_IMPORT
PRO_QS
PHOTOREAL_RENDERING
```

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

Create the foundation of the BuildWise domain.

## Critical rule

Canonical geometry unit:

```text
millimetres
```

## Deliverables

`@buildwise/units`:

- LengthMm
- AreaMm2 or safe conversion representation
- VolumeMm3 or domain conversion helpers
- angle types
- conversion helpers

`@buildwise/building-model`:

- ProjectModel
- Building
- Level
- element IDs
- element base types

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

Room:

- closed-boundary detection
- room name
- room area
- room label

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

Prove BuildWise's most important architectural promise.

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

## UI

- material library
- assembly selector
- assign assembly to wall
- material/finish inspector

## Acceptance criteria

A wall can reference an assembly without embedding duplicated material calculations in the wall object.

---

# Sprint 23 — Quantity Surveying Engine

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

## Acceptance criteria

No LLM participates in authoritative quantity calculation.

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

BuildWise now provides:

```text
Design → 3D → Quantity → BOQ → Cost
```

without AI.

---

# 11. Stage D — AI

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
→ BuildWise Model
```

## Export

Basic supported IFC export can be a stretch goal.

## Acceptance criteria

IFC remains an interchange format, not BuildWise's internal source of truth.

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

Make output useful outside BuildWise.

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

# Sprint 36 — Billing, Plans & Credits

## Goal

Turn BuildWise into a revenue-generating SaaS.

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

## Feature entitlement

Examples:

- project limits
- AI credits
- exports
- IFC
- professional QS

## Acceptance criteria

Client cannot unlock paid capabilities by modifying browser state.

---

# Sprint 37 — Beta Hardening & Onboarding

## Goal

Prepare BuildWise for real external users.

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

Release the first paid commercial BuildWise version.

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

Create field-friendly BuildWise access.

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

Prepare BuildWise for larger construction organisations.

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

Evolve BuildWise from startup SaaS into a mature platform.

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

BuildWise is not only an engineering application.

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

Because BuildWise is a one-person SaaS build, apply these rules strictly.

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

AI sprint:

```text
docs/architecture/10_AI_Gateway_and_Model_Routing.md
docs/architecture/11_AI_Copilot_Tool_Architecture.md
```

QS sprint:

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

For BuildWise as a solo-developed SaaS:

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
