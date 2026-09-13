# Buildora AI — Frontend Engineering Standards & Reference Implementation

> **File name:** `Buildora_AI_Frontend_Engineering_Standards.md`  
> **Audience:** Codex, Claude, human contributors, reviewers  
> **Applies to:** `apps/web`, `packages/design-system`, `packages/model-session`, `packages/cad-2d`, `packages/engine-3d`  
> **Primary stack:** Next.js App Router + React + TypeScript + Tailwind + Radix + TanStack Query + React Hook Form + Zod + Zustand (limited) + PixiJS + Three.js  
> **Status:** Canonical frontend coding standard unless superseded by an approved ADR
> **Subordinate to:** `docs/architecture/FINAL_SYSTEM_ARCHITECTURE.md` and ADR-001 … ADR-022.
> Where this document and an ADR disagree, **the ADR wins** — report the conflict.

---

# 1. Purpose

This document defines how Buildora AI frontend code should be written so the product remains consistent while supporting very different interfaces:

```text
Dashboard
Projects
2D CAD
3D BIM
Quantities
BOQ
Cost
Documents
AI Copilot
Billing
Admin
```

The frontend must be:

- reusable,
- accessible,
- strongly typed,
- predictable,
- performant,
- testable,
- visually consistent,
- safe for AI-assisted development.

This guide covers code architecture, state ownership, API usage, reusable UI, error handling, rendering engines, accessibility, performance and testing.

---

# 2. Non-Negotiable Principles

1. **Use Server Components by default.**
2. **Use Client Components only when interactivity/browser APIs require them.**
3. **The UI does not own authoritative Building Model state.**
4. **Separate server state, form state, UI state and engine state.**
5. **Use the shared design system instead of page-specific controls.**
6. **Never duplicate QS/cost calculations in React.**
7. **PixiJS and Three.js are rendering engines, not project databases.**
8. **Expected errors get specific UX; unexpected errors use boundaries/telemetry.**
9. **Accessibility is part of correctness.**
10. **Heavy editor code must not inflate unrelated SaaS routes.**
11. **Every user action must reflect real backend authorization/entitlements.**
12. **AI-generated actions are previews until validated and committed.**
13. **The editor working model lives in `packages/model-session`** — renderer-neutral, framework-free, and **not** a second authoritative database. *(ADR-008)*
14. **TanStack Query never owns working geometry. Zustand never owns the Building Model.** *(ADR-008)*
15. **`cad-2d` and `engine-3d` never import each other.** Both consume `model-session`. *(ADR-011)*
16. **Furniture is a domain element, never a Three.js object held in the scene.** *(ADR-019)*
17. **A finish change is a domain command**, not a renderer material mutation. *(ADR-020)*
18. **Saved views are presentation state** and never mutate canonical geometry.
19. **Permission-aware UI reads entitlements, never plan names.** *(ADR-021)*
20. **Client entitlement checks are advisory.** Hide, disable or badge a capability for usability — the server is the authority and will refuse an unentitled call. *(ADR-022)*
21. **Distinguish `FEATURE_NOT_ENTITLED` from `INSUFFICIENT_CREDITS`.** The first needs an upgrade CTA, the second a top-up CTA. *(ADR-022)*

---

# 3. Frontend Architecture

```text
Next.js Routes / Layouts
        ↓
Feature Components
        ↓
Feature Hooks / Services
        ↓
API Contracts / API Client
        ↓
Backend API
```

Professional workspace (ADR-008):

```text
React UI
  ├── Toolbar
  ├── Inspector
  ├── Panels
  └── AI Panel
        ↓
packages/model-session          ← the working model
  · in-memory Building Model
  · baseVersion
  · optimistic command application
  · pending command queue
  · server reconciliation
  · MODEL_VERSION_CONFLICT handling
  · subscriptions
        ↓
  ┌─────┴─────┐
  ↓           ↓
cad-2d    engine-3d             ← both subscribe to the SAME source
(PixiJS)  (Three.js)
        ↓
Derived render objects (disposable)
```

Because both renderers read one source, 2D and 3D **cannot** diverge. There is no
synchronisation code, so there is no synchronisation bug.

`packages/model-session` must not import React, Next.js, PixiJS, Three.js, NestJS,
Drizzle or OpenAI. It is testable in a plain Node process.

---

# 4. Recommended Structure

```text
apps/web/src/
├── app/
│   ├── (public)/
│   ├── (auth)/
│   ├── (app)/
│   │   ├── dashboard/
│   │   ├── projects/
│   │   └── projects/[projectId]/
│   │       ├── page.tsx
│   │       ├── 2d/
│   │       ├── 3d/
│   │       ├── quantities/
│   │       ├── boq/
│   │       ├── cost/
│   │       ├── documents/
│   │       └── reports/
│   ├── error.tsx
│   ├── global-error.tsx
│   ├── loading.tsx
│   └── not-found.tsx
├── components/
│   └── app-shell/
├── features/
│   ├── projects/
│   ├── organizations/
│   ├── documents/
│   ├── ai-copilot/
│   └── billing/
├── lib/
│   ├── api/
│   ├── auth/
│   ├── errors/
│   ├── format/
│   ├── query/
│   └── telemetry/
└── providers/

packages/
├── design-system/
├── api-contracts/
├── units/
├── building-model/
├── model-session/     ← working model (ADR-008)
├── cad-2d/
└── engine-3d/
```

