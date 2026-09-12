# BuildWise — Codex Implementation Reference

> **Purpose:** Source-of-truth implementation guidance for BuildWise. Use this file together with other project `.md` files added later.
>
> **Codex rule:** If a newer project document explicitly overrides a decision here, follow the newer explicit decision. Otherwise preserve the rules in this file.

---

## 1. Product Definition

BuildWise is an **AI Building Design, Quantity Surveying, Costing, and Construction Intelligence Platform**.

Core workflow:

```text
Idea / Prompt / Existing Drawing
        ↓
Structured Building Model
        ↓
Editable 2D Plan
        ↓
Editable 3D / BIM View
        ↓
Quantity Takeoff
        ↓
BOQ
        ↓
Cost Estimate
        ↓
Material Requirements
        ↓
Cost Optimisation
        ↓
Schedule / Construction Intelligence
        ↓
Project-aware AI Copilot
```

BuildWise must not behave like a simple AI image generator. The output must become structured, editable construction data.

---

## 2. Non-Negotiable Architectural Principle

> **The BuildWise Building Model is the single source of truth.**

Do not maintain separate unrelated truth models for 2D, 3D, BIM, quantities, BOQ, or cost.

```text
                BUILDWISE BUILDING MODEL
                         │
        ┌────────────────┼────────────────┐
        │                │                │
       2D               3D              IFC
        │                │                │
        └────────────────┼────────────────┘
                         │
                    Quantities
                         │
                        BOQ
                         │
                       Cost
                         │
                     Schedule
                         │
                    AI Copilot
```

Example: `Window W-102` is the same logical object in the 2D plan, 3D view, IFC export, quantity takeoff, BOQ, cost estimate, revision history, and AI context.

A change to one building element must invalidate/recalculate all dependent outputs.

---

## 3. Recommended Technology Stack

### Web / Main UI
- React
- Next.js
- TypeScript
- Tailwind CSS
- Radix UI primitives where useful
- Storybook for the component library

### 2D CAD
- PixiJS for GPU-accelerated rendering
- Custom CAD/editor domain engine
- TypeScript geometry for normal operations
- Rust → WebAssembly for performance-critical geometry when justified

### 3D
- Three.js
- Direct Three.js engine package rather than making React component state the 3D engine
- WebGPU when available
- WebGL2 fallback

### BIM
- IFC for interoperability
- web-ifc / That Open ecosystem / Fragments for browser BIM workflows
- IfcOpenShell for server-side BIM processing

### CAD / Geometry
- OpenCascade for advanced CAD/B-Rep geometry when required
- ODA SDK for serious DWG/DXF support; do not implement a custom DWG parser

### Core Backend
- NestJS + TypeScript

### AI / Computer Vision
- AI Gateway in TypeScript
- Python + FastAPI for CV/ML/BIM-heavy services
- OpenCV
- PyTorch
- ONNX Runtime where suitable

### Database
- PostgreSQL
- PostGIS
- pgvector

### Cache / Realtime / Workflows
- Redis
- WebSockets
- Yjs and/or Liveblocks selectively for presence/comments/cursors
- Temporal for durable multi-step workflows

### Files / Rendering
- Cloudflare R2 or AWS S3
- Browser GPU for interactive 3D
- Blender worker pool for photoreal rendering

### Infrastructure
- Docker
- Cloudflare
- AWS or equivalent managed cloud
- OpenTelemetry
- Sentry

Do not introduce Kubernetes or many microservices before there is a measured need.

---

## 4. Repository Shape

Recommended monorepo:

```text
buildwise/
├── apps/
│   ├── web/
│   ├── api/
│   ├── ai-worker/
│   ├── bim-worker/
│   └── render-worker/
├── packages/
│   ├── design-system/
│   ├── domain/
│   ├── building-model/
│   ├── cad-2d/
│   ├── engine-3d/
│   ├── qs-engine/
│   ├── cost-engine/
│   ├── ai-tools/
│   ├── api-contracts/
│   └── units/
├── native/
│   └── geometry-wasm/
└── infrastructure/
```

Recommended TypeScript tooling:
- pnpm
- Turborepo

---

## 5. Building Model

The model represents construction objects, not meshes.

Example:

```json
{
  "id": "wall_102",
  "type": "wall",
  "levelId": "ground",
  "start": [1200, 4500],
  "end": [6400, 4500],
  "height": 2700,
  "thickness": 200,
  "assemblyId": "EXT-BLOCK-200"
}
```

Canonical internal geometry unit: **millimetres**.

