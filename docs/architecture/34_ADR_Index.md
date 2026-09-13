# Buildora AI — Architecture Decision Record Index

> **Status:** Current
> **Last updated:** 2026-09-12
> **Location of ADRs:** `docs/architecture/adr/`
> **Template:** `docs/architecture/adr/0000-template.md`

---

# 1. How ADRs Work Here

An ADR records a **durable architectural decision** — one that is expensive to
reverse, resolves a contradiction, or establishes an invariant that future
implementation must respect.

ADRs do **not** record ordinary implementation detail. If a choice can be changed
freely in a normal PR, it is not an ADR.

## Precedence

From `AGENTS.md` §4, in order:

1. Current explicit user instruction
2. Root `AGENTS.md`
3. **An approved ADR that explicitly supersedes an older decision**
4. Canonical architecture documents
5. Engineering standards
6. Phase/sprint planning documents
7. Examples
8. Existing implementation

**An ADR overrides older architecture prose only when it explicitly supersedes
it.** An ADR that merely restates an existing decision does not create a new
precedence claim.

## Changing a frozen decision

Every ADR below with status **Accepted** is part of the architecture freeze
(`docs/architecture/FINAL_SYSTEM_ARCHITECTURE.md`).

An implementation agent may **not** change any of these without a new, approved
ADR that:

- names the ADR it supersedes,
- states what changed and why,
- states the migration cost,
- is approved by the Product Owner / CTO.

Silently encoding a different decision in code is a release-blocking defect.

---

# 2. Accepted ADRs

ADR-001 … ADR-021 were accepted on **2026-09-12**; ADR-022 on **2026-09-13**. ADR-001 … ADR-018 formed the
original architecture freeze. ADR-019 and ADR-020 were added the same day for the
3D walkthrough, interior and FF&E capability; ADR-021 for the two-lifecycle
commercial model — all so those capabilities need no later redesign.

| ADR | Title | Domain | Reversibility |
|---|---|---|---|
| [ADR-001](adr/0001-canonical-building-model.md) | Canonical Building Model as single source of truth | Domain | Irreversible |
| [ADR-002](adr/0002-model-persistence-and-versioning.md) | Model persistence: current state + commands + change items + snapshots | Persistence | Irreversible |
| [ADR-003](adr/0003-modular-monolith.md) | NestJS modular monolith with separate language workers | Backend | Moderate |
| [ADR-004](adr/0004-id-strategy.md) | UUID primary keys with separate public identifiers | Database | Irreversible after first migration |
| [ADR-005](adr/0005-datum-and-level-geometry.md) | Building-local datum and level-relative element geometry | Domain | Irreversible after geometry stored |
| [ADR-006](adr/0006-room-space-topology.md) | Room/Space topology: interior-face boundary, persisted identity | Domain | Irreversible after quantities issued |
| [ADR-007](adr/0007-element-type-instance.md) | Element type / instance identity | Domain | Moderate |
| [ADR-008](adr/0008-model-session.md) | Renderer-neutral model-session architecture | Frontend | Moderate |
| [ADR-009](adr/0009-qs-measurement-rulesets.md) | Versioned QS measurement rulesets; NRM2 initial default | QS | Default reversible pre-launch |
| [ADR-010](adr/0010-provider-models-and-pricing-as-configuration.md) | Provider models and pricing as versioned configuration | AI / Billing | Reversible |
| [ADR-011](adr/0011-package-boundaries.md) | Monorepo package boundaries and dependency direction | Monorepo | Reversible |
| [ADR-012](adr/0012-infrastructure-provisioning-gates.md) | Infrastructure provisioning gates | DevOps | Reversible |
| [ADR-013](adr/0013-ai-changeset-safety.md) | AI ChangeSet safety architecture | AI | Irreversible (safety direction) |
| [ADR-014](adr/0014-billing-ledger-and-reservations.md) | Billing ledger and credit reservation architecture | Billing | Irreversible |
| [ADR-015](adr/0015-qs-boq-cost-reproducibility.md) | QS / BOQ / cost reproducibility lineage | QS / Cost | Irreversible |
| [ADR-016](adr/0016-tenant-isolation-and-rls.md) | Tenant isolation and the RLS production gate | Security | Irreversible (safety direction) |
| [ADR-017](adr/0017-webgl2-graphics-baseline.md) | WebGL2 graphics baseline; WebGPU deferred | 3D | Reversible |
| [ADR-018](adr/0018-ifc-as-interchange.md) | IFC as interchange, not canonical state | BIM | Permanent direction |
| [ADR-019](adr/0019-ffe-as-domain-elements.md) | FF&E as first-class domain elements | Domain | Cheap to remove, expensive to add |
| [ADR-020](adr/0020-finish-assignment-separate-from-assembly.md) | Finish assignment separate from construction assembly | Domain / QS | Irreversible once finishes are priced |
| [ADR-021](adr/0021-two-lifecycle-commercial-model.md) | Two customer lifecycles on one platform | Commercial / Tenancy | Universal org irreversible; term model reversible |
| [ADR-022](adr/0022-runtime-feature-gating.md) | Runtime feature gating and admin-managed catalogue | Commercial / Security | Reversible |

