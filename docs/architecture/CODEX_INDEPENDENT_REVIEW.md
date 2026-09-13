# Buildora AI — Independent Adversarial Architecture Review

**Review date:** 2026-09-12  
**Scope:** Frozen architecture, ADR-001–ADR-018, canonical documents, standards,
planning, finance, examples, setup material, previous review, and the repository
filesystem as presented.  
**Method:** Mechanical path/count/search checks, cross-document reconciliation,
failure-scenario analysis, and primary-source external verification.  
**Independence:** The previous review was treated as an auditable claim set, not as
evidence.

This review began with the requested mechanical verification. It does not alter
the frozen architecture or implement any feature.

---

# 1. VERIFICATION RESULTS

## 1.1 Path integrity

I extracted literal `docs/*.md` references from `AGENTS.md`, `CLAUDE.md`, and
`docs/README.md`, normalized section/anchor suffixes, and checked the filesystem.

**Result: PASS for paths represented as existing.** Every explicit canonical
document path in the two agent contracts exists. The architecture ADR directory
and all referenced ADR files exist. `docs/README.md` deliberately lists unwritten
documents in its “planned” section by filename rather than representing them as
existing files; that is consistent with the contracts, which explicitly route
missing subjects through the nearest existing source (`AGENTS.md:138-141`,
`CLAUDE.md:149-152`). No broken mandatory documentation path was found.

## 1.2 Contradiction sweep

Case-insensitive whole-repository text matches at review time:

| Search | Matches | Classification |
|---|---:|---|
| `BuildWise` / `buildwise` | 8 | Mostly migration history/prohibitions; two self-defeating “none remains” statements, discussed below |
| `packages/domain` | 28 | Mostly prohibitions/history; **live filesystem contradiction**: the directory exists |
| `packages/core` | 24 | Prohibitions/history; no directory exists, which is correct |
| `varchar` | 21 | No live varchar-primary-key contradiction found |
| `prj_` | 4 | Examples of `public_id`, not FK/primary-key violations |
| `WebGPU` | 30 | Deferred/prohibited-as-baseline references; no live adoption found |
| literal `floor_to_floor` | 5 | Prose/schema discussion; no stored `floor_to_floor` column found |
| `centreline` | 33 | Mostly the intended wall/connectivity distinction; one QS semantic conflict is a live issue (§2.4) |
| `Zustand` | 37 | Consistently restricted to ephemeral UI state; no Building Model authority assigned to it |
| `fabricated` | 8 | Historical record of the withdrawn pricing allegation, not a current architecture claim |

The material live contradictions are:

1. The filesystem contains `packages/domain/`, although the accepted decision says
   it does not exist and must not be created (`AGENTS.md:301-302`,
   `docs/planning/Buildora_AI_Sprintwise_Project_Plan.md:273-274`).
2. The filesystem lacks `packages/model-session/`, although it is an MVP package
   and a named dependency boundary (`docs/architecture/04_Monorepo_and_Module_Architecture.md:65-69,75-109`).
3. `apps/ai-worker`, `apps/bim-worker`, `apps/render-worker`, `packages/ai-tools`,
   and `native/geometry-wasm` exist as empty directories, but the architecture
   permits premature placeholders only when they contain a README naming the gate
   (`docs/architecture/FINAL_ARCHITECTURE_REVIEW.md:220-220`). None contains that
   README.
4. `package.json`, `pnpm-workspace.yaml`, and `turbo.json` exist but are zero-byte
   files; `.github/workflows/` has no workflow. Therefore “Enforced in CI” is a
   future requirement, not a present repository fact
   (`docs/architecture/04_Monorepo_and_Module_Architecture.md:127-148`).
5. The database catalogue contains `levels.height_mm`
   (`docs/architecture/15_Database_and_Data_Architecture.md:899-916`) while the
   canonical domain permits only optional, non-authoritative
   `defaultStoreyHeightMm` and says elevation is the single vertical fact
   (`docs/architecture/05_Building_Model_Domain.md:83-104`).

The repository is at the Sprint 01 boundary, so empty `apps/web` and `apps/api`
are not a missed implementation deliverable: Sprint 01 itself is where bootstrap
begins (`docs/planning/Buildora_AI_Sprintwise_Project_Plan.md:309-315`). The drift
above is still worth correcting because agents use the current tree as evidence.

## 1.3 Count integrity

- There are **18 accepted ADR files**, `0001` through `0018`; `0000-template.md`
  is correctly excluded. The accepted-ADR table has 18 rows and explicitly reports
  “18 accepted” (`docs/architecture/34_ADR_Index.md:53-79`). The frozen baseline's
  list also contains 18 entries (`docs/architecture/FINAL_SYSTEM_ARCHITECTURE.md:521-544`).
  Every material “18 ADRs” prose claim agrees. **PASS.**
- `FINAL_ARCHITECTURE_REVIEW.md` contains eight P0 headings, P0-1 through P0-8,
  and its reconciliation table has eight rows
  (`docs/architecture/FINAL_ARCHITECTURE_REVIEW.md:834-850`).
  `OPEN_ARCHITECTURE_DECISIONS.md` also states eight reconciled P0 items
  (`docs/architecture/OPEN_ARCHITECTURE_DECISIONS.md:14-28`). **PASS.**
- The open-decision arithmetic is 9 resolved + 18 implementation-time + 16
  deferred = 43, matching OD-01 through OD-43
  (`docs/architecture/OPEN_ARCHITECTURE_DECISIONS.md:107-113`). **PASS.**
- Qualification: P0-5 was withdrawn rather than corrected, so “eight P0 labels”
  means seven substantive repaired defects plus one retained/withdrawn label. The
  arithmetic is still internally consistent
  (`docs/architecture/FINAL_ARCHITECTURE_REVIEW.md:840-849`).

## 1.4 External fact check

Only primary/official sources were used. “Maintained” means there was current
official documentation or release activity; it does not prove Buildora-specific
performance or fidelity.

### OpenAI pricing

