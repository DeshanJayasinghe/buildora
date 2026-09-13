# Buildora AI — Comprehensive Development & Production Costing Plan

> **File name:** `Buildora_AI_Development_Production_Costing_Plan.md`
>
> **Purpose:** Master financial and operating cost model for developing, launching, and scaling Buildora AI.
>
> **Applies to:** solo-development costs, development tooling, staging, production infrastructure, AI usage, rendering, storage, authentication, billing fees, observability, workflows, licensing, contingency, gross-margin targets, budget controls and scaling thresholds.
>
> **Currency:** USD unless a row explicitly uses GBP.
>
> **Pricing reference date:** 11 September 2026.
>
> **Important:** Provider prices change. Every externally priced service must be represented in Buildora AI planning/configuration as a variable rather than embedded permanently in business logic. Re-check pricing before purchasing commitments or changing plans.
>
> **Important:** Taxes, VAT, foreign-exchange charges, corporate overheads and salaries are excluded unless explicitly stated.

---

# 1. Executive Costing Strategy

Buildora AI should be financed and operated in four cost stages:

```text
STAGE A
Solo Development
Local-first / very low cloud burn

        ↓

STAGE B
Cloud Development + Staging
Real deployment environment

        ↓

STAGE C
Paid Beta / Early Production
Production redundancy + billing + monitoring

        ↓

STAGE D
Growth Production
Usage-driven scaling
```

The financial objective is:

```text
Minimize fixed cost before revenue
        +
Measure every variable AI/render cost
        +
Maintain 70%+ target variable gross margin
        +
Scale infrastructure from evidence
```

Do not spend production-scale infrastructure money during early product development.

---

# 2. Cost Categories

Buildora AI costs fall into seven categories.

## 2.1 Development Tooling

Examples:

```text
Claude Pro
Codex / ChatGPT plan
Antigravity Pro
GitHub
IDE/editor tools
design tools
```

These are mostly fixed monthly development costs.

---

## 2.2 Core Infrastructure

Examples:

```text
AWS ECS/Fargate
Application Load Balancer
network/NAT
PostgreSQL
Redis
Temporal Cloud
Cloudflare
R2
monitoring
```

---

## 2.3 Variable AI

Examples:

```text
OpenAI input tokens
cached input
output/reasoning tokens
embeddings
vision
future image generation
```

---

## 2.4 Variable Processing

Examples:

```text
plan recognition
CPU workers
BIM processing
GPU rendering
video rendering
large reports
```

---

## 2.5 Commercial Transaction Costs

Examples:

```text
Stripe Payments
Stripe Billing
currency conversion
refunds/disputes
tax services
```

---

## 2.6 Commercial Data / SDK Licensing

Potential future costs:

```text
ODA Drawings SDK
construction cost databases
BCIS / RSMeans / regional data
Autodesk services
supplier data feeds
mapping/GIS data
```

Many of these are quote-based.

Do not include them in base infrastructure COGS until contracted.

---

## 2.7 Human / Opportunity Cost

The founder may not pay themselves during development, but the project still consumes engineering time.

For investment/ROI analysis, calculate:

```text
Economic Development Cost
=
Development Hours
×
Internal Developer Rate
```

Keep this separate from cash burn.

---

# 3. Cash Cost vs Economic Cost

Always maintain two numbers.

## Cash Burn

Actual cash leaving Buildora AI:

```text
subscriptions
cloud
API
licenses
domain
professional services
```

## Economic Development Cost

Cash Burn plus the market/internal value of founder time.

Example:

```text
1,500 development hours
×
$35/hour internal rate
=
$52,500 founder development value
```

This does not mean Buildora AI must pay $52,500 in cash.

It is useful for:

- ROI,
- investor discussions,
- product valuation,
- future team planning.

---

# 4. Current Verified Provider Pricing — September 2026

> **Verified:** 2026-09-12 against official OpenAI API pricing documentation.
> **Re-verify before any public pricing commitment.**
>
> **These figures are a human-readable snapshot, not the runtime source.**
> The authoritative source is the versioned `provider_rate_cards` table
> (**ADR-010**), which carries `provider`, `model_id`, `input_rate`,
> `cached_input_rate`, `cache_write_rate`, `output_rate`, `effective_from`,
> `effective_to`, `verified_at` and `source_reference`.
>
> Domain code uses only the logical aliases **FAST / BALANCED / ADVANCED /
> EXPERT**. Provider model identifiers must never appear in domain or business
> logic.

These values are references for the initial model.

They are not permanent assumptions.

---

# 5. OpenAI Standard API Pricing

Current standard text pricing per 1 million tokens (verified 2026-09-12):

| Model | Model ID | Input | Cached Input | Output |
|---|---|---:|---:|---:|
| GPT-5.6 Luna | `gpt-5.6-luna` | $0.20 | $0.02 | $1.20 |
| GPT-5.6 Terra | `gpt-5.6-terra` | $2.00 | $0.20 | $12.00 |
| GPT-5.6 Sol | `gpt-5.6-sol` | $4.00 | $0.40 | $20.00 |
| GPT-6 Astra | `gpt-6-astra` | $10.00 | $1.00 | $50.00 |

Important:

```text
GPT-5.6 Sol pricing is promotional as of this document date
and is stated by OpenAI to remain available at least through
21 November 2026.
```

Therefore:

> Do not build long-term Pro-plan economics assuming Sol can never become more expensive.

Budget a safety margin or route normal workloads to Luna/Terra.

OpenAI also states:

```text
Astra Batch / Flex
≈ 50% of Standard rates

Fast mode
≈ 2× applicable rates
```

where supported.

Use Standard processing as the normal Buildora AI cost baseline.

---

# 6. Example Raw AI Costs

These examples represent one provider call, not necessarily a complete user operation.

## Luna — Routine Assistant Request

Assumption:

```text
20,000 input
3,000 output
```

Cost:

```text
20,000 / 1M × $0.20 = $0.0040
3,000  / 1M × $1.20 = $0.0036

Total ≈ $0.0076
```

Less than one cent.

---

# 7. Terra — Balanced Request

Assumption:

```text
20,000 input
5,000 output
```

Cost:

```text
Input  = $0.040
Output = $0.060

Total ≈ $0.10
```

---

# 8. Sol — Professional Reasoning Request

Assumption:

```text
20,000 input
5,000 output
```

Current promotional cost:

```text
Input  = $0.080
Output = $0.100

Total ≈ $0.18
```

A complete Buildora AI design operation may involve multiple calls.

Planning target:

```text
normal complete AI design workflow
≈ $0.40–$0.80
```

after routing/caching/tool optimization.

This is an internal planning target, not a provider guarantee.

---

# 9. Astra — Expert Design Study

Assumption:

```text
30,000 input
8,000 output
```

Cost:

```text
Input  = $0.30
Output = $0.40

Total ≈ $0.70
```

Multi-call expert workflows may reach:

```text
$1.50–$3+
```

depending on:

- context,
- tool loops,
- reasoning,
- retries,
- escalation.

Astra must therefore remain a controlled premium resource.

---

# 10. Prompt Caching Economics

Cached input currently costs roughly one-tenth of normal input for the listed current OpenAI models.

Therefore stable content should be cache-friendly:

```text
system prompt
tool schemas
stable policies
shared project summaries
```

Buildora AI must monitor:

```text
cached_input_tokens
uncached_input_tokens
cache-write usage
```

High cache hit ratio materially protects AI margin.

---

# 11. Development Tool Subscriptions

The founder currently has:

```text
Claude Pro
Codex Plus / ChatGPT development access
Antigravity Pro
```

Treat them as:

```text
EXISTING FIXED DEVELOPMENT OVERHEAD
```

rather than adding duplicate API-agent subscriptions unnecessarily.

Because account pricing can vary by country, plan and promotion, maintain:

```text
DEV_CLAUDE_MONTHLY = actual invoice
DEV_CODEX_MONTHLY = actual invoice
DEV_ANTIGRAVITY_MONTHLY = actual invoice
```

Do not hardcode a guessed total into the financial model.

---

# 12. GitHub

Initial requirement:

```text
private repository
GitHub Actions
packages/container integration
```

GitHub Free may be sufficient initially.

Upgrade only when required by:

- higher Actions consumption,
- organization controls,
- security features,
- team growth.

