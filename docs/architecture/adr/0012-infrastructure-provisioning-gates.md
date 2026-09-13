# ADR-012 — Infrastructure Provisioning Gates

> **Status:** Accepted
> **Date:** 2026-09-12
> **Deciders:** Product Owner / CTO
> **Supersedes:** none
> **Superseded by:** none

---

## Context

The DevOps plan and sprint plan introduced Temporal Cloud, managed Redis, AWS
production infrastructure, PostGIS and pgvector early — in several cases many
months before any consumer existed.

For a solo, pre-revenue developer each prematurely provisioned service costs:

- setup and configuration time,
- local development friction,
- recurring spend at zero revenue,
- ongoing cognitive load,
- and in Temporal's case, replay-compatibility constraints on all future
  workflow code before the first workflow exists.

The architecture itself is well chosen. Only the *timing* was wrong.

## Decision

**A service is provisioned when its first genuine consumer exists — not when its
folder, document, or roadmap entry exists.**

### Activation gates

| Service | Gate | Expected sprint |
|---|---|---|
| PostgreSQL (local Docker) | Immediately | 02 |
| Neon (managed PostgreSQL) | Immediately — free tier | 02 |
| Sentry | Immediately — free tier | 02 |
| GitHub Actions CI | Immediately | 02 |
| Cloudflare R2 | First file upload feature | ~31 |
| **Redis** | AI/runtime rate limiting and related real need | ~27 |
| **Temporal** | First genuinely durable multi-step workflow — expected at plan recognition/import | ~31 |
| **pgvector** | RAG / document phase | ~34 |
| **PostGIS** | Site / GIS phase | ~39 |
| **AWS production infrastructure** | First external beta / production-readiness phase | ~35 |
| **Rust/WASM** | Only after profiling proves TypeScript geometry misses a required performance target | Evidence-gated |

### Not part of the planned architecture

```text
Kubernetes            — not planned; ECS/Fargate covers the modelled scale
Go services           — not planned; requires a future ADR
Kafka                 — not planned
service mesh          — not planned
multi-region          — not planned
database sharding     — not planned
per-tenant databases  — not planned
```

### Deferred, gated on demonstrated demand

```text
Realtime collaboration / Yjs  — deferred
ODA SDK (DWG)                 — only on demonstrated paid DWG demand
OpenCascade                   — only if parametric geometry proves insufficient
                                for a real professional requirement
GPU render workers            — follows photoreal rendering, post-MVP
Read replicas                 — metric-gated
Table partitioning            — ~50–100M rows in a single table
```

### Before the gate

Sprints 01–26 run on **local Docker Compose (PostgreSQL) plus Neon**. Any
asynchronous need before Temporal is met by a `processing_jobs` table and a
simple in-process worker.

**Interfaces stay stable across the gate.** Job creation and status polling are
designed so that adopting Temporal later is an implementation change behind an
existing interface, not an architecture change.

### Placeholders

Directories for deferred apps and packages may exist with a `README.md` stating
the gate, but must carry **no build, CI, or runtime configuration**. An empty
directory costs nothing; a configured-but-unused deployable costs on every CI
run and every deploy.

Not implemented yet: `apps/render-worker`, `native/geometry-wasm`,
`packages/ai-tools`.
Created when their gate arrives: `apps/ai-worker` (~31), `apps/bim-worker` (~33).

## Alternatives considered

| Option | Pros | Cons | Why not chosen |
|---|---|---|---|
| A — Provision everything up front | Never blocked later; architecture "complete" | Months of spend and friction before first use; Temporal constrains code before any workflow exists | Pure cost for a pre-revenue solo developer |
| B — Provision at first consumer (chosen) | Minimum burn and cognitive load; each service justified when adopted | Brief setup work at the gate | — |
| C — Never provision; simplest possible stack | Cheapest | Some capabilities genuinely require these services | Would cap the product |

## Consequences

**Positive**

- Development-stage spend is close to zero.
- Local development stays simple for the longest possible time.
- Each service is adopted with a concrete use case in hand, so it is configured
  correctly rather than speculatively.

**Negative / accepted cost**

- A brief setup task lands inside the sprint that needs the service.
- The pre-Temporal job mechanism is throwaway work — accepted, as it is small
  and keeps the interface honest.

**Neutral**

- Gates are approximate sprint numbers, not contractual dates. The trigger is the
  consumer, not the number.

## Migration / implementation impact

No migration. Updates the DevOps plan and sprint plan to move provisioning tasks
to their gate sprints.

Affects: `docs/architecture/27_DevOps_Environments_and_Deployment.md`,
`docs/planning/Buildora_AI_Sprintwise_Project_Plan.md`,
`docs/planning/Buildora_AI_Phase_Wise_Development_Checklist.md`,
`docs/finance/Buildora_AI_Development_Production_Costing_Plan.md`.

## Rollback / exit strategy

Fully reversible — any gate can be brought forward if a real need appears sooner.
The rule is "first consumer", not "fixed date".

## Verification

- Review: adding a managed service without naming its consumer is a HIGH finding.
- Review: introducing Kubernetes, Kafka, Go, sharding or multi-region without a
  superseding ADR is a BLOCKER.
- Review: introducing Rust/WASM without profiling evidence is a BLOCKER.
- Cost check: monthly infrastructure spend during Sprints 01–26 should remain
  near zero.

## References

- `AGENTS.md` §8, §56, §77
- `CLAUDE.md` §33
- `docs/architecture/27_DevOps_Environments_and_Deployment.md` §250–§251
- ADR-003, ADR-011
