# ADR-021 — Two Customer Lifecycles on One Platform

> **Status:** Accepted
> **Date:** 2026-09-12
> **Deciders:** Product Owner / CTO
> **Supersedes:** none
> **Superseded by:** none
> **Extends:** ADR-014 (billing ledger), ADR-016 (tenancy)

---

## Context

Buildora AI serves two customer groups whose **lifecycles differ fundamentally**:

| | Professional | Project-lifecycle |
|---|---|---|
| Who | Architects, QSs, estimators, contractors, developers, PMs | Homeowners, self-builders, renovators |
| Usage | Daily/weekly, multi-project, indefinite | Intense for 6–18 months, then naturally ends |
| Revenue | Recurring subscription (MRR/ARR) | Fixed-term project plan |
| Success | Renews for years | **Completes the project and stops paying** |

The second row is the one that has architectural consequences. A homeowner who
finishes their house and cancels is a **successful customer**, not churn. Most SaaS
architecture assumes indefinite recurring subscription and treats cancellation as
failure — which would push us toward dark patterns, or toward deleting a completed
project the customer still values.

Three questions must be answered before billing and tenancy are implemented,
because all three are expensive to retrofit:

1. Can a plan be **fixed-term** rather than indefinitely recurring?
2. What happens to a project when the paid term ends?
3. Does a solo user need an organization, and can they later become one?

## Decision

### 1. Plans may be recurring **or** fixed-term

`billing_subscriptions` gains an explicit **term model**:

```text
term_model        RECURRING | FIXED_TERM
term_ends_at      nullable — set for FIXED_TERM
auto_renew        boolean  — false for a project pass
```

A **project pass** (6, 12 or 18 months) is a first-class product, not a
subscription that someone remembers to cancel. It grants entitlements for a
defined window and then ends — by design, without a cancellation event.

Everything else in ADR-014 is unchanged: the ledger, reservations, the webhook
inbox, exact decimal money and server-confirmed grants apply identically.

### 2. Archive mode is a first-class project state

When a paid term ends, the project moves to **archive**, not deletion:

```text
project.status:  draft → active → archived → deleting → deleted
                                      ▲
                       paid term ends here, NOT at `deleting`
```

Archive mode is defined by **entitlements, not by data removal**:

| In archive mode | Allowed |
|---|---|
| Sign in, open the project | ✓ |
| View 2D, 3D, walk through | ✓ |
| View previously generated reports and exports | ✓ |
| View project history and versions | ✓ |
| Export own data per policy | ✓ |
| **Edit the model** | ✗ |
| **Run AI** | ✗ |
| **Recalculate cost / generate new exports** | ✗ |

The project's data is **preserved intact**. Reactivation restores full capability
against the existing model — the user resumes rather than restarts.

This is not generosity. A homeowner who extends, renovates or builds again in
three years is a reactivation, and a preserved project is the reason they come back.

### 3. Every user has an organization — always

There is **no separate "personal" ownership path**. A solo user gets a personal
organization created automatically at signup, of which they are the sole owner.

```text
signup → user + personal organization (auto) → projects belong to that org
```

This is the single most important decision in this ADR, and it is made for a
structural reason: **ADR-016 makes organization the tenant boundary.** A parallel
"projects owned directly by a user" path would mean every tenant-scoped query,
every RLS policy and every permission check needs a second code path — and the
second path is where the cross-tenant leak eventually happens.

Because ownership never changes shape, a homeowner becoming a developer, or a
freelance architect forming a practice, is a **rename and an invitation** — not a
data migration.

### 4. Entitlements are capabilities, never plan names

Already correct in the billing architecture; this ADR makes it binding across
both lifecycles.

```text
✓ feature.qs.advanced        ✓ limit.active_projects
✓ feature.boq.edit           ✓ limit.ai_credits
✓ feature.ratebooks.manage   ✓ feature.home.furnishing
✗ if (plan === "Pro")
```

Archive mode is then simply an entitlement set with editing capabilities absent —
requiring no bespoke "archive" branch in feature code.

### 5. One engine, two experiences

**The QS, BOQ, cost and Building Model engines are never duplicated by segment.**
A homeowner's estimate and a quantity surveyor's estimate come from the same
deterministic engines and the same pinned lineage (ADR-015).