Planning range during solo development:

```text
$0–$20/month
```

---

# 13. Local Development Infrastructure

Local Docker services:

```text
PostgreSQL
Redis
Temporal dev server
```

Software cost:

```text
$0
```

Actual cost is existing:

```text
computer
electricity
internet
```

These can be treated as founder operating overhead.

---

# 14. Development PostgreSQL — Neon

Current Neon reference:

```text
Free:
$0
100 CU-hours/project
0.5 GB included/project
scale-to-zero

Launch:
$0.106/CU-hour
$0.35/GB-month storage
typical small intermittent example ≈ $15/month
```

Recommendation:

```text
Local development → Docker PostgreSQL
CI → Docker PostgreSQL
Early hosted dev → Neon Free
Staging → Neon Launch
Production → Neon Launch initially
Scale plan only when features/load justify it
```

---

# 15. Development Authentication — Clerk

Current Clerk reference:

```text
Hobby
$0
up to 50,000 MRU/app

Pro
approximately $20/month when billed annually
with current published included MRU allowance
```

Recommendation:

```text
Development → Hobby
Early production → Hobby if feature requirements allow
Upgrade to Pro when required by production auth/business features
```

Do not upgrade only because the product has paying customers if Hobby technically and contractually supports the required production feature set.

---

# 16. Cloudflare

Current Cloudflare application-plan reference:

```text
Free       $0
Pro        $25/month monthly
           or $20/month annual billing
Business   $250/month monthly
           or $200/month annual billing
```

Recommendation:

```text
Development → Free
Early staging → Free
Paid beta → Free or Pro depending WAF/security requirements
Growth → Pro
Business only when features/support justify $200–$250/month
```

---

# 17. Cloudflare R2

Current Standard storage pricing:

```text
10 GB-month free/month

Beyond free:
Storage            $0.015 / GB-month
Class A operations $4.50 / million
Class B operations $0.36 / million
Internet egress    $0
```

At Buildora AI's early stage, object storage should be very inexpensive.

Example:

```text
100 GB stored
```

Raw storage component:

```text
≈ $1.50/month
```

before operation charges and free allowance treatment.

Renders/BIM storage are unlikely to dominate early costs.

---

# 18. AWS Fargate Reference

AWS Fargate pricing varies by:

- region,
- CPU architecture,
- OS.

For planning illustrations, a commonly published US-East Linux/X86 reference is approximately:

```text
vCPU     $0.04048 per vCPU-hour
Memory   $0.004445 per GB-hour
```

Use AWS Pricing Calculator for the actual selected production region.

---

# 19. Fargate Monthly Reference — 720 Hours

Approximate using the reference above:

| Task Size | Approx. Monthly Compute |
|---|---:|
| 0.25 vCPU / 0.5 GB | $8.89 |
| 0.5 vCPU / 1 GB | $17.77 |
| 1 vCPU / 2 GB | $35.55 |
| 2 vCPU / 4 GB | $71.09 |

These exclude:

- data transfer,
- ALB,
- NAT,
- logs,
- ECR,
- secrets,
- region premium.

For financial planning outside US-East:

```text
apply 15–25% infrastructure safety allowance
```

until the real region is selected in AWS Pricing Calculator.

This percentage is a planning buffer, not an AWS published regional tariff.

---

# 20. Application Load Balancer

AWS's published US-East example for a lightly loaded Application Load Balancer is approximately:

```text
$22.42/month
```

based on:

```text
$0.0225/hour ALB
+
light LCU usage
```

Real cost depends on:

- connections,
- active connections,
- processed bytes,
- rule evaluations,
- region.

Planning:

```text
$22–$50/month early
```

---

# 21. AWS NAT Cost

Private ECS tasks usually require outbound connectivity.

A NAT Gateway can become a noticeable startup cost.

A commonly published AWS US-region reference is approximately:

```text
$0.045/hour
```

plus data processing.

One always-on gateway:

```text
≈ $32.40/month
```

Two AZs:

```text
≈ $64.80/month
```

plus processed data.

This is why Buildora AI should track NAT/egress separately.

Do not accidentally spend more on NAT than API compute.

---

# 22. Redis / Valkey

Current Amazon ElastiCache reference indicates:

```text
Valkey can start around $6/month
```

for small/serverless configurations.

Planning:

```text
Development local Redis   $0
Early production          $6–$25/month
Growth                    usage dependent
```

---

# 23. Temporal Cloud

A current Temporal AWS Marketplace pay-as-you-go reference is:

```text
Plan fee        $100/month
Actions         $50 / million actions
```

Actual contract/direct Temporal pricing may differ.

Important:

Temporal is valuable, but it can be one of the larger fixed early infrastructure costs.

Recommendation:

```text
Local development:
Temporal dev server = $0

Early phases before workflows:
do not pay for Temporal Cloud yet

Staging when workflow work begins:
enable cloud when production-like validation is needed

Paid production:
budget $100+ usage
```

Do not activate the paid service months before Buildora AI uses durable workflows.

---

# 24. Observability

Use:

```text
Sentry
OpenTelemetry
AWS metrics/logs
```

Development:

```text
free/dev tiers where possible
```

Early production planning:

```text
$0–$50/month
```

Growth:

```text
usage dependent
```

Set log/event retention intentionally.

Do not send unnecessary high-cardinality or private data that increases both cost and privacy risk.

---

# 25. Stripe — UK Planning Reference

Current Stripe UK standard payment reference:

```text
Standard UK card:
1.5% + £0.20
```

Stripe Billing pay-as-you-go currently publishes:

```text
0.7% of Billing volume
```

in its UK pricing.

These are separate cost layers if using both products.

---

# 26. Example £49 Subscription Processing Cost

Assume:

```text
Subscription price = £49
standard UK card
Stripe Payments = 1.5% + £0.20
Stripe Billing = 0.7%
```

Payment processing:

```text
£49 × 1.5% + £0.20
=
£0.935
```

Billing:

```text
£49 × 0.7%
=
£0.343
```

Approximate combined:

```text
£1.278
```

before:

- VAT/tax,
- premium cards,
- international card surcharge,
- FX,
- refunds,
- disputes.

Round planning allowance:

```text
£1.30–£2.50 per £49 payment
```

depending on card/customer geography.

---

# 27. Example $49 US Subscription Reference

Using current standard US example pricing:

```text
2.9% + $0.30
+
0.7% Stripe Billing
```

Approximate:

```text
Payments ≈ $1.72
Billing  ≈ $0.34

Total ≈ $2.06
```

Use the merchant's actual Stripe country/rate in the live model.

---

# 28. Development Stage Costing

Development should be split into three sub-stages.

---

# 29. Development Stage A — Local-First Foundation

Covers approximately:

```text
Architecture
Repository
SaaS core
Building Model
Early 2D
```

Services:

```text
Local PostgreSQL
Local Redis
Local Temporal
GitHub
Neon Free optional
Clerk Hobby
Cloudflare Free
R2 Free
Sentry free/dev
```

Variable:

```text
OpenAI API testing
```

Recommended cash budget excluding already-owned AI coding subscriptions:

| Cost | Monthly |
|---|---:|
| GitHub | $0–$20 |
| Hosted DB | $0 |
| Redis | $0 |
| Temporal | $0 |
| R2 | $0 |
| Auth | $0 |
| Cloudflare | $0 |
| Monitoring | $0 |
| OpenAI API development | $20–$100 |
| Miscellaneous | $10–$30 |
| **Total** | **$30–$150/month** |

Add:

```text
Claude/Codex/Antigravity actual subscription invoices
```

separately.

---

# 30. Development Stage B — Active AI / 3D / QS Development

Covers approximately:

```text
2D
3D
QS
BOQ
Cost
AI Gateway
AI Copilot
```

AI testing becomes heavier.

Recommended:

| Cost | Monthly |
|---|---:|
| Existing coding subscriptions | Actual |
| OpenAI product/API testing | $50–$200 |
| Neon Free/Launch | $0–$20 |
| R2 | $0–$5 |
| Cloudflare | $0–$25 |
| Sentry/dev monitoring | $0–$25 |
| Optional hosted staging compute | $50–$100 |
| Miscellaneous | $20–$50 |
| **Incremental cash excluding owned subscriptions** | **$120–$425/month** |

