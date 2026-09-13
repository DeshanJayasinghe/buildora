# ADR-008 — Renderer-Neutral Model-Session Architecture

> **Status:** Accepted
> **Date:** 2026-09-12
> **Deciders:** Product Owner / CTO
> **Supersedes:** none
> **Superseded by:** none

---

## Context

No document defined how the editor loads, holds and mutates the working Building
Model client-side. This is the central frontend architecture decision for a CAD
application.

The state ownership matrix correctly assigns authoritative walls to
backend/domain and forbids a giant Zustand store — but that left the working-model
question unanswered, and both likely agent-invented answers are wrong:

- TanStack Query per interaction — far too slow for 60 fps CAD interaction.
- The whole model in Zustand — creates a second authority, violating `AGENTS.md` §34.

A first draft placed this store inside `packages/cad-2d`. That was corrected
during review: it would make the 3D engine conceptually depend on the 2D package.

## Decision

Introduce a **renderer-neutral package: `packages/model-session`**.

### Dependency position

```text
packages/units
       ↓
packages/building-model
       ↓
packages/model-session
       ↓
 ┌─────┴─────┐
 ↓           ↓
cad-2d    engine-3d
```

Both renderers depend on `model-session`. Neither renderer depends on the other.

### Responsibilities

`packages/model-session` owns:

- the in-memory working Building Model for the active project/level,
- `baseVersion` tracking,
- optimistic local command application,
- the pending command queue,
- server acknowledgement and reconciliation,
- `MODEL_VERSION_CONFLICT` handling,
- subscriptions and change notification,
- command result and cascade propagation to subscribers.

### Forbidden imports

`packages/model-session` **must not import**:

```text
React      Next.js    PixiJS     Three.js
NestJS     Drizzle    OpenAI
```

It is plain TypeScript over `packages/building-model`, testable headlessly.
Enforced in CI (ADR-011).

### It is not a second database

**PostgreSQL and the server remain the authoritative persisted state** (ADR-001).
The model-session is a **synchronized working copy** of the canonical model —
nothing more. It holds no state that cannot be discarded and reloaded from the
server.

### State ownership, restated

| State | Owner |
|---|---|
| Persisted Building Model | PostgreSQL (authoritative) |
| Working model + `baseVersion` + pending commands | `packages/model-session` |
| Project metadata, lists, QS/BOQ/cost read models | TanStack Query |
| Active tool, selection, panel visibility, camera/UI preferences, temporary interaction state | Zustand (ephemeral UI only) |

**TanStack Query never owns working geometry.**
**Zustand never owns the Building Model.**

### Interaction flow

```text
user gesture
  → draft interaction (renderer-local preview, no command)
  → gesture completes
  → semantic command applied optimistically to model-session
  → subscribers (2D + 3D adapters) update from ONE source
  → command queued to API with baseVersion
  → server validates, commits, returns new version + change items
  → model-session reconciles
      ├─ ack      → confirm optimistic state, advance baseVersion
      └─ conflict → MODEL_VERSION_CONFLICT → reload and replay,
                    or surface the conflict to the user
```

Because both adapters subscribe to the same session, `AGENTS.md` §37 is satisfied
structurally: one domain change updates both representations from one source.
2D never patches 3D; 3D never writes the database.

## Alternatives considered

| Option | Pros | Cons | Why not chosen |
|---|---|---|---|
| A — TanStack Query only | One state tool | Unusable at CAD interaction rates; refetch semantics wrong for a working document | Fails the primary requirement |
| B — Whole model in Zustand | Familiar | Second authority; violates the state matrix; no principled reconciliation | Architecturally wrong |
| C — Session store inside `cad-2d` | Fewer packages | 3D would depend on the 2D package; wrong dependency direction | Corrected during review |
| D — Renderer-neutral `model-session` (chosen) | Correct dependency direction; one update source for both renderers; headlessly testable | One more package | — |

## Consequences

**Positive**

- 2D and 3D consume identical state from one place — divergence is structurally
  prevented rather than tested for.
- Optimistic interaction is possible without surrendering server authority.
- Conflict handling has one owner instead of being scattered across the editor.
- The session is testable without a browser, a canvas or a GPU.

**Negative / accepted cost**

- Reconciliation logic (optimistic apply, ack, rollback, replay) is genuinely
  subtle and must be tested carefully.
- One additional package boundary to maintain.

**Neutral**

- The session is per project/level scope; loading strategy for very large models
  is an implementation concern, not an architecture change.

## Migration / implementation impact

No migration. `packages/model-session` is created in the MVP package set
(ADR-011) and built alongside the 2D editor.

Affects: `docs/standards/Buildora_AI_Frontend_Engineering_Standards.md` §34/§43,
`docs/architecture/06_2D_CAD_Editor_Implementation.md`,
`07_3D_Engine_Implementation.md`, `04_Monorepo_and_Module_Architecture.md`.

Owned by: Phase 6 / Sprint 12–13, before wall drawing.

## Rollback / exit strategy

Reversible at moderate cost while the editor is small. Discovering the need for
it at Sprint 13+ would mean a frontend rewrite — which is why it is decided now.

## Verification

- CI: `model-session` imports none of React, Next, Pixi, Three, Nest, Drizzle, OpenAI.
- CI: `engine-3d` does not import `cad-2d`, and vice versa.
- Test: optimistic apply → server ack → state converges.
- Test: optimistic apply → `MODEL_VERSION_CONFLICT` → reload/replay leaves no
  orphaned local state.
- Test: one command notifies both 2D and 3D subscribers with the same change set.
- Review: any persistent Building Model state in Zustand is a BLOCKER.

## References

- `AGENTS.md` §34, §35, §36, §37
- `docs/standards/Buildora_AI_Frontend_Engineering_Standards.md` §10, §11, §34
- OD-06 in `docs/architecture/OPEN_ARCHITECTURE_DECISIONS.md`
- ADR-001, ADR-002, ADR-011