**VERIFIED.** The finance figures for `gpt-5.6-luna`, `gpt-5.6-terra`,
`gpt-5.6-sol`, and `gpt-6-astra`
(`docs/finance/Buildora_AI_Development_Production_Costing_Plan.md:246-255`)
match OpenAI's official model pages as checked on the review date. The
`gpt-5.6-sol` page also states promotional pricing through
**21 November 2026**.[^openai-sol] The previous “fabricated” allegation was false;
its withdrawal is correct (`docs/architecture/FINAL_ARCHITECTURE_REVIEW.md:350-350`).

### NRM2

**VERIFIED, with a scope qualification.** NRM 2 is a real and current RICS
measurement standard for detailed measurement of building works. RICS identifies
the October 2021 edition, reissued October 2022 without material change.[^rics-nrm]
It is a defensible default for a UK-oriented detailed-BOQ product. It is not a
universal UK workflow guarantee, and “NRM2-compatible” must not be claimed until a
defined supported subset has conformance fixtures.

The requested arithmetic is correct **if 5000 × 4000 mm are wall centreline
dimensions and wall thickness is 200 mm**:

```text
(5.000 − 0.200) × (4.000 − 0.200) = 4.800 × 3.800 = 18.240 m²
```

NRM2 measures floor finishes by the area in contact with the base and applies
work-section-specific deduction rules. That is compatible with using derived
interior faces to obtain room finish geometry, but NRM2 does not itself mandate
Buildora's persisted Room topology.[^nrm2-pdf]

NRM2 roof/sheet covering measurement follows the actual base/profile and
distinguishes horizontal, sloping, vertical, and curved work. Thus “sloping area,
not plan projection” is correct for a sloped covering surface, but it is too broad
as a universal invariant for every roof quantity.[^nrm2-pdf]

### Technology currency and suitability

| Technology | Currency/maintenance | Suitability conclusion |
|---|---|---|
| PixiJS | **VERIFIED maintained** through official releases.[^pixi] | Suitable 2D rendering candidate. CAD hit-testing and 50k-element frame time are **UNVERIFIED** until a representative spike. |
| Three.js | **VERIFIED maintained** through official releases.[^three] | Suitable direct WebGL renderer. Buildora scene/update/disposal performance is **UNVERIFIED**. |
| web-ifc | **VERIFIED maintained/current docs**; it exposes IFC read/write in JavaScript/WASM.[^web-ifc] | Suitable browser-side IFC parsing candidate. Large-file memory, supported-schema fidelity, and round-trip coverage are **UNVERIFIED**. |
| That Open / Fragments | **VERIFIED current official documentation**.[^that-open] | Suitable BIM viewing/fragment candidate. Version compatibility with the selected web-ifc version and production-scale files is **UNVERIFIED**. |
| IfcOpenShell | **VERIFIED maintained** through its official repository/releases.[^ifcopenshell] | Suitable server-side IFC processing candidate; exact worker packaging and required entity coverage are **UNVERIFIED**. |
| Drizzle ORM | **VERIFIED maintained** through official releases.[^drizzle] | Suitable for typed PostgreSQL access, provided PostGIS/RLS/advanced SQL may use explicit SQL. Pin versions; API churn risk remains. |
| Temporal TypeScript SDK | **VERIFIED maintained** through official releases.[^temporal] | Suitable for the gated durable workflows. It remains unnecessary for ordinary CRUD, consistent with ADR-012. |
| Neon | **VERIFIED current service/docs**.[^neon] | Suitable initial managed PostgreSQL candidate. RLS must be tested with the exact driver and pooling mode (§2.10). |
| Clerk | **VERIFIED current official documentation**.[^clerk] | Suitable identity provider candidate if Buildora retains authorization ownership. Migration friction and final commercial fit are **UNVERIFIED**. |
| pgvector | **VERIFIED maintained** through official releases.[^pgvector] | Suitable PostgreSQL-first embedding baseline. Recall/latency for Buildora's corpus is **UNVERIFIED** until measured. |

### Infrastructure pricing

- **Cloudflare R2: VERIFIED.** The finance assumptions
  (`docs/finance/Buildora_AI_Development_Production_Costing_Plan.md:592-604`)—$0.015/GB-month standard
  storage, $4.50/million Class A, $0.36/million Class B, stated free tiers, and no
  Internet egress charge—match the official pricing page.[^r2]
- **AWS Fargate: VERIFIED for the documented example region/rates.** The assumptions
  and arithmetic (`docs/finance/Buildora_AI_Development_Production_Costing_Plan.md:626-674`)
  based on Linux/x86 vCPU-hour and GB-hour pricing are correct. Actual spend remains
  region/configuration dependent.[^fargate]
- **AWS Application Load Balancer and NAT Gateway: VERIFIED as illustrative
  regional assumptions.** The documented figures
  (`docs/finance/Buildora_AI_Development_Production_Costing_Plan.md:678-733`)
  and their hourly-plus-capacity/data structure are consistent with AWS official
  pricing.[^alb][^vpc]
- **Neon: VERIFIED as a point-in-time assumption.** The documented compute/storage
  figures (`docs/finance/Buildora_AI_Development_Production_Costing_Plan.md:512-527`)
  and entry-level estimate are directionally and arithmetically consistent
  with Neon pricing checked on the review date.[^neon-pricing]
- The primary AWS region remains undecided
  (`docs/architecture/OPEN_ARCHITECTURE_DECISIONS.md:47-57`). Therefore none of
  these AWS numbers is a committed production budget. The finance documents must
  retain their re-price-at-deployment caveat.

## 1.5 Repository reality summary

Observed filesystem structure:

```text
apps/          web, api, ai-worker, bim-worker, render-worker (all empty)
packages/      api-contracts, building-model, cad-2d, cost-engine,
               design-system, engine-3d, qs-engine, units (all empty)
               domain (empty, but forbidden)
               model-session (missing)
               ai-tools (empty, prematurely present)
native/        geometry-wasm (empty, prematurely present)
infrastructure/environments, modules, scripts (empty)
```

This is not implementation drift—the project has not begun Sprint 01—but it is
**bootstrap-shape drift** from ADR-011/ADR-012. The documentation is much more
complete than the executable repository, so present-tense claims such as “enforced
in CI” must be read as acceptance criteria until Sprint 01 establishes the checks.

---

# 2. ERRORS FOUND IN THE FROZEN ARCHITECTURE

