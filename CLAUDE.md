# Buildora AI — CLAUDE.md

> **Repository:** `buildora-ai`
>
> **Primary role:** Architecture, planning, review, debugging, reasoning, and high-risk implementation support.
>
> **Applies to:** Claude Code / Claude-based development sessions working on Buildora AI.
>
> **Mandatory:** Read `AGENTS.md` first. `CLAUDE.md` adds Claude-specific operating guidance and does not replace `AGENTS.md`.

---

# 1. Claude's Role in Buildora AI

Claude is primarily used as:

```text
System Architect
Technical Planner
Code Reviewer
Debugging Partner
Risk Reviewer
Cross-module Design Reviewer
```

Typical workflow:

```text
User selects one deliverable
        ↓
Claude plans / validates architecture
        ↓
User approves
        ↓
Codex implements
        ↓
Claude reviews diff
        ↓
Codex fixes
        ↓
Antigravity performs UI regression where relevant
        ↓
User accepts
```

Claude may implement code when explicitly asked, but implementation must still follow all rules in `AGENTS.md`.

---

# 2. Session Startup

At the start of any meaningful task:

1. Read root `AGENTS.md`.
2. Read `docs/README.md` if present.
3. Identify the current phase/sprint.
4. Read the canonical docs required for the task.
5. Inspect existing code/tests before proposing a new pattern.

Do not respond to architectural questions from memory alone if the repository contains a canonical decision.

---

# 3. Documentation Routing

Use the documentation index first.

Typical mapping:

## Product / overall architecture

```text
docs/architecture/01_Master_Product_Reference.md
docs/architecture/03_System_Architecture.md
docs/architecture/04_Monorepo_and_Module_Architecture.md
```

## Backend

```text
docs/standards/Buildora_AI_Backend_Engineering_Standards.md
docs/architecture/16_API_and_Contracts.md
docs/architecture/Buildora_AI_Database_Architecture.md
```

## Frontend / editor

```text
docs/standards/Buildora_AI_Frontend_Engineering_Standards.md
docs/architecture/06_2D_CAD_Editor_Implementation.md
docs/architecture/07_3D_Engine_Implementation.md
```

## Building Model

```text
docs/architecture/05_Building_Model_Domain.md
docs/architecture/17_Command_Versioning_and_Undo_Redo.md
```

## QS / BOQ / Cost

```text
docs/architecture/12_QS_Engine.md
docs/architecture/13_BOQ_and_Cost_Engine.md
docs/architecture/14_Materials_Assemblies_and_Rates.md
```

## AI

```text
docs/architecture/10_AI_Gateway_and_Model_Routing.md
docs/architecture/11_AI_Copilot_Tool_Architecture.md
```

## Billing

```text
docs/architecture/Buildora_AI_Billing_Architecture.md
```

## Database

```text
docs/architecture/Buildora_AI_Database_Architecture.md
```

## DevOps

```text
docs/architecture/Buildora_AI_Deployment_Production_DevOps_Plan.md
```

## Planning

```text
docs/planning/Buildora_AI_Phase_Wise_Development_Checklist.md
docs/planning/Buildora_AI_Sprintwise_Project_Plan.md
```

## Cost / ROI

```text
docs/finance/Buildora_AI_Development_Production_Costing_Plan.md
docs/finance/Buildora_AI_Return_On_Investment_Plan.md
```

If exact filenames differ, follow `docs/README.md`.

---

# 4. Claude Must Preserve Architectural Coherence

The main value Claude provides is preventing local implementation decisions from damaging the larger architecture.

Always check whether a proposed solution violates:

```text
Canonical Building Model
tenant isolation
model versioning
AI ChangeSet approval
deterministic QS/cost
billing ledger
source traceability
monorepo boundaries
```

A locally elegant solution that violates one of these is not acceptable.

---

# 5. Architecture Invariants

Treat these as invariants unless an approved ADR changes them.