Do not keep 24/7 staging running if it is not currently required.

---

# 31. Development Stage C — Pre-Beta / Production-Like Staging

Covers:

```text
Upload recognition
BIM
RAG
Billing
Reports
Production hardening
```

Expected recurring cost:

| Service | Planning Range |
|---|---:|
| AWS staging compute + ALB | $70–$150 |
| Neon Launch | $15–$50 |
| Managed Redis | $6–$20 |
| R2 | $0–$10 |
| Cloudflare | $0–$25 |
| Temporal Cloud | $100–$150 |
| Monitoring | $0–$50 |
| Auth | $0–$25 |
| OpenAI development/testing | $100–$300 |
| Miscellaneous | $25–$75 |
| **Total** | **$316–$855/month** |

Optimization:

```text
Temporal Cloud can be delayed until required.
Staging task count can be reduced.
Noncritical staging workloads can scale down.
```

Practical target:

```text
$300–$600/month
```

before paid beta.

---

# 32. Development One-Time Costs

Potential:

| Item | Planning |
|---|---:|
| Domain name | $10–$30/year |
| Brand/design assets | $0–$500 |
| Company legal docs/T&Cs/privacy | $0–$2,000+ |
| Security review before paid launch | $0–$5,000+ |
| Penetration test later | Quote-based |
| Commercial CAD SDK | Quote-based |
| Construction cost data license | Quote-based |
| Professional insurance/legal review | Business-specific |

Do not buy expensive CAD/QS data licenses before the relevant product feature is commercially validated.

---

# 33. Founder Time Budget

For planning only.

Example sprint effort:

```text
48 sprint roadmap
average 25–40 focused hours/sprint
```

Total:

```text
1,200–1,920 hours
```

At internal economic rate:

```text
$25/hour → $30,000–$48,000
$35/hour → $42,000–$67,200
$50/hour → $60,000–$96,000
```

This is not an estimate of outsourcing cost.

It is a useful founder-time valuation.

---

# 34. Total Development Cash Budget Before Launch

Assume:

```text
9–15 months
```

of solo development with cloud spend increasing gradually.

A reasonable direct-cash planning range is:

```text
Lean:
$2,000–$5,000

Comfortable:
$5,000–$10,000

With paid data/SDK/security/legal work:
$10,000–$30,000+
```

excluding:

- founder salary,
- commercial CAD licenses,
- major construction-data licensing,
- outsourced professional development.

The architecture is deliberately chosen so infrastructure does not require a six-figure pre-launch budget.

---

# 35. Production Cost Structure

Production cost should be modeled as:

```text
MONTHLY COST
=
FIXED PLATFORM
+
USAGE INFRASTRUCTURE
+
AI
+
RENDER
+
STORAGE
+
PAYMENT FEES
+
THIRD-PARTY VARIABLE SERVICES
+
CONTINGENCY
```

---

# 36. Fixed Production Platform

Early paid production target:

```text
2 Web tasks
2 API tasks
1+ workers
ALB
networking
PostgreSQL
Redis
Temporal
Cloudflare
monitoring
```

---

# 37. Early Production Fargate Reference

Example:

```text
2 × Web
0.5 vCPU / 1 GB

2 × API
0.5 vCPU / 1 GB

1 × Worker
0.5 vCPU / 1 GB
```

US-East reference compute:

```text
5 × $17.77
≈ $88.85/month
```

Apply actual region price before launch.

---

# 38. Early AWS Shared Costs

Example planning:

```text
Fargate             $90–$130
ALB                 $22–$40
NAT/network         $35–$80
logs/ECR/secrets    $10–$40
```

AWS subtotal:

```text
≈ $157–$290/month
```

for an early resilient configuration.

A cost-optimized beta with fewer replicas may be cheaper but accepts reduced availability.

---

# 39. Early Managed Platform Costs

Example:

```text
Neon PostgreSQL       $15–$60
Redis                  $6–$25
Temporal Cloud        $100–$150
Cloudflare              $0–$25
R2                      $0–$10
Monitoring              $0–$50
Auth                    $0–$25
```

Subtotal:

```text
≈ $121–$345/month
```

---

# 40. Paid Beta Fixed Platform Target

Combined fixed platform target:

```text
approximately $300–$600/month
```

before:

- AI usage,
- GPU/rendering,
- payment fees,
- quote-based commercial data.

This should be used as a budget target, not an invoice prediction.

---

# 41. Minimum-Cost Paid Beta Option

If revenue is initially very small:

```text
1 Web task
1 API task
worker on demand
one NAT strategy or alternate low-cost networking
Neon Launch
small managed Redis
Temporal only when required
Cloudflare Free/Pro
```

Possible fixed infrastructure:

```text
roughly $150–$350/month
```

Trade-off:

```text
lower availability
less redundancy
```

This can be acceptable for invitation-only beta if clearly understood internally.

---

# 42. Production AI Cost Targets by User Type

Internal monthly AI targets:

| User Type | Target AI Cost / MAU |
|---|---:|
| Free/light | $0.10–$0.50 |
| Home/light paid | $0.75–$2.00 |
| Normal paid | $1.50–$3.00 |
| Heavy Pro | $5–$12 |
| Very heavy professional | $15–$30+ |

These are internal target guardrails.

If actual users exceed them:

- inspect routing,
- credits,
- context size,
- misuse,
- plan fit.

---

# 43. Free User Cost Control

Free users should not be allowed to create unbounded AI cost.

Target:

```text
$0.10–$0.50 MAU/month
```

Methods:

- Luna-first routing,
- small AI allowance,
- no unlimited expert design,
- no free 4K rendering,
- rate limits,
- reduced storage/project limits.

---

# 44. Home Plan Target Economics

Illustrative:

```text
Plan price ≈ $14.99
```

Target variable COGS:

```text
$2–$4
```

Target variable gross margin:

```text
≈ 73–87%
```

depending on payment/geography/tax.

---

# 45. Pro Plan Target Economics

Illustrative:

```text
Plan price = $49
```

Internal target variable COGS:

```text
$11–$15/user/month
```

Possible allocation:

| Cost | Target |
|---|---:|
| Payment/billing | $1.5–$2.5 |
| Allocated platform infrastructure | $1–$2 |
| Normal Copilot | $1.5–$3 |
| AI Design allowance | $4–$6 |
| Rendering | $1–$2 |
| Storage/processing | $0.5–$1 |
| Monitoring/other variable | $0.5 |
| **Total** | **$10–$17** |

Operational target:

```text
keep normal blended Pro COGS near $11–$15
```

Contribution:

```text
$49 - $14
≈ $35
```

Gross contribution margin:

```text
≈ 71%
```

before fixed corporate costs/tax.

---

# 46. Pro AI Credit Model

Recommended:

```text
30 AI Design Credits/month
```

Illustrative charge rules:

```text
Standard concept      2
Advanced concept      3
Expert Astra study    5
Major AI modification 1
Manual edits           0
2D ↔ 3D sync           0
QS recalculation       0
BOQ recalculation      0
Cost recalculation     0
```

This makes variable AI cost bounded.

---

# 47. Pro Render Model

Illustrative:

```text
20 Render Credits/month
```

Example:

```text
HD still        1
4K still        2
360             3
Video           5–10
Browser 3D      0
```

Do not finalize Render Credit economics until GPU benchmarking is completed.

---

# 48. Business Plan Target

Illustrative:

```text
$149/month organization base
```

Possible costs:

```text
higher team use
shared AI
shared documents
storage
higher support
```

Target variable COGS:

```text
20–30% of revenue
```

unless enterprise-heavy usage is explicitly charged separately.

---

# 49. Enterprise

Do not promise:

```text
unlimited everything
```

Enterprise pricing should include:

- seats,
- AI allowance,
- storage,
- support,
- integration,
- negotiated overage.

Aim for:

```text
minimum annual commitment
```

that covers operational/support complexity.

---

# 50. AI Design Cost Budget

If average complete standard concept costs:

```text
$0.40
```

Then:

```text
15 concepts
≈ $6
```

If average advanced concept:

```text
$0.60
```

Then:

```text
10 concepts
≈ $6
```

This aligns with 30 credit design.

---

# 51. Astra Exposure

If an Astra-heavy workflow costs:

```text
$2 average
```

Then:

```text
6 expert studies
≈ $12
```

