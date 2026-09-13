# Buildora AI — Open Architecture Decisions

> **Companion to:** `docs/architecture/FINAL_ARCHITECTURE_REVIEW.md`
> **Status:** Updated 2026-09-12, after the architecture freeze
> **Purpose:** What remains genuinely unresolved, and when each item should be decided.
> **Scope rule:** This document records only **open** questions. Decisions that are
> settled are recorded in `docs/architecture/34_ADR_Index.md` and frozen in
> `docs/architecture/FINAL_SYSTEM_ARCHITECTURE.md`. They are not repeated here.

---

# 1. Status

**The architecture-lock set is complete.** All eight P0 items identified in the
review are resolved, and 22 ADRs are accepted.

| Was | Now |
|---|---|
| 9 "MUST DECIDE NOW" items here, 6 "P0" items in the review | **Reconciled into one set of 8 (P0-1 … P0-8)** |
| OD-01 ID strategy | **Resolved — ADR-004** |
| OD-02 Vertical datum | **Resolved — ADR-005** |
| OD-03 Room topology | **Resolved — ADR-006** (corrected to interior wall faces) |
| OD-04 Opening representation | **Resolved** — single hosted representation, `05_Building_Model_Domain.md` §7.3 |
| OD-05 Element type identity | **Resolved — ADR-007** |
| OD-06 Client model store | **Resolved — ADR-008** (corrected to renderer-neutral `packages/model-session`) |
| OD-07 QS measurement standard | **Resolved — ADR-009** (NRM2 initial default) |
| OD-08 Canonical documents / naming | **Resolved** — `docs/README.md` created, duplicates deleted, renamed to Buildora AI |
| OD-09 AI provider pricing | **Withdrawn and resolved — ADR-010.** The "fabricated pricing" finding was incorrect; pricing was re-verified accurate on 2026-09-12. The surviving principle is versioned rate-card configuration behind logical aliases. |

**Sprint 01 may proceed.** However, independent review (2026-09-12) raised three
**contract gates** — §1.1 — that must be closed before their owning sprints ship.
They do not change any architectural decision; they specify semantics those
decisions left open.

---

# 1.1 Contract Gates — Raised by Independent Review (2026-09-12)

An independent adversarial review (`CODEX_INDEPENDENT_REVIEW.md`) confirmed the
architectural **direction** but found that three decisions freeze an *outcome*
while leaving the *semantics that determine that outcome* unspecified.

These are **not** open architecture questions — the decisions stand. They are
**implementation contracts that must be published before their owning work
ships.** Each is a hard gate.

| # | Contract | Must exist before | Why it blocks |
|---|---|---|---|
| **CG-1** | **Room/junction topology determinism.** Numeric tolerance model; junction classification (T, X, acute) and ownership (miter vs butt vs bevel); variable-thickness behaviour; open-boundary and room-separator concept; transient invalid-state policy; region-matching score; **split/merge/delete room-identity policy**. | Room persistence (~S15), and relied upon from S09 wall geometry | Two correct geometry libraries can return different areas *and different identities* from the same walls. ADR-006 fixes the principle (interior faces) but not the behaviour. Wrong here corrupts every finishes quantity and every room ID downstream. |
| **CG-2** | **Session reconciliation determinism.** Accepted-prefix processing; per-command preconditions; dependency/cascade metadata; rebase rules; quarantine of an invalid suffix; conflict UX; **durable local persistence of the pending queue**. Undo must be a *new command against current `baseVersion`* with preview — never a blind inverse payload. | Optimistic persistence and undo (~S11–S13) | ADR-008 says "reload and replay, **or** surface the conflict". That disjunction is the product policy, not a detail. If command 7 of 30 fails, commands 8–30 were computed from a state including 7. Undefined behaviour here silently loses user work. |
| **CG-3** | **QS rule contract + complete cost lineage.** A versioned *hybrid*: validated declarative rule documents over a small, versioned executable primitive set, with hashing of both, defined rounding/evaluation order, and conformance fixtures. Plus the corrected `cost_estimates` lineage columns. | Any NRM2-branded quantity or issued estimate (~S23–S26b) | ADR-009's "rules are data, not code" is already qualified by its own admission that novel primitives need code. Without the hybrid contract, two engine versions can interpret the same ruleset hash differently — which destroys the reproducibility claim that is the product's moat. |