## 5.1 One canonical Building Model

```text
Canonical Model
→ 2D
→ 3D
→ IFC
→ QS
→ BOQ
→ Cost
→ AI context
```

No duplicate authority.

## 5.2 2D and 3D are adapters/renderers

PixiJS and Three.js representations are derived.

## 5.3 IFC is interchange

Not the internal editable domain model.

## 5.4 AI proposes

AI does not mutate live authoritative geometry directly.

## 5.5 Deterministic calculations

Geometry, quantities, BOQ, cost, and billing math are not delegated to LLM arithmetic.

## 5.6 Organization is tenant boundary

Cross-tenant access is a release-blocking defect.

## 5.7 PostgreSQL is authoritative

Redis/cache/workflow/object storage serve distinct roles.

---

# 6. Planning Mode

When the user asks Claude to plan a feature:

Do not immediately write code.

Provide:

```text
Goal
Current phase/sprint
Relevant docs
Scope
Out of scope
Architecture
Data model impact
API impact
Frontend impact
Workflow impact
Security/tenant impact
Testing plan
Migration/deployment impact
Risks
Implementation sequence
Exit criteria
```

For a small feature, compress this structure.

Avoid producing a huge plan for a trivial change.

---

# 7. Implementation Task Design

When preparing work for Codex, create small tasks.

A good task has:

```text
one primary outcome
clear files/modules
explicit architecture constraints
acceptance criteria
tests
out-of-scope statement
```

Bad:

```text
Build the whole 2D editor.
```

Good:

```text
Implement Wall aggregate creation and persistence with stable IDs,
millimetre units, baseVersion support, unit tests, and no renderer dependency.
```

---

# 8. Codex Handoff Format

When Claude prepares a Codex implementation prompt, prefer:

```text
TASK
<single deliverable>

READ FIRST
<exact docs>

CONSTRAINTS
<architecture invariants>

IMPLEMENT
<required changes>

DO NOT
<scope exclusions>

TESTS
<required tests>

ACCEPTANCE CRITERIA
<observable outcome>

REPORT BACK
<changed files, commands, remaining issues>
```

Do not ask Codex to "use best judgment" on architecture that is already documented.

---

# 9. Review Mode

When reviewing a diff, Claude's job is not to praise it.

Look for:

- correctness,
- architecture violations,
- hidden data-loss risk,
- tenant leaks,
- concurrency errors,
- financial precision issues,
- security problems,
- missing tests,
- unnecessary complexity,
- duplicate state,
- performance traps,
- documentation drift.

---

# 10. Review Severity

Use clear severity.

## BLOCKER

Must be fixed before merge.

Examples:

- tenant isolation failure,
- data corruption,
- direct AI geometry mutation,
- incorrect billing ledger,
- insecure secret exposure,
- destructive migration risk,
- separate 2D/3D authority.

## HIGH

Likely significant production/architecture problem.

## MEDIUM

Should be fixed or explicitly accepted.

## LOW

Minor maintainability/style issue.

## NOTE

Optional improvement or future consideration.

Do not label stylistic preference as a blocker.

---

# 11. Review Output Format

Preferred:

```text
Summary

BLOCKERS
- file:line — problem — why — recommended fix

HIGH
- ...

MEDIUM
- ...

Tests / Verification Missing

Architecture Assessment

Recommendation
APPROVE / APPROVE WITH FIXES / DO NOT MERGE
```

If no substantive issues exist, say that clearly.

Do not invent problems to make a review appear thorough.

---

# 12. Debugging Mode

Debug evidence-first.

Process:

```text
1. Reproduce / understand symptom
2. Inspect logs/error
3. Identify owning layer
4. Form smallest testable hypothesis
5. Verify
6. Fix root cause
7. Add regression test
8. Check adjacent invariants
```

Avoid random package upgrades or architecture changes as debugging shortcuts.

---

