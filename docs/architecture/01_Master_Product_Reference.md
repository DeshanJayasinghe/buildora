# Buildora AI — Master Product, Architecture, UX, AI, QS & Technology Reference

> **Document purpose**
>
> This document is the master implementation reference for **Buildora AI**, an AI-native construction design, BIM, quantity surveying, costing, and construction intelligence platform.
>
> It consolidates the full product direction, functional scope, UX flows, AI architecture, technology decisions, commercial logic, and implementation principles discussed to date.
>
> It is designed to be given directly to **Codex or other implementation agents** as a source of truth.

---

# 1. Product Vision

Buildora AI should not be positioned as only:

> “An AI 3D house designer.”

That market already contains strong products such as Planner 5D, Coohom, RoomSketcher, HomeByMe, Cedreo, and others.

The stronger positioning is:

> **An AI Construction Design + Quantity Surveying + Cost Intelligence Platform**

or:

> **An AI construction copilot that turns an idea or an existing drawing into a buildable, measurable, and costed digital building.**

Recommended product message:

> **Describe it. Design it. Cost it. Build it.**

The core transformation is:

```text
Idea / Prompt / Existing Plan
        ↓
AI Understanding
        ↓
Structured Building Model
        ↓
Editable 2D Plan
        ↓
Editable 3D / BIM View
        ↓
Material Definitions
        ↓
Quantity Takeoff
        ↓
BOQ
        ↓
Cost Estimate
        ↓
Cost Optimisation
        ↓
Construction Schedule
        ↓
Project Documents
        ↓
AI Construction Copilot
```

---

# 2. Core Product Differentiation

The current market is broadly split into two groups.

## 2.1 Design-first products

Examples:

- Planner 5D
- Coohom / AIHom
- RoomSketcher
- HomeByMe
- Floorplanner
- Cedreo

They are generally strong at:

- floor plans,
- 3D visualisation,
- interiors,
- furniture,
- renders,
- walkthroughs,
- AI design generation.

They are generally weaker at:

- professional QS,
- structural material quantities,
- detailed BOQ,
- deterministic construction costing,
- traceable quantity takeoff,
- professional revision-cost analysis,
- construction intelligence.

## 2.2 Construction-first products

Examples:

- Autodesk Construction Cloud / Takeoff / Forma
- Kreo
- Houzz Pro
- Buildxact
- PlanSwift
- CostX
- Bluebeam
- magicplan

They are generally strong at:

- takeoff,
- measurements,
- BOQ,
- estimating,
- supplier pricing,
- revisions,
- procurement,
- construction workflows.

They generally do not provide a simple consumer-style:

```text
"Design me a four-bedroom modern house under £250,000."
        ↓
AI-generated editable building
        ↓
3D walkthrough
        ↓
Full quantity and cost intelligence
```

Buildora AI should connect these two worlds.

---

# 3. Competitive Features to Learn From

Buildora AI should learn from competitors without copying their product architecture blindly.

## Planner 5D

Relevant ideas:

- uploaded floor-plan recognition,
- editable 2D/3D,
- AI interior design,
- furniture/material libraries,
- photorealistic rendering,
- simple homeowner UX.

## Coohom / AIHom

Relevant ideas:

- prompt → floor plan,
- upload plan → editable 2D/3D,
- AI-assisted design changes,
- conversational design interaction,
- strong rendering and furnishing.

## RoomSketcher

Relevant ideas:

- clear plan editing,
- measurement tools,
- 3D walkthrough,
- room and area calculation,
- accessible homeowner workflow.

## Houzz Pro

Relevant ideas:

- plan → 3D,
- AI takeoffs,
- quantities,
- estimates,
- materials + labour,
- schedules,
- client workflows.

## magicplan

Relevant ideas:

- mobile capture,
- measurements,
- estimates,
- material/labour calculations,
- field-friendly workflow.

## Kreo

Relevant ideas:

- AI agent that actually performs QS actions,
- measurement generation,
- BOQ preparation,
- annotations,
- reports,
- project-document understanding.

## PlanSwift

Relevant ideas:

- takeoff assemblies,
- formulas,
- materials,
- labour,
- equipment,
- trade-specific estimating.

## CostX

Relevant ideas:

- professional BIM takeoff,
- live-linked estimates,
- revision comparison,
- auditability,
- cost databases,
- cost impact of drawing changes.

## Bluebeam

Relevant ideas:

- markup,
- measurement,
- revisions,
- collaboration,
- audit trails,
- document-centred workflows.

## Buildxact

Relevant ideas:

- estimates,
- supplier prices,
- RFQs,
- purchase orders,
- committed vs actual cost,
- change orders,
- accounting integrations,
- client portal.

## Autodesk Forma / Takeoff / Build

Relevant ideas:

- site analysis,
- generative alternatives,
- BIM quantity takeoff,
- construction document/model workflows,
- environmental analysis,
- cost/project management.

---

# 4. Primary User Segments

Buildora AI should eventually support:

- Homeowner
- Architect
- Architectural Designer
- Quantity Surveyor
- Estimator
- Contractor / Builder
- Structural Engineer
- MEP Engineer
- Interior Designer
- Developer
- Supplier
- Client / Viewer
- Company Admin
- Platform Admin

---

# 5. Product Modes

## 5.1 Simple Mode

For:

- homeowners,
- early-stage users,
- basic residential design.

Expose:

- project brief,
- AI generation,
- upload plan,
- simple 2D editing,
- 3D walkthrough,
- materials,
- approximate quantities,
- conceptual cost,
- simple reports,
- AI Copilot.

Hide unnecessary professional complexity.

## 5.2 Professional Mode

For:

- architects,
- QS,
- estimators,
- contractors,
- engineers.

Expose:

- BIM,
- IFC,
- advanced takeoff,
- assemblies,
- professional BOQ,
- rate build-ups,
- quantity audit trail,
- revisions,
- tender analysis,
- collaboration,
- approvals,
- exports,
- cost intelligence.

---

# 6. Main Product Modules

Recommended high-level modules:

```text
Dashboard
Projects

DESIGN
├── Project Overview
├── AI Project Brief
├── Upload Plan
├── 2D Plan
├── 3D Model
├── Materials & Finishes
└── Design Versions

CONSTRUCTION
├── Quantities
├── BOQ
├── Cost Estimate
├── Material Breakdown
├── Cost Analysis
├── Schedule
├── Procurement
└── Actual Cost

PROJECT
├── Documents
├── Reports
├── Revisions
├── Team
├── Approvals
└── Activity

AI
└── Buildora AI Copilot

RESOURCES
├── Material Library
├── Supplier Marketplace
├── Professionals
└── Knowledge Base

ADMIN
├── Users
├── Organisations
├── Subscriptions
├── AI Usage
├── Construction Rates
├── Materials
├── Assemblies
└── System Settings
```