---

# 5. Feature Folder Structure

Example:

```text
features/projects/
├── components/
│   ├── project-card.tsx
│   ├── project-list.tsx
│   └── create-project-form.tsx
├── hooks/
│   ├── use-project.ts
│   └── use-create-project.ts
├── api/
│   ├── get-project.ts
│   ├── create-project.ts
│   └── project-query-keys.ts
├── schemas/
├── state/
└── index.ts
```

Avoid one global `components/` folder containing every product feature.

---

# 6. Naming Standards

Files:

```text
project-card.tsx
create-project-form.tsx
use-project.ts
project-query-keys.ts
api-client.ts
```

Components:

```text
ProjectCard
CreateProjectForm
```

Hooks:

```text
useProject
useProjectPermissions
```

Handlers:

```text
handleSubmit
handleWallSelect
handleApproveAiAction
```

Booleans:

```text
isOpen
isLoading
isDisabled
hasError
canEdit
```

Avoid:

```text
thing
data2
flag
handleClick2
```

---

# 7. Server Components by Default

Use Server Components for:

- dashboard initial content,
- project overview,
- settings,
- reports list,
- subscriptions,
- static/read-heavy pages.

Benefits:

- smaller browser JS,
- server-side composition,
- streaming,
- fewer client state problems.

Do not write `"use client"` at a high-level layout merely because one nested control is interactive.

---

# 8. Client Components

Use Client Components when required for:

- `useState`,
- events,
- effects,
- browser APIs,
- drag/drop,
- PixiJS,
- Three.js,
- AI streaming UI,
- live collaboration,
- client polling,
- rich forms.

Push the Client Component boundary as low as practical.

---

# 9. React Purity

Components and hooks must be pure during render.

Do not perform during render:

- network requests,
- localStorage writes,
- DOM mutation,
- Three.js scene mutation,
- PixiJS mutation,
- random ID creation,
- mutable singleton updates.

Effects are for external synchronization.

Event handlers are for user-triggered side effects.

---

# 10. State Ownership Matrix

Before choosing a state tool, classify the state.

| State | Owner |
|---|---|
| URL filters / project ID | URL/router |
| **persisted Building Model** | **PostgreSQL (authoritative)** |
| **working model + `baseVersion` + pending commands** | **`packages/model-session`** |
| server project data, lists, QS/BOQ/cost read models | Server Component / TanStack Query |
| form fields | React Hook Form/local |
| open drawer/tab | local state/Zustand |
| selected editor tool | editor UI store (Zustand) |
| camera | Zustand (ephemeral UI) |
| hover / transient interaction | engine/local |
| authoritative walls | backend/domain |
| authoritative BOQ | backend/domain |
| placed furniture (FF&E) | **backend/domain — never the Three.js scene** |
| element finishes | **backend/domain — a command, not a material tweak** |
| saved views | backend (presentation state, versioned separately) |
| walkthrough camera, movement, collision | engine runtime (never persisted) |
| subscription | backend server state |
| AI credits | backend server state |

**Two rules that are easy to get wrong and expensive to undo:**

- **TanStack Query must not own working geometry.** Its refetch semantics are wrong
  for a working document, and per-interaction fetching is unusable at CAD frame rates.
- **Zustand must not own the Building Model.** That creates a second authority.

---

# 11. Zustand Rules

Zustand is only for appropriate ephemeral client state.

Good:

```text
active tool
sidebar state
selection UI
camera mode
snap visibility
```

Bad:

```text
full persisted Building Model
project database
BOQ totals
cost estimate
subscription authority
```

**File:** `apps/web/src/features/editor/state/editor-ui.store.ts`

```ts
import { create } from "zustand";

type EditorTool =
  | "select"
  | "wall"
  | "door"
  | "window"
  | "measure";

interface EditorUiState {
  activeTool: EditorTool;
  setActiveTool(tool: EditorTool): void;
}

export const useEditorUiStore =
  create<EditorUiState>((set) => ({
    activeTool: "select",
    setActiveTool: (activeTool) => set({ activeTool }),
  }));
```

Do not build one giant `useAppStore`.

---

# 12. API Access Must Be Centralized

Do not call `fetch()` randomly in components.

Use:

```text
lib/api/api-client.ts
features/<feature>/api/*.ts
```

Components should not manually:

- build URLs,
- parse Problem Details,
- attach headers,
- interpret HTTP statuses.

---

# 13. Problem Details Type

**File:** `apps/web/src/lib/api/problem-details.ts`

```ts
export interface ProblemDetails {
  type: string;
  title: string;
  status: number;
  detail?: string;
  instance?: string;
  code?: string;
  traceId?: string;
  errors?: Array<{
    path: string;
    message: string;
  }>;
}
```

---

# 14. API Error Class

**File:** `apps/web/src/lib/api/api-error.ts`

```ts
import type { ProblemDetails } from "./problem-details";

export class ApiError extends Error {
  constructor(readonly problem: ProblemDetails) {
    super(problem.detail ?? problem.title);
    this.name = "ApiError";
  }

  get status(): number {
    return this.problem.status;
  }

  get code(): string | undefined {
    return this.problem.code;
  }

  get traceId(): string | undefined {
    return this.problem.traceId;
  }
}
```

