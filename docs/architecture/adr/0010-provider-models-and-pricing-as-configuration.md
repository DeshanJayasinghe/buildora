# ADR-010 — Provider Models and Pricing as Versioned Configuration

> **Status:** Accepted
> **Date:** 2026-09-12
> **Deciders:** Product Owner / CTO
> **Supersedes:** none
> **Superseded by:** none

---

## Context

Buildora AI's unit economics depend on AI provider pricing. Providers change
models, rates, caching behaviour and promotional terms on their own schedule.

An earlier architecture review incorrectly asserted that the OpenAI models and
prices recorded in the finance documents were fabricated. **That assertion was
wrong and has been withdrawn.** The pricing was verified against official OpenAI
documentation on 2026-09-12 and found accurate in every respect.

The real architectural question is not whether today's numbers are right. It is
where provider identity and pricing are allowed to live, so that tomorrow's
numbers can change without touching domain code or customer-facing plans.

## Decision

### Provider models and pricing are versioned configuration data

Rate cards are stored as data with an explicit shape:

```text
provider_rate_cards
- provider                 e.g. "openai"
- model_id                 e.g. "gpt-5.6-terra"
- input_rate               per 1M tokens
- cached_input_rate        per 1M tokens
- cache_write_rate         per 1M tokens, where applicable
- output_rate              per 1M tokens
- effective_from
- effective_to             nullable
- verified_at              when a human last checked this against the provider
- source_reference         URL / document consulted
```

Rates are `NUMERIC`. Every recorded provider call resolves its cost against the
rate card effective at the time of the call, so historical cost remains
reproducible when prices change.

### Domain code uses logical aliases only

```text
FAST
BALANCED
ADVANCED
EXPERT
```

**Provider model identifiers must never appear in domain or business logic.**
Routing, escalation, entitlements and credit weighting are expressed in terms of
aliases. The alias-to-model mapping is configuration.

This is what makes a provider price change, a model deprecation, or a provider
switch a configuration update rather than a code change or a re-pricing crisis.

### Customer-facing units stay decoupled from provider units

Customers see **AI Design Credits** and **Render Credits** (ADR-014). They never
see tokens. Internal provider cost and customer credit consumption are separate
ledgers that happen to be correlated — never the same number.

### Verified pricing as of 2026-09-12

Confirmed against official OpenAI API documentation on 2026-09-12:

| Model | Input / 1M | Cached input / 1M | Output / 1M |
|---|---:|---:|---:|
| `gpt-5.6-luna` | $0.20 | $0.02 | $1.20 |
| `gpt-5.6-terra` | $2.00 | $0.20 | $12.00 |
| `gpt-5.6-sol` | $4.00 | $0.40 | $20.00 |
| `gpt-6-astra` | $10.00 | $1.00 | $50.00 |

Also confirmed: Batch/Flex processing at approximately 50% of standard rates;
Fast mode at approximately 2× standard rates.

**`gpt-5.6-sol` pricing is promotional, stated as available at least through
21 November 2026.** Long-term Pro-plan economics must not assume that promotional
rate persists. Budget a safety margin, or route normal workloads to Luna/Terra.

These figures belong in the rate-card table with `verified_at = 2026-09-12`. The
table above is a snapshot for human reference, not the runtime source.

### Standing rule

**Provider prices are configuration data carrying a verification date — never
prose in an architecture document.** Any document quoting a rate must state when
it was verified and defer to the rate card as authoritative.

## Alternatives considered

| Option | Pros | Cons | Why not chosen |
|---|---|---|---|
| A — Model IDs and rates inline in code | Direct | Every price change is a deployment; historical cost not reproducible; provider lock-in | Fails reproducibility and agility |
| B — Rates in config, model IDs in code | Partial | Domain logic still names providers; escalation and routing become provider-specific | Half measure |
| C — Both as versioned configuration behind aliases (chosen) | Price changes are data; provider swap is config; historical cost reproducible | Requires an alias-resolution layer and rate-card discipline | — |

## Consequences

**Positive**

- Provider price changes never touch domain code or customer plan language.
- Historical AI cost is reproducible against the rate card in force at the time.
- Provider substitution or multi-provider routing is a configuration exercise.
- Margin analytics can be computed accurately per call, per job, per organization.

**Negative / accepted cost**

- Rate cards must actually be kept current; `verified_at` makes staleness visible
  but does not prevent it.
- An alias-resolution layer is indirection that must be documented well enough
  that agents do not bypass it.

**Neutral**

- Promotional pricing is representable via `effective_to`.

## Migration / implementation impact

No migration.

Corrections applied as part of this decision:

- The "fabricated pricing" claim is removed from
  `FINAL_ARCHITECTURE_REVIEW.md` and `OPEN_ARCHITECTURE_DECISIONS.md`.
- The former P0-5 recommendation is withdrawn and replaced by this
  configuration principle.
- Finance documents retain their figures, annotated with verification date.

Affects: `docs/architecture/24_Billing_AI_Credits_and_Usage.md`,
`docs/finance/Buildora_AI_Development_Production_Costing_Plan.md`,
`docs/finance/Buildora_AI_Return_On_Investment_Plan.md`,
AI Gateway implementation.

Owned by: Phase 12 / Sprint 27 (AI Gateway & model routing).

## Rollback / exit strategy

Fully reversible — this is a structural discipline, not a data commitment.

## Verification

- Review: any provider model identifier in domain or business logic is a BLOCKER.
- Test: a provider call resolves cost against the rate card effective at call time.
- Test: changing a rate card does not alter previously recorded call costs.
- Operational: a periodic check that `verified_at` on active rate cards is not stale.

## References

- OpenAI API pricing documentation, consulted 2026-09-12
- `AGENTS.md` §18, §44
- `docs/architecture/24_Billing_AI_Credits_and_Usage.md` §25–§29
- ADR-013, ADR-014
