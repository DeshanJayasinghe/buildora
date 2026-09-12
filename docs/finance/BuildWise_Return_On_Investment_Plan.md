# BuildWise — Comprehensive Return on Investment (ROI) Plan

> **File name:** `BuildWise_Return_On_Investment_Plan.md`
>
> **Purpose:** Master ROI, break-even, payback, unit-economics, reinvestment, and investment-gate plan for BuildWise.
>
> **Audience:** Founder, internal management, future investors, financial advisers, and AI development agents.
>
> **Use together with:**
>
> - `BuildWise_Development_Production_Costing_Plan.md`
> - `BuildWise_Billing_Architecture.md`
> - `BuildWise_Phase_Wise_Development_Checklist.md`
> - `BuildWise_Sprintwise_Project_Plan.md`
> - `BuildWise_Deployment_Production_DevOps_Plan.md`
>
> **Important:** This document is a planning model, not a revenue forecast, valuation opinion, tax calculation, or investment guarantee. All growth, conversion, churn, CAC, ARPA and cost assumptions must be replaced with real telemetry as BuildWise launches.
>
> **Currency:** USD unless explicitly stated otherwise.

---

# 1. Executive ROI Strategy

BuildWise should not be judged by one ROI number.

It needs four separate return measures:

```text
1. TECHNICAL BREAK-EVEN
Can customer revenue cover cloud, AI, payment and production costs?

2. FOUNDER CASH PAYBACK
How quickly does collected gross contribution recover the cash invested before launch?

3. ECONOMIC ROI
Does the return justify the economic value of founder development time and future team costs?

4. BUSINESS ROI
After acquisition, salaries, legal, support, tax and operating costs,
does BuildWise produce sustainable profit and company value?
```

These measures answer different questions.

Never say:

```text
"BuildWise is profitable"
```

only because:

```text
MRR > AWS bill
```

---

# 2. Primary Investment Thesis

BuildWise has attractive potential economics because the highest-value professional outputs:

```text
2D design
3D synchronization
quantity calculation
BOQ
cost calculation
```

are mostly deterministic software operations.

The expensive variable operations are concentrated in:

```text
AI generation
expert reasoning
plan recognition
photoreal rendering
```

and those can be:

```text
routed
metered
credit-controlled
cached
limited
```

Therefore the desired model is:

```text
HIGH CUSTOMER VALUE
        +
LOW MARGINAL DETERMINISTIC COMPUTE
        +
CONTROLLED AI / GPU COST
        =
ATTRACTIVE SaaS GROSS MARGIN
```

---

# 3. ROI Definitions

## 3.1 Cash ROI

```text
Cash ROI %
=
(Cumulative Cash Return - Cash Investment)
/
Cash Investment
× 100
```

Example:

```text
Initial cash investment = $10,000
Cumulative returned contribution = $25,000

ROI
=
($25,000 - $10,000) / $10,000
=
150%
```

---

# 4. Payback Period

```text
Payback Period
=
Initial Investment
/
Monthly Net Contribution
```

Example:

```text
Initial cash investment = $10,000
Monthly net contribution = $2,000

Payback ≈ 5 months
```

Use stable monthly contribution, not one unusually strong month.

---

# 5. Gross Margin

```text
Gross Margin
=
Revenue - COGS
---------------
Revenue
```

BuildWise target:

```text
70%+ variable gross margin
```

over time.

Early beta may be lower because fixed platform costs are spread across very few customers.

---

# 6. Contribution Margin per Paid Customer

```text
Paid Customer Contribution
=
ARPA
-
Payment Fees
-
AI Cost
-
Render Cost
-
Variable Infrastructure
-
Other Direct Variable Cost
```

This amount contributes toward:

- shared platform cost,
- founder salary,
- support,
- sales/marketing,
- profit.

---

# 7. Economic ROI

Founder time has value even if no salary is paid.

```text
Economic Investment
=
Cash Investment
+
Founder Development Time Value
+
Unpaid Professional Work Value
```

Economic ROI:

```text
Economic ROI
=
Cumulative Business Return - Economic Investment
-----------------------------------------------
Economic Investment
```

This gives a more realistic strategic comparison against:

- employment,
- consulting,
- another SaaS idea,
- hiring developers.

---

# 8. Full Business ROI

Eventually:

```text
Net Operating Return
=
Revenue
-
COGS
-
Sales & Marketing
-
Payroll
-
Support
-
Legal
-
Insurance
-
Admin
-
Other Operating Expenses
```

The true company ROI uses this number.

This document intentionally keeps:

```text
technical/product ROI
```

separate from:

```text
full corporate profitability
```

until real operating expenses exist.

---

# 9. Core Commercial Assumptions

Illustrative plan architecture:

| Plan | Monthly Price |
|---|---:|
| Free | $0 |
| Home | $14.99 |
| Pro | $49 |
| Business | $149 |
| Enterprise | Custom |

These are working commercial assumptions and remain versioned/configurable.

---

# 10. Paid Plan Variable COGS Targets

Illustrative monthly direct COGS targets:

| Plan | Target Variable COGS |
|---|---:|
| Home | ~$4 |
| Pro | ~$14 |
| Business | ~$38 |

These figures include an approximate blend of:

- payment/billing fees,
- AI,
- allocated variable infrastructure,
- storage,
- processing,
- normal included rendering.

They are planning targets rather than guarantees.

---

# 11. Free User Cost Target

Target monthly Free MAU cost:

```text
$0.10–$0.50
```

ROI model baseline:

```text
$0.15 / Free MAU / month
```

Free users are an acquisition cost.

They should be treated intentionally.

---

# 12. Three Paid-Mix Scenarios

## Conservative Mix

```text
Home      50%
Pro       45%
Business   5%
```

Blended monthly revenue/account:

```text
≈ $36.70
```

Blended direct paid-account COGS:

```text
≈ $10.20
```

---

# 13. Base Mix

```text
Home      35%
Pro       55%
Business  10%
```

Blended monthly revenue/account:

```text
≈ $47.10
```

Blended direct paid-account COGS:

```text
≈ $12.90
```

---

# 14. Upside Mix

```text
Home      25%
Pro       55%
Business  20%
```

Blended monthly revenue/account:

```text
≈ $60.50
```

Blended direct paid-account COGS:

```text
≈ $16.30
```

The upside comes primarily from a greater share of Business accounts.

---

# 15. Why ARPA Matters More Than User Count

Two products can both have:

```text
10,000 MAU
```

but one may generate:

```text
$20,000 MRR
```

and another:

```text
$100,000 MRR
```

depending on:

- paid conversion,
- plan mix,
- top-ups,
- enterprise accounts.

Therefore BuildWise should optimize:

```text
Qualified MAU
→ Paid conversion
→ Pro/Business adoption
→ Retention
```

not vanity signups.

---

# 16. ROI Stage Plan

BuildWise ROI should be managed through five stages:

```text
ROI Stage 0
Pre-revenue Validation

ROI Stage 1
Technical Break-Even

ROI Stage 2
Founder Cash Payback

ROI Stage 3
Sustainable Solo SaaS

ROI Stage 4
Growth / Enterprise Expansion
```

---

# 17. ROI Stage 0 — Pre-Revenue Validation

## Investment

Expected direct development cash:

```text
Lean:
$2,000–$5,000

Comfortable:
$5,000–$10,000
```

before major commercial licenses.

## Return Objective

The return is not revenue yet.

It is risk reduction.

The development investment should prove:

```text
Can BuildWise produce a useful design?
Can the same model drive 2D and 3D?
Can quantities be trusted?
Can BOQ/cost be produced?
Will professionals pay?
```

## Success Gate

Before major additional investment, obtain evidence such as:

- [ ] 10+ relevant user interviews.
- [ ] 5+ users test a prototype.
- [ ] 3+ professionals say the workflow saves meaningful time.
- [ ] at least several users express willingness to pay.
- [ ] QS/BOQ output is understandable/trustworthy enough for beta.
- [ ] one clear target customer segment emerges.

## Stop / Reframe Gate

Do not keep spending because code already exists.

Reconsider if:

- [ ] users like 3D visuals but do not value QS/cost.
- [ ] professional users do not trust generated data.
- [ ] workflow saves little time.
- [ ] no viable pricing willingness appears.
- [ ] required data licenses make pricing uneconomic.

---

# 18. ROI Stage 1 — Technical Break-Even

Technical break-even means:

```text
monthly subscription contribution
≥
shared production platform cost
```

It does **not** mean founder/company profitability.

---

# 19. Base Year-1 Technical Break-Even

Illustrative base assumptions:

```text
Blended ARPA              $47.10
Paid direct COGS          $12.90
Paid conversion              6%
Free cost / MAU            $0.15
Shared platform/month       $750
```

At 6% paid conversion, each paid account is economically associated with approximately:

```text
15.67 free MAU
```

Free-user subsidy per paid account:

```text
15.67 × $0.15
≈ $2.35
```

Effective contribution per paid account:

```text
$47.10
-
$12.90
-
$2.35
≈
$31.85/month
```

Technical break-even:

```text
$750 / $31.85
≈
24 paying accounts
```

Planning target:

> **Reach 25–30 paying accounts to cover an early ~$750/month technical platform.**

---

# 20. Conservative Technical Break-Even

Illustrative:

```text
ARPA                         $36.70
Paid COGS                    $10.20
Conversion                      4%
Free subsidy / paid acct      $3.60
Effective contribution       $22.90
Platform                       $600
```

Break-even:

```text
$600 / $22.90
≈
27 paid accounts
```

---

# 21. Upside Technical Break-Even

Illustrative:

```text
ARPA                         $60.50
Paid COGS                    $16.30
Conversion                      8%
Free subsidy / paid acct      $1.73
Effective contribution       $42.47
Platform                     $1,000
```

Break-even:

```text
$1,000 / $42.47
≈
24 paid accounts
```

The result is similar because the infrastructure budget grows with the stronger scenario.

---

# 22. Technical Break-Even Target

Use this as the first commercial milestone:

```text
25–30 paid-equivalent accounts
```

where a Business account may contribute more than one Home account.

This is a stronger target than:

```text
1,000 registrations
```

---

# 23. ROI Stage 2 — Founder Cash Payback

Suppose pre-launch direct cash investment:

```text
$10,000
```

and after technical fixed costs, BuildWise produces:

```text
$2,500 monthly contribution
```

Cash payback:

```text
$10,000 / $2,500
=
4 months
```

If contribution is:

```text
$1,000/month
```

payback is:

```text
10 months
```

---

# 24. Recommended Founder Cash Payback Goal

Target:

```text
12–18 months from paid launch
```

to recover direct pre-launch cash.

Why not demand immediate repayment?

Because early cash should also fund:

- product improvement,
- customer support,
- acquisition,
- production reliability.

---

# 25. Cash Payback Milestones

## Milestone A

```text
MRR = technical platform cost
```

## Milestone B

```text
monthly contribution = $1,000+
```

## Milestone C

```text
cumulative contribution = pre-launch cash investment
```

## Milestone D

```text
founder can draw sustainable compensation
without starving growth
```

---

# 26. ROI Stage 3 — Sustainable Solo SaaS

Target:

```text
150–300 paid accounts
```

Using base blended ARPA:

```text
150 × $47.10
≈ $7,065 MRR

300 × $47.10
≈ $14,130 MRR
```

Before top-ups/enterprise.

---

# 27. Contribution at 150 Paid Accounts

Base direct paid COGS:

```text
150 × $12.90
≈ $1,935
```

At approximately 8% conversion:

```text
free MAU per paid ≈ 11.5
```

Free subsidy:

```text
150 × 11.5 × $0.15
≈ $259
```

Assume shared platform:

```text
$1,500/month
```

Approximate technical contribution:

```text
$7,065
-
$1,935
-
$259
-
$1,500
≈
$3,371/month
```

before:

- founder salary,
- marketing,
- legal,
- tax.

---

# 28. Contribution at 300 Paid Accounts

Revenue:

```text
≈ $14,130
```

Direct paid COGS:

```text
≈ $3,870
```

Free subsidy:

```text
≈ $518
```

Assume shared platform:

```text
$2,000
```

Approximate technical contribution:

```text
≈ $7,742/month
```

This is the point where the SaaS can begin meaningfully funding:

- founder compensation,
- customer acquisition,
- additional engineering.

---

# 29. ROI Stage 4 — Growth

Target:

```text
1,000+ paid accounts
```

Base revenue:

```text
1,000 × $47.10
≈ $47,100 MRR
```

Annualized:

```text
≈ $565,200 ARR
```

before top-ups/enterprise.

---

# 30. 1,000-Paid Technical Contribution Example

Assume:

```text
10,000 MAU
1,000 paid
9,000 free

ARPA                   $47.10
Paid COGS/account      $12.90
Free COGS/MAU           $0.15
Shared platform         $5,000/month
```

Monthly:

```text
Revenue:
≈ $47,100

Paid variable COGS:
≈ $12,900

Free subsidy:
≈ $1,350

Platform:
≈ $5,000
```

Technical contribution:

```text
≈ $27,850/month
```

Technical margin:

```text
≈ 59%
```

This includes a relatively generous $5,000 shared platform allocation.

Further optimization should drive the mature gross margin toward 70%+.

---

# 31. Unit Economics — Base Case

Base blended:

```text
ARPA = $47.10
Paid direct COGS = $12.90
```

Direct paid-account contribution before free subsidy/shared infrastructure:

```text
$34.20/month
```

Direct contribution margin:

```text
≈ 72.6%
```

This is close to BuildWise's long-term goal.

---

# 32. LTV Formula

Simple gross-contribution LTV:

```text
LTV
=
Monthly Contribution
/
Monthly Churn
```

This is a simplified SaaS model.

It assumes relatively stable contribution/churn.

---

# 33. LTV Sensitivity — Base Contribution

Using:

```text
Monthly contribution ≈ $34.20
```

## 3% Monthly Churn

```text
LTV
≈ $1,140
```

## 5% Monthly Churn

```text
LTV
≈ $684
```

## 8% Monthly Churn

```text
LTV
≈ $428
```

Retention has enormous effect on ROI.

---

# 34. Why Churn Matters

Improving monthly churn:

```text
8%
→
4%
```

roughly doubles simplified LTV.

Therefore:

> improving professional usefulness and retention can generate more ROI than cutting cloud cost by 20%.

---

# 35. LTV:CAC Target

A common SaaS planning rule:

```text
LTV:CAC ≥ 3:1
```

BuildWise can initially target:

```text
3:1 minimum
5:1 healthy for founder-led organic growth
```

Do not over-optimize for extremely high LTV:CAC if it simply means underinvesting in growth.

---

# 36. CAC Payback

Using base monthly paid-account contribution:

```text
≈ $34.20
```

Approximate CAC payback:

| CAC | Payback |
|---:|---:|
| $50 | 1.5 months |
| $100 | 2.9 months |
| $150 | 4.4 months |
| $200 | 5.8 months |
| $300 | 8.8 months |
| $500 | 14.6 months |

---

# 37. CAC Target

Early founder-led BuildWise target:

```text
CAC < $150
```

ideally via:

- organic content,
- referrals,
- professional communities,
- direct demos,
- construction partnerships.