**Scope discipline added:** until the supported NRM2 work sections are explicitly
audited with conformance fixtures, Sprint 23 output is labelled **"Buildora QS —
NRM2 subset"**, never "NRM2-compliant".

**Geometry envelope added (CG-4, lower severity):** the canonical Wall is a
straight centreline with scalar thickness; Roof is a polygon plus a plane. The
model therefore cannot natively express curved walls, thickness varying along a
wall, vaulted ceilings independent of the roof, or multi-plane roofs. **Publish
the supported MVP geometry envelope and reject unsupported input explicitly**
rather than silently segmenting it into fake elements with fake junctions.

---

# 2. Open — Decide During Implementation

Safe to resolve when the relevant sprint arrives. None creates foundational
lock-in, and none blocks Sprint 00 or Sprint 01.

| # | Decision | Sprint | Recommendation |
|---|---|---:|---|
| OD-10 | Room detection algorithm — half-edge planar subdivision vs polygon/face boolean | 15 | Either satisfies ADR-006. Choose on measured performance at ~5,000 elements. **Do not reach for Rust/WASM prematurely.** |
| OD-11 | Snapshot cadence tuning | 11 | ADR-002 sets ~50 versions as a configurable default. Tune against measured replay cost. |
| OD-12 | Web Worker boundary in the 2D editor | 14–17 | Offload room detection / hit-testing only if they exceed ~16 ms at 5,000 elements. |
| OD-13 | Per-element-type cascade sets | 21 | Wall cascade is defined (`05_Building_Model_Domain.md` §9). Define slab, column, beam, stair cascades as those elements ship. |
| OD-14 | Partial ChangeSet approval | 29 | All-or-nothing for MVP; `ai_change_items` already supports per-item later. |
| OD-15 | AI tool return contract — payload shape and pagination | 27 | Narrow, paginated returns. Never whole-model dumps. |
| OD-16 | Embedding profile, model and dimension | 34 | Record as an explicit profile; never mix dimensions in one search profile. Likely ADR-022. |
| OD-17 | Terraform vs OpenTofu | 35 | Recommend OpenTofu (licence certainty, drop-in). Low stakes. Likely ADR-020. |
| OD-18 | `apps/web` hosting — ECS vs Cloudflare Workers/Pages | 35 | Evaluate Workers/Pages: removes an ECS service, an ALB path and NAT egress. Likely ADR-021. |
| OD-19 | NAT Gateway avoidance | 35 | VPC endpoints for S3/ECR/Secrets Manager, or single-AZ NAT. Material pre-revenue cost. |
| OD-20 | Project permission matrix, including cost-data visibility | 06 | **Decide before project CRUD ships.** Cost visibility must be a distinct permission, not implied by project read — contractors do not show margin to clients. |
| OD-21 | Billing release split boundary | 36 | Release 1: subscription, wallet, ledger, reserve/settle. Release 2: top-ups, proration, reconciliation, admin UI. |
| OD-22 | Plan-recognition scale detection UX | 31–32 | Always an explicit user-confirmed step. A scale error silently corrupts every downstream quantity. |
| OD-23 | IFC schema version support (IFC4 / IFC2x3) | 33 | IFC4 primary; IFC2x3 import tolerance if real files demand it. Likely ADR-023. |
| OD-24 | `packages/ai-tools` — separate package or merged | 27 | Merge into `api-contracts` if it remains a thin schema layer (ADR-011). |
| OD-25 | Report rendering approach — server HTML→PDF vs library | 26 | Decide on output fidelity requirements at the reports sprint. |
| OD-26 | Primary AWS region and data residency | 35 | Driven by customer geography and residency requirements. Likely ADR-019. |
| OD-27 | Launch market confirmation | Pre-launch | ADR-009 defaults to NRM2 on the documentation's UK orientation. If the launch market differs, substitute the local standard — architecture unaffected. |