## 2.1 The database reintroduces a second level-height fact

**File:line:** `docs/architecture/15_Database_and_Data_Architecture.md:899-916`; conflicts with `docs/architecture/05_Building_Model_Domain.md:83-104` and ADR-005.

**Claim:** `Level.elevationMm` is the single authoritative vertical fact and
floor-to-floor is derived, yet the level table contains an unexplained `height_mm`.

**Why wrong/unimplementable:** A generic persisted height is indistinguishable to
an implementer from authoritative storey height. It does not preserve the semantic
distinction made by `defaultStoreyHeightMm`.

**Concrete failure:** Level 1 is at 0, Level 2 at 3000, while `height_mm=2800`.
One adapter extrudes a wall to 2800; another interprets storey height as 3000;
stair validation and 3D disagree although both follow a documented value.

**Correction:** Rename the column to `default_storey_height_mm`, explicitly mark it
nullable/non-authoritative, and forbid it in derived elevation/floor-to-floor
calculations. Add a schema invariant test.

**Severity: HIGH**

## 2.2 Level ordering and top-level floor-to-floor are undefined

**File:line:** `docs/architecture/05_Building_Model_Domain.md:83-104`;
`docs/architecture/15_Database_and_Data_Architecture.md:918-922`.

**Claim:** Floor-to-floor is next level elevation minus current elevation; the UI
edits it by changing the next level.

**Why wrong/unimplementable:** “Next” is not defined as elevation order or sequence
order, consistency between those orders is not required, equal elevations are not
addressed, and there is no next level for the top or only level.

**Concrete failure:** A user creates Roof (sequence 3, elevation 6000), then later
inserts Level 1 (sequence 2, elevation 3000), or accidentally assigns sequence 2 to
elevation 7000. Different sort choices yield different floor-to-floor values. On a
single-level project, the editor has no valid target for the advertised edit.

**Correction:** Define canonical ordering by `(elevationMm, stable-id)` or enforce
strictly increasing unique elevations aligned with sequence; define top-level
floor-to-floor as absent (`null`), never zero; expose the optional design-aid height
separately. Add out-of-order, equal-elevation, basement, and single-level tests.

**Severity: MEDIUM**

## 2.3 The stated professional model cannot represent several named building forms

**File:line:** `docs/architecture/05_Building_Model_Domain.md:209-224,327-382`.

**Claim:** The canonical Wall is one straight start/end centreline with scalar
thickness and height; Roof is a polygon plus a pitch/plane. There is no ceiling
element or ceiling-surface definition.

**Why wrong/unimplementable:** That schema cannot natively express an arc wall,
thickness varying along a wall, a vaulted/sloped ceiling independent of the roof,
or a non-planar/multi-plane roof. Split levels and mezzanines can be approximated
as Levels, and multi-storey columns are expressly supported by top-level reference
(`docs/architecture/05_Building_Model_Domain.md:337-340`), but the former lack
partial-storey semantics.

**Concrete failure:** A 6 m radius curved external wall tapering from 200 to 300 mm
must be tessellated into straight constant-thickness Walls. The segments introduce
fake junctions and IDs; a hosted window offset can cross a segment boundary; room
perimeter and opening deductions acquire seams. A barrel-vault ceiling has no
canonical surface, so ceiling-finish quantity cannot be reproduced from the model.

**Correction:** State a strict MVP geometry capability envelope and reject
unsupported inputs. Before marketing professional coverage beyond that envelope,
add explicit path variants (line/arc), thickness/height profiles, ceiling surfaces,
and multi-plane roof faces—or document a canonical compound-element segmentation
model that preserves one logical element and continuous measurement.

**Severity: HIGH**

## 2.4 Wall net area is incorrectly fixed in the domain instead of the ruleset

**File:line:** `docs/architecture/05_Building_Model_Domain.md:219-224`; conflicts
with ADR-009's versioned measurement policy
(`docs/architecture/adr/0009-qs-measurement-rulesets.md:25-69`).

**Claim:** `Net area = gross area − Σ hosted opening areas` is a Wall invariant.

**Why wrong/unimplementable:** NRM2 deductions depend on the measured work section
and threshold. For example, NRM2 masonry and finish sections use different small-
opening deduction thresholds.[^nrm2-pdf] “Net wall area” is therefore not a single
domain truth. The domain can expose geometric surface and opening contributions;
the selected QS ruleset must decide what is deductible.

**Concrete failure:** A 600 × 600 mm service opening is 0.36 m². The domain
invariant deducts 0.36 m² from all wall quantities. An NRM2 masonry item that does
not deduct a void at or below its applicable threshold should retain the area. The
same model now produces a ruleset-incompatible BOQ before the ruleset runs.

**Correction:** Remove “net area” as a universal Wall invariant. Persist/derive
gross geometry and typed opening contributions. Compute measurement-specific
deduction and net values only inside a pinned ruleset execution.

**Severity: HIGH**

## 2.5 Room topology chooses the boundary principle but not deterministic junction behavior

**File:line:** `docs/architecture/adr/0006-room-space-topology.md:28-62,79-104,126-135`.

**Claim:** Interior faces are offset from wall centrelines “with junctions
resolved”; either half-edge subdivision or polygon/face booleans may implement it.

**Why wrong/unimplementable:** The load-bearing semantics are not the library or
algorithm choice. They are tolerances, snap/merge rules, T/X/acute junction rules,
butt-versus-miter ownership, variable thickness behavior, open boundary behavior,
invalid transient states, and stable room identity across split/merge. None is
specified. Different correct geometry libraries can return different room areas
and identities.

**Concrete failure:** A 200 mm and 300 mm wall meet at 35°. Extending both interior
offsets to a miter, trimming one as the dominant wall, or beveling the corner
produces three valid polygons with different area/perimeter. Delete one bounding
wall: the enclosed face disappears. The ADR says the room keeps its stable ID after
a wall change but does not say whether an opened region deletes, orphans, stales,
or preserves that room. If one room splits into two, both cannot keep the one ID.

**Correction:** Before Sprint 09 wall geometry is relied upon and no later than
Sprint 15, publish a deterministic topology contract: numeric tolerance model,
junction classification/ownership, room-separator/open-boundary concept, transient
invalid-state policy, region matching score, split/merge ID policy, and deletion/
orphan transitions. Golden fixtures must include unequal thickness, acute/T/X
junctions, small gaps, nested loops, overlap, delete, split, and merge.