At scale, CAC can rise if LTV/retention remain strong.

---

# 38. Maximum CAC Formula

If target:

```text
LTV:CAC = 3
```

Then:

```text
Max CAC
=
LTV / 3
```

At 5% churn:

```text
LTV ≈ $684

Max CAC ≈ $228
```

At 3% churn:

```text
LTV ≈ $1,140

Max CAC ≈ $380
```

This shows why retention unlocks acquisition spend.

---

# 39. Monthly Churn Targets

Early:

```text
< 8%
```

Good:

```text
< 5%
```

Strong professional SaaS:

```text
< 3%
```

Business/enterprise contracts should be evaluated separately with:

```text
logo retention
NRR
contract renewal
```

---

# 40. Net Revenue Retention

At Business/Enterprise maturity track:

```text
NRR
=
Starting recurring revenue
+ upgrades
- downgrades
- churn
-----------------------------
Starting recurring revenue
```

Target over time:

```text
100%+
```

Strong B2B expansion:

```text
110–120%+
```

Not required for MVP.

---

# 41. 36-Month Scenario Framework

The following scenarios use average MAU for each year, not year-end user counts.

They are planning illustrations.

---

# 42. Conservative Scenario Assumptions

Average MAU:

```text
Year 1   1,500
Year 2   6,000
Year 3  20,000
```

Paid conversion:

```text
4%
5%
6%
```

Average paid accounts:

```text
60
300
1,200
```

Paid mix:

```text
50% Home
45% Pro
5% Business
```

Blended ARPA:

```text
$36.70
```

Pre-launch cash:

```text
$8,000
```

---

# 43. Conservative 3-Year Technical Model

| Year | Avg MAU | Avg Paid | Revenue | Technical Cost | Technical Contribution |
|---:|---:|---:|---:|---:|---:|
| 1 | 1,500 | 60 | ~$26,421 | ~$17,136 | ~$9,285 |
| 2 | 6,000 | 300 | ~$132,104 | ~$58,980 | ~$73,124 |
| 3 | 20,000 | 1,200 | ~$528,415 | ~$210,720 | ~$317,695 |

Cumulative technical contribution after initial $8,000 cash:

```text
≈ $392,104
```

Important:

This is **not net company profit**.

It excludes:

- salary,
- marketing,
- support staff,
- legal,
- tax,
- commercial data licenses.

---

# 44. Base Scenario Assumptions

Average MAU:

```text
Year 1    3,000
Year 2   15,000
Year 3   60,000
```

Paid conversion:

```text
6%
8%
10%
```

Average paid accounts:

```text
180
1,200
6,000
```

Paid mix:

```text
35% Home
55% Pro
10% Business
```

ARPA:

```text
≈ $47.10
```

Pre-launch cash:

```text
$10,000
```

---

# 45. Base 3-Year Technical Model

| Year | Avg MAU | Avg Paid | Revenue | Technical Cost | Technical Contribution |
|---:|---:|---:|---:|---:|---:|
| 1 | 3,000 | 180 | ~$101,728 | ~$41,940 | ~$59,788 |
| 2 | 15,000 | 1,200 | ~$678,190 | ~$234,600 | ~$443,590 |
| 3 | 60,000 | 6,000 | ~$3,390,948 | ~$1,086,000 | ~$2,304,948 |

Cumulative technical contribution after $10,000 initial cash:

```text
≈ $2.80M
```

Again:

> This is a scenario model of product technical contribution, not a forecast of net profit.

At this scale full company operating costs would be much larger than the solo-founder model.

---

# 46. Upside Scenario Assumptions

Average MAU:

```text
Year 1     5,000
Year 2    30,000
Year 3   150,000
```

Paid conversion:

```text
8%
10%
12%
```

Average paid:

```text
400
3,000
18,000
```

Paid mix:

```text
25% Home
55% Pro
20% Business
```

ARPA:

```text
≈ $60.50
```

Pre-launch cash:

```text
$15,000
```

---

# 47. Upside 3-Year Technical Model

| Year | Avg MAU | Avg Paid | Revenue | Technical Cost | Technical Contribution |
|---:|---:|---:|---:|---:|---:|
| 1 | 5,000 | 400 | ~$290,388 | ~$98,520 | ~$191,868 |
| 2 | 30,000 | 3,000 | ~$2,177,910 | ~$683,400 | ~$1,494,510 |
| 3 | 150,000 | 18,000 | ~$13,067,460 | ~$3,938,400 | ~$9,129,060 |

This scenario would require a real team and substantial non-technical operating spend.

Do not interpret this table as a solo-founder profit forecast.

---

# 48. Scenario Interpretation

The most useful lesson is not the large Year-3 numbers.

It is:

```text
paid conversion
×
ARPA
×
retention
```

creates far more leverage than minor infrastructure savings.

BuildWise ROI is highly sensitive to product-market fit.

---

# 49. Paid Conversion Sensitivity

Assume:

```text
10,000 MAU
ARPA = $47.10
```

Monthly subscription revenue:

| Paid Conversion | Paid Accounts | MRR |
|---:|---:|---:|
| 2% | 200 | ~$9,420 |
| 5% | 500 | ~$23,550 |
| 8% | 800 | ~$37,680 |
| 10% | 1,000 | ~$47,100 |
| 15% | 1,500 | ~$70,650 |

A few percentage points of conversion can be worth more than months of cloud optimization.

---

# 50. ARPA Sensitivity

At:

```text
1,000 paid accounts
```

| ARPA | MRR |
|---:|---:|
| $20 | $20,000 |
| $30 | $30,000 |
| $40 | $40,000 |
| $47.10 | ~$47,100 |
| $60 | $60,000 |
| $80 | $80,000 |

This is why professional/business value matters.

---

# 51. AI Cost Sensitivity — Pro

Pro price:

```text
$49
```

Assume non-AI direct COGS:

```text
$7
```

Then:

| Monthly AI Cost | Total COGS | Contribution | Margin |
|---:|---:|---:|---:|
| $2 | $9 | $40 | 81.6% |
| $5 | $12 | $37 | 75.5% |
| $8 | $15 | $34 | 69.4% |
| $12 | $19 | $30 | 61.2% |
| $20 | $27 | $22 | 44.9% |

Operational goal:

```text
average Pro AI cost ≲ $6–$8
```

unless extra credits generate additional revenue.

---

# 52. AI Credit ROI

AI Design Credits should create:

```text
predictable value
+
bounded cost
```

Example:

```text
30 credits included
Expected AI cost ≈ $6
```

If customer perceives those design actions as worth:

```text
$20–$50+
```

of professional time/value, the unit economics are attractive.

---

# 53. Top-Up ROI

Top-ups can convert heavy users from margin risk into expansion revenue.

Example:

```text
25 Design Credits = $10
```

If expected provider/processing cost:

```text
≈ $5
```

before payment fee:

```text
gross contribution ≈ $5
```

If cost is:

```text
$8
```

the pack is underpriced.

Therefore top-up prices must be updated from observed:

```text
cost per consumed credit
```

---

# 54. Render ROI

Rendering should not be treated as a mandatory core feature until customers demonstrate willingness to pay.

Decision:

```text
If rendering materially improves acquisition/upsell
AND
credit price exceeds GPU cost comfortably
→ invest
```

Otherwise keep:

```text
browser real-time 3D
```

as the low-cost default.

---

# 55. Feature ROI Framework

Every major feature should be judged on one or more:

```text
Acquisition
Conversion
ARPA
Retention
Expansion
Cost Reduction
Risk Reduction
```

A feature that improves none of these needs strong strategic justification.

---

# 56. Feature ROI Score

Use:

```text
Feature ROI Score
=
Expected Annual Incremental Contribution
/
Development + Operating Cost
```

