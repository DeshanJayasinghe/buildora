# Buildora AI — Phase-Wise Development Checklist

> **File name:** `Buildora_AI_Phase_Wise_Development_Checklist.md`
>
> **Purpose:** Master execution checklist for Buildora AI development.
>
> **Development model:** One-person SaaS development assisted by Codex Plus, Claude Pro, and Antigravity Pro.
>
> **Use together with:**
>
> - `Buildora_AI_Sprintwise_Project_Plan.md`
> - `15_Database_and_Data_Architecture.md`
> - `24_Billing_AI_Credits_and_Usage.md`
> - `Buildora_AI_Backend_Engineering_Standards.md`
> - `Buildora_AI_Frontend_Engineering_Standards.md`
> - `Buildora_AI_Sample_Backend_CRUD.md`
> - `Buildora_AI_Sample_Frontend_CRUD.md`
> - Buildora AI architecture pack under `docs/architecture/`
>
> **Rule:** Do not move to the next phase merely because coding has started. Move only when the current phase exit criteria are satisfied.

---

# 1. How to Use This Checklist

This document is the **master development control checklist**.

The sprint plan tells you:

```text
WHAT TO BUILD
```

This checklist tells you:

```text
WHAT MUST BE TRUE
BEFORE YOU MOVE FORWARD
```

Each phase includes:

- objectives,
- architecture checks,
- implementation tasks,
- testing,
- security,
- observability,
- documentation,
- acceptance,
- exit gate.

Recommended status notation:

```text
[ ] Not started
[-] In progress
[x] Complete
[!] Blocked / decision required
```

Do not mark an item complete unless it is actually verified.

---

# 2. Master Phase Overview

| Phase | Name | Main Outcome |
|---:|---|---|
| **0** | **Product & Architecture Lock** | **COMPLETE 2026-09-12 — architecture frozen, 22 ADRs accepted** |
| 1 | Developer Environment | Reproducible machine setup |
| 2 | Repository & Engineering Foundation | Monorepo, standards, CI |
| 3 | SaaS Identity & Tenancy | Users, orgs, permissions |
| 4 | Project Management Foundation | Project CRUD and dashboard |
| 5 | Canonical Building Model | Core domain source of truth |
| 6 | 2D CAD Foundation | Usable plan editor |
| 7 | 3D Engine | Building Model → 3D |
| 8 | 2D ↔ 3D Synchronisation | Unified model experience |
| **8b** | **3D Walkthrough & Saved Views** | **Walk inside the building — zero AI** |
| 9 | Materials & Assemblies | Construction meaning |
| 10 | Quantity Surveying | Deterministic quantities |
| 11 | BOQ & Costing | Commercial intelligence |
| **11b** | **Reports & Exports (moved earlier)** | **FIRST SELLABLE MILESTONE — deterministic Model-to-Cost** |
| **11c** | **FF&E Catalogue & Manual Furnishing** | **Place and cost furniture — zero AI** |
| 12 | AI Platform Foundation | Gateway, routing, tools |
| 13 | AI Copilot & Change Actions | Controlled AI actions |
| **13b** | **AI Interior Assistant** | **Furnishing and style — on top of the deterministic layer** |
| 14 | File Upload & Recognition | PDF/image → verified model |
| 15 | BIM / IFC | Professional interoperability |
| 16 | Documents & RAG | Project-aware document AI |
| 17 | Billing & Usage | Monetisation, credits, **term models + archive** |
| 18 | Reports & Exports | Professional deliverables |
| 19 | Beta Readiness | External user readiness |
| 20 | Production Launch | Paid SaaS release |
| 21 | Post-Launch Operations | Monitoring and iteration |
| 22 | Professional Expansion | Scheduling, procurement, site |
| 23 | Enterprise & Scale | Large customer readiness |

---

# 2.1 Contract Gates — Hard Blockers

Three decisions freeze an *outcome* while leaving the *semantics that produce it*
unspecified. Each must be published **before** its owning phase ships. Closing a
gate is a design deliverable — written, reviewed, fixture-tested — not code.

| Gate | Contract | Blocks |
|---|---|---|
| **CG-1** | Room/junction topology determinism — tolerances, junction classification and ownership, variable thickness, open boundaries, transient invalid states, **split/merge/delete room identity** | **Phase 6** (rooms in 2D) |
| **CG-2** | Session reconciliation determinism — accepted-prefix, preconditions, rebase, invalid-suffix quarantine, conflict UX, **durable pending queue**; undo as a new command, never a blind inverse | **Phase 5–6** (persistence, optimistic editing) |
| **CG-3** | QS rule contract — versioned declarative rules over versioned executable primitives, both hashed; rounding order; conformance fixtures. Plus complete cost lineage columns | **Phase 10–11** (quantities, cost) |
| **CG-4** | MVP geometry envelope — publish what the model supports; **reject unsupported input explicitly** | **Phase 5–6** |

**A phase that depends on an open gate does not start.**

---

# 3. Phase 0 — Product & Architecture Lock

> **STATUS: COMPLETE — 2026-09-12.**
>
> The foundational architecture is **frozen**. Baseline:
> `docs/architecture/FINAL_SYSTEM_ARCHITECTURE.md`. Decisions: ADR-001 … ADR-022
> in `docs/architecture/34_ADR_Index.md`.
>
> Completed in this phase:
> - `docs/README.md` created as the documentation router
> - duplicated setup pack removed from `docs/architecture/`
> - all documents renamed BuildWise → Buildora AI
> - canonical architecture documents 03, 04, 05, 06, 07 written
> - 22 ADRs accepted; ADR index and template created
> - `FINAL_SYSTEM_ARCHITECTURE.md` published with the freeze statement
> - `AGENTS.md` and `CLAUDE.md` updated to the frozen architecture
> - ID strategy corrected in the CRUD examples
> - `packages/domain` removed; `packages/model-session` added
>
> **Implementation agents must not relitigate frozen decisions without a
> superseding ADR.**


## Objective

Ensure all major architectural decisions are written before implementation expands.

## Product Definition

- [ ] Product name/working name confirmed.
- [ ] Product positioning documented.
- [ ] Core value proposition documented.
- [ ] Target users documented.
- [ ] MVP scope documented.
- [ ] Explicit out-of-scope list created.
- [ ] Commercial launch scope documented.
- [ ] Post-launch features separated from MVP.

## Core Product Flow

Confirm:

```text
Idea / Prompt / Upload
        ↓
Building Model
        ↓
2D
        ↓
3D
        ↓
Materials
        ↓
QS
        ↓
BOQ
        ↓
Cost
        ↓
AI Copilot
        ↓
Reports
```

Checklist:

- [ ] Single-source Building Model rule approved.
- [ ] IFC declared interchange format, not source of truth.
- [ ] 2D and 3D declared derived/adapted views.
- [ ] QS declared deterministic.
- [ ] Cost declared deterministic.
- [ ] AI mutation declared ChangeSet-based.
- [ ] AI cannot directly mutate live model.

## Technology Decisions

- [ ] Next.js selected for web.
- [ ] NestJS selected for core API.
- [ ] PostgreSQL selected as main database.
- [ ] Drizzle selected as ORM/migration layer.
- [ ] Redis selected for ephemeral/cache needs.
- [ ] PixiJS selected for 2D.
- [ ] Three.js selected for 3D.
- [ ] Python/FastAPI selected for AI/BIM processing.
- [ ] Rust/WASM reserved for geometry hotspots.
- [ ] Temporal selected for durable workflows.
- [ ] R2/S3 selected for object storage.
- [ ] Stripe selected for billing.
- [ ] pgvector selected for semantic search.
- [ ] PostGIS selected for site/GIS.
- [ ] OpenTelemetry/Sentry approach documented.

## Coding Standards

- [ ] Backend standards document added.
- [ ] Frontend standards document added.
- [ ] Backend CRUD reference added.
- [ ] Frontend CRUD reference added.
- [ ] Database architecture added.
- [ ] Billing architecture added.
- [ ] Sprint plan added.
- [ ] AGENTS.md references standards.
- [ ] CLAUDE.md references standards.

## ADRs

Create initial ADRs for:

- [ ] Building Model as source of truth.
- [ ] Canonical geometry unit = millimetres.
- [ ] PostgreSQL as primary database.
- [ ] Hybrid relational + JSONB element persistence.
- [ ] AI ChangeSet approval model.
- [ ] Customer credits vs raw token billing.
- [ ] Organization as billing/tenant boundary.

## Phase 0 Exit Gate

Do not continue until:

- [ ] Architecture documents are stored in repository.
- [ ] No major stack decision remains ambiguous.
- [ ] MVP scope is limited and explicit.
- [ ] AI agents are instructed to use architecture docs.

---

# 4. Phase 1 — Developer Environment

## Objective

Ensure the machine can reproduce the Buildora AI environment cleanly.

## Core Tools

- [ ] Git installed.
- [ ] GitHub access configured.
- [ ] SSH key configured.
- [ ] Node 24 LTS installed.
- [ ] pnpm installed.
- [ ] Docker Desktop installed.
- [ ] GitHub CLI installed.
- [ ] jq installed.
- [ ] ripgrep installed.
- [ ] tree installed.

## Verification

Run:

```bash
git --version
node --version
pnpm --version
docker --version
docker compose version
gh --version
```

Checklist:

- [ ] All commands work.
- [ ] Node major matches repository requirement.
- [ ] Docker can run hello-world.
- [ ] GitHub SSH test passes.
- [ ] GitHub CLI authenticated.

## AI Development Tools

- [ ] Codex configured.
- [ ] Claude Code configured.
- [ ] Antigravity configured.
- [ ] Agents can access repository.
- [ ] Agents are told not to edit same branch concurrently.
- [ ] Worktree workflow tested.

## Optional Later Tools

Do not block Phase 1 on:

- [ ] Python/uv.
- [ ] Rust.
- [ ] Temporal CLI.
- [ ] Blender.
- [ ] BIM tooling.

These are installed when their phase starts.

## Phase 1 Exit Gate

- [ ] Fresh terminal can run all required core tools.
- [ ] Git push/pull works.
- [ ] Docker local services can run.
- [ ] AI agents can safely work in repository.

---

# 5. Phase 2 — Repository & Engineering Foundation

## Objective

Create a clean, maintainable monorepo with quality gates.

## Repository

- [ ] Private GitHub repository created.
- [ ] `main` branch established.
- [ ] Branch naming convention defined.
- [ ] Worktree folder convention documented.
- [ ] Conventional Commit style documented.

## Monorepo

Create:

```text
apps/
packages/
native/
infrastructure/
docs/
```

Checklist:

- [ ] pnpm workspace configured.
- [ ] Turborepo configured.
- [ ] root package.json configured.
- [ ] `.nvmrc` created.
- [ ] `.editorconfig` created.
- [ ] `.gitignore` created.
- [ ] `.env.example` created.

## Core Apps

- [ ] `apps/web` created.
- [ ] `apps/api` created.
- [ ] Next.js runs.
- [ ] NestJS runs.
- [ ] ports do not conflict.

Suggested:

```text
web → 3000
api → 4000
```

## Initial Packages

- [ ] `packages/api-contracts`
- [ ] `packages/units`
- [ ] `packages/design-system`

Created in Phase 5 / Phase 6 when their consumers exist:

- [ ] `packages/building-model` (Phase 5)
- [ ] `packages/model-session` (Phase 6, before the 2D editor — ADR-008)

**`packages/domain` must not be created** (ADR-011).
**`packages/core` must not be created** without meeting the three conditions in
`04_Monorepo_and_Module_Architecture.md` §5.1.

Do not create empty packages for future features without need (ADR-012).

## Code Quality

- [ ] TypeScript strict enabled.
- [ ] ESLint configured.
- [ ] Prettier/formatting policy defined.
- [ ] Vitest configured.
- [ ] root `pnpm verify` configured.

Target:

```bash
pnpm lint
pnpm typecheck
pnpm test
pnpm build
pnpm verify
```

## Local Infrastructure

- [ ] PostgreSQL Docker service.
- [ ] Redis Docker service.
- [ ] health checks configured.
- [ ] local reset procedure documented.

## CI

- [ ] GitHub Actions created.
- [ ] install job works.
- [ ] lint job works.
- [ ] typecheck job works.
- [ ] test job works.
- [ ] build job works.
- [ ] PostgreSQL service configured for integration tests.
- [ ] Redis service configured where required.

## Database

- [ ] Drizzle installed.
- [ ] migration config created.
- [ ] empty migration works.
- [ ] DB connectivity tested.
- [ ] no production credentials used locally.

## Observability Baseline

- [ ] request ID middleware designed/implemented.
- [ ] structured logger selected.
- [ ] `/health/live` created.
- [ ] `/health/ready` created or planned.

## Phase 2 Exit Gate

- [ ] Fresh clone builds.
- [ ] Docker services start.
- [ ] migrations run.
- [ ] CI passes.
- [ ] `pnpm verify` passes.
- [ ] no secrets committed.
- [ ] architecture standards linked from agent docs.

---

# 6. Phase 3 — SaaS Identity & Tenancy

## Objective

Create secure user and organization boundaries.

## Authentication

- [ ] Auth provider configured.
- [ ] Sign up works.
- [ ] Sign in works.
- [ ] Sign out works.
- [ ] Password/recovery flow works if applicable.
- [ ] Protected application routes implemented.
- [ ] API verifies identity server-side.
- [ ] Local user mirror created.
- [ ] Auth provider IDs are not primary domain IDs.

## Users

Database:

- [ ] `users`
- [ ] `auth_identities`
- [ ] user status
- [ ] locale/timezone baseline

## Organizations

> **ADR-021 — binding.** Every user gets a **personal organization**
> (`kind = PERSONAL`) auto-created at signup. **No user-owned project path exists.**
> Tenancy has exactly one shape (ADR-016).

- [ ] `organizations.kind` = `PERSONAL | TEAM`.
- [ ] personal organization auto-created at signup, user as sole owner.
- [ ] `PERSONAL → TEAM` conversion is a rename + invite; **no project row moves**.
- [ ] test: signup creates exactly one personal organization.
- [ ] test: every project resolves to exactly one organization.

- [ ] `organizations` table.
- [ ] create organization.
- [ ] switch organization.
- [ ] update organization.
- [ ] organization status.
- [ ] organization settings.

## Membership

- [ ] `organization_members`.
- [ ] owner.
- [ ] admin.
- [ ] member.
- [ ] viewer.
- [ ] unique membership constraint.

## Invitations

- [ ] invite member.
- [ ] invitation expiration.
- [ ] invitation acceptance.
- [ ] revoked invitations.
- [ ] role selection.

## Authorization

- [ ] authentication separated from authorization.
- [ ] server-side role policy.
- [ ] tenant context resolved from verified session.
- [ ] no tenant ID trusted from request body.

## Security Tests

- [ ] Org A cannot access Org B membership.
- [ ] Viewer cannot perform admin operations.
- [ ] Invalid/expired session rejected.
- [ ] Organization switching cannot leak previous tenant data.

## Audit

Record:

- [ ] login/security events as appropriate.
- [ ] organization creation.
- [ ] invite.
- [ ] member addition/removal.
- [ ] role changes.

## Phase 3 Exit Gate

- [ ] Multi-tenant identity works.
- [ ] cross-tenant tests pass.
- [ ] role rules pass.
- [ ] audit baseline works.
- [ ] API never trusts client organization ownership.

---

# 7. Phase 4 — Project Management Foundation

## Objective

Build the first useful SaaS workspace.

## Database

- [ ] `projects`
- [ ] `project_settings`
- [ ] project lifecycle status
- [ ] organization ownership
- [ ] created-by audit fields

## Project CRUD

- [ ] create.
- [ ] list.
- [ ] read.
- [ ] update.
- [ ] archive.
- [ ] restore.
- [ ] delete request flow.

## Project Permissions

- [ ] organization-scoped repository queries.
- [ ] optional project-member model decision.
- [ ] delete restricted to owner/admin.
- [ ] viewer restrictions.

## Project Create Screen

Options displayed:

- [ ] AI Brief.
- [ ] Upload Plan.
- [ ] Draw Manually.
- [ ] Import BIM.

Only enabled options need to function initially.

## Dashboard

- [ ] recent projects.
- [ ] active projects.
- [ ] archived projects.
- [ ] create project CTA.
- [ ] empty state.
- [ ] loading state.
- [ ] error state.

## CRUD Standards

- [ ] backend follows sample CRUD pattern.
- [ ] frontend follows sample CRUD pattern.
- [ ] RFC 9457 errors.
- [ ] Zod contracts.
- [ ] query/mutation hooks.
- [ ] accessibility.

## Tests

- [ ] project CRUD unit tests.
- [ ] tenant isolation.
- [ ] role tests.
- [ ] E2E create/edit/archive.
- [ ] delete workflow tests.

## Phase 4 Exit Gate

A new user can:

```text
Sign in
→ Create organization
→ Create project
→ Open project
→ Edit metadata
→ Archive/restore
```

without developer intervention.

---

# 8. Phase 5 — Canonical Building Model

## Objective

Create the core domain that every future feature depends on.

## Units

- [ ] millimetre canonical length type.
- [ ] area conversions.
- [ ] volume conversions.
- [ ] angle type.
- [ ] unit display conversion.
- [ ] no screen pixels in domain geometry.

## Domain Root

- [ ] Project Model.
- [ ] Site placeholder (real-world georeference only — kept separate from
      building-local coordinates).
- [ ] Building.
- [ ] Level with `elevationMm` as the **single authoritative vertical fact**
      (ADR-005).
- [ ] Building Element base.
- [ ] stable UUID IDs (ADR-004).

## Datum — ADR-005 (locked)

- [ ] project-local datum: ground-floor FFL = 0 mm.
- [ ] `Level.elevationMm` signed from project datum.
- [ ] element z **level-relative**.
- [ ] `absoluteZ = level.elevationMm + (baseOffsetMm ?? 0)` — derived, never stored.
- [ ] `floorToFloor` derived from adjacent elevations — **never stored**.
- [ ] slab structural thickness on the Slab/assembly, **not** on Level.
- [ ] test: element on a negative (basement) level resolves correctly.
- [ ] test: moving a level moves its contents with no element writes.

## Element Class — ADR-019 (from the first migration)

- [ ] `element_class` on every element: `BUILDING | SPACE | FFE`.
- [ ] `ffe_asset_version_id` nullable column present (FF&E built in Phase 11c).
- [ ] `element_type_id` nullable column present (ADR-007).
- [ ] provenance set: `source_kind` · `source_ref` · `confidence` ·
      `verification_status`.
- [ ] partial index `(project_id, element_class)` on live rows.

> The class column ships **now**, before FF&E exists. Adding it after elements
> exist means reclassifying every row and re-auditing every quantity query.

## CG-4 — MVP Geometry Envelope (publish before wall geometry is relied upon)

- [ ] document what the canonical model **supports**: straight-centreline walls
      with scalar thickness, polygon-plus-plane roofs.
- [ ] document what it **cannot express**: curved walls, thickness varying along
      a wall, vaulted ceilings independent of the roof, multi-plane roofs.
- [ ] **reject unsupported input explicitly** rather than segmenting it into fake
      elements with fake junctions.

## Initial Elements

- [ ] Wall.
- [ ] Door.
- [ ] Window.
- [ ] Slab.
- [ ] Space/Room.

## Wall

- [ ] start.
- [ ] end.
- [ ] height.
- [ ] thickness.
- [ ] level.
- [ ] assembly placeholder.
- [ ] validation.

## Door

