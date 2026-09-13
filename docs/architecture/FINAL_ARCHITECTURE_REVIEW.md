# Buildora AI — Final Architecture Review

> **Reviewer role:** Principal Software Architect / final architecture reviewer
> **Date of review:** 2026-09-12
> **Repository state at review:** documentation-complete, implementation-empty
> **Verdict (summary):** **APPROVE WITH CHANGES**
>
> ---
>
> ## ⚠ Post-review corrections — applied 2026-09-12
>
> This review was independently assessed by the Product Owner / CTO and
> **accepted with corrections**. The corrections below are authoritative and
> override the original text wherever they differ. The body of this document has
> been amended to match.
>
> 1. **OpenAI pricing — original finding WITHDRAWN.** The review claimed the
>    model names and prices in the finance documents were fabricated. **That was
>    wrong.** All four models and every rate were re-verified against official
>    OpenAI API documentation on 2026-09-12 and found accurate, including the
>    `gpt-5.6-sol` promotional expiry of 21 November 2026. **Former P0-5 is
>    withdrawn.** The surviving principle — provider models and prices are
>    versioned configuration behind logical aliases — is recorded in **ADR-010**.
> 2. **Model-session package.** The editor working model is owned by a
>    renderer-neutral **`packages/model-session`**, not by `packages/cad-2d`.
>    Placing it in the 2D package would make 3D depend on 2D. See **ADR-008**.
> 3. **Room boundary.** The persisted room boundary is derived from **interior
>    wall faces**, not wall centrelines. Centrelines establish connectivity only.
>    See **ADR-006**.
> 4. **Level geometry.** `Level.elevationMm` is the single authoritative vertical
>    fact. Floor-to-floor is **derived** from adjacent elevations, not stored.
>    Slab thickness belongs to the Slab and its assembly. See **ADR-005**.
> 5. **`packages/core` not created.** `packages/domain` is deleted, but it is
>    **not** replaced by `packages/core` now — that would relocate the same
>    problem. Concepts stay with their owning module. See **ADR-011**.
> 6. **RLS timing.** Design, policies, session-context helper and pooled-connection
>    verification from Sprint 05; **enforcement is a hard gate before the first
>    external beta**. See **ADR-016**.
>
> **The architecture-lock set is 8 items (P0-1 … P0-8), not 6.** The former
> separate "MUST DECIDE NOW" list in `OPEN_ARCHITECTURE_DECISIONS.md` used a
> different count; both documents now use one reconciled set. See §9.
>
> **Current status:** all 8 architecture-lock items are **resolved**; 18 ADRs are
> accepted; the baseline is frozen in
> `docs/architecture/FINAL_SYSTEM_ARCHITECTURE.md`.

---

# 0. What Was Reviewed

## 0.1 Documents actually present and read

| Document | Size | Status |
|---|---:|---|
| `AGENTS.md` | 34 KB | Read in full |
| `CLAUDE.md` | 28 KB | Read in full |
| `README.md` (root) | 5 KB | Read in full |
| `docs/architecture/01_Master_Product_Reference.md` | 49 KB / 3,360 lines | Structure + all architecture sections |
| `docs/architecture/15_Database_and_Data_Architecture.md` | 91 KB / 6,040 lines | Structure + all core sections |
| `docs/architecture/24_Billing_AI_Credits_and_Usage.md` | 91 KB / 5,920 lines | Structure + core decisions |
| `docs/architecture/27_DevOps_Environments_and_Deployment.md` | 89 KB / 5,440 lines | Structure + topology/stages |
| `docs/standards/Buildora_AI_Backend_Engineering_Standards.md` | 38 KB | Structure + key sections |
| `docs/standards/Buildora_AI_Frontend_Engineering_Standards.md` | 38 KB | Structure + key sections |
| `docs/standards/Buildora_AI_Codex_Implementation_Reference.md` | 28 KB | Structure + key sections |
| `docs/standards/Buildora_AI_Solo_Developer_AI_Assisted_Development_Guide.md` | 23 KB | Structure |
| `docs/planning/Buildora_AI_Phase_Wise_Development_Checklist.md` | 52 KB | Structure + phases 0–9 |
| `docs/planning/Buildora_AI_Sprintwise_Project_Plan.md` | 43 KB | Structure + full sprint map |
| `docs/finance/Buildora_AI_Development_Production_Costing_Plan.md` | 64 KB | Structure + pricing model |
| `docs/finance/Buildora_AI_Return_On_Investment_Plan.md` | 60 KB | Structure |
| `docs/examples/Buildora_AI_Sample_Backend_CRUD.md` | 39 KB | Schema + ID generation |
| `docs/examples/Buildora_AI_Sample_Frontend_CRUD.md` | 45 KB | Structure |
| `docs/setup/*` (46 files) | ~90 KB | Index + spot checks |

**Total corpus reviewed: ~965 KB of documentation.**

## 0.2 Repository state

Every source directory is **empty**. `package.json`, `turbo.json`, and `pnpm-workspace.yaml` are **0 bytes**. `.github/workflows/` is empty.

```text
apps/{web,api,ai-worker,bim-worker,render-worker}   0 entries each
packages/{design-system,api-contracts,domain,building-model,
          cad-2d,engine-3d,qs-engine,cost-engine,ai-tools,units}  0 entries each
native/geometry-wasm                                0 entries
infrastructure/{environments,modules,scripts}       0 entries
```

**This is the single most favourable fact in this review.** There is no implementation debt, no migration burden, and no deployed state. Every P0 recommendation below costs documentation edits only — zero code migration. This is the cheapest possible moment to fix the issues identified here, and the cost of *not* fixing them rises steeply from Sprint 01 onward.

---

# 1. Executive Summary

## 1.1 The honest headline

**The domain architecture is excellent. The documentation system around it is broken.**

The intellectual core of Buildora AI — canonical Building Model, semantic commands, version/journal/snapshot persistence, deterministic QS→BOQ→Cost lineage, AI-proposes/domain-commits, credit ledger with reservations — is genuinely well designed. It is better than most seed-stage AEC platforms, and materially better than what a solo developer typically produces before writing code. The non-negotiables in `AGENTS.md` §9–§22 and §44–§48 are correct, specific, and enforceable.

But the documentation **system** has three structural defects that will actively damage AI-assisted implementation:

1. **~30 of the 34 canonical architecture documents referenced by `README.md`, `AGENTS.md` and `CLAUDE.md` do not exist.** The agent contracts route Codex and Claude to files that are absent.
2. **`docs/architecture/` and `docs/setup/` are byte-identical duplicates** of 46 setup files, plus 4 unique architecture documents. There is no single location for architecture.
3. **The product is named `Buildora AI` in every document** while `AGENTS.md` §89 forbids that name. Every canonical filename is wrong relative to the agent contract.

For a human team this is untidy. For an **AI-agent-driven build — this project's explicit delivery model — it is a first-order defect.** Codex and Claude are instructed to read documents that do not exist, to resolve conflicts using an ADR index that does not exist, and to use a product name that contradicts every file they open. The most likely failure mode is not that agents stop; it is that they **silently invent** the missing architecture, differently each session, and that invented architecture becomes the de facto design.

## 1.2 What this means practically

The architecture is not the problem. **The architecture's discoverability is the problem.** The fix is roughly 2–3 days of documentation consolidation, not a redesign. That work must happen **before Sprint 01**, because Sprint 01 is "Repository & Architecture Bootstrap" — the sprint that hardcodes package names, ID strategy, and module boundaries into real files.

## 1.3 Scores at a glance

| Area | Score | Note |
|---|---:|---|
| Product/domain architecture | 9 | Differentiation and flow are genuinely strong |
| Building Model | 8 | Excellent core; 4 specific gaps (§12) |
| 2D architecture | 8 | Correct boundaries; room-topology gap |
| 3D architecture | 8 | Correct; direct Three.js is the right call |
| Database | 9 | The strongest document in the repository |
| Backend | 9 | Modular monolith, thin controllers, RFC 9457 |
| Frontend | 8 | State matrix right; editor-load path unspecified |
| AI | 8 | Safety model correct; costing on fictional models |
| QS/cost | 9 | Fully auditable lineage; professional-grade |
| Billing | 9 | Ledger + reservation + inbox is correct and safe |
| Security | 7 | Good controls; RLS deferred, injection thin |
| DevOps | 8 | Well-calibrated for solo; slightly early on AWS |
| Observability | 7 | Correct direction, no defined beta minimum |
| Scalability | 9 | No rewrite required to 100k MAU |
| Solo-dev maintainability | 5 | **Lowest score.** ~2 years to revenue |
| Cost efficiency | 8 | Sound; pricing verified. Held back only by early provisioning |

Scores below 8 are explained in §10.

---

# 2. Answers to the 20 Primary Review Questions

**1. Is this architecture technically sound?**
Yes. The domain model, persistence strategy, determinism boundary, and tenancy model are all sound. I found no fatal technical flaw in the design itself.

**2. Is anything fundamentally wrong?**
Not in the architecture. Three things are fundamentally wrong in the *documentation system*: missing canonical documents, `architecture`/`setup` duplication, product-name contradiction. One thing is wrong in the *plan*: 38 sprints before any revenue for a solo developer.

**3. Are there contradictory decisions across the documents?**
Yes — 11 identified in §5. Material ones: ID strategy (UUID vs prefixed varchar), product naming, document paths, and the 3D API (WebGPU-first vs WebGL-first). **All are now resolved.**

**4. Is anything significantly over-engineered for a solo developer?**
Yes. Five `apps/*` deployables and ten `packages/*` before a line of code; Temporal + Redis + PostGIS + pgvector provisioned early; the full billing scope of §196 as MVP; `render-worker` and `native/geometry-wasm` existing at all right now. See §6.

**5. Is anything under-engineered that could cause a major rewrite later?**
Yes, four things, all in the Building Model: **room/space topology derivation**, **the level↔element vertical datum**, **grouping/assembly-instance identity**, and **the client-side model access path**. See §7 and §12.

**6. Are any technology choices inappropriate?**
No. Every major technology choice is defensible and I recommend keeping all of them. Two need conditions (Temporal, Clerk) and several need deferral. See §8.

**7. Are package/module boundaries correct?**
Mostly. `packages/domain` is referenced in five documents but **never defined anywhere** — a boundary with no meaning that will become a dumping ground. `packages/ai-tools` is premature. Otherwise correct.

**8. Is the canonical Building Model sufficient for 2D/3D/BIM/QS/costing/AI/collaboration?**
For 2D, 3D, QS, costing and AI: yes. For BIM: yes with the external-GUID mapping already specified. For collaboration: yes at the command level, but room topology and grouping gaps surface first. See §12.

**9. Is our Building Model persistence/versioning architecture correct?**
Yes — the best-argued part of the corpus. Current materialized state + append-only command journal + change items + periodic snapshots + `baseVersion` optimistic concurrency is exactly right. Do not change it.

**10. Is our database design appropriate for MVP and future scale?**
Yes. Hybrid relational+JSONB with schema-versioned geometry, direct `organization_id` on high-risk tables, UUID PKs, `NUMERIC` money, deferred partitioning. This document could go to production essentially as written.

**11. Is our AI architecture safe, cost-aware, and extensible?**
Safe: yes — the ChangeSet/draft/approval path is correct and `AGENTS.md` §16 forbids the dangerous pattern explicitly. Cost-aware: yes — pricing is verified and the alias + rate-card design insulates the product from provider price changes (ADR-010). Extensible: yes, via provider adapters and logical aliases.

**12. Is QS/BOQ/cost deterministic and auditable enough?**
Yes. `quantity_runs` pinning `model_version` + `ruleset_version` + `calculation_engine_version`, `quantity_item_sources` for element traceability, and the six-part cost reproducibility tuple meet professional QS standards — better than several commercial products.

**13. Is the billing architecture financially safe?**
Yes. Ledger-not-balance, reserve→settle→release, webhook inbox with idempotency, customer credits decoupled from provider tokens, and "never grant on browser redirect" together close every common revenue-loss hole.

**14. Is our workflow/Temporal usage appropriate?**
The *rules* are appropriate. The *timing* is not — Temporal should not be provisioned until Sprint 31 (Upload Plan pipeline), its first genuine use case. No BullMQ contradiction exists in the corpus.

**15. Is our frontend state architecture appropriate for a CAD/BIM application?**
Yes. The state ownership matrix is correct and "no giant Zustand store" is right. The gap: no document defines **how the editor loads and holds the working model client-side** — see P1-3.

**16. Are our PixiJS / Three.js boundaries correct?**
Yes. React shell → editor controller → renderer, with Pixi/Three objects as disposable views and domain IDs preserved, is the correct professional-editor architecture. Avoiding React Three Fiber for the core editor is right.

**17. Is our DevOps architecture too complex or too simple?**
Correctly scoped, slightly early in timing. Cloudflare + ECS/Fargate + Neon + R2 + Temporal Cloud with no Kubernetes is right for one person. The §251 "what not to deploy" list is excellent.

**18. Is the initial production architecture financially sensible?**
Yes, with one change: do not provision AWS production until there is a paying user or an external beta. Local Docker + Neon free tier covers Sprints 01–30.

**19. Can this scale 10 → 100,000+ users without a fundamental rewrite?**
Yes. The architecture has the right seams: modular monolith → service extraction, single Postgres → read replicas → partitioning, stateless ECS → autoscaling, derived artifacts already in object storage.

**20. What should we deliberately NOT build yet?**
`render-worker`, `native/geometry-wasm`, `packages/ai-tools`, PostGIS, realtime collaboration/Yjs, ODA, OpenCascade, GPU workers, Kubernetes, read replicas, multi-region, marketplace, MEP/structural, scheduling. (`packages/domain` is deleted outright, not deferred — ADR-011.) See §6.

---

# 3. Domain-by-Domain Review

## A. Repository / Monorepo

**Assessment: mostly correct, three boundary problems.**

The pnpm + Turborepo monorepo with `apps/` + `packages/` + `native/` + `infrastructure/` is right, and `AGENTS.md` §7's "one repository does not mean one deployment" is exactly the correct framing.

### Problems

**A1 — `packages/domain` is undefined (P0).**
It appears in `README.md`, the Master Product Reference §69, the Codex Reference §4, the Backend Standards, and a Dockerfile `COPY` line. **No document ever says what belongs in it.** Meanwhile `packages/building-model` has a precise definition. An undefined package next to a defined one is not a boundary; it is an invitation for every agent to put ambiguous code in `domain/`. Within ten sprints it becomes the `utils/` dumping ground `AGENTS.md` §68 forbids.

