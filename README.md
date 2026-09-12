# BuildWise Architecture & Implementation Pack

**Status:** Architecture baseline  
**Audience:** Engineering, product, QA, DevOps, security, Codex/AI implementation agents  
**Purpose:** This folder is the implementation source of truth for BuildWise.

## How to use this pack

1. Read `01_Master_Product_Reference.md` for the full product direction.
2. Read `02_Product_Scope_and_Bounded_Contexts.md` and `03_System_Architecture.md` before creating new services or packages.
3. Read the domain-specific guide for the feature being implemented.
4. Read `29_Coding_Standards.md`, `26_Testing_Strategy.md`, and `31_Definition_of_Done_and_Release_Gates.md` before opening a PR.
5. Check `34_ADR_Index.md` before changing a deliberate architecture decision.
6. Treat newer explicitly approved ADRs as higher priority than older prose when there is a conflict.

## Non-negotiable principles

- The **BuildWise Building Model** is the single source of truth for design state.
- 2D, 3D, BIM/IFC, quantities, BOQ, cost, and schedule are derived from the same model.
- AI proposes structured operations; deterministic engines validate and execute them.
- LLM output is never the authoritative source for quantities, geometry, prices, or financial arithmetic.
- High-impact AI changes use preview, validation, audit, and user approval.
- Interactive 3D runs primarily on the client GPU; expensive photoreal rendering is queued and credit-controlled.
- Use a modular architecture first. Split services only for independent scaling, security boundaries, runtime needs, or fault isolation.
- Model/provider names, prices, subscription limits, credit weights, confidence thresholds, and regional rules are configuration.

## Documentation map

| File | Purpose |
|---|---|
| `01_Master_Product_Reference.md` | Product vision, features, flows, stack and commercial baseline |
| `02_Product_Scope_and_Bounded_Contexts.md` | Domain boundaries and ownership |
| `03_System_Architecture.md` | Runtime architecture and deployment topology |
| `04_Monorepo_and_Module_Architecture.md` | Repository/package rules |
| `05_Building_Model_Domain.md` | Core design domain model |
| `06_2D_CAD_Editor_Implementation.md` | 2D editor architecture |
| `07_3D_Engine_Implementation.md` | Three.js engine and scene lifecycle |
| `08_BIM_IFC_Interoperability.md` | IFC import/export and BIM strategy |
| `09_Plan_Recognition_and_Import_Pipeline.md` | PDF/image/CAD recognition flow |
| `10_AI_Gateway_and_Model_Routing.md` | AI provider abstraction and automatic routing |
| `11_AI_Copilot_Tool_Architecture.md` | Tool calling and safe AI actions |
| `12_QS_Engine.md` | Deterministic quantity surveying engine |
| `13_BOQ_and_Cost_Engine.md` | BOQ, rate build-up and cost engine |
| `14_Materials_Assemblies_and_Rates.md` | Materials, recipes, waste and rates |
| `15_Database_and_Data_Architecture.md` | PostgreSQL/PostGIS/pgvector design |
| `16_API_and_Contracts.md` | REST, WebSocket and internal contracts |
| `17_Command_Versioning_and_Undo_Redo.md` | Commands, versions, drafts and history |
| `18_Workflows_and_Background_Jobs.md` | Temporal workflows and worker design |
| `19_Realtime_Collaboration.md` | Presence, comments and geometry conflicts |
| `20_Documents_RAG_and_Search.md` | Project documents, embeddings and citations |
| `21_Reports_and_Exports.md` | PDF/XLSX/IFC/GLB/report generation |
| `22_Auth_RBAC_and_Tenant_Architecture.md` | Organisations, roles and permissions |
| `23_Security_Threat_Model.md` | Threats, controls and secure AI handling |
| `24_Billing_AI_Credits_and_Usage.md` | Plans, metering and margin controls |
| `25_Observability_and_SLOs.md` | Logs, traces, metrics and SLOs |
| `26_Testing_Strategy.md` | Unit, geometry, AI, integration and E2E tests |
| `27_DevOps_Environments_and_Deployment.md` | Environments, CI/CD and cloud deployment |
| `28_Performance_and_Scalability.md` | Browser/server performance budgets and scale |
| `29_Coding_Standards.md` | TypeScript, Python, SQL and native code rules |
| `30_Implementation_Roadmap.md` | Phased delivery plan and dependencies |
| `31_Definition_of_Done_and_Release_Gates.md` | Quality/release checklist |
| `32_Admin_and_Operations.md` | Internal admin and operational tooling |
| `33_Disaster_Recovery_and_Backups.md` | Backup, RPO/RTO and restoration |
| `34_ADR_Index.md` | Architecture Decision Records index |

## Recommended Codex bootstrap prompt

```text
Read 00_README.md, 01_Master_Product_Reference.md, the relevant implementation guide,
and all ADRs referenced by that guide before writing code.

Preserve the BuildWise Building Model as the single source of truth. Do not duplicate 2D,
3D, BIM, QS, BOQ, cost, or schedule state. Keep AI actions structured, validated,
auditable, and reversible. Keep authoritative geometry, quantities, and financial
calculations deterministic. Use existing packages and contracts before creating new ones.
If a requested implementation conflicts with an ADR, identify the conflict before changing it.
```