---

# 7. Project Creation Methods

A user should eventually have five possible starting points.

## 7.1 Generate with AI

User describes what they want.

Example:

```text
Create a modern two-storey house.
Plot: 15m × 25m
Bedrooms: 4
Bathrooms: 3
Open-plan kitchen/living
Double garage
Office
Balcony
Budget: £250,000
```

AI creates:

- structured project brief,
- room programme,
- design constraints,
- initial concept options,
- estimated area,
- initial cost range,
- Buildora AI Building Model.

## 7.2 Upload Existing Plan

Formats:

- PDF
- JPG
- PNG
- TIFF
- DXF
- DWG later
- IFC / BIM

## 7.3 Draw Manually

Blank 2D canvas.

## 7.4 Import BIM

Initially:

- IFC

Later:

- RVT via integration or commercial SDK,
- additional professional formats.

## 7.5 Site / Scan Capture

Later:

- mobile camera,
- LiDAR,
- AR measurements,
- site scanning.

---

# 8. AI Project Brief Flow

For users selecting **Generate with AI**:

Capture:

- Project Type
- Location
- Plot Size
- Floors
- Bedrooms
- Bathrooms
- Target Area
- Budget
- Design Style
- Construction Preference
- Finish Level
- Additional Requirements

Example:

```text
Project Type
Residential House

Location
Nottingham, UK

Plot Size
15m × 25m

Floors
2

Bedrooms
4

Bathrooms
3

Target Area
210 m²

Budget
£250,000

Design Style
Modern Contemporary

Additional Requirements
Open-plan kitchen and living room,
double garage, office and balcony.
```

Flow:

```text
User Inputs
    ↓
AI Analysis
    ↓
Structured Brief
    ↓
Conflict / Constraint Detection
    ↓
User Review
    ↓
Generate Concept Options
    ↓
Compare
    ↓
Select Concept
    ↓
Create Building Model
```

The AI must create a structured brief before geometry.

---

# 9. Upload Plan Flow

This is the second major onboarding path.

```text
Upload Floor Plan
       ↓
File Type Detection
       ↓
Vector / Raster Analysis
       ↓
AI / CV Processing
       ↓
Walls detected
Rooms detected
Doors detected
Windows detected
Dimensions detected
       ↓
Recognition Results
       ↓
Confidence Review
       ↓
Low-confidence Warnings
       ↓
User Verification / Correction
       ↓
Successful Conversion
       ↓
Open in 2D Editor
```

Dedicated UI states:

- upload,
- processing,
- recognition results,
- low-confidence warnings,
- verification,
- success,
- error.

---

# 10. Plan Recognition Architecture

Use a hybrid pipeline.

```text
                UPLOADED PLAN
                     │
              Detect File Type
                     │
      ┌──────────────┴──────────────┐
      │                             │
 Vector PDF / DXF              Scanned Image
      │                             │
 Parse Vector Data              Vision / CV
      │                             │
      └──────────────┬──────────────┘
                     │
                AI Semantics
                     │
              Building Elements
                     │
                 Confidence
                     │
               Human Review
```

Important rule:

> If vector data exists, use the vector data.

Do not unnecessarily rasterise precise CAD data and ask AI vision to rediscover the geometry.

Detect where feasible:

- scale,
- dimensions,
- external walls,
- internal walls,
- wall thickness,
- rooms,
- room labels,
- doors,
- windows,
- stairs,
- columns,
- beams,
- floor levels,
- openings,
- fixtures,
- grids,
- north arrow,
- section references,
- elevation references,
- notes,
- title block,
- drawing number,
- revision.

---

# 11. Confidence Review

AI recognition must expose confidence.

Example:

```text
External Wall            98%
Bedroom                   99%
Window W03                93%
Wall Thickness            72%
Column C4                 61%
```

User can:

- confirm,
- reject,
- edit,
- merge,
- split,
- redraw,
- reclassify.

This is a professional trust requirement.

---

# 12. The Buildora AI Building Model

This is the most important engineering decision.

> **The Buildora AI Building Model is the single source of truth.**

Do not create separate unrelated states for:

- 2D,
- 3D,
- BIM,
- quantities,
- BOQ,
- costs.

Everything must derive from one parametric building model.

Example element:

```json
{
  "id": "wall_102",
  "type": "wall",
  "levelId": "ground",
  "start": [1200, 4500],
  "end": [6400, 4500],
  "height": 2700,
  "thickness": 200,
  "assemblyId": "EXT-BLOCK-200"
}
```

Recommended canonical geometry unit:

> **millimetres**

UI may display:

- mm,
- cm,
- m,
- ft/in.

---

# 13. Building Model Hierarchy

```text
Project
 └── Site
     └── Building
         ├── Level
         │   ├── Space / Room
         │   ├── Wall
         │   ├── Door
         │   ├── Window
         │   ├── Column
         │   ├── Beam
         │   ├── Slab
         │   ├── Stair
         │   ├── Opening
         │   └── Equipment
         ├── Roof
         ├── Structural
         ├── Electrical
         ├── Plumbing
         └── HVAC
```

Possible element properties:

- geometry,
- type,
- level,
- construction type,
- material,
- assembly,
- finish,
- classification,
- fire rating,
- source drawing,
- source revision,
- cost code,
- quantity metadata,
- IFC class,
- relationships,
- confidence,
- verification status.

---

# 14. Why the Building Model Matters

Example:

```text
Window W-102
```

must refer to the same object in:

- 2D,
- 3D,
- IFC,
- schedules,
- takeoff,
- BOQ,
- cost,
- supplier order.

If W-102 changes:

```text
1.2m × 1.2m
       ↓
1.5m × 1.5m
```

the system should automatically update:

- 2D,
- 3D,
- area,
- material requirement,
- window schedule,
- BOQ,
- cost,
- report output.

---

# 15. 2D Editor

This is a professional workspace.

Recommended layout:

```text
┌─────────────────────────────────────────────────────────────┐
│ Project / Ground Floor       Undo Redo       Save   Share  │
├───────┬────────────────────────────────────────┬────────────┤
│       │                                        │            │
│TOOLS  │                                        │ PROPERTIES │
│       │              2D CANVAS                 │            │
│Wall   │                                        │ Wall type  │
│Door   │                                        │ Thickness  │
│Window │                                        │ Height     │
│Room   │                                        │ Material   │
│Stair  │                                        │ Cost       │
│       │                                        │            │
├───────┴────────────────────────────────────────┴────────────┤
│ 2D      3D      Quantities      BOQ       Cost             │
└─────────────────────────────────────────────────────────────┘
```

Core tools:

- Select
- Wall
- Partition
- Room
- Door
- Window
- Column
- Beam
- Slab
- Stair
- Ramp
- Opening
- Dimension
- Measure
- Text
- Annotation
- Shapes

CAD behaviour:

- pan,
- zoom,
- grid,
- snapping,
- orthogonal mode,
- alignment,
- offset,
- trim,
- extend,
- copy,
- paste,
- mirror,
- rotate,
- group,
- lock,
- hide,
- layer control,
- calibration.

---

# 16. 3D Workspace

Transition:

```text
2D
 ↓
Generate / Sync 3D
 ↓
3D Workspace
```

Core features:

- orbit,
- walkthrough,
- first-person navigation,
- floor isolation,
- model tree,
- layers,
- material editing,
- section cut,
- clipping,
- daylight mode,
- object selection,
- property inspector,
- measurements,
- isometric/top/front/right views,
- hide/show objects,
- isolate object,
- exploded view later.

The selected object should expose:

- ID,
- type,
- dimensions,
- material,
- assembly,
- cost,
- quantity,
- source drawing,
- revision,
- linked BOQ items.

---

# 17. Interior Design

Homeowners expect this.

Features:

- furniture,
- kitchen cabinets,
- worktops,
- beds,
- wardrobes,
- sanitaryware,
- appliances,
- lights,
- curtains,
- décor.

Materials:

- tile,
- timber,
- carpet,
- plaster,
- paint,
- stone,
- brick,
- concrete,
- glass,
- metal.

AI examples:

```text
Make the living room Scandinavian.

Keep the flooring but make the kitchen modern.

Give me three lower-cost interior options.
```

---

# 18. Exterior / Landscape

Later support:

- façade materials,
- roofs,
- gutters,
- downpipes,
- driveways,
- paths,
- decking,
- patios,
- fencing,
- gates,
- lawn,
- trees,
- retaining walls,
- pools,
- outdoor lighting.

---

# 19. Site Intelligence

Advanced roadmap module:

- address/geolocation,
- plot boundary,
- orientation,
- terrain,
- contours,
- slope,
- surrounding buildings,
- road/access,
- sun path,
- daylight,
- shadows,
- solar potential,
- wind,
- noise,
- views,
- earthworks,
- carbon.

---

# 20. Quantity Surveying Engine

QS must be deterministic.

The LLM must not perform authoritative construction quantity arithmetic.

Inputs:

- Building Model,
- element geometry,
- assemblies,
- construction rules,
- waste rules.

Outputs:

- count,
- length,
- perimeter,
- area,
- volume,
- weight,
- material quantities.

Initial domains:

## Groundworks

- excavation,
- trenches,
- disposal,
- backfill,
- cut/fill.

## Foundations

- strip footing,
- pads,
- raft,
- piles,
- concrete,
- reinforcement,
- formwork.

## Concrete

- columns,
- beams,
- slabs,
- stairs,
- concrete volume,
- reinforcement,
- formwork.

## Masonry

- blocks,
- bricks,
- mortar,
- lintels.

## Roofing

- roof area,
- tiles/sheets,
- timber,
- membrane,
- insulation,
- gutters,
- downpipes.

## Finishes

- plaster,
- render,
- paint,
- flooring,
- tile,
- ceilings,
- skirting.

## Openings

- doors,
- frames,
- ironmongery,
- windows,
- glazing.

Later:

- electrical,
- plumbing,
- HVAC,
- fire,
- external works.

---

# 21. Assemblies / Construction Recipes

Assemblies are essential.

Example:

```text
1 m² External Block Wall
```

may require:

- blocks,
- cement,
- sand,
- mortar,
- internal plaster,
- external render,
- primer,
- paint,
- labour,
- scaffolding allowance,
- waste.

Changing the wall area recalculates the assembly.

Allow users to:

- create,
- clone,
- edit,
- version,
- share,
- import,
- export,
- lock company templates.

---

# 22. Waste Rules

Support:

- global waste,
- trade waste,
- material-specific waste,
- project override.

Example:

```text
Tiles        10%
Timber        8%
Blocks        5%
Paint         5%
Roof Tiles    7%
```

---

# 23. BOQ

Structure:

```text
Section
 └── Trade
     └── Element
         └── Item
```

Example:

```text
03 Concrete

03.01 Foundations
03.01.01 Excavation
03.01.02 Blinding
03.01.03 Concrete
03.01.04 Reinforcement
03.01.05 Formwork
```

Columns:

- item number,
- description,
- unit,
- quantity,
- material rate,
- labour rate,
- plant rate,
- subcontract rate,
- waste,
- total rate,
- total amount,
- floor/location,
- source drawing,
- revision.

Exports:

- XLSX
- CSV
- PDF

---

# 24. Cost Engine

Separate:

- materials,
- labour,
- plant/equipment,
- subcontractors,
- transport,
- waste,
- overheads,
- profit,
- tax,
- contingency.

Calculation:

```text
Direct Cost
+
Indirect Cost
+
Overheads
+
Profit
+
Tax
+
Contingency
=
Project Cost
```

Use decimal-safe financial arithmetic.

Do not use raw JavaScript floating-point for authoritative totals.

---

# 25. Cost Estimate Levels

The app must distinguish between levels of certainty.

## Concept Estimate

Based on:

- area,
- building type,
- location,
- broad finish level.

Example:

```text
£230,000–£270,000
Confidence: ±15–20%
```

## Elemental Estimate

Based on:

- model geometry,
- selected systems,
- elements.

## Detailed Estimate

Based on:

- architectural,
- structural,
- MEP,
- specifications,
- detailed assemblies.

## Tender Estimate

Based on:

- supplier rates,
- subcontractor quotes,
- actual tender data.

---

# 26. Regional Pricing

Store:

- material,
- unit,
- currency,
- country,
- region,
- supplier,
- price,
- effective date,
- tax,
- delivery,
- source,
- confidence,
- user override.

Example:

```text
Ready-Mix Concrete

London        £X
Nottingham    £Y
Manchester    £Z
Colombo       LKR X
Delhi         INR X
```

Do not assume one global price.

---

# 27. Supplier Pricing

Later:

```text
Cement
Required: 420 bags

Supplier A   £6.90
Supplier B   £7.10
Supplier C   £6.75
```

Actions:

- Request Quote
- Compare
- Select Supplier
- Create Purchase Order

Potential revenue:

- supplier subscriptions,
- lead fees,
- commissions,
- promoted supplier listings.

---

# 28. Quantity & Cost Auditability

Every quantity must be traceable.

Click:

```text
Internal Plaster
186.4 m²
```

Show:

- contributing walls,
- geometry,
- source drawing,
- model revision,
- formula,
- assembly,
- waste assumption,
- rate source,
- overrides.

This is required for professional trust.

---

# 29. Cost Optimisation / Value Engineering

Example:

```text
Reduce total project cost from £280,000 to £250,000.
Keep 4 bedrooms.
Do not reduce total floor area below 190 m².
```

AI should:

1. query current cost breakdown,
2. identify high-value savings,
3. propose alternatives,
4. estimate cost impact,
5. estimate design impact,
6. show options,
7. require approval,
8. apply approved changes,
9. recalculate all dependent quantities/costs.

Example:

| Change | Saving |
|---|---:|
| Simplify roof | £7,800 |
| Change window spec | £4,100 |
| Alternative tile | £3,600 |
| Wall system | £5,200 |
| Bathroom finishes | £3,100 |
| Kitchen spec | £7,500 |

---

# 30. AI Copilot

AI should be available throughout the product, not only on a separate page.

Display:

> **Buildora AI — Automatic**

Possible professional options:

- Automatic
- Fast
- Deep Analysis

Example questions:

## 3D Editor

```text
Reduce this room by 1m and expand the kitchen.
```

## BOQ

```text
Why is concrete cost so high?
```

## Cost

```text
Reduce overall project cost to £230,000.
```

## Documents

```text
What concrete grade did the engineer specify?
```

## Quantities

```text
How many blocks are needed for the ground floor?
```

## Versions

```text
What changed between Version 3 and Version 4?
```

---

# 31. AI Is an Orchestrator, Not the Calculator

Use:

```text
User
 ↓
AI Gateway
 ↓
Intent / Complexity / Risk
 ↓
Model Selection
 ↓
Tool Calls
 ↓
Geometry / BIM / QS / Cost / Documents
 ↓
Validation
 ↓
Answer
```

The AI should call internal tools.

Example:

```text
get_project()
get_building_elements()
calculate_quantities()
get_local_prices()
calculate_estimate()
compare_versions()
search_project_documents()
```

---

# 32. Automatic Model Routing

Do not send all tasks to the most expensive model.

Internally use model aliases:

```text
FAST_MODEL
BALANCED_MODEL
ADVANCED_MODEL
EXPERT_MODEL
```

Current conceptual mapping discussed:

- Luna
- Terra
- Sol
- Astra

However:

> Do not hardcode business logic to provider marketing names.

Route by:

- task type,
- complexity,
- context size,
- number of tools,
- number of affected objects,
- financial impact,
- design impact,
- document complexity,
- confidence requirement,
- plan/credit limits,
- latency target.

Illustrative:

```text
Simple lookup
→ FAST_MODEL

Normal construction Q&A
→ BALANCED_MODEL

Advanced analysis
→ ADVANCED_MODEL

Major geometry/design/cost optimisation
→ EXPERT_MODEL
```

---

# 33. Model Escalation

Support:

```text
BALANCED_MODEL
      ↓
Low Confidence
      ↓
ADVANCED_MODEL
      ↓
Still Unresolved
      ↓
EXPERT_MODEL
```

Also automatically downgrade for simple follow-up requests.

Track:

- initial model,
- final model,
- escalation reason,
- token usage,
- cost,
- latency,
- confidence,
- tool failures.

---

# 34. AI Gateway

Recommended:

- TypeScript service
- OpenAI Responses API / current tool-capable API

Responsibilities:

- provider/model abstraction,
- routing,
- reasoning effort,
- prompts,
- context construction,
- structured outputs,
- tool permissions,
- usage limits,
- caching,
- retries,
- escalation,
- cost logging,
- audit logs.

---

# 35. AI Geometry Changes

AI must not directly modify production geometry.

Correct flow:

```text
AI Request
   ↓
Structured ChangeSet
   ↓
Draft / Sandbox Model
   ↓
Geometry Validation
   ↓
QS Recalculation
   ↓
Cost Impact
   ↓
Preview
   ↓
User Approval
   ↓
Commit
```

Example:

```json
{
  "action": "move_wall",
  "elementId": "W-102",
  "deltaMm": 1000
}
```

---

# 36. AI Plan Modifications

Example:

```text
Reduce Bedroom 2 by 1m.
Increase the kitchen.
Do not move structural Column C4.
Keep corridor width above minimum.
Keep total cost below £250,000.
```

Flow:

```text
Read Building Model
 ↓
Identify Affected Elements
 ↓
Identify Constraints
 ↓
Generate ChangeSet
 ↓
Geometry Engine Validates
 ↓
QS Recalculates
 ↓
Cost Recalculates
 ↓
Preview
 ↓
User Approval
 ↓
Commit Revision
```

---

# 37. AI Document Intelligence

Project RAG should support:

- architectural plans,
- structural drawings,
- MEP,
- specifications,
- soil report,
- quotations,
- contracts,
- invoices,
- product manuals,
- regulations where licensed/appropriate.

Questions:

```text
What concrete grade is specified?

Which drawing shows the drainage route?

What changed in revision B?

Which supplier quote includes delivery?
```

Answers should cite the project source inside the app.

---

# 38. Structural Concept Module

Later-stage feature.

Potential outputs:

- conceptual foundation,
- column layout,
- beam layout,
- slab system,
- indicative concrete,
- indicative reinforcement.

Critical product rule:

> Structural AI output must be clearly marked as conceptual and requiring qualified engineer review unless a validated professional engineering workflow exists.

---

# 39. MEP

Later modules:

## Electrical

- sockets,
- switches,
- lights,
- boards,
- circuits,
- cable quantities,
- load estimates.

## Plumbing

- hot/cold water,
- drainage,
- sanitary fixtures,
- pipe lengths,
- tanks.

## HVAC

- room volumes,
- heating/cooling demand,
- ducts,
- vents,
- equipment.