---

# 15. Reusable API Client

**File:** `apps/web/src/lib/api/api-client.ts`

```ts
import { ApiError } from "./api-error";
import type { ProblemDetails } from "./problem-details";

interface ApiRequestOptions extends RequestInit {
  signal?: AbortSignal;
}

export async function apiRequest<T>(
  url: string,
  options: ApiRequestOptions = {},
): Promise<T> {
  const response = await fetch(url, {
    ...options,
    headers: {
      Accept: "application/json, application/problem+json",
      "Content-Type": "application/json",
      ...options.headers,
    },
  });

  if (!response.ok) {
    const contentType = response.headers.get("content-type");

    if (contentType?.includes("application/problem+json")) {
      const problem = (await response.json()) as ProblemDetails;
      throw new ApiError(problem);
    }

    throw new ApiError({
      type: "about:blank",
      title: "Request failed",
      status: response.status,
    });
  }

  if (response.status === 204) {
    return undefined as T;
  }

  return (await response.json()) as T;
}
```

This transport helper is not enough by itself for untrusted data. Validate important responses with shared schemas.

---

# 16. Runtime-Validated API Function

**File:** `apps/web/src/features/projects/api/get-project.ts`

```ts
import { apiRequest } from "@/lib/api/api-client";
import {
  projectSchema,
  type Project,
} from "@buildora/api-contracts/projects";

export async function getProject(
  projectId: string,
  signal?: AbortSignal,
): Promise<Project> {
  const response = await apiRequest<unknown>(
    `/api/v1/projects/${projectId}`,
    { signal },
  );

  return projectSchema.parse(response);
}
```

---

# 17. Query Keys

Centralize TanStack Query keys.

**File:** `apps/web/src/features/projects/api/project-query-keys.ts`

```ts
export const projectQueryKeys = {
  all: ["projects"] as const,

  lists: () =>
    [...projectQueryKeys.all, "list"] as const,

  list: (organizationId: string) =>
    [...projectQueryKeys.lists(), organizationId] as const,

  details: () =>
    [...projectQueryKeys.all, "detail"] as const,

  detail: (projectId: string) =>
    [...projectQueryKeys.details(), projectId] as const,
};
```

Do not use arbitrary string arrays independently across components.

---

# 18. Client Query Hook

**File:** `apps/web/src/features/projects/hooks/use-project.ts`

```tsx
"use client";

import { useQuery } from "@tanstack/react-query";
import { getProject } from "../api/get-project";
import { projectQueryKeys } from "../api/project-query-keys";

export function useProject(projectId: string) {
  return useQuery({
    queryKey: projectQueryKeys.detail(projectId),
    queryFn: ({ signal }) => getProject(projectId, signal),
    staleTime: 30_000,
  });
}
```

Use TanStack Query only where client-side server state is actually useful.

---

# 19. Server-Side Route Data

For initial route data, prefer server-side composition where practical.

**File:** `apps/web/src/app/(app)/projects/[projectId]/page.tsx`

```tsx
import { ProjectOverview } from "@/features/projects/components/project-overview";
import { getProjectForServer } from "@/features/projects/api/get-project.server";

interface PageProps {
  params: Promise<{
    projectId: string;
  }>;
}

export default async function ProjectPage({ params }: PageProps) {
  const { projectId } = await params;
  const project = await getProjectForServer(projectId);

  return <ProjectOverview project={project} />;
}
```

Do not move every read into client queries solely for consistency.

---

# 20. Forms

Use:

- semantic HTML,
- React Hook Form for complex client forms,
- shared Zod schemas,
- backend validation as final authority.

Client validation is UX, not security.

---

# 21. Form Example

**File:** `apps/web/src/features/projects/components/create-project-form.tsx`

```tsx
"use client";

import { zodResolver } from "@hookform/resolvers/zod";
import { useForm } from "react-hook-form";
import {
  createProjectSchema,
  type CreateProjectInput,
} from "@buildora/api-contracts/projects";

export function CreateProjectForm() {
  const form = useForm<CreateProjectInput>({
    resolver: zodResolver(createProjectSchema),
    defaultValues: {
      name: "",
      currency: "GBP",
      unitSystem: "metric",
    },
  });

  const handleSubmit = form.handleSubmit(async (values) => {
    // Invoke a feature mutation service/hook.
  });

  return (
    <form onSubmit={handleSubmit} noValidate>
      {/* Use Buildora AI design-system form controls. */}
    </form>
  );
}
```

---

# 22. Expected vs Unexpected Errors

Expected:

- validation,
- permission,
- not found,
- version conflict,
- insufficient credits,
- unsupported file.

These get contextual UI.

Unexpected:

- runtime bug,
- unhandled rendering failure,
- unknown invariant break.

These go to:

- error boundary,
- telemetry,
- safe fallback.

---

# 23. Route Error Boundary

**File:** `apps/web/src/app/(app)/projects/[projectId]/error.tsx`