Therefore unrestricted expert studies are unsuitable for a $49 plan.

The 5-credit pricing ensures the user cannot consume dozens without top-ups.

---

# 52. AI Retry Budget

Internal provider failures increase Buildora AI cost.

Do not charge the customer multiple times.

Add AI operational contingency:

```text
5–15%
```

above expected successful-call cost for:

- retries,
- failures,
- escalations.

This is a planning assumption.

---

# 53. Render Cost Budget

Until measured, use conservative internal planning:

```text
HD still             $0.25–$1.00
4K still             $0.75–$2.50
360                   $1.00–$4.00
video walkthrough    $5–$25+
```

These are placeholders, not provider quotations.

Before production rendering:

benchmark:

```text
GPU type
scene complexity
render duration
resolution
samples
Blender settings
```

Then define Render Credit value from real telemetry.

---

# 54. BIM / Recognition Processing

Most plan recognition/BIM processing can be:

```text
CPU worker
+
occasional AI/Vision
```

Internal target per ordinary residential import:

```text
$0.05–$0.50
```

Complex/AI-heavy recognition:

```text
$0.50–$2+
```

These are engineering targets.

Measure them.

---

# 55. RAG Cost

RAG costs:

```text
document extraction
embedding
storage
retrieval
AI answer
```

Embedding/storage are normally small relative to reasoning.

Main control:

```text
do not send entire document corpus into model context
```

Use retrieval.

---

# 56. Storage Model

Example planning:

```text
Average active project:
100–500 MB
```

depending on:

- drawings,
- IFC,
- renders.

At R2:

```text
1 TB
×
$0.015/GB-month
≈
$15/month raw Standard storage
```

before operation charges/free tier.

Storage should not be the main early Buildora AI expense.

GPU/AI likely matter more.

---

# 57. Database Cost Drivers

Neon cost grows with:

```text
compute CU-hours
storage
restore/history storage
branches
network egress beyond included amount
```

Do not estimate database cost solely from user count.

Major driver:

```text
active query workload
```

---

# 58. Temporal Cost Drivers

Temporal cost is driven by:

```text
actions
active workflow history
retained history
```

Avoid unnecessary:

- tiny workflows for trivial CRUD,
- aggressive retry loops,
- high-frequency timers.

Use Temporal for genuinely durable multi-step operations.

---

# 59. Payment Cost

Payment cost is revenue-linked.

This is good:

```text
no customer payment
→ no Stripe payment transaction fee
```

Model separately from infrastructure.

---

# 60. Production Scenario Assumptions

The following scenarios are illustrations.

Assume:

```text
10% of MAU paid
90% Free
```

Example average AI cost:

```text
Free MAU          $0.10
Paid MAU          $3.00
```

This is intentionally usage-sensitive.

It is not a forecast of actual Buildora AI behaviour.

---

# 61. Scenario A — Private Paid Beta

Assumption:

```text
100 MAU
10 paid
90 free
```

AI:

```text
90 × $0.10 = $9
10 × $3.00 = $30

≈ $39
```

Other variable processing:

```text
$25–$100
```

Fixed platform:

```text
$300–$600
```

Estimated monthly technical cost:

```text
≈ $365–$740
```

excluding:

- payment fees,
- founder salary,
- commercial licenses.

---

# 62. Scenario B — 1,000 MAU

Assumption:

```text
1,000 MAU
100 paid
900 free
```

AI:

```text
900 × $0.10 = $90
100 × $3 = $300

≈ $390
```

Infrastructure may need:

```text
more ECS capacity
higher DB compute
more workflows
```

Planning:

| Cost | Range |
|---|---:|
| Platform fixed/scaled | $450–$900 |
| AI | $390–$800 |
| Render/processing | $50–$250 |
| Storage | $5–$30 |
| Monitoring/other | $20–$100 |
| **Technical total** | **$915–$2,080/month** |

Plus payment fees.

---

# 63. Scenario C — 10,000 MAU

Assumption:

```text
10,000 MAU
1,000 paid
9,000 free
```

AI reference:

```text
9,000 × $0.10 = $900
1,000 × $3 = $3,000

≈ $3,900
```

Planning:

| Cost | Range |
|---|---:|
| Compute/network | $700–$1,800 |
| PostgreSQL | $200–$800 |
| Redis | $50–$250 |
| Temporal | $150–$500 |
| AI | $3,000–$5,500 |
| Render/processing | $300–$1,000 |
| R2 | $25–$150 |
| Observability | $50–$300 |
| Auth/other | $25–$250 |
| **Technical total** | **~$4,500–$10,550/month** |

A highly optimized, light-AI workload could be near the lower side.

Heavy Pro usage can be materially higher.

---

# 64. Scenario D — 100,000 MAU

Assumption:

```text
100,000 MAU
10,000 paid
90,000 free
```

AI reference:

```text
90,000 × $0.10 = $9,000
10,000 × $3 = $30,000

≈ $39,000
```

At this scale:

- auth overage may apply,
- DB/Redis substantially larger,
- worker pools,
- observability,
- support,
- potentially enterprise infrastructure.

Planning:

```text
Technical platform:
$45,000–$90,000+/month
```

depending mostly on:

```text
AI
rendering
usage intensity
enterprise workload
```

This is why Buildora AI must scale revenue/credits with usage rather than advertise truly unlimited expensive AI.

---

# 65. User Count Is Not the Main Cost Driver

Compare:

```text
10,000 Free/light users
```

with:

```text
1,000 architects running Expert AI studies daily
```

The second group may cost more.

Main cost metrics:

```text
active users
paid mix
AI jobs
model route
tokens
render minutes
BIM processing
documents
storage
workflow actions
```

---

# 66. Production Cost Formula

Maintain monthly finance model:

```text
TOTAL_TECH_COST
=
AWS_COMPUTE
+ AWS_NETWORK
+ DATABASE
+ REDIS
+ TEMPORAL
+ STORAGE
+ AUTH
+ OBSERVABILITY
+ AI
+ RENDER
+ OTHER_PROCESSING
+ PAYMENT_FEES
```

---

# 67. AI Cost Formula

```text
AI_COST
=
Σ ProviderCallCost
```

For each call:

```text
uncached_input_cost
+
cached_input_cost
+
cache_write_cost
+
output_cost
+
tool/provider-specific charges
```

Never approximate AI cost only from number of requests.

---

# 68. Per-Organization COGS

```text
ORG_COGS
=
allocated platform
+
AI usage
+
render
+
storage
+
payment
+
variable third-party cost
```

Track per billing period.

---

# 69. Per-Plan Gross Margin

```text
GROSS_MARGIN
=
(Revenue - Variable COGS)
/
Revenue
```

Target:

```text
70%+
```

for standard SaaS plans.

Higher is preferable after optimization.

---

# 70. Margin Warning Thresholds

Planning:

```text
Healthy        ≥70%
Review         60–70%
Warning        50–60%
Critical       <50%
```

Do not automatically terminate users because of one low-margin month.

Investigate:

- misuse,
- routing,
- pricing,
- plan fit.

---

# 71. Break-Even — Fixed Technical Platform

Example paid beta:

```text
Fixed platform = $500/month
```

At Pro contribution before fixed platform:

```text
≈ $35/user
```

Technical fixed-cost break-even:

```text
$500 / $35
≈ 15 Pro-equivalent users
```

This is not company profitability.

It only covers illustrative fixed technical platform cost.

---

# 72. Broader Operating Break-Even

If monthly total company operating cost becomes:

```text
$5,000
```

and contribution/user:

```text
$35
```

Then:

```text
≈ 143 Pro-equivalent customers
```

before taxes and revenue mix.

Use blended plan contribution for real calculations.

---

# 73. Payment Fees Should Not Be Forgotten

Example:

```text
1,000 × £49 subscriptions
```

Payment + Billing fees can become well over £1,000/month depending card mix.

Treat them as COGS.

Do not compare infrastructure cost to revenue while ignoring payment fees.

---

# 74. VAT / Sales Tax

Do not include VAT as product margin.

Depending jurisdiction:

```text
VAT collected
→ tax liability
```

Use Stripe Tax/accounting advice as appropriate.

Do not let AI calculate tax policy authoritatively.

---

# 75. Refund / Dispute Reserve

Planning reserve:

```text
0.5–2% of revenue
```