*Recommendation (as corrected — ADR-011):* **Delete `packages/domain`.** **Do not create `packages/core` in its place** — that relocates the same dumping-ground problem. Concepts stay with their owning module: building semantics → `building-model`; units → `units`; API schemas → `api-contracts`; billing domain → the `apps/api` billing module. `packages/core` may be introduced later only under the three conditions in `04_Monorepo_and_Module_Architecture.md` §5.1.

**A2 — Five deployables before one exists (P1).**
`ai-worker`, `bim-worker`, `render-worker` are separate apps from day one. `render-worker` (Blender photoreal) is not in the MVP at all and has no sprint before 38. Each empty app still costs a Dockerfile, CI path filter, ECS service, env config, and health checks.

*Recommendation:* Create `apps/web` and `apps/api` at Sprint 01. Create `apps/ai-worker` (Python/FastAPI) at Sprint 31. Create `apps/bim-worker` at Sprint 33. **Do not create `apps/render-worker` during the MVP.** Directories may exist with a `README.md` stating "not yet — see Sprint N", but no build config.

**A3 — Ten packages before one is used (P2).**
`packages/ai-tools` has no consumer until Sprint 27; its contents (typed tool schemas) are a thin layer over `api-contracts` + `building-model` and may never justify separation.

*Recommendation:* Start with `units`, `building-model`, `api-contracts`, `design-system`. Add `qs-engine`, `cost-engine` at Sprints 23/25; `cad-2d` at 12; `engine-3d` at 18. Defer `ai-tools` until Sprint 27 and merge into `api-contracts` if it stays thin.

### Dependency direction

No circular-dependency risk exists in the documented design, because the domain-purity rule (`AGENTS.md` §11) forces a strict layering. The allowed graph is specified in §11.2 of this review.

## B. Canonical Building Model

**Assessment: the strongest part of the architecture, with four specific gaps.**

What is right: single source of truth with 2D/3D/IFC/QS/BOQ/cost as derived representations; millimetres canonical; stable IDs across all subsystems; parametric-not-mesh geometry (DB §35); hybrid relational+JSONB with schema-versioned payloads (DB §31–34); `element_relationships` as a first-class table rather than JSON graph edges (DB §36); provenance (`source_kind`, `source_ref`, `confidence`, `verification_status`) built in from the start rather than retrofitted for recognition.

That provenance decision in particular is one most teams get wrong and pay for later. It is correct here.

### Gaps — see §12 for full detail

- **B1 (P0): Room/Space topology is undefined.** Rooms are listed as an element type, but nothing says whether a room's boundary is user-drawn or derived from wall geometry, nor what happens to a room when a bounding wall moves. Room area drives finishes QS and therefore cost. This is the single most expensive gap to fix late.
- **B2 (P0): Vertical datum is ambiguous.** Wall geometry has `heightMm` and a `levelId`, but no document defines whether `z` is measured from level datum or project datum, or where level elevation/floor-to-floor is stored. Every 3D adapter, every IFC export, and every volume quantity depends on this.
- **B3 (P1): Grouping/instance identity is missing.** No concept of a repeated assembly instance (a window type used 30 times, a bathroom pod, a stair flight). Needed for schedules, type-level property edits, and BIM round-trips.
- **B4 (P1): Openings are modelled twice.** `Opening` is a distinct element type, but the Door geometry example embeds `hostWallId`/`offsetMm`/`widthMm` directly. Two representations of the same physical hole will diverge in QS deduction logic.

## C. Command / Versioning Model

**Assessment: correct. Do not change it.**

`model_versions` + `model_commands` (journal) + `model_change_items` (diffs) + `model_snapshots` (periodic) + `baseVersion` optimistic concurrency + `MODEL_VERSION_CONFLICT` is the right design. It supports undo/redo, audit, AI preview, rollback, version compare, and future collaboration without event-sourcing the entire system — the pragmatic middle path, correctly chosen.

Notable correct details: commands carry `command_schema_version` (replay survives evolution); `model_change_items` avoids reconstructing diffs from scratch; `AGENTS.md` §37 pattern (drag preview client-side → one command on drag end) prevents journal explosion; drafts hold ChangeSets relative to base version rather than full model copies.

### Refinements

**C1 (P1): Snapshot cadence is undefined.** "Periodically" is not implementable. Specify: snapshot every N committed versions (suggest N=50) or when journal replay would exceed a target time; always snapshot before a bulk import or AI commit of >100 elements.

**C2 (P2): Undo across a version boundary needs a rule.** DB §228 says undo creates a compensating command — correct. But if user A undoes while user B has committed version N+1, the compensating command must be rejected or rebased. Define now, implement at collaboration time.

**C3 (P2): Not over-storing.** You are storing journal + change items + snapshots. This is *not* excessive — change items are what make revision comparison cheap, which is a headline product feature (CostX-style revision cost analysis). Keep all three.

## D. 2D CAD Engine

**Assessment: correct architecture, one significant gap.**

PixiJS + custom CAD logic is the right choice over Konva/Fabric (too high-level for CAD) or raw canvas (too much work). The React-shell/editor-controller/Pixi-renderer split with an explicit mm↔px transform and disposable Pixi views is professional-grade.

**D1 (P1): Room detection and constraints are unassigned.** Sprint 15 delivers "rooms in 2D" but no document specifies the algorithm (planar subdivision / half-edge / polygon boolean) or which package owns it. This is the hardest geometry in the 2D editor and the most likely source of a mid-project rewrite. It belongs in `building-model` (domain, testable headlessly), not `cad-2d`.

**D2 (P2): No constraint solver — correct for now.** Snapping and inferred alignment cover the MVP. A parametric constraint solver is a genuine "do not build yet."

**D3 (P2): Worker boundary undefined.** "Web Workers where justified" is right, but pre-commit to: room detection and large-model hit-testing go to a worker if they exceed 16ms at 5,000 elements. Measure first.

## E. 3D Engine

**Assessment: correct.**

Direct Three.js over React Three Fiber is the right call for a professional editor — R3F's reconciler fights you on imperative scene updates, incremental adapter diffing, and explicit resource disposal, all of which this design requires. `AGENTS.md` §8 correctly requires an ADR before adopting R3F.

The adapter pattern (canonical model → `engine-3d` adapters → Three scene), domain-ID↔object mapping, explicit geometry/material disposal, and "no business rules in scene-graph metadata" are all right.

**E1 — RESOLVED (ADR-017).** Master Product Reference §55 said "WebGPU where available, WebGL2 fallback"; `AGENTS.md` §8 said "WebGL stable path, WebGPU progressive enhancement later." `AGENTS.md` won. **WebGL2 is the MVP baseline, single render path; WebGPU deferred.** `01_Master_Product_Reference.md` §55 has been corrected.

**E2 (P2): Fragments/BIM path timing.** That Open Fragments is for *imported* IFC display, not for rendering Buildora's own model. Keep these two paths separate and explicit; do not let Fragments become the internal scene format.

## F. 2D ↔ 3D Synchronization

**Assessment: correct, and the rules are unusually well stated.**

`AGENTS.md` §37 and `CLAUDE.md` §18 both state the invariant correctly and name the forbidden patterns explicitly (2D patches 3D; 3D writes DB directly). Because both representations are pure functions of the canonical model, there is no synchronization problem to solve — only adapter correctness.

**F1 (P1): Hidden risk — adapter update granularity.** The documents say "model changed → adapters update" but not *how*. Full rebuild at 10,000 elements will drop frames; naive per-element diffing risks missing cascades (moving a wall must update its hosted doors, the bounding rooms, and possibly the slab edge). The `model_change_items` table already provides the diff — feed adapters from change items plus a documented cascade set, and test that cascade explicitly.

## G. BIM / IFC / CAD

**Assessment: correct decision, correct sequencing.**

"IFC is interchange, not the internal source of truth" is right. Many AEC startups adopt IFC as their internal model and are then permanently constrained by its representation choices. Avoiding that is a significant strength. External GUID mapping (`external_element_mapping`, DB §110) is exactly what round-tripping requires.

Sequencing: web-ifc + That Open for browser display and IfcOpenShell (Python) for server-side parse/export at Sprint 33 is right. **OpenCascade: do not build** — it enters only if true B-rep solid modelling becomes necessary, which the parametric model avoids. **ODA: do not build** — DWG support is a licensing decision (~$10k+/yr) that requires demonstrated professional demand.

## H. Database

**Assessment: the strongest document in the repository. Score 9.**

Correct throughout: one Postgres database, one schema; UUID PKs (UUIDv7 preferred) with `public_id` for human-readable references; `BIGINT` for monotonic counters; `NUMERIC` for all money; `TIMESTAMPTZ` everywhere; direct `organization_id` on high-risk tables even when derivable; hybrid relational+JSONB with mandatory schema versions and runtime validation; no meshes in relational columns; deferred partitioning with BRIN candidates identified; expand→migrate→switch→contract migrations; no migrations at app startup.

**No, a different database is not required.** PostgreSQL with JSONB handles parametric geometry correctly. The "geometry means NoSQL" instinct is wrong here: the geometry payloads are small, schema-versioned, and validated, while everything *around* them (tenancy, versions, quantities, money) is strongly relational and transactional. Postgres is the right choice.

**H1 (P1): RLS is deferred to "before paid production beta."** Reasonable, but it must be a hard gate, not an aspiration — and the `app.current_organization_id` session-variable pattern (DB §14) must be exercised in integration tests from Sprint 05, or retrofitting will be painful.

**H2 (P2): PostGIS is not needed for the MVP.** It serves real-world site geography (Sprint 39). `geography(Point)` for a project location does not require it early; enable the extension when Sprint 39 arrives.

## I. Multi-Tenancy / Authorization

**Assessment: correct, with one gap.**

Organization as tenant boundary, direct `organization_id` tagging, repository signatures carrying tenant scope (`findProjectById(organizationId, projectId)`), cross-tenant access as release-blocking, mandatory isolation tests — all correct. The "authenticated ≠ authorized" rule and the Buildora-owns-authorization stance (with Clerk behind an abstraction) are right.

**I1 (P1): Project-level RBAC is underspecified.** `project_members` exists with roles, and organization roles exist, but no document defines the interaction: can an org admin always read every project? Is a project-level viewer role able to see cost data? Cost visibility in particular is commercially sensitive in AEC — contractors do not show margin to clients. Define the permission matrix before Sprint 06.

## J. Backend

**Assessment: correct.**

NestJS modular monolith with bounded-context modules, thin controllers, use cases, module-owned repositories with explicit tenant scope, shared contracts in `api-contracts`, REST/OpenAPI, RFC 9457 problem details with stable error codes. This is textbook-correct and appropriate.

**Modular monolith is right.** Microservices become justified only when a module needs independent scaling (the AI and BIM workers already are separate processes — the correct first split), a different runtime (Python CV — already separate), or genuine fault isolation. None of these require breaking the API itself apart. Do not split `apps/api` before 10,000 MAU, and only then with measurement.

Modules that should stay internally isolated regardless: `billing` (financial invariants), `building-model` persistence (version integrity), `ai` (cost control surface).

## K. Frontend

**Assessment: correct, one specification gap.**

Server Components by default, Client Components only where needed, TanStack Query for server state, Zustand strictly for ephemeral editor/UI state, RHF+Zod forms, centralized API client, design tokens in the design system. The state ownership matrix (Frontend Standards §10) is genuinely well done and correctly assigns "authoritative walls" to backend/domain.

**K1 (P1): The editor's model access path is undefined.** This is the most important frontend gap. A CAD editor cannot fetch walls through TanStack Query per interaction; it needs an in-memory working model that renders at 60fps, applies optimistic commands locally, and reconciles with server versions. No document says what holds it. See P1-3 for the recommended design.

**K2 (P2): Next.js is correct but the editor route should be a client island.** The editor page is inherently client-side; use Server Components for the shell/chrome and dynamic-import the editor bundle. Do not let this justify `"use client"` on whole route trees.

## L. AI Architecture

**Assessment: safety model correct; economics built on fiction.**

Correct: AI Gateway with provider adapters; logical aliases (FAST/BALANCED/ADVANCED/EXPERT) with provider IDs in configuration; typed, runtime-validated tools; no raw DB/SQL access for the LLM; ChangeSet → draft → validate → QS/cost impact → preview → approve → commit; provider usage accounting separate from customer credits; escalation with cost tracking; prompt caching as an explicit strategy.

**AI cannot bypass deterministic domain logic** — this is architecturally enforced, not merely stated, because AI writes only to `ai_change_sets`/`ai_change_items`, and only the domain commit path writes `building_elements`. That is the right enforcement mechanism.

**L1 — WITHDRAWN.** This review originally asserted that the model names and prices in the costing plan were fabricated. **That assertion was incorrect.** Re-verified against official OpenAI API documentation on 2026-09-12: `gpt-5.6-luna` ($0.20 / $0.02 / $1.20), `gpt-5.6-terra` ($2.00 / $0.20 / $12.00), `gpt-5.6-sol` ($4.00 / $0.40 / $20.00) and `gpt-6-astra` ($10.00 / $1.00 / $50.00) per 1M tokens are all accurate, as is the `gpt-5.6-sol` promotional expiry of 21 November 2026.

The surviving architectural point stands and is stronger than the retracted one: **provider models and prices belong in versioned configuration with a `verified_at` date, behind logical aliases** (FAST/BALANCED/ADVANCED/EXPERT), never in domain code or architecture prose. Recorded as **ADR-010**. The one genuine commercial caution is that Sol's promotional rate must not be assumed permanent in long-term Pro-plan economics.

**L2 (P1): Prompt-injection defence is thin.** `CLAUDE.md` §31 mentions it for RAG, but uploaded construction documents are untrusted input flowing into a tool-calling agent that can propose model changes. Required: untrusted document content clearly delimited in prompts; tools never invoked directly from document text; ChangeSets from document-derived context flagged for mandatory human approval; no tool that can exfiltrate cross-project data.

**L3 (P2): Cost-reduction opportunities.** The architecture already supports the main levers (caching, routing, escalation). Add: cache the project-summary context block across a session; give tools narrow, paginated returns rather than whole-model dumps; make "explain" answers use FAST by default.

## M. AI / Building Modification Flow

