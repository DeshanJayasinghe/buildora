# BuildWise — Solo Developer AI-Assisted Development Guide

> **Purpose**
>
> This document defines how BuildWise should be developed as a **one-person SaaS project** using:
>
> - Claude Pro
> - Codex Plus
> - Antigravity Pro
>
> The goal is to use these tools as an AI-assisted engineering team while keeping one clear architecture, one source of truth, and strong implementation discipline.

---

# 1. Core Development Principle

The main constraint for a solo BuildWise developer is **not coding speed**.

The real risks are:

- scope explosion,
- architectural drift,
- inconsistent implementations,
- duplicated state,
- weak testing,
- AI-generated shortcuts,
- trying to build too many modules at once.

The correct mindset is:

> **You are the Product Owner + CTO + Technical Lead.**

The AI tools act as your engineering team.

```text
                         YOU
                  Product Owner / CTO
                         │
          ┌──────────────┼──────────────┐
          │              │              │
        Codex        Claude Code    Antigravity
          │              │              │
    Implementation    Review /       Browser/UI
    + complex code    reasoning       verification
          │              │              │
          └──────────────┼──────────────┘
                         │
                       Git
                         │
                    BuildWise
```

---

# 2. Recommended Tool Roles

## Codex

Primary role:

> **Main implementation agent**

Use Codex for:

- feature implementation,
- refactors,
- migrations,
- unit tests,
- integration tests,
- backend work,
- domain logic,
- 2D editor implementation,
- 3D engine implementation,
- QS engine implementation,
- API work,
- CI fixes.

Typical tasks:

```text
Implement Project bounded context
Implement Wall aggregate
Create database migrations
Implement REST API
Create PixiJS selection system
Implement Three.js wall mesh generator
Implement QS wall calculations
Add unit tests
Fix failing integration tests
Refactor command handler
```

---

# 3. Claude Code Role

Primary role:

> **Senior architecture reviewer / debugger**

Use Claude for:

- implementation planning,
- architecture review,
- design critique,
- complex debugging,
- finding edge cases,
- reviewing Codex diffs,
- challenging assumptions,
- identifying overengineering,
- identifying missing tests,
- security review.

Typical prompt:

```text
Review this implementation against:

docs/architecture/05_Building_Model_Domain.md
docs/architecture/06_2D_CAD_Editor_Implementation.md
docs/architecture/adrs/

Do not modify code yet.

Find:
- architectural violations
- duplicated state
- incorrect abstractions
- concurrency issues
- performance problems
- missing tests
- security problems
- overengineering

Rank findings P0/P1/P2.
```

Recommended loop:

```text
Codex
  ↓
Implementation
  ↓
Claude
  ↓
Review
  ↓
Codex
  ↓
Fix
```

---

# 4. Antigravity Role

Primary role:

> **Browser QA / UI verification engineer**

Use Antigravity for:

- visual verification,
- UI workflows,
- responsive checks,
- browser testing,
- console error checking,
- network inspection,
- regression testing,
- end-to-end workflow validation,
- screenshot review.

Example:

```text
Test the Create Project → Upload Plan workflow.

Verify:
1. drag/drop works
2. invalid files show errors
3. processing state appears
4. confidence results render
5. warning states work
6. verification can be edited
7. successful conversion routes to /2d
8. browser console contains no errors
9. no failed network requests

Do not modify implementation until the test report is complete.
```

Then either:

- Antigravity fixes small UI issues,
- or the findings go back to Codex.

---

# 5. AI Tool Responsibility Matrix

| Tool | Primary Responsibility | Secondary Responsibility |
|---|---|---|
| Codex | Main implementation | Tests, migrations, refactors |
| Claude Code | Architecture/review/debugging | Complex implementation |
| Antigravity | Browser/UI testing | Parallel experiments |
| You | Product decisions and approval | Integration and acceptance |

Do not constantly change these roles.

Stable roles reduce confusion and duplication.

---

# 6. You Must Own Final Architecture Decisions

Do not delegate final authority over:

- Building Model design,
- database boundaries,
- geometry representation,
- units,
- financial precision,
- API contracts,
- permission model,
- command/version architecture,
- AI tool permissions,
- subscription/credit rules,
- security boundaries.

AI can:

- propose,
- critique,
- implement.

You approve.

---

# 7. Architecture Documentation Controls All Agents

Recommended repository structure:

```text
buildwise/
│
├── AGENTS.md
├── CLAUDE.md
│
├── docs/
│   └── architecture/
│       ├── 00_README.md
│       ├── 01_Master_Product_Reference.md
│       ├── ...
│       └── adrs/
│
├── apps/
├── packages/
└── infrastructure/
```