- [ ] host wall.
- [ ] offset.
- [ ] width.
- [ ] height.
- [ ] swing.
- [ ] validation.

## Window

- [ ] host wall.
- [ ] offset.
- [ ] width.
- [ ] height.
- [ ] sill.
- [ ] validation.

## Commands

- [ ] CreateLevel.
- [ ] CreateWall.
- [ ] UpdateWall.
- [ ] DeleteWall.
- [ ] CreateDoor.
- [ ] UpdateDoor.
- [ ] CreateWindow.
- [ ] UpdateWindow.
- [ ] DeleteElement.

## Persistence

- [ ] relational ownership fields.
- [ ] schema-versioned geometry JSONB.
- [ ] schema-versioned properties JSONB.
- [ ] element relationships.
- [ ] source/provenance fields.

## Versioning

- [ ] `model_versions`.
- [ ] `model_commands`.
- [ ] `model_change_items`.
- [ ] current model version.
- [ ] base-version checks.
- [ ] optimistic concurrency.

## Snapshots

- [ ] snapshot strategy documented.
- [ ] periodic snapshot implementation can wait until needed.
- [ ] architecture supports it.

## Undo / Redo

- [ ] command/inverse approach approved.
- [ ] deterministic undo.
- [ ] deterministic redo.
- [ ] model version updated appropriately.

## Tests

- [ ] wall validation.
- [ ] door hosting.
- [ ] window hosting.
- [ ] serialization.
- [ ] deserialization.
- [ ] version conflict.
- [ ] command replay.
- [ ] save/reload identical.

## Architecture Check

Confirm domain imports none of:

- [ ] React.
- [ ] PixiJS.
- [ ] Three.js.
- [ ] NestJS.
- [ ] Drizzle.
- [ ] OpenAI.

## Phase 5 Exit Gate

A headless test can:

```text
Create building
→ Add level
→ Add walls
→ Add door/window
→ Serialize
→ Persist
→ Reload
→ Compare exact domain result
```

and pass.

---

# 9. Phase 6 — 2D CAD Foundation

> ## ⚠ Gates CG-1 and CG-2 must be closed before this phase ships
>
> **CG-1** — room/junction topology determinism, before any room is persisted.
> **CG-2** — session reconciliation determinism, before optimistic editing.
> See §2.1. A phase depending on an open gate does not start.

## Objective

Create a reliable plan editor on top of the Building Model.

## PixiJS Engine

- [ ] package created.
- [ ] app lifecycle.
- [ ] renderer.
- [ ] resize.
- [ ] cleanup.

## Coordinate Systems

- [ ] model mm.
- [ ] screen px.
- [ ] camera transform.
- [ ] zoom conversion.
- [ ] coordinate helpers tested.

## Camera

- [ ] pan.
- [ ] zoom.
- [ ] fit plan.
- [ ] grid.
- [ ] origin/reference.

## Rendering

- [ ] walls.
- [ ] doors.
- [ ] windows.
- [ ] slabs.
- [ ] room labels.

## Selection

- [ ] element hit testing.
- [ ] selection highlight.
- [ ] multi-select decision.
- [ ] selected element property panel.

## Drawing Tools

- [ ] wall tool.
- [ ] door tool.
- [ ] window tool.
- [ ] cancel tool.
- [ ] delete selected.

## Snapping

Initial:

- [ ] endpoint.
- [ ] grid.
- [ ] orthogonal.

Later:

- [ ] midpoint.
- [ ] perpendicular.
- [ ] intersection.

## Editing

- [ ] move wall.
- [ ] move endpoint.
- [ ] exact length.
- [ ] property edit.
- [ ] hosted opening moves correctly.

## Rooms — ADR-006 (locked)

Topology lives in `packages/building-model`, **not** in the editor.

- [ ] wall centreline graph for **connectivity** (enclosed-region detection).
- [ ] junction resolution.
- [ ] **interior wall face** derivation.
- [ ] usable room boundary from interior faces — **not** from centrelines.
- [ ] persist Room/Space with **stable ID**.
- [ ] `BOUNDS` relationships to bounding walls.
- [ ] `user_defined` override flag.
- [ ] `user_defined = false` → recompute on wall change; preserve ID, name,
      number, finishes; flag material area change.
- [ ] `user_defined = true` → **never silently overwritten**.
- [ ] **area computed from boundary** — user-entered area is never authoritative.
- [ ] rooms on one level do not overlap.
- [ ] room naming and numbering.
- [ ] room label rendering.

### Golden test — required

```text
5000 × 4000 mm room bounded by 200 mm walls
centreline area  = 20.00 m²   ← WRONG
interior faces   = 4800 × 3800 = 18.24 m²   ← REQUIRED RESULT
```

- [ ] golden test asserts **18.24 m²**.

## Openings — locked

- [ ] a door or window **IS** its opening — no second `Opening` element.
- [ ] standalone `opening` only for fixture-less holes (penetration, arch).
- [ ] QS reads hosted elements and standalone openings through **one**
      deduction abstraction.

## Levels

- [ ] floor selector.
- [ ] add level.
- [ ] rename level.
- [ ] level elevation.
- [ ] visibility.

## Keyboard

- [ ] Esc.
- [ ] Delete.
- [ ] Undo.
- [ ] Redo.
- [ ] shortcut registry.

## Autosave

- [ ] meaningful command boundaries.
- [ ] drag preview does not write every mousemove.
- [ ] saving state UI.
- [ ] save failure state.
- [ ] version conflict state.

## Tests

- [ ] wall rendering.
- [ ] coordinate transformations.
- [ ] snapping.
- [ ] selection.
- [ ] command output.
- [ ] undo/redo.
- [ ] reload.
- [ ] screenshot regression baseline.

## Phase 6 Exit Gate

User can reliably:

```text
Create blank project
→ Draw four walls
→ Add door
→ Add window
→ Create room
→ Save
→ Reload
```

and result remains correct.

---

# 10. Phase 7 — 3D Engine

## Objective

Render the same Building Model as an editable/inspectable 3D experience.

## Three.js Engine

- [ ] dedicated package.
- [ ] scene.
- [ ] renderer.
- [ ] camera.
- [ ] lighting.
- [ ] resize.
- [ ] resource disposal.

## Unit Mapping

- [ ] documented mm → world conversion.
- [ ] all adapters use same conversion.

## Element Adapters

- [ ] wall mesh.
- [ ] slab mesh.
- [ ] door representation.
- [ ] window representation.
- [ ] levels/building grouping.

## Navigation

- [ ] orbit.
- [ ] pan.
- [ ] zoom.
- [ ] fit model.
- [ ] top view.
- [ ] isometric.
- [ ] walkthrough baseline.

## Selection

- [ ] raycasting.
- [ ] domain ID mapping.
- [ ] selected object highlight.
- [ ] property inspector.

## Visibility

- [ ] floor isolation.
- [ ] element hide/show.
- [ ] material preview.
- [ ] section cut baseline.

## Performance

- [ ] renderer does not rebuild scene every React render.
- [ ] mesh cache strategy.
- [ ] geometry/material disposal.
- [ ] memory leak checks.
- [ ] representative fixture FPS measured.

## Tests

- [ ] mesh dimensions.
- [ ] domain ID mapping.
- [ ] update after element change.
- [ ] disposal.
- [ ] multi-level fixture.

## Phase 7 Exit Gate

The simple test house appears dimensionally correct in 3D and selected objects reference the same domain IDs used in 2D.

---

# 11. Phase 8 — 2D ↔ 3D Synchronisation

## Objective

Prove the architecture's core source-of-truth model.

## Synchronisation

- [ ] edit wall in 2D.
- [ ] Building Model command committed.
- [ ] model version advances.
- [ ] 2D refreshes.
- [ ] 3D refreshes.

## 3D Property Edit

Where supported:

- [ ] edit material/property in 3D.
- [ ] domain model changes.
- [ ] 2D reflects result.

## Persistence

- [ ] save.
- [ ] close.
- [ ] reopen.
- [ ] 2D same.
- [ ] 3D same.

## Conflict

- [ ] two-session stale-version test.
- [ ] frontend displays useful conflict.
- [ ] no lost update.

## Regression

- [ ] create.
- [ ] edit.
- [ ] delete.
- [ ] undo.
- [ ] redo.
- [ ] switch workspaces.
- [ ] reload.

## Phase 8 Exit Gate

There is exactly **one authoritative building state**.

No independent 2D/3D copies exist.

---

# 11b. Phase 8b — 3D Walkthrough & Saved Views

## Objective

Let a user walk inside their building. **Zero AI.**

## Walkthrough

- [ ] first-person camera (W/S/A/D + mouse look, Shift faster, Esc exit).
- [ ] **wall collision** — cannot walk through walls.
- [ ] floor detection and stair traversal.
- [ ] configurable eye height (default 1600–1700 mm).
- [ ] leaving and re-entering loses no model state.

## Saved views

- [ ] `saved_views` — camera, visibility, section planes, walkthrough position.
- [ ] records `model_version` so a stale view can be **flagged**.
- [ ] a saved view **never mutates canonical geometry**.

## Viewport state

- [ ] hide / show / isolate / hide level / hide category.
- [ ] section planes and clipping — renderer only.
- [ ] day/night lighting presets.

## Feature gating — ADR-022

- [ ] register `feature.walkthrough` in the entitlement catalogue.
- [ ] **server check site** on 3D session authorization — not a hidden button.
- [ ] denial returns `FEATURE_NOT_ENTITLED` with the code.

## Architecture check

- [ ] walkthrough controller is **renderer state**; camera and collision volumes
      are not persisted except inside a saved view.
- [ ] collision derived from canonical geometry, never hand-authored.
- [ ] **no LLM call occurs anywhere in this phase.**

## Phase 8b Exit Gate

- [ ] A user walks the test fixture without passing through walls.
- [ ] A saved view restores camera, visibility and section state exactly.

