# ADR-022 — Runtime Feature Gating and the Admin-Managed Feature Catalogue

> **Status:** Accepted
> **Date:** 2026-09-13
> **Deciders:** Product Owner / CTO
> **Supersedes:** none
> **Superseded by:** none
> **Extends:** ADR-021 (entitlements are capabilities, never plan names)

---

## Context

ADR-021 established that feature code reads **entitlements**, never plan names.
`feature_definitions`, `plan_entitlements`, `entitlement_snapshots` and admin
overrides all exist in the billing architecture.

What is **not** specified is the operational reality the product owner needs:

> *"A free user cannot use walkthrough; a Pro user can. I want to change that
> from the admin panel without a deployment."*

Three gaps prevent that today:

1. **The feature catalogue is a code-level list.** `feature_definitions` exists
   as a table, but nothing says whether a new feature code can be created at
   runtime, or who owns the mapping from a code to the UI surface it gates.
2. **There is no declared enforcement contract.** Nothing states that a gated
   capability must be checked server-side, or what the client is allowed to do
   with an entitlement response. Without that rule, "the button is hidden"
   becomes the enforcement — which is not enforcement.
3. **Changing a live plan's entitlements has no defined semantics.** Does
   removing `feature.walkthrough` from Pro affect existing subscribers
   immediately, at renewal, or not at all? ADR-021's grandfathering rule implies
   plan *versions* are immutable, but the admin's day-to-day need is to adjust a
   plan without minting a new version for every tweak.

Interior features make this concrete and urgent: walkthrough, furnishing, finish
editing and AI interior assistance (ADR-019, ADR-020) are exactly the capabilities
that differ by tier, and they are the first features where the product owner will
want to move the line without shipping code.

## Decision

### 1. The feature catalogue is data, and admin-managed

`feature_definitions` is a **runtime-editable catalogue**. A platform admin can
register a new feature code, and set its default, without a deployment.

```text
feature_definitions
-------------------
id
code                      e.g. feature.walkthrough
name                      human label for the admin UI
description               what this gates, in plain language
category                  DESIGN | INTERIOR | QS | AI | EXPORT | COLLAB | PLATFORM
value_type                boolean | integer | decimal | string | unlimited
default_value             applied when no plan entitlement exists
is_active                 retired codes stop gating without being deleted
created_at / updated_at
```

**Registering a code does not make it enforce anything.** A code gates a
capability only when application code calls the entitlement service with it. The
catalogue is the admin's vocabulary; the call sites are the enforcement.

### 2. Feature codes are owned by the code that checks them

A feature code has exactly **one** owning check site in the application, recorded
alongside the definition:

```text
feature.walkthrough      → apps/api  3D session authorization
feature.home.furnishing  → apps/api  PLACE_FURNITURE command guard
feature.qs.advanced      → apps/api  quantity run entry point
```

This prevents the failure mode where an admin creates `feature.walkthrough_v2`,
sets it on a plan, and nothing happens because no code reads it. The admin UI
shows **which codes are wired and which are orphaned**, and an orphaned code is
displayed as inactive rather than silently ignored.

### 3. Enforcement is server-side; the client is advisory only

```text
Server   decides. Every gated capability is checked in apps/api before the
         work happens. A denied capability returns
         FEATURE_NOT_ENTITLED (402/403) with the feature code.

Client   reflects. The UI reads the same entitlements to hide, disable or
         badge the capability — for usability, never for security.
```

**Hiding a button is not access control.** A user who calls the API directly must
be refused by the server. Every gated capability therefore has a server check
site, and that check site is what the feature code is bound to.

For long-running or metered work, the entitlement check happens **before the
credit reservation** (ADR-014), so an unentitled request never consumes credit.

### 4. Live entitlement changes: overrides, not silent plan mutation

Two distinct operations, with different semantics:

| Operation | Applies to | When it takes effect | Grandfathering |
|---|---|---|---|
| **Publish a new plan version** | New subscribers, and existing ones on migration | At subscribe or explicit migration | Existing subscribers **keep their version** (ADR-021) |
| **Entitlement override** | One organization | **Immediately** | Not applicable — it is per-tenant |

For the common case — *"give this customer walkthrough access"* — the admin
applies an **override**, which is immediate, audited and reversible.

For *"Pro now includes walkthrough for everyone"* — the admin publishes a new
plan version. Existing subscribers stay on their version until migrated, which is
the grandfathering guarantee. The admin UI must make that consequence explicit at
the point of publishing, because the intuitive expectation is "it changes now".

```text
plan_entitlement_overrides
--------------------------
id
organization_id
feature_code
value
reason                    required — appears in the audit trail
effective_from
effective_to              nullable; NULL = until revoked
created_by_admin_id
created_at
```

### 5. Effective entitlement resolution order

```text
1. enterprise contract override      highest precedence
2. organization override             admin-applied, immediate
3. add-on / promotion                purchased or granted
4. plan version entitlement          from the subscribed plan version
5. feature_definitions.default_value lowest precedence
```

Archive mode (ADR-021) applies as an **entitlement set at layer 2** — a read-only
capability profile — so no feature needs an "is archived" branch.

Resolution is **cached, with explicit invalidation** on subscription change,
override change, plan migration and period rollover. It must never query Stripe
on a feature check.