All agents must use:

```text
docs/architecture/
```

as the canonical architecture reference.

Do not create separate architecture rules for:

- Codex,
- Claude,
- Antigravity.

All should point to the same documentation.

---

# 8. Recommended AGENTS.md

Keep it short.

```markdown
# BuildWise Agent Instructions

BuildWise architecture is documented in:
docs/architecture/

Before implementing any feature:

1. Read docs/architecture/00_README.md.
2. Read docs/architecture/01_Master_Product_Reference.md.
3. Read documents relevant to the module being modified.
4. Read applicable ADRs.

Critical rules:

- Building Model is the source of truth.
- Never duplicate 2D / 3D / BIM / QS state.
- AI does not calculate authoritative quantities.
- AI geometry changes use validated ChangeSets.
- Financial calculations are decimal-safe.
- New architectural decisions require an ADR.
- Tests are mandatory.

Never silently change these rules.
```

---

# 9. Git Strategy

Do not let all three tools modify `main`.

Use feature branches.

Example:

```text
main
 │
 └── develop
       │
       ├── feature/project-domain
       ├── feature/building-model
       ├── feature/2d-selection
       ├── feature/3d-wall-rendering
       └── feature/qs-wall-takeoff
```

Even as a solo developer.

Recommended flow:

```text
Create Issue
   ↓
Create Branch
   ↓
Agent Implements
   ↓
Tests
   ↓
Second Agent Reviews
   ↓
Fix
   ↓
You Review Diff
   ↓
Merge
```

---

# 10. Git Worktrees

Use worktrees for parallel AI work.

Example:

```text
/buildwise

/buildwise-worktrees/
    building-model/
    upload-plan/
    dashboard/
```

Use cases:

- Codex implements one feature,
- Claude reviews another,
- Antigravity tests a third.

This prevents agents from editing the same working tree.

---

# 11. Do Not Start With AI Generation

The first serious technical milestone should be:

> **Building Model → 2D → 3D**

Before implementing:

```text
Prompt
 ↓
AI
 ↓
House
```

prove:

```text
Create Wall
       ↓
Building Model
       ↓
2D displays wall
       ↓
3D displays wall
       ↓
Move wall
       ↓
Both update
```

This foundation must be stable first.

---

# 12. First Vertical Slice

Recommended first end-to-end slice:

```text
USER
 ↓
Create Project
 ↓
Create Ground Floor
 ↓
Draw Four Walls
 ↓
Create Room
 ↓
Add Door
 ↓
Add Window
 ↓
Switch to 3D
 ↓
See Correct Building
 ↓
Modify Wall
 ↓
3D Updates
 ↓
Save
 ↓
Reload
 ↓
Everything Is Identical
```

Do not add at this point:

- AI,
- QS,
- rendering,
- BIM import,
- supplier marketplace.

---

# 13. Second Vertical Slice

Add QS to the tiny test house.

Model:

```text
4 Walls
1 Door
1 Window
1 Slab
```

Calculate:

```text
Gross Wall Area
-
Openings
=
Net Wall Area
```

Then:

```text
Net Wall Area
   ↓
Wall Assembly
   ↓
Blocks
Mortar
Cement
Sand
Plaster
Paint
```

Then:

```text
Quantities
 ↓
BOQ
 ↓
Cost
```

This proves the real BuildWise value.

---

# 14. Third Vertical Slice

Only after the above is reliable, add AI.

Example:

```text
Increase this room by 1 metre.
```

Flow:

```text
Natural Language
       ↓
AI Gateway
       ↓
ChangeSet
       ↓
MOVE_WALL W-04 +1000mm
       ↓
Geometry
       ↓
2D
       ↓
3D
       ↓
QS
       ↓
BOQ
       ↓
Cost
```

AI should control a reliable deterministic system.

---

# 15. Recommended 12 Milestones

| Milestone | Deliverable |
|---|---|
| M0 | Repository + CI + architecture rules |
| M1 | Auth + organisations + projects |
| M2 | Building Model |
| M3 | Basic 2D editor |
| M4 | Basic 3D engine |
| M5 | 2D ↔ 3D synchronisation |
| M6 | Materials + assemblies |
| M7 | QS engine |
| M8 | BOQ + cost |
| M9 | AI Gateway + Copilot |
| M10 | Plan upload + recognition |
| M11 | Reports + billing + beta |

---

# 16. Why Plan Recognition Comes Later

If plan recognition is added too early, debugging becomes unclear.

Potential failure sources:

```text
CV error?
AI error?
wall parser error?
geometry error?
building model error?
2D renderer error?
3D renderer error?
```

Instead establish first:

```text
Known-good Building Model
        ↓
Known-good 2D
        ↓
Known-good 3D
```

Then recognition has one job:

> Convert an uploaded plan into valid Building Model commands.

---

# 17. Weekly Vertical Slice Strategy

Do not assign:

> Build the entire 2D editor.

Split it.

Example slices:

```text
Slice 1
Draw wall

Slice 2
Select wall

Slice 3
Move wall

Slice 4
Delete wall

Slice 5
Snap wall

Slice 6
Create closed room
```

Each slice must include:

```text
Domain
UI
Persistence
Tests
```

---

# 18. Standard Issue Template

Every implementation task should have a clear brief.

```text
TITLE
Implement wall endpoint snapping

GOAL
Allow wall endpoints to snap to existing wall endpoints.

ARCHITECTURE
Read:
05_Building_Model_Domain.md
06_2D_CAD_Editor_Implementation.md
17_Command_Versioning_and_Undo_Redo.md

IN SCOPE
- endpoint spatial query
- visual snap indicator
- snap threshold
- command integration
- tests

OUT OF SCOPE
- midpoint snapping
- perpendicular snapping
- grid snapping

ACCEPTANCE CRITERIA
- cursor snaps within threshold
- resulting wall coordinate is exact
- undo restores previous state
- no duplicate element state
- tests pass

TESTS
- unit
- editor integration
```

---

# 19. Plan With One Agent, Build With Another

For difficult features:

## Step 1 — Claude

```text
Plan implementation for wall intersection splitting.

Do not code.

Give:
- domain changes
- algorithms
- affected files
- edge cases
- tests
- migration needs
```

## Step 2 — You

Review and approve.

## Step 3 — Codex

```text
Implement the approved plan below...
```

## Step 4 — Claude

Review the implementation.

This separates planning from implementation and review.

---

# 20. Frontend Workflow

Recommended cycle:

```text
Codex
 ↓
Implement UI
 ↓
Run app
 ↓
Antigravity
 ↓
Browser / visual testing
 ↓
Findings
 ↓
Codex
 ↓
Fix
```

Use this for:

- onboarding,
- dashboard,
- 2D editor,
- 3D workspace,
- QS,
- BOQ,
- reports.

---

# 21. Independent Bug Diagnosis

For difficult bugs:

```text
Codex:
Diagnose without changing code.

Claude:
Diagnose independently without reading Codex's explanation.
```

Compare:

- root cause,
- proposed fix,
- risk,
- affected modules.

If both agree, confidence is higher.

---

# 22. Antigravity Regression Prompt

Reusable regression scenario:

```text
Run the BuildWise core regression suite through the browser:

1. Login
2. Create project
3. Create floor
4. Draw room
5. Add door
6. Add window
7. Switch to 3D
8. Change material
9. Open Quantities
10. Verify totals
11. Reload project
12. Verify persistence

Report:
- PASS/FAIL per step
- screenshots
- console errors
- failed API requests
- unexpected UI changes
```

---

# 23. Quality Verification Command

Create a single command:

```bash
pnpm verify
```

It should run:

```text
Formatting
ESLint
TypeScript
Unit Tests
Integration Tests
Architecture Tests
Build
```

Every agent task should end with:

```text
Run pnpm verify before considering this task complete.
```

---

# 24. Python Quality Gates

For Python services:

```text
ruff
mypy or pyright
pytest
```

---

# 25. Rust Quality Gates

For Rust/WASM:

```text
cargo fmt
cargo clippy
cargo test
```

---

# 26. Architectural Boundary Tests

Prevent AI-generated shortcuts.

Examples:

```text
UI cannot import database layer
3D engine cannot import NestJS API
QS engine cannot import OpenAI SDK
Building Model cannot import Three.js
Domain cannot import React
```

Architecture tests are especially important in AI-assisted development.

---

# 27. Dependency Direction

Enforce:

```text
             UI
              │
          Application
              │
            Domain
```

Rendering adapters:

```text
Three.js ──→ Adapter
                │
          Building Model

PixiJS ───→ Adapter
                │
          Building Model
```

Do not allow:

```text
Building Model
      ↓
Three.js
```

The domain must remain independent of rendering libraries.

---

# 28. Delegate Aggressively

Good tasks to delegate:

- boilerplate,
- CRUD,
- migrations,
- validation,
- tests,
- API clients,
- component implementation,
- docs,
- repetitive refactors,
- fixtures,
- CI,
- browser regression.

---

# 29. Avoid Overbuilding Infrastructure

