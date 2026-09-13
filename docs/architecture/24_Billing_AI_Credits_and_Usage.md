# Buildora AI — Billing, Subscription, Credits & AI Usage Architecture

> **File name:** `24_Billing_AI_Credits_and_Usage.md`
>
> **Purpose:** Production-grade billing architecture for Buildora AI as a multi-tenant AI SaaS.
>
> **Applies to:** subscriptions, packages, plan entitlements, AI usage, AI token cost tracking, AI Design Credits, Render Credits, top-ups, overages, invoices, payments, refunds, trials, promotions, billing administration, margin monitoring, and provider reconciliation.
>
> **Primary implementation stack:** NestJS + TypeScript + PostgreSQL + Drizzle + Stripe + Buildora AI Gateway + Temporal
>
> **Architecture status:** Canonical billing design unless superseded by an approved ADR.
>
> **Important:** All prices, plan limits, credit quantities, model rates, tax rules, grace periods, and credit-expiry rules are configuration/versioned business data. Do not hardcode them into application logic.

---

# 1. Billing Architecture Goals

Buildora AI billing must solve more than recurring subscriptions.

Buildora AI has variable costs from:

- AI input tokens,
- cached input tokens,
- AI output/reasoning tokens,
- embeddings,
- document processing,
- computer vision,
- storage,
- server/GPU rendering,
- background workers,
- potentially third-party BIM/CAD services.

Therefore the billing system must support:

```text
SUBSCRIPTION REVENUE
+
ONE-TIME CREDIT PACKS
+
OPTIONAL FUTURE USAGE OVERAGES
+
OPTIONAL SEAT / STORAGE ADD-ONS
```

while measuring:

```text
ACTUAL PROVIDER COST
+
INFRASTRUCTURE COST
+
CUSTOMER USAGE
+
GROSS MARGIN
```

The primary goals are:

1. predictable customer pricing,
2. strong gross-margin protection,
3. auditable credit accounting,
4. correct subscription lifecycle,
5. safe handling of payment failures,
6. exact AI provider usage recording,
7. plan changes without data loss,
8. support for future enterprise contracts,
9. provider independence,
10. accounting/reconciliation capability.

---

# 1.1 Two Commercial Lifecycles (ADR-021)

Billing must support **two term models**, not one:

```text
RECURRING     professional plans      renews until cancelled
FIXED_TERM    home / project passes   ends at term_ends_at
                                      auto_renew = false
                                      NO cancellation event
```

A **project pass** (6, 12 or 18 months) is a product in its own right. A homeowner
who completes their build and whose term expires is a **successful customer, not
churn** — analytics must not count an expired fixed term as cancellation.

When a fixed term ends:

```text
entitlements lapse → project moves to `archived` (data preserved)
→ archive entitlement set applies → reactivation restores full capability
```

**Archive mode is an entitlement set**, not a data-removal event:

| Archived | Allowed |
|---|---|
| View model, 2D, 3D, reports, history | ✓ |
| Export own data per policy | ✓ |
| Edit, AI, recalculate, new exports | ✗ |

Everything in the rest of this document — the ledger, reservations, the webhook
inbox, exact decimal money, server-confirmed grants — applies **identically** to
both term models. Only the term shape differs.

---

# 2. The Most Important Billing Decision

Do **not** expose raw AI tokens as the main customer billing unit.

The customer should understand:

```text
AI Design Credits
Render Credits
Plan limits
Projects
Seats
Storage
```

Buildora AI internally tracks:

```text
input tokens
cached input tokens
cache-write tokens
output tokens
reasoning tokens
image-generation usage
embedding tokens
GPU seconds
provider cost
```

This separates:

```text
CUSTOMER VALUE UNIT
```

from:

```text
PROVIDER COST UNIT
```

That distinction is critical.

If an AI provider changes:

- model,
- token price,
- caching price,
- reasoning behaviour,

Buildora AI can update its internal routing/rate card without immediately changing the customer's plan language.

---

# 3. Sources of Truth

Buildora AI billing uses multiple authoritative systems for different responsibilities.

## 3.1 Stripe

Stripe is authoritative for:

- payment method,
- payment status,
- externally issued invoice,
- Stripe subscription object,
- refunds,
- payment disputes,
- tax calculation when Stripe Tax is used,
- external billing transaction IDs.

Stripe is **not** the authoritative source for:

- Buildora AI feature permissions,
- AI credit balances,
- project limits,
- AI token cost,
- Buildora AI usage reservations,
- Buildora AI gross-margin decisions.

---

## 3.2 Buildora AI Billing Core

Buildora AI is authoritative for:

- plan catalog,
- plan version,
- entitlement definitions,
- organization entitlement state,
- usage limits,
- internal subscription mirror,
- credit wallets,
- credit grants,
- credit consumption,
- AI usage accounting,
- render usage,
- feature access,
- grace-period behaviour.

---

## 3.3 AI Gateway / Provider Usage Records

The Buildora AI Gateway is authoritative for captured provider usage such as:

```text
provider
provider model
input tokens
cached input tokens
cache-write tokens
output tokens
reasoning tokens
provider request ID
```

where the provider returns these values.

The current OpenAI Responses API returns usage including input tokens, output tokens, total tokens, cached input details, cache-write details, and output reasoning-token details. Buildora AI must persist the raw usage breakdown rather than only storing `total_tokens`.

---

## 3.4 Provider Cost Rate Cards

Buildora AI stores a versioned internal rate card.

Never calculate historical provider cost using today's AI prices.

Example:

```text
OpenAI
model = ADVANCED_MODEL / provider model ID
effective_from = ...
input_price_per_million = ...
cached_input_price_per_million = ...
output_price_per_million = ...
```

Historical usage points to the rate-card version used for cost estimation/reconciliation.

---

# 4. High-Level Architecture

```text
                     BUILDORA USER
                           │
                           ▼
                     WEB APPLICATION
                           │
              ┌────────────┼────────────┐
              │            │            │
              ▼            ▼            ▼
          Checkout       Product      AI / Render
              │          Actions        Actions
              │            │            │
              ▼            ▼            ▼
          Stripe       Entitlement    Usage Guard
                         Service          │
                           │              ▼
                           │        Credit Reservation
                           │              │
                           └───────┬──────┘
                                   ▼
                             Product Action
                                   │
                         ┌─────────┴──────────┐
                         │                    │
                         ▼                    ▼
                    AI Gateway          Render Worker
                         │                    │
                         ▼                    ▼
                   AI Providers          GPU/Blender
                         │                    │
                         └─────────┬──────────┘
                                   ▼
                            Usage Settlement
                                   │
                                   ▼
                             Usage Ledger
                                   │
                                   ▼
                          Cost / Margin Data

Stripe Webhooks
      │
      ▼
Webhook Inbox
      │
      ▼
Billing Reconciler
      │
      ├── Subscription Mirror
      ├── Invoices
      ├── Payments
      ├── Plan Grants
      └── Entitlement Refresh
```

---

# 5. Billing Bounded Contexts

Do not build billing as one giant `BillingService`.

Recommended modules:

```text
Billing Catalog
Subscription Management
Entitlements
Usage Metering
Credit Wallets
AI Cost Accounting
Payments
Invoices
Webhook Processing
Billing Reconciliation
Billing Administration
Margin Analytics
```

Suggested repository structure:

```text
apps/api/src/modules/billing/
├── catalog/
├── subscriptions/
├── entitlements/
├── credits/
├── usage/
├── payments/
├── invoices/
├── webhooks/
├── reconciliation/
├── admin/
└── analytics/

packages/
├── billing-domain/
└── billing-contracts/
```

For MVP, these can remain modules inside the NestJS modular monolith.

Do **not** create separate billing microservices initially.

---

# 6. Billing Account Ownership

Billing is organization-level.

Use:

```text
organization
    │
    └── billing_account
```

Do not normally create a separate subscription for each human user.

A Buildora AI organization is the customer account.

Example:

```text
Echosoft Design Ltd
   ├── Owner
   ├── Architect
   ├── QS
   └── Viewer

Billing Account
   └── Pro / Business subscription
```

This allows:

- team usage,
- organization credits,
- organization AI budget,
- organization invoices.

---

# 7. Core Billing Entities

Recommended primary entities:

```text
billing_accounts
plan_definitions
plan_versions
plan_prices
feature_definitions
plan_entitlements
billing_subscriptions
billing_periods
entitlement_snapshots

credit_wallets
credit_grants
credit_reservations
credit_ledger_entries

usage_events
usage_aggregates
provider_usage_records
provider_rate_cards
provider_cost_components

billing_orders
payment_records
refund_records
invoice_records
invoice_line_records

billing_webhook_events
billing_adjustments
billing_alerts
billing_reconciliation_runs
```

---

# 8. Billing Account

Conceptual schema:

```text
billing_accounts
----------------
id
organization_id
provider
provider_customer_id
billing_email
default_currency
country
tax_status
status
created_at
updated_at
```

One organization normally has one active billing account.

Do not use the Stripe customer ID as the Buildora AI primary key.

---

# 9. Plan Catalog Architecture

Separate:

```text
PLAN DEFINITION
```

from:

```text
PLAN VERSION
```

Example:

```text
Pro
  ├── Version 1
  ├── Version 2
  └── Version 3
```

Why?

Because Buildora AI may later change:

- price,
- AI credits,
- project limits,
- included features.

Existing customers may be grandfathered.

Never mutate historical plan rules and pretend they always existed.

---

# 10. Plan Definition

Example:

```text
plan_definitions
----------------
id
code
name
description
is_public
is_active
created_at
```

Codes:

```text
FREE
HOME
PRO
BUSINESS
ENTERPRISE
```

---

# 11. Plan Version

Example:

```text
plan_versions
-------------
id
plan_definition_id
version
effective_from
effective_to
status
created_at
```

Example:

```text
PRO v1
PRO v2
```

A subscription references a specific plan version.

---

# 12. Plan Prices

A plan version can have multiple commercial prices.

Example:

```text
plan_prices
-----------
id
plan_version_id
currency
billing_interval
amount_decimal
stripe_price_id
country_scope
is_active
```

Examples:

```text
PRO / USD / monthly / 49.00
PRO / USD / annual
PRO / GBP / monthly
```

Never use `stripe_price_id` as the internal plan ID.

---

# 13. Feature Definitions

Create a feature catalog.

Examples:

```text
PROJECTS
TEAM_MEMBERS
AI_COPILOT
AI_DESIGN
PLAN_RECOGNITION
IFC_IMPORT
PRO_QS
PDF_EXPORT
XLSX_EXPORT
RENDER_HD
RENDER_4K
DOCUMENT_RAG
API_ACCESS
SSO
CUSTOM_RATE_LIBRARY
```

## The catalogue is runtime-editable (ADR-022)

A platform admin registers a feature code **without a deployment**:

```text
feature_definitions
-------------------
id
code                      e.g. feature.walkthrough
name                      human label shown in the admin UI
description               what this gates, in plain language
category                  DESIGN | INTERIOR | QS | AI | EXPORT | COLLAB | PLATFORM
value_type                boolean | integer | decimal | string | unlimited
default_value             applied when no plan entitlement exists
is_active                 retired codes stop gating without being deleted
created_at / updated_at
```

**Registering a code does not make it enforce anything.** A code gates a
capability only where application code calls the entitlement service with it.
Each code is bound to exactly **one owning check site**, recorded with the
definition:

```text
feature.walkthrough        → apps/api  3D session authorization
feature.home.furnishing    → apps/api  PLACE_FURNITURE command guard
feature.interior.finishes  → apps/api  SET_ELEMENT_FINISH command guard
feature.ai.interior        → apps/api  AI interior tool authorization
feature.qs.advanced        → apps/api  quantity run entry point
```

The admin UI must surface **orphaned codes** — registered but never checked —
as inactive. Otherwise an admin sets a code on a plan and nothing happens.