UI may display metric or imperial units, but the internal model must use one canonical system.

Core hierarchy:

```text
Project
 └── Site
     └── Building
         ├── Level
         │   ├── Space / Room
         │   ├── Wall
         │   ├── Door
         │   ├── Window
         │   ├── Column
         │   ├── Beam
         │   ├── Slab
         │   ├── Stair
         │   └── Equipment
         └── Roof
```

Potential element metadata:
- geometry
- type
- level
- relationships
- assembly
- material
- finish
- fire rating
- IFC class
- source drawing/revision
- recognition confidence
- verification state
- cost references

IFC and GLB are derivatives/interchange formats, not the live source of truth.

---

## 6. 2D Editor

Core tools:
- Select
- Wall
- Door
- Window
- Room
- Stair
- Column
- Beam
- Slab
- Opening
- Dimension
- Measure
- Text / annotation
- Shapes
- Layers

Core behaviour:
- pan
- zoom
- grid
- snapping
- orthogonal constraints
- selection/multi-selection
- move/rotate
- copy/mirror
- visibility
- locking
- floor switching
- undo/redo
- property inspector

The PixiJS display tree is a rendering layer only. Do not persist it as the project model.

Potential Rust/WASM responsibilities:
- segment intersections
- polygon clipping
- offsets
- room-boundary detection
- triangulation
- spatial indexing
- collision tests
- geometry validation

---

## 7. 3D Workspace

Core features:
- orbit
- walkthrough / first-person
- pan / zoom
- object selection
- floor isolation
- layers
- material editing
- section cut
- daylight mode
- explode mode if useful
- object/property inspector
- standard views
- measurements

Transition:

```text
2D
 ↓
Generate / Sync 3D
 ↓
3D Workspace
```

Manual changes should update the building model and regenerate only affected geometry where possible.

Do not use a server GPU for ordinary interactive 3D navigation.

---

## 8. Create Project Onboarding

Start methods:

```text
Generate with AI
Upload Existing Plan
Draw Manually
Import BIM
```

### Generate with AI
Collect:
- project type
- location
- plot size
- floors
- bedrooms
- bathrooms
- target area
- budget
- design style
- additional requirements

Flow:

```text
Project Brief
 ↓
AI Analysis
 ↓
Structured Brief
 ↓
User Review
 ↓
Generate Design Concepts
 ↓
Compare
 ↓
Select
 ↓
Create BuildWise Building Model
```

AI must create a structured brief before geometry generation.

---

## 9. Upload Plan Flow

```text
Upload Floor Plan
       ↓
AI Processing
       ↓
Walls detected
Rooms detected
Doors detected
Windows detected
       ↓
Recognition Results
       ↓
Confidence Review
       ↓
Low-confidence warnings
       ↓
User verifies/corrects
       ↓
Successful Conversion
       ↓
Open in 2D Editor
```

Required UI states:
- upload
- processing
- recognition results
- low-confidence warnings
- verification
- successful conversion
- error

Hybrid recognition pipeline:

```text
                UPLOADED PLAN
                     │
              Detect file type
                     │
      ┌──────────────┴──────────────┐
      │                             │
 Vector PDF / DXF              Scanned Image
      │                             │
 Parse vector data              Vision / CV
      │                             │
      └──────────────┬──────────────┘
                     │
              AI semantic layer
                     │
              Building elements
                     │
                 Confidence
                     │
              Human verification
```

If vector information exists, use it. Do not unnecessarily rasterise precise vectors and ask vision to rediscover them.

---

## 10. AI Copilot Architecture

Users see one assistant:

> **BuildWise AI**

The AI must be available throughout the application, not only on a separate page.

Examples:

### BOQ
`Why is concrete cost so high?`

### 3D Editor
`Reduce this room by 1m and expand the kitchen.`

### Cost
`Reduce overall project cost to £230,000.`

### Documents
`What concrete grade does the structural engineer specify?`

The AI should call project tools and deterministic engines rather than invent facts.

---

## 11. Automatic Model Routing

Normal users should not need to choose an AI model.

UI modes may be:
- Automatic
- Fast
- Deep Analysis

Internally use aliases such as:

```text
FAST_MODEL
BALANCED_MODEL
ADVANCED_MODEL
EXPERT_MODEL
```

Current planning discussions map these conceptually to Luna / Terra / Sol / Astra, but **provider/model IDs must be configuration, not hardcoded business logic**.