depending actual refund/dispute behaviour.

This is a finance planning assumption, not a provider fee.

Review from real data.

---

# 76. Infrastructure Contingency

Budget:

```text
15–25%
```

above modeled cloud cost during early production.

Reasons:

- traffic spikes,
- logs,
- NAT,
- test resources,
- forgotten preview resources,
- DB growth.

Reduce contingency once operating data is stable.

---

# 77. AI Contingency

Budget:

```text
15–25%
```

above expected provider cost until:

- retry rate known,
- routing optimized,
- cache hit measured.

Do not sell plans using theoretical best-case AI costs.

---

# 78. Commercial License Contingency

Keep separate reserve:

```text
$5,000–$25,000+
```

before serious professional expansion if Buildora AI needs commercial:

- CAD SDK,
- cost databases,
- GIS datasets,
- code/compliance databases.

Actual pricing requires vendor quotes.

Do not treat this reserve as committed spend.

---

# 79. ODA / DWG Strategy

Professional DWG/DXF support may require commercial ODA licensing.

Cost:

```text
QUOTE-BASED
```

Development plan:

```text
do not commit license cost
until DWG becomes an MVP/customer requirement
```

Alternative early support:

```text
DXF/open formats
IFC
PDF
```

where legally/technically suitable.

---

# 80. Construction Cost Data Licensing

Professional cost data may be one of Buildora AI's largest non-cloud costs.

Potential:

```text
BCIS
RSMeans
regional databases
supplier datasets
```

Licensing can involve:

- subscription,
- redistribution rights,
- API fees,
- per-seat,
- commercial restrictions.

Do not assume buying an ebook/database subscription grants SaaS redistribution rights.

Budget separately after legal/vendor confirmation.

---

# 81. Maps / GIS

Basic coordinates/PostGIS:

```text
low infrastructure cost
```

Third-party maps/terrain/satellite data may become usage-billed later.

Do not add them to MVP fixed cost before site analysis exists.

---

# 82. Development Hardware

Current Mac/computer:

```text
sunk/existing asset
```

Future optional:

- additional RAM/storage,
- test Windows machine,
- mobile devices,
- GPU workstation.

Do not buy GPU workstation solely for cloud rendering before benchmarking hosted GPU costs.

---

# 83. Mobile Future Cost

Later:

```text
Apple Developer Program
Google Play account
mobile CI/device testing
```

Exclude from web MVP budget.

Add when Phase 43/mobile begins.

---

# 84. Cost Center Structure

Finance should classify:

```text
DEV-TOOLS
CLOUD-COMPUTE
CLOUD-DATABASE
CLOUD-NETWORK
STORAGE
AI
RENDER
OBSERVABILITY
AUTH
WORKFLOW
PAYMENTS
DATA-LICENSES
SECURITY
LEGAL
```

This makes cost optimization easier.

---

# 85. Tagging

AWS tags:

```text
Product=Buildora AI
Environment=Production
Service=API
CostCenter=CLOUD-COMPUTE
```

AI usage:

```text
organization
project
user
job
model
operation
```

Database usage records support product-level FinOps.

---

# 86. Monthly Cost Dashboard

Build an internal dashboard eventually showing:

```text
MRR
Technical cost
AI cost
Cloud cost
Payment fees
COGS
Gross margin
Cost/MAU
Cost/paid user
AI cost/paid user
Cost/project
```

---

# 87. Daily AI Budget Dashboard

Display:

```text
Today
Yesterday
7-day
Month-to-date
Projected month-end
```

Breakdown:

```text
Luna
Terra
Sol
Astra
operation type
plan
organization
```

---

# 88. Budget Alerts

Set provider/cloud budget alerts.

Development:

```text
AI API
$50
$100
$200
```

Production thresholds should be proportional to expected revenue.

Examples:

```text
50%
80%
100%
120%
```

of monthly budget.

---

# 89. AI Runaway Protection

Hard limits:

```text
max output tokens
max tool iterations
max retries
job cost ceiling
daily organization cost ceiling
```

Do not allow a code bug to run an unlimited Astra loop.

---

# 90. Render Runaway Protection

Limit:

```text
max resolution
max samples
max frame count
max render duration
max concurrent jobs
```

Require credit reservation before GPU starts.

---

# 91. Development Cost Optimization

Use:

```text
local first
free tiers
scale-to-zero
staging only when relevant
small test fixtures
Luna/Terra for most test traffic
Astra only for expert test cases
```

Do not use expert AI model for routine CRUD development tests.

---

# 92. Production AI Optimization

Order:

```text
avoid AI when deterministic code works
 ↓
Luna
 ↓
Terra
 ↓
Sol
 ↓
Astra only when necessary
```

Cache stable context.

Use incremental actions.

Do not regenerate entire project for one wall edit.

---

# 93. Incremental AI Cost Example

Request:

```text
Make Bedroom 2 300 mm wider.
```

Efficient flow:

```text
AI identifies wall
→ MOVE_WALL command
→ geometry
→ QS delta
```

Target AI cost:

```text
$0.05–$0.20
```

rather than regenerating the full design.

This is a Buildora AI engineering target.

---

# 94. Batch / Flex

For non-interactive AI tasks:

- nightly analysis,
- document enrichment,
- optional background studies,

consider lower-cost Batch/Flex where supported.

Do not use slower processing for interactive design actions where UX suffers.

---

# 95. Storage Optimization

Do not store duplicate:

```text
original
full-res intermediate
full-res temporary
```

forever.

Lifecycle:

```text
original retained
important derivative retained
temporary artifacts expire
```

---

# 96. Database Optimization

Do not upgrade DB plan first.

First:

- index correctly,
- fix N+1,
- inspect slow queries,
- cache safe reads,
- reduce unnecessary model payloads.

Then scale compute.

---

# 97. Temporal Optimization

Reduce:

- unnecessary workflow starts,
- overly frequent timers,
- retry storms.

Use durable workflow only when needed.

---

# 98. Network Optimization

Watch:

```text
NAT data processing
AWS egress
ALB bytes
```

R2's no-internet-egress model helps large customer file delivery.

Do not proxy R2 downloads through API unless security/workflow requires it.

---

# 99. Development Budget Recommendation

For the solo founder, set:

```text
Baseline monthly dev budget:
$300
```

excluding existing AI coding subscriptions.

Allow temporary increase to:

```text
$600
```

during:

- AI/BIM,
- staging,
- billing,
- launch hardening.

This should cover most pre-beta engineering if expensive licenses are deferred.

---

# 100. Pre-Launch Reserve

Keep:

```text
$3,000–$5,000
```

available for the final pre-launch/first-production period.

Can cover:

- infrastructure,
- AI testing,
- legal/docs,
- monitoring,
- unforeseen service upgrades,
- security work.

This is planning guidance, not a mandatory expense.

---

# 101. First-Year Technical Cash Budget

Illustrative founder-operated year:

## Development-heavy year

```text
6 months average $200
+
3 months average $400
+
3 months beta average $700
```

Approx:

```text
$1,200
+
$1,200
+
$2,100
=
$4,500
```

Add existing coding subscriptions and one-time services.

Planning range:

```text
$5,000–$12,000
```

for infrastructure/API/tool cash is reasonable before expensive commercial data/licensing.

---

# 102. First-Year Full Project Cash Range

Including:

```text
technical cash
legal/policy help
domain/email
security
miscellaneous
```

Target reserve:

```text
$8,000–$20,000
```

still excluding:

- founder salary,
- hired developers,
- major CAD SDK,
- professional cost database license.

---

# 103. Outsourced Equivalent Development Cost

The architecture/product scope is substantial.

If outsourced professionally, the cost could be many times the founder cash burn.

Earlier planning range for a proper MVP:

```text
$75,000–$180,000
```

and mature production V1:

```text
$200,000–$500,000+
```

These should be treated as broad strategic valuation ranges, not vendor quotations.

Solo AI-assisted development reduces cash cost but increases:

- time,
- founder dependency,
- review burden,
- launch timeline risk.

---

# 104. Costing by Development Phase

Suggested allocation:

| Phase | Cash Intensity |
|---|---|
| Architecture/Foundation | Very Low |
| SaaS/Auth/Projects | Low |
| Building Model | Low |
| 2D/3D | Low–Medium |
| QS/Cost | Low |
| AI Platform | Medium |
| Recognition/BIM | Medium |
| RAG | Medium |
| Billing | Medium |
| Production Hardening | Medium–High |
| Rendering | High only when used |

---

# 105. Cost Gate Before Starting a New Module

Before a phase introduces a paid service:

Ask:

```text
Is this required now?
Can local/free service validate it?
What fixed monthly cost begins?
Can it scale to zero?
Does it create vendor lock-in?
Does the product generate revenue from it?
```

---

# 106. Production Scaling Financial Gates

## Move from one to two replicas when:

```text
paid production availability requires redundancy
```

not because of arbitrary user count.

## Increase database plan when:

```text
measured compute/storage/feature requirement
```

## Add GPU when:

```text
render revenue/credits justify cost
```

## Upgrade Cloudflare:

```text
security/traffic feature required
```

## Upgrade Clerk:

```text
feature/MRU requirement reached
```

---

# 107. Unit Economics Dashboard

Per plan display internally:

```text
Subscription revenue
Payment fee
AI cost
Render cost
Infra allocation
Storage
Other variable
Contribution
Margin %
```

---

# 108. Infrastructure Allocation Method

Do not attempt perfect cost allocation initially.

Simple monthly method:

```text
Fixed platform
/
weighted active users
```

or:

```text
Fixed platform
/
paid organizations
```

For advanced usage:

allocate by:

```text
API requests
compute minutes
storage
```

Keep financial model understandable.

---

# 109. Free-to-Paid Subsidy

Paid users support free acquisition users.

Track:

```text
Free COGS
/
Paid contribution
```

Example:

```text
9,000 free
×
$0.10
=
$900

1,000 paid
×
$35 contribution
=
$35,000
```

Free AI cost:

```text
≈ 2.6% of paid contribution
```

acceptable in this illustrative case.

If Free cost becomes $2/user:

```text
$18,000
```

the model changes completely.

This is why Free must be controlled.

---

# 110. Paid Conversion Sensitivity

For 10,000 MAU:

```text
5% paid  = 500
10% paid = 1,000
20% paid = 2,000
```

Infrastructure may be similar, but revenue changes dramatically.

Business viability is more sensitive to:

```text
conversion
retention
ARPU
AI cost
```

than raw signup count.

---

# 111. Churn Cost

Customer acquisition cost is not included in this technical plan.

But future business model must include:

```text
marketing
sales
support
CAC
```

A technically profitable user may still be unprofitable after acquisition/support.

---

# 112. Support Cost

Solo founder:

```text
support = founder time
```

At growth:

budget:

```text
support staff
customer success
enterprise onboarding
```

Do not count these as infrastructure COGS.

Track separately under operating expenses.

---

# 113. Cost Model Variables

Maintain config/spreadsheet variables:

```text
MAU
PAID_USERS
FREE_USERS

HOME_USERS
PRO_USERS
BUSINESS_ORGS

AVG_AI_COST_FREE
AVG_AI_COST_HOME
AVG_AI_COST_PRO
AVG_AI_COST_BUSINESS

RENDER_COST
STORAGE_GB
TEMPORAL_ACTIONS

AWS_FIXED
DB_COST
REDIS_COST
OBSERVABILITY_COST
AUTH_COST

PAYMENT_RATE
PAYMENT_FIXED_FEE
BILLING_RATE
```

---

# 114. Recommended Finance Formula

```text
MRR
=
HomeRevenue
+
ProRevenue
+
BusinessRevenue
+
EnterpriseRevenue
+
TopUpRevenue
```

```text
COGS
=
PaymentFees
+
AI
+
Rendering
+
VariableCompute
+
VariableStorage
+
OtherUsage
```

```text
GrossProfit
=
MRR - COGS
```

```text
GrossMargin
=
GrossProfit / MRR
```

Keep fixed pre-revenue engineering subscriptions out of SaaS COGS once they are primarily development expenses.

---

# 115. Top-Up Economics

Illustrative packs:

```text
10 credits   $5
25 credits   $10
60 credits   $20
150 credits  $40
```

Before launch verify:

```text
expected cost/credit
payment fee
gross margin
```

Small packs suffer more fixed Stripe transaction fee.

Consider minimum purchase price.

---

# 116. Example Credit Internal Value

If target average Design Credit provider cost:

```text
$0.20
```

Then:

```text
30 credits
≈ $6 expected AI cost
```

Use real telemetry to update.

Do not publicly promise dollar conversion:

```text
1 credit = $0.20
```

because credits represent product value, not raw provider spend.

---

# 117. Render Credit Internal Value

After GPU testing determine:

```text
expected GPU cost/render credit
```

Target example:

```text
$0.25–$0.50 internal average per credit
```

Then 20 credits:

```text
$5–$10 maximum expected allowance
```

If the actual average is $1/credit, reduce included credits or adjust plan.

---

# 118. Cost Telemetry Requirements Before Launch

Must record:

```text
AI provider cost/job
AI cost/user
AI cost/org
AI cost/plan
Render compute/job
Plan recognition/job
BIM processing/job
R2 bytes/project
DB size
Temporal actions/workflow
```

Without telemetry, pricing is guessing.

---

# 119. Price Review Schedule

Before launch:

```text
weekly during load testing
```

After launch:

```text
monthly
```

Quarterly:

```text
plan/credit economics review
```

Do not constantly change customer pricing because of normal monthly variation.

---

# 120. Provider Price Change Process

If OpenAI/AWS/etc changes prices:

1. update provider rate card,
2. calculate impact on current plans,
3. inspect margin,
4. optimize routing first,
5. adjust future plan version/credits if required,
6. grandfather or migrate existing customers deliberately.

Do not rewrite historical usage cost.

---

# 121. AI Model Price Risk

Current Sol rate is promotional.

Risk response:

```text
Do not sell credits based only on current promo.
```

Budget normal Sol-intensive operation with:

```text
25–50% AI price contingency
```

until post-promotion pricing is confirmed.

Luna/Terra provide low-cost routing alternatives.

---

# 122. Currency Risk

Buildora AI may:

```text
bill GBP
pay AI/AWS in USD
```

Therefore margin can change with FX.

At growth:

```text
track revenue and costs in reporting base currency
```

Use a controlled FX rate source for management reporting.

Do not use today's spot rate to rewrite historical financial records.

---

# 123. Tax Exclusion

Cost examples in this document are generally:

```text
pre-tax / provider list-price references
```

Your company may pay:

- VAT,
- sales tax,
- withholding,
- foreign transaction fees.

Accounting treatment requires professional advice.

---

# 124. Pricing Plan Safety Rules

Never offer:

```text
unlimited Astra
unlimited 4K render
unlimited storage
```

without fair-use/technical controls.

Safe unlimited claims:

```text
manual 2D editing
manual 3D editing
deterministic recalculation
```

because marginal compute is small.

---

# 125. Cost-Efficient Architecture Decisions Already Made

Buildora AI saves cost by:

```text
browser-side 3D
deterministic QS
deterministic costing
client GPU
R2 egress-free storage
managed PostgreSQL
automatic model routing
prompt caching
incremental geometry changes
background workers only when needed
GPU scale-to-zero
```

These are not merely technical choices.

They are unit-economic decisions.

---

# 126. What Would Make Buildora AI Expensive

Avoid:

```text
Astra for every chat
server rendering every 3D frame
recreating complete building on every edit
storing huge meshes in DB
running permanent GPU
Kubernetes too early
one Temporal workflow for every tiny CRUD
massive context prompts
free unlimited renders
public API abuse
```

---

# 127. Development Cost Decision Matrix

| Capability | Development Approach | Cost Strategy |
|---|---|---|
| PostgreSQL | local Docker | $0 |
| Redis | local Docker | $0 |
| Temporal | local server | $0 |
| Auth | Clerk dev/free | $0 |
| R2 | free tier | ~$0 |
| AI | API budget | capped |
| 3D | browser GPU | $0 server GPU |
| QS | TypeScript | deterministic |
| BIM | local Python | local CPU |
| Render | postpone | $0 initially |

---

# 128. Production Cost Decision Matrix