## Interior feature codes (ADR-019, ADR-020)

Registered when their sprints ship:

```text
feature.walkthrough          Sprint 21b    walk inside the building
feature.interior.finishes    Sprint 22     change materials and colours
feature.home.furnishing      Sprint 26c    place and move FF&E
feature.ai.interior          Sprint 30b    AI furnishing and styling
```

These are the first capabilities that genuinely differ by tier, which is why the
gating mechanism must be real before they ship.

Value types:

```text
boolean
integer
decimal
string
unlimited
```

---

# 13.1 Entitlements Are Capabilities, Never Plan Names

Binding across both lifecycles (ADR-021):

```text
✓ feature.qs.advanced          ✓ limit.active_projects
✓ feature.boq.edit             ✓ limit.ai_credits
✓ feature.ratebooks.manage     ✓ limit.team_seats
✓ feature.home.furnishing      ✓ feature.reports.branded

✗ if (plan === "Pro") { ... }
```

**Feature code reads entitlements only.** This is what makes archive mode, trials,
fixed-term passes, grandfathered plans and enterprise overrides expressible
without touching feature code — archive mode is simply an entitlement set with
editing capabilities absent.

Any `if (plan === "...")` in feature code is a HIGH review finding.

## 13.2 Enforcement contract (ADR-022)

```text
SERVER   decides.  Every gated capability is checked in apps/api BEFORE the
                   work happens. Denial returns FEATURE_NOT_ENTITLED
                   (402 or 403) carrying the feature code.

CLIENT   reflects. The UI reads the same entitlements to hide, disable or
                   badge a capability — for usability, NEVER for security.
```

**Hiding a button is not access control.** A user calling the API directly must
be refused by the server. Any gated capability without a server check site is a
**BLOCKER**.

Order of checks:

```text
1. feature flag      is this code path enabled at all?      (rollout)
2. entitlement       is this customer allowed?              (commercial)
3. credit reserve    do they have enough to consume?        (metered)
```

The flag wins over the entitlement — an unfinished feature must never be sold.
The entitlement check precedes the credit reservation, so an **unentitled request
consumes no credit**.

## 13.3 Feature flags are NOT entitlements

Separate systems, deliberately:

| | Feature flags | Entitlements |
|---|---|---|
| Question | "Is this code path enabled?" | "Is this customer allowed?" |
| Purpose | Rollout, canary, kill switch | Commercial rights |
| Audience | Engineering | Product / finance |
| Scope | Global or cohort | Per organization |
| Changes | During a release | During a commercial decision |

Conflating them means a rollout flag can grant paid access, and a billing change
can break a canary.

## 13.4 feature.* vs limit.* vs credits

```text
feature.*   CAN they do this at all?    boolean     → entitlement
limit.*     HOW MUCH is allowed?        counted     → entitlement
credits     METERED consumption         wallet      → reservation (ADR-014)
```

`feature.ai.interior` says AI furnishing is available. AI Design Credits say how
much of it can be consumed. A user may be **entitled and out of credit** — those
are different errors with different UX:

```text
FEATURE_NOT_ENTITLED   → upgrade / plan CTA
INSUFFICIENT_CREDITS   → top-up / wait for renewal CTA
```

---

# 14. Plan Entitlements

Example:

```text
plan_entitlements
-----------------
id
plan_version_id
feature_code
value
soft_limit
hard_limit
metadata_json
```

Example Pro entitlements:

```text
PROJECTS               unlimited
AI_COPILOT              true
AI_DESIGN_CREDITS       30
RENDER_CREDITS          20
IFC_IMPORT              true
PRO_QS                  true
TEAM_MEMBERS            5
STORAGE_GB              20
```

Exact commercial values remain configurable.

---

# 15. Effective Entitlements

Do not query Stripe on every feature request.

Buildora AI creates an effective entitlement state from:

```text
plan
+
subscription status
+
add-ons
+
promotions
+
admin overrides
+
enterprise contract overrides
```

## Resolution order (ADR-022)

```text
1. enterprise contract override      highest precedence
2. organization override             admin-applied, IMMEDIATE
3. add-on / promotion                purchased or granted
4. plan version entitlement          from the subscribed plan version
5. feature_definitions.default_value lowest precedence
```

**Archive mode (ADR-021) applies at layer 2** as a read-only capability profile —
so no feature needs an "is archived" branch.

## Organization overrides — the immediate lever

```text
plan_entitlement_overrides
--------------------------
id
organization_id
feature_code
value
reason                    REQUIRED — appears in the audit trail
effective_from
effective_to              nullable; NULL = until revoked
created_by_admin_id
created_at
```

Two distinct admin operations with **different semantics**:

| Operation | Applies to | Effect | Grandfathering |
|---|---|---|---|
| **Entitlement override** | One organization | **Immediate** | N/A — per-tenant |
| **Publish a plan version** | New subscribers; existing on migration | At subscribe or explicit migration | Existing subscribers **keep their version** (ADR-021) |

> **The admin UI must state this at the point of action.** The intuitive
> expectation when editing a plan is "it changes now" — it does not. Existing
> subscribers stay on their version until migrated. For "give this customer
> access now", use an override.

Conceptually:

```text
EntitlementService
    ↓
getEffectiveEntitlements(organizationId)
```

Resolution is **cached with explicit invalidation** on: subscription change,
override change, plan migration, period rollover. It must never query Stripe on
a feature check.

Response:

```json
{
  "plan": "PRO",
  "status": "active",
  "features": {
    "PRO_QS": true,
    "IFC_IMPORT": true,
    "TEAM_MEMBERS": 5,
    "STORAGE_GB": 20
  }
}
```

Cache carefully, but the database remains authoritative.

---

# 16. Entitlement Decision

Every guarded feature should return an explicit decision.

```ts
interface EntitlementDecision {
  allowed: boolean;
  code:
    | "ALLOWED"
    | "FEATURE_NOT_INCLUDED"
    | "LIMIT_REACHED"
    | "SUBSCRIPTION_INACTIVE"
    | "PAYMENT_REQUIRED"
    | "ACCOUNT_SUSPENDED";
  limit?: string;
  current?: string;
}
```

Do not scatter:

```ts
if (plan === "PRO")
```

through the product.

---

# 17. Subscription State Model

Buildora AI should normalize external subscription status into an internal state.

Recommended:

```text
TRIALING
ACTIVE
PAST_DUE
GRACE
SUSPENDED
CANCEL_AT_PERIOD_END
CANCELED
INCOMPLETE
```

Do not blindly expose Stripe status as the domain state.

Keep external status separately.

---

# 18. Billing Subscription Record

Conceptual:

```text
billing_subscriptions
---------------------
id
billing_account_id
plan_version_id
provider
provider_subscription_id
provider_status
internal_status

term_model              RECURRING | FIXED_TERM     (ADR-021)
term_ends_at            nullable — set for FIXED_TERM
auto_renew              boolean — false for a project pass

current_period_start
current_period_end
cancel_at_period_end
canceled_at
trial_start
trial_end
created_at
updated_at
```

## Term model semantics (ADR-021)

```text
RECURRING     renews until cancelled        professional plans
FIXED_TERM    ends at term_ends_at          home / project passes
              auto_renew = false
              NO cancellation event
```

At `term_ends_at`:

1. entitlements lapse,
2. the associated project moves to `archived` (**data preserved**),
3. the **archive entitlement set** applies — read-only capabilities,
4. **no cancellation is recorded**; analytics must not count this as churn,
5. reactivation restores full capability against the existing model.

The ledger, reservations, webhook inbox, exact decimal money and
server-confirmed grants apply **identically** to both term models.

---

# 19. Billing Period

Create explicit billing-period records.

```text
billing_periods
---------------
id
subscription_id
period_start
period_end
status
renewal_invoice_id
grant_status
created_at
```

This gives Buildora AI a stable period for:

- monthly included credits,
- usage aggregation,
- margin calculations.

---

# 20. Why Billing Period Records Matter

Without explicit periods, it becomes difficult to answer:

```text
How many AI credits did this customer receive this month?
Which provider costs belong to this subscription cycle?
Did we grant credits twice after a webhook retry?
How profitable was this organization this billing cycle?
```

---

# 21. Customer-Facing Credit Types

Initial Buildora AI wallets:

```text
AI_DESIGN
RENDER
```

Potential future wallets:

```text
PLAN_RECOGNITION
API_COMPUTE
STORAGE
```

Do not create too many credit types initially.

---

# 22. AI Design Credits

Suggested customer-facing charging model:

```text
Standard AI concept        2
Advanced AI concept        3
Expert AI design study     5
Major AI modification      1
Manual 2D/3D sync          0
QS recalculation           0
Cost recalculation         0
```

These are examples.

The actual charge table must be versioned configuration.

---

# 23. Render Credits

Example:

```text
Real-time browser 3D       0
Standard preview           0/fair-use
HD still                   1
4K still                   2
360 panorama               3
Video walkthrough          5–10
```

Again: configuration, not hardcoded logic.

---

# 24. AI Copilot Fair-Use Architecture

Ordinary AI Copilot chat may be marketed as included/fair-use.

However Buildora AI must still protect margin.

Internally track:

```text
monthly AI provider cost
requests per minute
requests per day
high-cost model escalation
context size
tool calls
```

Recommended plan controls:

```text
soft AI cost budget
hard abuse threshold
per-minute rate limit
per-day high-cost action threshold
```

Do not necessarily expose raw dollar limits to the user.

When soft budget is reached:

```text
route to cheaper model
reduce unnecessary context
disable expert escalation unless a Design Credit action
```

When abuse threshold is reached:

```text
temporary throttle
or
require Design Credits for expensive operation
```

This prevents “unlimited chat” from becoming unlimited provider spend.

---

# 25. Raw AI Usage Record

Every provider request should create or update a provider usage record.

Conceptual:

```text
provider_usage_records
----------------------
id
organization_id
project_id
user_id
ai_job_id
provider
provider_model
provider_request_id
rate_card_id

input_tokens
cached_input_tokens
cache_write_tokens
output_tokens
reasoning_tokens
total_tokens

provider_cost_estimated
provider_cost_currency

request_started_at
request_completed_at
status
metadata_json
```

Use integer token columns.

Use exact decimal for cost.

---

# 26. One AI User Action May Contain Multiple Provider Calls

Example:

```text
"Reduce project cost by 10%"
```

may execute:

```text
AI planning call
+
tool calls
+
AI synthesis call
```

Therefore model:

```text
ai_job
    ├── provider_usage_record 1
    ├── provider_usage_record 2
    └── provider_usage_record 3
```

Customer Design Credits are charged against the **job/action**, not separately for every provider call.

Provider cost is summed from all provider calls.

---

# 27. Provider Rate Card

Conceptual:

```text
provider_rate_cards
-------------------
id
provider
provider_model
effective_from
effective_to
currency

input_per_million
cached_input_per_million
cache_write_per_million
output_per_million

metadata_json
```

Some provider components may not use every field.

A more flexible component model may be:

```text
provider_rate_components
------------------------
rate_card_id
component
unit
price
```

Examples:

```text
INPUT_TOKEN
CACHED_INPUT_TOKEN
OUTPUT_TOKEN
IMAGE
EMBEDDING_TOKEN
WEB_SEARCH_CALL
GPU_SECOND
```

---

# 28. AI Cost Calculation

Pseudo-formula:

```text
uncached_input_tokens
=
input_tokens
-
cached_input_tokens
```

Then:

```text
input_cost
=
uncached_input_tokens
× input_rate

cached_cost
=
cached_input_tokens
× cached_rate

output_cost
=
output_tokens
× output_rate
```

Add provider-specific components as required.

Do not assume every provider prices:

- reasoning tokens,
- cache writes,
- tools,

the same way.

Use the rate-card component configuration.

---