Example:

```text
IFC export costs $8,000 economic development effort
and is expected to add $20,000/year contribution

ROI multiple = 2.5×
```

This is directional, not precise.

---

# 57. Canonical Building Model ROI

Classification:

```text
MANDATORY PLATFORM INVESTMENT
```

Direct monetization:

```text
low
```

Strategic ROI:

```text
extremely high
```

because it enables:

- 2D,
- 3D,
- QS,
- BOQ,
- AI actions,
- BIM.

Do not judge platform foundations only by direct feature revenue.

---

# 58. 2D Editor ROI

Primary effect:

```text
product usability
retention
professional credibility
```

Must-have for:

- manual design,
- correction,
- recognition verification.

Build before expensive secondary features.

---

# 59. 3D ROI

Primary effects:

- acquisition,
- conversion,
- homeowner appeal,
- professional communication.

Cost is attractive because browser GPU does most normal rendering.

Therefore 3D has strong visual ROI.

---

# 60. QS / BOQ / Cost ROI

Likely strongest monetization layer.

Effects:

```text
Pro conversion
Business conversion
retention
professional differentiation
```

This is where BuildWise moves from:

```text
design toy
```

to:

```text
construction intelligence product
```

High ROI priority.

---

# 61. AI Copilot ROI

Potential effects:

- conversion,
- retention,
- onboarding,
- user productivity,
- premium credits.

Risk:

- direct variable cost.

Therefore AI Copilot should be measured by:

```text
AI cost
vs
retention / conversion / expansion
```

not merely number of messages sent.

---

# 62. Plan Recognition ROI

Potential:

```text
very high
```

because users can onboard existing drawings quickly.

Metrics:

- upload → verified model conversion,
- minutes saved,
- recognition correction time,
- conversion rate of users who upload plans.

If users spend more time fixing recognition than drawing manually, ROI is weak.

---

# 63. IFC ROI

Likely professional upsell feature.

Build when:

- architects/QS/builders request it,
- Pro/Business users need interoperability.

Good candidate for:

```text
Pro+
```

rather than Free.

---

# 64. RAG / Documents ROI

Likely retention/business value:

```text
"Ask this project"
```

Usefulness grows as project documents grow.

Measure:

- users asking document questions,
- successful cited answers,
- Business conversion.

---

# 65. Scheduling ROI

Do not build early.

Build if:

```text
customers want BuildWise beyond estimating
```

and it improves:

```text
Business/Enterprise ARPA
```

---

# 66. Procurement ROI

Potentially large long-term revenue:

```text
supplier leads
commissions
Business subscription
```

but also high complexity.

Requires customer traction first.

---

# 67. Marketplace ROI

Do not rely on marketplace revenue in the initial investment case.

Treat as upside.

Primary SaaS must work independently.

---

# 68. ROI Priority Matrix

## Tier A — Mandatory / Highest ROI

```text
Building Model
2D
3D
QS
BOQ
Cost
Projects/SaaS
Billing
```

## Tier B — Strong Differentiation

```text
AI Copilot
Plan Recognition
IFC
Documents/RAG
Reports
```

## Tier C — Post-PMF Expansion

```text
Schedule
Procurement
Actual Cost
Change Orders
Mobile
```

## Tier D — Strategic Optional

```text
Marketplace
advanced GIS
full MEP
full structural engineering
multi-region enterprise infrastructure
```

---

# 69. Development ROI Gate per Phase

Before beginning each major phase ask:

1. Does it unlock another critical module?
2. Does it improve acquisition/conversion/ARPA/retention?
3. Is it required for launch?
4. Is there customer evidence?
5. What cash cost begins?
6. What ongoing variable cost begins?
7. How will success be measured?

If none has a strong answer:

```text
defer
```

---

# 70. Customer Acquisition ROI

Initial channels should favor low CAC.

Priority:

```text
Founder-led demos
Construction community
Architect/QS relationships
LinkedIn content
YouTube/tutorial content
SEO/problem-specific content
Referral
Industry partnerships
```

Avoid large paid advertising budget before activation and retention are proven.

---

# 71. Organic Content ROI

Content can have high founder-time cost but low cash CAC.

Measure:

```text
content hours
→ qualified visitors
→ signups
→ activated projects
→ paid customers
```

Do not count views as ROI.

---

# 72. Paid Ads ROI

Only scale when:

```text
CAC payback
+
retention
```

are understood.

If:

```text
CAC = $250
```

and base monthly contribution:

```text
$34
```

payback:

```text
~7.4 months
```

Can be acceptable with strong retention.

If churn is high:

do not scale ads.

---

# 73. Founder Sales ROI

For Business/Enterprise:

```text
founder hours
× internal hourly rate
```

is part of CAC.

Example:

```text
5 hours of calls/onboarding
× $40 economic rate
=
$200 founder-sales cost
```

If Business account generates:

```text
$149/month
```

with strong retention, this can be attractive.

---

# 74. Enterprise ROI

Enterprise deals need a minimum annual value.

Avoid:

```text
$2,000/year contract
```

requiring:

- custom integration,
- SSO,
- security review,
- dedicated support,
- custom data imports.

Create:

```text
minimum enterprise contract value
```

based on implementation/support cost.

---

# 75. Enterprise Deal ROI Formula

```text
First-Year Deal Contribution
=
Contract Value
-
Implementation Cost
-
Support Cost
-
Variable COGS
-
Sales Cost
```

Require positive first-year contribution unless strategic value is explicit.

---

# 76. Founder Time ROI

Track development time by major module:

```text
Building Model
2D
3D
QS
AI
Recognition
Billing
```

Then ask:

```text
Which modules produced actual customer willingness to pay?
```

Future time should follow evidence.

---

# 77. Economic Founder Payback

Example:

```text
Founder time value = $50,000
Cash investment    = $10,000

Economic investment = $60,000
```

If annual post-COGS contribution reaches:

```text
$60,000
```

economic payback:

```text
~1 year
```

before additional operating expenses.

---

# 78. Founder Compensation Gate

Do not withdraw all early contribution.

Suggested progression:

## Before technical break-even

```text
$0 product-funded founder salary
```

## After 3 stable profitable months

allow modest founder draw.

## After 6+ months stable MRR

target predictable compensation.

## Before hiring

ensure:

```text
runway + recurring contribution
```

supports the role.

---

# 79. Reinvestment Plan

Suggested allocation after product produces meaningful monthly contribution.

Early growth:

```text
40% product/engineering
25% acquisition
15% reserves
10% customer success/support
10% founder distribution
```

Illustrative only.

At later maturity allocations change.

---

# 80. Cash Reserve

Before aggressive reinvestment maintain:

```text
at least 3 months technical/operating cash
```

preferably:

```text
6 months
```

for founder-dependent SaaS.

---

# 81. Hiring ROI Gate

Do not hire because:

```text
"we are busy"
```

Hire when:

```text
expected incremental contribution
>
fully loaded employee/contractor cost
```

or the hire materially reduces critical operational risk.

---

# 82. First Developer Hire

Potential trigger:

- founder is bottleneck,
- MRR stable,
- backlog has validated revenue features,
- support is consuming development time.

Example cost:

```text
Developer fully loaded = X/year
```

Require confidence that the business has:

```text
12+ months runway
```

or sufficient recurring contribution.

---

# 83. Sales Hire ROI

Only after founder can repeatedly sell.

Do not hire salesperson to discover product-market fit.

Target:

```text
repeatable ICP
known deal cycle
known close rate
known contract value
```

---

# 84. Support Hire ROI

Trigger when founder support time:

```text
blocks roadmap/revenue
```

Measure:

```text
support hours/week
```

against cost of support staff/contractor.