**Assessment: complete, with two additions.**

The documented flow (prompt → gateway → ChangeSet → draft → validation → QS → cost → preview → approval → commit) is correct and covers more than most products ship.

**M1 (P1): Base-version staleness at approval.** A ChangeSet is created against `base_model_version`, but the user may approve minutes later after other edits. The commit must re-check `baseVersion` and either rebase or fail with `MODEL_VERSION_CONFLICT`. Documented for direct edits; must be explicit for ChangeSets too.

**M2 (P2): Partial approval.** Users will want to accept 8 of 10 proposed changes. `ai_change_items` already has per-item rows and `validation_status`, so the data model supports it — state the policy (all-or-nothing for MVP, per-item later).

## N. Plan Recognition

**Assessment: the hybrid approach is correct and clearly superior to LLM-only.**

Vector-first extraction (never rasterize a good vector source), CV for raster, AI for semantics and ambiguity only, confidence scores, mandatory human verification for low confidence, then Building Model commands. `AGENTS.md` §39 states the key insight precisely: "AI handles ambiguity and semantics, not authoritative raw geometry when precise vector data exists."

An LLM-only approach would be worse on precision (hallucinated dimensions), cost (vision tokens per page), auditability (no provenance), and correctness (no confidence signal). The chosen design wins on all four. The `confidence` + `verification_status` + `source_ref` columns already in `building_elements` make it implementable.

**N1 (P2): Scale detection is the hard problem.** Deriving mm-per-pixel from a scale bar, a known dimension string, or user input is the highest-risk step — a scale error silently corrupts every downstream quantity. Treat scale as an explicit, always-user-confirmed step, never inferred silently.

## O. QS / BOQ / Cost

**Assessment: professional-grade. Score 9.**

This meets the bar for professional QS use. The reproducibility tuple — model version + quantity run + BOQ version + rate book version + cost engine version + assumption set — is exactly what auditability requires, and DB §71's rule ("never recalculate an old estimate using a new rate book and call it the same version") is the single most important sentence in the costing design.

Also correct: `quantity_item_sources` giving element-level traceability for grouped quantities; `calculation_trace_jsonb` per quantity item; gross/deduction/net/waste/final as separate persisted columns; versioned rule sets with `rules_hash`; assemblies expanding elements into materials/labour/plant/waste; configurable waste profiles; `NUMERIC` throughout; the canonical golden test (5000×2700 wall − 1000×2100 opening = 11.4 m²) enshrined in `AGENTS.md` §20.

**O1 (P1): Measurement standard is unstated.** Professional QS in the UK follows NRM2 or SMM7; elsewhere POMI or local equivalents. Which standard governs deduction rules (e.g. are openings under 0.5 m² deducted?) is a *product* decision with direct cost-accuracy consequences. `qs_rule_sets` correctly makes this versionable data — but the MVP's default ruleset must be named and documented.

## P. Documents / RAG

**Assessment: correct, and Elasticsearch is genuinely unnecessary.**

Documents → revisions → pages → chunks → embeddings with explicit embedding profiles, pgvector, tenant/project filtering applied with (not after) ranking, and full citation lineage (answer → retrieval run → chunk → revision → file). `AGENTS.md` §42 correctly forbids Elasticsearch before Postgres FTS/trigram/pgvector proves insufficient — that threshold will not be reached at this scale.

**P1-a (P2): Filter-before-search must be structural.** With pgvector, applying a tenant filter after ANN retrieval can silently return too few rows or leak across tenants under index conditions. Enforce partial indexes or pre-filtered queries and add a cross-tenant retrieval test.

## Q. Workflows

**Assessment: rules right, timing wrong.**

Should use Temporal: plan recognition (long, multi-step, human-in-loop, resumable), BIM import/export, report generation, render jobs, project deletion, billing reconciliation, credit expiry.

Should **not** use Temporal: project CRUD, model commands (synchronous transactions), QS/cost runs for normal models (fast enough to be synchronous — make them workflows only if they exceed a few seconds), sending one email.

**Is BullMQ needed at all? No.** Redis-backed queues would duplicate Temporal's role and add a second failure mode. The corpus contains no BullMQ contradiction — this is already resolved correctly.

**Q1 (P1): Defer Temporal to Sprint 31.** Nothing before the upload pipeline needs durable orchestration. Sprints 01–30 need at most a `processing_jobs` table and a simple in-process worker. Adding Temporal Cloud at Sprint 02 buys a vendor, a local dev dependency, and replay-compatibility constraints ~7 months before first use.

## R. Billing

**Assessment: financially safe. Score 9.**

The architecture protects against all four named risks:

- **Duplicate charges** — webhook inbox with unique event IDs, idempotency keys on credit operations, Stripe client idempotency keys.
- **Retries** — reservation/settlement means a retried provider call settles once against the reservation; §73 explicitly forbids double-charging on retry.
- **Runaway AI cost** — reserve before work, per-request budgets, margin guardrails, cost-aware routing, kill switches, anomaly detection.
- **Provider price changes** — customer credits are decoupled from tokens; rate cards are versioned data; routing can change without touching plan language. This is the single best decision in the billing design.

Also correct: ledger not balance; `GRANT/RESERVE/RELEASE/CONSUME/REFUND/REVOKE/ADJUSTMENT/EXPIRE`; never grant on browser redirect; Stripe authoritative for payments, Buildora authoritative for entitlements; `NUMERIC` money; immutable usage events; reconciliation jobs.

**R1 (P1): MVP billing scope is too large.** §196 lists ~25 capabilities as "build first," including top-ups, upgrade/downgrade proration, grace periods, reconciliation, and admin UI. For a solo developer that is 2–3 sprints, not one (Sprint 36). Cut the first release to: plan catalog, entitlements, Stripe Checkout subscription, webhook inbox, wallet + ledger + grants, reserve/settle/release, provider usage records, customer usage UI, cancel. Defer top-ups, proration, downgrade rules, and reconciliation automation to a second billing sprint.

## S. DevOps

**Assessment: appropriate for a solo developer; two timing changes.**

The V1 topology (Cloudflare → ALB → ECS Fargate web+api → Neon/Redis/Temporal Cloud/R2) with no Kubernetes is correct. Build-once-promote-by-digest, no `latest`, OIDC to AWS, migrations as an explicit pipeline step separate from app boot, IaC with remote state, staging before production — all correct. §251's "what not to deploy initially" list is excellent and should be treated as binding.

**Answers to the specific questions:**

- **Can anything be simplified?** Yes — see the two items below. Also consider deploying `web` to Cloudflare Workers/Pages rather than ECS, removing one ECS service, the ALB path for web, and NAT costs. Keep `api` on ECS.
- **What should NOT be provisioned during development?** AWS entirely (ECS, ALB, NAT, ECR), Temporal Cloud, managed Redis, Sentry paid tier, staging. Sprints 01–30 run on local Docker Compose + Neon free tier.
- **When should AWS production be introduced?** At the first external beta user or Sprint 35 — whichever is earlier. Not at Sprint 02.
- **When would RDS/Aurora replace Neon?** When Neon's cost at steady load exceeds RDS equivalent, when you need >1 read replica with fine control, or when a compliance requirement demands VPC-local data. Realistically past 10,000 MAU. Neon's branching is worth real money to a solo developer before then.
- **When would Kubernetes be justified?** Realistically never for this product. The honest triggers: >15 distinct services, a platform team of 3+, or a hard multi-region active/active requirement. ECS/Fargate covers 100k MAU.

**S1 (P1): NAT Gateway is a silent cost trap.** ~$32/month base plus data processing, per AZ, charged whether or not traffic flows. For a pre-revenue solo project this is one of the largest avoidable line items. Use VPC endpoints for S3/ECR/Secrets Manager, or a single-AZ NAT, or public-subnet Fargate with strict security groups during early production.

**S2 (P2): Terraform vs OpenTofu is unresolved.** Pick OpenTofu (licence certainty, drop-in) and record an ADR. Low stakes, but leave it decided rather than open.

## T. Security

**Assessment: good controls, three gaps.**

Strong: tenant isolation as release-blocking; private-by-default uploads with presigned URLs and finalize steps; parser isolation in dedicated workers; secrets never in Git with a defined rotation path; explicit never-log list (tokens, keys, presigned URLs, document contents); no raw DB access for LLMs; OIDC rather than long-lived cloud keys; RFC 9457 errors without stack traces.

**T1 (P1): RLS as defence-in-depth is deferred.** Application scoping is the primary control and is well specified — but a single missing `WHERE organization_id` in a repository is a cross-tenant breach. RLS turns that from a breach into an empty result set. Make RLS a hard gate before the first external user, and exercise the session-variable pattern from Sprint 05.

**T2 (P1): SSRF via document/plan import.** If a user can supply a URL for import, the fetch happens from inside the VPC with an instance role. Required: no user-supplied URL fetching in MVP (upload only), or a strict allowlist plus egress-restricted worker.

**T3 (P2): Presigned URL scope.** Bind them to exact key, short TTL (≤5 min for upload, ≤60 s for download redirects), and content-type/size conditions. Never issue prefix-scoped credentials.

## U. Testing

**Assessment: correct emphasis, needs a concrete minimum.**

The documents correctly target tenant isolation, Building Model, version conflicts, geometry invariants, QS, cost, billing/credits, webhook idempotency, AI ChangeSets, and migrations — and correctly avoid blanket coverage mandates.

**The tests that actually matter, ranked:**

1. **Tenant isolation** — parameterised across every tenant-owned repository: org A cannot read/write org B. Cheapest possible insurance against the one defect class that ends the company.
2. **QS golden tests** — the 11.4 m² case plus a fixture set per measurement class (linear, area, volume, count), each pinned to a ruleset version.
3. **Model round-trip** — create → persist → reload → deep-equal. Catches serialization drift, the defect that silently corrupts saved projects.
4. **Optimistic concurrency** — two concurrent commands on the same base version; exactly one succeeds, the other returns `MODEL_VERSION_CONFLICT`.
5. **Credit concurrency** — parallel reservations against a wallet with insufficient balance never go negative (DB §252 already calls for this).
6. **Webhook idempotency** — same Stripe event delivered twice grants credits once.
7. **Cost reproducibility** — recomputing an old estimate with its pinned versions yields byte-identical totals.
8. **Migration tests** — every migration runs forward on a production-shaped fixture.
9. **2D↔3D cascade** — moving a wall updates hosted doors, bounding rooms, and the 3D adapter.
10. **Geometry invariants** — property-based where cheap (wall net area ≤ gross area; openings within host bounds).

Do **not** unit-test: React presentational components, thin controllers, generated Drizzle types, design-system variants.

## V. Observability

**Assessment: correct direction, no defined beta minimum.**

Sentry + OpenTelemetry + structured logs with `traceId`/`organizationId`/`projectId`/`modelVersion`, health/readiness separation, AI and billing telemetry — all correct. Liveness must not fail because OpenAI is down (§59) is a detail many teams get wrong.

**The minimum for beta (everything else deferred):**

- Sentry on `web` and `api` with release tagging and source maps.
- Structured JSON logs to CloudWatch with the standard identifier set.
- Four alerts only: API 5xx rate, DB connection saturation, failed Stripe webhook processing, AI spend/day over threshold.
- Two dashboards: platform health, AI cost/margin.
- Health + readiness endpoints.

Full OTel tracing, SLO/error-budget machinery, and the four-dashboard suite in §110 are post-launch.

## W. Performance

**Predicted bottlenecks by model size:**

| Elements | Expected behaviour | Likely first bottleneck |
|---:|---|---|
| 1,000 | Comfortable everywhere | None |
| 10,000 | Fine with care | Full-scene rebuilds on change; React re-render leakage into the canvas |
| 50,000 | Requires engineering | Room/topology recomputation; Three.js draw calls without instancing/merging; whole-model JSON payload size |

**By user scale:**

| MAU | Expected constraint | Correct response |
|---:|---|---|
| 1,000 | None | Nothing |
| 10,000 | DB connections; AI spend variance | PgBouncer/Neon pooling; budget controls |
| 100,000 | Read-heavy queries; journal/audit table size | Read replica; partition `audit_events`, `model_commands`, `ai_provider_calls` by time |

**Where profiling — not speculation — should decide:**

- **Rust/WASM** — only if polygon boolean/offset/room detection exceeds frame budget at realistic sizes in TypeScript. Most likely candidate, still not MVP.
- **Go** — no identified need. Do not introduce a fourth language.
- **Read replicas** — when primary read CPU is sustained high and reads dominate.
- **GPU workers** — only for photoreal rendering, which is post-MVP.
- **Kubernetes** — see §S.
- **Partitioning** — when a single table passes ~50–100M rows.

## X. Cost / ROI

**Expensive decisions that do not pay for themselves yet:**

- Temporal Cloud from Sprint 02 (~7 months before first use).
- AWS production infrastructure before external users — NAT + ALB + ECS baseline is meaningful monthly burn at zero revenue.
- `render-worker` and GPU capacity for a post-MVP feature.
- Paid observability tiers pre-beta.

**Managed services that clearly justify their cost for a solo founder:**

- **Neon** — branching per PR replaces meaningful DevOps work; free tier covers development.
- **Clerk** — auth, orgs, invitations, and MFA are weeks of work, and getting them wrong is a security incident. Correctly kept behind an abstraction.
- **Stripe** — non-negotiable.
- **R2** — zero egress fees matter for a product serving 3D artifacts and PDFs.
- **Temporal Cloud** *(once needed)* — self-hosting Temporal is a genuine operational burden.
- **Sentry** — first tool that pays for itself the day something breaks.

**On pricing:** the finance documents' provider pricing was re-verified on 2026-09-12 and is accurate. The one standing caution is that `gpt-5.6-sol`'s promotional rate expires no earlier than 21 November 2026 and must not be assumed permanent in Pro-plan economics. Rate cards carry `verified_at` so staleness is visible (ADR-010).

---

# 4. Contradiction Audit