# 29. Provider Cost vs Customer Charge

Keep both.

Example:

```text
Customer action:
Advanced Design = 3 Design Credits

Actual provider cost:
$0.74
```

Store both.

This allows margin analytics.

Do not calculate customer credit charges directly from today's provider dollar cost unless the product explicitly becomes cost-plus usage billing.

---

# 30. Usage Event Architecture

Every billable/metered operation creates a durable usage event.

```text
usage_events
------------
id
organization_id
user_id
project_id
meter_code
operation_code
quantity
unit
occurred_at
idempotency_key
source_type
source_id
metadata_json
```

Examples:

```text
AI_DESIGN / ADVANCED_CONCEPT / 1 operation
RENDER / HD_STILL / 1 render
STORAGE / OBJECT_BYTES / ...
```

---

# 31. Usage Events Are Immutable

Do not update a historical usage event to “fix” it.

If correction is required:

```text
original usage
+
adjustment/reversal
```

This creates an auditable ledger.

---

# 32. Credit Wallet

Conceptual:

```text
credit_wallets
--------------
id
organization_id
credit_type
status
created_at
```

Unique:

```text
organization_id + credit_type
```

---

# 33. Credit Grant

Credits are not one undifferentiated balance.

Track the source.

```text
credit_grants
-------------
id
wallet_id
source_type
source_id
granted_quantity
remaining_quantity
valid_from
expires_at
priority
status
created_at
```

Sources:

```text
SUBSCRIPTION_PERIOD
TOP_UP_PURCHASE
PROMOTION
ADMIN_ADJUSTMENT
REFUND_ADJUSTMENT
ENTERPRISE_CONTRACT
```

---

# 34. Credit Expiry Policy

Recommended architecture supports per-grant expiry.

Example policy:

Subscription credits:

```text
expire at period end
do not roll over
```

Top-up credits:

```text
longer expiry or no expiry
```

Promotional credits:

```text
explicit expiry
```

Do not bake one global expiry rule into wallet code.

---

# 35. Consumption Ordering

Recommended consumption algorithm:

1. eligible grant,
2. earliest expiry first,
3. then grant priority,
4. then oldest grant.

This minimizes customer loss from expiring balances.

Example:

```text
promo expires tomorrow
subscription expires in 10 days
top-up expires next year
```

consume promo first.

---

# 36. Credit Ledger

The ledger is the audit trail.

```text
credit_ledger_entries
---------------------
id
wallet_id
grant_id
entry_type
quantity
balance_effect
reservation_id
usage_event_id
reference_type
reference_id
created_at
created_by
metadata_json
```

Entry types:

```text
GRANT
RESERVE
RELEASE
CONSUME
REFUND
REVOKE
ADJUSTMENT
EXPIRE
```

---

# 37. Do Not Store Only `credits_remaining`

A mutable number alone is insufficient.

Bad:

```text
organization.ai_credits = 27
```

Required:

```text
ledger
+
grant balances
+
optional cached aggregate balance
```

If an aggregate balance is stored for speed, it must be transactionally maintained and reconstructable from ledger data.

---

# 38. Credit Reservation

High-cost operations must reserve credits before beginning.

Example:

```text
Expert AI design requires 5 credits
```

Flow:

```text
check entitlement
    ↓
begin transaction
    ↓
lock/validate available eligible grants
    ↓
reserve 5
    ↓
commit
    ↓
run AI job
```

This prevents two simultaneous expensive requests from spending the same credits.

---

# 39. Reservation Record

```text
credit_reservations
-------------------
id
wallet_id
organization_id
operation_code
requested_quantity
reserved_quantity
status
expires_at
idempotency_key
source_type
source_id
created_at
settled_at
```

Statuses:

```text
PENDING
RESERVED
SETTLED
RELEASED
EXPIRED
FAILED
```

---

# 40. Settlement

After the operation:

```text
reserved 5
actual charge 3
```

settlement:

```text
consume 3
release 2
```

If actual charge equals reserve:

```text
consume all
```

If action fails before delivering value:

```text
release all
```

Policy for partially successful operations must be explicit per operation.

---

# 41. Reservation TTL

Every reservation has an expiry.

Example:

```text
AI job: 15–30 minutes
render: several hours
```

A cleanup/reconciliation job handles abandoned reservations.

Do not permanently lock credits because a worker crashed.

---

# 42. Reservation Idempotency

Every credit-reserving request should have an idempotency key.

Example:

```text
ai-job:job_123
```

Retrying the same request returns/reuses the existing reservation.

It must not reserve twice.

---

# 43. Concurrency Control

Credit reservation is a financial-like operation.

Use:

- PostgreSQL transaction,
- row locking or another safe concurrency strategy,
- unique idempotency constraints.

Do not implement credit balance protection only with Redis.

Redis may assist with rate limiting but PostgreSQL remains authoritative.

---

# 44. Monthly Subscription Credit Grant

Credits should normally be granted when the billing period is financially confirmed.

Recommended:

```text
invoice paid
    ↓
identify subscription period
    ↓
create period grant
```

Unique constraint example:

```text
(subscription_period_id, credit_type, grant_reason)
```

This prevents duplicate grants when Stripe retries webhooks.

---

# 45. Trial Credits

Trial credits use separate grants.

Example:

```text
source_type = TRIAL
expires_at = trial_end
```

Do not reuse paid subscription grant records.

This allows:

- smaller trial allowance,
- no accidental rollover,
- trial abuse detection.

---

# 46. Free Plan Credits

If Free includes AI:

create a recurring/free-period allowance model.

Do not require a Stripe subscription merely to create Free-plan grants.

Free billing periods can be generated internally.

---

# 47. Top-Up Credit Packs

Example:

```text
10 AI Design Credits
25 AI Design Credits
60 AI Design Credits
```

Payment flow:

```text
Create Buildora AI order
    ↓
Create Stripe Checkout/Payment
    ↓
payment confirmed
    ↓
order marked paid
    ↓
credit grant created
```

Never grant top-up credits merely because the browser returned from Checkout.

Wait for authoritative payment confirmation.

---

# 48. Billing Order

Conceptual:

```text
billing_orders
--------------
id
billing_account_id
order_type
status
currency
amount
provider_checkout_session_id
provider_payment_intent_id
created_at
paid_at
metadata_json
```

Order types:

```text
CREDIT_TOP_UP
ONE_TIME_REPORT
OTHER_ADD_ON
```

---

# 49. Top-Up Idempotency

Unique:

```text
paid order → one grant
```

Webhook retries must not create multiple grants.

Use:

```text
source_type = TOP_UP_PURCHASE
source_id = order_id
```

with a unique database constraint.

---

# 50. Promotional Credits

Marketing/admin can issue promotional credits.

Record:

```text
reason
campaign
issuer
expiry
```

Promotional credits should never silently appear as subscription credits.

---

# 51. Admin Adjustments

All manual credit changes require:

- admin actor,
- reason,
- ticket/reference,
- quantity,
- timestamp.

Avoid direct SQL balance edits.

Provide admin actions:

```text
grant credits
revoke credits
extend expiry
refund credits
```

Each produces ledger entries.

---

# 52. Subscription Checkout Flow

```text
User chooses plan
    ↓
Buildora AI creates checkout intent/session
    ↓
Stripe Checkout
    ↓
Stripe handles payment
    ↓
User returns to Buildora AI
    ↓
Buildora AI shows "Activating subscription…"
    ↓
Stripe webhook received
    ↓
Webhook verified
    ↓
Subscription mirror updated
    ↓
Invoice/payment confirmed
    ↓
Entitlements activated
    ↓
Credits granted
```

Do not activate paid entitlements only from the browser redirect.

---

# 53. Checkout Return Race

Common scenario:

```text
browser returns before webhook
```

Frontend should poll:

```text
GET /billing/status
```

or use a short status refresh.

Display:

```text
Payment received. Activating your plan…
```

Do not display an error immediately because webhook processing has not completed yet.

---

# 54. Subscription Renewal Flow

```text
new invoice created
    ↓
payment attempted
    ↓
invoice paid
    ↓
internal subscription period confirmed
    ↓
monthly grants created
    ↓
entitlement period refreshed
```

Do not grant next-period credits when invoice is merely created.

Grant based on the configured financial policy, typically successful payment.

---

# 55. Upgrade Policy

Recommended Buildora AI default:

```text
Upgrade → immediate
Downgrade → next billing period
```

Upgrade flow:

1. preview proration,
2. show customer amount,
3. confirm,
4. update Stripe subscription,
5. wait for required payment result/webhook,
6. activate upgraded entitlements,
7. apply credit-delta policy.

Stripe can create prorations when subscription prices/items change. Buildora AI should preview the proration before applying the upgrade.

---

# 56. Upgrade Credit Policy

A plan upgrade may increase monthly included credits.

Choose and configure one of:

## Option A — Prorated extra credits

Example:

```text
old plan = 10 credits
new plan = 30 credits
50% period remaining

extra grant
=
(30 - 10) × 50%
=
10 credits
```

## Option B — Full difference immediately

Example:

```text
grant 20 additional credits
```

Simpler customer experience but slightly higher variable-cost exposure.

## Recommended initial policy

For Buildora AI MVP:

```text
upgrade immediately
grant full remaining plan difference once payment succeeds
```

provided the commercial margins support it.

Track this as a grant linked to the upgrade event.

---

# 57. Downgrade Policy

Recommended:

```text
downgrade takes effect at next period
```

Why:

- customer already paid current plan,
- avoids removing current access,
- avoids complicated mid-cycle credit clawback,
- reduces support issues.

Schedule new plan for:

```text
current_period_end
```

Current entitlements remain until then.

---

# 58. Downgrade and Existing Resources

Never delete customer data because of downgrade.

If new plan has:

```text
max 3 active projects
```

but customer has 8:

- preserve all projects,
- allow viewing,
- block creation of new projects,
- optionally require archiving before more creation,
- clearly show over-limit state.

Do not automatically delete or hide five projects.

---

# 59. Downgrade and Premium Features

Example:

Business → Home.

Existing IFC file remains stored.

The customer may:

- view preserved project content where product policy allows,
- lose ability to perform new premium import/export actions.

Do not destroy premium-created data.

---

# 60. Cancellation

Default:

```text
cancel at period end
```

Internal status:

```text
CANCEL_AT_PERIOD_END
```

Customer keeps paid entitlements until:

```text
current_period_end
```

At end:

```text
subscription → canceled
entitlements → free/downgraded state
new paid grants stop
```

---

# 61. Reactivation

If cancellation is scheduled but period not ended:

```text
Reactivate
```

removes `cancel_at_period_end`.

Do not create a second subscription.

---

# 62. Immediate Cancellation

Reserve for:

- admin/security action,
- refund case,
- enterprise contract terms.

Immediate cancellation requires explicit policy for:

- unused credits,
- refunds,
- project access.

Do not use immediate cancellation as the normal user flow.

---

# 63. Payment Failure

Recommended state progression:

```text
ACTIVE
    ↓
PAST_DUE
    ↓
GRACE
    ↓
SUSPENDED
```

Exact time windows are configurable.

Example policy:

```text
PAST_DUE:
show warning
allow normal project access
restrict unusually expensive new AI/render operations if risk is high

GRACE:
allow viewing/editing core project data
block new premium compute actions

SUSPENDED:
read-only access
block new paid operations
```

Do not immediately lock users out of their project data after one failed renewal attempt.

---

# 64. Data Access During Billing Suspension

Customer construction data should remain accessible according to retention policy.

Recommended:

```text
view/download own data
```

while premium compute/actions can be restricted.

Do not hold user data hostage for a failed payment.

---

# 65. Unpaid Invoice and Plan Change

Be careful with upgrades/downgrades while the current invoice is unpaid.

Proration systems can create credits based on time the customer has not actually paid for.

Recommended Buildora AI policy:

```text
if latest invoice is unpaid:
do not perform ordinary prorated plan changes automatically
```

Instead:

- require invoice resolution,
- or use a deliberate non-prorated/manual billing policy.

---

# 66. Proration Policy

Do not implement your own proration arithmetic unless there is a strong reason.

Use Stripe's subscription/invoice capabilities for actual customer money.

Buildora AI can display a preview returned from Stripe.

Keep Buildora AI internal credit adjustments separate from currency proration.

---

# 67. Annual Plans

Annual subscription may still grant AI credits:

```text
monthly
```

rather than all at once.

Recommended:

```text
annual payment
+
monthly internal credit periods
```

Why:

- protects margin,
- prevents one-month exhaustion of a year's AI budget,
- simplifies fair-use controls.

This means:

```text
financial billing period = yearly
usage allowance period = monthly
```

The architecture must support distinct:

```text
invoice period
usage grant period
```

---

# 68. Enterprise Contracts

Future enterprise requirements may include:

- annual committed spend,
- custom seat quantities,
- custom AI allowance,
- pooled credits,
- overage invoicing,
- custom rate cards,
- negotiated discounts,
- invoicing instead of card payment,
- purchase order number,
- contract start/end,
- ramp schedule.

Do not force Enterprise into the exact same simple plan table.

Use:

```text
contract overrides
```

on top of versioned plan/entitlement infrastructure.

---

# 69. Seats

For MVP:

```text
team member limit
```

can be an entitlement.

Later true seat billing can use:

```text
subscription quantity
```

with policy such as:

- increase seats immediately,
- decrease at next period.

Do not count pending invites as paid seats unless product policy explicitly says so.

---

# 70. Storage Limits

Track storage usage separately.

Examples:

```text
object bytes
document bytes
render bytes
```

Do not query object storage and sum the whole bucket for every request.

Maintain usage counters/events and reconcile periodically.

On limit:

- block new upload,
- allow existing access,
- offer upgrade/storage add-on.

---

# 71. AI Usage Reservation Flow

Example: Advanced AI Concept.

```text
Request
    ↓
Auth
    ↓
Entitlement check
    ↓
Determine operation class
    ↓
Required Design Credits = 3
    ↓
Reserve 3
    ↓
Create AI job
    ↓
Run routed provider calls
    ↓
Capture token usage/cost
    ↓
Validate design result
    ↓
Deliver result
    ↓
Settle 3 credits
```

If generation fails before usable result:

```text
release reservation
```

---

# 72. AI Escalation and Billing

Suppose an operation starts on a cheaper model then escalates.

Customer charge remains:

```text
Advanced Concept = 3 Design Credits
```

unless product terms explicitly define another charge.

Internal cost records reflect:

```text
Terra call
+
Sol call
+
Astra call
```

This prevents customer pricing from becoming unpredictable due to internal routing.

---

# 73. AI Retry and Billing

If provider fails and Buildora AI retries:

Do not double-charge the user.

All provider retries belong to the same:

```text
AI job
+
credit reservation
```

Internal provider cost may increase.

That cost belongs in margin analysis.

---

# 74. User-Cancelled AI Job

Policy examples:

If canceled before provider execution:

```text
release all
```

If provider already completed expensive work and result exists:

```text
charge according to operation delivery policy
```

Define this clearly for each operation.

For MVP, prefer:

```text
charge only when Buildora AI produces a usable result
```

unless abuse becomes material.

---

# 75. AI Partial Failure

Example:

```text
design generated
cost optimization step fails
```

The job should define:

- required steps,
- optional steps,
- completion threshold.

Settlement should follow the customer-visible value delivered, not arbitrary provider call count.

---

# 76. Rendering Usage Flow

```text
User requests 4K render
    ↓
reserve Render Credits
    ↓
enqueue Temporal workflow
    ↓
render worker
    ↓
upload output
    ↓
mark render complete
    ↓
settle credits
```

Worker retry does not cause another customer charge.

---

# 77. Expired/Abandoned Jobs

Scheduled reconciliation:

```text
find RESERVED reservations past expires_at
    ↓
inspect linked job
    ↓
settle if completed
or
release if abandoned/failed
```

Never leave reserved balances permanently stranded.

---

# 78. Webhook Inbox Pattern

Never perform complex billing mutation directly before recording the webhook.

Flow:

```text
Stripe webhook
    ↓
verify signature
    ↓
insert billing_webhook_event
    ↓
unique provider_event_id
    ↓
acknowledge/queue processing
    ↓
process event idempotently
```

Conceptual:

```text
billing_webhook_events
----------------------
id
provider
provider_event_id
event_type
payload_json
received_at
processing_status
processed_at
attempts
last_error
```

---

# 79. Webhook Duplicate Handling

Stripe may retry delivery.

Use unique:

```text
provider + provider_event_id
```

Duplicate event:

```text
acknowledge
do not process financial effect twice
```

---

# 80. Webhook Ordering

Do not assume webhooks arrive in the order you want.

The processor should be able to fetch/reconcile current provider state when necessary.

Example:

```text
invoice.paid
```

may be processed around other subscription update events.

Use provider object IDs and current subscription/invoice state for reconciliation.

---

# 81. Important Stripe Events

The exact set depends on integration, but Buildora AI should expect events around:

```text
checkout completion
subscription created/updated/deleted
invoice created/finalized/paid/payment_failed
payment succeeded/failed
refund
dispute
```

Do not code business logic purely around event-name strings spread across many files.

Centralize event mapping.

---

# 82. Stripe Client Idempotency

For Stripe POST operations, use idempotency keys where appropriate.

Example:

```text
buildora:subscription-upgrade:{operationId}
```

This prevents network retries from creating duplicate provider actions.

---

# 83. Billing Reconciliation

Webhooks are important but not the only safety mechanism.

Run reconciliation jobs.

## Daily subscription reconciliation

Compare:

```text
Buildora AI subscription mirror
vs
Stripe subscription
```

## Invoice reconciliation

Compare:

```text
internal invoice records
vs
Stripe invoices
```

## Credit-grant reconciliation

Check:

```text
paid period
→ exactly expected grants
```

## Top-up reconciliation

Check:

```text
paid order
→ exactly one credit grant
```

---

# 84. Provider Usage Reconciliation

Where provider usage/reporting APIs permit:

```text
internal provider usage records
vs
provider usage/cost reports
```

Purpose:

- detect missing usage,
- detect pricing/rate-card error,
- validate gross margin.

Do not use provider monthly invoice alone as the customer usage ledger.

---

# 85. Billing Reconciliation Run

```text
billing_reconciliation_runs
---------------------------
id
type
period_start
period_end
status
started_at
completed_at
difference_count
difference_amount
report_json
```

Differences create admin alerts.

---

# 86. Invoice Mirror

Store a mirror/reference of externally issued invoices.

```text
invoice_records
---------------
id
billing_account_id
provider
provider_invoice_id
subscription_id
currency
subtotal
tax
total
amount_paid
amount_due
status
period_start
period_end
invoice_url
created_at
paid_at
```

Stripe remains authoritative for the external invoice.

---

# 87. Payment Record

```text
payment_records
---------------
id
billing_account_id
provider
provider_payment_intent_id
provider_charge_id
invoice_id
currency
amount
status
paid_at
failure_code
created_at
```

Do not store full card data.

---

# 88. PCI Scope

Use Stripe-hosted payment components/Checkout/Portal wherever practical.

Buildora AI should not receive/store:

- card number,
- CVC.

Store only provider references and safe payment metadata.

---

# 89. Refund Policy Architecture

Refund is a financial event.

Credits are a product entitlement.

They need deliberate coordination.

Examples:

## Subscription refund

Usually:

- record refund,
- apply subscription policy,
- do not retroactively alter historical usage ledger.

## Unused top-up refund

May:

```text
refund payment
+
revoke remaining unused credits
```

If some credits already consumed:

- partial refund,
- manual review,
- or policy-defined non-refundable consumed portion.

Do not allow silent negative credit balances unless explicitly supported.

---

# 90. Refund Record

```text
refund_records
--------------
id
payment_record_id
provider_refund_id
currency
amount
reason
status
created_at
```

Credit revocations reference the refund.

---

# 91. Chargebacks / Disputes

On dispute:

- flag billing account,
- notify admin,
- optionally suspend new high-cost AI/render actions,
- preserve user data access according to policy.

If a disputed top-up created credits:

- revoke unspent credits,
- review consumed amount.

Do not automatically destroy projects.

---

# 92. Coupons and Promotions

Keep commercial discount separate from product entitlements unless intentionally bundled.

Example:

```text
20% subscription discount
```

does not automatically change:

```text
30 Design Credits
```

unless promotion explicitly changes both.

Store promotion metadata for reporting.

---

# 93. Taxes

Tax calculation belongs to the payment/invoicing layer.

Potentially use Stripe Tax.

Buildora AI should store:

- tax status,
- tax amount,
- tax provider references,
- customer billing country/address metadata as legally appropriate.

Do not calculate tax using an LLM.

---

# 94. Currency

Plan prices are explicit per currency.

Avoid automatic real-time FX conversion in application code for subscription prices.

Example:

```text
PRO_USD_MONTHLY
PRO_GBP_MONTHLY
```

Customer credit quantities can remain same or differ by commercial strategy.

Provider AI costs may be measured in USD internally even when subscription is GBP.

Margin analytics must normalize currency using a controlled FX source/version if cross-currency analysis is required.

---

# 95. Gross Margin Architecture

Per organization/period calculate:

```text
recognized/collected revenue
-
variable provider cost
=
contribution
```

Variable costs may include:

- AI,
- render compute,
- embeddings,
- document OCR,
- third-party BIM conversion,
- variable storage/egress allocation.

Do not mix salaries/fixed overhead into product usage metering unless doing full accounting analytics.

---

# 96. AI Margin Metrics

Track:

```text
AI cost / organization
AI cost / active user
AI cost / project
AI cost / subscription
AI cost / Design Credit
AI cost / AI job type
AI cost by provider/model
```

Critical:

```text
AI cost as % of subscription revenue
```

---

# 97. Margin Guardrails

Example configurable thresholds:

```text
target variable gross margin: 70%+
warning: <65%
critical: <50%
```

Architecture should support alerts.

Do not automatically disable a legitimate customer solely because their margin is temporarily low.

Instead:

- inspect usage,
- tune routing,
- tune credits,
- contact enterprise-heavy users,
- adjust future plan versions.

---

# 98. Cost-Aware Model Routing

AI router can consider:

```text
task complexity
quality requirement
user plan
current fair-use budget
organization cost
model price
latency
```

But routing must not degrade correctness for safety-critical operations.

For ordinary chat:

```text
prefer cheaper capable model
```

For expert design:

```text
use required model tier
+
charge Design Credits
```

---

# 99. Monthly Cost Budget

Each plan version may include internal cost-control metadata.

Example:

```text
copilot_soft_cost_budget_usd
copilot_hard_abuse_budget_usd
```

These are internal controls.

They need not be advertised as customer dollars.

---

# 100. Plan Snapshot

At the start of a paid period, store an entitlement/plan snapshot.

Why?

If plan catalog changes during the month, Buildora AI can explain:

```text
what the customer was entitled to
```

for that period.

Conceptual:

```text
entitlement_snapshots
---------------------
id
organization_id
subscription_id
billing_period_id
plan_version_id
snapshot_json
created_at
```

---

# 101. Grandfathering

When Pro v2 launches:

Existing Pro v1 customers may remain on v1.

Do not mutate:

```text
PRO v1
```

to match v2.

Migration can be:

- automatic at renewal,
- opt-in,
- explicit migration campaign.

Record it.

---

# 102. Entitlement Overrides

Use explicit override records.

```text
entitlement_overrides
---------------------
id
organization_id
feature_code
override_value
valid_from
valid_to
reason
created_by
```

