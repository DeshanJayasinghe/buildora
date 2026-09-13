# Buildora AI — Documentation Router

> **Status:** Current — architecture frozen 2026-09-12
> **Purpose:** The authoritative index of Buildora AI documentation.
> **Rule:** Every path listed under "Existing documents" resolves to a real file.
> Planned documents are listed separately and are **never** mandatory reads.

---

# 1. Start Here

| If you are… | Read, in order |
|---|---|
| Any agent, before any change | `AGENTS.md` → this file → the domain doc for your task |
| Claude (architecture/review) | `AGENTS.md` → `CLAUDE.md` → this file |
| Implementing a feature | `AGENTS.md` → this file → `34_ADR_Index.md` → the relevant architecture doc → the relevant standards doc |
| Setting up a machine | `docs/setup/00_README_START_HERE.md` |
| Checking what is locked | `docs/architecture/FINAL_SYSTEM_ARCHITECTURE.md` |

**The architecture is frozen.** Decisions recorded in the ADRs may not be changed
by an implementation agent without a new approved ADR that explicitly supersedes
the old one. See `docs/architecture/34_ADR_Index.md` §1.

---

# 2. Existing Documents

Everything in this section exists. Paths are relative to the repository root.

## 2.1 Architecture — frozen baseline and decisions

| Document | Purpose |
|---|---|
| `docs/architecture/FINAL_SYSTEM_ARCHITECTURE.md` | **The concise frozen baseline. Read this first for architecture.** |
| `docs/architecture/34_ADR_Index.md` | Index of all 22 accepted ADRs and precedence rules |
| `docs/architecture/adr/` | The ADRs themselves (`0001`–`0022`) plus the template |
| `docs/architecture/FINAL_ARCHITECTURE_REVIEW.md` | The review that produced the freeze; findings and rationale |
| `docs/architecture/OPEN_ARCHITECTURE_DECISIONS.md` | What remains genuinely unresolved, plus the **contract gates** (§1.1) |
| `docs/architecture/CODEX_INDEPENDENT_REVIEW.md` | Independent adversarial review of the frozen baseline (2026-09-12) |

## 2.2 Architecture — canonical subject documents

| Document | Purpose |
|---|---|
| `docs/architecture/01_Master_Product_Reference.md` | Product vision, scope, UX flows, feature set, commercial baseline |
| `docs/architecture/03_System_Architecture.md` | Runtime architecture and deployment topology |
| `docs/architecture/04_Monorepo_and_Module_Architecture.md` | Repository shape, package boundaries, dependency rules |
| `docs/architecture/05_Building_Model_Domain.md` | **The canonical Building Model.** Hierarchy, elements, datum, rooms, openings, types, commands |
| `docs/architecture/06_2D_CAD_Editor_Implementation.md` | 2D editor architecture, PixiJS boundaries, interaction model |
| `docs/architecture/07_3D_Engine_Implementation.md` | Three.js engine, adapters, scene lifecycle, disposal |
| `docs/architecture/11_3D_Walkthrough_Interior_and_FFE.md` | Walkthrough, material/finish editing, FF&E, optional AI interior layer |
| `docs/architecture/12_User_Segments_and_Commercial_Model.md` | Customer segments, two commercial lifecycles, retention strategy |
| `docs/architecture/15_Database_and_Data_Architecture.md` | PostgreSQL schema strategy, versioning tables, indexes, migrations, lineage |
| `docs/architecture/24_Billing_AI_Credits_and_Usage.md` | Plans, entitlements, wallets, ledger, reservations, Stripe, provider usage |
| `docs/architecture/27_DevOps_Environments_and_Deployment.md` | Environments, CI/CD, cloud topology, observability, DR, runbooks |

## 2.3 Engineering standards

| Document | Purpose |
|---|---|
| `docs/standards/Buildora_AI_Backend_Engineering_Standards.md` | NestJS structure, use cases, repositories, errors, money, transactions |
| `docs/standards/Buildora_AI_Frontend_Engineering_Standards.md` | Next.js boundaries, state ownership, API client, editor/renderer rules |
| `docs/standards/Buildora_AI_Codex_Implementation_Reference.md` | Condensed implementation reference for coding agents |
| `docs/standards/Buildora_AI_Solo_Developer_AI_Assisted_Development_Guide.md` | Working practice for solo + AI-assisted development |