### 6. Snapshots explain the past; they do not decide the present

`entitlement_snapshots` records what an organization was entitled to during a
billing period, so a support question — *"why could they use this last month?"* —
is answerable.

**A snapshot is evidence, never the authority for a live check.** Live checks
resolve fresh through layers 1–5.

### 7. Credits are metered, not gated

Two different mechanisms that must not be conflated:

```text
feature.*    CAN they do this at all?     boolean or limit → entitlement
limit.*      HOW MUCH can they do?        counted → entitlement
credits      METERED consumption          wallet + reservation (ADR-014)
```

`feature.ai.interior` says whether AI furnishing is available. AI Design Credits
say how much of it they can consume. A user may be entitled to a feature and
still be out of credit — those are different errors with different UX
(`FEATURE_NOT_ENTITLED` vs `INSUFFICIENT_CREDITS`).

## Alternatives considered

| Option | Pros | Cons | Why not chosen |
|---|---|---|---|
| A — Hard-coded plan checks (`if plan === "Pro"`) | Trivial | Every commercial change is a deployment; unusable for a solo operator; forbidden by ADR-021 | Already rejected |
| B — Feature flags reused as entitlements | One system | Flags are for **rollout**, entitlements are for **commercial rights**. Conflating them means a rollout flag can grant paid access, and a billing change can break a canary | Different lifecycles, different audiences, different audit needs |
| C — Entitlements editable only by deployment | Reviewable in git | Defeats the purpose — the product owner cannot adjust packaging without an engineer | Fails the requirement |
| D — Admin-managed catalogue + overrides + versioned plans (chosen) | Immediate per-tenant change; safe global change via versioning; grandfathering preserved; fully audited | Two operations the admin must understand; orphaned codes need surfacing | — |

### On feature flags vs entitlements

They are **separate systems** and must stay separate:

```text
feature_flags      "is this code path enabled?"     rollout, kill switches, canary
                                                    audience: engineering
feature_definitions "is this customer allowed?"     commercial rights
                    + plan_entitlements             audience: product / finance
```

A capability can be flag-disabled for everyone *and* entitled to Pro — the flag
wins, because an unfinished feature must not be sold. Check order is **flag
first, entitlement second**.

## Consequences

**Positive**

- The product owner can move a capability between tiers without a deployment:
  publish a plan version for everyone, or apply an override for one customer.
- Trials, promotions, enterprise deals and goodwill grants are all overrides —
  no bespoke code per commercial arrangement.
- Archive mode, fixed-term expiry and grandfathering all reduce to entitlement
  sets, requiring no branches in feature code.
- Every entitlement change is attributable and reversible.

**Negative / accepted cost**

- The admin must understand that publishing a plan version does **not** change
  existing subscribers. The UI must state this at the point of action, or the
  admin will believe a change took effect when it did not.
- Orphaned feature codes are possible; the admin UI must surface them.
- Entitlement resolution is cached, so invalidation correctness matters — a stale
  cache either sells access that was revoked or blocks access that was granted.

**Neutral**

- The catalogue starts small. Only capabilities that genuinely differ by tier
  need a code; gating everything produces unusable complexity.

## Migration / implementation impact

No migration — decided before billing is implemented.

Affects: `24_Billing_AI_Credits_and_Usage.md` (catalogue, overrides, resolution
order, admin UI), `15_Database_and_Data_Architecture.md`
(`plan_entitlement_overrides`, `feature_definitions` columns),
`12_User_Segments_and_Commercial_Model.md`, both planning documents.

Owned by: **Sprint 07** (feature-flag/admin baseline — the *distinction* between
flags and entitlements must land here) and **Sprint 36** (entitlement catalogue,
overrides, admin UI, enforcement contract).

Interior feature codes (`feature.walkthrough`, `feature.home.furnishing`,
`feature.interior.finishes`, `feature.ai.interior`) are registered when their
sprints ship — 21b, 26c and 30b respectively.

## Rollback / exit strategy

Fully reversible. The catalogue is data; removing a code stops it gating. The
enforcement contract is a discipline, not a schema commitment.

## Verification

- Test: a capability denied by entitlement returns `FEATURE_NOT_ENTITLED` from
  the **API**, not merely a hidden button.
- Test: a direct API call bypassing the UI is refused for an unentitled org.
- Test: an override grants access **immediately**, without a deployment or a
  subscription change.
- Test: publishing a new plan version does **not** alter existing subscribers'
  entitlements.
- Test: entitlement check precedes credit reservation — an unentitled request
  consumes no credit.
- Test: a feature flag disabled globally overrides an entitlement that grants it.
- Test: every privileged entitlement action writes an audit event with a reason.
- Review: any `if (plan === "...")` in feature code is a **BLOCKER**.
- Review: any gated capability with no server-side check site is a **BLOCKER**.
- Operational: the admin UI lists orphaned feature codes (registered, never checked).

## References

- ADR-014 (billing ledger, reservations) · ADR-021 (entitlements, two lifecycles)
- ADR-019 / ADR-020 (the interior capabilities this first gates)
- `docs/architecture/24_Billing_AI_Credits_and_Usage.md` §13–§16, §100–§103
- `docs/architecture/12_User_Segments_and_Commercial_Model.md` §7