Examples:

- support compensation,
- enterprise contract,
- beta access.

Do not modify plan rows for one customer.

---

# 103. Billing Admin Capabilities

Admin UI should support:

```text
view billing account
view subscription
view plan version
view entitlement snapshot
view invoices
view payments
view credit wallets
view credit grants
view ledger
view AI usage
view provider cost
view margin
view webhook events
view reconciliation alerts
```

Privileged actions:

```text
grant/revoke credits
apply entitlement override            immediate, per organization
revoke entitlement override
register/retire a feature code        runtime catalogue edit
publish a new plan version            affects NEW subscribers only
migrate an organization to a plan version
retry reconciliation
suspend billing account
```

All privileged actions require audit logging **with a reason**.

## Feature catalogue admin screens (ADR-022)

```text
Feature catalogue      list codes · category · value type · default
                       ORPHANED badge where no check site is wired
                       register / retire a code

Plan editor            per plan version, set entitlement values
                       explicit warning: "this affects NEW subscribers only;
                       existing subscribers remain on their version"

Organization view      effective entitlements with their RESOLVED SOURCE
                       (override / add-on / plan / default)
                       apply or revoke an override, with reason

Audit                  who changed what entitlement, when, and why
```

Showing the **resolved source** of each entitlement is what makes support
answerable: *"they can use walkthrough because of an override applied on 3 March,
not because of their plan."*

---

# 104. Customer Billing UI

Recommended:

```text
Billing Overview
Current Plan
Renewal Date
Payment Status
Usage
AI Design Credits
Render Credits
Invoices
Payment Method
Upgrade/Downgrade
Buy Credits
Cancel Subscription
```

Do not show raw provider token cost unless product strategy deliberately includes it.

---

# 105. Usage UI

Example:

```text
AI Design Credits
18 / 30 remaining

Render Credits
14 / 20 remaining

AI Copilot
Included — normal usage

Storage
4.3 GB / 20 GB
```

Usage history:

```text
Sep 10
Advanced design concept
-3 credits

Sep 8
HD render
-1 render credit
```

---

# 106. Credit Explanation

The UI should explain:

```text
Design Credits are used for computationally intensive AI design generation and major AI modifications.
Normal manual editing and deterministic quantity/cost recalculation do not use Design Credits.
```

Predictability builds trust.

---

# 107. Credit Warning Thresholds

Configurable notifications:

```text
50%
80%
100%
```

Example:

```text
You have 6 AI Design Credits remaining.
```

Avoid surprising users only after a job fails.

---

# 108. Usage Notifications

Potential notifications:

- low Design Credits,
- low Render Credits,
- trial ending,
- payment failed,
- subscription renewed,
- downgrade scheduled,
- cancellation scheduled,
- top-up purchased.

Email/in-app notification should be event-driven.

---

# 109. Billing API Surface

Example:

```text
GET  /billing/overview
GET  /billing/plans
GET  /billing/usage
GET  /billing/invoices

POST /billing/checkout/subscription
POST /billing/checkout/top-up
POST /billing/portal

POST /billing/subscription/upgrade
POST /billing/subscription/downgrade
POST /billing/subscription/cancel
POST /billing/subscription/reactivate

POST /billing/webhooks/stripe
```

Internal/admin:

```text
POST /admin/billing/credits/grant
POST /admin/billing/credits/revoke
POST /admin/billing/reconcile
```

---

# 110. Internal Billing Services

Recommended interfaces:

```text
PlanCatalogService
EntitlementService
SubscriptionService
CreditWalletService
CreditReservationService
UsageMeterService
ProviderCostService
BillingOrderService
PaymentService
BillingReconciliationService
BillingAnalyticsService
```

Avoid:

```text
BillingService
```

with hundreds of unrelated methods.

---

# 111. Credit Reservation API

Conceptual application interface:

```ts
interface ReserveCreditsInput {
  organizationId: string;
  creditType: "AI_DESIGN" | "RENDER";
  quantity: number;
  operationCode: string;
  idempotencyKey: string;
  sourceType: string;
  sourceId: string;
}

interface CreditReservation {
  id: string;
  status: "RESERVED";
  reservedQuantity: number;
  expiresAt: Date;
}
```

---

# 112. Settlement API

```ts
interface SettleReservationInput {
  reservationId: string;
  consumedQuantity: number;
  usageEventId: string;
  idempotencyKey: string;
}
```

Rules:

```text
consumed <= reserved
```

unless a deliberately designed extension supports additional atomic reservation.

Do not silently allow overdraft.

---

# 113. Credit Balance Read Model

Customer-facing balance can be derived/cached:

```ts
interface CreditBalance {
  creditType: "AI_DESIGN" | "RENDER";
  available: number;
  reserved: number;
  expiringSoon: number;
  nextExpiryAt?: string;
}
```

Available means:

```text
unconsumed
-
reserved
```

---

# 114. Credit Transaction Example

Suppose Pro period grants:

```text
30 Design Credits
```

Ledger:

```text
GRANT +30
```

User begins Advanced Concept:

```text
RESERVE -3 available
```

Job succeeds:

```text
CONSUME 3
```

Balance:

```text
27 available
```

If job fails:

```text
RELEASE 3
```

Balance returns to:

```text
30 available
```

---

# 115. Plan Recognition Charging

For MVP, basic plan recognition may be included in the plan.

Internally still record:

- processing usage,
- AI/CV cost,
- pages/images.

Future product can introduce:

```text
recognition credits
```

without redesigning the usage architecture.

---

# 116. Report Generation Charging

Standard deterministic reports:

```text
0 credits
```

Potential premium one-off professional reports can be:

- included by plan,
- add-on,
- one-time order.

Do not route ordinary PDF export through AI credits unless AI generation is genuinely required.

---

# 117. Image/Render Generation

If Buildora AI later uses third-party generative image APIs for visualization:

record as provider usage components.

Customer-facing charge can remain:

```text
Render Credits
```

irrespective of which render provider is used.

---

# 118. Future True Usage-Based Billing

If Buildora AI later charges customers postpaid for actual metered usage:

Examples:

```text
$X per 100 AI compute units
$Y per render minute
$Z per GB storage
```

keep the Buildora AI internal usage ledger.

Do not depend only on a payment provider's meter.

The internal ledger is needed for:

- authorization,
- realtime usage UI,
- dispute investigation,
- provider reconciliation.

---

# 119. Stripe Usage-Based Billing Direction

Stripe currently provides usage-based billing and recommends its Metronome-based usage platform for new integrations that need capabilities such as:

- prepaid credits,
- high-volume events,
- flexible/tiered/dimensional pricing,
- enterprise commitments,
- real-time usage visibility.

Buildora AI does **not** need to adopt provider-side usage metering for MVP because its initial model is primarily:

```text
subscription + included internal credits + top-ups
```

If Buildora AI later introduces true postpaid overage billing, evaluate Stripe's current usage-based/Metronome capabilities rather than building invoice-grade metering from scratch.

The Buildora AI internal ledger remains necessary regardless.

---

# 120. Billing Events

Recommended internal events:

```text
billing.subscription.started
billing.subscription.renewed
billing.subscription.upgrade_scheduled
billing.subscription.upgraded
billing.subscription.downgrade_scheduled
billing.subscription.canceled
billing.subscription.payment_failed

billing.period.opened
billing.period.paid

billing.credit.granted
billing.credit.reserved
billing.credit.consumed
billing.credit.released
billing.credit.expired
billing.credit.adjusted

billing.topup.paid
billing.refund.completed

billing.entitlement.changed
billing.margin.threshold_exceeded
```

---

# 121. Outbox Pattern

For critical event publication:

```text
DB transaction
    ↓
business mutation
+
outbox row
    ↓
commit
    ↓
async publisher
```

This prevents:

```text
database committed
but event lost
```

Use where event reliability matters.

---

# 122. Billing Audit Events

Audit separately:

```text
admin credit grant
admin credit revoke
plan change
billing email change
subscription cancellation
payment method action
enterprise override
manual reconciliation action
refund approval
```

Record:

- actor,
- organization,
- before/after,
- reason,
- timestamp.

---

# 123. Security Boundaries

Only server-side billing code can:

- create Stripe checkout sessions,
- change subscriptions,
- grant credits,
- settle reservations,
- apply entitlement overrides.

Frontend sends intent.

It does not decide entitlement.

---

# 124. Webhook Security

Requirements:

- raw body where signature library requires it,
- Stripe signature verification,
- correct endpoint secret,
- timestamp tolerance according to provider SDK,
- no business processing before verification.

Return appropriate success after safe ingestion.

---

# 125. Frontend Cannot Unlock Features

Bad:

```ts
localStorage.plan = "PRO";
```

The API must check effective entitlements.

Frontend entitlement checks are only UX optimizations.

---

# 126. Credit API Authorization

Credits are organization resources.

Only authorized organization users may:

- view usage,
- purchase top-ups according to role,
- change subscription.

Recommended:

```text
Owner/Admin → manage billing
Member → use entitled features
Viewer → no billing changes
```

Exact roles configurable.

---

# 127. Billing Data Privacy

Do not expose to ordinary members:

- full provider payment details,
- internal AI provider cost,
- gross margin,
- admin adjustments unrelated to them.

Separate:

```text
customer billing view
admin financial view
```

---

# 128. Failure Scenario — Stripe Unavailable

If Stripe is temporarily unavailable:

Existing active customers should still be able to:

- open projects,
- edit,
- use already-granted credits,

subject to cached/current internal entitlement state.

Block:

- new checkout,
- plan change,
- top-up purchase.

Do not make the entire product unavailable.

---

# 129. Failure Scenario — AI Provider Unavailable

Billing remains available.

Reserved credits for failed jobs should:

```text
release
```

according to job policy.

Do not mark subscription/payment failure.

---

# 130. Failure Scenario — Webhook Processing Fails

Webhook record remains:

```text
FAILED / RETRYABLE
```

Worker retries.

Reconciliation later catches missed state.

Do not discard event after returning an internal error without recording it.

---

# 131. Failure Scenario — Duplicate AI Completion

Worker retry may report completion twice.

Settlement idempotency ensures:

```text
one reservation
→ one settlement
```

Never consume credits twice.

---

# 132. Failure Scenario — Provider Returns No Usage

Record:

```text
usage status = USAGE_PENDING / MISSING
```

Do not invent token counts.

The job can still settle customer-facing credits according to operation delivery.

Reconciliation can fill/correct provider-cost data later.

---

# 133. Failure Scenario — Rate Card Missing

Do not fail a completed user AI response solely because internal cost rate card is missing.

Record usage.

Flag:

```text
COST_PENDING
```

Admin reconciliation calculates cost once rate card is available.

Customer charge remains based on the operation's credit policy.

---

# 134. Failure Scenario — Negative Credit Race

Two jobs attempt to reserve final 3 credits.

Database transaction/locking must allow exactly one if only 3 remain.

The other returns:

```text
INSUFFICIENT_AI_CREDITS
```

Do not rely on a prior non-locking `GET balance`.

---

# 135. Failure Scenario — Payment Succeeds but Webhook Delayed

Browser:

```text
Activating subscription…
```

Background:

- polling/internal status,
- webhook eventually updates.

Do not grant duplicate temporary credits simply to hide the delay.

---

# 136. Failure Scenario — Webhook Delivered Twice

Unique `provider_event_id`.

Second delivery:

```text
already processed
→ return success
```

No duplicate grant.

---

# 137. Failure Scenario — Refund After Credits Used

Do not blindly reverse consumed credits into a negative wallet.

Use configured policy:

- partial refund,
- no refund of consumed value,
- admin review.

The architecture records both financial refund and product-credit adjustment separately.

---

# 138. Failure Scenario — Cancellation With Top-Up Credits

Top-up credits are independent grants.

Policy can allow them to remain after subscription cancellation if the terms say so.