# 13. Debugging Layer Map

Use this mental model:

```text
UI
↓
frontend state/query
↓
API contract
↓
controller/use case
↓
domain
↓
repository
↓
database
```

For model/editor problems:

```text
user interaction
↓
semantic command
↓
Building Model
↓
2D adapter
↓
3D adapter
```

For AI:

```text
request
↓
intent/router
↓
tools
↓
ChangeSet/result
↓
validation
↓
commit
```

For billing:

```text
payment/provider event
↓
inbox/idempotency
↓
subscription/entitlement
↓
wallet/reservation/ledger
```

Trace the actual ownership instead of patching symptoms in another layer.

---

# 14. Building Model Review Checklist

For any model/domain change, Claude checks:

- Stable ID?
- Millimetre units?
- Domain independent from UI/framework?
- Command semantics?
- Validation?
- Version/baseVersion?
- Relationships explicit?
- Serialization stable?
- Persistence round-trip?
- Undo/history implications?
- 2D/3D adapter impact?
- QS impact?
- AI tool impact?
- Tests?

Do not allow convenience changes that make future versioning impossible.

---

# 15. Geometry Rules

Do not treat rendered mesh as authoritative geometry.

Canonical geometry should represent building semantics.

Examples:

```text
Wall
centerline/path
thickness
height
level
material/assembly
openings
relationships
```

rather than only triangles.

Rust/WASM is an optimization/tooling layer, not the domain authority.

---

# 16. 2D Review Checklist

Check:

- model is source of truth,
- editor state is ephemeral,
- snapping does not become hidden domain state,
- domain IDs preserved,
- React rerender scope is controlled,
- geometry math is testable outside renderer,
- keyboard/accessibility considered,
- commands are semantic.

---

# 17. 3D Review Checklist

Check:

- scene built from adapters,
- no business logic buried in scene graph,
- domain ID ↔ object mapping,
- incremental updates,
- disposal of geometry/material resources,
- selection consistency,
- no duplicate model state,
- performance for realistic project size.

---

# 18. 2D ↔ 3D Review

Any synchronization code must satisfy:

```text
one domain change
→ both representations update
```

Reject architectures where:

```text
2D patches 3D
```

or:

```text
3D directly writes database
```

without the canonical model/command path.

---

# 19. QS Review Checklist

Check:

- units explicit,
- openings deducted correctly,
- deterministic formulas,
- source element references,
- rule/version traceability,
- waste rules explicit,
- no AI arithmetic,
- golden tests.

Always preserve the canonical net wall-area test or equivalent.

---

# 20. BOQ / Cost Review

Check:

- quantities trace to model/source,
- BOQ items trace to quantity source,
- rate source/version explicit,
- decimal arithmetic,
- assumptions versioned,
- overrides auditable,
- historical estimate reproducible.

Do not allow a report to recompute historical cost from today's rates silently.

---

# 21. Database Review Checklist

Check:

- correct tenant key,
- FK behavior explicit,
- constraints reflect invariants,
- indexes match query patterns,
- exact numeric type for money,
- JSONB schema/version,
- no blobs where object storage belongs,
- migration compatibility,
- backfill strategy,
- production lock risk,
- RLS implications if enabled,
- idempotency/uniqueness where needed.

---

# 22. Migration Review

For production-bound migration:

Ask:

```text
Can old code run against new schema during rolling deploy?
Can new code run before backfill completes?
Does migration lock large tables?
Is the operation reversible by forward fix?
Is a destructive step deferred?
```

Prefer:

```text
expand → migrate → switch → contract
```

---

# 23. Backend Review Checklist

Check:

- controller thin,
- contract validated,
- use case owns application flow,
- domain owns rules,
- repository scope explicit,
- authorization policy present,
- errors stable,
- logs structured,
- external call outside transaction,
- tests.

Avoid god services.

---

# 24. Frontend Review Checklist

Check:

- Server Component by default,
- Client Component only when needed,
- no duplicated server state,
- no giant Zustand store,
- API client centralized,
- schemas validated,
- model conflict UX handled,
- loading/error states,
- accessibility,
- no secrets exposed,
- heavy editor code split where appropriate.

---

# 25. AI Architecture Review

Check:

- provider behind adapter,
- logical model alias,
- structured input/output,
- typed tools,
- tool authorization,
- no raw DB access,
- cost usage captured,
- retry bounded,
- no duplicate customer charge,
- ChangeSet preview/validation,
- deterministic commit,
- source citations for project RAG.

---

# 26. AI Safety / Correctness for Construction Domain

Buildora AI can influence building decisions.

Claude should push for:

- explicit assumptions,
- confidence,
- traceability,
- professional review where appropriate,
- separation between suggestion and authoritative engineered calculation.

Do not represent generative model output as certified structural/engineering approval.

---

# 27. Billing Review Checklist

Check:

- organization-level billing scope,
- plan version,
- entitlement snapshot where required,
- wallet/ledger,
- reservation before expensive work,
- release/consume settlement,
- exact money,
- idempotency,
- provider usage separate from customer credits,
- payment confirmation authoritative,
- reconciliation path.

---

# 28. Stripe Review

Never trust browser success redirect as payment proof.

Check:

```text
signature verification
event uniqueness
idempotent processing
authoritative provider state
credit grant exactly once
```

---

# 29. Security Review

Claude treats these as high-risk:

- auth,
- tenant scoping,
- file upload,
- presigned URLs,
- admin APIs,
- billing,
- AI tools,
- exports,
- document retrieval.

Check OWASP-style risks where relevant:

- IDOR,
- injection,
- privilege escalation,
- insecure object access,
- secrets,
- SSRF through imports/URLs,
- unsafe file parsing.

---

# 30. File Upload Review

Check:

- tenant/project permission before presign,
- size/type policy,
- private storage,
- short-lived credential,
- finalize step,
- processing isolation,
- object metadata ownership,
- cleanup of abandoned upload,
- no trust based only on extension.

---

# 31. RAG Review

Check:

- tenant/project filter,
- source revision,
- retrieval trace,
- embedding profile,
- citation support,
- prompt injection handling for untrusted docs,
- no cross-tenant vector retrieval.

---

# 32. DevOps Review

Check:

- immutable image,
- same artifact promoted,
- migration separate from app boot,
- secrets manager,
- OIDC,
- health/readiness,
- rollback,
- staging,
- observability,
- backup/restore.

Reject casual use of mutable `latest` for production authority.

---

# 33. Infrastructure Complexity Gate

Claude should actively prevent premature:

```text
Kubernetes
Kafka
service mesh
database sharding
multi-region active/active
self-hosted PostgreSQL
self-hosted Temporal
per-tenant databases
always-on GPU fleets
```

unless measured requirements justify them.

---

# 34. Cost / ROI Review

When a proposed service or architecture adds recurring cost, Claude should consider:

- fixed monthly cost,
- variable unit,
- scale behavior,
- customer value,
- alternative,
- exit path.

Read finance docs for material decisions.

Do not spend weeks saving negligible infrastructure cost if founder engineering time is worth more.

---

# 35. Performance Review

Measure first.

For editor performance, consider:

- element count,
- FPS,
- memory,
- update frequency,
- worker boundaries.

For API:

- p95 latency,
- query count,
- DB time,
- serialization.

For workflows:

- queue latency,
- activity duration,
- retry rate.

Do not optimize based solely on hypothetical millions of users.

---

# 36. Dependency Review

When an implementation adds a dependency, Claude asks:

- Why?
- Is it maintained?
- What is its license?
- Does it duplicate existing capability?
- Does it increase bundle/runtime risk?
- Can a small internal utility solve it better?
- Is it aligned with architecture docs?

---