Do not start with:

```text
Kubernetes
20 microservices
Kafka
service mesh
multi-region
custom auth
custom billing
custom vector database
```

Initial architecture can be:

```text
Cloudflare
    ↓
Next.js

NestJS API
    ↓
PostgreSQL
Redis
R2/S3

Python Worker
Temporal

Later:
GPU Render Worker
```

---

# 30. Initial Deployable Units

Start with roughly:

```text
web
api
worker-ai
worker-bim
```

Later add:

```text
worker-render
```

Avoid unnecessary services.

---

# 31. Feature Flags

Implement feature flags early.

Examples:

```text
AI_CONCEPT_GENERATION
IFC_IMPORT
PLAN_RECOGNITION
PHOTOREAL_RENDERING
PRO_QS
SUPPLIER_MARKETPLACE
```

This allows incomplete modules to stay safely in the codebase.

---

# 32. Use Fake Adapters First

Before expensive integrations, use fakes.

Examples:

```text
FakeAIProvider
StaticRateProvider
MockSupplierProvider
LocalRenderProvider
```

Benefits:

- faster development,
- deterministic tests,
- lower cost,
- easier debugging.

---

# 33. Keep the First QS Dataset Small

Do not begin with a global construction database.

MVP target:

```text
1 country
1 currency
1 residential construction method
20–50 materials
10–20 assemblies
```

Example assemblies:

```text
External Block Wall
Internal Block Wall
Concrete Slab
Tile Floor
Painted Plaster Wall
Simple Pitched Roof
```

---

# 34. Keep Initial BIM Scope Small

Start with:

```text
Wall
Door
Window
Slab
Room
Level
```

Then add IFC import/export.

Do not attempt complete BIM coverage immediately.

---

# 35. Keep Initial AI Tool Scope Small

Version 1 Copilot tools:

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

A few reliable tools are better than dozens of unreliable tools.

---

# 36. Recommended Daily Development Loop

```text
START

You:
Choose ONE deliverable

        ↓

Claude:
Review requirements / plan difficult parts

        ↓

You:
Approve approach

        ↓

Codex:
Implement

        ↓

Codex:
Tests + verify

        ↓

Claude:
Review diff

        ↓

Codex:
Fix findings

        ↓

Antigravity:
Browser regression if UI affected

        ↓

You:
Manual sanity check

        ↓

Commit / Merge

END
```

---

# 37. Avoid Context Switching

Bad day:

```text
2D
AI
billing
3D
marketing
QS
auth
```

Better:

```text
Today:
Wall domain + wall editor only.
```

Then:

```text
Tomorrow:
Door domain + door editor.
```

Keep repository context focused.

---

# 38. Keep Commits Small

Recommended examples:

```text
feat(model): add wall aggregate
feat(editor): render wall entities
feat(editor): add wall selection
test(editor): cover endpoint snapping
```

Avoid massive unreviewed AI-generated diffs.

---

# 39. Agent Task Size

## Small

```text
1–5 files
```

Can be implemented autonomously.

## Medium

```text
5–15 files
```

Require planning first.

## Large

```text
15+ files / schema changes / architecture changes
```

Split into multiple tasks.

Never assign:

> Implement BIM support.

---

# 40. Backlog Management

Use GitHub Issues.

Suggested milestones:

```text
M0 Architecture/Foundation
M1 Projects
M2 Building Model
M3 2D
M4 3D
M5 QS
M6 BOQ/Cost
M7 AI
M8 Recognition
M9 Beta
```

Each issue should have:

- priority,
- milestone,
- domain,
- complexity,
- architecture docs,
- acceptance criteria.

---

# 41. Permanent Test Fixtures

Create:

```text
fixtures/
 ├── single-room/
 ├── two-room/
 ├── two-storey/
 ├── invalid-overlap/
 ├── door-window/
 └── qs-basic-house/
```

Every release should validate:

```text
Building Model
↓
2D
↓
3D
↓
QS
↓
BOQ
↓
Cost
```

---

# 42. Golden Calculation Tests

Example:

```text
Wall:
5000 × 2700 × 200

Opening:
1000 × 2100

Expected Net Wall Area:
11.4 m²
```

QS logic must match deterministic expected output.

---

# 43. Visual Regression Tests

Later:

```text
fixture
 ↓
render
 ↓
screenshot
 ↓
compare baseline
```

Useful for:

- 2D editor,
- selection states,
- room labels,
- confidence overlays,
- 3D views,
- dashboards.

---

# 44. Use AI to Generate Edge Cases

For every feature, ask a second agent:

```text
What could break this implementation?
```

Ask for:

- edge cases,
- malformed input,
- concurrency,
- invalid geometry,
- user errors,
- security issues,
- numerical precision problems.

---

# 45. Security

Do not commit:

```text
.env
API keys
database credentials
OpenAI keys
Stripe secrets
AWS credentials
```

Keep:

```text
.env.example
```

Use sandboxing/isolation where possible for autonomous agents.

---

# 46. Development Subscriptions Are Not Production Infrastructure

Claude Pro, Codex Plus, and Antigravity Pro are:

> **development tools**

Production BuildWise AI must use:

```text
OpenAI API
and/or
other provider APIs
```

through the BuildWise AI Gateway.

Do not build production logic around personal subscription quotas.

---

# 47. Use Multiple Models for Independent Review

For important architecture questions:

```text
Codex Review
Claude Review
Antigravity Review if relevant
```

Compare:

```text
Agreement
Disagreement
Risk
Complexity
```

You make the final decision.

---

# 48. Avoid AI Ping-Pong

Do not repeatedly pass answers between models without making decisions.

Use clear states:

```text
PROPOSED
APPROVED
IMPLEMENTED
REVIEWED
MERGED
```

Once approved, do not reopen a decision unless new evidence appears.

---

# 49. Architecture Decision Records

Continue adding ADRs.

Examples:

```text
ADR-013 Canonical Geometry Unit = mm
ADR-014 Wall Intersection Strategy
ADR-015 Room Topology Model
ADR-016 Geometry Worker Boundary
```

When an agent proposes an architecture change:

```text
Create a proposed ADR.
Do not implement yet.
```

You approve before implementation.

---

# 50. Scalability Mindset

Design for future scale but optimise initial implementation for:

```text
100 users
↓
1,000
↓
10,000
```

Do not optimise every decision for:

```text
1,000,000 users on day one
```

The chosen architecture already gives a good path to scale.

---

# 51. Suggested First 90 Development Sessions

Approximate guidance:

| Sessions | Focus |
|---:|---|
| 1–10 | Repo / CI / auth / projects |
| 11–25 | Building Model |
| 26–45 | 2D editor |
| 46–55 | 3D engine |
| 56–65 | 2D ↔ 3D sync |
| 66–72 | Materials |
| 73–80 | QS |
| 81–86 | BOQ / cost |
| 87–90 | First AI tools |

The exact session count can vary.

The sequence matters more than the numbers.

---

# 52. First Alpha Scope

The first alpha does not need:

- photoreal rendering,
- mobile,
- structural AI,
- MEP,
- supplier marketplace,
- scheduling,
- GIS.

It needs:

```text
Create House
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
```

If this works reliably, BuildWise has a strong core.

---

# 53. First AI Alpha

Initial BuildWise AI should support:

- room questions,
- quantity questions,
- cost questions,
- document questions,
- simple proposed geometry changes.

Do not start with:

> AI architect generates fully approved construction drawings.

---

# 54. Add Plan Recognition After Core Engines

Only after:

```text
Building Model
2D
3D
QS
Cost
```

are stable.

Then:

```text
PDF
 ↓
Plan Recognition
 ↓
Verification
 ↓
BuildWise Building Model
```

---

# 55. Add AI Generation After That

After the core model is stable:

```text
Natural Language
 ↓
Structured Brief
 ↓
Constraints
 ↓
Building Model
 ↓
2D / 3D
 ↓
QS / Cost
```

This makes AI generation commercially meaningful.

---

# 56. Think in Terms of Six Core Engines

BuildWise is not mainly a collection of screens.

It is six major engines:

```text
1. BUILDING MODEL
2. 2D ENGINE
3. 3D ENGINE
4. QS ENGINE
5. COST ENGINE
6. AI ORCHESTRATION ENGINE
```

Everything else supports these.

---

# 57. Operating Model Summary

```text
                         YOU
                    CTO / PRODUCT
                         │
              Architecture Decisions
                         │
       ┌─────────────────┼─────────────────┐
       │                 │                 │
    CLAUDE             CODEX          ANTIGRAVITY
       │                 │                 │
    Design              Build             Test
    Review              Refactor          Browser
    Debug               Tests             Visual QA
    Challenge           Integrate         Regression
       │                 │                 │
       └─────────────────┼─────────────────┘
                         │
                     GIT / CI
                         │
                    BUILDWISE
```

---

# 58. Final Rule

Keep this principle visible throughout development:

> **One architecture. One source of truth. One small feature at a time. One agent implements. Another verifies. Nothing merges without tests.**

This is the recommended operating model for developing BuildWise as a one-person AI-assisted SaaS.