## Fire

- alarms,
- extinguishers,
- escape routes.

---

# 40. Construction Schedule

Generate from:

- project scope,
- quantities,
- assemblies,
- dependencies,
- expected productivity.

Example:

```text
Site Preparation
Excavation
Foundations
Ground Floor
Frame
Roof
Walls
MEP First Fix
Plaster
Flooring
Painting
MEP Second Fix
Testing
Handover
```

Features:

- task,
- start/end,
- duration,
- dependencies,
- milestone,
- responsible party,
- progress,
- Gantt,
- critical path later.

---

# 41. Cash Flow

Combine:

- cost,
- schedule,
- procurement.

Outputs:

```text
January    £20k
February   £41k
March      £35k
April      £52k
```

Track:

- planned spend,
- actual spend,
- forecast,
- variance,
- cumulative cash flow.

---

# 42. Procurement

Later:

- RFQ,
- supplier invitation,
- quote comparison,
- purchase orders,
- delivery tracking,
- partial deliveries,
- invoices,
- committed cost.

---

# 43. Tender Comparison

Example:

| Contractor | Quote |
|---|---:|
| A | £245,000 |
| B | £267,000 |
| C | £253,000 |

AI should identify:

- exclusions,
- unusually high lines,
- missing scope,
- inconsistent allowances,
- risk.

---

# 44. Actual Cost Tracking

Track:

```text
Estimated Concrete   £12,400
Committed Concrete   £12,850
Actual Concrete      £13,100
Variance             +£700
```

Dimensions:

- estimated,
- committed,
- actual,
- forecast,
- remaining.

---

# 45. Change Orders

Example:

```text
Client requests larger patio.
```

System calculates:

- additional concrete,
- tile,
- labour,
- time.

Generate:

```text
Change Order
£3,250
```

Require approval.

---

# 46. Progress Tracking

Later:

- planned %,
- actual %,
- milestones,
- photos,
- site diary,
- delays,
- issues,
- progress payment,
- inspection records.

---

# 47. Mobile Companion

Do not make full CAD/BIM mobile editing the initial target.

Mobile should focus on:

- view drawings,
- view 3D,
- AI Copilot,
- site photos,
- notes,
- issues,
- measurements,
- approvals,
- progress,
- site diary,
- deliveries.

Potential native integrations:

- iOS: Swift + ARKit / LiDAR
- Android: Kotlin + ARCore

---

# 48. Collaboration

Support:

- invite,
- roles,
- comments,
- mentions,
- tasks,
- approvals,
- notifications,
- selection sharing,
- cursor presence,
- activity feed.

Use semantic conflict handling for geometry.

Do not blindly CRDT-merge major geometry edits.

---

# 49. Client Portal

Potential layout:

```text
Your Project
 ├── 3D View
 ├── Drawings
 ├── Cost
 ├── Material Selections
 ├── Progress
 ├── Decisions
 └── Documents
```

Client can:

- comment,
- approve design,
- approve materials,
- approve variations,
- see cost,
- see progress.

---

# 50. Reports

Generate:

- Project Summary
- Concept Design Report
- Quantity Takeoff
- BOQ
- Cost Estimate
- Material Schedule
- Labour Report
- Cost Analysis
- Value Engineering
- Revision Comparison
- Tender Comparison
- Schedule
- Cash Flow
- Sustainability / Carbon later

Exports:

- PDF
- XLSX
- CSV

---

# 51. Dashboard

Keep relatively simple.

Show:

- recent projects,
- create project,
- active projects,
- estimated portfolio cost,
- pending AI actions,
- recent documents,
- recent activity,
- quick actions.

Quick actions:

- Generate with AI
- Upload Plan
- Draw Manually
- Import BIM

---

# 52. Application Shell

Recommended navigation:

```text
Dashboard
Projects

DESIGN
├── Overview
├── 2D Plan
├── 3D Model
├── Materials
└── Versions

CONSTRUCTION
├── Quantities
├── BOQ
├── Cost Estimate
├── Material Breakdown
├── Cost Analysis
└── Schedule

PROJECT
├── Documents
├── Reports
├── Team
└── Activity

AI COPILOT

Settings
```

---

# 53. Design System

Develop first.

Include:

- Brand
- Typography
- Colors
- Spacing
- Radius
- Shadows
- Buttons
- Inputs
- Search
- Dropdowns
- Checkboxes
- Radio
- Switches
- Sliders
- Tabs
- Breadcrumbs
- Navigation
- Cards
- Stat cards
- Project cards
- Tables
- BOQ rows
- Quantity rows
- Cost rows
- Badges
- Confidence states
- Dialogs
- Drawers
- Tooltips
- AI prompt box
- AI messages
- AI action cards
- AI suggestions
- Approval cards
- CAD toolbar buttons
- Layer rows
- Object tree
- Measurement tags
- Dimension labels
- Notifications
- Loading
- Processing
- Empty state
- Error state

Use one reusable component system.

---

# 54. UI Development Order

Recommended:

```text
1. Design System
2. Application Shell
3. Dashboard
4. Create Project
5. AI Project Brief
6. Upload Plan Flow
7. 2D Editor
8. 3D Workspace
9. Quantities
10. BOQ
11. Cost Estimate
12. Material Breakdown
13. Cost Analysis
14. Reports
15. AI Copilot Everywhere
16. Version Comparison
17. Documents
18. Collaboration
```

Do not generate the entire UI in one AI prompt.

Build module by module.

---

# 55. Technology Stack — Final Recommendation

## Web

- React
- Next.js
- TypeScript

## Design System

- Tailwind CSS
- Radix UI
- Storybook

## 2D

- PixiJS
- custom CAD/editor domain engine

## Client Geometry

- TypeScript
- Rust → WebAssembly for performance-critical operations

## 3D

- Three.js, used directly (not React Three Fiber)
- **WebGL2 baseline — single render path** (ADR-017)
- WebGPU deferred to post-beta progressive enhancement, and only if profiling
  shows a ceiling that WebGPU relieves

## Browser BIM

- That Open ecosystem
- web-ifc
- Fragments

## BIM Server

- Python
- IfcOpenShell

## Advanced Geometry

- OpenCascade

## DWG/DXF Professional Support

- Open Design Alliance SDK

## Backend

- NestJS
- TypeScript

## AI Gateway

- TypeScript
- OpenAI Responses API

## AI / Computer Vision

- Python
- FastAPI
- OpenCV
- PyTorch
- ONNX Runtime