```tsx
"use client";

import { useEffect } from "react";

export default function ProjectError({
  error,
  reset,
}: {
  error: Error & { digest?: string };
  reset: () => void;
}) {
  useEffect(() => {
    // Capture through Sentry/telemetry.
  }, [error]);

  return (
    <div role="alert">
      <h2>We could not load this project.</h2>
      <p>Your saved project data has not been changed.</p>
      <button type="button" onClick={reset}>
        Try again
      </button>
    </div>
  );
}
```

---

# 24. Loading UI

**File:** `apps/web/src/app/(app)/projects/[projectId]/loading.tsx`

```tsx
import { ProjectOverviewSkeleton } from "@/features/projects/components/project-overview-skeleton";

export default function Loading() {
  return <ProjectOverviewSkeleton />;
}
```

Prefer layout-preserving skeletons to full-screen spinners.

---

# 25. Not Found

Use `not-found.tsx` for true missing resources where appropriate.

Do not route missing project and internal crash to the same generic screen.

---

# 26. User Error Messages

Good:

```text
We couldn't save this wall because the project changed in another session.
Load the latest version and try again.
```

Bad:

```text
409 Drizzle transaction conflict.
```

Show trace/reference ID when support value justifies it.

---

# 27. Toast Standard

Use toast for:

- short confirmation,
- background completion,
- recoverable non-blocking failure.

Do not use toast for:

- destructive confirmation,
- complex form errors,
- billing failures requiring action,
- major AI design approval.

Those require inline/dialog UI.

---

# 28. Design System Ownership

Reusable UI belongs in:

```text
packages/design-system
```

Examples:

```text
Button
IconButton
Input
Select
Checkbox
Radio
Switch
Tabs
Badge
Card
DataTable
Dialog
Drawer
Tooltip
Alert
Skeleton
EmptyState
ErrorState
```

Feature-specific components stay with the feature.

---

# 29. Component Variants

Prefer typed variants:

```tsx
<Button variant="primary" size="md">
  Save
</Button>
```

Avoid creating separate components for every visual combination:

```text
GreenBigButton
GreenSmallButton
RedButton2
```

---

# 30. Design Tokens

Use central tokens for:

- colours,
- typography,
- spacing,
- radius,
- shadow,
- z-index layers,
- animation duration.

Avoid repeated arbitrary values:

```text
bg-[#14A872]
mt-[13px]
```

unless intentionally documented.

---

# 31. Accessibility

Required for ordinary UI:

- keyboard operability,
- visible focus,
- labels,
- semantic buttons,
- input error associations,
- sufficient contrast,
- no colour-only meaning,
- correct table headers,
- accessible dialogs,
- correct focus restoration.

Use Radix primitives where they reduce implementation risk.

---

# 32. Icon Buttons

Bad:

```tsx
<button>
  <TrashIcon />
</button>
```

Good:

```tsx
<button aria-label="Delete wall">
  <TrashIcon aria-hidden="true" />
</button>
```

---

# 33. Editor Keyboard Commands

Maintain a central command/shortcut registry.

Examples:

```text
Esc               cancel current tool
Delete            delete selection
Cmd/Ctrl + Z      undo
Cmd/Ctrl + Shift+Z redo
Space             temporary pan when appropriate
```

Do not register the same shortcut independently across unrelated components.

---

# 34. 2D Editor Architecture

```text
React shell
  ├── Toolbar
  ├── Properties
  ├── Floor selector
  └── Status UI
        ↓
Editor controller
        ↓
PixiJS renderer
        ↓
Derived render objects
```

Model mutation path (ADR-008):

```text
user gesture
→ draft interaction              local preview only, NO command
→ gesture completes
→ semantic domain command
→ model-session (optimistic apply)
→ subscribers notified           2D AND 3D update from ONE source
→ API with baseVersion
→ server validates and commits
→ reconciliation                 ack | MODEL_VERSION_CONFLICT
```

**One completed gesture → one command → one model version.** Never one command per
pointer move.

PixiJS objects are disposable views carrying domain element IDs. Renderers never
generate replacement identities.

---

# 35. PixiJS Lifecycle

Do not recreate Pixi application on every React render.

Lifecycle:

```text
mount
→ initialize renderer
→ subscribe/bind
→ render model
→ interaction loop
→ cleanup/destroy
```

Canvas pixels are not construction coordinates.

Maintain explicit transform:

```text
model millimetres ↔ screen pixels
```

---

# 36. 3D Engine Architecture

```text
React workspace
    ↓
3D controller
    ↓
packages/model-session          ← same source as cad-2d
    ↓
wall/slab/window adapters
    ↓
Three.js scene (WebGL2 — ADR-017)
    ↓
derived meshes
```

Three.js objects do not become the Building Model.

**Direct Three.js, not React Three Fiber** for the editor (ADR-017). Adopting R3F
for the core editor requires a superseding ADR.

**WebGL2 is the baseline.** WebGPU is deferred.

Adapters resolve absolute height at render time —
`absoluteZ = level.elevationMm + (baseOffsetMm ?? 0)` (ADR-005). Absolute z is
never read from the element, which is why moving a level requires no element writes.

---

# 37. Three.js Resource Disposal

Dispose resources when removed/replaced:

- geometry,
- material,
- texture,
- render targets,
- renderer where required.

3D memory leaks are production defects.

---

# 38. 3D Adapter Example

**File:** `packages/engine-3d/src/adapters/wall-mesh.adapter.ts`