**Severity: BLOCKER before room/QS implementation**

## 2.6 “Rulesets are data, not code” is internally qualified into a hybrid but no hybrid contract exists

**File:line:** `docs/architecture/adr/0009-qs-measurement-rulesets.md:25-63`.

**Claim:** Rulesets are versioned data; alternate standards require no code change,
except when a genuinely novel rule primitive is needed.

**Why wrong/unimplementable:** The exception negates the absolute claim. The shown
schema contains identity, dates, a hash, and `metadata_jsonb`, but no typed rule
language, primitive registry, evaluation semantics, rounding order, validation, or
compatibility policy. Real NRM2 rules include classifications, dimension bands,
conditional deductions, descriptions, and section-specific inclusions.

**Concrete failure:** A second ruleset requires a conditional deduction based on
opening area, wall form, and measured item. If encoded ad hoc in `metadata_jsonb`,
two engine versions interpret the same hash differently. If added as code, “no
code change” is false and old runs are not reproducible unless the executable
primitive version is also pinned.

**Correction:** Amend ADR-009 to a versioned hybrid: validated declarative rule
documents over a deliberately small, versioned executable primitive set. Hash the
canonical rule document and primitive/engine bundle. Define rounding/evaluation
order and conformance fixtures. Call Sprint 23 output “Buildora Basic/NRM2 subset”
until the supported NRM2 work sections are explicitly audited.

**Severity: HIGH**

## 2.7 Snapshot cadence does not bound reconstruction after a large one-version change

**File:line:** `docs/architecture/adr/0002-model-persistence-and-versioning.md:49-56,81-88`.

**Claim:** A snapshot about every 50 versions and **before** large imports/AI
commits bounds worst-case reconstruction cost.

**Why wrong/unimplementable:** Version count is not replay work. A single command
may contain thousands of change items. A pre-operation snapshot leaves the entire
large operation on the replay tail.

**Concrete failure:** At v49 the model has a snapshot. v50 imports 40,000 elements
as one atomic command. A recovery from the “last snapshot” still decodes and
applies all 40,000 items; the promised bound of roughly 50 ordinary edits is
irrelevant.

**Correction:** Trigger snapshots/checkpoints after successful large commits as
well as before them, and use measured replay cost, serialized bytes, and change-
item count in addition to version count. Specify asynchronous snapshot completion
and fallback behavior.

**Severity: MEDIUM**

## 2.8 Replay versioning has a column but no compatibility mechanism

**File:line:** `docs/architecture/adr/0002-model-persistence-and-versioning.md:83-94`;
`docs/architecture/15_Database_and_Data_Architecture.md:1183-1201`.

**Claim:** Mandatory `command_schema_version` keeps evolving commands replayable.

**Why wrong/unimplementable:** A version discriminator does not deserialize or
apply old semantics. No upcaster registry, frozen handler policy, canonical
serialization, snapshot upgrade policy, or retirement rule is specified.

**Concrete failure:** `MOVE_WALL` v1 stores absolute endpoints; v2 stores a delta
and constraint mode. After the v1 handler is removed, a v1 journal entry either
fails or is interpreted as v2. A snapshot made under geometry schema v1 may also
be unreadable even if the command is upcast.

**Correction:** Define a command/snapshot compatibility contract: immutable
envelope, runtime schema per version, deterministic upcaster chain or retained
handlers, migration tests from every supported production version, and a rule that
history cannot be pruned until a durable upgraded snapshot is verified.

**Severity: HIGH**

## 2.9 Undo and queued-command reconciliation are not defined for dependent edits

**File:line:** `docs/architecture/adr/0002-model-persistence-and-versioning.md:35-47,92-95`;
`docs/architecture/adr/0008-model-session.md:91-105,130-139`.

**Claim:** Undo is a compensating command; on conflict the session will “reload
and replay, or surface the conflict.”

**Why wrong/unimplementable:** The disjunction is the critical product policy,
not an implementation detail. Neither inverse-command preconditions nor dependency
handling for queued optimistic commands is specified. The assertion that session
state is always discardable (`docs/architecture/adr/0008-model-session.md:72-77`)
conflicts with unacknowledged user commands unless they are durably retained.

**Concrete failure:** A commits v5 `MOVE_WALL`; B commits v6 moving a hosted door.
A requests undo. Applying the raw inverse at v7 can invalidate B's door or overwrite
B's intent; rejecting it needs a defined UX. Separately, commands 1–30 are queued
and command 7 fails validation; 8–30 were optimistically calculated from the state
including 7. Dropping 7 and replaying the suffix may change or invalidate all of
them. Reloading without a durable queue loses user work.

**Correction:** Before Sprint 12, define accepted-prefix processing, per-command
preconditions, dependency/cascade metadata, deterministic rebase rules, quarantine
of an invalid suffix, conflict UI, and local crash persistence. Undo must be a new
command against current `baseVersion`, with preview and semantic conflict—not a
blind inverse payload.

**Severity: BLOCKER before optimistic persistence/undo**

## 2.10 Command idempotency is optional where retry safety requires it

**File:line:** `docs/architecture/15_Database_and_Data_Architecture.md:1183-1201`.

**Claim:** Model commands have a nullable `idempotency_key`.

**Why wrong/unimplementable:** A client cannot know whether a timed-out command
committed. Retrying without a required key can apply a semantic mutation twice.

**Concrete failure:** `MOVE_WALL_BY(deltaX=100)` commits, but the HTTP response is
lost. The client retries; the wall moves 200 mm and two immutable journal entries
look individually valid.

**Correction:** Require a client-generated idempotency key for every externally
submitted mutation, unique within organization/project (or command stream), and
persist the original response/version for replay. Internal deterministic cascade
items remain covered by the parent command transaction.

**Severity: HIGH**

## 2.11 Cost-estimate storage does not contain the lineage frozen by ADR-015

**File:line:** `docs/architecture/adr/0015-qs-boq-cost-reproducibility.md:25-46`;
`docs/architecture/15_Database_and_Data_Architecture.md:1905-1928,1981-1999`.