---

# 12. Phase 9 — Materials & Assemblies

## Objective

Connect design elements to construction systems.

## Materials

- [ ] system materials.
- [ ] organization materials.
- [ ] categories.
- [ ] units.
- [ ] density where needed.
- [ ] properties.
- [ ] status.
- [ ] versioning decision.

## Assemblies

- [ ] assembly definition.
- [ ] assembly versions.
- [ ] measurement basis.
- [ ] assembly items.
- [ ] primary material.
- [ ] labour placeholder.
- [ ] plant placeholder.
- [ ] waste rule.

## Initial Assembly Catalog

Create only a small verified set:

- [ ] external block wall.
- [ ] internal block wall.
- [ ] concrete slab.
- [ ] tile floor.
- [ ] painted plaster wall.
- [ ] simple roof.

## Element Assignment

- [ ] wall assembly.
- [ ] slab assembly.
- [ ] finish roles.
- [ ] assignment version/provenance.

## Finish Assignments — ADR-020

Finish is a **third identity axis**, separate from type and assembly:

```text
element_type_id       WHAT IT IS
assembly_version_id   WHAT IT'S BUILT OF
finish_assignment     WHAT IT LOOKS LIKE
```

- [ ] `finish_assignments` table — **surface-scoped** (`INTERIOR_A`,
      `INTERIOR_B`, `EXTERIOR`, `TOP`, `BOTTOM`, `SOFFIT`, `ALL`), optional `room_id`.
- [ ] `SET_ELEMENT_FINISH` command.
- [ ] unique per `(element_id, surface)` on live rows.
- [ ] materials carry **both** visual properties (colour, texture, roughness)
      **and** construction properties (unit, coverage rate, coats, waste, labour,
      rate reference).
- [ ] **repainting creates a finish assignment, never a new assembly version.**
- [ ] finish deduction rules come from the **QS ruleset** (ADR-009), never a
      domain constant.

> **Open contract — close before the finish editor ships:** the per-element
> **surface taxonomy** — which named surfaces each element type exposes, and how
> a partial-coverage finish declares its area.

## Feature gating — ADR-022

- [ ] register `feature.interior.finishes` in the entitlement catalogue.
- [ ] **server check site** on the `SET_ELEMENT_FINISH` command guard.

## UI

- [ ] materials library.
- [ ] assemblies library.
- [ ] assignment panel.
- [ ] material properties.

## Tests

- [ ] assembly version immutability.
- [ ] material references.
- [ ] organization isolation.
- [ ] assignment persistence.

## Phase 9 Exit Gate

A wall can have a versioned construction assembly without embedding calculation rules directly into the wall entity.

---

# 13. Phase 10 — Quantity Surveying

> ## ⚠ Gate CG-3 must be closed before any NRM2-branded quantity
>
> Publish the hybrid rule contract (versioned declarative rules over versioned
> executable primitives, both hashed; rounding order; conformance fixtures).
> Until the supported NRM2 work sections are audited, label output
> **"Buildora QS — NRM2 subset"**, never "NRM2-compliant". Engine

## Objective

Create deterministic, auditable quantity calculation.

## QS Engine Architecture

- [ ] no LLM dependency.
- [ ] engine version.
- [ ] rule-set version.
- [ ] quantity run version.
- [ ] traceability.

## Initial Calculations

Wall:

- [ ] length.
- [ ] gross area.
- [ ] opening deductions.
- [ ] net area.
- [ ] volume.

Slab:

- [ ] area.
- [ ] volume.

Opening:

- [ ] count.
- [ ] area.

## Assembly Expansion

- [ ] block quantity.
- [ ] cement.
- [ ] sand.
- [ ] plaster.
- [ ] paint.
- [ ] waste.

## Precision

- [ ] exact/controlled decimal behaviour.
- [ ] unit conversions standardized.
- [ ] no display formatting in calculations.

## Persistence

- [ ] quantity runs.
- [ ] quantity items.
- [ ] source links.
- [ ] calculation trace.

## Golden Tests

- [ ] 5m x 2.7m wall.
- [ ] 1m x 2.1m opening.
- [ ] expected 11.4 m² net.
- [ ] multi-opening.
- [ ] waste.
- [ ] invalid input.

## Phase 10 Exit Gate

Given identical:

```text
model version
+
assemblies
+
QS rules
```

the engine always produces the same result.

---

# 14. Phase 11 — BOQ & Costing

## Objective

Convert design quantities into professional commercial outputs.

## BOQ

- [ ] BOQ version.
- [ ] sections.
- [ ] hierarchy.
- [ ] items.
- [ ] units.
- [ ] quantity links.
- [ ] source element trace.

## Cost Rates

- [ ] rate book.
- [ ] rate book version.
- [ ] region.
- [ ] currency.
- [ ] source.
- [ ] material rate.
- [ ] labour rate.
- [ ] plant rate.
- [ ] subcontract rate.

## Cost Engine

- [ ] material cost.
- [ ] labour cost.
- [ ] plant.
- [ ] subcontract.
- [ ] transport.
- [ ] waste.
- [ ] overhead.
- [ ] profit.
- [ ] tax.
- [ ] contingency.

## Estimate Levels

- [ ] concept.
- [ ] elemental.
- [ ] detailed placeholder.

## Manual Overrides

- [ ] calculated rate retained.
- [ ] override recorded separately.
- [ ] actor recorded.
- [ ] timestamp.
- [ ] source/provenance preserved.

## Cost UI

- [ ] total.
- [ ] cost by trade.
- [ ] cost by material.
- [ ] cost by element.
- [ ] cost/m².
- [ ] source/rate view.
- [ ] override controls.

## Change Propagation

- [ ] model version changes.
- [ ] QS becomes stale/recalculates.
- [ ] BOQ becomes stale/recalculates.
- [ ] cost becomes stale/recalculates.
- [ ] previous versions remain reproducible.

## Tests

- [ ] exact money.
- [ ] cost fixture.
- [ ] rate version.
- [ ] override.
- [ ] tax.
- [ ] waste.
- [ ] stale version detection.

## Phase 11 Exit Gate

Buildora AI can deliver:

```text
Design
→ Quantities
→ BOQ
→ Cost
```

without AI.

This is the first strong commercial product milestone.

---

# 14a. Phase 11b — Reports & Exports — FIRST SELLABLE MILESTONE

> **Moved earlier** from Phase 18. Professional users judge the product by its
> output, and reports are the natural paywall boundary. Shipping the core report
> set here makes the deterministic Model-to-Cost product demonstrable and
> **sellable without any AI**.

## Objective

Complete the deterministic core so it can be sold. **Zero AI.**

## Core reports (this phase)

- [ ] BOQ export (XLSX).
- [ ] Cost estimate report (PDF).
- [ ] Quantity takeoff report with **element-level traceability**.

## Lineage — ADR-015

Every report pins and records:

- [ ] model version.
- [ ] quantity run id.
- [ ] BOQ version id.
- [ ] rate book version id.
- [ ] cost assumption set version.
- [ ] QS ruleset version.
- [ ] cost engine version.
- [ ] report template version + checksum.

## Processing

- [ ] report job (async where needed).
- [ ] artifact stored in object storage with metadata.
- [ ] **issued reports preserved as immutable checksummed artifacts.**

## Acceptance criteria

- [ ] A report recalculated from its pinned versions is **numerically identical**.
      *(Byte-identical PDFs are not guaranteed — template, fonts and locale affect
      bytes; that is why issued artifacts are preserved rather than regenerated.)*
- [ ] A report **never** recomputes historical cost with current rates.
- [ ] Quantities in a report expand to their source elements.
- [ ] The full flow works with **no AI involved**:

```text
create project → level → walls → room → door → window → 3D
  → assign assemblies → quantities → BOQ → cost → PDF/XLSX
```

## Phase 11b Exit Gate

- [ ] The deterministic Model-to-Cost product is demonstrable end to end.
- [ ] **This is the first point at which Buildora AI is sellable.**

---

# 14b. Phase 11c — FF&E Catalogue & Manual Furnishing

## Objective

Place, move and cost furniture. **Zero AI.** Follows the sellable milestone.

## Catalogue

- [ ] `ffe_assets` / `ffe_asset_versions` (tenant and global).
- [ ] GLB/GLTF in object storage with `storage_objects` metadata.
- [ ] **no binary assets in relational columns.**
- [ ] category browse, search, thumbnails.

## Placement

- [ ] `PLACE_FURNITURE` · `MOVE_FURNITURE` · `ROTATE_FURNITURE` ·
      `REMOVE_FURNITURE` · `SET_FURNITURE_FINISH`.
- [ ] snap to floor, against wall, to room centre.
- [ ] prevent placement outside a room, intersecting a wall, blocking a door swing.
- [ ] configurable circulation clearance.

## Cost

- [ ] FF&E as its own BOQ section (`boq_versions.include_ffe`).
- [ ] priced from `ffe_asset_versions.unit_cost`, **never from the mesh**.
- [ ] a construction-only estimate excludes FF&E cleanly.

## Rendering

- [ ] lazy load by `ffe_asset_version_id`; cache; **explicit disposal**.
- [ ] placeholder while loading — never block the frame.

## Feature gating — ADR-022

- [ ] register `feature.home.furnishing` in the entitlement catalogue.
- [ ] **server check site** on the `PLACE_FURNITURE` command guard.
- [ ] entitlement checked **before** any credit reservation.

## Architecture check

- [ ] FF&E carries `element_class = FFE`.
- [ ] **excluded from every construction quantity and from room topology.**
- [ ] instances reference an asset **version**, not a bare asset.
- [ ] FF&E commands share the **same** version history and undo stack.
- [ ] no furniture held only in a Three.js scene.

## Tests

- [ ] **FF&E contributes zero to wall, floor and volume quantities.**
- [ ] a catalogue price change does not alter a previously issued estimate.
- [ ] placement survives save → reload exactly.
- [ ] cross-tenant FF&E access returns nothing.

