# 12 — User Segments, Revenue Model & Retention Strategy

> **Status:** Accepted — 2026-09-12
> **Owns:** customer segmentation, the two commercial lifecycles, retention
> strategy, and the architectural obligations they create.
> **Governed by:** ADR-014 (billing ledger), ADR-015 (cost lineage),
> ADR-016 (tenancy), **ADR-021 (two lifecycles)**
> **Nature:** Product and commercial strategy. Pricing is **not** fixed here.

---

# 1. The Core Business Principle

Buildora AI serves two customer groups with **fundamentally different lifecycles**:

```text
PROFESSIONAL                      PROJECT-LIFECYCLE
architects · QSs · estimators     homeowners · self-builders
contractors · developers · PMs    renovators

daily/weekly, multi-project       intense for 6–18 months
indefinite                        then naturally ends

recurring subscription            fixed-term project plan
MRR / ARR                         project revenue + referrals

SUCCESS = renews for years        SUCCESS = completes the project
                                            and stops paying
```

**A homeowner who finishes their house and cancels is a successful customer, not
churn.** Architecture and pricing must not treat the two lifecycles identically.

---

# 2. Why This Is in the Architecture Docs

Most of this document is commercial strategy. Four parts are **architectural
obligations** — decisions that are expensive or impossible to retrofit:

| Obligation | Why it is architectural | ADR |
|---|---|---|
| Plans may be **fixed-term**, not only recurring | Billing schema assumption | ADR-021 |
| **Archive mode** is a project lifecycle state | Project state machine + entitlements | ADR-021 |
| **Every user has an organization**, always | Tenancy has one shape (ADR-016) | ADR-021 |
| **One engine, two experiences** | Duplicate engines violate ADR-001 | ADR-021 |

Everything else below is strategy that guides the roadmap without binding the schema.

---

# 3. Professional Segment

## Users and workflows

| Role | Recurring workflows |
|---|---|
| **Architect / designer** | Projects · 2D · 3D · revisions · materials · client presentation · documents · reports |
| **QS / estimator** | Takeoff · quantities · measurement rules · BOQ · rate books · estimates · revision comparison · tender prep |
| **Contractor / builder** | Estimates · BOQ · materials · labour · variations · procurement · actual vs estimate |
| **Developer** | Feasibility · project comparison · cost analysis · value engineering · portfolio reporting |
| **Project manager** | Status · documents · revisions · approvals · change control · cost impact |

**QSs and estimators are the strongest recurring-revenue segment** — their work is
inherently repetitive, measurable and version-sensitive, which is exactly what the
deterministic lineage architecture (ADR-015) is built for.

## The retention moat

Retention comes from **accumulated business value**, not lock-in:

```text
200 projects · 500 assemblies · custom rate books · company templates
historical estimates · saved report formats · revision history · client data
```

At that point Buildora is part of the firm's operating system.

**Good retention:** templates, history, reusable assemblies, rate books, team
workflows, professional outputs.

**Forbidden retention:** preventing export, hiding user data, obstructing
cancellation, gratuitous proprietary lock-in. Users must be able to leave with
their data.

## Features that drive retention more than novelty AI

```text
Project templates · organization templates · rate books · assemblies
saved materials · estimate history · revision comparison · BOQ templates
branded reports · project duplication · client sharing · role-based collaboration
document management · search · dashboards
```

**Professional retention must not depend on AI novelty.** It depends on the
deterministic workflow being genuinely better than a spreadsheet.

---

# 4. Project-Lifecycle Segment

## The journey

```text
idea → create project → design → 2D → 3D walkthrough → try materials
→ try colours → furnish → estimate cost → compare alternatives
→ quantities → share with professionals → construction → COMPLETE
```

Usage declines after completion. **This is expected and correct.**

## Pricing shape

Fixed-term, not indefinite subscription:

```text
Monthly Home Plan  ·  6-Month Project Pass  ·  12-Month Project Pass
optionally + AI / render credits
```

Actual prices and term lengths follow real usage and conversion data.

## Archive mode (ADR-021)

When a paid term ends the project is **archived, never deleted**:

| Preserved | Suspended |
|---|---|
| Sign in and open the project | Editing the model |
| View 2D, 3D, walkthrough | AI generation |
| View previously generated reports | New cost recalculation |
| View project history and versions | New exports |
| Export own data per policy | Advanced collaboration |

Archive mode is **an entitlement set**, not a data-removal event. Feature code
needs no "archive" branch — capabilities simply absent.

## Reactivation

```text
years later: extension · renovation · new kitchen · loft conversion · second property
→ open archived project → reactivate → CONTINUE from the existing model
```