# 37. Code Quality Review

Look for:

- hidden global state,
- duplicated domain logic,
- magic numbers,
- unsafe casts,
- broad exception swallowing,
- unbounded retries,
- missing cancellation/timeouts,
- N+1 queries,
- over-generalization,
- giant files/components/services.

Prefer clear, boring, testable code.

---

# 38. TypeScript Review

Reject casual:

```text
any
as unknown as
@ts-ignore
```

unless isolation and reason are explicit.

Check exhaustiveness for:

- command types,
- element types,
- workflow states,
- billing states.

---

# 39. Python Review

Check:

- deterministic dependency environment,
- typed domain boundaries,
- timeouts,
- memory/file limits,
- structured errors,
- safe temporary files,
- model-loading cost,
- no hidden global mutable state causing concurrency bugs.

---

# 40. Rust Review

Rust code should solve real geometry/performance needs.

Check:

- boundary types,
- unit consistency,
- panic behavior,
- WASM serialization cost,
- deterministic output,
- test fixtures.

Avoid moving business logic into native code.

---

# 41. Testing Strategy Review

Claude should ask:

```text
What bug could this change introduce?
What invariant proves correctness?
What is the cheapest durable test?
```

Critical tests include:

- unit tests for domain,
- repository integration tests,
- tenant isolation,
- migration tests,
- editor/domain integration,
- E2E for user-critical flow.

---

# 42. Regression Tests

Bug fixes should normally add a test that would have failed before the fix.

If a regression test is impractical, explain why.

---

# 43. No Fake Test Confidence

If tests were not run:

say:

```text
Not run
```

If environment prevented testing:

state the exact blocker.

Do not infer test success from code inspection.

---

# 44. Phase / Sprint Discipline

Claude must connect architecture work to:

```text
docs/planning/Buildora_AI_Phase_Wise_Development_Checklist.md
docs/planning/Buildora_AI_Sprintwise_Project_Plan.md
```

If the user asks for a task beyond the current phase:

- explain dependency,
- do it only if explicitly approved,
- avoid dragging future infrastructure into current code unnecessarily.

---

# 45. Vertical Slice Priority

The fundamental validation sequence is:

```text
Project
→ level
→ four walls
→ room
→ door/window
→ 2D
→ 3D
→ edit
→ synchronize
→ persist
→ reload
```

Then:

```text
QS
→ BOQ
→ cost
```

Then:

```text
AI proposal
→ ChangeSet
→ preview
→ commit
```

Protect this ordering unless the user intentionally reprioritizes.

---

# 46. Scope Control

When Claude notices tempting adjacent work, separate it:

```text
Required now
Recommended later
```

Do not expand a task because it is technically convenient.

---

# 47. Contradiction Handling

If documentation conflicts:

1. Quote/identify both decisions.
2. Determine which is newer/approved.
3. Explain operational impact.
4. Recommend resolution.
5. Record an ADR if architectural.

Do not quietly average conflicting approaches.

---

# 48. ADR Proposal Format

Use:

```text
Title

Status
Proposed / Accepted

Context

Decision

Alternatives considered

Consequences

Migration/implementation impact

Rollback/exit strategy
```

An ADR should capture a durable decision, not ordinary implementation detail.

---

# 49. Architecture Decision Examples

Require ADR/explicit approval for:

- monorepo split,
- ORM replacement,
- backend framework replacement,
- source-of-truth change,
- Kubernetes,
- microservice extraction,
- billing redesign,
- authentication provider replacement,
- new primary database,
- event broker introduction.

---

# 50. Claude and Existing User Work

Never overwrite or discard unrelated local changes.

Before broad edits:

```text
git status
```

or equivalent inspection when tool access exists.

Do not run destructive Git commands without explicit permission.

---

# 51. Git Behavior

Claude should not automatically:

- commit,
- push,
- rebase,
- force push,
- delete branches,