## Phase 11c Exit Gate

- [ ] Furniture placed, moved and deleted appears in version history and undoes.
- [ ] **No LLM call occurs anywhere in this phase.**

---

# 15. Phase 12 — AI Platform Foundation

## Objective

Create controlled AI infrastructure before exposing AI actions widely.

## AI Gateway

- [ ] provider abstraction.
- [ ] OpenAI provider.
- [ ] model aliases.
- [ ] router.
- [ ] request timeout.
- [ ] retries.
- [ ] provider error translation.
- [ ] usage recording.
- [ ] tracing.

## Model Aliases

- [ ] FAST.
- [ ] BALANCED.
- [ ] ADVANCED.
- [ ] EXPERT.

No product/business logic should depend directly on provider model marketing names.

## Usage Records

- [ ] AI job.
- [ ] provider call.
- [ ] provider request ID.
- [ ] input tokens.
- [ ] cached input tokens.
- [ ] cache write tokens if available.
- [ ] output tokens.
- [ ] reasoning usage where available.
- [ ] provider cost rate version.
- [ ] estimated cost.

## Tool Framework

Every tool:

- [ ] typed input.
- [ ] typed output.
- [ ] runtime validation.
- [ ] permission check.
- [ ] timeout.
- [ ] audit behaviour.
- [ ] stable name/version.

## First Read-Only Tools

- [ ] get_project_summary.
- [ ] get_room.
- [ ] get_element.
- [ ] get_quantities.
- [ ] get_boq.
- [ ] get_cost.

## Safety

- [ ] AI cannot run SQL directly.
- [ ] AI cannot alter billing.
- [ ] AI cannot calculate authoritative quantity.
- [ ] AI cannot write Building Model directly.

## Tests

- [ ] fake provider.
- [ ] routing.
- [ ] tool validation.
- [ ] provider failure.
- [ ] timeout.
- [ ] usage accounting.

## Phase 12 Exit Gate

AI can ask controlled tools for project facts, and all provider usage is measurable.

---

# 16. Phase 13 — AI Copilot & Change Actions

## Objective

Allow useful project-aware AI and controlled design modification.

## Copilot UI

- [ ] project-level chat.
- [ ] workspace context.
- [ ] selected element context.
- [ ] tool progress.
- [ ] loading/error.
- [ ] retry.

## Read-Only Questions

- [ ] project summary.
- [ ] room info.
- [ ] element details.
- [ ] quantities.
- [ ] BOQ.
- [ ] cost.

## ChangeSet Architecture

- [ ] AI ChangeSet.
- [ ] base model version.
- [ ] change items.
- [ ] schema validation.
- [ ] draft/sandbox application.
- [ ] geometry validation.
- [ ] QS impact.
- [ ] cost impact.
- [ ] warnings.

## First Mutation

- [ ] propose_move_wall.
- [ ] preview.
- [ ] approve.
- [ ] reject.
- [ ] commit through normal model command.
- [ ] undo.

## Value Engineering

- [ ] identify high-cost elements.
- [ ] propose alternative.
- [ ] deterministic savings calculation.
- [ ] display before/after.

## Tests

- [ ] stale base version.
- [ ] invalid AI payload.
- [ ] unauthorized element.
- [ ] failed geometry validation.
- [ ] rejected action.
- [ ] successful action.
- [ ] cost impact.

## Phase 13 Exit Gate

AI cannot directly mutate the live model, and every AI change can be previewed, validated, approved, audited, and undone.

---

# 16b. Phase 13b — AI Interior Assistant

## Objective

Layer AI on the deterministic interior system. **Requires Phases 9, 11c and 12.**

## Tools (typed, project-scoped)

- [ ] `get_room_geometry` · `get_room_openings` · `get_room_furniture`
- [ ] `search_furniture_catalogue` · `search_material_catalogue`
- [ ] `validate_furniture_layout` · `calculate_furnishing_cost`
- [ ] `propose_furniture_placement` · `propose_finish_change`

## Capabilities

- [ ] "Furnish this living room in a modern style under £4,000."
- [ ] "Make this room warmer." → finish proposals
- [ ] "Find a cheaper floor that looks similar." → recommendation + deterministic cost

## Feature gating — ADR-022

- [ ] register `feature.ai.interior` in the entitlement catalogue.
- [ ] **server check site** on AI interior tool authorization.
- [ ] entitlement checked **before** credit reservation — an unentitled request
      consumes no credit.
- [ ] `FEATURE_NOT_ENTITLED` (upgrade CTA) is distinct from
      `INSUFFICIENT_CREDITS` (top-up CTA).

## Architecture check

- [ ] AI returns a **typed ChangeSet**; it never writes `building_elements`,
      FF&E instances or `finish_assignments`.
- [ ] deterministic validation before preview: bounds · collision · clearance ·
      **then cost impact**.
- [ ] `baseVersion` **re-checked at approval**, not only at proposal.
- [ ] commits use the **same** `PLACE_FURNITURE` / `SET_ELEMENT_FINISH` commands
      as manual placement — no second write path.
- [ ] the **recommendation** may be AI-generated; the **cost is always computed
      by the cost engine**.
- [ ] AI execution context has a least-privilege DB role with **no write grant**
      on canonical model tables (ADR-013).

## Phase 13b Exit Gate

- [ ] A proposal shows affected elements and cost delta before commit.
- [ ] Rejecting a proposal changes nothing.
- [ ] A stale-version approval returns `MODEL_VERSION_CONFLICT`.
- [ ] Privilege test proves AI cannot write model tables.

---

# 17. Phase 14 — File Upload & Plan Recognition

## Objective

Turn customer drawings into Buildora AI model candidates.

## Object Storage

- [ ] local adapter.
- [ ] R2/S3 adapter.
- [ ] private objects.
- [ ] signed URLs.
- [ ] metadata registry.
- [ ] checksum.

## Upload Security

- [ ] size limit.
- [ ] content type validation.
- [ ] extension not trusted.
- [ ] file signature validation where practical.
- [ ] tenant permission.
- [ ] malware/unsafe-processing policy.

## Supported MVP Formats

- [ ] PDF.
- [ ] PNG.
- [ ] JPG/JPEG.

Later:

- [ ] DXF.
- [ ] DWG.
- [ ] IFC.

## Workflow

- [ ] upload.
- [ ] storage record.
- [ ] processing job.
- [ ] extraction.
- [ ] recognition.
- [ ] confidence.
- [ ] verification.
- [ ] conversion.

## Recognition Candidates

Initial:

- [ ] walls.
- [ ] doors.
- [ ] windows.
- [ ] rooms.
- [ ] dimensions/scale where possible.

## Confidence UI

- [ ] candidate overlay.
- [ ] confidence.
- [ ] low confidence list.
- [ ] confirm.
- [ ] reject.
- [ ] edit.
- [ ] redraw.

## Vector vs Raster

- [ ] detect vector PDF.
- [ ] preserve vector geometry where possible.
- [ ] raster/CV only when needed.

## Python Worker

- [ ] uv.
- [ ] FastAPI.
- [ ] OpenCV.
- [ ] ML dependencies only when required.
- [ ] internal-only access.

## Temporal

- [ ] durable plan workflow.
- [ ] retry.
- [ ] failure.
- [ ] resume/review wait.

## Phase 14 Exit Gate

An uploaded simple plan can become a user-verified Building Model draft without silently trusting recognition output.

---

# 18. Phase 15 — BIM / IFC

## Objective

Support professional interoperability without changing Buildora AI source of truth.

## Browser BIM

- [ ] That Open setup.
- [ ] load IFC.
- [ ] select element.
- [ ] inspect properties.
- [ ] fragments where useful.

## Server BIM

- [ ] IfcOpenShell worker.
- [ ] parser compatibility tested.
- [ ] malformed-file handling.

## Initial Mapping

- [ ] IfcBuildingStorey → Level.
- [ ] IfcWall → Wall.
- [ ] IfcDoor → Door.
- [ ] IfcWindow → Window.
- [ ] IfcSlab → Slab.
- [ ] IfcSpace → Space.

## External Mapping

- [ ] preserve IFC GUID.
- [ ] mapping status.
- [ ] unsupported entity records.

## Import Flow

- [ ] upload.
- [ ] parse.
- [ ] review.
- [ ] map.
- [ ] validate.
- [ ] convert through Building Model.

## Export

- [ ] basic supported IFC export considered.
- [ ] exported source model version recorded.

## Tests

- [ ] basic house IFC.
- [ ] multi-level IFC.
- [ ] unsupported element.
- [ ] malformed file.
- [ ] unit conversion.

## Phase 15 Exit Gate

IFC import/export is an adapter around Buildora AI rather than becoming the internal model.

---

# 19. Phase 16 — Documents & RAG

## Objective

Make project documentation available to AI with provenance.

## Documents

- [ ] documents table.
- [ ] revisions.
- [ ] version numbering.
- [ ] object storage links.
- [ ] metadata.
- [ ] processing status.

## Extraction

- [ ] text extraction.
- [ ] OCR only where needed.
- [ ] page mapping.
- [ ] chunking.
- [ ] metadata.

## pgvector

- [ ] extension enabled.
- [ ] embedding profile.
- [ ] dimension/version recorded.
- [ ] exact search initially.
- [ ] HNSW later only if needed.

## Search

- [ ] tenant filter.
- [ ] project filter.
- [ ] document revision filter.
- [ ] full-text.
- [ ] semantic.
- [ ] hybrid strategy.

## AI Tool

- [ ] search_project_documents.
- [ ] citations/provenance.
- [ ] document title.
- [ ] revision.
- [ ] page/section.

## Security

- [ ] document text treated as untrusted.
- [ ] prompt injection policy.
- [ ] no cross-tenant chunks.
- [ ] no raw confidential document logs.