---

# 85. Investment / Funding ROI

External capital is not automatically positive ROI.

Raise if capital can accelerate:

```text
validated growth
```

more than dilution/control cost.

Do not raise simply to fund premature infrastructure.

---

# 86. Bootstrapped ROI Plan

Recommended default for BuildWise initially.

Goals:

```text
low pre-launch cash
fast validation
technical break-even
cash-flow-funded growth
```

Target progression:

```text
30 paid
→ technical break-even

100 paid
→ meaningful validation

250 paid
→ sustainable solo SaaS potential

1,000 paid
→ growth-company decision
```

---

# 87. Growth-Funded ROI Plan

Consider external investment only after signals such as:

- strong retention,
- repeatable conversion,
- Pro/Business adoption,
- attractive gross margin,
- clear large market.

Use funding for:

```text
sales
product speed
data licensing
enterprise integrations
international expansion
```

not simply AWS bills.

---

# 88. ROI Stop-Loss Gates

Every project needs stop-loss rules.

## Gate 1 — Prototype

If professionals do not value workflow:

```text
reframe before major spending
```

## Gate 2 — Beta

If activated users do not return:

```text
fix retention before paid acquisition
```

## Gate 3 — Paid Launch

If customers will not pay enough to support COGS:

```text
change packaging/pricing/product
```

## Gate 4 — Growth

If CAC payback > 12–18 months with weak retention:

```text
stop scaling paid acquisition
```

---

# 89. Activation Metrics

A signup is not an activated BuildWise user.

Suggested activation:

```text
Create project
+
Create/import plan
+
Open 3D
+
View quantity/cost result
```

Measure:

```text
signup → activation %
```

ROI depends on this funnel.

---

# 90. Activation Target

Initial beta:

```text
> 30%
```

Good:

```text
> 50%
```

depending traffic quality.

Do not scale marketing until activation is understandable.

---

# 91. Trial-to-Paid Conversion

Measure:

```text
activated trial users
→ paid
```

Not all raw signups.

Initial target range:

```text
5–10%
```

for broad self-serve traffic may be a useful benchmark to test against, but BuildWise should use its own data.

Professional targeted traffic may be much higher.

---

# 92. Funnel ROI

Track:

```text
Visitors
↓
Signup
↓
Activated
↓
Trial engaged
↓
Paid
↓
Retained
↓
Expanded
```

Optimize the weakest economically meaningful step.

---

# 93. Pricing Experiment ROI

Do not optimize price only for signup conversion.

Measure:

```text
Revenue per visitor
Gross contribution per visitor
90-day retained revenue
```

A $29 plan with higher conversion may still perform worse than a $49 plan.

---

# 94. Discount ROI

Discount only if it improves:

- conversion,
- annual commitment,
- retention,
- cash flow.

Do not create permanent low-paying customers with heavy AI usage.

---

# 95. Annual Plan ROI

Annual plans improve:

```text
cash flow
retention
payment processing frequency
```

Offer discount only if:

```text
retention/cash value > discount cost
```

Example:

```text
2 months free
```

is a ~16.7% annual discount.

Test against churn.

---

# 96. Annual AI Credit Policy

Annual plan should still grant AI credits monthly.

Why:

```text
protects unit economics
prevents one-month annual credit exhaustion
```

This improves ROI stability.

---

# 97. Free Plan ROI

The Free plan exists to:

```text
acquire
educate
activate
convert
```

It is not a charitable unlimited service.

Track:

```text
Free cost
Free → paid conversion
Time to conversion
```

---

# 98. Free Cohort ROI

Example:

```text
1,000 Free users
Monthly free cost = $150
20 eventually convert
```

Acquisition subsidy:

```text
$150 / 20
=
$7.50
```

per converted user for one month of free subsidy.

This can be excellent.

If only 1 converts:

```text
$150 acquisition subsidy
```

before marketing.

Review.

---

# 99. Free Plan Stop-Loss

Reduce expensive Free allowances if:

```text
Free COGS grows
without conversion or referral value
```

Do not reduce useful deterministic functionality unnecessarily.

Control AI/render first.

---

# 100. Home Plan ROI

Purpose:

```text
consumer/self-builder conversion
```

Should be:

- low support,
- lower AI allowance,
- simple estimates.

If Home users require heavy professional support, pricing/model is wrong.

---

# 101. Pro Plan ROI

Likely core commercial plan.

Purpose:

```text
architects
designers
QS
builders
professional self-employed
```

Target:

```text
70%+ contribution margin
strong retention
top-up expansion
```

---

# 102. Business Plan ROI

Purpose:

```text
team usage
shared libraries
organization controls
higher storage/credits
```

Business should increase ARPA without proportionally increasing support/COGS.

---

# 103. Enterprise ROI

Purpose:

```text
high contract value
security/compliance
integration
support
```

Avoid underpriced customization.

---

# 104. Gross Margin by Plan — Example Targets

| Plan | Price | Target COGS | Target Gross Margin |
|---|---:|---:|---:|
| Home | $14.99 | $4 | ~73% |
| Pro | $49 | $14 | ~71% |
| Business | $149 | $38 | ~74% |

These are design targets.

Real telemetry overrides them.

---

# 105. Expansion Revenue ROI

Expansion includes:

```text
top-ups
plan upgrades
extra seats later
storage add-ons
enterprise modules
```

Target:

```text
5–20%+ of recurring revenue
```

over time if users receive real incremental value.

Do not depend on expansion for initial survival.

---

# 106. Top-Up Conversion

Track:

```text
% paid customers purchasing top-ups
average top-up revenue
top-up gross margin
```

Heavy top-up use may indicate:

- healthy expansion,
- or included credits too low.

Interpret with satisfaction data.

---

# 107. Value-Based ROI for Customers

BuildWise should communicate user ROI, not only features.

Example professional:

```text
Manual takeoff + BOQ = 4 hours
BuildWise-assisted = 45 minutes

Time saved ≈ 3.25 hours
```

If professional time value:

```text
$40/hour
```

Customer value:

```text
≈ $130
```

A $49 Pro plan can be compelling.

---

# 108. Customer ROI Formula

```text
Customer ROI
=
(Time Saved Value
+ Error Reduction Value
+ Revenue/Decision Value
- Subscription Cost)
/
Subscription Cost
```

BuildWise marketing should eventually quantify this with real customer studies.

---

# 109. Customer ROI Goal

Aim for:

```text
5×+ perceived/quantified monthly value
```

relative to subscription price.

For $49 Pro:

```text
target customer value:
$250+/month
```

through:

- saved professional time,
- better estimating,
- fewer revisions,
- faster proposals.

---

# 110. Case Study ROI

After launch gather:

```text
Before BuildWise
hours/process
cost/process

After BuildWise
hours/process
cost/process
```

Use real case studies.

This can materially reduce CAC.

---

# 111. ROI Dashboard — Founder

Monthly dashboard:

```text
MRR
ARR
Paid accounts
ARPA
Paid conversion
Logo churn
Revenue churn
NRR
COGS
Gross margin
AI cost
Render cost
Free subsidy
CAC
CAC payback
LTV
LTV:CAC
Cash burn
Runway
```

---

# 112. ROI Dashboard — Product

Track:

```text
activation
time to first model
time to first 3D
time to first BOQ
AI usage
AI success
recognition success
report exports
```

These explain financial outcomes.

---

# 113. ROI Dashboard — Plan

For each plan:

```text
users
MRR
AI cost
support burden
storage
churn
upgrade rate
gross margin
```

Do not assume all plans are equally healthy.

---

# 114. Cohort Analysis

Measure customers by signup month.

Track:

```text
Month 0
Month 1
Month 3
Month 6
Month 12
```