```ts
import * as THREE from "three";

interface WallViewModel {
  id: string;
  lengthMm: number;
  heightMm: number;
  thicknessMm: number;
}

const MM_PER_WORLD_UNIT = 1000;

export function createWallMesh(
  wall: WallViewModel,
): THREE.Mesh {
  const geometry = new THREE.BoxGeometry(
    wall.lengthMm / MM_PER_WORLD_UNIT,
    wall.heightMm / MM_PER_WORLD_UNIT,
    wall.thicknessMm / MM_PER_WORLD_UNIT,
  );

  const material = new THREE.MeshStandardMaterial();
  const mesh = new THREE.Mesh(geometry, material);

  mesh.userData = {
    elementId: wall.id,
  };

  return mesh;
}
```

This is rendering code only.

---

# 39. High-Frequency Interaction

Do not cause full React tree re-renders on:

- pointer move,
- camera orbit,
- hover,
- drag preview.

High-frequency interaction belongs in the engine/controller.

Commit meaningful model changes at explicit command boundaries such as drag-end.

---

# 40. Web Workers

Consider Web Workers for measured expensive browser tasks:

- geometry,
- parsing,
- indexing,
- large derived calculations.

Do not add worker complexity without a real performance reason.

---

# 41. Dynamic Imports

Heavy browser-only editor modules may be loaded dynamically.

```tsx
import dynamic from "next/dynamic";

const EditorWorkspace = dynamic(
  () =>
    import("./editor-workspace").then(
      (module) => module.EditorWorkspace,
    ),
  {
    ssr: false,
    loading: () => <EditorWorkspaceSkeleton />,
  },
);
```

Use `ssr: false` only where browser-only libraries require it.

Do not make ordinary dashboard components client-only.

---

# 42. Rendering Read Models

Map large API/domain data into renderer-specific read models.

```text
Building Model DTO
    ↓
WallRenderModel[]
    ↓
Pixi / Three adapters
```

Read models are derived/disposable.

They are not a second authoritative project model.

---

# 43. Model Version Awareness

The editor must know the current model version.

Every mutation includes:

```text
baseVersion
```

On:

```text
MODEL_VERSION_CONFLICT
```

UI must:

- stop committing stale update,
- fetch latest state,
- preserve user intent if safe,
- offer retry/reapply where supported.

Never silently overwrite.

> ## Contract gate CG-2 — must be closed before optimistic persistence ships
>
> "Reload and replay, **or** surface the conflict" is a **product policy**, not an
> implementation detail. It must be specified before the editor persists anything.
>
> The hard case: commands 1–30 are queued optimistically and command 7 fails
> server-side. Commands 8–30 were computed from a state that *included* 7.
> Dropping 7 and replaying the suffix may invalidate all of them.
>
> The following must be defined and tested first:
>
> - accepted-prefix processing,
> - per-command preconditions,
> - dependency/cascade metadata,
> - deterministic rebase rules,
> - quarantine of an invalid suffix,
> - conflict UX,
> - **durable local persistence of the pending queue** — otherwise "reload" loses
>   unsent user work, and the claim that session state is always discardable is false.
>
> **Undo is a new command against the current `baseVersion`, with preview — never a
> blind inverse payload replayed at a later version.**

---

# 44. Optimistic UI

Use optimistic updates only when rollback is reliable.

**Geometry is the exception that proves the rule.** The editor *must* apply
commands optimistically to stay interactive — but only through `model-session`,
which owns the rollback and reconciliation protocol (CG-2 above). Do not hand-roll
optimistic geometry updates in components.

Be conservative for:

- BOQ,
- cost,
- billing.

These have no interactive-latency requirement, so a draft/preview state is safer
than pretending success before the backend commits.

For high-impact operations, a draft/preview state is safer than pretending success before the backend commits.

---

# 45. Autosave

Autosave states:

```text
Saving…
Saved
Save failed
Conflict
```

Do not send one API model command per mousemove.

Use explicit editor command boundaries and sensible debounce/batching where allowed.

---

# 46. AI UI Components

Reuse a standard set:

```text
AiPromptBox
AiMessage
AiToolProgress
AiActionProposal
AiImpactSummary
AiApprovalCard
AiErrorState
```

AI mutation preview must show:

- requested change,
- affected elements,
- quantity impact,
- cost impact,
- warnings,
- approve/reject.

---

# 47. AI Context

Pass structured context.

Example:

```json
{
  "projectId": "project_123",
  "workspace": "2d",
  "selectedElementIds": ["wall_123"],
  "modelVersion": 42
}
```

Avoid vague hidden context like:

```text
that wall on the left
```

---

# 48. AI Streaming

Streaming can improve perceived speed.

But:

- partial prose is not authoritative,
- tool progress is structured,
- Apply remains disabled until validation/impact finishes,
- final commit result comes from backend.

---

# 49. AI Error UX

Examples:

```text
AI provider unavailable
→ retry / continue without AI

Insufficient AI credits
→ plan/credit CTA

Tool failed
→ explain no project change was applied

Geometry validation failed
→ show correction needed
```

Never say a change succeeded when commit failed.

---

# 50. Complex Workflow States

Prefer explicit unions/state machines over many booleans.

**File:** `apps/web/src/features/plan-import/state/recognition-state.ts`