## QS Engine

- deterministic TypeScript domain package

## Cost Engine

- deterministic TypeScript domain package
- decimal-safe arithmetic

## Database

- PostgreSQL

## Spatial

- PostGIS

## Vector / RAG

- pgvector

## Cache

- Redis

## Workflows

- Temporal

## Realtime

- WebSockets
- Yjs / Liveblocks selectively

## File Storage

- Cloudflare R2 / AWS S3

## Rendering

- client GPU for interactive 3D
- Blender headless workers for photoreal rendering

## Infrastructure

- Docker
- AWS or equivalent managed cloud
- Cloudflare
- OpenTelemetry
- Sentry

---

# 56. Why TypeScript-first

Buildora AI's browser layer already needs:

- React,
- Three.js,
- PixiJS,
- BIM web libraries,
- WebSockets,
- AI Gateway,
- shared schemas.

Using TypeScript in the core backend makes it easier to share:

- Building Model types,
- commands,
- events,
- units,
- AI tool schemas,
- API contracts,
- permissions,
- validation.

---

# 57. Why Not Laravel Everywhere

Laravel is capable and scalable.

However Buildora AI is not a normal CRUD product.

It needs:

- browser CAD,
- Three.js,
- BIM,
- WebSockets,
- shared TypeScript contracts,
- AI tools,
- native geometry,
- Python CV/BIM.

Laravel may still be used for a specific business/admin service if needed, but it is not the preferred core architecture.

---

# 58. Why Not Go Everywhere

Go is excellent for:

- concurrency,
- networking,
- low-memory services,
- high-throughput APIs.

But Buildora AI does not benefit from forcing all domains into Go.

Potential future Go services:

- high-throughput WebSocket gateway,
- binary model streaming,
- telemetry ingestion,
- proven performance bottleneck.

Use only when needed.

---

# 59. Why Python

Python is a first-class Buildora AI technology for:

- AI,
- computer vision,
- ML,
- IFC,
- document processing,
- image understanding,
- model training,
- scientific libraries.

Use FastAPI around Python services.

---

# 60. Why Rust / WASM

Use only for performance-critical geometry.

Candidate operations:

- line intersections,
- polygon clipping,
- room detection,
- offsets,
- triangulation,
- snapping,
- collision,
- spatial indexing,
- geometry validation.

Do not build the whole backend in Rust.

---

# 61. Three.js vs Server Rendering

Interactive 3D:

```text
Server
 ↓
Building Model / Optimised 3D
 ↓
Browser
 ↓
Three.js
 ↓
User GPU
```

This keeps server cost low.

Server GPU is for:

- photoreal rendering,
- complex conversion,
- optional heavy processing.

---

# 62. IFC Strategy

IFC is:

- import,
- export,
- interoperability.

IFC is not:

- the live editing database,
- the single application state.

Workflow:

```text
IFC Import
  ↓
Parse
  ↓
Map to Buildora AI Model
  ↓
Edit / Calculate
  ↓
Export IFC
```

---

# 63. 3D Format Strategy

## Authoritative

- Buildora AI parametric model

## BIM Interchange

- IFC

## Browser / Web

- Fragments
- glTF / GLB

## Optional

- OBJ
- FBX

Do not use triangle meshes as the authoritative QS model.

---

# 64. Database Strategy

Use PostgreSQL.

Possible entities:

```text
users
organisations
organisation_members
projects
sites
buildings
levels
building_elements
element_properties
materials
assemblies
assembly_items
quantities
boq_sections
boq_items
cost_rates
cost_estimates
documents
document_revisions
model_versions
commands
ai_actions
audit_events
subscriptions
usage_records
```

Store parametric data and metadata.

Store large binary assets in object storage.

---

# 65. Versioning / Commands

Every important model edit should be a command.

Example:

```json
{
  "command": "MOVE_WALL",
  "elementId": "W-102",
  "from": [4000, 5000],
  "to": [5000, 5000],
  "baseVersion": 42
}
```

Benefits:

- undo,
- redo,
- AI preview,
- audit,
- collaboration,
- rollback,
- revision comparison.

---

# 66. Workflow Orchestration

Use Temporal for long-running flows.

Example:

```text
Upload
 ↓
Extract
 ↓
Recognise
 ↓
Interpret
 ↓
Wait for User Verification
 ↓
Build Model
 ↓
Generate 3D
 ↓
Calculate Quantities
 ↓
Calculate Cost
 ↓
Generate Preview
```

Failure should be resumable.

---

# 67. Realtime Collaboration Architecture

Use:

- WebSockets
- Yjs / Liveblocks for presence and text-like collaboration

Good use cases:

- comments,
- annotations,
- text,
- cursor,
- selection,
- presence.

Geometry:

- server-authoritative commands,
- version checks,
- optimistic locking,
- semantic conflict resolution.

---

# 68. Performance Principles

Run locally when possible:

- camera,
- rendering,
- selection,
- snapping,
- transforms,
- visibility,
- layer filtering,
- lightweight geometry.

Run server-side where authority/heavy processing matters:

- persistence,
- permissions,
- AI,
- BIM conversion,
- plan recognition,
- heavy geometry,
- photoreal rendering,
- QS/cost workflows.

---

# 69. Repository Structure

> **Authoritative version:** `docs/architecture/04_Monorepo_and_Module_Architecture.md`
> (ADR-011, ADR-012). The summary below must match it.

```text
buildora-ai/
│
├── apps/
│   ├── web/              MVP
│   ├── api/              MVP
│   ├── ai-worker/        gated (~Sprint 31)
│   ├── bim-worker/       gated (~Sprint 33)
│   └── render-worker/    NOT IMPLEMENTED — post-MVP
│
├── packages/
│   ├── design-system/    MVP
│   ├── api-contracts/    MVP
│   ├── units/            MVP
│   ├── building-model/   MVP
│   ├── model-session/    MVP — renderer-neutral working model (ADR-008)
│   ├── cad-2d/           MVP
│   ├── engine-3d/        MVP
│   ├── qs-engine/        MVP
│   ├── cost-engine/      MVP
│   └── ai-tools/         NOT IMPLEMENTED — gated (~Sprint 27)
│
├── native/
│   └── geometry-wasm/    NOT IMPLEMENTED — profiling-gated
│
└── infrastructure/
```

**`packages/domain` does not exist and must not be created** (ADR-011).
`packages/core` does not exist; it may be introduced later only under the
conditions in `04_Monorepo_and_Module_Architecture.md` §5.1.