Metrics:

- retained,
- MRR,
- AI cost,
- upgrade.

Cohorts reveal whether product improvements actually improve ROI.

---

# 115. Payback Cohort

For paid acquisition cohort:

```text
Cumulative gross contribution
```

until it exceeds:

```text
CAC
```

This gives real CAC payback.

---

# 116. Customer Segment ROI

Separate:

```text
Homeowner
Architect
QS
Builder
Software/Construction Company
```

One segment may have:

- higher conversion,
- higher ARPA,
- lower churn.

Focus investment there.

---

# 117. Geographic ROI

Track:

```text
UK
India
Sri Lanka
EU
US
```

by:

- ARPA,
- payment cost,
- AI cost,
- support,
- conversion.

Do not expand internationally only because the app technically works there.

---

# 118. Regional Pricing ROI

Regional pricing can improve conversion but reduce ARPA.

Measure:

```text
Gross contribution / visitor
```

not only paid conversion.

---

# 119. Professional Data License ROI

Before purchasing a $X/year cost database:

```text
Required Incremental Contribution
>
License Cost
```

Prefer:

```text
2×+ expected first-year contribution
```

before committing, unless license is essential to product credibility.

---

# 120. Commercial CAD SDK ROI

If ODA/DWG license costs:

```text
$X/year
```

calculate:

```text
customers requiring DWG
×
incremental ARPA/conversion
```

If expected annual contribution is below license + engineering/support cost:

```text
defer
```

---

# 121. AI Model Upgrade ROI

Do not choose Astra because it is "best".

Compare:

```text
Quality improvement
Conversion/retention improvement
Failure reduction
```

against:

```text
incremental model cost
```

Use A/B evaluation.

---

# 122. AI Routing ROI

Suppose:

```text
100,000 monthly AI calls
```

If routing reduces average call cost by:

```text
$0.05
```

monthly saving:

```text
$5,000
```

At scale, routing engineering has direct measurable ROI.

---

# 123. Prompt Caching ROI

If caching reduces input cost by:

```text
$1,000/month
```

and implementation/maintenance cost is:

```text
$3,000 economic effort
```

payback:

```text
3 months
```

Good optimization.

---

# 124. Infrastructure Optimization ROI

Do not spend a week saving:

```text
$20/month
```

unless it also improves reliability.

Founder engineering time may cost more than the saving.

---

# 125. FinOps ROI Rule

Optimization engineering is worthwhile when:

```text
12-month expected saving
>
2× engineering economic cost
```

as a useful default.

Exceptions:

- security,
- reliability,
- scalability blockers.

---

# 126. Database Upgrade ROI

Upgrade database when:

```text
developer/productivity/customer performance benefit
>
incremental monthly cost
```

Do not waste hours optimizing an underpowered $15 DB if a $50 plan solves the bottleneck safely.

Founder time matters.

---

# 127. Build vs Buy ROI

Formula:

```text
Build Cost
=
Engineering Time
+
Maintenance
+
Hosting
+
Risk

Buy Cost
=
Subscription
+
Integration
+
Vendor Risk
```

Choose lower total economic cost that still meets strategic needs.

---

# 128. Example — Temporal

Self-hosting may look cheaper than:

```text
$100/month
```

but requires:

- DB,
- deployment,
- monitoring,
- upgrades,
- incident response.

For one-person SaaS, Temporal Cloud may have positive ROI because founder time is more valuable than raw hosting.

---

# 129. Example — Managed Postgres

Managed PostgreSQL is more expensive than a cheap VM.

But it buys:

- backups,
- recovery,
- operations,
- scaling.

Positive ROI for solo SaaS.

---

# 130. Pricing ROI Review

Review every quarter:

```text
What plan has highest margin?
Which users retain?
Who buys top-ups?
What feature causes upgrades?
What feature causes churn?
```

Use this to evolve plan versions.

---

# 131. When to Raise Pro Price

Consider higher Pro price if:

- customers consistently quantify high ROI,
- churn remains low,
- usage/support is professional/high,
- margins are compressed,
- competitor alternatives cost more.

Test new customers first.

Do not change existing customers impulsively.

---

# 132. Pricing Floor

Never price below:

```text
Variable COGS
+
Payment Cost
+
Required Contribution
```

unless intentionally subsidizing acquisition.

---

# 133. Discount Floor

Promotions must still have acceptable expected lifetime margin.

A 50% discount can destroy margin for AI-heavy users.

---

# 134. ROI Risk Register

Major ROI risks:

```text
weak paid conversion
high churn
AI cost inflation
overuse by free users
render cost
commercial data licenses
long development timeline
professional trust
regulatory/liability issues
high CAC
underpriced Business customization
```

Each must have mitigation.

---

# 135. Risk — Development Takes Too Long

Impact:

```text
opportunity cost rises
market may move
founder fatigue
```

Mitigation:

```text
strict MVP phase gates
defer secondary modules
launch controlled beta earlier
```

---

# 136. Risk — AI Costs Rise

Mitigation:

- model routing,
- credits,
- top-ups,
- caching,
- future plan versions,
- provider abstraction.

---

# 137. Risk — Low Conversion

Mitigation:

- focus ICP,
- improve onboarding,
- demo customer ROI,
- adjust pricing/package,
- remove free users who never activate from cost assumptions.

---

# 138. Risk — High Churn

Mitigation:

- identify retained use case,
- improve project reuse/history,
- professional workflows,
- Business team collaboration.

Do not solve churn only with discounts.

---

# 139. Risk — Professional Trust

BuildWise outputs affect construction decisions.

Mitigation:

- deterministic QS,
- provenance,
- assumptions,
- confidence,
- professional review warnings,
- versioning.

Trust is directly tied to retention and ROI.

---

# 140. Risk — Commercial License Shock

Mitigation:

- open/interchange formats first,
- negotiate early before roadmap dependency,
- make paid data optional/add-on where appropriate.

---

# 141. Monthly ROI Review Meeting — Founder

Even as one person, perform a formal monthly review:

```text
1. Revenue
2. Paid users
3. Churn
4. COGS
5. AI cost
6. Margin
7. Acquisition
8. Cash runway
9. Highest ROI feature
10. Lowest ROI activity
11. Next month's investment
```

Document decisions.

---

# 142. Quarterly Investment Review

Ask:

```text
Should I invest next quarter in:
Product?
AI?
Sales?
Marketing?
Data?
Infrastructure?
Hiring?
```

Choose based on bottleneck.

Do not default to more engineering.

---

# 143. Reinvestment Decision Tree

```text
Low activation?
→ Product/onboarding

Good activation, low paid conversion?
→ Pricing/value communication

Good conversion, high churn?
→ Core value/retention

Good retention, low traffic?
→ Acquisition

Good growth, operational pain?
→ Team/infrastructure

Strong enterprise interest?
→ Security/integrations
```

---

# 144. Break-Even Hierarchy

BuildWise should track four separate break-even points.

## 1. Infrastructure Break-Even

```text
Revenue covers production technology
```

Approx:

```text
25–30 paid-equivalent accounts
```

under initial assumptions.

## 2. Founder Cash Break-Even

```text
cumulative contribution recovers direct development cash
```

## 3. Founder Economic Break-Even

```text
return recovers cash + founder time value
```

## 4. Company Break-Even

```text
revenue covers all company operating expenses
```

Do not mix them.

---

# 145. Recommended Milestone Targets

## Milestone 1 — Validation

```text
5 active beta users
```

## Milestone 2 — Willingness to Pay

```text
10 paying customers
```

## Milestone 3 — Technical Break-Even

```text
30 paying-equivalent customers
```

## Milestone 4 — Market Signal

```text
100 paid
```

## Milestone 5 — Sustainable Solo Business