| # | Topic | Document A | Document B | Conflict | Recommended Final Decision |
|---|---|---|---|---|---|
| 1 | Product name | `AGENTS.md` §89 — "Do not introduce `BuildWise`"; use "Buildora AI" | Every document in `docs/` is titled and named `Buildora_AI_*` | The agent contract forbids the name used by 100% of the corpus | **Rename all documents and headings to Buildora AI** in one dedicated commit before Sprint 01. Keep a one-line note that Buildora AI was the former name. |
| 2 | Canonical doc paths | `AGENTS.md` §3, `CLAUDE.md` §3, `README.md` reference `01_Master_Product_Reference.md`, `03_System_Architecture.md`, `05_Building_Model_Domain.md`, `12_QS_Engine.md`, `34_ADR_Index.md`, etc. | Only 4 architecture docs exist, all named `Buildora_AI_*` | ~30 referenced canonical documents do not exist; agents are routed to missing files | **Create `docs/README.md` as the real index**, renumber the 4 real documents, and mark every not-yet-written document explicitly as "NOT YET WRITTEN — nearest source: X". |
| 3 | `docs/architecture` vs `docs/setup` | 46 byte-identical files in both directories | — | No single source for architecture; edits will diverge | **Keep `docs/setup/` for the 46 setup guides. Delete the duplicates from `docs/architecture/`**, leaving only true architecture documents there. |
| 4 | Primary key strategy | DB Architecture §7 — UUID PKs (UUIDv7 preferred) | Backend CRUD example — `varchar(64)` PK, IDs generated as `prj_<uuid>` | Two incompatible ID strategies; the example is the one Codex will copy | **UUID PKs.** Add `public_id varchar` for human-readable references. **Rewrite the CRUD example** — the DB doc already flags this and defers to an ADR; decide it now (ADR-004). |
| 5 | 3D graphics API | Master Product Reference §55 — "WebGPU where available, WebGL2 fallback" | `AGENTS.md` §8 — "WebGL stable path, WebGPU progressive enhancement later" | WebGPU-first vs WebGL-first | **Resolved — ADR-017.** WebGL2 baseline for MVP; WebGPU deferred. `01_Master_Product_Reference.md` §55 corrected. |
| 6 | AI model identity | Costing Plan §5 — real model IDs and rates, verified 2026-09-12 | `AGENTS.md` §18 — logical aliases FAST/BALANCED/ADVANCED/EXPERT, provider IDs in config | **Not a contradiction.** Finance documents legitimately cite concrete verified rates; domain code uses aliases. The original "fabricated" finding is withdrawn. | **Resolved — ADR-010.** Keep the alias architecture; hold provider IDs and rates in versioned rate cards carrying `verified_at`. |
| 7 | Model alias naming | Master Product Reference §32 — `FAST_MODEL`/`BALANCED_MODEL`/... | Billing §72 and Codex Ref §11 use Luna/Terra/Sol/Astra in flow examples | Two vocabularies in worked examples | **Use the logical aliases everywhere in code and docs.** Confirmed non-blocking — the Codex Reference already states provider names must be config — but normalise the examples. |
| 8 | Repository root name | Master Product Reference §69 — `buildora/` | `AGENTS.md` §89 — `buildora-ai`; actual repo is `buildora-ai` | Stale path in docs | **`buildora-ai`.** Fix with the §1 rename. |
| 9 | `packages/domain` | Listed in `README.md`, MPR §69, Codex Ref §4, Backend Standards, Dockerfile | Never defined in any document | A package boundary with no definition | **Resolved — ADR-011.** Delete `packages/domain`. **Do not create `packages/core` yet**; keep concepts with their owning module. |
| 10 | MVP scope | Codex Ref §38 — MVP includes AI brief, plan recognition, AI Copilot, reports (16 items) | Sprint plan — those land at Sprints 27–35, with paid launch at 38 | "MVP" means two different scopes | **Adopt the sprint plan's staging** and rename Codex Ref §38 to "V1 Commercial Scope". Define a narrower true MVP (§13). |
| 11 | ROI duplicate file | `Buildora_AI_Return_On_Investment_Plan.md` | `Buildora_AI_Return_On_Investment_Plan copy.md` | Byte-identical duplicate | **Delete the ` copy` file.** |

**Checked and found NOT contradictory** (worth recording, since the brief asked): BullMQ vs Temporal — BullMQ appears nowhere; Temporal is uncontested. Auth — Clerk is consistent across all documents, always behind a Buildora abstraction. Deployment target — AWS ECS/Fargate is consistent; no Vercel/Kubernetes conflict. Source of truth — the canonical Building Model is stated identically in every document. Money — `NUMERIC`/decimal is consistent everywhere. Tenancy — organization-as-tenant is consistent.

---

# 5. Over-Engineering Audit

| Component | Classification | Reasoning |
|---|---|---|
| Modular monolith (`apps/api`) | **MUST HAVE NOW** | Correct for solo scale |
| PostgreSQL (single DB) | **MUST HAVE NOW** | Authoritative state |
| Canonical Building Model + commands + versions | **MUST HAVE NOW** | The product |
| `packages/units` | **MUST HAVE NOW** | Cheap; expensive to retrofit |
| Drizzle + explicit migrations | **MUST HAVE NOW** | Foundational |
| Clerk (behind abstraction) | **MUST HAVE NOW** | Buys weeks |
| R2 object storage | **MUST HAVE NOW** | Needed at first upload |
| Sentry | **MUST HAVE NOW** | Free tier; pays for itself |
| GitHub Actions CI | **MUST HAVE NOW** | Cheap, compounding |
| PixiJS 2D engine | **MUST HAVE NOW** (Sprint 12) | Core |
| Three.js 3D engine | **MUST HAVE NOW** (Sprint 18) | Core |
| QS / cost engines | **MUST HAVE NOW** (Sprints 23–25) | The differentiator |
| **Redis** | **SOON** (first need: rate limiting / sessions, ~Sprint 27) | Not needed Sprints 01–26. Local Docker until then. |
| **Temporal** | **SOON** (Sprint 31) | Real need begins with the upload pipeline. **Do not provision at Sprint 02.** |
| **pgvector** | **SOON** (Sprint 34) | Extension enable — trivial to add later |
| **AWS production** | **SOON** (Sprint 35 or first external user) | Local + Neon covers all prior development |
| Stripe billing (reduced scope) | **SOON** (Sprint 36) | Needed only at monetisation |
| IFC import/export | **SOON** (Sprint 33) | Professional credibility, not MVP survival |
| **PostGIS** | **LATER** (Sprint 39) | Site/GIS analysis only |
| **Rust/WASM** | **LATER** — profiling-gated | Only if TS geometry misses frame budget |
| Read replicas | **LATER** — metric-gated | Past 10k MAU |
| Partitioning | **LATER** — ~50M rows | Already correctly deferred |
| Blue/green deploys | **LATER** | Rolling is fine early |
| **`render-worker` / Blender** | **DO NOT BUILD YET** | Post-MVP feature; delete the app directory |
| **`native/geometry-wasm`** | **DO NOT BUILD YET** | No measured bottleneck exists |
| **`packages/ai-tools`** | **DO NOT BUILD YET** | No consumer until Sprint 27 |
| **`packages/domain`** | **DELETED (ADR-011)** | Undefined boundary. `packages/core` also NOT created. |
| **Realtime collaboration / Yjs / Liveblocks** | **DO NOT BUILD YET** | Single-user editing is the MVP. Command architecture already leaves the door open. |
| **GPU workers** | **DO NOT BUILD YET** | Follows rendering |
| **Kubernetes** | **DO NOT BUILD** | ECS covers 100k MAU |
| **Microservices** | **DO NOT BUILD** | Workers already provide the needed split |
| **Go services** | **DO NOT BUILD** | Would be a fourth language with no justification |
| **ODA (DWG)** | **DO NOT BUILD YET** | ~$10k+/yr licence; needs demonstrated demand |
| **OpenCascade** | **DO NOT BUILD YET** | Parametric model avoids B-rep need |
| **Multi-region** | **DO NOT BUILD** | No requirement at any modelled scale |
| Marketplace / leads / MEP / structural / scheduling | **DO NOT BUILD YET** | Sprints 39–47; correctly staged |

---

# 6. Under-Engineering Audit — Expensive to Change Later

Ranked by cost-of-late-change.

**1. Room/Space topology (P0).** Not specified at all. Room area drives finishes quantities and therefore a large share of cost. If rooms start as user-drawn polygons and later must become wall-derived, every saved project needs migration and every finish quantity changes. **Decide before Sprint 09.**

**2. Vertical datum / level elevation (P0).** Whether element `z` is level-relative or project-relative, and where floor-to-floor height lives, is undefined. Changing it later invalidates every stored geometry payload, every 3D adapter, every volume quantity, and every IFC export. **Decide before Sprint 09.** (Recommendation: level-relative element geometry; `levels.elevation_mm` from project datum; project datum at ground-floor finished floor level.)

**3. ID strategy (P0).** UUID vs prefixed varchar. Changing PK type after data exists means rewriting every table and FK. **Decide before the first migration** (ADR-004).

**4. Units (already correct).** Millimetres canonical, conversion isolated in `packages/units`. Correct and cheap now; catastrophic to retrofit. Keep it.

**5. Tenant ownership (already correct).** Direct `organization_id` on tenant-owned tables. Adding it later means backfilling and re-auditing every query. Correct.

**6. Provenance and confidence (already correct).** `source_kind`, `source_ref`, `confidence`, `verification_status` on elements from day one. Most teams add these after recognition ships and then cannot backfill history. Correct.

**7. Model versioning (already correct).** Version/journal/change-items from the first wall. Retrofitting history onto a mutable model is effectively impossible. Correct.

**8. Credit ledger (already correct).** Ledger-not-balance from the start. Reconstructing history from a mutable integer is impossible. Correct.

**9. Rate/version lineage (already correct).** Cost estimates pinned to rate-book and engine versions. Retrofitting reproducibility onto unpinned estimates cannot be done. Correct.

**10. Element grouping / type instances (P1).** No type/instance concept. Adding it later is a data migration plus schedule and QS rework. **Decide before Sprint 10.**

**11. Opening representation (P1).** Two competing representations. Consolidate before doors and windows ship at Sprint 10.

**12. Command schema versioning (already correct).** `command_schema_version` present from the start — this is what keeps the journal replayable as the domain evolves. Correct and often missed.

---

# 7. Technology Review

| Technology | Decision | Reason | Alternative Considered | When Re-evaluate |
|---|---|---|---|---|
| **Next.js (App Router)** | **KEEP** | Correct SC/CC discipline documented; editor as client island | Vite SPA + separate API | If SSR provides no value for the app shell by V1 |
| **NestJS** | **KEEP** | Modular monolith, DI, thin controllers, OpenAPI — matches the documented standards | Fastify + manual structure | Not before 10k MAU |
| **TypeScript everywhere** | **KEEP** | One language across browser geometry, API, and domain packages is the single biggest solo-dev multiplier | — | Never |
| **Drizzle** | **KEEP** | SQL-first, typed, explicit migrations; correct for versioned/JSONB schemas | Prisma | If migration ergonomics block work |
| **PostgreSQL** | **KEEP** | Transactions + JSONB + pgvector + PostGIS in one engine | — | Never |
| **Neon** | **KEEP WITH CONDITIONS** | Branch-per-PR is worth real money solo; free tier covers dev | RDS/Aurora, Supabase | When cost exceeds RDS equivalent or VPC-local data is required |
| **Redis / Valkey** | **DEFER** | No authoritative use; needed for rate limiting/presence later | In-memory (single instance) | Sprint 27 |
| **Temporal** | **KEEP WITH CONDITIONS** | Right for recognition/BIM/report pipelines; wrong to provision early | Postgres job table + worker | **Provision at Sprint 31, not Sprint 02** |
| **PixiJS** | **KEEP** | Right abstraction level for CAD; WebGL batching without a scene-graph framework | Konva, raw canvas | If 50k-element hit-testing fails |
| **Three.js (direct)** | **KEEP** | Imperative control needed for adapter diffing and disposal | React Three Fiber | Only via ADR |
| **Rust/WASM** | **DEFER** | No measured bottleneck | TS geometry + workers | After profiling the 2D engine at 10k+ elements |
| **Python / FastAPI** | **KEEP** | CV/ML ecosystem (OpenCV, PyTorch, IfcOpenShell) is genuinely Python-only | TS-only | Never — it is justified |
| **OpenAI** | **KEEP WITH CONDITIONS** | Adapter + alias architecture makes provider swap cheap | Anthropic, multi-provider | Verify real model IDs and pricing **before** publishing plan prices |
| **pgvector** | **KEEP (defer enabling)** | Sufficient at this scale; avoids a second datastore | Pinecone, Elasticsearch | If recall/latency degrades past ~10M chunks |
| **PostGIS** | **DEFER** | Only needed for real-world site analysis | Plain lat/lng columns | Sprint 39 |
| **Cloudflare R2** | **KEEP** | Zero egress matters for 3D artifacts and PDFs | S3 | Never |
| **Cloudflare (DNS/WAF/CDN)** | **KEEP** | Cheap, effective origin protection | AWS-native | Never |
| **AWS ECS/Fargate** | **KEEP WITH CONDITIONS** | No cluster to manage; right ceiling | Fly.io, Railway, Render | **Provision at first external user**, not during development. Consider Workers/Pages for `web`. |
| **Clerk** | **KEEP WITH CONDITIONS** | Orgs/invitations/MFA out of the box; correctly abstracted | Auth.js, self-hosted | If per-MAU cost becomes material, or B2B SSO needs exceed its offering |
| **Stripe** | **KEEP** | Correct source-of-truth split already documented | — | Never |
| **Sentry** | **KEEP** | Highest-value observability per pound | — | Never |
| **OpenTelemetry** | **KEEP WITH CONDITIONS** | Right long-term; full tracing is not a beta requirement | Logs only | Post-launch for full tracing |
| **That Open / web-ifc** | **KEEP** | Best browser IFC path; correctly scoped to imported models | Custom parser | Sprint 33 |
| **IfcOpenShell** | **KEEP** | The reference server-side IFC toolkit | — | Sprint 33 |
| **OpenCascade** | **DEFER** | Parametric model avoids the need | — | Only if true B-rep is required |
| **ODA SDK** | **DEFER** | Licence cost needs proven demand | DXF-only parsing | On paid professional demand for DWG |
| **Turborepo + pnpm** | **KEEP** | Correct monorepo tooling | Nx | Never |

**No technology is recommended for replacement.** The stack is well chosen.

---

# 8. Architecture Scoring — Explanations for Scores Below 8