## Tests

- [ ] tenant isolation.
- [ ] stale revision.
- [ ] source citation.
- [ ] irrelevant retrieval.
- [ ] no-result behaviour.

## Phase 16 Exit Gate

AI can answer project-document questions and always trace retrieved evidence back to a project document revision.

---

# 20. Phase 17 — Billing & Usage

## Runtime feature gating — ADR-022

- [ ] `feature_definitions` runtime-editable: code · name · description ·
      category · value_type · `default_value` · `is_active`.
- [ ] `plan_entitlement_overrides`: per-organization, **immediate**, with a
      required `reason`.
- [ ] resolution order: enterprise → **org override** → add-on → plan → default.
- [ ] **every gated capability checked server-side** before the work happens.
- [ ] denial returns `FEATURE_NOT_ENTITLED` carrying the feature code.
- [ ] check order **flag → entitlement → credit**; unentitled consumes no credit.
- [ ] admin: feature catalogue with **ORPHANED** badge for unwired codes.
- [ ] admin: plan editor states publishing affects **new subscribers only**.
- [ ] admin: organization view shows each entitlement's **resolved source**.
- [ ] every entitlement change audited with a reason.
- [ ] **test: a direct API call is refused for an unentitled organization.**
- [ ] test: an override takes effect immediately, without deployment.
- [ ] test: a disabled feature flag overrides a granting entitlement.

> **Hiding a button is not access control.** Any gated capability without a
> server check site is a BLOCKER.

## Term models — ADR-021

- [ ] `billing_subscriptions.term_model` = `RECURRING | FIXED_TERM`.
- [ ] `term_ends_at` (nullable) and `auto_renew` columns.
- [ ] `FIXED_TERM` ends at `term_ends_at` with **no cancellation event**.
- [ ] expiry moves the project to `archived` — **data preserved**.
- [ ] **archive entitlement set** — read-only capabilities for an expired term.
- [ ] reactivation restores full capability against the existing model.
- [ ] analytics **must not** count an expired fixed term as churn.

## Entitlements, never plan names

- [ ] feature code reads `feature.*` / `limit.*` capabilities.
- [ ] no `if (plan === "Pro")` anywhere in feature code.
- [ ] archive mode is an entitlement set, requiring no archive branch in features.


## Objective

Turn Buildora AI into a sustainable SaaS.

## Plan Catalog

- [ ] Free.
- [ ] Home.
- [ ] Pro.
- [ ] Business.
- [ ] Enterprise placeholder.
- [ ] versioned plan definitions.
- [ ] versioned prices.
- [ ] versioned entitlements.

## Billing Account

- [ ] organization-level.
- [ ] Stripe customer mapping.
- [ ] billing email.
- [ ] currency.
- [ ] tax fields.

## Subscription

- [ ] checkout.
- [ ] subscription mirror.
- [ ] billing periods.
- [ ] invoices mirror.
- [ ] payment records.
- [ ] portal.

## Entitlements

- [ ] central entitlement service.
- [ ] project limit.
- [ ] team limit.
- [ ] storage.
- [ ] AI Design.
- [ ] Pro QS.
- [ ] IFC.
- [ ] exports.

## Credits

- [ ] AI Design wallet.
- [ ] Render wallet.
- [ ] grants.
- [ ] ledger.
- [ ] reservation.
- [ ] settlement.
- [ ] release.
- [ ] expiry.
- [ ] no naked mutable balance.

## AI Usage

- [ ] customer credit charge.
- [ ] raw provider usage.
- [ ] rate-card version.
- [ ] actual provider cost estimate.
- [ ] retry does not double-charge.
- [ ] escalation does not surprise customer charge.

## Top-Ups

- [ ] one-time order.
- [ ] Stripe payment.
- [ ] authoritative webhook.
- [ ] exactly-one credit grant.

## Subscription Lifecycle

- [ ] trial.
- [ ] active.
- [ ] upgrade.
- [ ] downgrade next period.
- [ ] cancel at period end.
- [ ] reactivate.
- [ ] failed payment.
- [ ] grace.
- [ ] suspension.

## Webhooks

- [ ] signature verification.
- [ ] inbox.
- [ ] provider event uniqueness.
- [ ] retry.
- [ ] out-of-order handling.

## Reconciliation

- [ ] subscriptions.
- [ ] invoices.
- [ ] payments.
- [ ] grants.
- [ ] top-ups.
- [ ] provider usage/cost.

## Billing UI

- [ ] current plan.
- [ ] renewal.
- [ ] credits.
- [ ] usage history.
- [ ] invoices.
- [ ] upgrade.
- [ ] downgrade.
- [ ] buy credits.
- [ ] cancel/reactivate.

## Admin Billing

- [ ] customer account.
- [ ] plan/version.
- [ ] subscription.
- [ ] ledger.
- [ ] grants.
- [ ] reservations.
- [ ] AI cost.
- [ ] margin.
- [ ] webhook events.
- [ ] reconciliation.

## Tests

- [ ] duplicate webhook.
- [ ] simultaneous credit reservation.
- [ ] provider retry.
- [ ] failed renewal.
- [ ] upgrade.
- [ ] downgrade.
- [ ] top-up.
- [ ] refund policy.
- [ ] tenant isolation.

## Phase 17 Exit Gate

A paid customer can subscribe, receive entitlements/credits, consume AI safely, top up, renew, cancel, and see transparent usage without duplicate billing side effects.

---

# 21. Phase 18 — Reports & Exports (Extended)

> **Core reports shipped in Phase 11b** — BOQ XLSX, cost PDF, quantity takeoff.
> This phase adds the remainder, after AI and BIM exist.

## Objective

Extend the report set and add sharing.

## Additional reports (this phase)

- [ ] project summary.
- [ ] materials schedule.
- [ ] value engineering *(requires Phase 13 AI)*.
- [ ] revision comparison.

## Already delivered in Phase 11b

- [x] BOQ (XLSX).
- [x] cost estimate (PDF).
- [x] quantity takeoff with element traceability.

## Formats

- [ ] PDF.
- [ ] XLSX.
- [ ] CSV.
- [ ] IFC export where supported.
- [ ] GLB export where appropriate.

## Versioning

Every report records:

- [ ] model version.
- [ ] quantity run.
- [ ] BOQ version.
- [ ] cost estimate.
- [ ] report template version.
- [ ] checksum.

## Processing

- [ ] report job.
- [ ] asynchronous generation where needed.
- [ ] object storage.
- [ ] signed download.
- [ ] failure state.

## Sharing

- [ ] invited viewer.
- [ ] secure share link if enabled.
- [ ] expiry/revocation.
- [ ] no cross-tenant access.

## Phase 18 Exit Gate

A professional user can produce a reproducible BOQ/cost/report package from an explicit project version.

---

# 22. Phase 19 — Beta Readiness

## Objective

Make the product safe and understandable for external testers.

## Onboarding

- [ ] welcome screen.
- [ ] first-project flow.
- [ ] guided create flow.
- [ ] sample/demo project.
- [ ] tooltips.
- [ ] empty states.
- [ ] help links.
- [ ] billing explanation.
- [ ] credit explanation.

## UX

- [ ] consistent design system.
- [ ] responsive desktop/laptop.
- [ ] tablet review.
- [ ] keyboard navigation.
- [ ] accessibility pass.
- [ ] loading/error/empty states everywhere important.

## Security

- [ ] tenant isolation full suite.
- [ ] auth/session review.
- [ ] upload security.
- [ ] billing webhook security.
- [ ] signed storage URLs.
- [ ] rate limiting.
- [ ] secret scan.
- [ ] admin permissions.

## Reliability

- [ ] save/reload regression.
- [ ] Building Model fixture regression.
- [ ] 2D/3D.
- [ ] QS/BOQ/cost.
- [ ] AI failure.
- [ ] plan processing failure.
- [ ] billing failure.

## Observability

- [ ] Sentry web.
- [ ] Sentry API.
- [ ] Sentry workers.
- [ ] OpenTelemetry baseline.
- [ ] structured logs.
- [ ] request IDs.
- [ ] workflow/job IDs.

## Analytics

Track:

- [ ] signup.
- [ ] project created.
- [ ] wall created.
- [ ] 3D opened.
- [ ] quantity calculated.
- [ ] BOQ generated.
- [ ] AI used.
- [ ] plan uploaded.
- [ ] subscription started.

## Support

- [ ] support email.
- [ ] trace/reference IDs.
- [ ] bug reporting.
- [ ] admin project diagnostics.
- [ ] AI job diagnostics.
- [ ] billing diagnostics.

## Legal/Product

- [ ] Terms.
- [ ] Privacy.
- [ ] subscription/cancellation policy.
- [ ] AI disclaimer.
- [ ] construction/engineering disclaimer.
- [ ] cookie policy if needed.

## Phase 19 Exit Gate

A new tester can complete:

```text
signup
→ project
→ 2D
→ 3D
→ quantities
→ BOQ
→ cost
→ AI
→ report
```

without assistance.

---

# 23. Phase 20 — Production Launch

## Objective

Launch Buildora AI as a paid SaaS.

## Production Infrastructure

- [ ] production DB.
- [ ] production Redis.
- [ ] production object storage.
- [ ] production Temporal.
- [ ] production auth.
- [ ] production Stripe.
- [ ] production AI provider.
- [ ] production DNS.
- [ ] TLS.

## Database

- [ ] backups enabled.
- [ ] PITR enabled if available.
- [ ] DB roles least-privilege.
- [ ] connection pooling.
- [ ] migrations tested in staging.
- [ ] restore drill passed.

## Secrets

- [ ] no secrets in Git.
- [ ] production secret store.
- [ ] keys separated from staging.
- [ ] AI spending limits/alerts.

## Monitoring

- [ ] API error alert.
- [ ] DB unavailable alert.
- [ ] workflow failure alert.
- [ ] AI cost anomaly.
- [ ] billing webhook alert.
- [ ] payment failure alert.
- [ ] storage failure alert.