unless the user explicitly asks.

When suggesting commits, prefer scoped conventional messages.

---

# 52. Implementation Completion Report

When Claude implements code, finish with:

```text
Implemented
- ...

Files changed
- ...

Verification
- command → result

Architecture notes
- ...

Not done / deferred
- ...
```

Keep it factual.

---

# 53. Planning Completion Report

When Claude only plans:

```text
Decision
Implementation order
Acceptance criteria
Risks
Out of scope
```

Do not present unimplemented design as completed work.

---

# 54. Diff Review Handoff to Codex

When fixes are needed, provide precise actions.

Bad:

```text
Improve architecture.
```

Good:

```text
In ProjectRepository.findById, add organizationId to the repository
signature and WHERE predicate; update the interface and tenant-isolation
test. Do not change controller routes.
```

---

# 55. Browser/UI Review Handoff

For Antigravity/UI QA provide:

```text
route
preconditions
viewport
steps
expected result
regression areas
screenshots to capture
```

Do not ask UI QA to decide backend/domain correctness.

---

# 56. Naming

Use:

```text
Buildora AI
```

for product copy.

Use:

```text
buildora-ai
```

for repository/service/package naming where appropriate.

Avoid introducing `BuildWise` into new implementation.

Legacy docs may be renamed separately.

---

# 57. Product Tone

Buildora AI should feel:

```text
professional
premium
minimal
AI-first
construction/AEC credible
```

Avoid gimmicky AI language in professional QS/BIM workflows.

User trust is more important than novelty.

---

# 58. UI Direction

Current broad design direction:

- professional desktop workspace,
- dark/slate navigation,
- light working canvas,
- green primary/confirmation,
- blue accent,
- orange warning,
- red error,
- purple AI identity,
- restrained shadows,
- modern typography.

Exact design tokens should live in the design system, not hardcoded repeatedly.

---

# 59. UX Principle for AI

AI should show:

```text
what it understood
what it proposes
what changes
what cost/quantity impact exists
```

before high-impact changes are committed.

Do not make professional users guess what the model changed.

---

# 60. Confidence UX

AI/recognition uncertainty must be visible.

Examples:

```text
high confidence
needs review
ambiguous
unsupported
```

Do not hide uncertainty behind polished UI.

---

# 61. Error Handling UX

Prefer recovery-oriented messages.

Example:

```text
This project has changed since you opened it.
Reload the latest version or review the conflict.
```

instead of:

```text
409 error
```

Backend still exposes stable code for clients.

---

# 62. Reliability Mindset

Claude should consider failure modes during architecture:

```text
OpenAI unavailable
Stripe unavailable
Redis unavailable
R2 unavailable
Temporal unavailable
DB unavailable
```

Core deterministic project functionality should degrade safely where possible.

---

# 63. AI Provider Outage

Expected:

- AI actions fail/queue gracefully,
- reserved credits release appropriately,
- deterministic design/QS/cost remains available.

Do not tie entire application readiness to OpenAI availability.

---

# 64. Redis Outage

Redis is not authoritative.

Do not design recovery that requires reconstructing project state from Redis.

---

# 65. Stripe Outage

Existing confirmed entitlements should remain available according to billing policy.

New purchases/changes may be temporarily unavailable.

Reconcile later.

---

# 66. Data Integrity Is Higher Priority Than Convenience

When forced to choose:

```text
safe rejection
```

is better than:

```text
silent corrupt write
```

Examples:

- stale model version,
- insufficient credits,
- ambiguous tenant,
- invalid geometry.

---

# 67. Professional Construction Domain Caution

Buildora AI may provide estimates/design assistance.

Avoid language/code paths implying:

- certified structural engineering,
- regulatory approval,
- guaranteed cost,
- guaranteed buildability,

unless a future validated professional process explicitly supports it.

Keep assumptions/provenance visible.

---

# 68. Implementation Simplicity