**Security — 7.** The controls are right, but RLS (the defence-in-depth layer that turns a missing `WHERE` clause from a breach into an empty result) is deferred to an undated "before paid beta," SSRF through import paths is unaddressed, and prompt-injection handling for untrusted construction documents is one line in `CLAUDE.md`. Each is fixable cheaply now; none is fixable cheaply after a breach.

**Observability — 7.** The direction is correct and the AI/billing telemetry thinking is above average, but there is no stated *minimum* for beta. The documents describe a mature observability practice (SLOs, error budgets, four dashboards) that a solo developer will not build, and the absence of a defined floor makes it likely that nothing gets built instead. §V defines that floor.

**Cost efficiency — 8 (revised from 7).** The architecture is cost-aware in all the right structural ways (metering, reservations, caching, kill switches, alias-based routing), and the provider pricing underpinning the ROI model is verified rather than fabricated as originally stated. The remaining deduction is for early provisioning of Temporal Cloud, AWS and NAT before revenue — addressed by ADR-012.

**Solo-developer maintainability — 5.** This is the lowest score and the most important one. The architecture is sound, but the *plan built on it* asks one person to reach paid launch at Sprint 38 — realistically 18–24 months of unpaid work before the first pound of revenue, with a documentation system that currently routes AI agents to ~30 missing files. The architecture does not need simplifying; the **delivery sequence does** (§13), and the documentation system needs repairing (§9, P0-1).

---

# 9. Recommended Changes

## P0 — Must change before implementation continues

### P0-1 — Repair the documentation system

**Problem.** ~30 of 34 canonical architecture documents referenced by `README.md`, `AGENTS.md`, and `CLAUDE.md` do not exist. `docs/README.md` — which both agent contracts instruct agents to read as step 2 — is absent. `docs/architecture/` and `docs/setup/` hold 46 byte-identical duplicates. Every document carried the forbidden legacy name `BuildWise`.

**Why it matters.** The delivery model is AI-agent implementation. Agents instructed to read missing files will invent the missing architecture, differently each session, and those inventions will become the de facto design. This defect attacks the project's core delivery mechanism.

**Current design.** Documentation map describing 34 files; 4 real architecture documents under legacy `BuildWise_*` names; 46 duplicated setup files; no index; no ADR directory.

**Recommended design.**
1. Create `docs/README.md` as the authoritative index listing **only files that exist**, with a clearly marked "Not yet written — nearest source" section for the rest.
2. Delete the 46 duplicates from `docs/architecture/`; keep `docs/setup/` as their single home.
3. Rename all `Buildora_AI_*` files and in-document headings to `Buildora_AI_*` / "Buildora AI" in one dedicated commit.
4. Renumber the four real architecture documents to match the agent contracts (`01_Master_Product_Reference.md`, `15_Database_and_Data_Architecture.md`, `24_Billing_AI_Credits_and_Usage.md`, `27_DevOps_Environments_and_Deployment.md`).
5. Create `docs/architecture/adr/` with `0000-template.md` and `34_ADR_Index.md`.
6. Replace the root `README.md` (currently a stale doc map) with a real project README; move the map to `docs/README.md`.
7. Delete `Buildora_AI_Return_On_Investment_Plan copy.md`.
8. Update `AGENTS.md` §3 and `CLAUDE.md` §3 path lists to match reality.

**Migration impact.** Documentation only. No code exists.
**Documents affected.** All.
**ADR required.** No — this is repair, not decision.

### P0-2 — Define room/space topology

**Problem.** Rooms are listed as an element type with no definition of how their boundaries relate to walls.

**Why it matters.** Room area drives finishes quantities and therefore a material share of every cost estimate. Whether rooms are user-drawn or wall-derived changes the data model, the 2D editor, the QS engine, and the update cascade. Changing it after projects exist requires migrating saved geometry and re-deriving every finish quantity.

**Current design.** `Space / Room` appears in the hierarchy; `element_relationships` supports a `BOUNDS` type. Nothing further.

**Recommended design (as corrected).** Rooms are **derived-then-persisted**. The wall **centreline graph establishes connectivity** — which regions are enclosed. The **persisted boundary is then derived from interior wall faces**, offset inward from each bounding wall's centreline by that wall's half-thickness, with junctions resolved. Each room becomes an element with a stable ID, a persisted boundary, and `BOUNDS` relationships to its bounding walls. A bounding-wall change triggers recomputation (preserving ID, name, number and finishes) and flags material area changes. A `user_defined` room is never silently overwritten. **Area is always computed from the boundary**; a user-entered area is never authoritative geometry. Room detection lives in `packages/building-model` (headless, testable), not in `cad-2d`.

> **Correction applied.** The original text derived the boundary from centrelines. That overstates floor area by half a wall thickness on every edge — a 5000 × 4000 room in 200 mm walls would report 20.00 m² instead of the correct 18.24 m². Interior faces are required for defensible finishes quantities.

**Migration impact.** None now. Very large if deferred past Sprint 15.
**Documents affected.** Building Model domain doc (to be written), DB Architecture §31/§36, 2D editor doc, QS engine doc.
**ADR required.** **Yes — ADR-006 (accepted).**

### P0-3 — Define the vertical datum and level elevation model

**Problem.** No document states whether element `z` is level-relative or project-relative, or where level elevation and floor-to-floor height are stored.

**Why it matters.** Every 3D adapter, volume quantity, stair calculation, and IFC export depends on it. Changing it later invalidates every stored geometry payload.

**Current design.** `levels` table exists; wall geometry carries `heightMm`; the relationship between them is unstated.

**Recommended design (as corrected).** Project-local datum = ground-floor finished floor level = 0 mm. `levels.elevation_mm` is signed from project datum (basements negative) and is the **single authoritative vertical fact** for a level. Element geometry `z` is **level-relative**: `absoluteZ = level.elevationMm + (baseOffsetMm ?? 0)`, derived on read and never stored. Real-world georeference lives on `Site`, separate from building-local coordinates.

> **Correction applied.** The original text also placed `floor_to_floor_mm` and `structural_thickness_mm` on `Level` as stored values. That creates duplicate authority for the same physical fact. **Floor-to-floor is derived** (`nextLevel.elevationMm − level.elevationMm`); **slab structural thickness belongs to the Slab element and its assembly**. `Level` may carry an optional `defaultStoreyHeightMm` as a creation-time design aid only — never authoritative geometry.

**Migration impact.** None now. Severe if deferred past Sprint 09.
**Documents affected.** Building Model domain doc, DB Architecture §30/§31, 3D engine doc, QS engine doc.
**ADR required.** **Yes — ADR-005 (accepted).**

### P0-4 — Resolve the ID strategy

**Problem.** DB Architecture mandates UUID PKs; the backend CRUD example — the file Codex will pattern-match — uses `varchar(64)` with `prj_<uuid>` values.

**Why it matters.** Codex copies examples. If the first migration ships varchar PKs, changing to UUID later means rewriting every table and foreign key in a live database.

**Current design.** Two conflicting strategies; the DB document explicitly defers to an ADR that does not exist.

**Recommended design.** `UUID` primary keys, application-generated UUIDv7 where a stable library is available, UUIDv4 otherwise. Human-readable references use a separate `public_id` column (`PRJ-8K4D2A`), unique per tenant, never a foreign key target. **Rewrite the backend and frontend CRUD examples** to match.

**Migration impact.** None now. Total schema rewrite if deferred.
**Documents affected.** DB Architecture §7, both CRUD examples, Backend Standards.
**ADR required.** **Yes — ADR-004 (accepted).**

### P0-5 — WITHDRAWN (AI pricing model)

**This item is withdrawn.** The review asserted that the OpenAI models and prices
in the finance documents were fabricated. **That assertion was incorrect.**

Re-verified against official OpenAI API documentation on 2026-09-12:

| Model | Input / 1M | Cached input / 1M | Output / 1M |
|---|---:|---:|---:|
| `gpt-5.6-luna` | $0.20 | $0.02 | $1.20 |
| `gpt-5.6-terra` | $2.00 | $0.20 | $12.00 |
| `gpt-5.6-sol` | $4.00 | $0.40 | $20.00 |
| `gpt-6-astra` | $10.00 | $1.00 | $50.00 |

All four models exist and every rate is accurate, as is the `gpt-5.6-sol`
promotional expiry of 21 November 2026. **No correction to the finance documents
is required.**

**What survives** is a stronger architectural principle, now recorded as
**ADR-010**: provider models and prices are **versioned configuration** carrying
`provider`, `model_id`, `input_rate`, `cached_input_rate`, `cache_write_rate`,
`output_rate`, `effective_from`, `effective_to`, `verified_at` and
`source_reference`. Domain code uses only the logical aliases FAST / BALANCED /
ADVANCED / EXPERT. Historical provider cost resolves against the rate card in
force at the time of the call.

**The one genuine commercial caution:** Sol's promotional rate must not be
assumed permanent in long-term Pro-plan economics. Budget a margin, or route
normal workloads to Luna/Terra.

**Replacement item.** The architecture-lock set retains eight items; see P0-7 and
P0-8 below, which were previously tracked only in
`OPEN_ARCHITECTURE_DECISIONS.md`.

### P0-6 — Delete `packages/domain` (and do NOT create `packages/core` yet)

**Problem.** `packages/domain` is referenced in five documents and defined in none.

**Why it matters.** Sprint 01 creates the workspace. An undefined package created alongside defined ones becomes the dumping ground that `AGENTS.md` §68 forbids, and untangling it later means moving code across package boundaries with churn in every import.

**Current design.** Listed in every structure diagram; never specified.

**Recommended design (as corrected).** Delete `packages/domain`. **Do not create `packages/core` now** — pre-creating a generic "shared primitives" package before a demonstrated need simply relocates the same dumping-ground problem. Concepts stay with their owning module: units → `packages/units`; building semantics → `packages/building-model`; API schemas → `packages/api-contracts`; billing domain → the `apps/api` billing module.

`packages/core` may be introduced **later**, and only when all three hold: (1) multiple real packages need the same genuinely domain-neutral primitive, (2) duplication has actually appeared rather than been predicted, and (3) a precise charter can be written that excludes building semantics.

> **Correction applied.** The original text recommended creating `packages/core` immediately.

**Migration impact.** None now.
**Documents affected.** `README.md`, MPR §69, Codex Ref §4, Backend Standards, `AGENTS.md` §7.
**ADR required.** No.

### P0-7 — Consolidate opening representation

**Problem.** `Opening` existed as a distinct element type while Door/Window geometry embedded `hostWallId`/`offsetMm`/`widthMm` directly — two representations of one physical hole.

**Why it matters.** QS net-area deduction must find every opening in a wall. Two representations mean two code paths and a near-certain divergence bug in net wall area, the most important QS number. Doors and windows ship at Sprint 10.

**Recommended design.** **One representation.** A door or window *is* the opening: it is a wall-hosted element whose geometry creates the hole. A standalone `opening` element exists only for fixture-less holes (service penetrations, arches, unfilled openings). **QS sees hosted doors/windows and standalone openings through a single opening/deduction abstraction.**

**Migration impact.** None now; data migration plus QS rework after Sprint 10.
**Documents affected.** `05_Building_Model_Domain.md` §7.2–§7.3, DB §31, QS engine doc.
**ADR required.** No — clarification within ADR-001; recorded in the Building Model domain document.

*(Previously tracked as OD-04 in `OPEN_ARCHITECTURE_DECISIONS.md`.)*

### P0-8 — Add element type / instance identity

**Problem.** No concept of a repeated type instance — one window type used thirty times.

**Why it matters.** Window/door schedules are a core professional deliverable; type-level edits are expected behaviour; IFC round-trips carry type definitions. Retrofitting means migrating every element and reworking schedules and QS grouping.

**Recommended design.** Tenant-scoped, versioned `element_types` (code, category, default geometry, default properties) with an optional `element_type_id` on `building_elements`. Instances inherit and may override. **Type identity stays separate from assembly identity**: `element_type_id` answers *what it is*; `assembly_version_id` answers *what it costs*.

**Migration impact.** Small now; moderate after Sprint 10.
**Documents affected.** `05_Building_Model_Domain.md` §5.4, DB §31, QS/BOQ grouping, IFC mapping.
**ADR required.** **Yes — ADR-007 (accepted).**

*(Previously tracked as OD-05 in `OPEN_ARCHITECTURE_DECISIONS.md`.)*

---

## Architecture-lock set — reconciled

The original review listed six P0 items; `OPEN_ARCHITECTURE_DECISIONS.md` listed
nine "MUST DECIDE NOW" items. The two lists overlapped but were not identical.
**They are now reconciled into one set of eight**, all resolved:

| # | Item | Resolution |
|---|---|---|
| P0-1 | Repair the documentation system | `docs/README.md` created; duplicates deleted; renamed to Buildora AI; 6 canonical docs written; ADR directory created |
| P0-2 | Room/space topology | **ADR-006** — interior-face boundary, persisted stable identity |
| P0-3 | Vertical datum and level elevation | **ADR-005** — level-relative, single authoritative elevation |
| P0-4 | ID strategy | **ADR-004** — `uuid` PK + `public_id`; examples corrected |
| P0-5 | AI pricing model | **WITHDRAWN** — pricing verified accurate; principle kept as **ADR-010** |
| P0-6 | Delete `packages/domain` | **ADR-011** — deleted; `packages/core` deliberately not created |
| P0-7 | Opening representation | Single hosted representation; recorded in `05_Building_Model_Domain.md` |
| P0-8 | Element type/instance identity | **ADR-007** — `element_types` + optional FK |

Former OD-06 (client model store) is P1-3 below, resolved by **ADR-008**.
Former OD-07 (QS standard) is P1-7 below, resolved by **ADR-009**.
Former OD-08 and OD-09 map to P0-1 and P0-5 respectively.

---

## P1 — Should change before beta

### P1-1 — Consolidate opening representation — **SUPERSEDED BY P0-7**

> This item was promoted to the architecture-lock set as **P0-7** and is
> **resolved**. The text below is retained as the original P1 analysis.


**Problem.** `Opening` exists as an element type while Door/Window geometry embeds host and position directly — two representations of one physical hole.

**Why it matters.** QS deduction logic must find every opening in a wall. Two representations mean two code paths and a guaranteed divergence bug in net-area calculations — the single most important QS number.

**Current design.** Both forms documented side by side.

**Recommended design.** **One representation.** A Door/Window is an element hosted on a wall (`host_element_id`) carrying its own position and dimensions; the "opening" is *derived* from the hosted element for QS and geometry purposes. Keep a standalone `opening` element type only for holes with no fixture (service penetrations, arches). QS deduction reads hosted elements plus standalone openings through one interface.