The user resumes rather than restarts. A preserved project is the reason they
return — which is why archive mode is a revenue decision, not a courtesy.

## Cancellation experience

**Do:** explain archive mode, preserve the project, allow reactivation, allow
data export, state clearly which capabilities stop.

**Do not:** threaten immediate deletion, obstruct cancellation, use dark patterns.

A good cancellation experience directly produces reactivation and referral.

---

# 5. The Cross-Segment Growth Loop

The two segments feed each other, and this is more valuable than treating signups
as isolated accounts:

```text
Professional creates a client project
        ↓
Homeowner joins to review it
        ↓
Homeowner invites their contractor and QS
        ↓
Those professionals create their own accounts
        ↓
They bring their own projects and clients
        ↓                                    ┐
More clients join ─────────────────────────────┘
```

One temporary homeowner can generate several recurring professional users.

**This loop only works because both segments share one platform and one project.**
Separate Home and Professional products would break it — the invitation would
become an export.

`Home user → professional invitation` is therefore one of the most important
metrics in the product.

---

# 6. One Platform, Two Experiences

```text
                        BUILDORA AI
                             │
                    Canonical Platform
        Building Model · 2D · 3D · Materials · QS Engine
        BOQ Engine · Cost Engine · Documents · AI · Reports
                             │
              ┌──────────────┴──────────────┐
              ▼                             ▼
      Professional Experience         Home Experience
        multi-project                   usually one project
        advanced QS · rate books        simplified cost view
        BOQ hierarchy                   room-by-room cost
        company templates               guided workflow
        professional reports            client-friendly reports
        team collaboration              invite professionals
```

## The rule that must not be broken

**Never duplicate the QS, BOQ, cost or Building Model engines by segment.**

```text
✗ QS Engine for Home  +  QS Engine for Professional
✓ One engine, two presentations
```

A homeowner's "estimated total" and a QS's itemised BOQ are **the same
calculation at different presentation depths**. Two engines would mean two sets
of golden tests, two sets of bugs, and eventually two different answers to the
same question — destroying the reproducibility claim that is the product's moat.

Differentiate only through: **UX · onboarding · entitlements · reporting depth ·
terminology · workflow guidance**.

## Persona affects UX; entitlements control authorization

```text
Architect   → design-first dashboard
QS          → estimate/quantity-first dashboard
Contractor  → cost/project-first dashboard
Homeowner   → guided home-project dashboard
```

Persona is a **presentation hint**. It is never a security boundary.

---

# 7. Entitlements, Never Plan Names

```text
✓ feature.qs.advanced          ✓ limit.active_projects
✓ feature.boq.edit             ✓ limit.ai_credits
✓ feature.ratebooks.manage     ✓ limit.team_seats
✓ feature.home.furnishing      ✓ feature.reports.branded

✗ if (plan === "Pro") { ... }
```

Plans map to entitlements. Feature code reads entitlements only.

This is what makes archive mode, trials, fixed-term passes, grandfathered plans
and enterprise overrides all expressible without touching feature code.

---

# 8. Suggested Commercial Structure

Conceptual. **Pricing is deliberately not fixed here.**

| Tier | Shape | Focus |
|---|---|---|
| **Free** | Recurring | Evaluation, limited project, invited collaboration, archive mode |
| **Home** | **Fixed-term** | One home project — design, 3D, materials, furnishing, estimate, reports |
| **Professional** | Recurring | Multiple projects, advanced QS, BOQ, costing, rate books, templates, exports |
| **Business** | Recurring | Teams, shared libraries, org standards, permissions, admin |
| **Enterprise** | Contract | SSO, governance, support, integrations, security controls |

---

# 9. AI Positioning by Segment

Same AI architecture (ADR-010, ADR-013), different presentation.

| Professionals — *save recurring time* | Homeowners — *reduce complexity* |
|---|---|
| Explain a BOQ item | "How much will this change cost?" |
| Propose design changes | "Make the living room larger." |
| Compare estimates | "Suggest a cheaper flooring option." |
| Suggest value engineering | "Furnish this room." |
| Summarise documents | "What is this BOQ item?" |
| Generate report narratives | "Can I reduce the project cost?" |

**In both cases the numbers are deterministic.** AI explains, proposes and
recommends; the QS and cost engines calculate (ADR-013, ADR-015).

---

# 10. Metrics That Matter

## Professional

```text
MRR · ARR · churn · retention · projects per professional
weekly/monthly active professionals · estimate and BOQ runs
reports generated · revision comparisons · team invitations
```

> **Registration ≠ success.** Better signals: created 3+ projects · returned
> weekly · created estimates repeatedly · invited colleagues or clients · renewed.