## Billing

- [ ] live Stripe keys.
- [ ] live products/prices.
- [ ] webhooks live.
- [ ] invoices verified.
- [ ] top-up verified.
- [ ] subscription lifecycle verified.
- [ ] cancellation verified.

## Performance

- [ ] representative house.
- [ ] large-house fixture.
- [ ] load time measured.
- [ ] editor FPS acceptable.
- [ ] 3D FPS acceptable.
- [ ] QS timing.
- [ ] AI timing.
- [ ] upload timing.

## Launch Gates

Must pass:

- [ ] cross-tenant suite.
- [ ] model save/reload.
- [ ] model conflict.
- [ ] QS golden tests.
- [ ] cost exactness.
- [ ] billing duplicate tests.
- [ ] backup/restore.
- [ ] report reproducibility.
- [ ] AI cannot direct-mutate model.

## Phase 20 Exit Gate

Production launch approved.

---

# 24. Phase 21 — Post-Launch Operations

## Objective

Operate the SaaS safely and learn from paying users.

## Daily

- [ ] monitor error rate.
- [ ] monitor failed workflows.
- [ ] monitor failed billing webhooks.
- [ ] monitor AI spend.
- [ ] monitor support tickets.

## Weekly

- [ ] top errors reviewed.
- [ ] AI cost per user reviewed.
- [ ] activation funnel reviewed.
- [ ] churn/cancellation reviewed.
- [ ] expensive queries reviewed.
- [ ] product feedback triaged.

## Monthly

- [ ] gross margin by plan.
- [ ] AI credits used.
- [ ] render usage.
- [ ] storage growth.
- [ ] MRR.
- [ ] churn.
- [ ] top feature usage.
- [ ] failed-payment recovery.
- [ ] backup health.
- [ ] security review.

## Engineering

- [ ] bug severity triage.
- [ ] architecture drift review.
- [ ] dependency updates.
- [ ] database indexes reviewed from evidence.
- [ ] agent standards refreshed if needed.

## Phase 21 Exit Gate

This phase does not “finish”.

It becomes continuous operations.

---

# 25. Phase 22 — Professional Expansion

Only start based on customer evidence.

> **No fixed exit gate — deliberately.** Phases 22 and 23 are demand-driven
> backlogs, not sequential deliverables. Each item ships when a paying customer
> requires it. "Completing Phase 22" is not a goal; shipping the items customers
> pay for is.
>
> Each item still carries the **standard Definition of Done** (tests, error
> states, permissions, tenant scoping, observability, docs) and must satisfy the
> seven database completeness rules where it adds tables.

---

## Site / GIS

- [ ] PostGIS enabled.
- [ ] site location.
- [ ] plot boundary.
- [ ] orientation.
- [ ] sun path.
- [ ] site metrics.
- [ ] terrain strategy.

---

## Scheduling

- [ ] schedule version.
- [ ] tasks.
- [ ] dependencies.
- [ ] productivity assumptions.
- [ ] Gantt.
- [ ] milestone.
- [ ] BOQ link.
- [ ] cash flow.

---

## Procurement

- [ ] supplier records.
- [ ] RFQ.
- [ ] quote upload.
- [ ] quote comparison.
- [ ] supplier selection.
- [ ] purchase order.
- [ ] delivery.

---

## Actual Cost

- [ ] estimated.
- [ ] committed.
- [ ] actual.
- [ ] forecast.
- [ ] variance.

---

## Change Orders

- [ ] request.
- [ ] model impact.
- [ ] cost delta.
- [ ] schedule delta.
- [ ] approval.
- [ ] revised budget.

---

## Mobile / Field

- [ ] React Native.
- [ ] project list.
- [ ] drawings.
- [ ] 3D view.
- [ ] AI.
- [ ] site photos.
- [ ] notes.
- [ ] issues.
- [ ] approvals.
- [ ] progress.

Do not build full mobile CAD initially.

---

# 26. Phase 23 — Enterprise & Scale

## Objective

Support larger customers only when usage justifies the complexity.

## Enterprise SaaS

- [ ] advanced RBAC.
- [ ] SSO.
- [ ] SCIM if required.
- [ ] audit export.
- [ ] retention policies.
- [ ] data residency strategy.
- [ ] custom contracts.
- [ ] enterprise billing overrides.
- [ ] purchase orders/invoice terms.

## Collaboration

- [ ] presence.
- [ ] cursors.
- [ ] comments.
- [ ] annotations.
- [ ] geometry edit conflict strategy.
- [ ] optimistic locking.
- [ ] Yjs/Liveblocks only where appropriate.

## Integrations

Prioritize only requested integrations:

- [ ] Autodesk.
- [ ] Revit.
- [ ] AutoCAD.
- [ ] Xero.
- [ ] QuickBooks.
- [ ] Google Drive.
- [ ] OneDrive.
- [ ] supplier APIs.

## Performance

Measure before implementing:

- [ ] WASM geometry hotspots.
- [ ] binary payloads.
- [ ] model streaming.
- [ ] fragments.
- [ ] large IFC.
- [ ] caching.
- [ ] read replicas.

## Infrastructure

Evaluate only from evidence:

- [ ] Go service.
- [ ] Kubernetes.
- [ ] multi-region.
- [ ] dedicated enterprise DB.
- [ ] sharding.

## Reliability

- [ ] formal SLOs.
- [ ] DR drills.
- [ ] capacity tests.
- [ ] dependency outage tests.
- [ ] incident response.

---

# 27. Cross-Phase Engineering Checklist

Every phase must include these.

## Architecture

- [ ] correct bounded context.
- [ ] no source-of-truth duplication.
- [ ] ADR created if architecture changes.
- [ ] dependencies flow correctly.

## Backend

- [ ] input validated.
- [ ] thin controller.
- [ ] use case.
- [ ] tenant scope.
- [ ] authorization.
- [ ] stable errors.
- [ ] transaction reviewed.

## Frontend

- [ ] Server/Client boundary correct.
- [ ] shared component reuse.
- [ ] no giant Zustand store.
- [ ] loading.
- [ ] error.
- [ ] empty state.
- [ ] accessibility.

## Database

- [ ] table ownership.
- [ ] organization ownership.
- [ ] foreign keys.
- [ ] constraints.
- [ ] indexes based on queries.
- [ ] migration.
- [ ] deletion policy.
- [ ] provenance/versioning.

## Security

- [ ] authorization server-side.
- [ ] cross-tenant test.
- [ ] sensitive fields not logged.
- [ ] secrets not exposed.
- [ ] upload safety if relevant.

## Testing

- [ ] unit.
- [ ] integration.
- [ ] E2E if user flow.
- [ ] regression fixture.
- [ ] failure cases.

## Observability

- [ ] structured log.
- [ ] trace ID.
- [ ] error capture.
- [ ] useful metrics.

## Documentation

- [ ] architecture docs updated.
- [ ] API contract updated.
- [ ] ADR updated.
- [ ] README/runbook if needed.

---

# 28. Phase Completion Template

Use after each phase.

```text
PHASE:
NAME:

OBJECTIVE:
...

COMPLETED:
[x] ...
[x] ...

NOT COMPLETED:
[ ] ...

BLOCKERS:
[!] ...

ARCHITECTURE DECISIONS:
- ...

ADRS CREATED:
- ...

DATABASE MIGRATIONS:
- ...

TESTS:
- Unit:
- Integration:
- E2E:
- Security:
- Performance:

KNOWN ISSUES:
- ...

SECURITY REVIEW:
...

PERFORMANCE:
...

OBSERVABILITY:
...

DOCUMENTATION UPDATED:
...

DEPLOYMENT:
local / staging / production

EXIT GATE:
PASS / FAIL

NEXT PHASE PREREQUISITES:
...
```

---

# 29. Feature Definition of Done

A feature is not complete until:

- [ ] requirements clear.
- [ ] architecture docs read.
- [ ] code implemented.
- [ ] type-safe.
- [ ] validation implemented.
- [ ] error handling implemented.
- [ ] permission implemented.
- [ ] tenant scoping implemented.
- [ ] DB migration if needed.
- [ ] unit tests.
- [ ] integration tests.
- [ ] UI loading state.
- [ ] UI error state.
- [ ] empty state where relevant.
- [ ] accessibility considered.
- [ ] logs/telemetry.
- [ ] documentation.
- [ ] `pnpm verify`.
- [ ] Claude review.
- [ ] Codex fixes.
- [ ] Antigravity regression if UI.
- [ ] you reviewed final diff.

---

# 30. Release Definition of Done

A release is not complete until:

- [ ] CI green.
- [ ] staging green.
- [ ] migrations safe.
- [ ] critical workflows tested.
- [ ] tenant isolation passed.
- [ ] billing passed.
- [ ] backups healthy.
- [ ] monitoring active.
- [ ] rollback plan exists.
- [ ] changelog/release notes.
- [ ] known issues documented.

---

# 31. Technical Debt Checklist

At the end of every 3–4 phases:

- [ ] dead code removed.
- [ ] abandoned feature flags removed.
- [ ] TODOs reviewed.
- [ ] dependency versions reviewed.
- [ ] duplicated code reviewed.
- [ ] architecture violations reviewed.
- [ ] slow queries reviewed.
- [ ] bundle size reviewed.
- [ ] test duration reviewed.
- [ ] flaky tests fixed.
- [ ] docs aligned with code.

Do not let AI-generated code accumulate without cleanup.

---

# 32. AI Agent Use Checklist

For every major task:

## Claude

- [ ] architecture/plan request.
- [ ] no code changes unless asked.
- [ ] P0/P1/P2 review after implementation.

## Codex

- [ ] exact scope.
- [ ] relevant docs linked.
- [ ] acceptance criteria.
- [ ] tests required.
- [ ] `pnpm verify`.