**Impact.** Documentation only now; a data migration plus QS rework after Sprint 10.
**Documents affected.** Building Model domain doc, DB §31, QS engine doc.
**ADR required.** No — clarification within an accepted design.

### P1-2 — Add element grouping / type-instance identity — **SUPERSEDED BY P0-8**

> This item was promoted to the architecture-lock set as **P0-8** and is
> **resolved** by ADR-007. The text below is retained as the original P1 analysis.


**Problem.** No concept of a repeated type instance (one window type used 30 times).

**Why it matters.** Window/door schedules are a core professional deliverable; type-level edits ("make all W-02 windows 1500 wide") are expected behaviour; IFC round-trips carry type definitions. Retrofitting means migrating elements and reworking schedules and QS grouping.

**Recommended design.** Add `element_types` (tenant-scoped, versioned: code, category, default geometry and properties) and an optional `element_type_id` on `building_elements`. Instances inherit type properties and may override individually. Schedules group by type; QS may aggregate by type. Independent of assemblies, which remain the cost expansion mechanism.

**Impact.** Small now; moderate after Sprint 10.
**Documents affected.** Building Model domain doc, DB §31, QS/BOQ docs.
**ADR required.** **Yes — ADR-007 (accepted).**

### P1-3 — Specify the client-side editor model store

**Problem.** No document defines how the 2D/3D editor loads, holds, and mutates the working model client-side.

**Why it matters.** This is the central frontend architecture decision for a CAD application. Without it, the likely agent-invented outcomes are both wrong: TanStack Query per interaction (far too slow), or the whole model in a Zustand store (violating `AGENTS.md` §34 and creating a second authority).

**Current design.** The state matrix assigns "authoritative walls" to backend/domain and forbids a giant Zustand store — correct, but it leaves the working-model question unanswered.

**Recommended design (as corrected).** An **editor session store** owned by a **renderer-neutral `packages/model-session`** — not Zustand, not React state, and **not `packages/cad-2d`**: an in-memory instance of the canonical model loaded once per project/level, holding `baseVersion` and a pending command queue. Interactions apply commands locally for immediate feedback (optimistic), queue them to the API, and reconcile on response — on `MODEL_VERSION_CONFLICT`, reload and replay or surface the conflict. TanStack Query owns project metadata, lists, and QS/BOQ/cost read models; **it does not own the working geometry**. Zustand owns only ephemeral UI state (active tool, selection, panel visibility, camera/UI preferences, temporary interaction state). The store exposes a subscription that 2D and 3D adapters consume, satisfying `AGENTS.md` §37 with one update source.

`model-session` imports no framework — not React, Next, PixiJS, Three.js, NestJS, Drizzle or OpenAI — and is **not a second authoritative database**. PostgreSQL remains authoritative; the session is a synchronized working copy of the canonical model.

> **Correction applied.** The original text placed this store in `packages/cad-2d`. That would make the 3D engine conceptually depend on the 2D package. `model-session` sits between `building-model` and both renderers, so `cad-2d` and `engine-3d` each depend on it and neither depends on the other.

**Impact.** None now; a frontend rewrite if discovered at Sprint 13.
**Documents affected.** Frontend Standards §34/§43, 2D editor doc, 3D engine doc.
**ADR required.** **Yes — ADR-008 (accepted).**

### P1-4 — Defer Temporal, Redis, and AWS to first real need

**Problem.** Sprint 02 provisions local infrastructure and the DevOps plan introduces Temporal Cloud, managed Redis, and AWS early; none has a consumer before Sprint 27–31.

**Why it matters.** Each carries setup cost, local-dev friction, recurring spend, and — for Temporal — replay-compatibility constraints on all future workflow code, months before the first workflow exists. For a pre-revenue solo developer this is avoidable burn and avoidable cognitive load.

**Recommended design.** Sprints 01–26: Docker Compose with Postgres only; a `processing_jobs` table plus a simple in-process worker for any async need. Redis at Sprint 27 (AI rate limiting). Temporal at Sprint 31 (upload pipeline). AWS production at Sprint 35 or first external user, whichever is earlier. Keep the interfaces (job creation, status polling) stable so the Temporal swap is an implementation change, not an architecture change.

**Impact.** Reduces development-stage spend and setup work.
**Documents affected.** DevOps Plan §202, Sprint Plan Sprints 02/27/31, Costing Plan stages.
**ADR required.** No.

### P1-5 — Make RLS a hard gate and exercise it from Sprint 05

**Problem.** RLS is planned "before paid production beta" with no owning sprint.

**Why it matters.** Application scoping is a single missing `WHERE` clause away from cross-tenant disclosure — the one defect class that can end a B2B SaaS. RLS converts that from breach to empty result.

**Recommended design (as corrected).** From **Sprint 05**, all of the following are mandatory: `organization_id` on tenant-owned tables where required; every repository method explicitly organization-scoped; **negative cross-tenant integration tests for every tenant-owned repository**; RLS schema and policies **designed and written**; the transaction/session-context helper implemented and tested; and that helper **verified against the actual pooled PostgreSQL connection model** in use.

**Hard gate:** RLS must be enabled, tested and verified **before the first external beta**. Production beta is not possible without it.

> **Correction applied.** The original text said simply "enable RLS in Sprint 05". Forcing a poorly-understood session-variable configuration into every development path merely to satisfy a sprint number creates its own failure mode, particularly around connection pooling. The requirement is *designed, implemented and proven* from Sprint 05, with *enforcement* gated before external exposure. The goal is working isolation, not a checkbox.

**Impact.** Small at Sprint 05; significant at Sprint 37.
**Documents affected.** DB Architecture §13–§15, Sprint 05, Security doc.
**ADR required.** **Yes — ADR-016 (accepted).**

### P1-6 — Reduce first-release billing scope

**Problem.** §196's MVP billing list contains ~25 capabilities scheduled into a single sprint.

**Why it matters.** Billing is the least forgiving code in the product. Compressing top-ups, proration, downgrade rules, grace periods, reconciliation, and two UIs into one sprint produces either a slipped sprint or under-tested financial code.

**Recommended design.** **Billing Release 1 (Sprint 36):** plan catalog, entitlements, Stripe Checkout subscription, webhook inbox, wallet + ledger + monthly grants, reserve/settle/release, provider usage records, customer usage UI, cancel/reactivate. **Billing Release 2 (Sprint 36b):** top-ups, upgrade/downgrade proration, payment-failure grace, reconciliation jobs, admin billing UI, margin analytics. Ship Release 1 with a single paid plan.

**Impact.** Improves the odds of correct financial code.
**Documents affected.** Billing Architecture §196–§197, Sprint Plan Sprint 36.
**ADR required.** No.

### P1-7 — Name the QS measurement standard

**Problem.** No document states which measurement standard governs deduction and measurement rules.

**Why it matters.** Whether openings below a threshold are deducted, how wall areas are measured at junctions, and how work is itemised are standard-defined. Professional users will reject quantities that do not follow a recognised standard, and the choice shapes the default ruleset.

**Recommended design.** Name the MVP default explicitly (NRM2 for a UK-first launch) and encode it as the seeded `qs_rule_sets` record. The versioned-ruleset design already supports alternatives — this only fixes the default and documents the deduction rules it implies.

**Impact.** Documentation and seed data.
**Documents affected.** QS engine doc, DB §61.
**ADR required.** **Yes — ADR-009 (accepted).**

### P1-8 — Specify adapter update granularity and the cascade set

**Problem.** "Model changed → adapters update" does not define granularity, and the cascade set for a wall change is unstated.

**Why it matters.** Full rebuilds will drop frames at scale; naive per-element diffing will miss cascades (a moved wall must update hosted doors, bounding rooms, and possibly slab edges), producing visible 2D/3D divergence — exactly what the canonical-model architecture exists to prevent.

**Recommended design.** Adapters consume `model_change_items` (or the equivalent client-side command result) and apply incremental updates. Define the cascade set explicitly per element type — for a wall: hosted openings/doors/windows, `BOUNDS`-related rooms, connected walls at junctions. Add an integration test asserting that after a wall move, all three derived representations agree.

**Impact.** Prevents a class of subtle divergence bugs.
**Documents affected.** 2D editor doc, 3D engine doc, Building Model domain doc.
**ADR required.** No.

### P1-9 — Add prompt-injection and SSRF controls

**Problem.** Untrusted uploaded documents flow into a tool-calling agent; import paths may accept URLs.

**Why it matters.** A construction PDF containing instruction-shaped text reaching an agent that can propose model changes or call tools is a real attack path, and server-side URL fetching from inside a VPC with an instance role is the classic SSRF setup.

**Recommended design.** Delimit untrusted document content explicitly in prompts and instruct the model to treat it as data; never let document text trigger tool invocation directly; flag any ChangeSet derived from document context for mandatory human approval; scope every tool to the caller's org and project. For imports: **upload only in MVP** — no user-supplied URL fetching; if added later, use an allowlist plus an egress-restricted worker.

**Impact.** Small now.
**Documents affected.** Security threat model, AI copilot doc, `CLAUDE.md` §31.
**ADR required.** No.

### P1-10 — Define the project permission matrix

**Problem.** Organization roles and `project_members` both exist; their interaction is undefined, and cost-data visibility is unspecified.

**Why it matters.** Cost and margin visibility is commercially sensitive in AEC — contractors do not show margin to clients. Getting this wrong is a trust failure, and it is far cheaper to define before project CRUD ships at Sprint 06.

**Recommended design.** An explicit matrix over {org owner, org admin, org member, project manager, project editor, project viewer, external client} × {project read, model edit, QS read, cost read, cost edit, documents, billing, invite}. Cost visibility is a distinct permission, not implied by project read.

**Impact.** Documentation now; permission rework later.
**Documents affected.** Auth/RBAC doc, Sprint 06, DB §24.
**ADR required.** No.

## P2 — Valuable optimizations

- **P2-1** Define snapshot cadence concretely (every ~50 versions; always before bulk import/AI commit >100 elements).
- **P2-2** Decide Terraform vs OpenTofu (recommend OpenTofu) and record it.
- **P2-3** Consider Cloudflare Workers/Pages for `apps/web`, removing an ECS service, an ALB path, and NAT egress.
- **P2-4** Avoid NAT Gateway in early production via VPC endpoints or single-AZ NAT.
- **P2-5** Define the AI tool return contract — narrow, paginated payloads rather than whole-model dumps, for both cost and context quality.
- **P2-6** Define partial ChangeSet approval policy (all-or-nothing for MVP; per-item later).
- **P2-7** Enforce filter-before-search in RAG structurally, with a cross-tenant retrieval test.
- **P2-8** Pre-commit worker thresholds (room detection / hit-testing to a worker above 16 ms at 5,000 elements).
- **P2-9** Treat plan-recognition scale detection as an always-confirmed user step.
- **P2-10** Normalise Luna/Terra/Sol/Astra out of worked examples in favour of the logical aliases.

## P3 — Future considerations

Realtime collaboration (Yjs for presence/comments only; geometry stays command-based); read replicas; time-based partitioning of `audit_events`/`model_commands`/`ai_provider_calls`; Rust/WASM for measured geometry hotspots; DWG via ODA on proven demand; multi-region only on a data-residency contract.

---

# 10. Recommended Final Architecture

## 10.1 Runtime architecture

```text
                          ┌─────────────────────────┐
                          │        BROWSER          │
                          │                         │
                          │  React / Next.js shell  │
                          │  ┌───────────────────┐  │
                          │  │ Editor Session    │  │
                          │  │ Store (in-memory  │  │
                          │  │ canonical model   │  │
                          │  │ + baseVersion)    │  │
                          │  └─────┬───────┬─────┘  │
                          │        │       │        │
                          │   2D adapter  3D adapter│
                          │   (PixiJS)    (Three.js)│
                          └────────────┬────────────┘
                                       │ HTTPS / WSS
                                       ▼
                          ┌─────────────────────────┐
                          │   CLOUDFLARE            │
                          │   DNS / WAF / CDN / TLS │
                          └────────────┬────────────┘
                                       ▼
                          ┌─────────────────────────┐
                          │   Next.js (apps/web)    │
                          │   Server Components     │
                          │   shell, auth, chrome   │
                          └────────────┬────────────┘
                                       ▼
      ┌────────────────────────────────────────────────────────────┐
      │              NestJS Modular Monolith (apps/api)            │
      │                                                            │
      │  Controllers (thin)  →  Use Cases  →  Domain  →  Repos     │
      │                                                            │
      │  ┌──────────┐ ┌──────────┐ ┌────────┐ ┌──────┐ ┌────────┐  │
      │  │ projects │ │  model   │ │  qs /  │ │  ai  │ │billing │  │
      │  │  / orgs  │ │ commands │ │ boq /  │ │gateway│ │ledger │  │
      │  │  / auth  │ │ versions │ │  cost  │ │      │ │        │  │
      │  └──────────┘ └──────────┘ └────────┘ └──────┘ └────────┘  │
      └───────┬─────────────────┬──────────────┬──────────┬────────┘
              │                 │              │          │
              ▼                 ▼              ▼          ▼
      ┌──────────────┐  ┌──────────────┐  ┌─────────┐ ┌──────────┐
      │  PostgreSQL  │  │ Cloudflare R2│  │ Temporal│ │ External │
      │              │  │              │  │ (S31+)  │ │ OpenAI   │
      │ ▸ canonical  │  │ ▸ uploads    │  └────┬────┘ │ Stripe   │
      │   elements   │  │ ▸ snapshots  │       │      │ Clerk    │
      │ ▸ versions   │  │ ▸ GLB / IFC  │       ▼      └──────────┘
      │ ▸ commands   │  │ ▸ reports    │  ┌─────────────────────┐
      │ ▸ change     │  └──────────────┘  │      WORKERS        │
      │   items      │                    │ ai-worker   (S31)   │
      │ ▸ quantities │  ┌──────────────┐  │  Python/FastAPI     │
      │ ▸ boq / cost │  │ Redis (S27+) │  │  OpenCV / ONNX      │
      │ ▸ ledger     │  │ rate limits  │  │                     │
      │ ▸ pgvector   │  │ cache        │  │ bim-worker  (S33)   │
      │   (S34+)     │  │ presence     │  │  IfcOpenShell       │
      └──────────────┘  └──────────────┘  └─────────────────────┘

      AUTHORITATIVE              DERIVED / CACHE        EXTERNAL
```