## Project-lifecycle

```text
signups · projects created · 3D walkthrough usage · material changes
cost estimates · AI usage · project pass conversion · project completion
archive conversion · REACTIVATION
```

> **The highest-leverage metric is `home user → professional invitation`** — it is
> the entry point to the growth loop in §5.

## Cohorts must be distinguished

Product analytics must separate the two lifecycles. A blended churn number is
meaningless when one segment is *expected* to end.

---

# 11. Lifetime Value

```text
Professional LTV = recurring subscription × retention duration
                 + usage revenue + seats + organization expansion

Home LTV         = project plan revenue + AI/render usage
                 + reactivation + REFERRAL VALUE
```

**Do not compare the segments by subscription duration alone.** A 12-month home
customer who introduces two professionals may be worth more than a professional
who stays two years.

---

# 12. Architectural Obligations

What this strategy requires the platform to support:

```text
persona · plan · entitlements · organization membership · project membership
project lifecycle states · professional libraries · personal organization
archive mode · project reactivation · cross-user invitations
recurring AND fixed-term commercial models
```

### Binding rules

1. **Every user has an organization** — a personal one auto-created at signup.
   There is no user-owned project path (ADR-021 §3, ADR-016).
2. **Plans may be recurring or fixed-term.** Billing must not assume indefinite
   subscription (ADR-021 §1).
3. **Archive is a project state**, expressed through entitlements, not deletion
   (ADR-021 §2).
4. **Feature code reads entitlements, never plan names.**
5. **One set of engines.** No segment-specific QS, BOQ, cost or model logic.
6. Solo → organization growth is a rename and an invitation, never a migration.
7. AI cost is metered per organization regardless of segment — a subscription is
   not unlimited provider spend (ADR-010, ADR-014).

---

# 13. Roadmap Implications

The deterministic professional workflow remains the priority:

```text
Projects → Building Model → 2D → 3D → Materials → QS → BOQ → Cost
→ Reports → Revisions → Collaboration → Documents
```

AI layers across all of it.

**The Model-to-Cost MVP serves both segments already.** A homeowner designing a
house and a QS pricing one use the same core; they differ in presentation depth,
which is UX work rather than new engines.

Home-specific UX and fixed-term billing are **post-MVP**, but the schema
accommodates them now (ADR-021), because tenancy shape and subscription term are
expensive to retrofit.

## Acquisition priority

```text
1. QSs / estimators          strongest repeat workflow
2. Architects / designers
3. Small contractors / builders
4. Property developers
```

Home users are strategically important for the growth loop but **should not
dominate the early roadmap at the expense of professional retention**.

---

# 14. Product Success

| Professional | Project-lifecycle |
|---|---|
| Returns every week | Completes their project journey |
| Uses Buildora across multiple projects | Understands their design and its cost |
| Stores reusable professional data | Invites professionals |
| Invites colleagues and clients | Recommends Buildora |
| Renews | Reactivates when they build again |

**Both are success, despite opposite retention curves.**

---

# 15. Invariants

1. Professionals are the primary recurring-revenue segment.
2. Project-lifecycle users have a finite, expected lifecycle — **this is not churn**.
3. Never force permanent subscription on a completed home project.
4. Preserve completed projects in archive mode.
5. Professional retention comes from recurring workflow value, not AI novelty.
6. Templates, rate books, historical projects and reusable data are strategic assets.
7. Each segment acquires the other through project collaboration.
8. Both segments share **one** platform and **one** set of engines.
9. **Never duplicate QS, BOQ, cost or model logic by segment.**
10. Differentiate through UX and entitlements only.
11. Plans map to capabilities; feature code never reads a plan name.
12. AI enhances both segments; it is not the sole product value.
13. Cancellation must never use dark patterns.
14. Reactivation must be supported.
15. Users may evolve from personal to organization use without data migration.
16. Analytics must distinguish the two cohorts.
17. Billing supports recurring **and** fixed-term products.
18. Optimise for customer value, not subscription duration.

---

# 16. Final Position

> **Professionals create recurring revenue. Project users create project revenue,
> referrals, network growth and future reactivation. Buildora is architected and
> priced to benefit from both without forcing them into the same lifecycle.**

---

# 17. References

- ADR-021 (two lifecycles) · ADR-014 (billing ledger) · ADR-015 (cost lineage)
- ADR-016 (tenancy) · ADR-010 (AI cost) · ADR-001 (one canonical model)
- `24_Billing_AI_Credits_and_Usage.md` · `15_Database_and_Data_Architecture.md`
- `01_Master_Product_Reference.md` (product scope and modes)