**Claim:** Every cost estimate pins quantity run and cost assumption-set version,
and the tuple yields byte-identical output forever.

**Why wrong/unimplementable:** The actual `cost_estimates` schema omits both
`quantity_run_id` and `cost_assumption_set_id/version`. A BOQ may indirectly link a
quantity run, but the assumption input is not pinned at all. Further, byte-identical
reports depend on template, renderer, fonts, locale/timezone, canonical serialization,
and build artifact; ADR-015 only promises an engine version.

**Concrete failure:** Estimate E uses 10% overhead from assumption set v3. The
project moves to v4 at 12%. E records totals but no v3 reference. Reproduction uses
v4 and changes the result. Even if totals match, a PDF regenerated after a font or
renderer update differs byte-for-byte.

**Correction:** Add immutable FKs to quantity run and assumption-set version.
Pin/hash rules, engine build artifact, rounding policy, unit-conversion version,
rate normalization, report template/renderer/font bundle, locale, and timezone as
applicable. Promise canonical **numeric equivalence** for recalculation; preserve
issued byte-identical artifacts by content hash/object storage rather than claiming
arbitrary future regeneration is byte-identical.

**Severity: BLOCKER before Sprint 25–26 costing/report issuance**

## 2.12 AI write isolation is not structurally guaranteed by a module boundary alone

**File:line:** `docs/architecture/adr/0013-ai-changeset-safety.md:25-50`.

**Claim:** It is structurally enforced—not conventional—that the AI module cannot
write model tables because it has no model repository access.

**Why wrong/unimplementable:** In a modular monolith, modules normally share the
process and database credentials. Import-boundary checks can prevent an intended
dependency, but they do not make the credential incapable of executing SQL or
protect against an accidentally injected generic DB handle.

**Concrete failure:** A refactor exposes Drizzle's database client to a shared
service. An AI orchestration handler imports it without the model repository and
updates `building_elements`; TypeScript module boundaries do not stop a credential
that has database permission.

**Correction:** Keep the module/import rule, but stop calling it a complete
structural barrier. Give the AI-facing execution context a least-privilege DB role
that can write drafts/change sets but not canonical model tables, or require all
canonical writes through a separately permissioned model-command gateway. Test DB
privileges as well as imports.

**Severity: HIGH**

## 2.13 AI preview cost and capability-aware routing are missing operational contracts

**File:line:** `docs/architecture/adr/0013-ai-changeset-safety.md:25-38,60-66`;
`docs/architecture/FINAL_SYSTEM_ARCHITECTURE.md:398-404,423-428`.

**Claim:** Every AI ChangeSet receives geometry validation and QS/cost impact;
four logical aliases abstract providers.

**Why wrong/unimplementable:** No synchronous/async threshold, incremental impact
strategy, compute budget, cancellation behavior, or stale-preview cache is defined.
Aliases describe quality/cost tiers but not required capabilities such as tool
calling, structured output, context length, modality, region, or data policy.

**Concrete failure:** An iterative AI session on a 50,000-element model proposes
ten small changes. Ten full topology/QS/cost recomputations exceed an interactive
latency and compute budget. Separately, routing `BALANCED` to a cheaper model that
lacks a required structured/tool capability makes a valid task fail despite the
alias being “available.”

**Correction:** Make route selection `(alias, required capabilities, policy)` and
record the resolved model snapshot. Define incremental affected-element previews,
budgets, async thresholds, cancellation, and stale-base invalidation. Full final
validation remains mandatory at commit.

**Severity: MEDIUM**

## 2.14 Billing protects the customer charge but leaves a provider-completion gap

**File:line:** `docs/architecture/FINAL_SYSTEM_ARCHITECTURE.md:433-446`; ADR-014
reservation/TTL policy.

**Claim:** Reserve, execute, then settle/release; retries and escalation never
double-charge; TTL reconciliation repairs abandoned reservations.

**Why wrong/unimplementable:** Idempotent ledger settlement can prevent a duplicate
customer charge, but it does not by itself make provider completion, durable job
completion, and settlement request atomic. A TTL reconciler can only act on durable
evidence it can see.

**Concrete failure:** The provider completes an expensive render/AI call and the
worker dies before recording completion. The reservation expires. Reconciliation
sees no completed job and releases it, causing lost revenue; a retry may incur the
provider cost again. If delivery occurred outside the durable job transition, the
system cannot infer the truth.

**Correction:** Persist provider-call attempts and provider result IDs; make durable
job completion and a settlement outbox record one local transaction before result
delivery. Reconciliation should inspect provider state where possible and route
ambiguous cases to an auditable exception state. Keep one idempotency key for the
customer-visible job.

**Severity: HIGH**

## 2.15 Billing read-model consistency and MVP scope remain underspecified

**File:line:** `docs/architecture/OPEN_ARCHITECTURE_DECISIONS.md:39-57`;
`docs/architecture/24_Billing_AI_Credits_and_Usage.md:1288-1305,4842-4869`.

**Claim:** A cached balance may be transactionally maintained and rebuildable, and
the billing release split can wait until Sprint 36.

**Why wrong/unimplementable:** “Transactionally maintained” needs an update and
rebuild protocol; delaying the release split lets implementation follow the much
broader canonical billing checklist before the product has revenue.

**Concrete failure:** A ledger entry commits but the balance projection update
fails in a separate operation. Entitlement reads reject a funded job, or accept an
unfunded one. Separately, a solo developer implements top-ups, proration,
reconciliation, and admin surfaces before the first simple subscription purchase,
delaying sellability without validating demand.

**Correction:** For MVP, derive balance in the same database transaction or update
a wallet aggregate under the same row lock and provide a deterministic rebuild/
verification job. Resolve OD-21 before billing implementation starts, not during
it: Release 1 should be subscription + entitlements + ledger + reserve/settle;
defer top-ups/proration/admin sophistication.

**Severity: MEDIUM**

## 2.16 RLS policy timing is sound, but the runtime contract must be made normative

**File:line:** `docs/architecture/15_Database_and_Data_Architecture.md:497-577`;
`docs/architecture/adr/0016-tenant-isolation-and-rls.md:35-74,124-134`.