| Capability | Production | Cost Driver |
|---|---|---|
| Web | Fargate | CPU/RAM hours |
| API | Fargate | CPU/RAM hours |
| Workers | Fargate | workload |
| DB | Neon | CU-hours/storage |
| Redis | Managed Valkey | memory/requests |
| Workflow | Temporal Cloud | actions/storage |
| Files | R2 | GB/operations |
| AI | OpenAI | tokens/tools |
| Render | GPU | compute time |
| Billing | Stripe | revenue/transactions |
| Auth | Clerk | plan/MRU |
| Monitoring | Sentry | usage/plan |

---

# 129. Recommended Monthly Budget Stages

## Months 1–3

```text
$50–$200
+
existing coding subscriptions
```

## Months 4–8

```text
$150–$400
+
existing subscriptions
```

## Months 9–12 / Pre-Beta

```text
$300–$700
```

## Paid Beta

```text
$400–$1,000+
```

depending AI traffic.

## Growth

```text
usage-driven
```

No longer budget as one fixed number.

---

# 130. Cost Approval Thresholds for Solo Founder

Suggested:

```text
< $25/month new service
→ normal architecture decision

$25–$100/month
→ record in cost register

$100–$500/month
→ ADR + financial justification

> $500/month
→ explicit commercial/revenue justification
```

This avoids subscription creep.

---

# 131. Cost Register

Maintain *(not yet created — start it when the first recurring cost is incurred)*:

```text
docs/finance/cost-register.md
```

Fields:

```text
Service
Purpose
Environment
Owner
Billing account
Monthly fixed
Variable basis
Budget
Alert threshold
Cancellation link/process
Started
Last reviewed
```

---

# 132. Sample Cost Register

```text
Service: Temporal Cloud
Purpose: Durable workflows
Environment: Production
Fixed: $100/mo reference
Variable: $50/million actions reference
Budget: $200/mo initially
Alert: 80%
Review: Monthly
```

---

# 133. Infrastructure Budget Alerts

AWS:

```text
$50
$100
$250
$500
...
```

adjust by stage.

OpenAI:

set project budget/alerts.

Neon:

use spend/compute limits where available.

Temporal:

monitor action volume.

---

# 134. Production Gross Margin Budget

For each month create:

```text
Revenue              $X
Payment fees        -$X
AI                   -$X
Render               -$X
Infra allocation     -$X
Other variable       -$X
--------------------------------
Gross contribution   $X
Gross margin          X%
```

---

# 135. Do Not Mix Development Spend with COGS

Example:

```text
Claude Pro used by founder
```

usually:

```text
R&D / operating expense
```

not customer COGS.

OpenAI calls serving customers:

```text
COGS
```

This distinction matters when evaluating SaaS margin.

---

# 136. Staging Cost Is Operating/R&D

Staging infrastructure normally:

```text
engineering operating expense
```

not per-customer production COGS.

Keep it separate in finance reporting.

---

# 137. Production Shared Infrastructure

Production ECS/DB has:

```text
fixed/semi-fixed COGS
```

Allocate internally for product economics, even if accounting classification differs.

---

# 138. Cost Forecast Bands

Never produce one exact monthly forecast.

Use:

```text
Low
Expected
High
```

Example:

```text
Paid Beta

Low       $400
Expected  $650
High      $1,000
```

This better reflects AI/usage uncertainty.

---

# 139. Production Scenario Table

| Stage | Technical Monthly Range |
|---|---:|
| Invite-only beta | $350–$750 |
| ~1,000 MAU | $900–$2,100 |
| ~10,000 MAU | $4,500–$10,500 |
| ~100,000 MAU | $45,000–$90,000+ |

These are planning scenarios under the assumptions documented above.

They are not quotes.

---

# 140. Revenue Sensitivity — 1,000 MAU Example

Assume:

```text
100 paid
900 free
```

If all paid were $49 Pro:

```text
MRR = $4,900
```

Illustrative technical cost:

```text
$900–$2,100
```

before payment fees.

This can support healthy unit economics if:

- retention is good,
- AI is controlled,
- paid conversion is maintained.

Real revenue will be a mix of Home/Pro/Business.

---

# 141. Revenue Sensitivity — 10,000 MAU

Assume:

```text
1,000 paid
```

All-Pro theoretical:

```text
$49,000 MRR
```

Technical cost planning:

```text
$4,500–$10,500
```

plus payment fees.

This shows why the business can be attractive if usage is efficiently routed.

It is not a revenue forecast.

---

# 142. Cost per Active User

Track:

```text
TOTAL_TECH_COST / MAU
```

But also track:

```text
VARIABLE_COGS / PAID_USER
```

and:

```text
AI_COST / PAID_USER
```

because total MAU alone can hide poor paid-user economics.

---

# 143. Cost per Project

Useful:

```text
AI cost/project
storage/project
processing/project
```

Professional users may have fewer projects but much heavier projects.

---

# 144. Cost per AI Feature

Track:

```text
Copilot chat
Standard concept
Advanced concept
Expert design
Plan recognition
Value engineering
Document Q&A
```

Do not combine all AI into one total when deciding credits.

---

# 145. Cost-per-Success

Especially for AI:

```text
total provider cost
/
successful customer-visible jobs
```

Retries/failures matter.

This metric is more meaningful than raw price/request.

---

# 146. Development Efficiency Metric

Track:

```text
development monthly cash burn
/
completed phase
```

Not to pressure development, but to spot unused subscriptions/environments.

---

# 147. Cost Optimization Review Checklist

Monthly:

- [ ] unused cloud services.
- [ ] old preview environments.
- [ ] stale database branches.
- [ ] old ECR images.
- [ ] excessive logs.
- [ ] NAT spend.
- [ ] oversized tasks.
- [ ] DB scale.
- [ ] Redis memory.
- [ ] Temporal retries.
- [ ] R2 temporary files.
- [ ] AI model distribution.
- [ ] cache hit.
- [ ] Astra usage.
- [ ] GPU idle time.
- [ ] auth plan.
- [ ] monitoring plan.

---

# 148. Pre-Production Cost Checklist

Before paid beta:

- [ ] fixed monthly platform estimate.
- [ ] expected 100-user cost.
- [ ] AI cost telemetry.
- [ ] design-credit telemetry.
- [ ] render benchmark or disabled rendering.
- [ ] payment fee model.
- [ ] provider budgets.
- [ ] cloud budget alerts.
- [ ] gross-margin dashboard.
- [ ] refund/dispute reserve.
- [ ] pre-launch cash reserve.

---

# 149. 1,000-User Review Gate

At approximately 1,000 MAU review:

- [ ] cost/MAU.
- [ ] paid conversion.
- [ ] AI cost paid user.
- [ ] Pro margin.
- [ ] Free subsidy.
- [ ] ECS utilization.
- [ ] DB spend.
- [ ] Redis spend.
- [ ] Temporal actions.
- [ ] storage.
- [ ] payment fees.

Do not scale architecture purely because the user count reached 1,000.

---

# 150. 10,000-User Review Gate

Review:

- [ ] Savings Plans / Fargate optimization.
- [ ] EC2 ECS vs Fargate economics.
- [ ] read replica.
- [ ] DB plan.
- [ ] Temporal pricing/contract.
- [ ] observability contract.
- [ ] auth overage.
- [ ] Cloudflare plan.
- [ ] AI volume pricing.
- [ ] Stripe negotiated pricing.
- [ ] commercial data licensing.

At this size, vendor negotiation can matter.

---

# 151. 100,000-User Review Gate

Evaluate:

- [ ] reserved/committed compute.
- [ ] dedicated DB architecture.
- [ ] enterprise support plans.
- [ ] AI volume agreements.
- [ ] multi-region needs.
- [ ] internal FinOps.
- [ ] warehouse.
- [ ] support team.
- [ ] security/compliance investment.

---

# 152. Cost Reduction Priority

When margin drops, optimize in this order:

```text
1. AI routing/context
2. Failed retries
3. GPU/render utilization
4. Query/database inefficiency
5. Compute sizing
6. Logs/observability noise
7. Network/NAT
8. Vendor contracts
```

Do not immediately downgrade product quality.

---

# 153. Never Optimize Away Correctness

Do not reduce cost by:

- using inaccurate QS,
- skipping geometry validation,
- removing backups,
- weakening security,
- disabling audit,
- using cheap AI when expert reasoning is genuinely necessary.

Cost optimization must preserve product trust.

---