```ts
import type { ApiError } from "@/lib/api/api-error";

export type RecognitionState =
  | { status: "idle" }
  | { status: "uploading"; progress: number }
  | { status: "processing"; jobId: string }
  | { status: "needs-review"; resultId: string }
  | { status: "converting"; resultId: string }
  | { status: "complete"; projectId: string }
  | { status: "failed"; error: ApiError };
```

This prevents impossible combinations such as:

```text
isUploading = true
isDone = true
hasError = true
```

---

# 51. Tables

Use shared table primitives for:

- quantities,
- BOQ,
- cost,
- materials,
- documents,
- admin.

Common capabilities:

- sorting,
- filtering,
- selection,
- sticky headers,
- loading,
- empty state,
- pagination/virtualization when needed.

Do not implement separate table interaction rules per page.

---

# 52. BOQ Table

BOQ may require:

- hierarchical sections,
- expandable source trace,
- numeric alignment,
- currency formatting,
- unit columns,
- keyboard access.

The table displays calculations.

It does not perform authoritative calculation.

---

# 53. Money Formatting

**File:** `apps/web/src/lib/format/format-money.ts`

```ts
export function formatMoney(
  amount: string | number,
  currency: string,
  locale = "en-GB",
): string {
  return new Intl.NumberFormat(locale, {
    style: "currency",
    currency,
  }).format(Number(amount));
}
```

This is display-only.

Never parse the formatted value back into authoritative financial calculations.

---

# 54. Measurement Formatting

Centralize:

```text
formatLength
formatArea
formatVolume
```

Input:

```text
canonical value + user preference
```

Do not scatter unit conversion formulas through components.

---

# 55. Date Formatting

Use `Intl.DateTimeFormat`.

Backend timestamps remain UTC.

Frontend chooses presentation based on user/project locale/timezone.

---

# 56. Empty States

Every major list/workspace must have an intentional empty state.

Example:

```text
No projects yet.
Create a project from an AI brief, uploaded plan or blank drawing.
```

An empty table with no explanation is incomplete UX.

---

# 57. Loading States

Use:

- route skeleton,
- component skeleton,
- button pending state,
- long-running progress/job status.

Do not block the entire application for one small request.

---

# 58. Long-Running Jobs

Use explicit job states:

```text
Queued
Processing
Needs review
Completed
Failed
```

Examples:

- plan recognition,
- IFC conversion,
- rendering,
- report generation.

Users should be able to navigate away where appropriate.

---

# 59. Upload UX

Show:

- allowed formats,
- maximum size,
- upload progress,
- validation status,
- processing status,
- retry,
- cancel if supported.

Do not show upload success before backend/storage confirmation.

---

# 60. Route Structure

Use route groups for major application surfaces.

Example:

```text
(public)
(auth)
(app)
```

Project workspaces live under:

```text
/projects/[projectId]/...
```

---

# 61. Layout Reuse

Use layouts for stable chrome:

- authenticated app shell,
- project shell,
- workspace shell.

Do not duplicate sidebar/header markup across pages.

---

# 62. Error Boundary Granularity

A small chart failure should not necessarily crash the whole project workspace.

Use feature boundaries where recovery adds value.

But do not wrap every tiny component in an error boundary.

---

# 63. Suspense

Use Suspense/loading boundaries for independent slow server-rendered sections where appropriate.

Fallbacks should maintain layout stability.

Do not use Suspense indiscriminately.

---

# 64. Accessibility in Canvas Workspaces

Canvas-based design tools still require accessible surrounding controls.

Provide:

- keyboard-accessible toolbar,
- accessible property inspector,
- textual selected-object state,
- clear labels,
- keyboard commands.

Full screen-reader geometry authoring may be a later specialized effort, but ordinary SaaS controls must remain accessible from day one.

---

# 65. Focus Management

Focus should be predictable after:

- dialog open/close,
- destructive action,
- inspector changes,
- error recovery.

Accessible primitives should manage focus where possible.

---

# 66. Destructive Actions

Confirm:

- permanent project deletion,
- irreversible document deletion,
- subscription cancellation where appropriate.

Do not confirm every normal wall deletion if undo safely handles it.

---

# 67. Permission-Aware UI

UI should reflect permission for usability.

```tsx
{permissions.canEditProject ? (
  <Button>Edit</Button>
) : null}
```

But backend remains the security authority.

---

# 68. Feature Flags

Use typed feature flag access.

Avoid:

```ts
process.env.NEXT_PUBLIC_FEATURE_X === "true"
```

inside many components.

Use a central abstraction or server-provided entitlement/flag state.

---

# 69. Subscription Entitlements

Frontend may receive capabilities such as:

```text
canUseIfc
canUseProQs
aiCreditsRemaining
renderCreditsRemaining
```

Do not reproduce full subscription rules client-side.

---

# 70. Frontend Security

Never expose:

- AI secret key,
- DB credentials,
- Stripe secret key,
- R2 secret,
- auth server secret.

Assume all `NEXT_PUBLIC_*` values are public.

---

# 71. XSS / HTML

Avoid `dangerouslySetInnerHTML`.

Treat as untrusted:

- AI content,
- document content,
- user text.

If rich HTML is unavoidable, sanitize with an explicitly reviewed allowlist.