**Claim:** RLS is designed/tested with the real pooled model from Sprint 05 and
enforced before external beta.

**Why incomplete:** This can work with Neon/PgBouncer, but only if tenant context is
transaction-local and every scoped query uses the same database transaction. In
transaction pooling, session state cannot safely be assumed across transactions.
PostgreSQL also exempts owners/`BYPASSRLS` roles unless role design/`FORCE ROW LEVEL
SECURITY` accounts for that.[^pg-set][^pgbouncer]

**Concrete failure:** Request A sets a session-level tenant variable and returns a
pooled connection. Request B receives that connection or its query runs on another
server connection; it sees the wrong tenant context or none. Alternatively, tests
run under a table-owner role and appear to pass application scoping while silently
bypassing RLS.

**Correction:** Promote the existing database-document guidance to the ADR's
normative recipe: begin transaction → `SET LOCAL`/`set_config(..., true)` → all
tenant queries on that transaction → commit/rollback. Runtime application roles
must be non-owner, non-superuser, and lack `BYPASSRLS`; decide where `FORCE ROW
LEVEL SECURITY` is required. Run CI integration tests with RLS actually enabled
from Sprint 05 even if every local path does not enable it.

**Severity: HIGH before Sprint 05 tenancy work exits**

## 2.17 Repository architecture claims are written as accomplished facts

**File:line:** `docs/architecture/04_Monorepo_and_Module_Architecture.md:75-148`;
`docs/planning/Buildora_AI_Sprintwise_Project_Plan.md:255-274`.

**Claim:** The package graph exists, `packages/domain` was removed everywhere, and
dependency rules are enforced in CI.

**Why wrong:** The observed tree contains forbidden `packages/domain`, lacks
`packages/model-session`, has empty premature placeholders, zero-byte root package
files, and no CI workflow.

**Concrete failure:** A Sprint 01 agent sees the actual directory and places shared
types in `packages/domain`, then creates a dependency that the nonexistent CI rule
cannot reject. The documentation has declared the opposite state, so neither the
plan nor automated review notices the drift.

**Correction:** Treat these as Sprint 01 acceptance criteria until implemented.
Delete the empty forbidden directory, create only packages with current consumers,
give allowed deferred placeholders their required gate README, create
`model-session` at its owning sprint, and add the dependency rule before claiming
CI enforcement.

**Severity: HIGH for bootstrap integrity; LOW implementation cost**

---

# 3. ERRORS FOUND IN THE PREVIOUS REVIEWER'S REASONING

1. **The original pricing non-existence claim was a verification failure.** The
   reviewer has correctly withdrawn it (`docs/architecture/FINAL_ARCHITECTURE_REVIEW.md:350-350`),
   and this review independently verified the models, prices, and promotion date.
   The process error was asserting non-existence without checking the provider's
   official model pages.

2. **The name-repair text is internally corrupted.** P0-1 says every document had
   the forbidden `Buildora AI` name and then recommends renaming `Buildora_AI_*` to
   the same spelling (`docs/architecture/FINAL_ARCHITECTURE_REVIEW.md:684-700`).
   The contradiction matrix makes the same inversion
   (`docs/architecture/FINAL_ARCHITECTURE_REVIEW.md:543-543`), and the final locked
   list says the name `Buildora AI` must not appear
   (`docs/architecture/FINAL_ARCHITECTURE_REVIEW.md:1457-1459`). The intended old
   name was `BuildWise`. This is editorial, but it shows that high-confidence
   reconciliation text was not mechanically checked.

3. **Resolved findings are duplicated as unresolved P1 work.** Opening
   consolidation and element type identity appear as P0-7/P0-8
   (`docs/architecture/FINAL_ARCHITECTURE_REVIEW.md:804-830`) and again as P1-1/P1-2
   (`docs/architecture/FINAL_ARCHITECTURE_REVIEW.md:857-883`) after the document
   says all eight P0 items are reconciled and resolved
   (`docs/architecture/FINAL_ARCHITECTURE_REVIEW.md:834-853`).

4. **The final locked list preserves superseded architecture.** It puts the editor
   session store in `packages/cad-2d` and lists a `core` domain package
   (`docs/architecture/FINAL_ARCHITECTURE_REVIEW.md:1439-1448`), contradicting its
   own corrected model-session recommendation
   (`docs/architecture/FINAL_ARCHITECTURE_REVIEW.md:893-897`) and ADR-011.

5. **“Nothing” must change overstates the result.** The conclusion says nothing
   remains before implementation (`docs/architecture/FINAL_ARCHITECTURE_REVIEW.md:1333-1333`)
   and approves the architecture (`docs/architecture/FINAL_ARCHITECTURE_REVIEW.md:1367-1367`).
   It missed the concrete schema contradictions in §§2.1 and 2.11, the unbounded
   large-command replay tail, optional command idempotency, and the absent
   model-session/forbidden domain directory in the actual tree.

6. **“Byte-identical forever” was accepted without testing the boundary.** The
   prior review repeats the six-part lineage claim
   (`docs/architecture/FINAL_ARCHITECTURE_REVIEW.md:1414-1418`) but did not reconcile
   it to the cost-estimate table or distinguish numeric determinism from report
   bytes. The actual schema omits a promised input (§2.11).

7. **Professional QS correctness was inferred from two simple golden examples.**
   The reviewer was right about the arithmetic and interior-face principle, but
   did not test the Wall domain invariant against NRM2's work-section-specific
   deduction rules. One arithmetic fixture is not NRM2 conformance.

8. **Scalability claims were too confident for a documentation-only repository.**
   The review itself concedes that 50,000 elements “requires engineering”
   (`docs/architecture/FINAL_ARCHITECTURE_REVIEW.md:498-498`), while the approved
   narrative implies the route to 100k avoids rewrite. With no representative
   model, renderer benchmark, topology implementation, or database load test,
   that conclusion is **UNVERIFIED**, not established.

9. **The self-audit claims are literally false as written.** `AGENTS.md` says no
   `BuildWise` reference remains while using the term in the same paragraph
   (`AGENTS.md:2266-2269`); the sprint exit gate similarly claims no references
   while retaining historical ones
   (`docs/planning/Buildora_AI_Sprintwise_Project_Plan.md:293-300`). The intended
   rule—no live product use—is sound; the grep-based claim is not.