# 154. Technical COGS Target

At mature operation:

```text
Technical + provider COGS
≤ 20–30% of SaaS revenue
```

supports:

```text
70–80%+ gross margin
```

before sales/R&D/admin.

This is the key product economics target.

---

# 155. Pricing Review Trigger

Review plan pricing if:

```text
Pro COGS > 35% for 3 months
```

or:

```text
heavy usage systematically consumes most margin
```

Actions:

- tune routing,
- reduce included credits,
- change new-plan version,
- introduce top-ups,
- enterprise tier.

Do not retroactively surprise current customers without clear terms.

---

# 156. Cost Data Architecture

Buildora AI's billing architecture should persist:

```text
provider usage
provider rate card
estimated provider cost
customer credits
plan
billing period
```

This enables automatic costing reports.

Do not maintain cost economics only in an external spreadsheet.

---

# 157. Finance Spreadsheet / Dashboard Outputs

At minimum export monthly:

```text
MAU
paid users
MRR
plan mix
AI tokens
AI cost
render cost
cloud cost
Stripe fees
COGS
gross contribution
gross margin
cash burn
runway
```

---

# 158. Runway

Formula:

```text
Runway Months
=
Available Cash
/
Monthly Net Burn
```

Before revenue:

```text
Net Burn ≈ operating cash costs
```

After revenue:

```text
Net Burn
=
Operating Cash Cost - Collected Revenue
```

---

# 159. Recommended Founder Financial Controls

Maintain:

```text
3 months minimum technical operating cash
```

before paid launch.

Prefer:

```text
6 months
```

if the business relies on the product full-time.

Technical infrastructure should not be shut down because one monthly invoice surprised the founder.

---

# 160. One-Person SaaS Recommendation

Your greatest cost risk initially is not AWS.

It is:

```text
time
scope expansion
AI-generated rework
premature complexity
```

The phase/sprint/architecture standards already created are therefore part of the costing strategy.

A month lost to rewriting the Building Model is more expensive than months of Neon/R2.

---

# 161. Spending Priority

Spend money first on:

```text
correct architecture
real customer validation
security
backups
usage telemetry
AI cost measurement
```

Spend later on:

```text
fancier infrastructure
premium rendering
enterprise tools
large commercial datasets
```

---

# 162. Cost Register Decision Example

Before adding a service costing $200/month:

```text
Annual cost = $2,400
```

Ask:

```text
Does it create/save at least $2,400/year of value?
```

or:

```text
Does it materially reduce operational/security risk?
```

If not, delay.

---

# 163. Recommended Cash Plan

## Pre-MVP

Keep monthly incremental technical cash near:

```text
$100–$300
```

## Pre-Beta

Allow:

```text
$300–$600
```

## Paid Beta

Budget:

```text
$600–$1,000
```

until real usage shows otherwise.

## After Product-Market Signal

Scale based on:

```text
revenue and measured COGS
```

not arbitrary user projections.

---

# 164. Master Cost Governance Rules

1. [ ] Every paid service has an owner/purpose.
2. [ ] Every variable provider has a budget alert.
3. [ ] AI usage is measured per job.
4. [ ] AI customer credits are separate from provider tokens.
5. [ ] Rendering is credit-controlled.
6. [ ] Free users have cost limits.
7. [ ] Staging does not silently become production-sized.
8. [ ] Preview resources expire.
9. [ ] Provider price changes update rate cards.
10. [ ] Historical cost records are never rewritten.
11. [ ] Plan margins are reviewed monthly.
12. [ ] Quote-based professional licenses are treated separately.
13. [ ] Taxes are not mistaken for margin.
14. [ ] Founder time is tracked separately from cash burn.
15. [ ] 70%+ variable gross margin remains the target.

---

# 165. Recommended Repository Location

Store this file as:

```text
docs/finance/
└── Buildora_AI_Development_Production_Costing_Plan.md
```

Recommended finance docs:

```text
docs/finance/
├── Buildora_AI_Development_Production_Costing_Plan.md
├── cost-register.md
├── monthly-unit-economics.md
└── provider-rate-review.md
```

---

# 166. AGENTS.md / Architecture Instruction

Add:

```text
Before introducing a new paid cloud service, AI model, background
processing provider, commercial SDK, storage layer, monitoring service
or production dependency, read:

docs/finance/Buildora_AI_Development_Production_Costing_Plan.md

Any new recurring service expected to exceed the project's approved
cost threshold must include:
- purpose,
- fixed monthly cost,
- variable pricing unit,
- scale behaviour,
- cancellation/exit path,
- cheaper alternative considered,
- impact on customer gross margin.
```

---

# 167. Cost Assumptions That Must Be Configurable

Never hardcode into permanent business logic:

```text
OpenAI token prices
credit costs
render credit values
plan prices
Stripe rates
storage cost
Temporal action cost
AI model IDs
provider discounts
tax rates
```

Use:

```text
rate cards
plan versions
configuration
```

---

# 168. Current Pricing Source Notes

Pricing checked on 11 September 2026 from current provider pages:

OpenAI:
- GPT-6 Astra model documentation
- GPT-5.6 Sol/Terra/Luna model documentation

AWS:
- Fargate pricing
- Elastic Load Balancing pricing
- ElastiCache pricing

Cloudflare:
- R2 pricing
- application plan pricing

Neon:
- Neon pricing

Temporal:
- Temporal Cloud/AWS Marketplace pricing reference

Stripe:
- United Kingdom pricing
- Billing pricing

Clerk:
- Clerk pricing

Always revisit the provider page before committing production budgets.

---

# 169. Source URLs

OpenAI:
- https://developers.openai.com/api/docs/models/gpt-6-astra
- https://developers.openai.com/api/docs/models/gpt-5.6-sol
- https://developers.openai.com/api/docs/models/gpt-5.6-terra
- https://developers.openai.com/api/docs/models/gpt-5.6-luna

AWS:
- https://aws.amazon.com/fargate/pricing/
- https://aws.amazon.com/elasticloadbalancing/pricing/
- https://aws.amazon.com/elasticache/pricing/

Cloudflare:
- https://developers.cloudflare.com/r2/pricing/
- https://www.cloudflare.com/plans/

Neon:
- https://neon.com/pricing

Temporal:
- https://temporal.io/get-cloud/aws-marketplace

Stripe UK:
- https://stripe.com/gb/pricing

Clerk:
- https://clerk.com/pricing

---

# 170. Final Costing Architecture

```text
                         BUILDORA COST
                               │
            ┌──────────────────┼──────────────────┐
            │                  │                  │
            ▼                  ▼                  ▼
         FIXED               VARIABLE           R&D
       PLATFORM                COGS              COST
            │                  │                  │
     ECS / DB / Redis        AI Tokens       Claude/Codex
     Temporal / ALB          Rendering       Dev API Tests
     Monitoring              Processing      Staging
                             Payments        Founder Time
                                │
                                ▼
                       CUSTOMER USAGE
                                │
                    ┌───────────┴────────────┐
                    ▼                        ▼
               ENTITLEMENTS              CREDITS
                                             │
                                             ▼
                                       COST CONTROL
                                             │
                                             ▼
                                       GROSS MARGIN
```

---

# 171. Final Recommended Financial Position

For Buildora AI as a solo-developed SaaS:

```text
Development:
keep technical cash burn low
≈ $100–$300/month for much of development

Pre-beta:
≈ $300–$600/month

Initial paid production:
fixed technical platform
≈ $300–$600/month

Paid-beta total with usage:
≈ $400–$1,000/month initially

Growth:
let costs scale with revenue and usage
```

Maintain:

```text
70%+ variable gross-margin target
```

and control expensive AI through:

```text
automatic model routing
+
AI Design Credits
+
prompt caching
+
incremental actions
+
usage telemetry
```

---

# 172. Final Principle

The financial architecture of Buildora AI should follow the same principle as the technical architecture:

> **Do not pay for scale before the product has scale, but measure every cost from the beginning so growth never becomes financially surprising.**

The most important equation is:

```text
CUSTOMER VALUE
must grow faster than
VARIABLE COMPUTE COST
```

Buildora AI can achieve this because:

```text
2D
3D
QS
BOQ
Cost
```

are primarily deterministic and inexpensive, while expensive AI/rendering work is intentionally routed, metered and credit-controlled.
