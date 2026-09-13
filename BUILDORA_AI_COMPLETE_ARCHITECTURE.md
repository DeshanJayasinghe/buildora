# Buildora AI — Complete System Architecture

**A single, self-contained description of the entire system.**

> **Version:** 1.0 — Architecture Frozen
> **Date:** 12 September 2026
> **Status:** Accepted. Implementation has not yet started.
> **Audience:** Anyone who needs to understand Buildora AI end to end — the founder, a future engineer, a contractor, an investor's technical advisor, or an AI coding agent.

**This document is complete on its own.** It contains every architectural decision, diagram, rule and rationale. You do not need to open any other file to understand the system.

---

## Table of Contents

**Part I — What We Are Building**
1. [The Product in One Page](#1-the-product-in-one-page)
2. [Why This Product Exists](#2-why-this-product-exists)
3. [Who Uses It](#3-who-uses-it)
4. [The One Idea That Drives Everything](#4-the-one-idea-that-drives-everything)

**Part II — How The System Is Shaped**
5. [System Overview](#5-system-overview)
6. [The Technology Stack](#6-the-technology-stack)
7. [Repository and Code Organisation](#7-repository-and-code-organisation)
8. [How The Pieces Depend On Each Other](#8-how-the-pieces-depend-on-each-other)

**Part III — The Heart of the System**
9. [The Canonical Building Model](#9-the-canonical-building-model)
10. [Measurements, Units and Heights](#10-measurements-units-and-heights)
11. [Every Building Element, Defined](#11-every-building-element-defined)
12. [Rooms — How They Are Calculated](#12-rooms--how-they-are-calculated)
13. [Changes, Versions and Undo](#13-changes-versions-and-undo)

**Part IV — What The User Sees**
14. [The Editing Experience](#14-the-editing-experience)
15. [The 2D Drawing Engine](#15-the-2d-drawing-engine)
16. [The 3D Engine](#16-the-3d-engine)
17. [How 2D and 3D Stay In Sync](#17-how-2d-and-3d-stay-in-sync)

**Part V — Turning Design Into Money**
18. [Materials and Assemblies](#18-materials-and-assemblies)
19. [The Quantity Surveying Engine](#19-the-quantity-surveying-engine)
20. [BOQ and Cost](#20-boq-and-cost)
21. [Why Every Number Can Be Proven](#21-why-every-number-can-be-proven)

**Part VI — Intelligence**
22. [The AI Architecture](#22-the-ai-architecture)
23. [How AI Changes A Building Safely](#23-how-ai-changes-a-building-safely)
24. [Reading Existing Drawings](#24-reading-existing-drawings)
25. [BIM and IFC](#25-bim-and-ifc)
26. [Documents and Project Q&A](#26-documents-and-project-qa)

**Part VII — Running A Business On It**
27. [Accounts, Organisations and Permissions](#27-accounts-organisations-and-permissions)
28. [The Database](#28-the-database)
29. [Billing, Plans and Credits](#29-billing-plans-and-credits)
30. [Security](#30-security)

**Part VIII — Operating It**
31. [Infrastructure and Deployment](#31-infrastructure-and-deployment)
32. [What Gets Built When](#32-what-gets-built-when)
33. [Testing](#33-testing)
34. [Monitoring](#34-monitoring)
35. [When Things Break](#35-when-things-break)
36. [Growing From 10 to 100,000 Users](#36-growing-from-10-to-100000-users)

**Part IX — The Decisions**
37. [All 18 Architecture Decisions Explained](#37-all-18-architecture-decisions-explained)
38. [What We Deliberately Are Not Building](#38-what-we-deliberately-are-not-building)
39. [The Rules That Must Never Be Broken](#39-the-rules-that-must-never-be-broken)
40. [What Is Still Undecided](#40-what-is-still-undecided)
41. [Glossary](#41-glossary)

---
---

# PART I — WHAT WE ARE BUILDING

---

# 1. The Product in One Page

**Buildora AI is an AI Building Design, BIM, Quantity Surveying, Costing and Construction Intelligence Platform.**

It takes an idea, a written description, or an existing drawing, and turns it into a building you can edit, view in 3D, measure, and price — where every price can be traced back to the thing that caused it.

```
     Idea, prompt, or existing drawing
                    │
                    ▼
     ┌──────────────────────────────┐
     │  CANONICAL BUILDING MODEL    │  ← the single source of truth
     └──────────────────────────────┘
                    │
        ┌───────────┼───────────┐
        ▼           ▼           ▼
     2D plan    3D model    IFC/BIM export
        │           │           │
        └───────────┼───────────┘
                    ▼
              Quantities          "how much material?"
                    ▼
                  BOQ             "itemised bill of quantities"
                    ▼
                  Cost            "what does it cost to build?"
                    ▼
             Optimisation         "how do we make it cheaper?"
                    ▼
       Construction Intelligence  "scheduling, procurement, tracking"
```

**The promise in five words:** *Describe it. Design it. Cost it.*

---

# 2. Why This Product Exists

The market today splits into two camps, and neither does the whole job.

### Camp 1 — Design-first tools
*Planner 5D, Coohom, RoomSketcher, HomeByMe, Cedreo, Floorplanner*

**Good at:** floor plans, 3D visualisation, interiors, furniture, beautiful renders, AI design generation, easy consumer experience.

**Weak at:** professional quantity surveying, structural material quantities, detailed bills of quantities, deterministic construction costing, traceable takeoff, revision cost analysis.

They make something that *looks* like a building. They cannot tell you what it costs to build.

### Camp 2 — Construction-first tools
*Autodesk Takeoff/Forma, Kreo, Buildxact, PlanSwift, CostX, Bluebeam, Houzz Pro, magicplan*

**Good at:** takeoff, measurement, BOQ, estimating, supplier pricing, revisions, procurement, construction workflow.

**Weak at:** they generally cannot do this —

> *"Design me a four-bedroom house under £250,000"* → an editable building → a 3D walkthrough → full cost intelligence

They can price a building someone else designed. They cannot design one.

### Buildora AI connects the two

We design the building **and** cost it, from one model, with an audit trail.

**The moat is not the AI.** Anyone can call a language model. The moat is that **every cost figure is reproducible and traceable** — you can point at a number in an estimate and follow it back through the bill of quantities, the quantity calculation, the measurement rules, the material rates, and the exact wall it came from. Professional quantity surveyors need that. Design tools cannot provide it. That is the defensible part.

---

# 3. Who Uses It

The platform eventually serves a broad range of people, but they fall into two working modes.

### Simple Mode — for homeowners and early-stage users

Project brief · AI generation · upload a plan · simple 2D editing · 3D walkthrough · materials · approximate quantities · conceptual cost · simple reports · AI assistant.

Professional complexity is hidden.

### Professional Mode — for the trade

Architects · architectural designers · quantity surveyors · estimators · contractors and builders · structural engineers · MEP engineers · interior designers · property developers.

Full BIM · IFC import/export · advanced takeoff · assemblies · professional BOQ · rate build-ups · quantity audit trail · revisions · tender analysis · collaboration · approvals · exports · cost intelligence.

Also served: suppliers, clients and viewers (read-only), company admins, and platform admins.

---

# 4. The One Idea That Drives Everything

If you remember nothing else from this document, remember this.

> ## There is ONE model of the building. Everything else is a view of it.

The 2D plan is a view. The 3D scene is a view. The IFC export is a view. The quantities are calculated from it. The bill of quantities comes from the quantities. The cost comes from the bill of quantities. The AI reads it and proposes changes to it.

**None of those is allowed to become a second version of the truth.**

### Why this matters so much

Imagine window W-102 is 1.2m × 1.2m, and the user changes it to 1.5m × 1.5m.

In a correctly built system, **one** change happens — to the model. Then automatically: the 2D plan redraws, the 3D model rebuilds, the glazing area updates, the window schedule updates, the lintel requirement changes, the wall's net area changes, the BOQ line changes, the cost changes, and the report reflects all of it.

In a badly built system, the 2D drawing and the 3D model each hold their own copy of that window. They drift apart. Within months, nobody trusts the quantities — and quantities are the entire product.

This is the single most common way AEC software fails. We avoid it structurally, not by discipline.

### What this forbids

- The 3D scene is **not** the database.
- The 2D canvas is **not** the database.
- IFC is **not** our internal format.
- Rendered triangles are **not** the building.
- The browser's working copy is **not** a second authority.
- The AI **cannot** write to the building directly.

---
---

# PART II — HOW THE SYSTEM IS SHAPED

---

# 5. System Overview

Here is the whole system on one diagram.

```
┌───────────────────────────────────────────────────────────────┐
│                          BROWSER                              │
│                                                               │
│   React / Next.js  — menus, panels, forms, navigation         │
│                                                               │
│   ┌─────────────────────────────────────────────┐             │
│   │          THE WORKING MODEL                  │             │
│   │  (an in-memory copy of the building)        │             │
│   │                                             │             │
│   │  • the building as currently displayed      │             │
│   │  • which version it is based on             │             │
│   │  • changes not yet saved to the server      │             │
│   └──────────────┬──────────────┬───────────────┘             │
│                  │              │                             │
│         ┌────────▼───┐    ┌─────▼────────┐                    │
│         │ 2D DRAWING │    │  3D VIEWER   │                    │
│         │  (PixiJS)  │    │ (Three.js)   │                    │
│         └────────────┘    └──────────────┘                    │
│         Both read from the SAME working model.                │
└──────────────────────────┬────────────────────────────────────┘
                           │  internet
                           ▼
              ┌─────────────────────────┐
              │       CLOUDFLARE        │
              │ security, caching, TLS  │
              └────────────┬────────────┘
                           ▼
              ┌─────────────────────────┐
              │   WEB SERVER (Next.js)  │
              │  pages, login, layout   │
              └────────────┬────────────┘
                           ▼
┌──────────────────────────────────────────────────────────────┐
│                   THE API  (NestJS)                          │
│                                                              │
│  One application, organised into separate internal modules:  │
│                                                              │
│  ┌──────────┐ ┌─────────┐ ┌──────────┐ ┌────┐ ┌──────────┐   │
│  │ accounts │ │ THE     │ │ quantities│ │ AI │ │ billing  │   │
│  │ projects │ │ MODEL   │ │ BOQ, cost │ │    │ │ credits  │   │
│  └──────────┘ └─────────┘ └──────────┘ └────┘ └──────────┘   │
│                                                              │
│  Every change to a building goes through THE MODEL module.   │
│  There is no other way in.                                   │
└────┬──────────────┬──────────────┬──────────────┬────────────┘
     ▼              ▼              ▼              ▼
┌──────────┐  ┌──────────┐  ┌───────────┐  ┌────────────┐
│POSTGRESQL│  │  FILE    │  │ WORKFLOW  │  │  OUTSIDE   │
│          │  │ STORAGE  │  │  ENGINE   │  │  SERVICES  │
│ THE      │  │          │  │           │  │            │
│ TRUTH    │  │ uploads  │  │ long jobs │  │ OpenAI     │
│          │  │ exports  │  │ (later)   │  │ Stripe     │
│ buildings│  │ reports  │  └─────┬─────┘  │ Clerk      │
│ versions │  │ 3D files │        │        └────────────┘
│ history  │  └──────────┘        ▼
│ costs    │                ┌───────────────┐
│ credits  │  ┌──────────┐  │ BACKGROUND    │
└──────────┘  │  CACHE   │  │ WORKERS       │
              │ (later)  │  │ (Python)      │
              └──────────┘  │ drawing recog.│
                            │ IFC/BIM files │
                            └───────────────┘
```

### The rule the diagram encodes

Follow any arrow that changes a building. **Every single one passes through THE MODEL module.** The drawing engine cannot write to the database. The 3D engine cannot write to the database. The AI cannot write to the database. Only the model module writes buildings.

---

# 6. The Technology Stack

Every choice, with the reason.

### Front end — what runs in the browser

| Technology | What it does | Why this one |
|---|---|---|
| **React** | Builds the user interface | Industry standard, huge ecosystem, the developer knows it |
| **Next.js** | Page routing, server rendering, app shell | Fast initial loads; handles routing and auth pages well |
| **TypeScript** | Adds type checking to JavaScript | Catches whole categories of bug before running. Non-negotiable on a project this size |
| **Tailwind CSS** | Styling | Fast to write, consistent, no separate stylesheet sprawl |
| **Radix UI** | Accessible components (dropdowns, dialogs) | Accessibility is genuinely hard; this solves it correctly |
| **TanStack Query** | Fetches and caches server data | Handles loading, errors, refetching, caching — but *not* the building model |
| **Zustand** | Small UI state (active tool, open panels) | Lightweight. Deliberately used for *only* throwaway UI state |
| **React Hook Form + Zod** | Forms and validation | Zod validates at runtime; TypeScript alone does not |

### The drawing and 3D engines

| Technology | What it does | Why this one |
|---|---|---|
| **PixiJS** | Renders the 2D plan | Fast 2D graphics with GPU acceleration. Right level of control for CAD — lower-level tools mean writing everything, higher-level ones fight you |
| **Three.js** | Renders the 3D model | The mature standard for browser 3D. Used **directly**, not through a React wrapper — explained in §16 |

### Back end — what runs on the server

| Technology | What it does | Why this one |
|---|---|---|
| **NestJS** | The API application | Enforces structure. For a solo developer with AI assistants, a framework with strong opinions prevents code sprawl |
| **PostgreSQL** | The database — the source of truth | Real transactions, flexible JSON columns, text search, vector search, geographic data. One database does everything we need |
| **Drizzle** | Talks to the database from code | SQL-first and type-safe. We see the actual queries, which matters for a versioned model |
| **Python + FastAPI** | Image and BIM processing | Only used where Python is genuinely required (below) |

### Everything else

| Technology | What it does | Why this one |
|---|---|---|
| **Cloudflare R2** | Stores uploads, exports, 3D files | Same as S3, but **no charge for data transfer out** — significant when serving 3D models and PDFs |
| **Cloudflare** | DNS, firewall, CDN, TLS | Cheap, effective, protects the origin server |
| **AWS ECS / Fargate** | Runs the application containers | Containers without managing servers or Kubernetes |
| **Neon** | Managed PostgreSQL | Can branch the database per pull request — genuinely valuable for one developer. Free tier covers all early development |
| **Clerk** | Login, signup, organisations | Authentication, invitations and MFA are weeks of work and a security incident if wrong. Kept behind our own interface so it can be swapped |
| **Stripe** | Payments | No serious alternative |
| **Temporal** | Long multi-step jobs | Survives crashes and resumes. Added only when genuinely needed |
| **Redis** | Cache and rate limiting | Added only when genuinely needed |
| **Sentry** | Error tracking | Free tier, pays for itself the first time something breaks |
| **OpenAI** | The AI models | Accessed through our own abstraction so the provider can change |
| **web-ifc / IfcOpenShell** | Reading and writing BIM files | The reference tools for IFC |

### Why TypeScript nearly everywhere

The browser must run JavaScript. The 2D engine, 3D engine, geometry maths and building model all live there. If the server also runs TypeScript, then **the building model, the quantity rules and the cost rules are written once and run in both places.**

For a solo developer, that is the single largest productivity decision in the stack.

### Why Python exists at all

Three things are effectively Python-only: **OpenCV** (computer vision for reading scanned drawings), **PyTorch / ONNX** (machine learning models), and **IfcOpenShell** (the reference BIM library).

Python is used **only** for those. It is not a general-purpose second backend.

---

# 7. Repository and Code Organisation

Everything lives in one repository. One repository does **not** mean one deployed application.

```
buildora-ai/
│
├── apps/                    ← things that get deployed
│   ├── web/                 The website and editor           [BUILD NOW]
│   ├── api/                 The API server                   [BUILD NOW]
│   ├── ai-worker/           Python: reads drawings           [LATER ~sprint 31]
│   ├── bim-worker/          Python: IFC files                [LATER ~sprint 33]
│   └── render-worker/       Photorealistic images            [NOT YET]
│
├── packages/                ← shared code libraries
│   ├── units/               Millimetres, conversion          [BUILD NOW]
│   ├── building-model/      THE BUILDING MODEL               [BUILD NOW]
│   ├── model-session/       Browser working copy             [BUILD NOW]
│   ├── cad-2d/              2D drawing engine                [BUILD NOW]
│   ├── engine-3d/           3D engine                        [BUILD NOW]
│   ├── qs-engine/           Quantity calculation             [BUILD NOW]
│   ├── cost-engine/         Cost calculation                 [BUILD NOW]
│   ├── api-contracts/       Shared request/response shapes   [BUILD NOW]
│   ├── design-system/       Buttons, inputs, colours         [BUILD NOW]
│   └── ai-tools/            AI tool definitions              [NOT YET ~sprint 27]
│
├── native/
│   └── geometry-wasm/       Fast geometry in Rust            [NOT YET - see below]
│
├── infrastructure/          Cloud configuration
└── docs/                    Documentation
```

### What each library is responsible for

| Library | Owns | Must never contain |
|---|---|---|
| `units` | Millimetres as the standard, all unit conversion | Anything about buildings |
| `building-model` | Walls, doors, rooms, geometry, validation, change commands | Anything about screens or databases |
| `model-session` | The browser's working copy, unsaved changes, syncing with server | Anything about drawing |
| `cad-2d` | Drawing the plan, mouse interaction, snapping | 3D. Database access |
| `engine-3d` | Building the 3D scene | 2D. Database access |
| `qs-engine` | Calculating quantities from the model | Costs |
| `cost-engine` | Calculating costs from quantities | Quantity rules |
| `api-contracts` | The shape of every API request and response | Business logic |
| `design-system` | Visual components | Anything about buildings |

### Two deliberate absences

**There is no `packages/domain`.** An earlier plan included one, but no document ever defined what belonged in it. An undefined library sitting beside well-defined ones becomes the place developers dump code they cannot categorise. Within months it is an unmaintainable tangle. It was deleted before a single line was written.

**There is no `packages/core` either.** The obvious fix — "create a shared-basics library instead" — just relocates the same problem. A library gets created when there is a real, demonstrated need, not a predicted one. If three libraries genuinely end up needing the same thing, we will create it then, with a written charter.

### Nothing is built before it is needed

`render-worker`, `geometry-wasm` and `ai-tools` do not exist yet. An empty folder is free. A configured-but-unused application costs time on every build and every deployment, forever.

---

# 8. How The Pieces Depend On Each Other

Dependencies flow in **one direction only**. This is checked automatically on every commit.

```
                    units
              (millimetres, conversion)
                      │
                      ▼
               building-model
         (walls, doors, rooms, commands)
                      │
                      ▼
               model-session
        (browser working copy, syncing)
                      │
           ┌──────────┴──────────┐
           ▼                     ▼
        cad-2d               engine-3d
      (2D drawing)          (3D viewing)


               building-model
                      │
                      ▼
                  qs-engine
             (quantity calculation)
                      │
                      ▼
                 cost-engine
               (cost calculation)
```

### The rules

1. **Arrows point down only.** Nothing points back up. No circles.
2. **The core libraries know nothing about frameworks.** `units`, `building-model`, `model-session`, `qs-engine` and `cost-engine` must never import React, Next.js, PixiJS, Three.js, NestJS, Drizzle, OpenAI, Stripe or Clerk.
3. **The 2D and 3D engines never import each other.** They both sit below `model-session` and communicate through it.
4. **The design system knows nothing about buildings.** It provides buttons and inputs.
5. **Applications use libraries. Libraries never use applications.**
6. **`cost-engine` may use `qs-engine`'s types.** Never the reverse.
7. **The Python workers share no TypeScript code.** They communicate over a documented API.

### Why rule 2 matters more than it looks

If the building model cannot import React, then the building model can be tested in a plain terminal with no browser, no graphics card and no database. Tests run in milliseconds. The most important and most intricate logic in the entire product — geometry, room detection, quantity rules — becomes the easiest part to verify.

It also means the model cannot accidentally depend on how it is displayed. That is what keeps the single-source-of-truth rule from eroding over time.

### Why `model-session` sits where it does

An earlier plan put the browser's working copy inside the 2D library. That would have meant the **3D engine depending on the 2D engine** — which makes no sense, and would have quietly coupled them forever.

Placing it in its own library, above both, means each engine depends on the working copy and **neither depends on the other**. Both are fed from the same place, so they physically cannot show different buildings.

---
---

# PART III — THE HEART OF THE SYSTEM

---

# 9. The Canonical Building Model

This is the most important part of the system.

### The structure

```
Project                     A job. Owned by an organisation.
  │                         Has currency, display units, datum.
  │
  └── Site                  Optional. Where the building is in the
      │                     real world (latitude/longitude).
      │
      └── Building          One or more per site.
          │
          └── Level         A storey. Ground floor, first floor,
              │             basement. Has a height above datum.
              │
              └── Element   Everything physical or spatial:
                            walls, doors, windows, rooms, slabs,
                            columns, beams, stairs, roofs.
```

### What every element carries

Regardless of what kind of element it is:

| Information | Why it exists |
|---|---|
| **A permanent unique ID** | The same ID identifies it in the 2D plan, the 3D scene, the IFC export, the quantities, the BOQ, the cost estimate and the audit log. Forever. |
| **Which organisation owns it** | Prevents one customer ever seeing another's data |
| **Which project and level** | Position in the structure |
| **What kind of thing it is** | Wall, door, window, room, slab… |
| **What it is attached to** | A door is attached to a wall |
| **Its type** | "Standard window type W-02" — see §11.8 |
| **Its assembly** | What it is made of, for costing — see §18 |
| **Its geometry** | Position and dimensions, stored as structured data |
| **Its properties** | Fire rating, finish, classification code… |
| **Where it came from** | Drawn by a person, proposed by AI, imported from a file, or read from a scan |
| **How confident we are** | Relevant when it came from AI or a scanned drawing |
| **Whether a human has checked it** | Low-confidence items must be reviewed |
| **Version history** | Which version created it, last changed it, deleted it |

### Provenance — a small decision with large consequences

Every element records where it came from and how confident we are about it, **from the very first wall drawn**, even though drawing recognition will not exist for many months.

Most teams add this later, when they build the scan-a-drawing feature. But by then the database is full of elements with no provenance, and it cannot be reconstructed. You end up with a system that cannot distinguish "an architect drew this precisely" from "an AI guessed this from a blurry photo" — which is exactly the distinction a professional needs.

Four columns, added on day one, that are impossible to add retroactively.

### Geometry is described, not drawn

A wall is stored like this:

```
A wall:
  centre line runs from point (1200, 4500) to point (6400, 4500)
  thickness: 200 mm
  height:    2700 mm
  made of:   external blockwork assembly
```

**Not** as a list of 3D triangles.

This matters enormously:

- You can **measure** it — the area is 5.2 m × 2.7 m. Triangles do not tell you that.
- You can **change** it — move one end and everything updates. Triangles must be regenerated.
- You can **cost** it — 200 mm of blockwork over that area. Triangles have no material.
- It is **tiny** — a few numbers instead of thousands of coordinates.
- It **means something** — it is a wall, not a shape.

Triangles are generated when needed for display, then thrown away.

### Relationships

Some connections are not simple ownership. A wall *bounds* a room. A beam is *supported by* a column. A duct *serves* a space. These are stored as explicit relationship records — `BOUNDS`, `CONNECTS_TO`, `SUPPORTED_BY`, `SERVES` — so they can be queried and enforced, rather than buried inside a JSON blob where nothing can find them.

---

# 10. Measurements, Units and Heights

### Everything is millimetres

All stored geometry is in millimetres. Always. No exceptions.

The user can *see* metres, centimetres, feet or inches — that is a display setting. But underneath, everything is millimetres, and all conversion happens in one small library.

**Why:** mixed units are a catastrophic and famous class of bug. A wall stored in metres and read as millimetres is 1000× wrong. By allowing exactly one internal unit, this class of error is eliminated rather than managed.

### Heights — the datum

This sounds like a small detail. It is not. Every 3D view, every volume calculation, every stair, and every BIM export depends on getting it right, and it is effectively impossible to change once buildings exist.

**The rule:**

```
Ground floor finished floor level  =  0 mm

Each level has ONE height value: its elevation above (or below) that point.

   Second floor    +6000 mm
   First floor     +3000 mm
   Ground floor         0 mm    ← the datum
   Basement        -3000 mm

Element heights are measured from THEIR OWN level, not from the ground.
```

So a 2700 mm wall on the first floor is stored as "2700 mm tall". Its actual height above ground (3000 + 2700 = 5700 mm) is **calculated when needed**, never stored.

### One number, not three

Floor-to-floor height is **not stored**. It is calculated:

```
floor-to-floor  =  next level's elevation  −  this level's elevation
```

An earlier draft of this architecture stored elevation, floor-to-floor height **and** slab thickness on each level. That was a mistake, caught in review.

Here is why. If a level says "I am at 3000mm" and also "my floor-to-floor is 3000mm", and the level above says "I am at 3200mm" — which is right? The moment anyone edits one without the other, the building contradicts itself, and there is no way to know which value to trust.

**One fact, stored once.** Everything else is calculated. Slab thickness belongs to the slab, where it physically belongs.

### Real-world location is kept separate

A building's position on Earth — latitude, longitude, survey datum — lives on the **Site**, not on walls and doors. Building geometry is always local to the building. Mixing the two would mean every wall carrying geographic coordinates, which is both wasteful and a source of precision errors.

---

# 11. Every Building Element, Defined

What each element is, and the rules it must obey.

### 11.1 Wall

```
centre line:  start point → end point
thickness:    e.g. 200 mm
height:       e.g. 2700 mm
assembly:     what it is made of
```

**Rules:**
- Thickness and height must be greater than zero.
- Start and end cannot be the same point.
- **The centre line is the truth.** The two faces are calculated from it, never stored.
- Net area = total area − all openings in it.
- How walls meet at corners is worked out when needed, never stored as separate geometry.

### 11.2 Door and 11.3 Window

```
attached to:  a specific wall
position:     distance along that wall
width, height
sill height:  windows only
swing/hand:   doors only
```

**Rules:**
- The host wall must exist and must be a wall.
- The opening must fit entirely within the wall's length and height.
- Deleting a wall must explicitly handle its doors and windows — never silently orphan them.
- Moving a wall moves its doors and windows with it.

### 11.4 Openings — one representation only

**A door *is* an opening. A window *is* an opening.**

We do not create a separate "opening" element for the hole a door sits in. The door is the hole.

This was a real inconsistency in the original design — openings existed both as their own element type *and* as properties embedded in doors and windows. Two ways to describe one hole.

The problem: quantity calculation must find every hole in every wall to subtract it from the wall area. Two representations means two pieces of code doing that. They will eventually disagree, and the net wall area — the single most important number in quantity surveying — will be wrong in a way nobody notices for months.

A standalone "opening" element exists **only** for holes with nothing in them: a service penetration, an archway, an unfilled gap.

**Quantity calculation sees all openings through one single interface.**

### 11.5 Room / Space

Covered fully in §12 — it is the most intricate element.

### 11.6 Slab (floors and ceilings)

```
boundary:     a closed shape
thickness
height offset from its level
voids:        holes, e.g. for stairs
```

**Rules:** boundary must be closed and not cross itself; voids must sit inside it; volume = (area − voids) × thickness.

### 11.7 Column, Beam, Stair, Roof

**Column:** a position, a cross-section, a height. Spans multiple floors only by explicit reference, never by accident.

**Beam:** a line, a cross-section. What it supports is recorded explicitly, never guessed later.

**Stair:** connects exactly two levels. Has tread depth, riser height, width, and a number of risers.
> **A validation that proves the datum design works:** risers × riser height must equal the actual height between the two levels. Because level elevations are stored as single authoritative values (§10), that check is simple and always correct. Had we stored floor-to-floor separately, the stair could validate against one number while the building used another.

**Roof:** a boundary and a slope.
> **A classic costing error, prevented:** roof area must be measured *on the slope*, not as seen from above. A 30° pitched roof has about 15% more surface than its footprint. Getting this wrong under-orders tiles on every project. There is an explicit test for it.

### 11.8 Element Types

A "type" is a repeated definition — for example, window type W-02, used thirty times in a building.

**Why it matters:**
- Window and door schedules are a core professional deliverable, and they group by type.
- "Make every W-02 window 1500 mm wide" should be one edit, not thirty.
- BIM files carry type definitions; round-tripping loses information without them.

**Kept strictly separate from assemblies:**

```
Type      = WHAT IT IS      "a 1200×1200 casement window, type W-02"
Assembly  = WHAT IT COSTS   "frame + glazing + ironmongery + fixing + labour"
```

These are different facts. Merging them would mean you cannot re-price a window type without redefining the window, or share one costing recipe across several types.

---

# 12. Rooms — How They Are Calculated

Rooms deserve their own section because the obvious approach is subtly wrong, and the error is expensive.

### Rooms are calculated, then saved

A user does not draw rooms. The system works out where they are from the walls, then saves the result so it can be named, numbered and given finishes.

### The process

```
1.  Look at how the walls connect      (using their centre lines)
         ↓
2.  Find the enclosed regions          ("this area is surrounded by walls")
         ↓
3.  Work out how walls meet at corners
         ↓
4.  Find the INNER FACE of each wall   ← the important step
         ↓
5.  Build the room boundary from those inner faces
         ↓
6.  Save the room, with a permanent ID
         ↓
7.  Record which walls bound it
```

### Step 4 is the one that matters

Wall centre lines are correct for working out *which* areas are enclosed. They are **wrong** for measuring the room.

```
        wall centre line
              ↓
    ┌─────────┼─────────┐    ← 200mm thick wall
    │         │         │
    │    ╔════╧════╗    │
    │    ║         ║    │
    │    ║  ROOM   ║    │    ← the actual usable space
    │    ║         ║    │       is between the INNER faces
    │    ╚════╤════╝    │
    │         │         │
    └─────────┼─────────┘

Measuring to the centre lines includes
HALF A WALL THICKNESS on every single side.
```

**The arithmetic:**

```
A room measuring 5000 × 4000 between wall centre lines, walls 200mm thick:

  Measured to centre lines:     5.000 × 4.000  =  20.00 m²   ← WRONG
  Measured to inner faces:      4.800 × 3.800  =  18.24 m²   ← CORRECT

  Error: 1.76 m² — about 9% too much, on every room.
```

That 9% flows into floor finishes, ceiling finishes, skirting length and wall finishes on **every room of every project**. An estimate built on it is wrong in a way a quantity surveyor will spot immediately and never trust again.

There is a permanent test asserting **18.24 m²**.

### What is saved

```
Room
  permanent ID       kept even when walls move
  which level
  the boundary       calculated from inner faces
  bounding walls     recorded as relationships
  name               "Kitchen"
  number             "G.04"
  finishes           floor, wall, ceiling
  user-edited flag   see below
```

### When walls move

**Normally:** the room is recalculated automatically. It **keeps** its ID, name, number and finishes. If the area changed noticeably, it is flagged so the user notices.

**If the user has manually adjusted the room:** it is marked as user-edited and is **never silently overwritten**. It may be flagged as possibly out of date, but the system does not overrule a person who has taken control.

### Area is always calculated

A user can never type in a room's area. Area comes from the boundary. If someone wants to record a target area, that is a note or a property — not the geometric truth.

### Where this code lives

Room detection sits in the **building model library**, not the drawing editor. It is deterministic geometry and must be testable without a screen. The editor triggers it and displays the result.

**On performance:** the algorithm will be written in ordinary TypeScript first. There is a temptation to reach for Rust for geometry work. We will not, unless measurement proves it necessary.

---

# 13. Changes, Versions and Undo

### Every change is a named instruction

The system never saves "the building looks like this now". It saves **what happened**:

```
CREATE_WALL   from (0,0) to (5000,0), 200mm thick, 2700mm high
MOVE_WALL     wall W-12, move end point to (5200, 0)
CREATE_DOOR   on wall W-12, 900mm from start, 900 × 2100
DELETE_ELEMENT  window W-03
CHANGE_MATERIAL element W-12 → insulated cavity wall
```

**Why:** you can undo it, audit it, replay it, show a client exactly what changed between revisions, and let AI propose one without applying it.

### What is stored

Four things work together:

```
1.  THE BUILDING AS IT IS NOW      Fast to read — no replay needed
2.  EVERY INSTRUCTION EVER GIVEN   Append-only. Never edited
3.  A SUMMARY OF EACH CHANGE       "these 3 elements changed, from X to Y"
4.  PERIODIC FULL SNAPSHOTS        About every 50 versions, for fast recovery
```

The third is what makes "what changed between revision C and revision D, and what did it cost?" a fast lookup rather than an expensive reconstruction. Revision cost analysis is a headline professional feature; this is what makes it practical.

### Two people editing at once

Every change says which version it is based on.

```
User A opens the project at version 42.
User B opens the project at version 42.

User B saves a change  →  becomes version 43.
User A saves a change  →  "based on version 42"

     The server sees the project is now at 43, not 42.
     User A's change is REJECTED with a clear message.
     User A reloads and reapplies their change.
```

**We reject rather than merge.** Silently merging two geometric edits produces a building nobody designed. A clear rejection is always better than a quiet corruption.

### One change is all-or-nothing

Moving a wall also moves its doors, recalculates the rooms it bounds, and adjusts connected walls at corners. **All of that succeeds together or none of it happens.** There is never a half-moved wall.

### History is never rewritten

A saved version is permanent. Undo does not delete history — it adds a new instruction that reverses the previous one. The record of what happened stays intact, which is what audit means.

### Dragging does not create a hundred versions

While the user drags a wall, the browser shows a live preview and saves **nothing**. When they release the mouse, **one** instruction is sent, creating **one** version.

---
---

# PART IV — WHAT THE USER SEES

---

# 14. The Editing Experience

### The browser's working copy

The editor cannot ask the server about every mouse movement — that would be unusably slow. It holds a **working copy** of the building in memory.

```
┌─────────────────────────────────────────┐
│         THE WORKING COPY                │
│                                         │
│  • the building, as currently shown     │
│  • which server version it came from    │
│  • changes not yet confirmed by server  │
└──────────┬──────────────────┬───────────┘
           │                  │
    ┌──────▼─────┐     ┌──────▼─────┐
    │  2D VIEW   │     │  3D VIEW   │
    └────────────┘     └────────────┘

  Both read the SAME copy.
  They cannot show different buildings.
```

**This is not a second database.** It is a working copy. The server remains the truth. If it were lost entirely, reloading the page would restore everything except unsaved edits.

### What happens when you move a wall

```
1.  You drag the wall              → live preview, nothing saved
2.  You release                    → instruction created
3.  Applied to the working copy    → 2D and 3D both update instantly
4.  Sent to the server             → with the version it was based on
5.  Server validates and saves     → new version returned
6.  Working copy confirms          → done

    If the server rejects it (someone else edited first):
    → you are told clearly, and can reload and reapply
```

Steps 3 and 4 happen at the same time, so the interface feels instant while the server stays authoritative.

### Who owns which state

| State | Where it lives | Why |
|---|---|---|
| The saved building | **PostgreSQL** | The truth |
| The working copy, unsaved edits | **`model-session`** | Fast interaction |
| Project lists, quantities, costs | **TanStack Query** | Ordinary server data, cached |
| Active tool, selection, open panels, camera | **Zustand** | Throwaway UI state |

**Two rules that matter:** the building model must never live in Zustand, and the query cache must never own working geometry. Both mistakes are easy to make and hard to undo.

---

# 15. The 2D Drawing Engine

### The split

```
React            →  toolbar, properties panel, level selector, status bar
Editor control   →  interprets mouse and keyboard, issues instructions
PixiJS           →  draws everything on the canvas
```

**React draws the interface around the canvas. PixiJS draws everything inside it.**

React is excellent at forms and panels. It is the wrong tool for ten thousand wall segments — it would grind to a halt. PixiJS talks to the graphics card directly.

### Screen pixels are never building measurements

There is exactly one conversion between the two:

```
building millimetres  ←→  screen pixels
```

Everything — clicking, snapping, dimensions — goes through it. **A pixel value must never reach the building model.** Zoom in and a wall is still 200 mm thick.

### What the editor does

| Feature | Notes |
|---|---|
| Selection | Click, shift-click, drag a box |
| Snapping | To endpoints, midpoints, intersections, perpendiculars, grid, alignments |
| Dimensions | Calculated from the model, not drawn separately |
| Wall drawing | Chained; corners resolved by the model |
| Doors and windows | Placed on a wall; position validated against it |
| Rooms | Detected automatically; user can name, merge, split, override |
| Levels | Switch floors |
| Layers | Show and hide categories |
| Undo/redo | Through the model, never a local list of screen states |

### Snapping is temporary

When a wall snaps to another wall's endpoint, that is a **drawing aid**. It is not stored. Only the resulting position is saved.

If snapping relationships were stored, they would become a hidden second model — constraints nobody declared, silently affecting geometry. Real constraint solving is a deliberate future feature, not an accident.

### Performance expectations

| Building size | Expectation |
|---|---|
| 1,000 elements | Comfortable, no special handling |
| 10,000 elements | Smooth, updating only what changed |
| 50,000 elements | Needs spatial indexing, possibly background threads |

Work moves to a background thread only when measurement shows it exceeding the frame budget — not on suspicion.

---

# 16. The 3D Engine

### How the 3D scene is built

```
The building model
        ↓
Adapters — one per element type
        ↓
Three.js scene
```

Each adapter converts one kind of element into 3D shapes:

- A **wall** becomes a box along its centre line, with openings cut out.
- A **door** becomes a frame and a panel at its position on the wall.
- A **slab** becomes an extruded shape, less any voids.
- A **column** becomes its section extruded along its height.

Heights are resolved here — the adapter adds the level's elevation to the element's local height. This is why moving a whole floor is a single-value change with no element edits.

### Three.js used directly, not through React

There is a popular library (React Three Fiber) that lets you write 3D scenes as React components. **We are not using it for the editor**, deliberately.

A professional editor needs to update precisely the objects that changed, dispose of graphics memory explicitly, map clicks to specific building elements, and control exactly what happens each frame. React's rendering model works against all four. It is a fine tool for a presentational 3D scene; it is the wrong tool for a CAD editor.

### WebGL2 only

There is a newer browser graphics standard (WebGPU). **We are not using it yet.**

Supporting both means two rendering paths, two sets of shaders, and double the testing — to solve a performance problem we have not yet demonstrated. If 3D performance becomes a constraint, the fixes in order are: instancing, level-of-detail, fewer draw calls, background geometry preparation. Only after all of those would a second rendering path be worth considering.

### Cleaning up memory

3D graphics memory is not cleaned up automatically. Every adapter must explicitly release geometry, materials and textures when an element is removed. Removing something from the scene is **not** the same as freeing its memory — that distinction is the most common cause of 3D applications slowly consuming all available memory during a long editing session.

### Imported BIM models are kept separate

```
Our own building  →  our adapters   →  Three.js scene
An imported IFC   →  IFC viewer     →  displayed separately
```

Two different paths, deliberately. The IFC display format is for showing files from other software. It is never the internal format for our own buildings.

---

# 17. How 2D and 3D Stay In Sync

### They are not synchronised. They read the same thing.

```
A change instruction
        ↓
The working copy
        ↓
   ┌────┴────┐
   ▼         ▼
2D view    3D view
```

There is no code that copies changes from 2D to 3D. There cannot be a sync bug, because there is no synchronisation — both views read one source.

### What is forbidden

```
✗  the 2D view updating the 3D view directly
✗  the 3D view writing to the database
✗  separate 2D and 3D models kept in step by code
✗  the renderers inventing their own IDs for things
```

### Knock-on effects are explicit

When a wall moves, other things must move too:

```
Moving a wall also affects:
   • its doors and windows          (they move with it)
   • rooms it bounds                (boundaries recalculated)
   • walls it meets at corners      (junctions redrawn)
```

That list is written down per element type. It is **not** "whatever the renderer happens to notice" — that is how 2D and 3D quietly diverge. There is a test asserting that after a wall moves, the model, the 2D view and the 3D view all agree.

---
---

# PART V — TURNING DESIGN INTO MONEY

---

# 18. Materials and Assemblies

### Geometry is not cost

A wall is 5.2 m × 2.7 m = 14.04 m². That is not a price. An **assembly** describes what the wall is actually made of:

```
ASSEMBLY: "200mm insulated cavity wall"

  per square metre:
    100mm concrete blocks      10.0 nr      + 5% waste
    50mm cavity insulation      1.0 m²      + 10% waste
    100mm facing brick         60.0 nr      + 5% waste
    wall ties                   2.5 nr
    mortar                      0.02 m³     + 15% waste
    bricklayer labour           1.2 hours
    labourer                    0.6 hours
    scaffolding                 (project-level)
```

So 14.04 m² of wall becomes: 140 blocks, 14 m² of insulation, 842 bricks, 35 ties, mortar, and 25 hours of labour — each with the right waste factor.

### Waste is data, not a constant in code

Different materials waste differently. Bricks cut to fit waste around 5%. Insulation board wastes more. Mortar wastes most. These are **configurable values**, versioned like everything else, because they vary by region, contractor and project.

The alternative — a `* 1.05` buried in the code — makes waste invisible, unauditable and unchangeable per project.

### Everything is versioned

Materials, assemblies and rates all have versions. When a cost estimate is produced, it records **exactly which version of each** it used. Changing a material's recipe next month must never silently change an estimate issued last month.

---

# 19. The Quantity Surveying Engine

### It is pure arithmetic

Given the same building and the same rules, it produces the same numbers. Every time. No AI, no randomness, no external calls.

### The canonical example

```
A wall:     5000 mm long × 2700 mm high
An opening: 1000 mm × 2100 mm

    Gross area:      5.0 × 2.7   =  13.50 m²
    Opening:         1.0 × 2.1   =   2.10 m²
    ─────────────────────────────────────────
    Net area:                       11.40 m²
```

This exact test exists permanently. If it ever fails, everything downstream is wrong.

### What is recorded for every quantity

```
Quantity: "external wall, blockwork, 200mm"

  Unit:               m²
  Gross:              156.40
  Deducted:            23.20        ← openings
  Net:                133.20
  Waste:                6.66        ← 5%
  Final:              139.86

  Calculated from:    47 specific wall elements
  Using rules:        NRM2 version 1
  From building:      version 128
  Engine version:     1.4.2

  Full working:       [step-by-step calculation stored]
```

Every intermediate value is kept. When a quantity surveyor asks "why is that 139.86?", the system can show every step and every wall that contributed.

### Measurement rules are data

Professional quantity surveying follows published standards. Ours defaults to **NRM2** (the RICS New Rules of Measurement 2, the UK standard for detailed measurement).

The rules are stored as **versioned data**, not written into the code. This means:

- Estimates from six months ago can be reproduced under the rules in force then.
- A different market can use a different standard without rewriting the engine.
- Which standard produced a number is always visible.

---

# 20. BOQ and Cost

### The chain

```
BUILDING (version 128)
      ↓
QUANTITIES        "133.20 m² of 200mm blockwork wall"
      ↓
BILL OF QUANTITIES  itemised, structured, professional format
      ↓
COST ESTIMATE     quantities × rates + overhead + profit + contingency
      ↓
REPORT            PDF or Excel
```

### Four levels of estimate

| Level | When | Based on | Accuracy |
|---|---|---|---|
| **Concept** | Early idea | Cost per m² | ±25% |
| **Elemental** | Design developing | Major elements | ±15% |
| **Detailed** | Design settled | Full BOQ with rates | ±10% |
| **Tender** | Going out to price | Supplier quotes | ±5% |

The same model supports all four. What changes is the depth of the rate data.

### Money is never a floating-point number

All money uses exact decimal arithmetic — in the database and in code.

**Why this is non-negotiable:** computers store `0.1 + 0.2` as `0.30000000000000004`. Over thousands of line items, those errors compound into visible discrepancies. An estimate that does not add up destroys credibility instantly. Exact decimal arithmetic makes the error impossible rather than unlikely.

---

# 21. Why Every Number Can Be Proven

**This is the commercial heart of the product.**

### Every estimate records exactly what produced it

```
COST ESTIMATE #4  —  "Revision C pricing"

  Building version:      128
  Quantity run:          #89
  BOQ version:           #34
  Rate book:             "UK South East 2026 Q3, v2"
  Cost engine:           v1.4.2
  Assumptions:           overhead 12%, profit 8%,
                         contingency 5%, VAT standard

  Total:                 £247,890.45
```

Given those inputs, the engine produces **exactly that figure**, today or in three years.

### The rule that protects trust

> **An old estimate is never recalculated using today's rates and still called the same estimate.**

If material prices rise 8%, last quarter's estimate must still show last quarter's figure. Producing an updated estimate creates a **new version** with new inputs recorded.

Without this rule, historical estimates change silently, revision comparison becomes meaningless, and every professional conversation about "what did this change cost?" becomes unanswerable.

### Tracing a number backwards

```
"Why is the blockwork £18,400?"

    £18,400
       ↑  rate: £138.12/m² from rate book "UK SE 2026 Q3 v2"
    133.20 m²
       ↑  gross 156.40 less openings 23.20, per NRM2 v1
    47 wall elements
       ↑  listed individually with each one's contribution
    Building version 128
       ↑  changed by "Move north wall 300mm" on 14 Aug 2026
```

Every step is recorded. This is what separates a professional tool from a calculator.

### Manual overrides are recorded, not hidden

A quantity surveyor may override a quantity or a rate based on judgement. The system keeps **both** the calculated value and the override, with who changed it and why. The trail is never broken.

---
---

# PART VI — INTELLIGENCE

---

# 22. The AI Architecture

### The rule

> ## AI suggests. It never changes anything itself.

The AI has **no ability to write to the database**. Not restricted by a rule someone might forget — it has no such capability. It writes proposals to a separate table. Only ordinary, tested, deterministic code applies changes to buildings.

### What AI is good for, and what it is not

| AI is used for | AI is never used for |
|---|---|
| Understanding what the user asked for | Calculating geometry |
| Suggesting design changes | Calculating quantities |
| Explaining what a number means | Adding up money |
| Reading and summarising documents | Working out tax |
| Classifying and labelling | Deciding credit balances |
| Interpreting an ambiguous drawing | Anything that must be exactly right |

A language model producing a plausible-looking wrong number is far more dangerous than one refusing to answer. Arithmetic belongs to arithmetic.

### Model tiers

The code refers to four capability levels:

```
FAST       simple lookups, quick answers
BALANCED   normal questions and conversation
ADVANCED   analysis, complex reasoning
EXPERT     major design or cost optimisation
```

Which actual model sits behind each tier is **configuration**, never written into the application code.

**Why:** providers change models and prices regularly. With this design, a price change or a model switch is a configuration update. Without it, it is a code change and a pricing crisis.

### Pricing is versioned data

Every AI provider rate is stored with the date it was last verified against the provider's published pricing:

```
provider, model, input rate, cached rate, output rate,
valid from, valid to, LAST VERIFIED ON, source
```

Costs for a past request are calculated using the rates that were in force **at that time**, so historical cost analysis stays accurate when prices change.

> **A note on verification.** During architecture review, one reviewer asserted that the model names and prices in the project's financial documents were invented, and raised it as a blocking issue. They were checked against the provider's official documentation and found to be entirely accurate. The claim was withdrawn.
>
> The lesson was worth keeping: the reviewer asserted something did not exist instead of verifying it. That is why every rate now carries a *verified on* date. It makes the difference between "checked" and "assumed" visible in the data itself.

### Keeping cost under control

- Cheaper models handle simple work; expensive models are used only when needed.
- Repeated context is cached rather than re-sent.
- Tools return small, focused results — never the whole building.
- Every request has a budget ceiling.
- Credits are reserved before expensive work starts.
- An emergency switch can disable AI features entirely without affecting design, quantities or costing.

---

# 23. How AI Changes A Building Safely

### The full path

```
1.  User: "make the kitchen 500mm wider"
              ↓
2.  AI Gateway — picks an appropriate model
              ↓
3.  AI uses read-only tools to understand the building
              ↓
4.  AI produces a PROPOSAL (not a change):
         "move wall W-12 north by 500mm"
         "widen window W-04 to suit"
              ↓
5.  Applied to a DRAFT COPY — the real building untouched
              ↓
6.  VALIDATED — is the geometry legal? does the door still fit?
              ↓
7.  QUANTITIES AND COST RECALCULATED on the draft
              ↓
8.  SHOWN TO THE USER:
         "Kitchen: 12.4 m² → 14.1 m²
          Blockwork: +2.3 m²
          Cost: +£340"
              ↓
9.  USER APPROVES
              ↓
10. VERSION RE-CHECKED — has anyone else edited meanwhile?
              ↓
11. APPLIED through the same ordinary code path as a manual edit
              ↓
12. New building version
```

### Step 10 matters more than it looks

A user might approve a proposal several minutes after it was generated. In between, a colleague may have edited the building. The version is therefore checked **again at the moment of approval**, not only when the proposal was created. Otherwise an approval could silently overwrite someone else's work.

### Uploaded documents are treated as untrusted

A construction PDF could contain text designed to manipulate an AI assistant — for example, hidden instructions telling it to delete data or reveal information.

The defences:
- Document content is clearly marked as *data*, never as instructions.
- Document text can never directly trigger a tool.
- Any proposal derived from a document **always** requires human approval.
- Every tool is restricted to the requesting user's own organisation and project.

### Confidence is always visible

AI and scan-derived results carry a confidence level. Low-confidence items are flagged for review and are **never** silently accepted as fact.

---

# 24. Reading Existing Drawings

Users can upload a PDF, an image or a CAD file and get an editable building.

### The approach

```
Upload
   ↓
What kind of file is it?
   ↓
┌──────────────────┬──────────────────┐
│  VECTOR (PDF/CAD)│  IMAGE (scan/photo)│
│  Exact lines     │  Pixels            │
│  already present │                    │
│       ↓          │        ↓           │
│  Read them       │  Computer vision   │
│  directly        │  finds shapes      │
└──────────────────┴──────────────────┘
   ↓
AI interprets MEANING
   "this rectangle is a door"
   "this text is a room name"
   ↓
Confidence recorded for everything
   ↓
USER REVIEWS AND CONFIRMS
   ↓
Building created
```

### Why not simply ask an AI to read the drawing?

Because when a file already contains exact line coordinates, converting it to an image and asking an AI to guess where the lines are is strictly worse: it is less precise, costs more, has no audit trail, and gives no confidence signal.

**Exact data is read exactly. AI is used for meaning and ambiguity** — which is what it is genuinely good at.

### Scale is always confirmed by a human

Establishing that "this many pixels equals one metre" is the highest-risk step in the whole process. A scale error does not look wrong — it silently makes **every quantity in the project** wrong by the same factor.

Scale is therefore always an explicit, user-confirmed step. Never inferred silently.

---

# 25. BIM and IFC

**IFC is the standard file format for exchanging building models between professional software.**

### IFC is for exchange, not for us

We import IFC and export IFC. We do **not** use it as our internal format.

**Why this matters:** several companies in this field have adopted IFC as their internal model, reasoning that it gives free interoperability. They then discover that IFC is verbose, slow to work with interactively, awkward for parametric editing, and changes with each schema revision — and their entire product is permanently shaped by those constraints.

Our model is designed for editing, measuring and costing. IFC is translated at the boundary.

### Identity is preserved across round trips

When a file is imported, we record which of our elements corresponds to which IFC object. Export, edit elsewhere, re-import — and elements are matched up rather than duplicated.

### The tools

| Purpose | Tool | When |
|---|---|---|
| Viewing imported IFC in the browser | web-ifc / That Open | ~sprint 33 |
| Reading and writing IFC on the server | IfcOpenShell (Python) | ~sprint 33 |
| Advanced solid geometry | OpenCascade | Only if genuinely needed |
| DWG (AutoCAD) support | Commercial licence | Only if customers pay for it |

DWG support requires a commercial licence costing several thousand pounds a year. It waits for demonstrated paying demand.

---

# 26. Documents and Project Q&A

Users upload drawings, specifications, contracts and reports, then ask questions about them.

### The chain

```
Document uploaded
   ↓
Revisions tracked          (v1, v2, v3 — which one was cited?)
   ↓
Split into passages
   ↓
Each passage indexed for meaning-based search
   ↓
User asks a question
   ↓
Relevant passages found — FILTERED TO THEIR PROJECT FIRST
   ↓
AI answers using those passages
   ↓
Answer cites its sources
```

### Answers must cite sources

An answer traces back to the passage, the document revision and the original file. A professional needs to verify a claim against the actual specification — an uncited answer is not useful in construction.

### Filtering before searching

The restriction to the user's own organisation and project is applied **as part of** the search, not as a filter afterwards. Filtering after the fact is a classic way to leak another customer's data when result limits interact with the search index. There is a specific test for this.

### No separate search system

PostgreSQL handles both text search and meaning-based search. A dedicated search product is a second system to run, secure, back up and pay for — and it will not be needed at this scale.

---
---

# PART VII — RUNNING A BUSINESS ON IT

---

# 27. Accounts, Organisations and Permissions

### The structure

```
User  ──belongs to──►  Organisation
                            │
                            ├── Projects
                            ├── Billing and credits
                            ├── Shared material libraries
                            └── Team members
```

A person can belong to several organisations — a consultant working for multiple firms, for example.

### The organisation is the wall

**The organisation is the security boundary.** One organisation must never see another's data. This is treated as the most serious category of defect in the system: it blocks a release outright.

### Two independent defences

**Defence 1 — every query is scoped.** Every database function requires the organisation as a parameter:

```
✓   findProject(organisationId, projectId)
✗   findProject(projectId)
```

**Defence 2 — the database enforces it too.** PostgreSQL row-level security means that even if a developer forgets defence 1, the database returns nothing rather than another customer's data.

### Timing, deliberately staged

From the earliest work on organisations: every table carries an organisation, every query is scoped, tests prove one organisation cannot read another's data, the security policies are written, and the mechanism is verified against the actual database connection pooling — because session-based security and connection pooling interact in ways that must be proven rather than assumed.

**Hard gate: enforcement must be switched on and verified before any external user ever accesses the system.**

The staging is deliberate. Forcing a poorly-understood security configuration into every development path early can create its own failures. The requirement is that it is designed, built and proven early — and enforced before exposure. The goal is working isolation, not a tick in a box.

### Cost visibility is a separate permission

Being able to see a project does not mean being able to see its cost. In construction, contractors routinely share designs with clients while keeping their margin private. Cost visibility is therefore its own distinct permission.

---

# 28. The Database

### One database

One PostgreSQL database holds everything. Not one per customer, not one per module.

**Why:** a single database means simple transactions, simple backups, simple joins, easier debugging, lower cost and easier migrations. Splitting is possible later if genuinely needed — but for one developer, splitting now would create constant distributed-system problems in exchange for scaling we do not need.

### What each storage system is for

| System | Holds | Never holds |
|---|---|---|
| **PostgreSQL** | Everything authoritative | Large files |
| **Object storage** | Uploads, exports, 3D files, reports, snapshots | Anything the database needs to query |
| **Cache** | Temporary speed-ups, rate limits | Anything irreplaceable |
| **Workflow engine** | Long-running job state | Building data |

If the cache were wiped entirely, nothing would be lost.

### Key decisions

**Identifiers.** Primary keys are UUIDs — 16 bytes, standard, efficient to index, generated without asking the database.

Human-facing references are separate: `PRJ-8K4D2A` for support conversations and URLs. That reference is **never** used to link records together, so the display format can change without touching the database's internal structure.

> The original example code used a different scheme — text keys like `prj_<uuid>`. Since AI coding assistants copy working examples far more readily than they follow written rules, that example was corrected before any code was written. Changing primary key types after data exists means rewriting every table and every link in the database.

**Money.** Exact decimal type. Never floating point.

**Dates.** Always stored with timezone, always UTC.

**Flexible data.** Geometry and properties are stored as structured JSON, but never as an untyped dumping ground — each has a defined shape, a version number, and is validated before saving.

**Large files.** Never in the database. Files go to object storage; the database keeps the reference.

### Changing the schema safely

Schema changes are committed files, applied as an explicit deployment step — **never automatically when the application starts**, which can corrupt data if two copies start at once.

Significant changes follow a four-step pattern: add the new structure, copy the data across, switch the application over, then remove the old structure — each step separately deployed, so the application always works during the transition.

---

# 29. Billing, Plans and Credits

### Customers buy credits, not tokens

```
CUSTOMER SEES              WE TRACK INTERNALLY
─────────────              ───────────────────
AI Design Credits          input tokens, cached tokens,
Render Credits             output tokens, actual cost
Project limits             per provider call
Team seats
Storage
```

**Why separate:** if AI provider prices change, we adjust our internal accounting. The customer's plan does not change. Had we billed raw tokens, every provider price change would become a customer pricing crisis.

### Plans

| Plan | Roughly for |
|---|---|
| **Free** | Trying it out — limited projects, small AI allowance |
| **Home** | Homeowners — small project limit, basic reports, conceptual estimates |
| **Pro** (~$49/month) | Professionals — full QS/BOQ/cost, IFC, higher AI allowance, render credits |
| **Business** | Teams — more seats, pooled credits, shared libraries, more storage |
| **Enterprise** | Custom contract |

All limits are configurable values, not hard-coded.

### Credits are a ledger, not a number

We never store "this customer has 27 credits". We store **every event**:

```
GRANT       +100   monthly subscription allowance
RESERVE      -10   starting an AI design task
RELEASE       +3   task used less than reserved
CONSUME       -7   final actual usage
REFUND        +7   task failed, credits returned
EXPIRE      -20    unused credits expired
```

The balance is calculated from the history.

**Why:** a single number cannot answer "why is it 27?", cannot survive two simultaneous requests safely, and cannot be reconciled. A ledger can do all three — and when a customer disputes a charge, there is a complete, ordered record.

### Reserve before working

```
Reserve credits  →  do the work  →  charge what was used, return the rest
                 →  work fails   →  return everything
```

This prevents a customer being charged for work that failed, and prevents two simultaneous requests both succeeding when only one had enough credit.

### Retries never double-charge

A single AI request might internally involve several model calls, including escalating to a more capable model. **The customer is charged once for the task.** Internal cost is recorded per call for our own margin analysis.

### Payment safety

- **Stripe** is authoritative for payments, invoices and refunds.
- **We** are authoritative for plans, entitlements, credits and feature access.
- Credits are granted **only** on server-confirmed payment — never because a browser reached a success page. A browser can be manipulated.
- Payment notifications are recorded once and processed safely even if delivered multiple times, which payment providers legitimately do.

---

# 30. Security

### Where the risk is

| Area | Control |
|---|---|
| **One customer seeing another's data** | Two independent layers (§27). The most serious defect class |
| **Login** | Handled by a specialist provider, behind our own interface |
| **Permissions** | Every sensitive action checks user, organisation, role and the specific resource |
| **File uploads** | Private by default; short-lived upload links to one exact location; type and size validated |
| **File processing** | Untrusted files parsed in isolated workers, never in the main API |
| **AI manipulation** | Document content treated as data; AI-derived changes require approval |
| **Fetching external URLs** | Not supported. Upload only |
| **Payment tampering** | Server-side confirmation only |
| **Secrets** | Never in the code repository; short-lived cloud credentials |
| **Logging** | Never log passwords, tokens, keys, download links or document contents |

### Why "no fetching external URLs"

If a user could ask the system to import a file from a web address, the server would fetch it from inside our private network, using our cloud credentials. That is a well-known attack path for reaching internal systems.

For now: upload only. If URL import is ever added, it will need an approved-addresses list and a network-restricted worker.

### Authenticated is not authorised

Being logged in proves who someone is. It says nothing about what they may do. Every sensitive operation checks the specific resource, not just the session.

---
---

# PART VIII — OPERATING IT

---

# 31. Infrastructure and Deployment

### Production shape

```
                    Users
                      │
                 CLOUDFLARE            security, caching, TLS
                      │
                 LOAD BALANCER
                 ┌────┴────┐
              WEB APP   API APP        containers, auto-scaled
                            │
         ┌──────────┬───────┴──┬──────────────┐
         ▼          ▼          ▼              ▼
     DATABASE   FILE STORAGE  WORKFLOW    EXTERNAL
     (managed)  (Cloudflare)  (later)     OpenAI, Stripe, Clerk
```

**No Kubernetes.** Containers run on a managed service with no cluster to operate. Kubernetes solves problems a one-person team does not have, and creates several it cannot afford.

### Deployment discipline

- A version is built **once** and the identical artefact is promoted through staging to production.
- Deployments reference an exact build fingerprint, never a moving label like "latest".
- Database changes are a separate, explicit step.
- The cloud is accessed with short-lived credentials, never stored keys.
- "Is it alive?" and "is it ready for traffic?" are separate checks — the application must not report itself dead simply because an external provider is down.

### Nothing is provisioned before it is needed

This is a real cost decision, not a preference.

| Service | Switched on when | Roughly |
|---|---|---|
| Database (local + managed) | Immediately | Sprint 02 |
| Error tracking, CI | Immediately | Sprint 02 |
| File storage | First upload feature | ~sprint 31 |
| Cache | AI rate limiting needs it | ~sprint 27 |
| Workflow engine | First genuinely long job | ~sprint 31 |
| Vector search | Document Q&A | ~sprint 34 |
| Geographic data | Site analysis | ~sprint 39 |
| **Cloud production** | **First external user** | **~sprint 35** |
| Fast geometry (Rust) | Only if measurement proves it necessary | Evidence only |

**Sprints 1 to 26 run on a laptop plus a free-tier database.** Roughly £0 per month during the longest stretch of development.

The original plan switched on the workflow engine around sprint 2 — about seven months before anything needed it. That is seven months of subscription cost, setup time, local development friction, and constraints on how workflow code could be written, all for nothing.

---

# 32. What Gets Built When

### The order is not negotiable

```
Foundations
     ↓
Accounts and projects
     ↓
THE BUILDING MODEL          ← nothing works without this
     ↓
2D editing
     ↓
3D viewing
     ↓
Materials and assemblies
     ↓
QUANTITIES
     ↓
BOQ
     ↓
COST
     ↓
REPORTS                     ← FIRST SELLABLE PRODUCT
     ↓
AI assistant
     ↓
Drawing recognition
     ↓
BIM / IFC
     ↓
Documents and Q&A
     ↓
Billing
     ↓
Public launch
```

Each stage genuinely requires the one before it. There are no quantities without a model. No cost without quantities. No useful AI without something deterministic for it to propose changes against.

### AI comes late, deliberately

The tempting move is to build the AI generation first — it demos well. It is the wrong order. AI generating buildings before there is a model to validate against, quantities to check it, or costs to evaluate it produces impressive-looking output that nobody can trust or verify.

The deterministic core comes first. Then AI has something to work with.

### Two milestones, not one

| Milestone | What it proves |
|---|---|
| **Model to Cost** | Design → 2D → 3D → materials → quantities → BOQ → cost → professional report. **Sellable. No AI required.** |
| **Full platform** | Everything above plus AI, drawing recognition, BIM, documents and billing |

The first is roughly two-thirds of the way through the plan. It is a real product on its own — it does the thing no design tool does simply and no costing tool does from a designed model.

**Reports were deliberately moved earlier** in the plan, from near the end to immediately after costing. Professional users judge software by the document it produces, and a report is the natural point at which to start charging.

### The honest risk

The full plan is long for one person. The architecture supports something sellable well before the end, and that gap is deliberate rather than accidental. The largest risk in this project is not technical — it is running out of energy before reaching revenue.

---

# 33. Testing

Test what breaks the business. Do not test what a framework already guarantees.

### The tests that matter, in order

**1. One customer cannot see another's data.** For every data access function. The single most important test in the system.

**2. Quantity golden tests.** The 11.40 m² net wall area. The 18.24 m² room area. Roof area on the slope. Each locked to a rule version.

**3. Save and reload.** Create a building, save it, load it, confirm it is identical. Catches the silent corruption of saved projects.

**4. Simultaneous edits.** Two changes from the same starting version — exactly one succeeds.

**5. Credit safety.** Parallel requests against a low balance never produce a negative one.

**6. Duplicate payment notifications.** The same event twice grants credits once.

**7. Cost reproducibility.** An old estimate recalculated from its recorded inputs produces exactly the same total.

**8. Schema changes.** Every database migration runs successfully against realistic data.

**9. Knock-on effects.** Moving a wall updates its doors, its rooms, the 2D view and the 3D view.

**10. Geometry rules.** Net area never exceeds gross area. Openings stay within their wall.

### Not worth testing

Visual components. Thin API controllers. Auto-generated types. Styling variants. Framework behaviour.

### Never do this

Never disable a failing test to make the build pass. A failing test is information. Turning it off does not fix the problem — it hides it until it costs more.

---

# 34. Monitoring

### The minimum before real users

- **Error tracking** on the website and the API, tagged with the release version.
- **Structured logs** including which organisation, project, job and building version each entry relates to.
- **Four alerts only:**
  1. API error rate rising
  2. Database connections running out
  3. Payment notifications failing to process
  4. AI spend exceeding its daily budget
- **Two dashboards:** system health, and AI cost versus revenue.
- **Health checks** so failing containers are replaced automatically.

That is genuinely enough. Elaborate monitoring described but never built is worse than a small amount actually in place.

### Never logged

Passwords, tokens, API keys, full download links, or the contents of customer documents.

---

# 35. When Things Break

Designed-for failure modes:

| What fails | What happens |
|---|---|
| **OpenAI is down** | AI features fail cleanly; reserved credits returned. **Design, editing, quantities and costing keep working.** |
| **Stripe is down** | Existing customers keep their access. New purchases wait. Reconciled afterwards. |
| **Cache is down** | Slower. Nothing lost. Never rebuild project data from cache. |
| **File storage is down** | Uploads and exports fail. Editing continues. |
| **Workflow engine is down** | Long jobs pause and resume. Normal work unaffected. |
| **Database is down** | Genuine outage. The one dependency with no graceful degradation. |

**The principle:** a failure in an optional part must never take down the core product. Someone editing a building and checking costs should not care that an AI provider is having a bad day.

### Backups

Continuous backup with point-in-time recovery, a snapshot before any risky schema change, and — the part most teams skip — **periodic tested restores**. A backup that has never been restored is a hope, not a backup.

---

# 36. Growing From 10 to 100,000 Users

**No fundamental rewrite is required.** Here is what actually changes.

| Users | What changes |
|---|---|
| **10 → 1,000** | Nothing. One application instance, one database. |
| **1,000 → 10,000** | Run more copies of the API. Add connection pooling. Watch AI spend. |
| **10,000 → 100,000** | Add a read-only database copy for reports. Split the largest history tables by date. Possibly separate the heaviest background work. |

### Why no rewrite is needed

The expansion points are built in from the start:

- **The API is organised into separate modules,** so one can be split out if it ever genuinely needs independent scaling.
- **The application holds no memory of individual users,** so adding copies is trivial.
- **Large files are already outside the database.**
- **History tables are already designed to be split by date.**
- **Quantities and costs are already calculated in isolated libraries** that could run separately.

### Editor performance

| Building size | Expectation |
|---|---|
| 1,000 elements | Comfortable |
| 10,000 elements | Smooth, updating only what changed |
| 50,000 elements | Needs spatial indexing and possibly background threads |

### Performance work is driven by measurement

Rust, background threads, database copies and table splitting all happen **when profiling proves they are needed**. Never on suspicion. Optimising the wrong thing costs weeks and usually makes the code worse.

---
---

# PART IX — THE DECISIONS

---

# 37. All 18 Architecture Decisions Explained

These are formally recorded. Changing any of them requires a documented, approved decision that explicitly replaces it.

---

**1 — There is one canonical Building Model.**
2D, 3D, IFC, quantities, BOQ and cost are all views of it.
*Alternative rejected:* separate 2D and 3D models kept in sync. They always drift, and quantities become untrustworthy.
*Reversibility:* effectively none. This is the founding decision.

**2 — Store the current state, plus every instruction, plus change summaries, plus periodic snapshots.**
Gives undo, audit, AI preview, rollback and revision comparison without the cost of a fully event-driven system.
*Alternative rejected:* storing only the current state. No history, no audit, no revision comparison.
*Reversibility:* none. History cannot be reconstructed after the fact.

**3 — One API application, internally modular; separate processes only for genuinely different runtimes.**
Python exists only for computer vision and IFC libraries.
*Alternative rejected:* microservices. For one developer that means distributed transactions and many deployments in exchange for scaling we do not need.
*Reversibility:* moderate — the module boundaries are the seams for later splitting.

**4 — UUID primary keys, with a separate human-readable reference.**
`PRJ-8K4D2A` for support and URLs; never used to link records.
*Alternative rejected:* text-based prefixed keys — four times the storage on every index, and invites treating IDs as parseable strings.
*Reversibility:* none once data exists.

**5 — Ground floor is zero; each level has one elevation; element heights are relative to their level.**
Floor-to-floor is calculated, never stored.
*Alternative rejected:* storing absolute heights on every element — moving one floor would rewrite every element in it.
*Corrected in review:* an earlier version stored three overlapping height values per level, which could contradict each other.
*Reversibility:* none once geometry exists.

**6 — Rooms are calculated from walls, bounded by inner wall faces, then saved with a permanent identity.**
*Alternative rejected:* measuring to wall centre lines — overstates every room by about 9%.
*Reversibility:* none once quantities have been issued to customers.

**7 — Element types are separate from cost assemblies.**
Type is what something is; assembly is what it costs.
*Alternative rejected:* using assemblies as types — you could not re-price a window type without redefining the window.
*Reversibility:* moderate.

**8 — The browser's working copy lives in its own library, independent of 2D and 3D.**
*Alternative rejected:* putting it in the 2D library — the 3D engine would have depended on the 2D engine.
*Corrected in review:* this was the original plan.
*Reversibility:* moderate while the editor is small.

**9 — Measurement rules are versioned data. NRM2 is the initial default.**
*Alternative rejected:* rules written into the code — no second market without a rewrite, and historical estimates not reproducible.
*Reversibility:* the default can change freely before launch.

**10 — AI model names and prices are versioned configuration, behind capability tiers.**
Every rate carries a verification date.
*Alternative rejected:* model names in the application code — every price change becomes a deployment.
*Reversibility:* full.

**11 — Strict package boundaries; no undefined `domain` library; no `core` library yet.**
*Alternative rejected:* creating a general-purpose shared library immediately — it becomes the same dumping ground under a better name.
*Reversibility:* full.

**12 — Infrastructure is switched on when something needs it.**
*Alternative rejected:* provisioning everything up front — months of cost and friction before first use.
*Reversibility:* full — any gate can be brought forward.

**13 — AI proposes; the system validates; the user approves; ordinary code applies.**
AI has no write access to building data.
*Alternative rejected:* AI writing directly with validation — one gap in validation corrupts the building, with no record of intent.
*Reversibility:* none in the safety direction.

**14 — Credits are a ledger with reservations. Money is exact decimal.**
*Alternative rejected:* a simple balance number — unexplainable, unsafe under concurrency, impossible to reconcile.
*Reversibility:* none.

**15 — Every estimate records exactly which model version, quantity run, BOQ, rates, engine version and assumptions produced it.**
Old estimates are never recalculated with current rates.
*Alternative rejected:* recalculating on demand — historical estimates change silently and revision comparison becomes meaningless.
*Reversibility:* none.

**16 — Organisation is the security boundary, with two independent layers.**
Designed and tested from the start; database-level enforcement mandatory before any external user.
*Alternative rejected:* application-level checks only — one forgotten line becomes a data breach.
*Reversibility:* none in the safety direction.

**17 — WebGL2 only. The newer graphics standard is deferred.**
*Alternative rejected:* supporting both — two rendering paths and double the testing for an undemonstrated benefit.
*Reversibility:* full — adding it later is additive.

**18 — IFC is an exchange format, never our internal model.**
*Alternative rejected:* IFC as the internal model — the entire product becomes constrained by its limitations.
*Reversibility:* effectively permanent in direction.

---

# 38. What We Deliberately Are Not Building

Naming these prevents them being re-proposed every few months.

### Not part of the architecture at all

```
Kubernetes              containers on a managed service are sufficient
Message brokers (Kafka) nothing needs them
Service mesh            no service mesh, no services to mesh
Database sharding       one database handles the modelled scale
Multiple regions        no requirement
One database per customer  no requirement
Go services             would be a fourth language for no reason
Microservices           module boundaries already provide the seams
```

Each of these requires a documented, approved decision to introduce.

### Deferred until genuinely needed

```
Real-time collaboration   single-user editing is the MVP.
                          The change-instruction design already
                          leaves the door open.
DWG (AutoCAD) support     needs paying customers to justify the licence
Advanced solid geometry   the current model handles what we need
Photorealistic rendering  a later feature
GPU servers               follows rendering
Read-only database copies when measurement shows the need
Splitting large tables    at roughly 50 million rows
Rust for geometry         only when profiling proves it necessary
A separate search system  PostgreSQL is sufficient at this scale
```

### The rule that keeps this honest

> **Appearing in a roadmap document does not make something approved.**

A technology is approved when it is needed, measured and decided — not when it is mentioned.

---

# 39. The Rules That Must Never Be Broken

If any of these is violated, the release stops.

**On the building:**
1. One canonical building model. 2D, 3D, IFC, quantities, BOQ and cost are views of it.
2. Every change goes through a named instruction.
3. Every change states which version it is based on. Saved versions are never altered.
4. All geometry in millimetres. Heights relative to their level. One elevation per level.
5. Room boundaries from inner wall faces. Area is calculated, never typed in.
6. A door or window **is** its opening. One route for opening deduction.
7. Element type and cost assembly are different things.
8. No triangles or meshes stored as the real building.

**On the software:**
9. The browser's working copy is a working copy, never a second truth.
10. Drawing engines hold no building data and invent no identities.
11. Core libraries import no frameworks. Dependencies flow one way. No circles.
12. No new library, application or paid service without something that actually needs it.

**On data:**
13. UUID primary keys. The human-readable reference never links records.
14. Money is exact decimal. Never floating point.
15. Every query on customer data is scoped to their organisation.
16. Database changes are explicit deployment steps, never run automatically at startup.

**On AI:**
17. AI proposes. It never writes building data.
18. AI never calculates geometry, quantities or money.
19. Uploaded documents are data, never instructions.

**On money:**
20. Credits are a ledger with reservations, never a single number.
21. Credits are granted only on server-confirmed payment.
22. Payment notifications are processed safely when repeated.
23. Old estimates are never recalculated with today's rates.

**On process:**
24. Never disable a failing test to make the build pass.
25. Never log secrets, tokens, download links or document contents.
26. Never claim something is done without verifying it.

---

# 40. What Is Still Undecided

Honesty about what is genuinely open.

### Nothing here blocks starting work.

### To decide during implementation

| Decision | Roughly when |
|---|---|
| Which room-detection algorithm | At the room feature |
| Exact snapshot frequency | At the versioning feature |
| When to move work to background threads | On measurement |
| Knock-on effect lists for slabs, columns, beams, stairs | As each is built |
| Whether AI proposals can be partly approved | At the AI feature |
| Which text-embedding model | At the documents feature |
| Infrastructure tooling choice | At production setup |
| Whether the website runs on the same platform as the API | At production setup |
| **Exactly who can see cost data** | **At the project feature — earliest item, and a business decision** |
| How reports are generated | At the reports feature |
| Which IFC version to target | At the BIM feature |
| Confirming the launch market | Before launch — NRM2 assumes the UK |

### Deferred until there is evidence

Rust for geometry · read-only database copies · splitting large tables · real-time collaboration · moving to a different managed database · splitting the API · Kubernetes · multiple regions · DWG support · advanced geometry · GPU rendering · a separate search system · usage-based billing.

Each needs production evidence, not speculation.

---

# 41. Glossary

**Adapter** — Code converting a building element into something drawable.

**Assembly** — A recipe of materials, labour and waste that turns geometry into cost.

**Audit trail** — The record of what changed, when, by whom, and what resulted.

**BIM** — Building Information Modelling. A building model carrying data, not just shapes.

**BOQ** — Bill of Quantities. The itemised professional list of everything needed to build.

**Canonical model** — The one authoritative description of the building.

**Command** — A named instruction describing a change, e.g. `MOVE_WALL`.

**Datum** — The reference point all heights are measured from. Here: ground floor finished floor level.

**Deterministic** — Same inputs always give the same output. No randomness, no AI.

**Element** — Any building object: wall, door, window, room, slab, column, beam, stair, roof.

**IFC** — The standard file format for exchanging building models between professional software.

**Interior face** — The inside surface of a wall. What room areas are measured to.

**Ledger** — An append-only record of every transaction. Balance is calculated from it.

**Level** — A storey of a building.

**Millimetre** — The unit all geometry is stored in. Always.

**Modular monolith** — One deployed application, internally divided into strict modules.

**NRM2** — RICS New Rules of Measurement 2. The UK standard for detailed measurement.

**Optimistic concurrency** — Allowing edits to proceed and rejecting them if the version has moved on.

**Parametric** — Described by measurements and rules rather than by fixed shapes.

**Provenance** — Where a piece of data came from and how much to trust it.

**QS** — Quantity Surveying. Measuring what a building needs and what it costs.

**RLS (Row-Level Security)** — Database-enforced rules restricting which rows a query can see.

**Snapshot** — A full saved copy of the building at a point in time, for fast recovery.

**Takeoff** — Extracting quantities from a design.

**Tenant** — A customer organisation. The security boundary.

**Version** — A numbered state of a building after a change. Never altered once saved.

**Waste factor** — The extra material ordered to allow for offcuts and breakage.

**Working copy** — The browser's in-memory copy of the building. Not the truth.

---
---

# Architecture Status

**FROZEN — 12 September 2026**

The following cannot be changed without a documented, approved decision that explicitly replaces the existing one:

- The canonical building model
- How versions and history are stored
- The coordinate and height system
- How rooms, openings and types work
- Where the browser's working copy lives
- The boundaries of the drawing engines
- The database as the source of truth
- The organisation as the security boundary
- Deterministic quantity and cost rules
- The AI safety model
- The credit ledger design
- Library dependency directions
- The core technology choices

Implementation detail may evolve freely within those boundaries.

Performance technologies are added on evidence, not expectation.

**Being mentioned in a roadmap does not make a technology approved.**

---

*End of document. This document is complete on its own; nothing else needs to be read to understand the architecture of Buildora AI.*
