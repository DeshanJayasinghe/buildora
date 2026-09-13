# ADR-014 — Billing Ledger and Credit Reservation Architecture

> **Status:** Accepted
> **Date:** 2026-09-12
> **Deciders:** Product Owner / CTO
> **Supersedes:** none
> **Superseded by:** none

---

## Context

Buildora AI sells subscriptions and meters expensive AI and rendering work. The
failure modes in this area are financial rather than technical: double-charging
a customer, granting credits that were never paid for, losing revenue to
untracked provider cost, or being unable to explain a balance to a disputing
customer.

A mutable `organization.ai_credits = 27` column cannot answer "why is it 27?",
cannot survive concurrent consumption safely, and cannot be reconciled.

## Decision

### Ledger, not balance

Credit state is an **append-only ledger**. A balance is a derived read model,
never the source of truth.

Ledger operations:

```text
GRANT      RESERVE     RELEASE     CONSUME
REFUND     REVOKE      ADJUSTMENT  EXPIRE
```

Every credit mutation is idempotent and auditable, carrying its cause
(subscription grant, top-up order, AI job, admin adjustment).

### Reserve → settle → release

Expensive work reserves credits **before** it starts:

```text
reserve (with TTL, idempotency key)
  → perform work
  → settle: CONSUME actual, RELEASE remainder
  or
  → failure/cancel: RELEASE full reservation
```

This prevents negative balances under concurrency and prevents a failed job from
charging the customer.

### Customer units are decoupled from provider units

| Customer-facing | Internal |
|---|---|
| AI Design Credits | input / cached / output / reasoning tokens |
| Render Credits | GPU seconds, image generation units |
| Plan limits, seats, storage | provider cost in currency |

One customer-visible AI action may involve several provider calls, including
escalation. **The customer is charged at job/action level; internal cost is the
sum of provider usage.** Retries and escalation never double-charge (ADR-010).

### Sources of truth

- **Stripe** is authoritative for payment facts: payment status, invoices,
  subscription objects, refunds, disputes.
- **Buildora AI** is authoritative for: plan catalog and versions, entitlements,
  credit wallets, usage, feature access, AI/render accounting.

**Credits are granted only on server-side payment confirmation.** A browser
reaching a success URL is never proof of payment.

### Webhook inbox

```text
verify signature
  → store unique event record (inbox)
  → acknowledge
  → process idempotently
```

Duplicate delivery is safe by construction. Out-of-order delivery is handled by
reconciling against authoritative provider state rather than assuming ordering.

### Exact arithmetic

All money, rates and credit quantities use `NUMERIC` in the database and
decimal-safe arithmetic in code. **Floating point is forbidden for authoritative
financial values** (`AGENTS.md` §21).

### Entitlements

Feature access is evaluated server-side from plan entitlements. The frontend
never unlocks a feature; it only reflects what the server already permits.

## Alternatives considered

| Option | Pros | Cons | Why not chosen |
|---|---|---|---|
| A — Mutable balance column | Trivial | No history, no dispute resolution, unsafe under concurrency, unreconcilable | Cannot run a business on it |
| B — Ledger without reservations | Full history | Concurrent expensive jobs can overdraw; failed jobs charge customers | Revenue and trust risk |
| C — Ledger + reservations (chosen) | Safe under concurrency; failed work does not charge; fully auditable | More moving parts | — |
| D — Charge raw tokens to customers | Perfectly aligned with cost | Unpredictable bills; provider price changes hit customers directly; poor product | Bad commercial design |

## Consequences

**Positive**

- Every balance is explainable from its ledger history.
- Concurrent consumption cannot drive a wallet negative.
- Provider price changes are absorbed internally (ADR-010).
- Duplicate webhooks, retries and escalation are all financially safe.

**Negative / accepted cost**

- More tables and more code than a balance column.
- Reservation TTL expiry needs a background job.
- Reconciliation jobs are required to catch drift against Stripe.

**Neutral**

- Billing delivery is split across two releases (see planning), because
  compressing the full scope into one sprint risks under-tested financial code.

## Migration / implementation impact

No migration.

Affects: `docs/architecture/24_Billing_AI_Credits_and_Usage.md` (already
specifies this design in detail), `apps/api` billing module.

Owned by: Phase 17 / Sprint 36 (Billing Release 1) and 36b (Release 2).

## Rollback / exit strategy

**Effectively irreversible** — ledger history cannot be reconstructed from a
mutable balance retroactively.

## Verification

- Test: parallel reservations against a wallet with insufficient balance never
  drive it negative.
- Test: the same Stripe event delivered twice grants credits exactly once.
- Test: a failed AI job releases its full reservation.
- Test: an escalated AI job charges the customer once while recording multiple
  provider calls.
- Test: balance derived from the ledger equals the read model.
- Review: any authoritative money value stored or computed as a float is a BLOCKER.
- Review: any credit grant triggered by a client-side signal is a BLOCKER.

## References

- `AGENTS.md` §21, §44, §45, §46, §47, §48
- `CLAUDE.md` §27, §28
- `docs/architecture/24_Billing_AI_Credits_and_Usage.md`
- ADR-010, ADR-015
