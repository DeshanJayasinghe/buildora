# ADR-003 — NestJS Modular Monolith with Separate Language-Specific Workers

> **Status:** Accepted
> **Date:** 2026-09-12
> **Deciders:** Product Owner / CTO
> **Supersedes:** none
> **Superseded by:** none

---

## Context

Buildora AI is built by one developer with AI assistance. The system spans SaaS
CRUD, a versioned domain model, deterministic calculation engines, AI
orchestration, computer vision, BIM parsing, and billing.

The temptation is to give each of these its own service. For a solo developer,
premature service boundaries multiply deployment surfaces, break local
transactions, and turn every cross-cutting change into a distributed release.

## Decision

**`apps/api` is a single NestJS modular monolith**, organized into bounded-context
modules with enforced internal boundaries:

```text
Controller (thin)
  → validation / authorization
  → use case / application service
  → domain
  → module-owned repository
```

Modules own their repositories. There is no global generic database service that
lets any module read any table. Repository signatures express tenant scope
(`findProjectById(organizationId, projectId)`).

**Separate processes exist only where the runtime genuinely differs**, not
because a module exists:

| App | Runtime | Reason | Created at |
|---|---|---|---|
| `apps/web` | Node / Next.js | Browser delivery | Sprint 01 |
| `apps/api` | Node / NestJS | The modular monolith | Sprint 01 |
| `apps/ai-worker` | Python / FastAPI | OpenCV, PyTorch, ONNX are Python-only | Sprint 31 |
| `apps/bim-worker` | Python | IfcOpenShell is Python-only | Sprint 33 |

**Microservice extraction requires a new ADR** and at least one of:

- a module demonstrably needs independent scaling,
- a genuine security or fault-isolation boundary,
- a runtime requirement the monolith cannot satisfy.

Not before 10,000 MAU, and only with measurement.

Modules that must stay internally isolated regardless of deployment shape:
`billing` (financial invariants), model persistence (version integrity),
`ai` (cost-control surface).

## Alternatives considered

| Option | Pros | Cons | Why not chosen |
|---|---|---|---|
| A — Modular monolith + language workers (chosen) | Local transactions; one deploy; clear seams for later extraction | Single scaling unit for the API | — |
| B — Microservices from the start | Independent scaling | Distributed transactions, N deployments, N observability surfaces — for one developer | Cost vastly exceeds benefit |
| C — Single monolith including Python | One deploy | Cannot run the CV/BIM ecosystem in Node | Technically infeasible |
| D — Serverless functions | Scales to zero | Cold starts on an interactive editor; awkward long jobs; harder local dev | Poor fit for CAD interaction and long workflows |

## Consequences

**Positive**

- Model commands commit in one local transaction with their cascade.
- One API deployment, one migration path, one log stream.
- Bounded contexts leave clean extraction seams if scale ever demands them.

**Negative / accepted cost**

- The API scales as a unit.
- Module boundaries are enforced by convention and review rather than by the
  network — requires discipline.

**Neutral**

- Python workers share no TypeScript code; their contract is the OpenAPI schema
  plus JSON payloads.

## Migration / implementation impact

No migration — greenfield. `apps/ai-worker` and `apps/bim-worker` are created
only when their sprint arrives (ADR-012).

Owned by: Phase 2 / Sprint 01.

## Rollback / exit strategy

Reversible at moderate cost. Bounded-context modules with their own repositories
are the standard precondition for extracting a service later. That is the
intended exit path, not a rewrite.

## Verification

- CI: module import boundaries (no cross-module repository imports).
- Review: a new `apps/*` deployable requires justification against the table above.
- Review: any global database service is a BLOCKER.

## References

- `AGENTS.md` §7, §8, §25, §30
- `docs/standards/Buildora_AI_Backend_Engineering_Standards.md`
- ADR-011 (package boundaries), ADR-012 (provisioning gates)
