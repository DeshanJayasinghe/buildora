# ADR-009 — Versioned QS Measurement Rulesets with NRM2 as Initial Default

> **Status:** Accepted
> **Date:** 2026-09-12
> **Deciders:** Product Owner / CTO
> **Supersedes:** none
> **Superseded by:** none

---

## Context

No document named the measurement standard governing deduction and measurement
rules — whether openings below a threshold are deducted, how wall areas are
measured at junctions, how work is itemised.

Professional quantity surveyors reject quantities that do not follow a
recognised standard. The choice shapes the default ruleset and its golden tests.

The product documentation is UK-oriented (GBP examples, UK Stripe pricing
references, £49 plan). No canonical document establishes a different initial
market.

## Decision

### Rulesets are versioned data, not code

Measurement rules live in **`qs_rule_sets`** as versioned records:

```text
qs_rule_sets
- id
- code                  e.g. "NRM2"
- version
- status
- effective_from
- rules_hash
- metadata_jsonb
- created_at
```

Every `quantity_runs` record pins the `ruleset_version` it used, alongside
`model_version`, `assembly_catalog_version` and `calculation_engine_version`
(ADR-015).

**Measurement rules must not be hard-coded throughout calculation code.** The QS
engine interprets a ruleset; it does not embed one.

### Initial default: NRM2

The initial ruleset is **NRM2** (RICS New Rules of Measurement 2 — Detailed
Measurement for Building Works), seeded as a versioned `qs_rule_sets` record
with its deduction rules documented explicitly.

This reflects the UK-oriented product documentation. If the launch market
changes, substitute the local standard — **the architecture is unaffected**,
which is the point of making rulesets data.

### The engine must permit alternative rulesets later

SMM7, POMI, regional standards and customer-specific rules are all expressible
as additional `qs_rule_sets` records. No code change should be required to add
one, beyond implementing any genuinely novel rule primitive it needs.

### Golden tests are pinned to a ruleset version

The canonical net wall area test and all measurement fixtures state which
ruleset version they assert against. A ruleset change that alters a golden
result requires a new ruleset version, not an edited test.

## Alternatives considered

| Option | Pros | Cons | Why not chosen |
|---|---|---|---|
| A — Hard-coded rules | Simplest to start | Cannot support a second market without a rewrite; historical runs not reproducible | Fails reproducibility (ADR-015) |
| B — Versioned rulesets, NRM2 default (chosen) | Professional credibility; multi-market ready; historical reproducibility | Ruleset interpretation layer is real work | — |
| C — Versioned rulesets, no default named | Maximum flexibility | Leaves the MVP with no defensible default; every quantity is "which standard?" | Indecision is not a decision |
| D — Buildora-specific ruleset | Full control | No professional recognition; QS users cannot validate against a known standard | Undermines the target market's trust |

## Consequences

**Positive**

- Quantities are defensible to professional users against a named standard.
- Historical quantity runs remain reproducible under the ruleset that produced them.
- A second market is a data exercise, not a rewrite.

**Negative / accepted cost**

- A ruleset interpretation layer is more work than inline rules.
- NRM2's deduction rules must be researched and encoded correctly — this is
  domain work, not just engineering.

**Neutral**

- The default can be changed before launch at no architectural cost.

## Migration / implementation impact

No migration. Seed data plus engine design.

Affects: `docs/architecture/12_QS_Engine.md` (written when Phase 10 begins),
`15_Database_and_Data_Architecture.md` §58–§61.

Owned by: Phase 10 / Sprint 23 (Quantity Surveying engine).

## Rollback / exit strategy

Changing the *default ruleset* before launch is free. Changing the *rules within
a published version* after quantities have been issued to customers is not —
issue a new ruleset version instead.

## Verification

- Golden test: 5000 × 2700 mm wall less a 1000 × 2100 mm opening — gross 13.5 m²,
  deduction 2.1 m², net 11.4 m² (`AGENTS.md` §20), pinned to ruleset version.
- Golden test: room area from interior faces per ADR-006.
- Test: a `quantity_runs` record pins its `ruleset_version`.
- Test: re-running a historical quantity run with its pinned ruleset reproduces
  identical results.
- Review: any measurement constant hard-coded in engine code rather than read
  from a ruleset is a HIGH finding.

## References

- `AGENTS.md` §19, §20
- `docs/architecture/15_Database_and_Data_Architecture.md` §58–§61
- OD-07 in `docs/architecture/OPEN_ARCHITECTURE_DECISIONS.md`
- ADR-006, ADR-015