## 2.4 Planning

| Document | Purpose |
|---|---|
| `docs/planning/Buildora_AI_Phase_Wise_Development_Checklist.md` | Phases, exit gates, detailed checklists |
| `docs/planning/Buildora_AI_Sprintwise_Project_Plan.md` | Sprint-by-sprint delivery plan and acceptance criteria |

## 2.5 Finance

| Document | Purpose |
|---|---|
| `docs/finance/Buildora_AI_Development_Production_Costing_Plan.md` | Development and production cost model; provider pricing reference |
| `docs/finance/Buildora_AI_Return_On_Investment_Plan.md` | Revenue model, unit economics, ROI projections |

## 2.6 Examples

| Document | Purpose |
|---|---|
| `docs/examples/Buildora_AI_Sample_Backend_CRUD.md` | Reference backend CRUD implementation pattern |
| `docs/examples/Buildora_AI_Sample_Frontend_CRUD.md` | Reference frontend CRUD implementation pattern |

> **Examples are patterns, not blind copy targets** (`AGENTS.md` §70).
> Do **not** apply simple CRUD architecture to Building Model mutations, credit
> ledgers, versioned financial history, or workflow orchestration.

## 2.7 Setup

`docs/setup/` contains the 46-document environment and tooling setup pack.
Start at `docs/setup/00_README_START_HERE.md`; the ordered checklist is
`docs/setup/01_SETUP_ORDER_AND_CHECKLIST.md`.

> Setup documents describe **how to install and configure tools**. They are not
> architecture. Where a setup document and an ADR disagree, the ADR wins.

## 2.8 Operations

`docs/operations/` is currently empty. Runbooks are written as production
readiness approaches; until then, operational guidance lives in
`docs/architecture/27_DevOps_Environments_and_Deployment.md` §136–§140 and §185.

---

# 3. Planned Documents — Not Yet Written

**These files do not exist.** They are listed so their absence is deliberate and
visible, not so agents attempt to read them.

**Do not treat any path in this section as a mandatory read. Do not create a
document here speculatively — write it when its phase begins.**

| Planned document | Subject | Expected phase |
|---|---|---|
| `02_Product_Scope_and_Bounded_Contexts.md` | Domain boundaries and ownership | Nearest source: `01_Master_Product_Reference.md` §6 |
| `08_BIM_IFC_Interoperability.md` | IFC import/export detail | Phase 15 — see ADR-018 |
| `09_Plan_Recognition_and_Import_Pipeline.md` | PDF/image/CAD recognition | Phase 14 — see `01_Master_Product_Reference.md` §10 |
| `10_AI_Gateway_and_Model_Routing.md` | Provider abstraction, routing | Phase 12 — see ADR-010 |
| `11_AI_Copilot_Tool_Architecture.md` | Tool calling, safe AI actions | Phase 13 — see ADR-013 |
| `12_QS_Engine.md` | Quantity engine detail | Phase 10 — see ADR-009, ADR-015 |
| `13_BOQ_and_Cost_Engine.md` | BOQ and rate build-up | Phase 11 — see ADR-015 |
| `14_Materials_Assemblies_and_Rates.md` | Materials, recipes, waste | Phase 9 — see `15_Database_and_Data_Architecture.md` §51–§57 |
| `16_API_and_Contracts.md` | REST/WebSocket contracts | Nearest source: Backend Standards §11–§17 |
| `17_Command_Versioning_and_Undo_Redo.md` | Command/version detail | See ADR-002 and `15_Database…` §37–§45 |
| `18_Workflows_and_Background_Jobs.md` | Temporal workflows | Phase 14 — see ADR-012 |
| `19_Realtime_Collaboration.md` | Presence, comments, conflicts | Deferred — see ADR-012 |
| `20_Documents_RAG_and_Search.md` | Embeddings and citations | Phase 16 |
| `21_Reports_and_Exports.md` | PDF/XLSX/IFC/GLB generation | Phase 18 |
| `22_Auth_RBAC_and_Tenant_Architecture.md` | Roles and permission matrix | Phase 3 — see ADR-016 |
| `23_Security_Threat_Model.md` | Threats and controls | Phase 19 — see ADR-013, ADR-016 |
| `25_Observability_and_SLOs.md` | Logs, traces, metrics, SLOs | Phase 19 — see `27_DevOps…` §94–§110 |
| `26_Testing_Strategy.md` | Test architecture | Nearest source: `AGENTS.md` §61, `CLAUDE.md` §41 |
| `28_Performance_and_Scalability.md` | Budgets and scale | Phase 23 |
| `29_Coding_Standards.md` | — | **Superseded** by the two standards documents in §2.3. Will not be written. |
| `30_Implementation_Roadmap.md` | — | **Superseded** by the planning documents in §2.4. Will not be written. |
| `31_Definition_of_Done_and_Release_Gates.md` | DoD and gates | Nearest source: `AGENTS.md` §85, phase checklist exit gates |
| `32_Admin_and_Operations.md` | Internal admin tooling | Phase 21 |
| `33_Disaster_Recovery_and_Backups.md` | Backup, RPO/RTO, restore | Nearest source: `27_DevOps…` §122–§133 |