Segments differ only in:

```text
UX and navigation · onboarding · entitlements · reporting depth
terminology · workflow guidance
```

A "simple" home cost view is a **presentation** of the same authoritative
calculation, never a second, looser calculation. Two engines would mean two sets
of golden tests, two sets of bugs, and eventually two different answers to the
same question.

## Alternatives considered

| Option | Pros | Cons | Why not chosen |
|---|---|---|---|
| A — Recurring subscriptions only | Simplest billing | Forces homeowners into indefinite subscription they do not want; invites dark patterns; loses reactivation | Misreads a successful outcome as churn |
| B — Separate Home and Professional products | Each UX optimal | Two codebases, two engines, two sets of QS bugs; no cross-segment project collaboration | Duplicate authoritative logic — violates ADR-001 |
| C — Delete projects on cancellation | No storage cost | Destroys reactivation, referral and goodwill; hostile | Bad product, negligible saving |
| D — Personal ownership separate from organizations | Feels natural for solo users | A second tenancy path through every query and RLS policy | The likeliest source of a cross-tenant breach |
| E — One platform, two lifecycles, universal org (chosen) | One engine; one tenancy path; both revenue models; reactivation preserved | Archive entitlement set must be defined; storage retained for archived projects | — |

## Consequences

**Positive**

- A homeowner can complete and stop paying without losing their project — and can
  return years later.
- Cross-segment collaboration works natively: a homeowner invites an architect,
  who becomes a professional customer. Both are ordinary organization members.
- One set of engines, one set of golden tests, one answer to any cost question.
- Tenancy has exactly one shape, so RLS and permission checks have one code path.
- Solo → team growth needs no data migration.

**Negative / accepted cost**

- Storage is retained for archived projects indefinitely (small — parametric
  geometry, not meshes; large artifacts can lifecycle to cold storage).
- The archive entitlement set must be defined and tested as carefully as a paid tier.
- "Personal organization" is mild conceptual overhead for a single user, and the
  UI should not make a solo user think about organizations at all.

**Neutral**

- Exact pricing, term lengths and plan boundaries remain commercial decisions,
  deliberately not fixed here.

## Migration / implementation impact

No migration — decided before billing or projects are implemented.

Affects: `15_Database_and_Data_Architecture.md` (subscription term fields,
project lifecycle), `24_Billing_AI_Credits_and_Usage.md` (fixed-term plans,
archive entitlements), `12_User_Segments_and_Commercial_Model.md`, planning docs.

Owned by: Sprint 05 (personal organization at signup), Sprint 36 (fixed-term
plans and archive entitlements).

**Sprint 05 is the binding one.** Auto-creating a personal organization costs
almost nothing then; adding a second ownership path later, or migrating
user-owned projects into organizations, is expensive and risky.

## Rollback / exit strategy

- Fixed-term plans: reversible — stop selling them.
- Archive mode: reversible in policy, though removing it after customers rely on
  it would be a trust failure.
- **Universal organization: effectively irreversible** once projects exist. This
  is why it is decided now.

## Verification

- Test: a fixed-term subscription expires without a cancellation event and moves
  the project to `archived`.
- Test: an archived project can be **viewed** (2D, 3D, reports, history) and
  **cannot** be edited, AI-processed or re-costed.
- Test: reactivation restores full capability against the **existing** model —
  no data was lost.
- Test: signup creates exactly one personal organization with the user as owner.
- Test: a project always resolves to exactly one organization; there is no
  user-owned project path.
- Test: a home user's cost estimate and a professional's, for the same model and
  pinned versions, produce **identical** figures.
- Review: any `if (plan === "...")` in feature code is a HIGH finding — use
  entitlements.
- Review: any segment-specific QS or cost engine is a BLOCKER.

## References

- ADR-001 (one canonical model / one engine), ADR-014 (billing ledger),
  ADR-015 (cost lineage), ADR-016 (organization as tenant boundary)
- `docs/architecture/12_User_Segments_and_Commercial_Model.md`
- `docs/architecture/24_Billing_AI_Credits_and_Usage.md`