However an operation may still require a base plan entitlement.

Example:

```text
User has 20 top-up Design Credits
but Free plan does not permit Expert Design
```

Credits alone do not imply feature entitlement.

Check:

```text
feature entitlement
AND
credit balance
```

---

# 139. Trial Abuse

Potential controls:

- verified identity/email,
- payment method for selected trials,
- organization/user/device risk signals,
- limited trial AI credits,
- no repeated trial grants after cancellation/re-sign-up.

Do not implement intrusive controls before evidence of abuse.

---

# 140. Credit Expiry Job

Scheduled:

```text
find grants with expires_at <= now
and remaining_quantity > 0
    ↓
create EXPIRE ledger entry
    ↓
remaining = 0
```

Idempotent.

Send optional expiry warning before expiration.

---

# 141. Renewal Credit Grant Job

Webhook-driven primary path.

Reconciliation-driven backup path.

Exactly once effect:

```text
billing period
+
credit type
+
grant category
```

unique.

---

# 142. Internal API — Entitlement Guard

Conceptual:

```ts
await entitlementService.require({
  organizationId,
  feature: "AI_DESIGN",
});
```

For limits:

```ts
await entitlementService.requireCapacity({
  organizationId,
  feature: "PROJECTS",
  requested: 1,
});
```

Do not let every module interpret plan JSON itself.

---

# 143. Internal API — Usage Guard

Conceptual:

```ts
const reservation =
  await creditReservationService.reserve({
    organizationId,
    creditType: "AI_DESIGN",
    quantity: 3,
    operationCode: "ADVANCED_CONCEPT",
    idempotencyKey: jobId,
    sourceType: "AI_JOB",
    sourceId: jobId,
  });
```

---

# 144. Internal API — Provider Usage

```ts
await providerUsageService.record({
  aiJobId,
  provider: "openai",
  providerModel,
  providerRequestId,
  inputTokens,
  cachedInputTokens,
  cacheWriteTokens,
  outputTokens,
  reasoningTokens,
  status: "COMPLETED",
});
```

Then:

```text
cost calculator
```

uses rate-card version.

---

# 145. Internal API — Settlement

```ts
await creditReservationService.settle({
  reservationId,
  consumedQuantity: 3,
  usageEventId,
  idempotencyKey:
    `settle:${jobId}`,
});
```

---

# 146. Database Constraints

Important constraints:

```text
billing_accounts.organization_id UNIQUE

billing_webhook_events
(provider, provider_event_id) UNIQUE

credit_wallets
(organization_id, credit_type) UNIQUE

top-up credit grant
(source_type, source_id, wallet_id) UNIQUE

provider usage
(provider, provider_request_id) UNIQUE where available

reservation
(idempotency_key, organization_id) UNIQUE

settlement idempotency unique

subscription provider ID unique
invoice provider ID unique
payment provider ID unique
```

Database constraints are the final defense against duplicate side effects.

---

# 147. Transaction Boundaries

Use one transaction for:

```text
reserve credits
+
ledger entries
+
grant remaining update
```

Use one transaction for:

```text
settle
+
consume
+
release remainder
+
usage link
```

Do not hold DB transactions open while calling Stripe or OpenAI.

---

# 148. Outbound Payment Provider Calls

Pattern:

```text
create internal operation record
    ↓
commit
    ↓
call Stripe with idempotency key
    ↓
persist provider result
```

If network response is uncertain:

```text
query provider by idempotency/reference
```

instead of creating a new uncorrelated operation.

---

# 149. Billing Saga / Workflow

Complex plan changes can use an explicit application workflow.

Example upgrade:

```text
validate request
→ create plan-change operation
→ preview Stripe proration
→ customer confirms
→ update Stripe
→ receive/verify payment outcome
→ update subscription mirror
→ update entitlements
→ grant credit delta
→ audit
```

Do not place all of this inside one controller method.

---

# 150. Temporal Use

Temporal is appropriate for:

- long reconciliation,
- annual/monthly usage grant workflows,
- large billing migrations,
- render billing lifecycle,
- complex enterprise invoicing later.

Ordinary webhook ingestion can remain queue/application based unless durable orchestration adds value.

---

# 151. Usage Aggregates

Raw usage events remain immutable.

Create aggregates for dashboards.

```text
usage_aggregates
----------------
organization_id
period
meter_code
quantity
provider_cost
updated_at
```

Aggregates are reconstructable.

Do not make aggregate rows the only usage history.

---

# 152. Margin Aggregates

Example:

```text
organization_margin_periods
---------------------------
organization_id
period_start
period_end

subscription_revenue
topup_revenue

ai_provider_cost
render_cost
storage_variable_cost

contribution_margin
margin_percent
status
```

Use for internal operations, not customer-facing invoices.

---

# 153. Cost Anomaly Detection

Alerts:

```text
single AI job cost > threshold
organization daily AI cost > threshold
provider model mix unexpectedly expensive
render job runaway
token usage sudden spike
```

Do not automatically bill the customer extra unless terms explicitly define overage.

---

# 154. AI Request Budget

Before provider request:

```text
estimated max input
estimated max output
model
tools
```

set provider limits such as:

- max output tokens,
- request timeout,
- tool limits.

This is both reliability and cost control.

---

# 155. Context Cost Control

AI Gateway should reduce unnecessary context by:

- retrieving targeted project objects,
- summarizing old conversation state,
- using prompt caching,
- not sending entire Building Model unnecessarily,
- selecting only relevant documents.

Provider usage fields for cached tokens should be stored so caching effectiveness can be measured.

---

# 156. Cache Economics

Track:

```text
cache hit tokens
cache-write tokens
uncached tokens
```

Metrics:

```text
cache hit ratio
cached token cost savings
```

Do not expose these operational details as customer billing units.

---

# 157. Free Plan Economics

Free users require strict cost controls.

Recommended:

- small AI allowance,
- cheapest capable model routing,
- limited expensive generation,
- no high-cost render or heavily limited render,
- rate limits,
- project/storage limits.

Free users still use the same usage architecture.

Do not create a separate unmetered code path.

---

# 158. Home Plan Economics

Home plan can emphasize:

- lower project limit,
- simple AI generation credits,
- conceptual cost,
- standard reports.

Internal cost guardrails stricter than Pro.

---

# 159. Pro Plan Economics

Pro can include:

```text
professional QS
IFC
higher AI credits
higher render credits
larger storage
```

Margin monitoring is critical because professional users can be heavy AI users.

---

# 160. Business Plan Economics

Business may support:

- pooled team usage,
- more seats,
- shared rate libraries,
- higher storage,
- organization admin,
- higher credits.

Usage belongs to organization wallet, not individual wallets, unless future policy adds per-seat sublimits.

---

# 161. Enterprise Economics

Use negotiated controls:

```text
contracted allowance
overage
minimum commitment
seat count
custom rates
invoice terms
```

Do not fake Enterprise using a huge hardcoded Pro plan.

---

# 162. Customer Usage Attribution

Every usage record should include where possible:

```text
organization
user
project
feature
job
operation
```

This enables:

- support investigation,
- team usage,
- margin analysis,
- enterprise chargeback reports.

---

# 163. Billing Ownership Transfer

If organization owner changes:

Billing account remains tied to the organization.

Do not create a new Stripe customer solely because ownership changes.

Billing-contact details can be updated separately.

---

# 164. Organization Deletion

Do not immediately delete billing records.

Financial/audit data may require retention.

Application data deletion and billing-record retention should follow legal/accounting policy.

Use tombstoning/anonymization where appropriate rather than destroying required financial records.

---

# 165. Customer Account Merge

Avoid automatic organization merges.

If ever supported, billing/credit merge requires explicit operational workflow.

Do not simply add balances from two wallets without retaining provenance.

---

# 166. Data Retention

Define separate retention policies for:

- invoices/payments,
- credit ledger,
- provider usage,
- audit logs,
- project usage,
- deleted organizations.

Billing/financial records may require longer retention than application data.

---

# 167. Observability

Every billing operation should emit structured logs/spans.

Context:

```text
traceId
organizationId
billingAccountId
subscriptionId
orderId
reservationId
invoiceId
providerEventId
```

Never log:

- card details,
- secrets,
- full webhook secrets,
- auth tokens.

---

# 168. Billing Metrics

Operational:

```text
webhook failure rate
webhook lag
payment failure rate
checkout conversion
reservation failure
reservation expiry count
reconciliation differences
```

Commercial:

```text
MRR
ARR
ARPU
trial conversion
paid conversion
churn
expansion
credit pack revenue
```

Cost:

```text
AI cost
render cost
cost / paid org
gross margin
```

---

# 169. Billing Alerts

Critical:

```text
webhook backlog
webhook signature failures spike
duplicate grant attempt
negative-balance invariant
Stripe reconciliation mismatch
failed payment spike
AI provider cost spike
gross margin critical
```

---

# 170. Invariants

The system must maintain:

```text
No consumed credit without a valid source/grant.

No credit grant from the same paid order twice.

No subscription-period grant twice.

No reservation settles twice.

No balance becomes negative unless an explicit overage policy permits it.

No paid entitlement activates solely from browser callback.

No other tenant can read billing data.

No AI retry double-charges a customer.

No refund silently rewrites historical usage.

No current rate card rewrites historical provider cost.
```

---

# 171. Billing Test Strategy

## Unit tests

- entitlement resolution
- grant consumption order
- credit reservation
- settlement
- expiry
- upgrade credit delta
- downgrade scheduling
- provider cost calculation
- plan version selection

## Integration tests

- PostgreSQL concurrency
- duplicate webhook
- duplicate order
- duplicate reservation
- tenant isolation
- transaction rollback

## Stripe test-mode integration

- checkout
- renewal
- failed payment
- upgrade
- downgrade
- cancellation
- refund

## E2E

- subscribe
- credits appear
- consume credits
- top-up
- cancel
- reactivate

---

# 172. Concurrency Test

Initial balance:

```text
3
```

Run simultaneously:

```text
reserve 3
reserve 3
```

Expected:

```text
one succeeds
one receives INSUFFICIENT_AI_CREDITS
```

Balance invariant holds.

---

# 173. Duplicate Webhook Test

Send identical Stripe event twice.

Expected:

```text
one processed
one recognized as duplicate
one credit grant total
```

---

# 174. Provider Retry Test

One AI job:

```text
provider call 1 fails
provider call 2 succeeds
```

Expected customer billing:

```text
one Design Credit charge
```

Expected cost accounting:

```text
record both provider calls/cost where measurable
```

---

# 175. Renewal Test

Invoice paid delivered twice.

Expected:

```text
one billing period
one monthly Design grant
one monthly Render grant
```

---

# 176. Failed Renewal Test

Expected:

```text
no new paid-period credits
status enters configured past-due/grace behaviour
existing unexpired top-ups remain according to policy
```

---

# 177. Downgrade Test

Pro has 10 projects.

Downgrade plan supports 3.

Expected:

```text
no projects deleted
existing data accessible
new project creation blocked if above limit
downgrade activates next period
```

---

# 178. Refund Test

Top-up:

```text
25 granted
10 consumed
15 remaining
```

Refund attempt tests policy:

- revoke remaining 15,
- refund eligible amount according to policy,
- preserve historical 10 usage.

No silent negative wallet.

---

# 179. Billing API Errors

Stable codes:

```text
BILLING_ACCOUNT_NOT_FOUND
SUBSCRIPTION_REQUIRED
SUBSCRIPTION_INACTIVE
PLAN_NOT_AVAILABLE
PLAN_CHANGE_NOT_ALLOWED
PAYMENT_REQUIRED
PAYMENT_PROCESSING
INSUFFICIENT_AI_CREDITS
INSUFFICIENT_RENDER_CREDITS
CREDIT_RESERVATION_CONFLICT
CREDIT_RESERVATION_EXPIRED
BILLING_PROVIDER_UNAVAILABLE
BILLING_WEBHOOK_INVALID
USAGE_LIMIT_REACHED
```