---

# 4. WHERE THE ARCHITECTURE IS CORRECT

- **ADR-001 holds.** I found no competing authoritative 2D, 3D, IFC, QS, or cost
  state. Renderer scenes are consistently disposable projections, and IFC is
  interchange. That is the right irreversible foundation
  (`docs/architecture/FINAL_SYSTEM_ARCHITECTURE.md:552-560`).
- **ADR-003 holds.** A NestJS modular monolith with separate Python workers at real
  language/runtime boundaries is proportionate for one developer. The plan avoids
  premature microservices and gates workers to consumers.
- **ADR-004 holds.** UUID database identity plus a separate human-facing
  `public_id` is coherent; the contradiction scan found no live `varchar`/`prj_`
  primary-key or FK design.
- **ADR-005's datum principle holds.** Ground-floor FFL = 0, signed level elevation,
  and level-relative element z compose cleanly for basements and split elevations.
  The defect is the database's stray `height_mm`, not the datum decision.
- **Opening semantics hold.** A Door/Window as the physical opening plus standalone
  fixture-less holes, with one QS adapter, avoids double deductions
  (`docs/architecture/FINAL_SYSTEM_ARCHITECTURE.md:245-249`).
- **ADR-007 holds.** Separating “what it is” from “what it costs” is necessary for
  schedules, type edits, and independent assembly/rate evolution
  (`docs/architecture/FINAL_SYSTEM_ARCHITECTURE.md:251-258`).
- **ADR-008's ownership boundary holds.** A renderer-neutral model session is the
  correct shared working copy and preserves the no-2D↔3D dependency rule. The
  reconciliation protocol, not the package placement, needs specification.
- **ADR-010 holds.** Logical aliases plus versioned resolved-provider/rate metadata
  are the right volatility boundary. The verified OpenAI pricing supports rather
  than undermines this decision.
- **ADR-011/012's direction holds.** Downward package dependencies and consumer-
  gated infrastructure are appropriate for a solo developer. Repository bootstrap
  has simply not made those controls real yet.
- **ADR-014's ledger/reservation core holds.** An append-oriented ledger,
  reservations, idempotent settlement, and separation of customer credits from
  provider tokens are materially safer than a mutable balance field. The provider
  completion handoff needs one additional durable boundary.
- **ADR-016's staged RLS decision holds.** There is no external-tenant exposure
  window if the hard gate is honored, and application scoping is correctly required
  throughout. `SET LOCAL` inside a transaction is compatible with transaction
  pooling; it needs to be made a tested normative recipe.
- **ADR-017 holds.** WebGL2 as baseline and WebGPU as later progressive enhancement
  is the conservative production choice. No current repository adoption
  contradicts it.
- **ADR-018 holds.** Treating IFC as interchange rather than the editable internal
  authority avoids forcing product semantics into an external schema while still
  permitting GUID mappings.
- **Golden arithmetic holds.** The 5.0 × 2.7 m wall less 1.0 × 2.1 m opening is
  13.5 − 2.1 = **11.4 m²**. The 5.0 × 4.0 m centreline room inside 200 mm walls is
  **18.24 m²**. The remaining issue is when a measurement standard permits each
  deduction, not arithmetic.
- **Finance arithmetic checked holds.** OpenAI, R2, and the cited illustrative
  AWS/Neon rates were not fabricated. Region and future price volatility are
  appropriately reasons to configure/re-price, not reasons to reject the stack.

---

# 5. DISAGREEMENTS WITH FROZEN DECISIONS

## ADR-006 — Room/Space topology

**Objection:** I agree with interior-face boundaries and persisted stable identity,
but disagree that the ADR has made the irreversible decision sufficiently complete.
It freezes an outcome while deferring the semantics that determine the outcome.

**Instead:** Freeze a deterministic topology behavior contract before room data is
persisted, while leaving the specific algorithm/library evidence-driven.

**Cost if changed later:** **High.** Saved room boundaries, IDs, finish quantities,
BOUNDS relationships, revisions, and issued BOQs may all need recomputation and
lineage migration.

## ADR-009 — Versioned QS measurement rulesets

**Objection:** I disagree with the absolute “rulesets are data, not code” premise.
It is already contradicted by the admitted need for new executable primitives.

**Instead:** Versioned declarative rules over a versioned, deterministic primitive
runtime, with explicit supported-standard scope and conformance fixtures.

**Cost if changed later:** **High.** Published ruleset hashes and historical runs
may become uninterpretable; every quantity result may need a compatibility runtime
or migration.

## ADR-015 — Reproducibility lineage

**Objection:** I agree with full pinned lineage but disagree with “byte-identical
forever” as phrased and with declaring the tuple complete while the DB schema omits
inputs.

**Instead:** Promise canonical numeric/result equivalence from a content-addressed
input and engine bundle. Preserve an issued report's exact bytes as an immutable,
checksummed artifact. Add the missing assumption and quantity references.

**Cost if changed later:** **Very high.** Missing lineage cannot be reconstructed
after estimates are issued; this is a blocker before Sprint 25.

I do **not** recommend replacing ADR-001, 003, 004, 005's datum decision, 007, 008's
ownership decision, 010–012, 014's ledger foundation, 016–018. Their defects are
implementation contracts or overclaims that can be corrected without changing the
chosen architectural direction.

---

# 6. IMPLEMENTATION RISKS — SPRINTS 01–26, RANKED