Router signals:
- task type
- complexity
- context size
- number of affected elements
- number of tools
- number of constraints
- design/financial impact
- cross-document reasoning
- confidence requirement
- user-plan limits
- latency preference

Illustrative routing:

```text
Simple retrieval/classification
→ FAST_MODEL

Normal construction Q&A
→ BALANCED_MODEL

Advanced cost/design analysis
→ ADVANCED_MODEL

Major geometry / optimisation / multi-step design work
→ EXPERT_MODEL
```

Support escalation:

```text
BALANCED_MODEL
 ↓ low confidence/conflict/tool failure
ADVANCED_MODEL
 ↓ still unresolved
EXPERT_MODEL
```

Also downgrade automatically for simple follow-up requests.

Track:
- initial model
- final model
- escalation reason
- tokens
- cost
- latency
- confidence
- tool failures

---

## 12. AI Gateway

The AI Gateway should own:
- model selection
- reasoning level
- prompts
- project-context construction
- tool schemas
- tool permissions
- token/context budgets
- provider caching
- retries
- escalation
- usage limits
- cost tracking
- audit logs
- structured-output validation

Use a tool-capable provider API such as the OpenAI Responses API behind an abstraction layer.

---

## 13. AI Must Not Be the Geometry Engine

Wrong:

```text
LLM outputs arbitrary polygons
→ save directly
```

Correct:

```text
AI intent
 ↓
Structured ChangeSet
 ↓
Geometry engine
 ↓
Validation
 ↓
Preview
 ↓
Approval when required
 ↓
Commit
```

Example:

```json
{
  "action": "move_wall",
  "elementId": "W-102",
  "deltaMm": 1000
}
```

High-impact AI changes should not silently mutate the production model.

---

## 14. AI Change Transaction

Use:

```text
AI Request
 ↓
Generate ChangeSet
 ↓
Draft / Sandbox Model
 ↓
Geometry Validation
 ↓
QS Recalculation
 ↓
Cost Impact
 ↓
Preview
 ↓
User Approval when required
 ↓
Commit Revision
```

Examples requiring approval:
- wall movements
- room resizing
- bulk material changes
- major cost changes
- structural-related changes
- schedule-impacting changes

---

## 15. QS Engine

QS must be deterministic.

Do not let an LLM perform authoritative quantity arithmetic.

Possible package:

```text
packages/qs-engine
```

Inputs:
- building elements
- geometry
- assemblies
- project rules
- waste rules

Outputs:
- quantity lines
- measurement breakdown
- formulas/source elements
- warnings

Support:
- count
- length
- perimeter
- area
- volume
- weight

Initial trades:
- groundworks
- foundations
- concrete
- masonry
- walls
- openings
- roofing
- finishes
- flooring
- painting

Later:
- electrical
- plumbing
- HVAC
- fire
- external works

---

## 16. Assemblies / Recipes

Example:

```text
1 m² external block wall
```

may derive:
- blocks
- mortar
- cement
- sand
- plaster
- render
- primer
- paint
- labour
- waste

Users should later be able to:
- create
- clone
- assign
- version
- import/export
- share company assemblies

---

## 17. BOQ Engine

Structure:

```text
Section
 └── Trade
     └── Element
         └── Item
```

Typical columns:
- item number
- description
- unit
- quantity
- material rate
- labour rate
- plant rate
- subcontract rate
- total rate
- amount
- waste
- location
- source drawing
- revision

Exports:
- XLSX
- CSV
- PDF

---

## 18. Cost Engine

Separate:
- materials
- labour
- plant/equipment
- subcontractors
- transport
- waste
- overheads
- profit
- tax
- contingency

Use decimal-safe financial calculations.

Recommended:
- PostgreSQL `NUMERIC`
- decimal library in TypeScript

Cost levels:
1. Concept Estimate
2. Elemental Estimate
3. Detailed Estimate
4. Tender Estimate

The UI must communicate estimate confidence/level.

---

## 19. Regional Pricing

Rates need:
- currency
- country
- region
- supplier
- effective date
- tax
- delivery
- source
- confidence
- user override

Rate-data integrations and licensing must remain pluggable.

---

## 20. Quantity / Cost Audit Trail

Every professional quantity/cost should be explainable.

A user should be able to see:
- contributing building elements
- calculation formula
- assembly
- waste assumption
- rate source
- project/model revision
- user overrides

Example:

```text
Internal plaster: 186.4 m²

Derived from:
W-102
W-103
W-111
...

Model Version: 42
```

---

## 21. QS Screens

Implement in this order:

1. Quantities
2. BOQ
3. Cost Estimate
4. Material Breakdown
5. Cost Analysis
6. Reports

This is a major BuildWise differentiator from consumer-only 3D planning applications.

---

## 22. Commands / Versions / Undo

Important building-model edits should be commands.

Example:

```json
{
  "command": "MOVE_WALL",
  "elementId": "W-102",
  "from": [4000, 5000],
  "to": [5000, 5000],
  "userId": "123",
  "baseVersion": 42
}
```

This supports:
- undo
- redo
- model versions
- AI preview
- audit
- rollback
- collaboration
- revision comparison

Server validates committed geometry changes.

---

## 23. Persistence

Use PostgreSQL for authoritative project data.

Conceptual tables:

```text
projects
buildings
levels
building_elements
element_properties
materials
assemblies
assembly_items
quantities
boq_sections
boq_items
cost_rates
cost_estimates
documents
document_revisions
model_versions
commands
ai_actions
audit_events
```

Use:
- PostGIS for site/spatial data
- pgvector for project-document/RAG embeddings

Do not store large meshes directly in PostgreSQL.

---

## 24. Object Storage

Use R2/S3 for:
- PDFs
- images
- IFC
- CAD files
- GLB/glTF
- Fragments
- textures
- renders
- generated reports

Database stores metadata/references.

---

## 25. Realtime Collaboration

Use WebSockets plus Yjs/Liveblocks selectively.

Good CRDT/presence use cases:
- comments
- annotations
- text
- cursors
- selections
- presence

Do not blindly CRDT-merge major building geometry.

Geometry changes should use:
- server-authoritative commands
- version checks
- optimistic locking
- semantic conflict handling

---

## 26. Durable Workflows

Use Temporal for long/multi-step operations such as:

```text
Upload Plan
 ↓
Extract
 ↓
Recognise
 ↓
Interpret
 ↓
Wait for Verification
 ↓
Build Model
 ↓
Generate 3D
 ↓
Calculate Quantities
 ↓
Calculate Cost
 ↓
Generate Preview
```

The workflow must resume after recoverable failures instead of forcing the user to restart everything.

Redis is for cache/rate limiting/presence/locks, not authoritative project state.

---

## 27. Dashboard

Keep it relatively simple.

Required:
- Recent projects
- Create Project
- Active projects
- Estimated portfolio cost
- Pending AI actions
- Recent documents
- Recent activity
- Quick actions

Quick actions:
- Generate with AI
- Upload Plan
- Draw Manually
- Import BIM

---

## 28. Application Navigation

```text
Dashboard
Projects

DESIGN
├── Overview
├── 2D Plan
├── 3D Model
└── Materials & Finishes

CONSTRUCTION
├── Quantities
├── BOQ
├── Cost Estimate
└── Schedule

PROJECT
├── Documents
├── Reports
├── Versions
└── Team & Collaboration

RESOURCES
├── Marketplace
└── Knowledge Base

AI Copilot
Settings
```

---

## 29. $49 Pro Plan — Usage Architecture

The $49 plan should target approximately **70%+ gross margin**.

### Effectively Unlimited / Low Marginal Cost
- manual 2D editing
- manual 3D editing
- normal interactive 3D viewing
- 2D ↔ 3D synchronisation
- deterministic quantity recalculation
- deterministic BOQ recalculation
- deterministic cost recalculation

These operations should not consume expensive AI design credits.

### AI Design Credits
Recommended starting allowance:

> **30 AI Design Credits / month**

Illustrative weights:

| Operation | Credits |
|---|---:|
| Standard AI house concept | 2 |
| Advanced AI concept | 3 |
| Expert/highest-model design study | 5 |
| Major AI design modification | 1 |
| Manual 2D → 3D sync | 0 |
| Manual 3D editing | 0 |
| Deterministic BOQ recalculation | 0 |
| Deterministic cost recalculation | 0 |

This corresponds approximately to:
- up to ~15 standard concepts, or
- ~10 advanced concepts, or
- ~6 expert concepts,
- or a mixture.

For marketing, describe it approximately as **10–15 full AI design concepts/month depending on complexity** rather than guaranteeing a fixed number of the most expensive jobs.

### Render Credits
Keep render credits separate.

Suggested starting allowance:

> **20 Render Credits / month**

Illustrative use:

| Rendering | Credits |
|---|---:|
| Real-time 3D | 0 |
| Standard preview | 0 / fair-use |
| HD render | 1 |
| 4K photoreal render | 2 |
| 360 panorama | 3 |
| Video walkthrough | 5–10 |