```text
250–300 paid
```

## Milestone 6 — Growth Decision

```text
1,000 paid
```

---

# 146. 10-Paid-Customer Gate

Purpose:

```text
prove money changes hands
```

Do not care about profit yet.

Questions:

- Why did they buy?
- Which feature?
- Which plan?
- Would they renew?

---

# 147. 30-Paid Gate

Purpose:

```text
technical break-even
```

Review:

- production cost,
- AI cost,
- support load,
- churn.

---

# 148. 100-Paid Gate

Purpose:

```text
product-market signal
```

Need:

- cohort retention,
- plan mix,
- CAC,
- gross margin.

Decide whether to increase marketing.

---

# 149. 300-Paid Gate

Purpose:

```text
sustainable founder economics
```

Evaluate:

- founder salary,
- first hire,
- annual plans,
- partnerships.

---

# 150. 1,000-Paid Gate

Purpose:

```text
growth company decision
```

Evaluate:

- team,
- funding,
- enterprise,
- international,
- vendor volume agreements.

---

# 151. 5,000-Paid Gate

Purpose:

```text
scale economics
```

Review:

- negotiated Stripe,
- AI provider agreement,
- infrastructure commitments,
- formal FinOps,
- security/compliance.

---

# 152. ROI Plan A — Lean Bootstrap

## Goal

Reach technical break-even with minimal external capital.

## Investment

```text
$5k–$10k direct pre-launch cash
founder development time
```

## Target

```text
30 paid
within 3–6 months of paid launch
```

## Success

```text
> 60% technical gross margin
positive customer feedback
renewal intent
```

## Next

```text
grow to 100 paid using founder-led acquisition
```

---

# 153. ROI Plan B — Sustainable Founder SaaS

## Goal

Produce meaningful founder income.

## Target

```text
250–300 paid
```

Illustrative:

```text
MRR ≈ $12k–$14k
```

## Required

- low churn,
- controlled AI,
- mostly self-service onboarding,
- manageable support.

## Outcome

Potential to fund:

- founder salary,
- product reinvestment,
- modest marketing.

---

# 154. ROI Plan C — Growth SaaS

## Goal

Reach:

```text
1,000–3,000 paid
```

Illustrative base MRR:

```text
$47k–$141k
```

## Investment

Reinvest in:

- acquisition,
- customer success,
- developers,
- data,
- enterprise.

## Gate

Do not enter if churn/CAC are still unknown.

---

# 155. ROI Plan D — Enterprise Expansion

## Goal

Increase ARPA and NRR.

Investment:

- SSO,
- audit,
- integrations,
- support,
- compliance.

Require:

```text
signed enterprise demand
```

before building large custom infrastructure.

---

# 156. Investor-Style Metrics

If seeking investment, prepare:

```text
MRR / ARR
growth rate
gross margin
paid conversion
retention
NRR
CAC
LTV
CAC payback
AI cost/paid account
ARPA
pipeline
market size
```

Avoid presenting only:

```text
registered users
```

---

# 157. Founder ROI vs Equity Funding

Bootstrapping preserves equity but may grow slower.

Funding increases speed but creates dilution.

Compare:

```text
Value of extra growth created by capital
vs
value of ownership diluted
```

There is no universal answer.

BuildWise should gather traction first.

---

# 158. Runway-Based Investment Rule

Never commit a large fixed annual vendor expense if it reduces runway below a safe threshold without clear revenue impact.

Example:

```text
$20k data license
```

should require:

```text
customer contracts
or
strong validated pipeline
```

---

# 159. Monthly Cash Forecast

Maintain:

```text
Opening Cash
+ Collections
- COGS
- Development Tools
- Cloud
- Marketing
- Payroll
- Legal/Admin
= Closing Cash
```

Then:

```text
Runway = Closing Cash / Expected Net Burn
```

---

# 160. Cash Collection Matters

MRR is not always cash received.

Annual invoice/payment failures/enterprise terms affect:

```text
cash flow
```

ROI planning should monitor both:

```text
recognized recurring revenue
and
cash collected
```

---

# 161. Annual Subscription Cash ROI

Annual plans can provide:

```text
12 months cash upfront
```

which can finance development.

But do not spend all annual cash immediately.

You still owe:

- future service,
- AI credits,
- support.

Maintain deferred-service reserve mentally/accountingly.

---

# 162. Gross Margin Target by Maturity

## Beta

```text
40–60% may temporarily occur
```

because fixed costs are spread thinly.

## Early Growth

```text
60–70%
```

## Mature

```text
70–80%+
```

for standard SaaS product tiers.

Enterprise custom work may differ.

---

# 163. ROI Trigger for Cost Optimization

Optimize cost when:

```text
annual saving
>
meaningful engineering cost
```

or gross margin is threatened.

Do not prematurely optimize tiny bills.

---

# 164. ROI Trigger for Reliability Spend

Reliability/security spending can have positive ROI even without direct revenue.

Example:

```text
$200/month redundancy
```

may be justified if one outage could lose:

- customers,
- trust,
- revenue.

Risk-adjusted ROI matters.

---

# 165. Risk-Adjusted ROI

Conceptually:

```text
Risk-Adjusted Return
=
Expected Return
-
Probability × Impact of Failure
```

Use when evaluating:

- backups,
- redundancy,
- security,
- compliance.

---

# 166. Development Decision Example

Option A:

```text
Build advanced terrain engine
4 weeks
```

Option B:

```text
Improve QS traceability
4 weeks
```

If professional users primarily pay for cost/BOQ:

```text
QS traceability
```

likely has higher near-term ROI.

This discipline protects the roadmap.

---

# 167. AI Agent ROI

Claude/Codex/Antigravity subscriptions are worthwhile if they reduce:

- development time,
- mistakes,
- review burden.

Measure:

```text
tasks completed
rework
defects
```

Do not let multiple agents produce conflicting architecture that creates negative ROI.

---

# 168. AI Coding Cost Principle

The largest AI-development cost is not subscription price.

It is:

```text
bad code accepted without review
```

A $20–$100 tool that prevents one week of rework has excellent ROI.

Use architecture documents as guardrails.

---

# 169. ROI of Documentation

The extensive BuildWise architecture docs reduce:

- agent drift,
- inconsistent code,
- future rewrite,
- onboarding time.

Treat documentation as:

```text
technical debt prevention investment
```

not overhead.

---

# 170. ROI of Tests

Tests reduce:

- production incidents,
- billing errors,
- geometry regressions,
- rework.

Highest ROI test areas:

```text
Building Model
QS
Cost
Billing
Tenant isolation
AI ChangeSets
```

---

# 171. ROI of Observability

Sentry/telemetry ROI:

```text
faster diagnosis
lower support time
less churn
lower incident duration
```

Do not remove monitoring to save small monthly fees.

---

# 172. ROI of Backups

Backups have near-zero visible return until disaster.

Potential avoided loss:

```text
entire company/product data
```

Therefore backups are mandatory risk-adjusted investment.

---

# 173. ROI of Security

Security spending protects:

- customer trust,
- contracts,
- legal exposure,
- company value.

It should not be evaluated only as short-term revenue.

---

# 174. ROI Calculation Spreadsheet Fields

The eventual spreadsheet should contain:

```text
Month
MAU
Paid Accounts
Paid Conversion
Home Accounts
Pro Accounts
Business Accounts
Enterprise Revenue
Top-Up Revenue

Subscription Revenue
Total Revenue

Free COGS
Home COGS
Pro COGS
Business COGS
AI
Render
Infrastructure
Payments
Other COGS

Gross Profit
Gross Margin

Marketing
Payroll
Tools
Legal/Admin
Other Opex

Operating Profit
Cash Flow
Closing Cash

CAC
LTV
Payback
Runway
```