**Count: 22 accepted. 0 proposed. 0 superseded. 0 deprecated.**

---

# 3. What Each ADR Settles

Grouped by the question it answers, for agents looking for the right document.

## Domain model

- **What is authoritative?** → ADR-001
- **How is history kept, and how do concurrent edits behave?** → ADR-002
- **Where is `z` measured from, and who owns level elevation?** → ADR-005
- **How is a room's boundary derived, and what is its area?** → ADR-006
- **How do repeated element types work?** → ADR-007
- **Where does furniture live?** → ADR-019 (domain elements, not scene state)
- **How is a paint colour modelled and costed?** → ADR-020 (surface-scoped finish)
- **How are door/window openings represented?** → ADR-001 and
  `05_Building_Model_Domain.md` (single hosted representation)

## Data

- **What type are primary keys?** → ADR-004
- **How is tenant data isolated?** → ADR-016

## Application

- **Monolith or services?** → ADR-003
- **Where does the client hold the working model?** → ADR-008
- **Which package may import which?** → ADR-011

## Calculation

- **Which measurement standard governs quantities?** → ADR-009
- **How is a cost estimate reproduced months later?** → ADR-015

## AI

- **Can AI write to the model?** → ADR-013 (no)
- **Where do provider model IDs and prices live?** → ADR-010

## Commercial

- **How are credits tracked and charged?** → ADR-014
- **Can a plan be fixed-term rather than recurring?** → ADR-021 (yes)
- **What happens to a project when a paid term ends?** → ADR-021 (archived, preserved)
- **Does a solo user need an organization?** → ADR-021 (yes — always)
- **How does an admin change what a plan includes, without a deploy?** → ADR-022
- **Are feature flags the same as entitlements?** → ADR-022 (no — separate systems)

## Platform

- **When is each service provisioned?** → ADR-012
- **WebGL or WebGPU?** → ADR-017
- **Is IFC the internal model?** → ADR-018 (no)

---

# 4. ADRs Expected Later

These are **not** decisions yet. They are recorded so that agents know the
question exists and is deliberately deferred, not overlooked. Each will be
written when its phase begins.

| Likely ADR | Question | Expected phase |
|---|---|---|
| ADR-023 | Primary AWS region and data residency | Production readiness |
| ADR-024 | IaC tooling (Terraform vs OpenTofu) | Production readiness |
| ADR-025 | `apps/web` hosting target (ECS vs Cloudflare Workers/Pages) | Production readiness |
| ADR-026 | Embedding profile, model and dimension | Documents / RAG |
| ADR-027 | IFC schema version support (IFC4 / IFC2x3) | BIM |
| ADR-028 | Realtime collaboration scope, if adopted | Deferred |
| ADR-029 | Rust/WASM adoption, if profiling justifies it | Evidence-gated |
| ADR-030 | Per-element surface taxonomy for finishes | Before the finish editor (ADR-020) |

> **Renumbered 2026-09-13.** ADR-019–022 were taken by the FF&E, finish,
> commercial-lifecycle and feature-gating decisions. Future ADRs start at 023.

Numbers above are provisional. Assign the next free number at the time of writing.

---

# 5. Writing a New ADR

1. Copy `adr/0000-template.md` to `adr/NNNN-short-kebab-title.md`.
2. Use the next free number. Never reuse a number.
3. If it supersedes an existing ADR, say so in both files — set the old ADR's
   status to `Superseded by ADR-NNNN`.
4. Fill in alternatives honestly. An ADR with no alternatives considered is
   usually not a real decision.
5. State the negative consequences. An ADR listing only benefits is not
   trustworthy.
6. Add a row to §2 of this index.
7. Update `FINAL_SYSTEM_ARCHITECTURE.md` if the decision changes a frozen
   invariant.

---

# 6. References

- `docs/README.md` — documentation router
- `docs/architecture/FINAL_SYSTEM_ARCHITECTURE.md` — frozen baseline
- `docs/architecture/FINAL_ARCHITECTURE_REVIEW.md` — the review that produced these ADRs
- `docs/architecture/OPEN_ARCHITECTURE_DECISIONS.md` — what remains open
- `AGENTS.md` §4 (precedence), §76 (when an ADR is required)
- `CLAUDE.md` §47 (contradiction handling), §48 (ADR format), §49 (examples)