**The invariant this diagram encodes:** every arrow into PostgreSQL's canonical element tables passes through the `model commands` module. 2D, 3D, IFC, QS, BOQ, cost, and AI are all downstream of it. No other path writes geometry.

## 10.2 Package dependency graph

```text
                          ┌───────────────┐
                          │ packages/core │   branded IDs, Money,
                          │               │   Quantity, Result, Zod
                          └───────┬───────┘   primitives. NO framework.
                                  │
                          ┌───────▼───────┐
                          │packages/units │   mm canonical, conversion
                          └───────┬───────┘
                                  │
                     ┌────────────▼────────────┐
                     │ packages/building-model │  elements, geometry,
                     │                         │  commands, validation,
                     │  (framework-free)       │  room topology, serde
                     └──┬────┬────┬────────┬───┘
                        │    │    │        │
        ┌───────────────┘    │    │        └──────────────┐
        │                    │    │                       │
┌───────▼────────┐  ┌────────▼──┐ │              ┌────────▼────────┐
│packages/qs-    │  │packages/  │ │              │packages/api-    │
│engine          │  │cad-2d     │ │              │contracts        │
│(deterministic) │  │(PixiJS)   │ │              │(Zod DTOs)       │
└───────┬────────┘  └───────────┘ │              └────────┬────────┘
        │                    ┌────▼─────────┐             │
┌───────▼────────┐           │packages/     │             │
│packages/cost-  │           │engine-3d     │             │
│engine          │           │(Three.js)    │             │
│(decimal-safe)  │           └──────────────┘             │
└────────────────┘                                        │
                                                          │
        ┌──────────────────┐                              │
        │packages/design-  │  (depends on nothing         │
        │system            │   in the domain)             │
        └──────────────────┘                              │
                                                          │
═══════════════════════ APPS ═════════════════════════════╪═══════
                                                          │
┌──────────────────────────┐        ┌─────────────────────▼──────┐
│ apps/web                 │        │ apps/api                   │
│ ← design-system          │        │ ← api-contracts            │
│ ← api-contracts          │        │ ← building-model           │
│ ← building-model         │        │ ← qs-engine, cost-engine   │
│ ← cad-2d, engine-3d      │        │ ← units, core              │
│ ← units, core            │        │                            │
└──────────────────────────┘        └────────────────────────────┘

┌──────────────────────────┐        ┌────────────────────────────┐
│ apps/ai-worker  (S31)    │        │ apps/bim-worker  (S33)     │
│ Python — no TS packages; │        │ Python — no TS packages;   │
│ contracts via OpenAPI    │        │ contracts via OpenAPI      │
└──────────────────────────┘        └────────────────────────────┘
```

### Dependency rules (enforce in CI)

1. **Dependencies point downward only.** `core` → `units` → `building-model` → {`qs-engine`, `cost-engine`, `cad-2d`, `engine-3d`}. No upward or lateral edges.
2. **`building-model`, `qs-engine`, `cost-engine`, `units`, `core` import no framework** — no React, Next, Pixi, Three, Nest, Drizzle, OpenAI, Stripe, or Clerk. Enforce with a dependency-cruiser or ESLint boundary rule in CI.
3. **`cad-2d` may import Pixi; `engine-3d` may import Three.** Neither imports the other. Neither imports React.
4. **`design-system` never imports domain packages.** It is presentation only.
5. **`api-contracts` depends only on `core`/`units`.** It is the shared transport schema and must stay importable by both `web` and `api`.
6. **Apps may import packages. Packages never import apps.**
7. **Python workers share no TypeScript code.** Their contract is the OpenAPI schema plus JSON payloads.
8. **`cost-engine` may import `qs-engine` types** (it consumes quantities) but not vice versa.

---

# 11. Final Building Model Recommendation

## 11.1 Canonical hierarchy

```text
Project                          organization-owned; currency, unit display, datum
 └── Site                        optional; real-world location (PostGIS later)
     └── Building                one or more per site
         └── Level               elevationMm from project datum — the SINGLE
             │                   authoritative vertical fact (ADR-005).
             │                   floorToFloor is DERIVED from adjacent
             │                   elevations; slab thickness belongs to the
             │                   Slab and its assembly, not to Level.
             └── Element         every physical/spatial object; z is
                                 level-relative
```

> **Corrected.** An earlier draft placed `floor_to_floor_mm` and
> `structural_thickness_mm` on `Level` as stored values. That creates duplicate
> authority for the same physical fact. See ADR-005.

Every element carries, without exception: `id` (UUID, stable across 2D/3D/IFC/QS/BOQ/cost), `organization_id`, `project_id`, `element_type`, `level_id`, optional `host_element_id` and `parent_element_id`, optional `element_type_id` (P1-2), `geometry_schema_version` + `geometry_jsonb`, `properties_schema_version` + `properties_jsonb`, optional `assembly_version_id`, provenance (`source_kind`, `source_ref`, `confidence`, `verification_status`), and version lineage (`created_model_version`, `updated_model_version`, `deleted_model_version`).

Relationships that are not simple host/parent edges live in `element_relationships` (`BOUNDS`, `CONNECTS_TO`, `SUPPORTED_BY`, `SERVES`), version-scoped like elements.

## 11.2 Element structures and invariants

**Wall** — centreline `start`/`end` (mm, level-relative), `thicknessMm`, `heightMm`, optional `baseOffsetMm`, `assemblyId`.
*Invariants:* thickness > 0; height > 0; start ≠ end; centreline is authoritative (faces are derived, never stored); net area = gross − Σ hosted openings; junction resolution is derived at render/QS time, never persisted as separate geometry.

**Door / Window** — `host_element_id` (a wall), `offsetMm` along host centreline, `widthMm`, `heightMm`, `sillHeightMm` (windows), `swing`/`handing` (doors), optional `element_type_id`.
*Invariants:* host must exist and be a wall; the opening footprint must lie entirely within the host's extent and height; deleting a host deletes or orphans hosted elements explicitly (never silently); moving a host moves hosted elements (the cascade of P1-8); **the hosted element is the single representation of its opening** (P1-1).

**Opening** — standalone holes only, where no fixture exists (service penetrations, arches). Same host/position/dimension shape as Door/Window, no fixture properties.
*Invariant:* QS reads hosted elements and standalone openings through **one** deduction interface.

**Room / Space** — derived-then-persisted boundary polygon (mm, level-relative), `BOUNDS` relationships to bounding walls, `name`, `number`, occupancy/finish properties, `user_defined` flag (P0-2).
*Invariants:* boundary is a closed, non-self-intersecting polygon; area is derived from the boundary, never entered directly; if `user_defined` is false the room is recomputed when a bounding wall changes and flagged if area shifts materially; rooms on a level must not overlap.

**Slab** — boundary polygon, `thicknessMm`, `baseOffsetMm` from level, `assemblyId`, optional openings (stair voids).
*Invariants:* polygon closed and non-self-intersecting; thickness > 0; voids lie within the boundary; volume = (area − void area) × thickness.

**Column** — insertion point, `widthMm`/`depthMm` or radius, `heightMm`, `baseOffsetMm`, `assemblyId`.
*Invariants:* positive section; height > 0; may span levels only via an explicit top-level reference, never by implicit extension.

**Beam** — `start`/`end` centreline, section dimensions, `baseOffsetMm` (typically to underside of slab), `assemblyId`.
*Invariants:* start ≠ end; positive section; `SUPPORTED_BY` relationships to columns/walls are explicit, never inferred at QS time.

**Stair** — `baseLevelId`, `topLevelId`, run path, `treadDepthMm`, `riserHeightMm`, `widthMm`, riser count.
*Invariants:* riser count × riser height must equal the level-to-level rise within tolerance (**this is where the P0-3 datum decision pays for itself**); spans exactly two levels; requires a matching slab void.

**Roof** — boundary polygon, pitch/plane definition, `thicknessMm`, `assemblyId`.
*Invariant:* area is measured on the sloping plane, not the plan projection — a classic QS error worth an explicit test.

## 11.3 What is missing from the current model — summary

| Gap | Severity | Section |
|---|---|---|
| Room/space topology undefined | **P0** | P0-2 |
| Vertical datum / level elevation undefined | **P0** | P0-3 |
| Opening represented two ways | **P1** | P1-1 |
| No type/instance identity | **P1** | P1-2 |
| Client-side working model unspecified | **P1** | P1-3 |
| Junction resolution rule unstated | P2 | §11.2 (Wall) |
| Roof slope-vs-plan area rule unstated | P2 | §11.2 (Roof) |

Everything else in the Building Model — IDs, units, provenance, versioning, relationships, serialization, JSONB discipline — is correct as designed.

---

# 12. Final MVP Architecture

**The minimum architecture for the first professional Buildora AI release.**

The honest answer to "what is the minimum?" is narrower than the current plan, because the plan's proof of value arrives at Sprint 26 (QS/cost) while payment arrives at Sprint 38. The architecture below front-loads the differentiator.

## MVP — "Model to Cost" (target: Sprints 01–26, reduced)

The one thing no competitor does simply: **an editable building that produces auditable quantities and cost.**

```text
Next.js web  →  NestJS API  →  PostgreSQL
                    ↓
        canonical Building Model
        (commands + versions + journal)
                    ↓
         2D (Pixi)  +  3D (Three)
                    ↓
      assemblies → QS → BOQ → cost
```

**In:** Clerk auth + orgs; projects; Building Model (walls, doors, windows, rooms, slabs) with commands/versions/undo; 2D editor with snapping and dimensions; 3D view synchronised via adapters; materials/assemblies; QS engine with golden tests; BOQ; concept/elemental cost estimate; PDF/XLSX export; Postgres only; local Docker; Neon; Sentry; GitHub Actions.

**Out:** AI entirely, plan recognition, IFC, RAG, billing, Temporal, Redis, PostGIS, pgvector, collaboration, workers, AWS.

**Proof it works:** create project → ground floor → four walls → room → door → window → 3D → move a wall → 2D and 3D both update → save → reload → identical → assign assemblies → quantities → BOQ → cost → export PDF. This is `AGENTS.md` §83 extended through cost, and it is a sellable product.

## V1 — "AI Copilot + Import" (Sprints 27–36, reduced)

Adds AI Gateway with routing and typed tools; read-only Copilot; AI ChangeSets with preview/approval; upload plan pipeline with confidence review; IFC import/export; documents + RAG; billing Release 1.

**Infrastructure added:** Redis (S27), Temporal (S31), pgvector (S34), AWS production (S35), Stripe (S36).

## POST-V1 (Sprints 37–43)

Reports and sharing; beta hardening; billing Release 2; site/GIS with PostGIS; schedule and cash flow; procurement; actual cost; mobile companion. Rust/WASM if and only if profiling demands it.

## ENTERPRISE (Sprints 44+)

Realtime collaboration; SSO/SCIM; structural and MEP concept modules; sustainability and compliance; marketplace; read replicas; partitioning; regional deployment.

---

# 13. Final Implementation Order

**The documented dependency chain is correct and I confirm it**, with three changes.

```text
Foundation → SaaS → Building Model → 2D → 3D → Sync →
Materials → QS → BOQ → Cost → AI → Recognition → BIM →
Documents → Billing → Reports → Production
```

This ordering is right because each stage is a hard prerequisite of the next: there is no QS without a model, no cost without quantities, no useful AI without something deterministic to propose changes against. Building AI generation first — the tempting inversion — would produce output with nothing to validate it, which is precisely the failure mode `AGENTS.md` §83 exists to prevent.

### Change 1 — Insert Sprint 00: Architecture Lock (P0)

Before Sprint 01, execute P0-1 through P0-6: repair the documentation system, rename to Buildora AI, write the missing Building Model domain document, and accept ADRs 001–009. Estimated 3–5 days. Without it, every subsequent sprint inherits the missing-document problem.

### Change 2 — Move Reports before Billing

Reports are currently Sprint 35, billing Sprint 36. Professional users judge the product by its PDF/XLSX output; reports are also the natural paywall boundary. Ship reports at the end of the cost stage (Sprint 26–27), so the MVP is demonstrable and sellable earlier.

### Change 3 — Split billing across two sprints

Per P1-6: Release 1 (subscription, wallet, ledger, reserve/settle) and Release 2 (top-ups, proration, reconciliation, admin UI).

### On the 48-sprint plan

The plan is well-constructed and its dependency logic is sound, but 38 sprints to revenue is the largest non-technical risk in this project. The mitigation is not to cut architecture — it is to **find the earliest sellable slice**, which the MVP above identifies as Sprint 26 ("model to cost"), and to treat everything after it as funded expansion rather than pre-revenue obligation.

---

# 14. Final "DO NOT DO" List

Permanent architectural guardrails. Every future agent must treat these as binding.

1. **DO NOT** create separate authoritative 2D, 3D, IFC, mesh, QS, BOQ, or cost models. One canonical Building Model.
2. **DO NOT** let an LLM write to `building_elements`, or any table, directly. AI writes ChangeSets; the domain commits.
3. **DO NOT** use floating-point arithmetic for money, rates, or credit balances. `NUMERIC` in the database, decimal-safe in code.
4. **DO NOT** use an LLM to compute geometry, quantities, BOQ totals, or financial arithmetic. Deterministic engines only.
5. **DO NOT** store meshes, vertices, or triangles as canonical geometry. Parametric definitions only; meshes are derived.
6. **DO NOT** query a tenant-owned table without `organization_id` in the predicate.
7. **DO NOT** store credits as a mutable balance column. Ledger plus reservations.
8. **DO NOT** grant credits or entitlements because a browser reached a success URL. Server-side confirmation only.
9. **DO NOT** process a webhook non-idempotently. Inbox, unique event ID, exactly-once effect.
10. **DO NOT** mutate a committed model version. History is append-only.
11. **DO NOT** silently overwrite a newer model version. `baseVersion` check, then `MODEL_VERSION_CONFLICT`.
12. **DO NOT** put the canonical model in a Zustand store, or make any client the authority for persistent state.
13. **DO NOT** patch 3D from 2D, or write to the database from a renderer. All edits flow through domain commands.
14. **DO NOT** treat IFC as the internal editable model. IFC is interchange.
15. **DO NOT** rasterize a good vector source and ask an LLM to rediscover precise geometry.
16. **DO NOT** convert low-confidence recognition into authoritative geometry without human verification.
17. **DO NOT** introduce microservices, Kubernetes, Kafka, sharding, or multi-region without measured need and an ADR.
18. **DO NOT** hold a database transaction open across an OpenAI, Stripe, R2, or any external call.
19. **DO NOT** run migrations automatically at application startup, or modify an applied migration file.
20. **DO NOT** use mutable `latest` image tags as production deployment authority.
21. **DO NOT** embed provider model names in domain or business logic. Logical aliases; provider IDs in configuration.
22. **DO NOT** expose raw SQL or raw database access to an LLM tool.
23. **DO NOT** log tokens, secrets, complete presigned URLs, or private document contents.
24. **DO NOT** make uploaded files public by default, or issue prefix-scoped presigned credentials.
25. **DO NOT** recalculate a historical cost estimate with today's rate book and present it as the same version.
26. **DO NOT** hardcode material quantities, waste factors, or rates into UI components.
27. **DO NOT** create a `packages/utils` or any undefined catch-all package.
28. **DO NOT** add a package, app, or managed service before a concrete consumer exists.
29. **DO NOT** skip, disable, or loosen a test to make CI green.
30. **DO NOT** introduce the name `Buildora AI` into new code or documentation.
31. **DO NOT** let an agent invent architecture when a document is missing. Stop, report the gap, request a decision.