Use RFC 9457 Problem Details.

---

# 180. Example Error

```json
{
  "type": "https://buildora.example/problems/insufficient-ai-credits",
  "title": "Insufficient AI Design Credits",
  "status": 402,
  "detail": "This operation requires 3 AI Design Credits. You currently have 2 available.",
  "code": "INSUFFICIENT_AI_CREDITS",
  "traceId": "..."
}
```

`402 Payment Required` may be used for commercial credit exhaustion if adopted consistently; alternatively Buildora AI can use a documented `403` policy. Choose once and standardize.

---

# 181. API Idempotency

Endpoints requiring idempotency:

```text
create checkout
purchase top-up
reserve credits
settle credits
admin adjustment
subscription plan change
```

Client sends:

```text
Idempotency-Key
```

Buildora AI records operation result.

Do not allow a double-click to create two top-up orders.

---

# 182. Billing Dashboard — Customer

Recommended sections:

```text
Current Plan
Plan Features
Renewal
Payment Method
AI Design Credits
Render Credits
Usage History
Invoices
Top-Up Packs
Upgrade
Cancel
```

---

# 183. Billing Dashboard — Admin

Recommended:

```text
Account
Subscription
Plan version
Entitlements
Stripe state
Invoices
Payments
Refunds
Credit wallets
Grants
Reservations
Ledger
AI usage
Provider cost
Margin
Webhook inbox
Reconciliation
Admin adjustments
```

---

# 184. Plan Change Preview

Before upgrade:

```text
Current plan
New plan
Effective date
Prorated charge
New monthly credits
Features gained
```

Before downgrade:

```text
Effective next renewal
Features lost
New limits
Current over-limit resources
```

---

# 185. Customer Transparency

Do not surprise users.

Always show before an expensive AI operation:

```text
Uses 3 AI Design Credits
```

where the charge is deterministic.

For variable-credit operations, show:

```text
Up to X credits
```

and reserve max amount.

---

# 186. No Credits for Deterministic Work

Normal:

```text
2D editing
3D sync
quantity calculation
BOQ recalculation
cost recalculation
```

should not consume AI Design Credits.

This is both good UX and aligned with the Buildora AI architecture.

---

# 187. Plan/Operation Charge Table

Create versioned configuration:

```text
operation_charge_rules
----------------------
id
operation_code
credit_type
quantity
plan_scope
effective_from
effective_to
status
```

Example:

```text
AI_STANDARD_CONCEPT     AI_DESIGN  2
AI_ADVANCED_CONCEPT     AI_DESIGN  3
AI_EXPERT_STUDY         AI_DESIGN  5
RENDER_HD                RENDER     1
RENDER_4K                RENDER     2
```

Do not put:

```ts
if (expert) credits = 5;
```

deep inside the AI code.

---

# 188. Charge Rule Versioning

An AI job stores:

```text
charge_rule_id
```

so Buildora AI can later explain why:

```text
this action cost 3 credits
```

even if the rule is now 4.

---

# 189. Subscription Grant Rule Versioning

Likewise plan version defines:

```text
monthly included Design Credits
monthly Render Credits
```

Historical periods remain reproducible.

---

# 190. Revenue Recognition Note

Buildora AI operational billing records are not automatically a full accounting ledger.

MRR/ARR dashboards can be operational metrics.

For statutory accounting/revenue recognition:

- use accounting system/invoice records,
- consult accounting requirements.

Do not treat AI-generated finance analysis as accounting authority.

---

# 191. Future Accounting Integration

Potential integrations:

```text
Xero
QuickBooks
NetSuite
```

Export:

- customer
- invoice
- payment
- tax
- refund.

Keep the billing domain independent from a specific accounting product.

---

# 192. Future Overage Architecture

Enterprise/Business may later allow:

```text
included credits
+
postpaid overage
```

Flow:

```text
included grant exhausted
    ↓
if overage_enabled
    ↓
usage continues
    ↓
overage usage recorded
    ↓
metered invoice
```

This is different from allowing wallet balance to become negative.

Use a separate:

```text
overage_usage
```

meter.

---

# 193. Overage Guardrails

Enterprise contract should define:

```text
overage enabled
unit price
monthly cap
alert thresholds
hard stop
```

Never accidentally enable unlimited postpaid AI usage.

---

# 194. Customer Cost Caps

For overage plans allow:

```text
monthly spend cap
email alerts
hard stop
```

This is especially important for AI usage.

---

# 195. Stripe Metronome Decision Point

For future postpaid usage:

Evaluate provider-side usage billing when requirements include:

- large event volume,
- complex enterprise contracts,
- prepaid/committed credits,
- dimensional pricing,
- automated overage invoicing.

Even then:

```text
Buildora AI usage ledger
```

remains the realtime product-control source.

---

# 196. Recommended MVP Billing Scope

Build first:

```text
Free / Home / Pro / Business plan catalog
Stripe subscriptions
Stripe Checkout
Stripe Billing Portal
organization billing account
internal subscription mirror
entitlements
AI Design wallet
Render wallet
monthly grants
top-up purchase
reserve / settle / release
AI provider usage records
provider cost calculation
customer usage UI
admin usage UI
webhook inbox
daily reconciliation
payment-failure/grace behaviour
cancel/reactivate
upgrade now
downgrade next period
```

Delay:

```text
true postpaid overage
complex seat billing
enterprise commits
custom invoicing
usage-based Stripe Metronome
multi-currency FX billing logic
marketplace payouts
revenue recognition engine
```

---

# 197. Implementation Order

Recommended:

## Phase 1

```text
Plan Catalog
Entitlements
Billing Account
Subscription Mirror
```

## Phase 2

```text
Stripe Checkout
Webhooks
Invoices
Payments
```

## Phase 3

```text
Credit Wallet
Grants
Ledger
Reservation
Settlement
```

## Phase 4

```text
AI Usage
Provider Cost
Rate Cards
Margin Analytics
```

## Phase 5

```text
Top-Ups
Upgrade/Downgrade
Cancel/Reactivate
Payment Failure
```

## Phase 6

```text
Customer Billing UI
Admin Billing UI
Reconciliation
Alerts
```

---

# 198. Module Structure

Suggested:

```text
apps/api/src/modules/billing/
├── billing.module.ts
├── catalog/
│   ├── application/
│   ├── domain/
│   └── infrastructure/
├── entitlements/
├── subscriptions/
├── credits/
│   ├── application/
│   │   ├── reserve-credits.use-case.ts
│   │   ├── settle-reservation.use-case.ts
│   │   ├── release-reservation.use-case.ts
│   │   └── grant-credits.use-case.ts
│   ├── domain/
│   ├── ports/
│   └── infrastructure/
├── usage/
├── provider-cost/
├── orders/
├── invoices/
├── payments/
├── webhooks/
├── reconciliation/
├── analytics/
└── admin/
```

---

# 199. Domain Package

Recommended:

```text
packages/billing-domain/
```

Contains:

- credit types,
- entitlement types,
- wallet rules,
- grant consumption ordering,
- reservation state machine,
- settlement invariants,
- subscription internal states.

No Stripe SDK.

No NestJS.

No Drizzle.

---

# 200. Stripe Adapter

Only Stripe integration package/module imports Stripe SDK.

Conceptual interface:

```ts
abstract class BillingProvider {
  createSubscriptionCheckout(...): Promise<...>;
  createTopUpCheckout(...): Promise<...>;
  previewPlanChange(...): Promise<...>;
  applyPlanChange(...): Promise<...>;
  cancelSubscription(...): Promise<...>;
  createBillingPortal(...): Promise<...>;
}
```

Implementation:

```text
StripeBillingProvider
```

---

# 201. Provider Independence

Do not let application code pass around:

```text
Stripe.Subscription
Stripe.Invoice
```

Map provider objects to internal DTOs.

This makes testing easier and prevents Stripe types from contaminating the domain.

---

# 202. Billing Webhook Processor

Architecture:

```text
StripeWebhookController
    ↓
verify signature
    ↓
WebhookInboxRepository.insertIfNew
    ↓
BillingWebhookProcessor
    ↓
event-specific handler
```

Event handlers call application use cases.

Do not put all event logic inside the controller.

---

# 203. Webhook Handler Example Responsibilities

`invoice.paid` handler:

```text
map invoice
update invoice mirror
update payment state
confirm billing period
grant period credits exactly once
refresh entitlement snapshot
emit billing.subscription.renewed if appropriate
```

---

# 204. Subscription Updated Handler

Should:

```text
refresh provider status
period dates
cancel_at_period_end
plan/provider price mapping
```

Do not issue duplicate credits merely because subscription object changed.

Credits come from the defined grant trigger.

---

# 205. Top-Up Paid Handler

```text
find internal order
verify amount/currency/provider refs
mark order paid exactly once
create grant exactly once
audit
```

---

# 206. Billing Portal

Use Stripe Billing Portal for:

- payment method,
- invoice access,
- selected subscription management,

where it matches product requirements.

Buildora AI may still need custom plan-change UI to coordinate:

- credits,
- entitlements,
- downgrade policy.

Do not allow a portal setting that bypasses Buildora AI internal plan-change rules unless webhook/reconciliation fully supports it.

---

# 207. Testing With Stripe

Use:

- Stripe test mode,
- Stripe CLI,
- test clocks where useful,
- webhook fixtures.

Test:

```text
monthly renewal
annual renewal
trial ending
payment failure
payment recovery
upgrade
downgrade
cancel
reactivate
refund
```

---

# 208. Time Handling

Billing logic must use explicit UTC timestamps.

Do not calculate period expiry using the user's browser clock.

Use provider/internal period timestamps.

---

# 209. Plan Limit Checks

Project creation:

```text
EntitlementService.requireCapacity(
  PROJECTS,
  currentCount,
  +1
)
```

Member invitation:

```text
TEAM_MEMBERS
```

Upload:

```text
STORAGE_GB
```

AI design:

```text
AI_DESIGN feature
+
Design Credit reservation
```

---

# 210. Feature vs Credit

Important:

```text
ENTITLEMENT
```

answers:

```text
May this organization use this feature?
```

Credit answers:

```text
Does it have enough consumption allowance?
```

You need both.

Example:

```text
Free user owns promotional Design Credits
but AI_DESIGN is not enabled
```

Result:

```text
blocked
```

unless promotion also grants the feature.

---

# 211. Billing Read Models

Do not expose raw ledger tables directly to frontend.

Create read models:

```text
BillingOverview
PlanComparison
CreditBalance
UsageHistory
InvoiceList
SubscriptionChangePreview
```

---

# 212. Customer Billing Overview Contract

Example:

```json
{
  "plan": {
    "code": "PRO",
    "name": "Pro",
    "status": "active",
    "renewsAt": "..."
  },
  "usage": {
    "aiDesign": {
      "available": 18,
      "reserved": 3,
      "monthlyIncluded": 30
    },
    "render": {
      "available": 14,
      "reserved": 0,
      "monthlyIncluded": 20
    }
  },
  "limits": {
    "projects": null,
    "teamMembers": 5,
    "storageGb": 20
  }
}
```

`null` can represent unlimited if documented.

---

# 213. Subscription Change Preview Contract

```json
{
  "fromPlan": "HOME",
  "toPlan": "PRO",
  "effective": "immediate",
  "currency": "USD",
  "amountDueNow": "17.42",
  "nextRecurringAmount": "49.00",
  "nextBillingAt": "...",
  "creditAdjustment": {
    "aiDesign": 20,
    "render": 15
  }
}
```

The actual money preview should be based on provider preview.

---

# 214. Credit History Contract