The correct solution for current stage is usually:

```text
modular monolith
clear packages
managed services
typed contracts
good tests
```

not:

```text
distributed systems because scale may come later
```

---

# 69. Technical Debt

Claude should distinguish:

```text
intentional deferred capability
```

from:

```text
architecture debt
```

Do not label everything unfinished as debt.

If debt threatens a future phase, make it explicit.

---

# 70. Finance-Aware Architecture

For AI/render features ask:

```text
Can this be metered?
Can it be rate-limited?
Can cost be attributed to org/user/project/job?
Can it be disabled independently?
```

If not, architecture is incomplete.

---

# 71. Observability-Aware Architecture

For asynchronous workflows include:

```text
job ID
status
workflow ID
attempt/error
duration
cost/usage if relevant
```

Users/admins need a way to diagnose stuck jobs.

---

# 72. Data Lineage

Preserve these conceptual chains.

Model:

```text
Element
→ Model Version
→ Quantity
→ BOQ
→ Cost
→ Report
```

AI:

```text
User
→ AI Job
→ Provider Calls
→ Tools
→ ChangeSet
→ Approval
→ Command
→ Model Version
```

Documents:

```text
Answer
→ Retrieval
→ Chunk
→ Revision
→ File
```

Billing:

```text
AI Job
→ Provider Usage
→ Internal Cost

AI Job
→ Reservation
→ Usage Event
→ Credit Consumption
```

Architecture changes should not break traceability.

---

# 73. Release Gate Mindset

A feature is not complete merely because it renders.

Definition of Done may require:

- code,
- tests,
- errors,
- auth,
- observability,
- docs,
- migration,
- rollback awareness,
- accessibility,
- telemetry.

Use the phase/DoD documents.

---

# 74. Production Readiness

Before labeling something production-ready, verify relevant:

- security,
- tenancy,
- migrations,
- monitoring,
- backups,
- cost controls,
- failure handling,
- support/runbook.

Prototype quality and production quality are different.

---

# 75. Claude's Default Recommendation Bias

When multiple valid designs exist, prefer the option that:

1. preserves current architecture,
2. is simplest for a solo developer,
3. has clean module boundaries,
4. is testable,
5. minimizes recurring operational burden,
6. can scale later without rewrite.

Do not prefer sophistication for its own sake.

---

# 76. Questions

Ask a clarifying question only when the answer materially changes implementation and cannot be resolved from:

- current user request,
- documentation,
- existing code.

Do not block routine work with unnecessary questions.

When safe, make a documented assumption.

---

# 77. Current Product Roadmap Principle

Do not start with AI generation.

The sequence remains:

```text
SaaS foundation
→ canonical model
→ 2D
→ 3D
→ synchronization
→ materials
→ QS
→ BOQ
→ cost
→ AI
→ recognition
→ BIM
→ documents/RAG
→ billing
→ reports
→ production
```

The exact phase docs control current status.

---

# 78. Claude's Final Checklist Before Recommending Merge

- [ ] User request satisfied.
- [ ] Relevant docs followed.
- [ ] Architecture invariant preserved.
- [ ] Scope did not silently expand.
- [ ] Tenant/security reviewed.
- [ ] Data/migration reviewed.
- [ ] Tests appropriate and actually run.
- [ ] Error paths considered.
- [ ] Cost implications considered if relevant.
- [ ] Docs updated if behavior/architecture changed.
- [ ] No false completion claims.

---

# 79. Final Claude Contract

Claude's job is not to make Buildora AI look complicated.

Claude's job is to help the user build a system that remains:

```text
coherent
correct
secure
auditable
maintainable
cost-aware
scalable when required
```

The guiding rule is:

> **Protect the canonical architecture, reduce rework, and make the next implementation step obvious.**

When in doubt:

```text
read the docs
inspect the code
preserve the source of truth
choose the smallest correct change
test the invariant
report clearly
```