---

# 175. Scenario Inputs

Make these editable:

```text
Home price
Pro price
Business price

Plan mix
Conversion
Churn
MAU growth
Free cost
Plan COGS
Platform cost
CAC
Marketing budget
Founder salary
Headcount
```

Never hardcode assumptions in spreadsheet formulas where user cannot change them.

---

# 176. Sensitivity Dashboard

Show effects of:

```text
Conversion ±2%
Churn ±2%
AI cost ±50%
CAC ±50%
ARPA ±20%
MAU growth ±30%
```

This is more useful than one forecast.

---

# 177. Worst-Case Planning

Plan for:

```text
slow growth
4% paid conversion
8% churn
higher AI cost
```

BuildWise should remain financially survivable.

Do not require upside scenario to pay infrastructure bills.

---

# 178. Best-Case Planning

Upside scenario is used for capacity/funding planning.

Do not spend today based on future upside.

---

# 179. Default Founder ROI Decision

Given the current architecture and cost model, the recommended strategy is:

```text
BOOTSTRAP THROUGH PRODUCT VALIDATION
        ↓
LAUNCH PAID BETA
        ↓
REACH 30 PAID
        ↓
MEASURE RETENTION / COGS
        ↓
REACH 100 PAID
        ↓
DECIDE WHETHER TO ACCELERATE
        ↓
REACH 250–300 PAID
        ↓
FOUNDER-SUSTAINABLE BUSINESS
        ↓
1,000 PAID
        ↓
GROWTH / FUNDING / ENTERPRISE DECISION
```

---

# 180. What Success Looks Like

A strong BuildWise business may eventually have:

```text
high professional customer ROI
70%+ gross margin
low AI cost relative to price
3:1+ LTV:CAC
< 5% monthly SMB churn
strong Pro/Business mix
expansion revenue
```

Those metrics matter more than simply:

```text
millions of generated 3D models
```

---

# 181. What Failure Looks Like

Warning pattern:

```text
large free user base
low paid conversion
high AI usage
high support demand
professional users churn
```

This can produce impressive usage and poor ROI.

Correct it before scaling.

---

# 182. Master ROI Checklist

## Before Launch

- [ ] direct cash investment recorded.
- [ ] founder hours recorded.
- [ ] plan pricing defined.
- [ ] plan COGS targets.
- [ ] AI telemetry.
- [ ] technical break-even target.
- [ ] customer ROI hypothesis.
- [ ] 3-month cash reserve.

## At 10 Paid

- [ ] why customers paid.
- [ ] activation.
- [ ] AI cost.
- [ ] renewal intent.

## At 30 Paid

- [ ] technical break-even checked.
- [ ] gross margin.
- [ ] support load.
- [ ] cash payback projection.

## At 100 Paid

- [ ] churn.
- [ ] CAC.
- [ ] LTV.
- [ ] LTV:CAC.
- [ ] cohort retention.
- [ ] plan mix.

## At 300 Paid

- [ ] founder salary viability.
- [ ] hiring decision.
- [ ] annual plan.
- [ ] growth channel.

## At 1,000 Paid

- [ ] vendor negotiations.
- [ ] team plan.
- [ ] funding decision.
- [ ] enterprise roadmap.

---

# 183. Monthly ROI Checklist

- [ ] MRR.
- [ ] new MRR.
- [ ] expansion MRR.
- [ ] churned MRR.
- [ ] ARPA.
- [ ] paid conversion.
- [ ] COGS.
- [ ] gross margin.
- [ ] AI cost.
- [ ] AI cost/paid.
- [ ] Free cost.
- [ ] CAC.
- [ ] CAC payback.
- [ ] churn.
- [ ] LTV.
- [ ] cash burn.
- [ ] runway.
- [ ] founder hours.
- [ ] top ROI feature.
- [ ] lowest ROI expense.

---

# 184. Recommended Repository Location

Store:

```text
docs/finance/
├── BuildWise_Return_On_Investment_Plan.md
├── BuildWise_Development_Production_Costing_Plan.md
├── cost-register.md
├── monthly-unit-economics.md
└── provider-rate-review.md
```

---

# 185. AGENTS.md Instruction

Add:

```text
Before proposing a major paid dependency, large infrastructure change,
new commercial module, new AI-heavy capability, GPU feature,
commercial data license, or enterprise customization, read:

docs/finance/BuildWise_Return_On_Investment_Plan.md

For significant investment proposals, state:
- expected customer/business value,
- development cost,
- recurring cost,
- expected revenue/retention impact,
- payback hypothesis,
- success metric,
- stop/defer condition.
```

---

# 186. Final ROI Model

```text
                      BUILDWISE INVESTMENT
                              │
               ┌──────────────┼──────────────┐
               ▼              ▼              ▼
          CASH COST       FOUNDER TIME      RISK
               │              │              │
               └──────────────┼──────────────┘
                              ▼
                          PRODUCT
                              │
               ┌──────────────┼──────────────┐
               ▼              ▼              ▼
          ACQUISITION     PROFESSIONAL      RETENTION
                             VALUE
               │              │              │
               └──────────────┼──────────────┘
                              ▼
                       PAID CUSTOMERS
                              │
                 ┌────────────┴────────────┐
                 ▼                         ▼
             REVENUE                     COGS
                 │                         │
                 └────────────┬────────────┘
                              ▼
                      GROSS CONTRIBUTION
                              │
               ┌──────────────┼──────────────┐
               ▼              ▼              ▼
          PAYBACK          REINVESTMENT     PROFIT
                              │
                              ▼
                            GROWTH
```

---

# 187. Final ROI Principles

1. **Technical break-even is not company profitability.**
2. **Track founder cash and founder time separately.**
3. **Aim for 25–30 paid-equivalent customers as the first technical break-even milestone.**
4. **Target 70%+ mature gross margin.**
5. **Retention matters more to LTV than small infrastructure savings.**
6. **Pro/Business adoption matters more than raw MAU.**
7. **Free users must have a measurable acquisition purpose.**
8. **AI must remain metered and cost-aware.**
9. **Top-ups convert heavy AI users from margin risk into expansion revenue.**
10. **Rendering should be demand-driven, not a mandatory early expense.**
11. **Build professional trust before scaling paid acquisition.**
12. **Target LTV:CAC of at least 3:1 before aggressive paid growth.**
13. **Target CAC payback under roughly 6–12 months where possible.**
14. **Use customer ROI to justify pricing.**
15. **Do not purchase expensive professional data/SDKs without a revenue case.**
16. **Do not optimize $20/month infrastructure at the cost of weeks of founder time.**
17. **Security, backups and auditability have risk-adjusted ROI.**
18. **Every major phase should have an investment gate.**
19. **Scenario forecasts are hypotheses until replaced by real cohorts.**
20. **The best next investment is the one that removes the current growth bottleneck.**

---

# 188. Final Recommendation

The most appropriate ROI strategy for BuildWise is:

> **Bootstrap to paid validation first.**

Do not design the business around an imagined 100,000-user future before proving that approximately 30 professionals or serious users will pay and remain.

The recommended progression is:

```text
Prototype
    ↓
5 active beta users
    ↓
10 paying users
    ↓
30 paid → technical break-even target
    ↓
100 paid → product-market signal
    ↓
250–300 paid → sustainable founder SaaS
    ↓
1,000 paid → growth-company decision
    ↓
Enterprise / funding / international expansion
```

The financial objective is not simply:

```text
generate revenue
```

It is:

```text
create customer value
that is many times greater than the subscription price
while keeping BuildWise's marginal cost
far below the revenue generated by that value.
```

That is the long-term ROI engine for BuildWise.