Do not render arbitrary AI HTML.

---

# 72. Telemetry

Capture meaningful failures:

- route crash,
- editor crash,
- model load failure,
- upload failure,
- AI failure,
- WebGL context loss where relevant.

Avoid sending confidential project contents to telemetry by default.

---

# 73. Product Analytics

Use a typed event taxonomy.

Example:

```ts
export type ProductEvent =
  | {
      name: "project_created";
      projectId: string;
    }
  | {
      name: "three_d_opened";
      projectId: string;
    };
```

Do not invent analytics strings in random components.

---

# 74. Performance Standards

Measure:

- JS bundle,
- route load,
- Core Web Vitals,
- 2D FPS,
- 3D FPS,
- model load,
- browser memory,
- long tasks.

Heavy CAD/BIM libraries must not load on marketing/auth routes.

---

# 75. Code Splitting

Keep:

- PixiJS,
- Three.js,
- BIM packages,

out of routes that do not use them.

Use route-level/dynamic loading where beneficial.

---

# 76. Memoization

Do not add `useMemo` and `useCallback` everywhere.

Start with:

- pure components,
- stable data ownership,
- correct engine boundaries.

Memoize when measured or required by a high-frequency integration.

---

# 77. Effects

Before writing `useEffect`, ask:

```text
Can this be derived in render?
Can this happen in an event handler?
Can this happen on the server?
```

Effects synchronize with external systems.

They should not become a general state-management mechanism.

---

# 78. Custom Hooks

Good hooks:

```text
useProjectPermissions
useUploadPlan
useEditorKeyboardShortcuts
```

Avoid:

```text
useEverything
```

A custom hook should have a coherent responsibility.

---

# 79. Component API Design

Good:

```tsx
<ProjectCard
  project={project}
  canEdit={canEdit}
  onOpen={handleOpen}
/>
```

Bad:

```tsx
<Card mode={7} thing={data} />
```

Prefer semantic props.

---

# 80. Storybook

Stories should cover reusable states:

```text
default
hover
focus
disabled
loading
error
empty
long text
small viewport
```

Priority Buildora AI stories:

- confidence badge,
- AI action card,
- property row,
- BOQ row,
- quantity row,
- editor tool button,
- loading/error states.

---

# 81. Testing Strategy

## Unit

- formatters,
- state reducers,
- API/error utilities,
- render adapters where practical.

## Component

- user behaviour,
- form states,
- permission display,
- errors.

## Storybook interactions

For reusable complex components.

## E2E

Playwright for critical workflows.

## Visual regression

For stable editor/workspace views.

---

# 82. Component Test Example

**File:** `apps/web/src/features/projects/components/project-card.test.tsx`

```tsx
import { render, screen } from "@testing-library/react";
import { ProjectCard } from "./project-card";

describe("ProjectCard", () => {
  it("shows the project name", () => {
    render(
      <ProjectCard
        project={{
          id: "project_1",
          name: "Modern Family Home",
        }}
        canEdit={true}
        onOpen={() => undefined}
      />,
    );

    expect(
      screen.getByText("Modern Family Home"),
    ).toBeInTheDocument();
  });
});
```

Test behaviour, not internal state implementation.

---

# 83. Critical E2E Flow

```text
sign in
→ create project
→ open blank plan
→ draw four walls
→ add door
→ switch to 3D
→ save
→ reload
→ verify same building
```

Later extend:

```text
quantities
→ BOQ
→ cost
→ AI
```

---

# 84. Error Path Tests

Explicitly test:

```text
401 session expired
403 forbidden
404 missing project
409 model conflict
422 validation
429 rate limit
503 dependency unavailable
```

Do not test only happy paths.

---

# 85. Model Conflict UX

For `MODEL_VERSION_CONFLICT`:

```text
This project changed in another session.
```

Actions:

- load latest,
- reapply where safe,
- compare later.

Never silently overwrite newer work.

---

# 86. Validation UX

Map field-level 422 errors to form fields.

Use form/global error only for form-wide failures.

Do not show every validation failure as a toast.

---

# 87. Rate / Credit Limit UX

Differentiate:

```text
technical rate limit
```

from:

```text
subscription/credit limit
```

The second may need plan/credit CTA.

The first may need retry time.

---

# 88. Connectivity

MVP does not require full offline editing.

But handle:

- timeout,
- transient failure,
- unsaved draft warning,
- retry.

Do not claim “Saved” until the backend confirms the commit.

---

# 89. Derived Frontend Data

Frontend may derive display data such as:

- sorted rows,
- selected label,
- visible floor list,
- UI grouping.

Frontend must not independently implement:

- authoritative block quantities,
- cost totals,
- VAT,
- model geometry truth.

---

# 90. Formatting vs Calculation

Frontend:

```text
formats £12,345.67
```

Backend/domain:

```text
calculates exact 12345.67 GBP
```

Do not reverse these responsibilities.

---

# 91. Public Package Boundaries

Prefer:

```ts
import { Button } from "@buildora/design-system";
```

Avoid deep private imports:

```ts
import { x } from "../../../../packages/design-system/src/private/internal";
```

Expose deliberate package public APIs.

---

# 92. Barrel Exports

Use `index.ts` selectively at package/feature public boundaries.