---

# 4. Routing by Task

| Task | Read |
|---|---|
| Changing the Building Model | `05_Building_Model_Domain.md`, ADR-001, ADR-002, ADR-005, ADR-006, ADR-007 |
| Adding an API endpoint | Backend Standards, `04_Monorepo_and_Module_Architecture.md`, `Buildora_AI_Sample_Backend_CRUD.md` |
| Database schema change | `15_Database_and_Data_Architecture.md`, ADR-004, ADR-016 |
| Frontend feature | Frontend Standards, `03_System_Architecture.md` |
| Editor (2D) work | `06_2D_CAD_Editor_Implementation.md`, ADR-008, ADR-006 |
| Editor (3D) work | `07_3D_Engine_Implementation.md`, ADR-008, ADR-017 |
| Walkthrough, materials, furniture | `11_3D_Walkthrough_Interior_and_FFE.md`, ADR-019, ADR-020 |
| Quantities or cost | ADR-009, ADR-015, `15_Database…` §58–§72 |
| Anything AI | ADR-010, ADR-013 |
| Anything billing | `24_Billing_AI_Credits_and_Usage.md`, ADR-014, ADR-021 |
| Plans, segments, archive mode | `12_User_Segments_and_Commercial_Model.md`, ADR-021 |
| Feature gating, admin entitlements | `24_Billing_AI_Credits_and_Usage.md` §13, ADR-022 |
| Infrastructure or deployment | `27_DevOps_Environments_and_Deployment.md`, ADR-012 |
| Adding a package or app | `04_Monorepo_and_Module_Architecture.md`, ADR-011, ADR-012 |
| Tenancy or permissions | ADR-016, `15_Database…` §10–§15 |
| Sprint scope | `Buildora_AI_Sprintwise_Project_Plan.md`, `Buildora_AI_Phase_Wise_Development_Checklist.md` |
| Cost of a decision | `Buildora_AI_Development_Production_Costing_Plan.md` |

---

# 5. Documentation Rules

1. **One subject, one owner.** Do not create a second document for a subject an
   existing canonical document already owns. Route to it instead.
2. **This router lists only real files.** If you add a document, add it to §2 and
   remove it from §3. If you reference a document, verify it exists.
3. **Do not create placeholder documents.** An entry in §3 is cheaper and more
   honest than an empty file that looks authoritative.
4. **Architecture changes require ADRs** (`AGENTS.md` §76). Update the affected
   document *and* the ADR index in the same change.
5. **Setup ≠ architecture.** Installation guidance lives in `docs/setup/`.
   Architectural decisions live in `docs/architecture/` and the ADRs.
6. **Where documents conflict**, follow the precedence order in `AGENTS.md` §4.
   Report the contradiction rather than silently choosing.

---

# 6. Naming

The product is **Buildora AI**. The repository and package scope is
**`buildora-ai`**.

The former name `BuildWise` must not appear in new documentation or code.