---

# 15. Final Verdict

### A. Is the Buildora AI architecture ready to implement?

**Not yet — but it is close, and the gap is days of work, not weeks.** The architecture itself is sound and, in several areas, notably strong. What is not ready is the documentation system that AI agents depend on to implement it, and four domain decisions (room topology, vertical datum, ID strategy, opening representation) that are cheap now and expensive after Sprint 09.

### B. What MUST change before continuing?

**Nothing — the architecture-lock set is complete.** All eight P0 items are resolved: documentation system repaired (P0-1), room topology (P0-2, ADR-006), vertical datum (P0-3, ADR-005), ID strategy (P0-4, ADR-004), AI pricing finding withdrawn and principle recorded (P0-5, ADR-010), `packages/domain` deleted (P0-6, ADR-011), opening representation consolidated (P0-7), element types added (P0-8, ADR-007). 18 ADRs are accepted and the baseline is frozen in `FINAL_SYSTEM_ARCHITECTURE.md`.

### C. What SHOULD change?

The ten P1 items — most importantly the client-side editor model store (P1-3), opening consolidation (P1-1), element type/instance identity (P1-2), deferring Temporal/Redis/AWS (P1-4), RLS from Sprint 05 (P1-5), and reduced first-release billing scope (P1-6).

### D. What should remain exactly as currently designed?

The canonical Building Model and its single-source-of-truth rule. The command/version/journal/snapshot persistence model with `baseVersion` optimistic concurrency. The entire database architecture — hybrid relational+JSONB, UUID PKs, `NUMERIC` money, direct `organization_id`, deferred partitioning. The QS→BOQ→cost reproducibility lineage. The billing ledger with reservations and the webhook inbox. The AI ChangeSet safety model. The NestJS modular monolith with thin controllers and RFC 9457. The frontend state ownership matrix. Direct Three.js and PixiJS with domain IDs preserved. The hybrid plan-recognition approach. The DevOps topology and the §251 "do not deploy" list. **Do not let any future agent relitigate these.**

### E. What should be removed or deferred?

Removed: `packages/domain` (deleted — ADR-011), the duplicated `docs/architecture` setup pack, the ROI copy file. `apps/render-worker` remains unimplemented. Defer: Temporal (S31), Redis (S27), pgvector (S34), PostGIS (S39), AWS production (S35), `packages/ai-tools` (S27), `apps/ai-worker` (S31), `apps/bim-worker` (S33), `native/geometry-wasm` (profiling-gated). Do not build: Kubernetes, microservices, Go services, Yjs collaboration, GPU workers, ODA, OpenCascade, multi-region, read replicas.

### F. The five largest technical risks

1. **Documentation-system failure causing architecture drift.** ~30 missing canonical documents in an AI-agent-driven build. Agents will invent, inconsistently, and inventions will calcify. *Mitigated by P0-1.*
2. **Room topology and vertical datum discovered late.** The two undefined decisions that touch 2D, 3D, QS, cost, and IFC simultaneously. Discovering them at Sprint 15 means migrating stored geometry and re-deriving every quantity. *Mitigated by P0-2, P0-3.*
3. **Scope exhaustion before revenue.** 38 sprints of unpaid solo work is the most likely cause of project failure, and it is not a technical problem the architecture can solve. *Mitigated by the Sprint-26 sellable MVP in §12.*
4. **Promotional pricing expiry.** `gpt-5.6-sol`'s rate is promotional through at least 21 November 2026. Pro-plan economics must not assume it persists. *Mitigated by ADR-010 — versioned rate cards with `verified_at`, alias-based routing, and margin guardrails.*
5. **2D editor performance at professional model sizes.** The hardest engineering in the product (room detection, snapping, hit-testing at 10k+ elements) has the least specified design. *Mitigated by P1-3, P1-8, and profiling before any Rust decision.*

### G. The five largest architectural strengths

1. **The canonical Building Model as enforced single source of truth** — stated identically across every document, with the forbidden patterns named explicitly. This is the decision that makes everything else possible, and it is correct.
2. **Version/journal/change-items/snapshot persistence** — the pragmatic middle path between mutable state and full event sourcing, giving undo, audit, AI preview, rollback, and revision comparison without the operational cost of event sourcing.
3. **End-to-end determinism and auditability in QS/BOQ/cost** — the six-part reproducibility tuple and element-level quantity traceability meet professional QS standards and exceed several commercial products. This is the actual moat.
4. **The billing architecture's separation of customer value units from provider cost units** — decoupling credits from tokens means provider price changes become a configuration update, not a pricing crisis. Genuinely sophisticated.
5. **Provenance and confidence as first-class fields from day one** — `source_kind`, `source_ref`, `confidence`, `verification_status` on every element. Almost every team adds these after recognition ships and then cannot backfill. Having them before the first wall is written is a real and rare advantage.

### H. Would I approve this architecture for implementation?

## **APPROVE WITH CHANGES**

> **Post-independent-review status (2026-09-12).** A second, adversarial review
> confirmed the architectural direction and the eight-item lock set, but found:
> two live schema contradictions (`levels.height_mm`; `cost_estimates` missing
> lineage) — **both now fixed**; filesystem drift (`packages/domain` present,
> `model-session` absent) — **now fixed**; an overclaim in ADR-013 and a missing
> RLS transaction recipe in ADR-016 — **both now amended**; and three
> **contract gates** (room topology determinism, session reconciliation, QS rule
> contract) recorded in `OPEN_ARCHITECTURE_DECISIONS.md` §1.1.
>
> The conclusion below that "nothing must change" was **too strong** and is
> superseded by that gate list.

If I were the CTO personally responsible for Buildora AI, I would approve this architecture. The original approval was conditional on an architecture-lock set being completed before Sprint 01; **that set (eight items) is now complete**, 18 ADRs are accepted, and the baseline is frozen.

**Why approve:** the hard decisions are right. Single canonical model, deterministic engines, AI that proposes rather than mutates, auditable cost lineage, ledger-based billing, exact money, strong tenancy, modular monolith, managed services over premature infrastructure. These are the decisions that are expensive to reverse, and they are correct. The technology choices are defensible and I recommend replacing none of them. The scalability seams are in the right places — I see no path from 10 to 100,000 users that requires a fundamental rewrite.

**Why with changes:** the documentation system that AI agents depend on is broken in a way that directly attacks the project's delivery model, and four Building Model decisions are undefined at precisely the moment they are cheapest to make. None of this is a redesign. It is 3–5 days of work that prevents months of rework.

**The one thing I would say as CTO beyond the architecture:** the design is strong enough that the primary risk is no longer technical. It is whether one person reaches revenue before exhaustion. The architecture supports a sellable product at Sprint 26; the plan defers payment to Sprint 38. I would close that gap deliberately.

---

# PROPOSED FINAL ARCHITECTURE BASELINE

**The following decisions are recommended to be locked.** Once accepted, they may be changed only by an approved ADR that explicitly supersedes them.

## Locked — Domain

1. The canonical Building Model in PostgreSQL is the single source of truth. 2D, 3D, IFC, meshes, QS, BOQ, and cost are derived representations with no independent authority.
2. Hierarchy: Project → Site → Building → Level → Element.
3. Canonical geometry unit: **millimetres**. All conversion isolated in `packages/units`.
4. Project datum = ground-floor finished floor level = 0. `levels.elevation_mm` is measured from project datum. **Element geometry is level-relative.** *(ADR-005)*
5. Rooms are **derived-then-persisted** from wall topology, with `BOUNDS` relationships and a `user_defined` override flag. *(ADR-006)*
6. Doors and windows are wall-hosted elements; **the hosted element is the sole representation of its opening**. Standalone `opening` exists only for fixture-less holes.
7. Stable UUID element IDs are preserved across model, 2D, 3D, QS, BOQ, cost, AI, and audit.
8. `element_relationships` is the only home for non-host/parent relationships.
9. Provenance (`source_kind`, `source_ref`, `confidence`, `verification_status`) is mandatory on every element.
10. `element_types` provides type/instance identity. *(ADR-007)*

## Locked — Persistence and versioning

11. Current materialized state + append-only `model_commands` journal + `model_change_items` + periodic `model_snapshots`.
12. Optimistic concurrency via `baseVersion`; stale writes return `MODEL_VERSION_CONFLICT`. Never silently merge geometry.
13. Committed model versions are immutable.
14. Snapshot every ~50 committed versions and before any bulk import or AI commit exceeding 100 elements.
15. One model action is one atomic transaction including its cascade.

## Locked — Database

16. One PostgreSQL database, one application schema, for the modular monolith.
17. **UUID primary keys** (UUIDv7 preferred); `public_id` for human-readable references, never a foreign key target. *(ADR-004)*
18. `NUMERIC` for all money, rates, and credit quantities. Floating point is forbidden for authoritative financial values.
19. `TIMESTAMPTZ` for all timestamps; UTC ISO 8601 at API boundaries.
20. Direct `organization_id` on all tenant-owned tables; every repository query scopes by it.
21. JSONB permitted only with a TypeScript type, runtime validation, and a schema version.
22. No meshes, binaries, or large artifacts in relational columns; object storage with database metadata.
23. RLS enabled on tenant-owned tables from Sprint 05 as defence in depth.
24. Migrations are committed artifacts run as an explicit pipeline step; expand → migrate → switch → contract.

## Locked — Determinism

25. Geometry, quantities, BOQ totals, costs, taxes, and credit balances are computed by deterministic engines. LLMs never compute authoritative values.
26. Cost estimates pin model version + quantity run + BOQ version + rate book version + cost engine version + assumption set. Historical estimates are never recomputed with current rates.
27. QS golden tests — including the 5000×2700 wall less 1000×2100 opening = 11.4 m² case — are permanent.
28. MVP default measurement standard: **NRM2**, encoded as a versioned `qs_rule_sets` record. *(ADR-009)*

## Locked — AI

29. AI proposes; the domain validates; the user or policy approves; deterministic code commits. AI never writes domain tables directly.
30. Flow: prompt → gateway → ChangeSet → draft → validation → QS/cost impact → preview → approval → transactional commit, with a `baseVersion` re-check at commit.
31. Logical model aliases (FAST/BALANCED/ADVANCED/EXPERT); provider model IDs are configuration.
32. AI tools are typed and runtime-validated, scoped to the caller's organization and project. No raw SQL or database access.
33. Untrusted document content is delimited as data, never permitted to trigger tools; document-derived ChangeSets require human approval.
34. Provider usage (tokens, cost) is recorded separately from customer-facing credits.

## Locked — Billing

35. Customer-facing units are credits and entitlements, never raw tokens.
36. Wallet + ledger + reservations. No mutable balance column as the source of truth.
37. Reserve before expensive work; settle or release afterwards. Retries and escalation never double-charge.
38. Stripe is authoritative for payment facts; Buildora is authoritative for plans, entitlements, credits, and usage.
39. Credits are granted only on server-side payment confirmation.
40. Webhooks are verified, stored in an inbox, and processed idempotently.

## Locked — Application architecture

41. NestJS modular monolith. Controllers thin; use cases own application flow; domain owns rules; module-owned repositories with explicit tenant scope.
42. REST/OpenAPI with RFC 9457 problem details and stable error codes. Shared schemas in `packages/api-contracts`.
43. Next.js App Router: Server Components by default; the editor is a client island.
44. Frontend state: server/database authoritative; TanStack Query for server state; Zustand for ephemeral UI only.
45. The **editor session store** in the renderer-neutral **`packages/model-session`** holds the in-memory working model with `baseVersion`; 2D and 3D adapters subscribe to it and neither depends on the other. It is a synchronized working copy, never a second authority. *(ADR-008)*
46. PixiJS for 2D, direct Three.js for 3D (WebGL2 for MVP); renderer objects are disposable views carrying domain IDs.
47. Domain packages (`units`, `building-model`, `model-session`, `qs-engine`, `cost-engine`) import no framework. Enforced in CI. **`packages/domain` is deleted and `packages/core` is deliberately not created.** *(ADR-011)*
48. Package dependencies point downward only, per §10.2.

## Locked — Infrastructure

49. Docker; Cloudflare; AWS ECS/Fargate; ECR; managed PostgreSQL (Neon initially); R2; Temporal Cloud when needed; GitHub Actions; Sentry; OpenTelemetry. **No Kubernetes.**
50. Build once, promote the same immutable digest. No `latest` in production.
51. Provisioning gates: Redis at Sprint 27; Temporal at Sprint 31; pgvector at Sprint 34; AWS production at Sprint 35 or first external user; PostGIS at Sprint 39.
52. Not built during MVP: `render-worker`, `native/geometry-wasm`, `packages/ai-tools`, realtime collaboration, GPU workers, ODA, OpenCascade, read replicas, multi-region, microservices.

## Locked — Naming

53. Product: **Buildora AI**. Repository and package scope: **buildora-ai**. The legacy name `BuildWise` does not appear in new code or documentation.

---

*End of review. Unresolved decisions and proposed ADR actions are recorded in `docs/architecture/OPEN_ARCHITECTURE_DECISIONS.md`.*