All credit rules must be configurable.

---

## 30. Pro Plan Unit Economics Target

Illustrative target only:

```text
Subscription Revenue        $49.00

Infrastructure              ~$2.00
Normal AI                   ~$2.50
Design AI                   ~$6.00
Rendering                   ~$1.50
Payment Processing          ~$1.50
Other Variable Cost         ~$0.50
─────────────────────────────────
Target Variable Cost        ~$14.00

Target Gross Contribution   ~$35.00
Target Gross Margin         ~71%
```

Do not hardcode these figures. Measure actual production usage and update commercial configuration.

---

## 31. AI Cost-Control Rule

Do not regenerate an entire building for a small modification.

Bad:

```text
Move one wall
 ↓
Send entire project to expert model
 ↓
Regenerate whole building
```

Correct:

```text
User request
 ↓
AI identifies affected elements
 ↓
Structured ChangeSet
 ↓
Geometry engine updates affected region
 ↓
Only impacted 3D regenerated
 ↓
QS delta recalculated
 ↓
Cost delta recalculated
```

This is a core profitability and performance requirement.

---

## 32. Prompt / Context Efficiency

Where supported, structure provider requests to benefit from caching.

Stable context candidates:
- system instructions
- tool definitions
- model schema
- construction classifications
- stable project rules

Prefer targeted retrieval and deltas over resending the entire project state.

---

## 33. Rendering Cost Architecture

Interactive 3D:

```text
BuildWise model
 ↓
Browser
 ↓
Three.js
 ↓
User GPU
```

Photoreal rendering:

```text
Render Request
 ↓
Temporal Workflow
 ↓
Blender Worker
 ↓
CPU/GPU
 ↓
R2/S3
```

Expensive rendering should be queued, metered, and credit-limited.

---

## 34. Reports

Target reports:
- project summary
- concept design report
- quantity takeoff
- detailed BOQ
- cost estimate
- material schedule
- labour report
- cost analysis
- value-engineering report
- revision comparison
- schedule
- cash flow later
- sustainability/carbon later

Exports:
- PDF
- XLSX
- CSV

---

## 35. Document Intelligence

Project documents may include:
- architectural drawings
- structural drawings
- MEP drawings
- specifications
- contracts
- quotations
- invoices
- reports
- certificates
- photos

AI document answers must come from actual project sources/tools.

The product should retain evidence/source references for professional traceability.

---

## 36. Security / Audit

Implement:
- RBAC
- project permissions
- MFA support
- secure file access
- encryption in transit
- audit logs
- model revision history
- AI action logs
- usage/cost logs
- backup/recovery strategy

AI action logs should capture:
- user
- timestamp
- task type
- model alias/provider model
- source documents/data
- tools called
- proposed changes
- previous/new values
- approval state
- usage cost metadata

---

## 37. User Roles — Long-Term

- Homeowner
- Architect
- Architectural Designer
- Quantity Surveyor
- Estimator
- Contractor / Builder
- Structural Engineer
- MEP Engineer
- Interior Designer
- Supplier
- Client / Viewer
- Company Admin
- Platform Admin

Permission dimensions:
- view
- comment
- edit design
- edit quantities
- edit rates
- approve
- export
- manage team
- manage financials

---

## 38. MVP Scope

Recommended first commercial MVP:

1. Accounts/projects
2. Create Project onboarding
3. Generate-with-AI project brief
4. PDF/image plan upload
5. Plan recognition + confidence review
6. 2D editor
7. BuildWise Building Model
8. 3D generation/synchronisation
9. Materials/assemblies
10. Quantity takeoff
11. BOQ
12. Concept/elemental cost estimate
13. AI Copilot
14. Design change → quantity/cost recalculation
15. PDF/XLSX reports
16. Version history

Primary MVP proof:

```text
Upload or Describe Building
        ↓
Verify / Generate Plan
        ↓
Edit in 2D
        ↓
Generate 3D
        ↓
Assign Materials
        ↓
Calculate Quantities
        ↓
Generate BOQ
        ↓
Estimate Cost
        ↓
Ask AI Questions
        ↓
Modify Design
        ↓
Recalculate Automatically
```

---

## 39. Later Modules

Do not block the MVP on these:
- advanced interiors
- GIS/site intelligence
- terrain
- daylight/sun analysis
- landscape
- advanced photoreal rendering
- structural concept tools
- MEP
- scheduling
- cash flow
- supplier live pricing
- RFQs
- purchase orders
- tender comparison
- actual cost tracking
- change orders
- site progress
- mobile companion
- client portal
- marketplace
- supplier/contractor leads
- carbon/sustainability
- regulatory assistance