## Antigravity

Use when UI changed:

- [ ] browser workflow.
- [ ] console.
- [ ] network.
- [ ] visual regressions.
- [ ] accessibility basics.
- [ ] responsive checks.

## You

- [ ] approve architecture.
- [ ] inspect diff.
- [ ] decide ADR.
- [ ] merge.

---

# 33. Scope Control Checklist

Before accepting a new feature during MVP:

Ask:

1. [ ] Does this directly help the MVP core flow?
2. [ ] Is it required by a paying/beta user?
3. [ ] Does another feature depend on it?
4. [ ] Can it wait until after launch?
5. [ ] Does it introduce another technology?
6. [ ] Does it increase regulatory/safety risk?
7. [ ] Does it increase ongoing support cost?
8. [ ] Can a smaller version prove the value?

If mostly "no":

```text
move to post-launch backlog
```

---

# 34. MVP Scope Freeze Checklist

Before commercial beta, Buildora AI should focus on:

- [ ] authentication.
- [ ] organizations.
- [ ] projects.
- [ ] Building Model.
- [ ] 2D.
- [ ] 3D.
- [ ] materials.
- [ ] assemblies.
- [ ] QS.
- [ ] BOQ.
- [ ] cost.
- [ ] AI Copilot.
- [ ] controlled AI actions.
- [ ] upload/recognition.
- [ ] basic IFC.
- [ ] documents/RAG.
- [ ] reports.
- [ ] billing.
- [ ] monitoring.
- [ ] support.

Explicitly defer unless proven necessary:

- [ ] full structural design.
- [ ] full MEP.
- [ ] procurement.
- [ ] marketplace.
- [ ] mobile CAD.
- [ ] global pricing database.
- [ ] advanced GIS.
- [ ] complete Revit round-trip.
- [ ] Kubernetes.
- [ ] microservices.
- [ ] multi-region.

---

# 35. Data Integrity Checklist

At every phase touching data:

- [ ] correct source of truth.
- [ ] version reference.
- [ ] provenance.
- [ ] foreign keys.
- [ ] constraints.
- [ ] transaction boundaries.
- [ ] tenant ownership.
- [ ] deletion behaviour.
- [ ] audit requirement.
- [ ] restore behaviour.

---

# 36. Building Model Integrity Checklist

- [ ] stable IDs.
- [ ] mm canonical units.
- [ ] no renderer state stored as domain.
- [ ] no IFC dependency in core.
- [ ] geometry schema version.
- [ ] model version.
- [ ] command history.
- [ ] stale version handling.
- [ ] derived data source versions.
- [ ] AI ChangeSet approval.

---

# 37. QS / Cost Integrity Checklist

- [ ] deterministic.
- [ ] no LLM arithmetic.
- [ ] exact units.
- [ ] exact money.
- [ ] assembly version.
- [ ] ruleset version.
- [ ] rate version.
- [ ] source elements.
- [ ] manual override provenance.
- [ ] historical output reproducible.

---

# 38. AI Integrity Checklist

- [ ] provider abstraction.
- [ ] typed tools.
- [ ] permissions.
- [ ] token/cost usage.
- [ ] no arbitrary SQL.
- [ ] no live model mutation.
- [ ] ChangeSet.
- [ ] validation.
- [ ] user approval.
- [ ] audit.
- [ ] fallback/error state.

---

# 39. Billing Integrity Checklist

- [ ] Stripe only payment authority.
- [ ] Buildora AI entitlement authority.
- [ ] plan version.
- [ ] entitlement version.
- [ ] ledger.
- [ ] reservation.
- [ ] idempotency.
- [ ] webhook uniqueness.
- [ ] exact money.
- [ ] provider usage separate from customer credits.
- [ ] reconciliation.

---

# 40. Security Checklist

Before beta and every major release:

- [ ] dependency vulnerability scan.
- [ ] secret scan.
- [ ] auth bypass test.
- [ ] tenant isolation.
- [ ] IDOR tests.
- [ ] rate limits.
- [ ] upload validation.
- [ ] object storage private.
- [ ] signed URLs.
- [ ] webhook signatures.
- [ ] admin authorization.
- [ ] AI prompt/tool boundaries.
- [ ] PII logging review.

---

# 41. Performance Checklist

Do not optimize without measurements.

Measure:

- [ ] dashboard load.
- [ ] project load.
- [ ] Building Model hydration.
- [ ] model command commit.
- [ ] 2D FPS.
- [ ] 3D FPS.
- [ ] memory.
- [ ] QS calculation.
- [ ] BOQ.
- [ ] cost.
- [ ] AI latency.
- [ ] upload processing.
- [ ] IFC import.
- [ ] report generation.

---

# 42. Database Checklist

Before adding any table:

- [ ] owning module.
- [ ] authoritative/derived.
- [ ] organization ownership.
- [ ] project ownership.
- [ ] ID type.
- [ ] foreign keys.
- [ ] unique constraints.
- [ ] indexes.
- [ ] money precision.
- [ ] JSONB justification.
- [ ] retention.
- [ ] deletion.
- [ ] migration.
- [ ] tests.

---

# 43. Deployment Checklist

## Staging

- [ ] migration applied.
- [ ] API deployed.
- [ ] web deployed.
- [ ] workers deployed.
- [ ] smoke tests.
- [ ] Sentry clean.
- [ ] DB healthy.
- [ ] billing test mode.
- [ ] AI budget controls.

## Production

- [ ] production migration reviewed.
- [ ] backup before risky migration.
- [ ] deploy.
- [ ] health.
- [ ] smoke.
- [ ] critical transaction test.
- [ ] monitor errors.
- [ ] rollback ready.

---

# 44. Launch Readiness Master Checklist

The following must all be true before paid launch:

## Product

- [ ] user can create project.
- [ ] user can draw plan.
- [ ] user can see 3D.
- [ ] user can assign materials.
- [ ] quantities work.
- [ ] BOQ works.
- [ ] cost works.
- [ ] AI works.
- [ ] upload works.
- [ ] reports work.

## SaaS

- [ ] tenant isolation.
- [ ] billing.
- [ ] usage credits.
- [ ] invoices.
- [ ] cancellation.
- [ ] onboarding.
- [ ] support.

## Technical

- [ ] CI.
- [ ] staging.
- [ ] backups.
- [ ] monitoring.
- [ ] security.
- [ ] observability.
- [ ] migration process.
- [ ] restore test.

## Commercial

- [ ] pricing.
- [ ] plan definitions.
- [ ] credit rules.
- [ ] refund policy.
- [ ] subscription terms.
- [ ] privacy.
- [ ] disclaimers.

---

# 45. First 10 Practical Development Milestones

If you want the simplest execution view:

## Milestone 1

```text
Repo + CI + local infrastructure
```

- [ ] complete.

## Milestone 2

```text
Auth + Organizations + Projects
```

- [ ] complete.

## Milestone 3

```text
Building Model
```

- [ ] complete.

## Milestone 4

```text
2D Editor
```

- [ ] complete.

## Milestone 5

```text
3D Engine + Sync
```

- [ ] complete.

## Milestone 6

```text
Materials + QS
```

- [ ] complete.

## Milestone 7

```text
BOQ + Cost
```

- [ ] complete.

## Milestone 8

```text
AI Copilot + ChangeSets
```

- [ ] complete.

## Milestone 9

```text
Upload + BIM + RAG
```

- [ ] complete.

## Milestone 10

```text
Billing + Reports + Beta + Launch
```

- [ ] complete.

---

# 46. Do Not Move Forward If

Stop progression if any of these are true:

```text
Building Model architecture unclear
CI is red
tests are skipped
tenant isolation fails
money uses floating point
AI directly mutates geometry
2D and 3D have separate authoritative state
QS depends on AI arithmetic
billing has only a mutable credit balance
provider webhooks are not idempotent
database migrations are performed manually in production
uploads are public
backups have never been restored
```

Fix the foundation first.

---

# 47. Recommended Repository Location

Store this file as:

```text
docs/planning/
Buildora_AI_Phase_Wise_Development_Checklist.md
```

Recommended planning folder:

```text
docs/
├── architecture/
├── standards/
├── examples/
└── planning/
    ├── Buildora_AI_Sprintwise_Project_Plan.md
    └── Buildora_AI_Phase_Wise_Development_Checklist.md
```

---

# 48. AGENTS.md Instruction

Add:

```text
Before starting a new major phase, read:

docs/planning/Buildora_AI_Phase_Wise_Development_Checklist.md

Do not implement work from a later phase if the required exit gate
from the preceding phase has not been met, unless explicitly approved.

For every task:
- identify the current phase,
- identify which checklist items it satisfies,
- follow the relevant architecture and coding standards,
- do not silently expand scope.
```

---

# 49. Final Development Sequence

The master order is:

```text
ARCHITECTURE
    ↓
ENVIRONMENT
    ↓
REPOSITORY
    ↓
SAAS IDENTITY
    ↓
PROJECTS
    ↓
BUILDING MODEL
    ↓
2D
    ↓
3D
    ↓
SYNC
    ↓
MATERIALS
    ↓
QS
    ↓
BOQ
    ↓
COST
    ↓
AI PLATFORM
    ↓
AI ACTIONS
    ↓
PLAN RECOGNITION
    ↓
BIM
    ↓
DOCUMENTS / RAG
    ↓
BILLING
    ↓
REPORTS
    ↓
BETA
    ↓
PRODUCTION
    ↓
CUSTOMER-DRIVEN EXPANSION
```

---

# 50. Final Principle

Buildora AI should never be developed as:

```text
many impressive independent features
```

It should be developed as:

```text
one reliable architecture
+
one verified layer at a time
+
one source of truth
+
one measurable exit gate per phase
```

The single rule to keep throughout the entire project is:

> **Do not move to the next phase until the current phase is reliable enough that the next phase can safely depend on it.**

That discipline matters more than development speed.