---

# 3. Deferred Until Scale

Deciding these now risks premature complexity. Each requires production evidence.
See ADR-012 for the provisioning discipline.

| # | Decision | Trigger for revisiting |
|---|---|---|
| OD-28 | Rust/WASM for geometry | TypeScript geometry misses a required frame budget, confirmed by profiling |
| OD-29 | Read replicas | Sustained high read CPU on primary with read-dominated workload |
| OD-30 | Table partitioning (`audit_events`, `model_commands`, `ai_provider_calls`) | A single table approaches ~50–100M rows |
| OD-31 | Realtime collaboration and CRDT scope | Paying customers request concurrent editing. Geometry stays command-based regardless; CRDTs only for presence, comments, annotations |
| OD-32 | Neon → RDS/Aurora migration | Neon cost exceeds RDS equivalent, or VPC-local data becomes a compliance requirement |
| OD-33 | Microservice extraction from `apps/api` | A module demonstrably needs independent scaling or fault isolation. Not before 10,000 MAU |
| OD-34 | Kubernetes | Not part of the planned architecture (ADR-012). Would need >15 services or a platform team |
| OD-35 | Go services | Not part of the planned architecture. Requires a future ADR |
| OD-36 | Multi-region | A signed data-residency requirement |
| OD-37 | ODA SDK for DWG | Paying professional customers block on DWG support |
| OD-38 | OpenCascade | A requirement emerges that the parametric model genuinely cannot express |
| OD-39 | GPU render workers | Photoreal rendering enters the roadmap with demonstrated demand |
| OD-40 | Elasticsearch | Postgres FTS + trigram + pgvector demonstrably insufficient — unlikely at this scale |
| OD-41 | Stripe → Metronome / usage-based billing | Postpaid overage becomes a required commercial model |
| OD-42 | Per-tenant databases | An enterprise contract mandates physical isolation |
| OD-43 | Analytics / OLAP database | BI queries begin affecting OLTP performance |

---

# 4. ADRs Expected Later

Not decisions yet — recorded so the questions are visibly deferred rather than
overlooked. Numbers are provisional; assign the next free number when writing.

| Likely ADR | Question | Phase |
|---|---|---|
| ADR-019 | Primary AWS region and data residency | Production readiness |
| ADR-020 | IaC tooling (Terraform vs OpenTofu) | Production readiness |
| ADR-021 | `apps/web` hosting target | Production readiness |
| ADR-022 | Embedding profile, model and dimension | Documents / RAG |
| ADR-023 | IFC schema version support | BIM |
| ADR-024 | Realtime collaboration scope, if adopted | Deferred |
| ADR-025 | Rust/WASM adoption, if profiling justifies it | Evidence-gated |

---

# 5. Summary

| Class | Count | Blocking? |
|---|---:|---|
| Resolved by the architecture freeze | 9 (OD-01 … OD-09) | No — complete |
| Open, decide during implementation | 18 (OD-10 … OD-27) | **No** |
| Deferred until scale | 16 (OD-28 … OD-43) | No |
| ADRs accepted | **18** | — |
| ADRs expected later | 7 | No |

**Nothing open blocks implementation.** The earliest item needing attention is
**OD-20 (project permission matrix) at Sprint 06**, and it is a product decision
rather than an architectural one.

---

*Decisions recorded here are recommendations until an ADR is accepted. Accepted
decisions live in `docs/architecture/34_ADR_Index.md`.*