```json
{
  "items": [
    {
      "id": "...",
      "type": "CONSUME",
      "creditType": "AI_DESIGN",
      "quantity": 3,
      "description": "Advanced AI concept",
      "occurredAt": "..."
    }
  ]
}
```

Do not expose internal grant IDs unless needed.

---

# 215. Billing Security Tests

Mandatory:

```text
org A cannot read org B billing
member cannot change plan if role not permitted
viewer cannot buy credits if policy forbids
client cannot forge credit balance
client cannot send negative top-up quantity
webhook without valid signature rejected
duplicate webhook does not duplicate effect
```

---

# 216. Financial Precision

All currency:

```text
PostgreSQL NUMERIC
```

or provider integer minor units where appropriate.

Do not use JavaScript floating point for authoritative invoice/refund arithmetic.

Provider token costs should also use exact decimal arithmetic.

---

# 217. Token Count Precision

Token counts:

```text
BIGINT / integer depending expected range
```

Do not store tokens as floating point.

Cost formula uses decimal.

---

# 218. Rounding

Define rounding by context.

Provider cost:

- calculate at high precision,
- round only for display/reporting.

Customer invoice:

- follow payment provider/currency minor-unit rules.

Do not use UI display rounding in billing calculations.

---

# 219. Credit Quantity Type

If all credits are integer units:

```text
integer
```

If future fractional credits are desired:

use explicit fixed decimal.

Recommendation:

```text
keep Design/Render Credits integer initially
```

This simplifies customer understanding and ledger correctness.

---

# 220. Migration Strategy

Billing migrations deserve extra review.

Never:

- drop ledger columns casually,
- change historical plan entitlements in place,
- overwrite monetary values.

Use additive/versioned migrations.

---

# 221. Backups

Billing data is critical.

Verify backup/PITR includes:

- subscriptions,
- grants,
- ledger,
- usage,
- invoices,
- payments,
- webhook inbox.

Do not rely solely on Stripe to reconstruct internal credit history.

---

# 222. Disaster Recovery

If internal billing DB is restored:

run reconciliation against:

- Stripe,
- usage/job records,

before normal high-cost usage resumes if recovery window creates uncertainty.

---

# 223. Support Investigation Flow

Support can answer:

```text
Why did my credits decrease?
```

using:

```text
wallet
→ ledger
→ reservation
→ AI job
→ provider usage
→ project/action
```

No guesswork.

---

# 224. Credit Dispute

If customer disputes an internal credit charge:

admin can inspect:

```text
operation
reservation
completion
result delivery
usage
provider calls
```

If refunding credits:

create:

```text
REFUND/ADJUSTMENT entry
```

Never delete original consumption.

---

# 225. Billing Change Audit

Plan-change operations should record:

```text
requested by
old plan
new plan
preview
provider operation
effective date
credit adjustment
final state
```

---

# 226. Feature Flag Safety

New billing behaviour should be feature-flagged when risk is high.

Examples:

```text
BILLING_TOPUPS
BILLING_ANNUAL
BILLING_PRORATED_UPGRADE
BILLING_OVERAGE
```

Roll out gradually.

---

# 227. Kill Switches

Operational switches:

```text
disable new AI design jobs
disable renders
disable checkout
disable top-ups
disable plan changes
```

Existing project access should remain available where possible.

---

# 228. Margin-Safe Default Behaviour

If cost metadata/config is uncertain:

- allow low-cost core deterministic product,
- fail closed for very expensive optional AI/render actions only when required to protect against runaway spend.

Do not take down core editing because a rate card is missing.

---

# 229. Product Analytics vs Billing Ledger

Product analytics can be approximate/eventual.

Billing ledger cannot.

Do not calculate customer credit balances from an analytics platform.

---

# 230. Recommended ADRs

Create ADRs for:

```text
ADR — Customer credits vs raw token billing
ADR — Organization-level billing ownership
ADR — Upgrade immediately / downgrade at renewal
ADR — Credit expiry and rollover policy
ADR — AI Design credit charge table
ADR — Annual-plan monthly usage grants
ADR — Payment-failure grace policy
ADR — Top-up refund policy
ADR — Future overage strategy
```

---

# 231. Initial Commercial Policy Recommendation

For first paid Buildora AI release:

## Free

- limited projects,
- small/trial AI usage,
- no or minimal premium rendering,
- cost-controlled AI routing.

## Home

- small project limit,
- AI Design Credits,
- basic reports,
- conceptual estimates.

## Pro

- approximately $49/month,
- professional QS/BOQ/cost,
- IFC,
- higher AI allowance,
- Design Credits,
- Render Credits.

## Business

- more team members,
- pooled credits,
- shared libraries,
- larger storage,
- organization management.

## Enterprise

- custom contract.

All exact quantities remain configurable.

---

# 232. Recommended Credit Policy

Subscription credits:

```text
refresh each usage period
non-rollover
```

Top-up credits:

```text
persist longer
consumed after sooner-expiring grants
```

Promotional:

```text
explicit expiry
```

Upgrade:

```text
immediate after payment
grant configured credit delta
```

Downgrade:

```text
next renewal
```

Cancellation:

```text
end of current paid period
```

---

# 233. Recommended Payment Failure Policy

Configurable initial recommendation:

```text
Day 0:
PAST_DUE
warn user

Short grace:
core access remains
high-cost premium compute may be limited

After grace:
SUSPENDED paid entitlements
projects remain viewable
billing/update-payment remains accessible
```

Do not hardcode exact days until commercial policy is approved.

---

# 234. Recommended AI Billing Policy

```text
Manual edit                  0 credits
2D/3D sync                   0
QS                           0
BOQ                          0
Cost recalc                  0

Normal Copilot               included/fair-use with internal cost guardrails

AI design generation         Design Credits
Major AI modification        Design Credits

Photoreal server rendering   Render Credits
```

---

# 235. Recommended Gross-Margin Control

Per paid billing period monitor:

```text
subscription + top-up revenue
vs
AI/render variable cost
```

Target operating policy can begin around:

```text
70%+ variable gross margin
```

but actual thresholds should be adjusted using real usage data.

Do not compromise product correctness merely to hit a short-term model-cost target.

---

# 236. Buildora AI Billing Lifecycle — Full View

```text
                    SIGN UP
                       │
                       ▼
                    FREE
                       │
             Checkout Subscription
                       │
                       ▼
              PAYMENT CONFIRMED
                       │
                       ▼
                    ACTIVE
                       │
             ┌─────────┼─────────┐
             │         │         │
             ▼         ▼         ▼
          USAGE     UPGRADE    TOP-UP
             │         │         │
             ▼         ▼         ▼
         RESERVE    PROVIDER    ORDER
             │       CHANGE       │
             ▼         │          ▼
          ACTION       ▼       PAYMENT
             │     ENTITLEMENT     │
             ▼        UPDATE       ▼
          SETTLE                  GRANT
             │
             ▼
      COST / MARGIN RECORD

Renewal:
INVOICE → PAYMENT → PERIOD → GRANTS

Failure:
PAYMENT FAILED → PAST_DUE → GRACE → SUSPENDED

Exit:
CANCEL AT PERIOD END → FREE / RETAINED DATA
```

---

# 237. Final Billing Rules

Buildora AI billing must always preserve these rules:

1. **Customer credits are not raw AI tokens.**
2. **Raw provider usage is still recorded precisely.**
3. **Stripe is payment authority; Buildora AI is entitlement/usage authority.**
4. **Credits use an auditable ledger.**
5. **Expensive operations reserve before execution.**
6. **Retries never double-charge customers.**
7. **Webhook retries never double-grant credits.**
8. **Plans and charge rules are versioned.**
9. **Downgrades never delete customer data.**
10. **AI provider pricing changes never rewrite historical usage cost.**
11. **Manual/deterministic editing does not consume AI credits.**
12. **Authorization and entitlement checks happen server-side.**
13. **Customer billing state never depends on browser redirects alone.**
14. **Provider outages should not disable the deterministic core product.**
15. **All monetary arithmetic is exact/decimal-safe.**
16. **All important billing effects are idempotent.**
17. **All admin adjustments are auditable.**
18. **Billing records can be reconciled against providers.**
19. **Usage data can explain every customer charge.**
20. **The system is designed for predictable pricing and measurable margin.**

---

# 238. Implementation Checklist

Before calling the first billing module production-ready:

- [ ] billing accounts
- [ ] plan catalog
- [ ] plan versions
- [ ] price versions
- [ ] entitlements
- [ ] subscription mirror
- [ ] billing periods
- [ ] Stripe Checkout
- [ ] Stripe webhook signature verification
- [ ] webhook inbox/idempotency
- [ ] invoices mirror
- [ ] payments mirror
- [ ] AI Design wallet
- [ ] Render wallet
- [ ] grants
- [ ] ledger
- [ ] reservations
- [ ] settlement
- [ ] expiration
- [ ] top-up orders
- [ ] provider usage records
- [ ] provider rate cards
- [ ] provider cost calculation
- [ ] AI usage linkage
- [ ] customer usage page
- [ ] billing page
- [ ] invoice page
- [ ] admin billing page
- [ ] plan upgrade
- [ ] scheduled downgrade
- [ ] cancel/reactivate
- [ ] payment-failure policy
- [ ] daily reconciliation
- [ ] margin reporting
- [ ] alerts
- [ ] concurrency tests
- [ ] webhook duplicate tests
- [ ] tenant-isolation tests
- [ ] refund policy
- [ ] backup/restore validation
- [ ] billing ADRs

---

# 239. Current External Platform Notes

As of September 2026:

- Stripe supports recurring subscription billing and subscription prorations; changing subscription prices/items can create prorated charges/credits, and Stripe supports previewing those prorations before applying them.
- Stripe currently recommends its Metronome-based usage platform for new integrations requiring advanced usage-based billing features such as prepaid credits, high-volume event ingestion, enterprise commitments, dimensional pricing, and real-time usage visibility. Buildora AI's MVP does not need to depend on that because customer-facing Design/Render credits can be maintained internally while Stripe handles subscription/top-up payments.
- OpenAI's Responses API exposes structured usage including input tokens, output tokens, total tokens, cached-input details, cache-write details, and reasoning-token details. Buildora AI should preserve these provider usage fields for internal cost and margin analysis.

Always re-check current Stripe/OpenAI documentation before implementing provider-specific billing behaviour.

---

# 240. Source References

Stripe:

- Usage-based billing  
  https://docs.stripe.com/billing/subscriptions/usage-based

- Subscription prorations  
  https://docs.stripe.com/billing/subscriptions/prorations

- Webhooks  
  https://docs.stripe.com/webhooks

- Idempotent requests  
  https://docs.stripe.com/api/idempotent_requests

OpenAI:

- Responses API  
  https://developers.openai.com/api/reference/resources/responses

- Model/usage documentation  
  https://developers.openai.com/api/

---

# 241. Final Architecture Principle

The correct Buildora AI billing model is:

```text
CUSTOMER PLAN
     │
     ▼
ENTITLEMENTS
     │
     ├───────────────┐
     ▼               ▼
FEATURE ACCESS    CREDIT WALLETS
                     │
                     ▼
                 RESERVATION
                     │
                     ▼
                PRODUCT ACTION
                     │
              ┌──────┴──────┐
              ▼             ▼
             AI           RENDER
              │             │
              └──────┬──────┘
                     ▼
               PROVIDER USAGE
                     │
                     ▼
               ACTUAL COST
                     │
                     ▼
                 SETTLEMENT
                     │
                     ▼
              MARGIN ANALYTICS
```

while:

```text
STRIPE
```

handles:

```text
payments
invoices
subscription money
refunds
tax/payment-provider concerns
```

This separation gives Buildora AI:

- predictable customer pricing,
- provider flexibility,
- reliable entitlements,
- auditable usage,
- safe concurrency,
- measurable AI economics,
- and a clear path from a $49 Pro SaaS product to future enterprise usage billing.
