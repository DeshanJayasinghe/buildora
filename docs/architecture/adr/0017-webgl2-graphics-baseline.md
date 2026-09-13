# ADR-017 — WebGL2 Graphics Baseline; WebGPU Deferred

> **Status:** Accepted
> **Date:** 2026-09-12
> **Deciders:** Product Owner / CTO
> **Supersedes:** none
> **Superseded by:** none

---

## Context

Two documents disagreed:

- `01_Master_Product_Reference.md` §55 — "Three.js, WebGPU where available,
  WebGL2 fallback".
- `AGENTS.md` §8 — "direct Three.js, WebGL stable path, WebGPU progressive
  enhancement later".

WebGPU-first and WebGL-first are materially different engineering commitments.
WebGPU implies a second render path, different shader authoring, different
capability detection, and a much larger testing matrix — for a solo developer
building a professional editor whose bottleneck is not raw GPU throughput.

## Decision

**WebGL2 is the MVP graphics baseline.** A single render path.

`AGENTS.md` §8 wins; `01_Master_Product_Reference.md` §55 is corrected to match.

- Three.js is used directly (not React Three Fiber — see below).
- One render path: WebGL2.
- WebGPU is **deferred** to post-beta progressive enhancement, and only if
  profiling shows a genuine ceiling that WebGPU relieves.

### Direct Three.js, not React Three Fiber

Confirmed as part of this decision. The editor requires imperative control over:

- incremental adapter diffing driven by model change items,
- explicit geometry and material disposal,
- selection and picking mapped to domain IDs,
- frame-budget management during interaction.

React Three Fiber's reconciler works against each of these. Adopting R3F for the
core editor requires a superseding ADR (`AGENTS.md` §8, §88).

### Performance work is evidence-driven

If 3D performance becomes a constraint, the ordered responses are:

1. instancing and geometry merging,
2. level of detail and frustum/occlusion culling,
3. reducing draw calls in the adapter layer,
4. offloading geometry preparation to workers,

**before** considering a second render path. WebGPU is not a substitute for
adapter efficiency.

## Alternatives considered

| Option | Pros | Cons | Why not chosen |
|---|---|---|---|
| A — WebGPU first, WebGL2 fallback | Future-facing; better compute potential | Two render paths; doubled testing; uneven browser support; solves a bottleneck not yet demonstrated | Cost without evidence of need |
| B — WebGL2 only (chosen) | One path; universal support; Three.js maturity; smallest surface | Leaves some future performance on the table | — |
| C — WebGPU only | Simplest of the two modern paths | Excludes users on unsupported browsers | Unacceptable for a professional tool |

## Consequences

**Positive**

- One render path to build, test and debug.
- Universal browser support for the target professional audience.
- Mature Three.js WebGL2 ecosystem and documentation.
- Engineering effort goes into adapter efficiency, which benefits every user.

**Negative / accepted cost**

- Some future GPU capability is unused.
- A later WebGPU path, if justified, is additional work at that time.

**Neutral**

- Three.js abstracts much of the eventual renderer switch, reducing the future cost.

## Migration / implementation impact

No migration. `01_Master_Product_Reference.md` §55 corrected.

Affects: `docs/architecture/07_3D_Engine_Implementation.md`,
`packages/engine-3d`.

Owned by: Phase 7 / Sprint 18.

## Rollback / exit strategy

Fully reversible — adding a WebGPU path later is additive, not a rewrite.
Deferring costs nothing structurally.

## Verification

- Review: any WebGPU-specific code path in the MVP is out of scope.
- Review: adopting React Three Fiber for the core editor without a superseding
  ADR is a BLOCKER.
- Performance: 3D interaction targets are met on WebGL2 at realistic model sizes
  before any renderer change is considered.

## References

- `AGENTS.md` §8, §36, §88
- `docs/architecture/01_Master_Product_Reference.md` §55
- ADR-001, ADR-008