Avoid deep barrel chains causing:

- circular dependencies,
- hidden ownership,
- bundle issues.

---

# 93. Circular Dependencies

Treat circular dependencies as architecture defects.

Use dependency checks as the codebase grows.

Do not fix cycles by moving unrelated code into a global `utils` folder.

---

# 94. Component File Size

There is no magical line limit, but large files are a smell.

Split when a component has multiple responsibilities such as:

- data fetching,
- form state,
- table logic,
- dialogs,
- chart configuration,
- business rules

all in one file.

---

# 95. Comments

Comments explain:

- why,
- assumptions,
- tricky browser/engine constraints,
- non-obvious performance decisions.

Do not narrate JSX.

---

# 96. TODO Standard

Good:

```ts
// TODO(BW-712): virtualize BOQ rows when datasets exceed the beta threshold.
```

Bad:

```ts
// TODO improve
```

---

# 97. Workspace Layout Standard

Professional workspace should remain consistent:

```text
┌──────────────────────────────────────────────────────┐
│ Project Header / Save / Share / AI                  │
├──────────┬─────────────────────────────┬─────────────┤
│ Tools    │ Main Workspace              │ Inspector   │
│          │                             │             │
├──────────┴─────────────────────────────┴─────────────┤
│ 2D | 3D | Quantities | BOQ | Cost                  │
└──────────────────────────────────────────────────────┘
```

Do not redesign each professional screen from scratch.

---

# 98. Responsive Strategy

Priority:

1. desktop,
2. laptop,
3. tablet,
4. mobile companion.

Do not destroy desktop CAD usability merely to force full editing onto narrow phones.

---

# 99. Browser Logging

Do not leave:

```ts
console.log(project);
console.log(response);
```

in production, especially with private project data.

Use controlled telemetry.

---

# 100. Environment Variables

Only safe public values use:

```text
NEXT_PUBLIC_*
```

Assume those values are visible to everyone.

Secrets stay server-side.

---

# 101. CI Frontend Gate

Every frontend PR must pass:

```text
lint
typecheck
unit/component tests
build
```

Important UI changes additionally require browser regression/E2E.

---

# 102. Frontend Anti-Patterns — Forbidden

Do not normalize these patterns:

```text
"use client" everywhere
fetch scattered in UI components
one giant Zustand store
authoritative Building Model in Zustand
working geometry owned by TanStack Query
cad-2d importing engine-3d (or the reverse)
model-session importing React/Pixi/Three
optimistic geometry rollback hand-rolled in a component
undo implemented as a blind inverse payload
furniture held only in a Three.js scene
finish changed by mutating a material instead of issuing a command
UI gated on a plan name instead of an entitlement
authoritative cost calculations in React
Three.js meshes persisted as domain state
Pixi objects persisted as domain state
permission enforced only by hidden buttons
copy/pasted modal implementations
page-specific arbitrary design tokens
dangerouslySetInnerHTML for AI output
console.log production debugging
manual API error parsing in every component
random query key strings
useEffect for ordinary derived state
memoization everywhere without measurement
800-line route components
```

---

# 103. Frontend Pull Request Checklist

Before merge:

- [ ] correct Server/Client boundary
- [ ] no duplicate authoritative state
- [ ] working model accessed through model-session, not Query/Zustand
- [ ] renderer packages do not import each other
- [ ] design-system primitives reused
- [ ] API access centralized
- [ ] runtime response validation where needed
- [ ] expected errors handled
- [ ] loading state
- [ ] empty state
- [ ] accessibility
- [ ] keyboard behaviour
- [ ] permissions reflected correctly
- [ ] no secret/public-env mistake
- [ ] no authoritative calculations duplicated
- [ ] engine resources cleaned up
- [ ] performance impact considered
- [ ] tests added
- [ ] responsive behaviour checked
- [ ] browser regression for major UI
- [ ] build passes

---

# 104. References

Keep implementation aligned with current official guidance:

- Next.js Server and Client Components  
  https://nextjs.org/docs/app/getting-started/server-and-client-components
- Next.js Error Handling  
  https://nextjs.org/docs/app/getting-started/error-handling
- Next.js File Conventions  
  https://nextjs.org/docs/app/api-reference/file-conventions
- Next.js Forms  
  https://nextjs.org/docs/app/guides/forms
- React Rules  
  https://react.dev/reference/rules
- React Components and Hooks Must Be Pure  
  https://react.dev/reference/rules/components-and-hooks-must-be-pure
- Tailwind CSS  
  https://tailwindcss.com/docs
- Radix UI  
  https://www.radix-ui.com/primitives
- TanStack Query  
  https://tanstack.com/query/latest
- Storybook  
  https://storybook.js.org/docs
- Playwright  
  https://playwright.dev/docs/intro
- PixiJS  
  https://pixijs.com/
- Three.js  
  https://threejs.org/

---

# 105. Final Frontend Rule

When several implementations are possible, prefer the one that preserves:

```text
Correct state ownership
→ Clarity
→ Accessibility
→ Reusability
→ Predictability
→ Performance
→ Visual consistency
```

The frontend is consistent when:

> **the user experiences one coherent Buildora AI product, while server state, form state, UI state, domain state and rendering-engine state remain clearly separated and every layer owns only what it should.**