---

## 40. Implementation Rules for Codex

Codex must preserve these unless explicitly overridden later:

1. The Building Model is the source of truth.
2. 2D and 3D are views/derivatives of the same model.
3. IFC is interoperability, not the live project database.
4. LLMs do not perform authoritative QS arithmetic.
5. LLMs do not directly commit unvalidated geometry.
6. AI modifications use structured ChangeSets.
7. QS/cost logic is deterministic and auditable.
8. Financial calculations use decimal-safe arithmetic.
9. Construction geometry uses one canonical unit internally.
10. Recognition confidence and human verification are first-class concepts.
11. Interactive 3D should run primarily on the client GPU.
12. Expensive rendering should be queued and credit-limited.
13. Model routing is automatic, cost-aware, and configurable.
14. Provider model names/prices are never hardcoded business rules.
15. Do not duplicate 2D/3D/QS/cost sources of truth.
16. Important model edits are versioned commands.
17. High-impact AI changes require validation/preview/approval.
18. Every professional quantity/cost should be traceable to source elements and assumptions.
19. Preserve vector data when available.
20. Do not introduce extra languages/services without a concrete reason.

---

## 41. Suggested Development Order

```text
Phase 1  Design system + application shell
Phase 2  Projects + onboarding
Phase 3  Building-model domain package
Phase 4  2D editor foundations
Phase 5  3D engine + 2D/3D sync
Phase 6  Upload/recognition pipeline
Phase 7  AI Gateway + project brief
Phase 8  Materials + assemblies
Phase 9  QS engine
Phase 10 BOQ + cost engine
Phase 11 AI Copilot tools
Phase 12 Versions + reports
Phase 13 Performance + security hardening
```

Do not prioritise advanced rendering or MEP before the Building Model and geometry lifecycle are stable.

---

## 42. Coding Guidance

For each implementation task:

1. Identify the owning domain/package.
2. Keep business logic out of React components.
3. Keep authoritative calculations out of controllers.
4. Reuse shared schemas/contracts.
5. Use explicit types for units and money.
6. Prefer pure deterministic functions for QS/cost logic.
7. Validate AI structured output.
8. Validate geometry ChangeSets.
9. Add unit tests for calculations.
10. Add integration tests for 2D → model → 3D sync.
11. Add model-version tests for geometry mutations.
12. Emit audit events for important changes.
13. Keep vendor APIs behind adapters.
14. Keep model routing/credit rules configurable.
15. Document assumptions.

---

## 43. Definition of Done — Core Features

A core feature is complete only when:
- UI uses the shared design system,
- domain logic is separated from presentation,
- validation is present,
- loading/error/empty states are handled,
- permissions are checked,
- relevant audit events are emitted,
- tests are added,
- model/version behaviour is verified,
- AI usage/cost is recorded where applicable,
- no duplicate source of truth is introduced,
- affected 2D/3D/QS/cost outputs are invalidated/recalculated correctly,
- contracts/documentation are updated.

---

## 44. Product Positioning

Do not optimise the product toward being only:

> an AI 3D house designer.

BuildWise should be:

> **An AI construction copilot that turns an idea or existing plan into a buildable, measurable, costed digital building.**

Strongest long-term behaviour:

> **Design change → geometry change → quantity change → BOQ change → cost change → schedule change, automatically and audibly.**

---

## 45. Configuration — Never Hardcode

Keep configurable:
- model/provider IDs
- model tier mapping
- reasoning effort
- AI token budgets
- AI credits
- render credits
- subscription limits
- provider token prices
- storage/upload limits
- confidence thresholds
- escalation thresholds
- regional rates
- tax/VAT
- waste defaults
- currency
- unit system
- render quality
- workflow timeouts

---

## 46. First Instruction to Give Codex

```text
Read BuildWise_Codex_Implementation_Reference.md as the current implementation reference.

Before writing code:
1. Identify the feature/module being implemented.
2. Identify the relevant architectural rules from this file.
3. Preserve the BuildWise Building Model as the source of truth.
4. Do not create separate 2D, 3D, BIM, QS, or cost truth models.
5. Reuse existing packages and design-system components.
6. Keep AI provider/model and commercial credit rules configurable.
7. If the requested implementation conflicts with this reference or another newer project file, identify the conflict explicitly before changing architecture.
8. Implement incrementally with production-oriented typing, validation, tests, auditability, and modularity.
```