| Rank | Risk | Sprint pressure | Likely consequence | Required gate |
|---:|---|---|---|---|
| 1 | Room/junction topology semantics and stable identity | S09–S15, then S23 | Wrong room areas, unstable IDs, cascading QS corruption; most likely component to consume **3×** its estimate | Deterministic topology specification and adversarial fixtures before Room persistence |
| 2 | Model-session rollback/rebase and semantic undo | S11–S13 | Lost optimistic edits, invalid dependent commands, multi-user overwrite | Accepted-prefix/rebase/conflict protocol and queue persistence tests before editor persistence ships |
| 3 | QS rule model mistaken for arbitrary JSON data | S23 | A “NRM2” label on non-conformant calculations; later engine rewrite | Typed hybrid rules runtime and declared supported NRM2 subset |
| 4 | Incomplete cost lineage | S24–S26b | Issued estimate/report cannot be reproduced | Schema must pin assumption set, quantity lineage, engine/rule bundles, and artifact metadata before first estimate |
| 5 | Canonical model capability envelope | S08–S16 | Imported/design geometry cannot represent curved/tapered/non-planar/ceiling cases without lossy segmentation | Publish supported/unsupported MVP geometry and reject unsupported cases explicitly |
| 6 | Tenant isolation under real pooling | S05 onward | Cross-tenant read/write despite apparently correct repositories | RLS-on CI tests using exact Neon driver/pool and least-privilege runtime role |
| 7 | Command replay/idempotency | S11 onward | Duplicate edits after timeout; old histories fail after schema change | Required mutation idempotency and command/snapshot upcaster contract |
| 8 | Solo-developer sellability at Sprint 26b | S23–S26b | A report demo exists but is not commercially defensible or operationally sellable | Define “sellable” narrowly: supported geometry + audited QS subset + reproducible cost + auth/tenancy + payment/entitlement path |
| 9 | CI/documentation reality gap | S01 | Agents follow empty/forbidden directories; architectural imports drift silently | Make Sprint 01 package graph and CI checks executable; label future-state prose accurately |
| 10 | Billing completion/settlement handoff | Approaches later billing sprint, design affects job model earlier | Lost revenue or repeated provider expense after worker crash | Durable attempt/result + transactional completion/outbox contract |
| 11 | 50k editor/3D performance | S13–S20 | Whole-level load, hit testing, topology, or draw calls miss latency budgets | Representative benchmark models at 1k/5k/50k; keep 50k support **UNVERIFIED** until measured |
| 12 | Scope overload | All S01–S26 | One developer spends effort on packages/infrastructure without a consumer | Continue ADR-012 gates; do not create real `ai-tools`, workers, WASM, Redis, Temporal, or BIM runtime early |

### Solo-developer conclusion

Sprint 26b can be a **sellable narrow product** only if “sellable” means a tightly
bounded geometry set, a named and audited subset of QS measurement, traceable cost,
a stable PDF artifact, tenant safety, and a minimal entitlement/payment path. It is
not yet supportable as a broad “professional AEC platform” milestone for arbitrary
buildings. The largest estimation risk is room/junction topology, followed closely
by reconciliation/undo; both hide combinatorial state rather than ordinary CRUD.

Nothing in the currently declared MVP package set should be removed from the
long-term product graph. However, `model-session` has no real consumer until the
editor, `engine-3d` has no consumer until 3D, and cost/QS packages have no consumer
until their phases. ADR-012's create-on-consumer rule should be applied literally;
empty future directories add no value.

---

# 7. VERDICT

## SAFE TO IMPLEMENT WITH LISTED CORRECTIONS

The foundational direction is substantially sound: one canonical Building Model,
semantic commands, renderer-neutral working state, modular monolith, deterministic
QS/cost engines, ledger billing, tenant scoping plus RLS, conservative graphics,
and IFC as interchange are the right choices for this product and team size.

The baseline is **not safe to implement blindly as “complete.”** Three corrections
are hard gates before their owning work ships:

1. deterministic room/junction and room-identity semantics before Room persistence;
2. deterministic queue reconciliation/undo/replay and required idempotency before
   model-session persistence;
3. a realizable hybrid QS rules contract plus complete cost lineage before any
   NRM2-branded quantity or issued estimate/report.

Sprint 01 may proceed after the cheap repository-shape correction. Sprints should
not reinterpret the architecture wholesale; they should close the listed contracts
at the stated gates. If the team refuses those corrections while retaining claims
of professional quantities and reproducibility, the verdict becomes **NOT SAFE TO
IMPLEMENT** for Sprints 15 and 23–26.

---

# Sources

[^openai-sol]: [OpenAI — GPT-5.6 Sol model and pricing](https://developers.openai.com/api/docs/models/gpt-5.6-sol)
[^rics-nrm]: [RICS — New Rules of Measurement](https://www.rics.org/profession-standards/rics-standards-and-guidance/sector-standards/construction-standards/nrm)
[^nrm2-pdf]: [RICS — NRM 2: Detailed measurement for building works (official PDF)](https://www.rics.org/content/dam/ricsglobal/documents/standards/october_2021_nrm_2.pdf)
[^pixi]: [PixiJS — official releases](https://github.com/pixijs/pixijs/releases)
[^three]: [Three.js — official releases](https://github.com/mrdoob/three.js/releases)
[^web-ifc]: [That Open Engine web-ifc — official documentation](https://thatopen.github.io/engine_web-ifc/docs/)
[^that-open]: [That Open — official documentation](https://docs.thatopen.com/)
[^ifcopenshell]: [IfcOpenShell — official releases](https://github.com/IfcOpenShell/IfcOpenShell/releases)
[^drizzle]: [Drizzle ORM — official releases](https://github.com/drizzle-team/drizzle-orm/releases)
[^temporal]: [Temporal TypeScript SDK — official releases](https://github.com/temporalio/sdk-typescript/releases)
[^neon]: [Neon — official documentation](https://neon.com/docs/introduction)
[^clerk]: [Clerk — official documentation](https://clerk.com/docs)
[^pgvector]: [pgvector — official releases](https://github.com/pgvector/pgvector/releases)
[^r2]: [Cloudflare R2 — official pricing](https://developers.cloudflare.com/r2/pricing/)
[^fargate]: [AWS Fargate — official pricing](https://aws.amazon.com/fargate/pricing/)
[^alb]: [AWS Elastic Load Balancing — official pricing](https://aws.amazon.com/elasticloadbalancing/pricing/)
[^vpc]: [AWS VPC/NAT Gateway — official pricing](https://aws.amazon.com/vpc/pricing/)
[^neon-pricing]: [Neon — official pricing](https://neon.com/pricing)
[^pg-set]: [PostgreSQL — `SET` / `SET LOCAL`](https://www.postgresql.org/docs/current/sql-set.html)
[^pgbouncer]: [PgBouncer — pooling feature compatibility](https://www.pgbouncer.org/features.html)