Tooling:

- pnpm
- Turborepo

---

# 69.1 Capability Documents Added After This Reference

Two capabilities were specified after this document was written and are
authoritative in their own files:

| Capability | Document | ADRs |
|---|---|---|
| 3D walkthrough, material/finish editing, FF&E, AI interior layer | `11_3D_Walkthrough_Interior_and_FFE.md` | ADR-019, ADR-020 |
| Customer segments, two commercial lifecycles, archive mode | `12_User_Segments_and_Commercial_Model.md` | ADR-021 |

Where this reference and those documents differ, **those documents win** — they
are newer and ADR-backed.

---

# 70. API Strategy

Use:

- REST / OpenAPI for core APIs,
- WebSockets for realtime,
- gRPC / Protobuf only for internal high-performance/native services where justified.

Avoid making GraphQL mandatory.

---

# 71. Security

Support:

- RBAC,
- project permissions,
- MFA,
- encryption in transit,
- secure file access,
- signed URLs,
- audit logs,
- model versioning,
- backups,
- deletion/recovery,
- export logs,
- AI action logs,
- usage logs.

---

# 72. AI Audit Log

Each AI action should record:

- user,
- timestamp,
- model alias,
- provider/model ID,
- task type,
- context references,
- documents,
- tools invoked,
- proposed changes,
- previous values,
- new values,
- approval state,
- token usage,
- provider cost,
- latency,
- confidence,
- errors.

---

# 73. Pricing Strategy

Recommended initial plans:

| Plan | Indicative Price |
|---|---:|
| Free | $0 |
| Home | ~$14.99/mo |
| Pro | ~$49/mo |
| Business | ~$149/mo |
| Enterprise | Custom |

Also consider:

- pay-per-project reports,
- AI credit packs,
- render credits,
- supplier marketplace revenue,
- lead fees,
- enterprise licensing.

---

# 74. $49 Pro Plan

Recommended:

- unlimited projects subject to fair use,
- unlimited manual 2D editing,
- unlimited manual 3D editing,
- unlimited 2D ↔ 3D sync,
- unlimited deterministic quantity recalculation,
- unlimited BOQ recalculation,
- unlimited cost recalculation,
- 30 AI Design Credits,
- 20 Render Credits,
- AI Copilot,
- professional reports,
- IFC/export features.

---

# 75. AI Design Credits

Illustrative:

| Operation | Credits |
|---|---:|
| Standard AI house concept | 2 |
| Advanced AI concept | 3 |
| Expert design study | 5 |
| Major AI design modification | 1 |
| Manual 2D → 3D sync | 0 |
| Manual 3D edit | 0 |
| BOQ recalc | 0 |
| Cost recalc | 0 |

Equivalent rough usage:

- ~15 standard concepts,
- ~10 advanced concepts,
- ~6 expert concepts,
- or a mix.

Marketing wording:

> **Approximately 10–15 AI design concepts per month, depending on complexity.**

---

# 76. Render Credits

Keep separate from AI design credits.

Suggested Pro allowance:

> 20 Render Credits / month

Illustrative:

| Render | Credits |
|---|---:|
| Real-time 3D | 0 |
| Standard preview | 0 / fair use |
| HD render | 1 |
| 4K render | 2 |
| 360 panorama | 3 |
| Video walkthrough | 5–10 |

---

# 77. Gross Margin Target

Aim for:

> **~70%+ gross margin**

Illustrative $49 monthly target:

```text
Revenue                    $49.00

Infrastructure             ~$2.00
Normal AI                  ~$2.50
Design AI                  ~$6.00
Rendering                  ~$1.50
Payments                   ~$1.50
Other Variable             ~$0.50
────────────────────────────────
Target Variable Cost       ~$14.00

Gross Contribution         ~$35.00
Gross Margin               ~71%
```

These are planning targets.

Do not hardcode provider prices.

---

# 78. AI Cost Control

Do not regenerate the whole building for local edits.

Bad:

```text
Move one wall
 ↓
Send entire project to expert model
 ↓
Regenerate everything
```

Correct:

```text
User Request
 ↓
AI identifies affected elements
 ↓
ChangeSet
 ↓
Geometry engine
 ↓
Regenerate affected geometry
 ↓
QS delta
 ↓
Cost delta
```

---

# 79. Rendering Cost Control

Normal 3D viewing should be almost entirely client-side.

Photoreal rendering is expensive and should use:

- render credits,
- queued workers,
- GPU pool,
- stored output.

---

# 80. Prompt / Context Caching

Stable contexts:

- system prompts,
- tool definitions,
- building schema,
- construction classification,
- company policies,
- static reference material.

Use provider caching where available.

Avoid sending the entire project state on every request.

Retrieve only relevant:

- elements,
- quantities,
- documents,
- versions,
- costs.

---

# 81. Additional Revenue Models

## Pay-per-project

Possible products:

- AI Concept Pack
- Construction Estimate
- Detailed BOQ
- Professional Export Pack

## Supplier Marketplace

Revenue:

- commission,
- lead fee,
- supplier subscription.

## Professional Leads

Connect users to:

- architect,
- QS,
- builder,
- structural engineer,
- electrician,
- plumber.

## Enterprise

Possible:

- private workspace,
- SSO,
- custom rates,
- company assemblies,
- API,
- enterprise BIM workflows.

---

# 82. Admin AI Operations Dashboard

Show:

```text
AI Requests Today
FAST_MODEL %
BALANCED_MODEL %
ADVANCED_MODEL %
EXPERT_MODEL %

Average AI Cost / User
Average AI Cost / Project
Escalation Rate
Failure Rate
Cache Hit Rate
Average Latency
```

Allow configuration:

- automatic escalation,
- automatic downgrade,
- cost-aware routing,
- confidence thresholds,
- usage limits,
- model aliases.

---

# 83. Material Library

Material entity may include:

```text
Name
Manufacturer
SKU
Category
Dimensions
Unit
Material Type
Unit Price
Supplier
Region
Lead Time
Carbon
Warranty
Datasheet
3D Asset
```

---

# 84. AI Material Alternatives

Example:

```text
Current Window        £670
Alternative A         £510
Alternative B         £475
Alternative C         £430
```

Compare:

- cost,
- appearance,
- energy,
- warranty,
- lead time,
- availability.

---

# 85. Sustainability

Later:

- embodied carbon,
- operational energy,
- material carbon,
- solar potential,
- daylight,
- water,
- waste,
- material comparison.

---

# 86. Compliance Assistant

Later regional feature.

Examples:

```text
Check this design for potential UK accessibility issues.

What areas need professional review?
```

Must:

- cite source,
- show uncertainty,
- avoid representing AI as official approval.

---

# 87. Non-Negotiable Engineering Rules

1. The Buildora AI Building Model is the source of truth.
2. 2D and 3D are views of the same model.
3. IFC is interoperability, not the live model database.
4. LLMs do not perform authoritative QS arithmetic.
5. LLMs do not commit unvalidated geometry.
6. QS calculations must be deterministic.
7. Cost calculations must be decimal-safe.
8. AI changes use structured ChangeSets.
9. High-impact AI changes require preview/approval.
10. Interactive 3D uses the client's GPU.
11. Photoreal rendering is queued and credit-controlled.
12. Model routing is automatic and cost-aware.
13. Model names/prices remain configurable.
14. Construction geometry uses one canonical internal unit.
15. Do not add microservices without a real reason.
16. Do not add languages without a real reason.
17. Do not start with Kubernetes by default.
18. Versioning/auditability must exist from early development.
19. Vector drawings must preserve vector precision where possible.
20. Recognition confidence is first-class.
21. Every professional quantity/cost must be traceable.
22. AI should use tools, not invent project facts.
23. Plan complexity must influence confidence and estimate type.
24. Concept estimates must not be presented as detailed BOQs.
25. Structural/MEP advice must respect professional review requirements.

---

# 88. Recommended MVP

First commercial MVP:

1. Authentication / users
2. Projects
3. Create Project onboarding
4. AI Project Brief
5. Upload PDF/image plan
6. Plan recognition
7. Confidence review
8. 2D editor
9. Buildora AI Building Model
10. Automatic 3D generation
11. Materials
12. Assemblies
13. Quantity Takeoff
14. BOQ
15. Concept / Elemental Cost
16. AI Copilot
17. Design change → quantity/cost recalculation
18. Versions
19. PDF/XLSX reports

Do not begin with:

- full structural engineering,
- full MEP,
- marketplace,
- procurement,
- advanced site analytics,
- full contractor ERP,
- full mobile CAD.

---

# 89. Suggested Implementation Phases

## Phase 1 — Foundation

- monorepo,
- Next.js,
- NestJS,
- PostgreSQL,
- Redis,
- object storage,
- auth,
- design system,
- application shell.

## Phase 2 — Projects & Building Model

- projects,
- sites,
- levels,
- element schemas,
- versioning,
- commands,
- units.

## Phase 3 — 2D Editor

- PixiJS,
- selection,
- pan/zoom,
- walls,
- rooms,
- doors/windows,
- snapping,
- dimensions,
- properties.

## Phase 4 — 3D

- Three.js,
- wall/slab meshes,
- doors/windows,
- floors,
- camera modes,
- materials,
- 2D ↔ 3D sync.

## Phase 5 — Upload Recognition

- PDF/image,
- vector parsing,
- CV,
- confidence,
- verification.

## Phase 6 — AI Project Brief

- AI Gateway,
- structured brief,
- concept generation,
- model routing.

## Phase 7 — Materials & Assemblies

- material catalog,
- assembly definitions,
- assignment.

## Phase 8 — QS

- deterministic takeoff,
- calculation trace,
- quantities UI.

## Phase 9 — BOQ & Cost

- BOQ,
- rates,
- cost levels,
- estimate reports.

## Phase 10 — AI Copilot

- Q&A,
- tools,
- cost questions,
- document RAG,
- controlled ChangeSets.

## Phase 11 — Revisions & Reports

- model versions,
- comparisons,
- exports,
- reports.

## Phase 12 — Hardening

- performance,
- security,
- monitoring,
- billing,
- usage,
- credit enforcement.

---

# 90. Definition of Done

A core feature is complete only when:

- UI uses shared design system,
- domain logic is separated from presentation,
- loading state exists,
- empty state exists,
- error state exists,
- permissions checked,
- validation added,
- audit event emitted where relevant,
- unit/integration tests added,
- 2D/3D sync tested where relevant,
- quantity/cost invalidation tested,
- AI usage logged where relevant,
- no duplicate source of truth introduced,
- documentation updated.

---

# 91. Codex Implementation Rules

When Codex receives this document:

1. Read the relevant section before implementing.
2. Identify the domain package responsible.
3. Preserve the Building Model source-of-truth rule.
4. Avoid business logic inside React components.
5. Avoid authoritative arithmetic inside AI prompts.
6. Reuse shared schemas.
7. Use explicit unit and money types.
8. Validate AI structured outputs.
9. Validate geometry mutations.
10. Add tests.
11. Record audit events.
12. Keep provider logic behind interfaces.
13. Keep model IDs and pricing configurable.
14. Keep credit rules configurable.
15. Explain before changing a deliberate architecture decision.

---

# 92. Suggested Codex Initial Prompt

Use this when starting implementation work:

```text
Read 01_Master_Product_Reference.md as the current product and architecture source of truth.

Before implementing:
1. Identify the requested module.
2. Identify the relevant architecture rules from the document.
3. Preserve the Buildora AI Building Model as the single source of truth.
4. Do not create duplicate 2D, 3D, BIM, QS, BOQ, or cost state.
5. Reuse the shared design system and domain packages.
6. Keep AI actions structured, validated, and auditable.
7. Keep calculations deterministic.
8. Keep provider/model/credit settings configurable.
9. Work incrementally and production-first.
10. If a requested implementation conflicts with this reference, explain the conflict before changing the design.
```

---

# 93. Final Product Summary

Buildora AI should ultimately behave like this:

```text
                  BUILDORA

                    USER
                     │
          ┌──────────┴──────────┐
          │                     │
       Prompt                Upload
          │                     │
          └──────────┬──────────┘
                     │
              AI Understanding
                     │
              Building Model
                     │
      ┌──────────────┼───────────────┐
      │              │               │
     2D             3D              BIM
      │              │               │
      └──────────────┼───────────────┘
                     │
                Quantities
                     │
                    BOQ
                     │
                   Cost
                     │
              Optimisation
                     │
                 Schedule
                     │
                Documents
                     │
              Buildora AI
                     │
     Design • Cost • Explain • Modify
```

The core long-term differentiator is:

> **Design change → geometry change → quantity change → BOQ change → cost change → schedule change, automatically and traceably.**

That is the architectural and product foundation every future Buildora AI implementation should preserve.
