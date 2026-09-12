# BuildWise — Sample Frontend CRUD Reference

> **File name:** `BuildWise_Sample_Frontend_CRUD.md`
>
> **Purpose:** Canonical sample frontend CRUD implementation for BuildWise.
>
> **Example feature:** `Projects`
>
> **Stack:** Next.js App Router + React + TypeScript + TanStack Query + React Hook Form + Zod + BuildWise Design System
>
> This sample follows the BuildWise frontend standards:
>
> - Server Components by default
> - Client Components only where interactivity is required
> - centralized API access
> - shared API contracts
> - consistent error handling
> - TanStack Query for client-side server state
> - React Hook Form for forms
> - no authoritative application data in Zustand
> - accessible dialogs/forms
> - reusable feature components
> - proper loading/error/empty states

---

# 1. Frontend User Flow

The sample implements:

```text
/projects
    ↓
list projects

Create Project
    ↓
POST /api/v1/projects

Open Project
    ↓
/projects/[projectId]

Edit
    ↓
PATCH /api/v1/projects/:projectId

Delete
    ↓
DELETE /api/v1/projects/:projectId
```

---

# 2. Suggested Frontend Structure

```text
apps/web/src/
├── app/
│   └── (app)/
│       └── projects/
│           ├── page.tsx
│           ├── loading.tsx
│           ├── error.tsx
│           └── [projectId]/
│               ├── page.tsx
│               ├── loading.tsx
│               ├── error.tsx
│               └── edit/
│                   └── page.tsx
│
├── features/
│   └── projects/
│       ├── api/
│       │   ├── create-project.ts
│       │   ├── delete-project.ts
│       │   ├── get-project.ts
│       │   ├── get-projects.ts
│       │   ├── update-project.ts
│       │   └── project-query-keys.ts
│       ├── components/
│       │   ├── create-project-dialog.tsx
│       │   ├── create-project-form.tsx
│       │   ├── delete-project-dialog.tsx
│       │   ├── edit-project-form.tsx
│       │   ├── project-card.tsx
│       │   ├── project-list.tsx
│       │   └── project-overview.tsx
│       └── hooks/
│           ├── use-create-project.ts
│           ├── use-delete-project.ts
│           ├── use-project.ts
│           ├── use-projects.ts
│           └── use-update-project.ts
│
└── lib/
    └── api/
        ├── api-client.ts
        ├── api-error.ts
        └── problem-details.ts
```

---

# 3. Problem Details Contract

## File: `apps/web/src/lib/api/problem-details.ts`

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

# 4. API Error Class

## File: `apps/web/src/lib/api/api-error.ts`

```ts
import type {
  ProblemDetails,
} from "./problem-details";

export class ApiError extends Error {
  constructor(
    readonly problem:
      ProblemDetails,
  ) {
    super(
      problem.detail ??
      problem.title,
    );

    this.name =
      "ApiError";
  }

  get status(): number {
    return this.problem.status;
  }

  get code():
    | string
    | undefined {
    return this.problem.code;
  }

  get traceId():
    | string
    | undefined {
    return this.problem.traceId;
  }
}
```

---

# 5. Central API Client

## File: `apps/web/src/lib/api/api-client.ts`

```ts
import {
  ApiError,
} from "./api-error";

import type {
  ProblemDetails,
} from "./problem-details";

interface ApiRequestOptions
  extends RequestInit {
  signal?: AbortSignal;
}

export async function apiRequest<T>(
  url: string,
  options: ApiRequestOptions = {},
): Promise<T> {
  const headers =
    new Headers(
      options.headers,
    );

  headers.set(
    "Accept",
    "application/json, application/problem+json",
  );

  if (
    options.body &&
    !headers.has(
      "Content-Type",
    )
  ) {
    headers.set(
      "Content-Type",
      "application/json",
    );
  }

  const response =
    await fetch(
      url,
      {
        ...options,
        headers,
      },
    );

  if (!response.ok) {
    const contentType =
      response.headers.get(
        "content-type",
      );

    if (
      contentType?.includes(
        "application/problem+json",
      )
    ) {
      const problem =
        (await response.json())
        as ProblemDetails;

      throw new ApiError(
        problem,
      );
    }

    throw new ApiError({
      type:
        "about:blank",
      title:
        "Request failed",
      status:
        response.status,
    });
  }

  if (
    response.status === 204
  ) {
    return undefined as T;
  }

  return (
    await response.json()
  ) as T;
}
```

In production this helper should also follow the chosen auth/server proxy strategy.

Do not expose backend secrets here.

---

# 6. API Base URL Helper

## File: `apps/web/src/lib/api/api-url.ts`

```ts
export function apiUrl(
  path: string,
): string {
  const baseUrl =
    process.env
      .NEXT_PUBLIC_API_URL ??
    "";

  return `${baseUrl}${path}`;
}
```

If BuildWise uses same-origin proxying, this helper can simply return:

```text
/api/v1/...
```

A server-only helper can use non-public environment variables.

---

# 7. Query Keys

## File: `apps/web/src/features/projects/api/project-query-keys.ts`

```ts
export const projectQueryKeys = {
  all:
    ["projects"] as const,

  lists: () =>
    [
      ...projectQueryKeys.all,
      "list",
    ] as const,

  list: () =>
    [
      ...projectQueryKeys.lists(),
    ] as const,

  details: () =>
    [
      ...projectQueryKeys.all,
      "detail",
    ] as const,

  detail:
    (projectId: string) =>
      [
        ...projectQueryKeys.details(),
        projectId,
      ] as const,
};
```

---

# 8. List Projects API Function

## File: `apps/web/src/features/projects/api/get-projects.ts`

```ts
import {
  projectSchema,
  type Project,
} from "@buildwise/api-contracts/projects";

import {
  apiRequest,
} from "@/lib/api/api-client";

import {
  apiUrl,
} from "@/lib/api/api-url";

export async function getProjects(
  signal?: AbortSignal,
): Promise<Project[]> {
  const response =
    await apiRequest<unknown>(
      apiUrl(
        "/api/v1/projects",
      ),
      {
        signal,
      },
    );

  return projectSchema
    .array()
    .parse(response);
}
```

---

# 9. Get Project API Function

## File: `apps/web/src/features/projects/api/get-project.ts`

```ts
import {
  projectSchema,
  type Project,
} from "@buildwise/api-contracts/projects";

import {
  apiRequest,
} from "@/lib/api/api-client";

import {
  apiUrl,
} from "@/lib/api/api-url";

export async function getProject(
  projectId: string,
  signal?: AbortSignal,
): Promise<Project> {
  const response =
    await apiRequest<unknown>(
      apiUrl(
        `/api/v1/projects/${projectId}`,
      ),
      {
        signal,
      },
    );

  return projectSchema.parse(
    response,
  );
}
```

---

# 10. Create Project API Function

## File: `apps/web/src/features/projects/api/create-project.ts`

```ts
import {
  projectSchema,
  type CreateProjectInput,
  type Project,
} from "@buildwise/api-contracts/projects";

import {
  apiRequest,
} from "@/lib/api/api-client";

import {
  apiUrl,
} from "@/lib/api/api-url";

export async function createProject(
  input: CreateProjectInput,
): Promise<Project> {
  const response =
    await apiRequest<unknown>(
      apiUrl(
        "/api/v1/projects",
      ),
      {
        method:
          "POST",

        body:
          JSON.stringify(
            input,
          ),
      },
    );

  return projectSchema.parse(
    response,
  );
}
```

---

# 11. Update Project API Function

## File: `apps/web/src/features/projects/api/update-project.ts`

```ts
import {
  projectSchema,
  type Project,
  type UpdateProjectInput,
} from "@buildwise/api-contracts/projects";

import {
  apiRequest,
} from "@/lib/api/api-client";

import {
  apiUrl,
} from "@/lib/api/api-url";

interface UpdateProjectParams {
  projectId: string;
  changes: UpdateProjectInput;
}

export async function updateProject(
  params: UpdateProjectParams,
): Promise<Project> {
  const response =
    await apiRequest<unknown>(
      apiUrl(
        `/api/v1/projects/${params.projectId}`,
      ),
      {
        method:
          "PATCH",

        body:
          JSON.stringify(
            params.changes,
          ),
      },
    );

  return projectSchema.parse(
    response,
  );
}
```

---

# 12. Delete Project API Function

## File: `apps/web/src/features/projects/api/delete-project.ts`

```ts
import {
  apiRequest,
} from "@/lib/api/api-client";

import {
  apiUrl,
} from "@/lib/api/api-url";

export async function deleteProject(
  projectId: string,
): Promise<void> {
  await apiRequest<void>(
    apiUrl(
      `/api/v1/projects/${projectId}`,
    ),
    {
      method:
        "DELETE",
    },
  );
}
```

---

# 13. TanStack Query Provider

## File: `apps/web/src/providers/query-provider.tsx`

```tsx
"use client";

import {
  QueryClient,
  QueryClientProvider,
} from "@tanstack/react-query";

import {
  useState,
  type ReactNode,
} from "react";

export function QueryProvider({
  children,
}: {
  children: ReactNode;
}) {
  const [queryClient] =
    useState(
      () =>
        new QueryClient({
          defaultOptions: {
            queries: {
              staleTime:
                30_000,

              retry:
                (failureCount, error) => {
                  if (
                    error instanceof Error &&
                    "status" in error
                  ) {
                    const status =
                      Number(
                        (
                          error as {
                            status:
                              number;
                          }
                        ).status,
                      );

                    if (
                      status >= 400 &&
                      status < 500 &&
                      status !== 429
                    ) {
                      return false;
                    }
                  }

                  return (
                    failureCount < 2
                  );
                },
            },
          },
        }),
    );

  return (
    <QueryClientProvider
      client={queryClient}
    >
      {children}
    </QueryClientProvider>
  );
}
```

Keep retry rules conservative.

Do not automatically retry validation/permission errors.

---

# 14. Root Providers

## File: `apps/web/src/providers/app-providers.tsx`

```tsx
"use client";

import type {
  ReactNode,
} from "react";

import {
  QueryProvider,
} from "./query-provider";

export function AppProviders({
  children,
}: {
  children: ReactNode;
}) {
  return (
    <QueryProvider>
      {children}
    </QueryProvider>
  );
}
```

---

# 15. Projects Query Hook

## File: `apps/web/src/features/projects/hooks/use-projects.ts`

```tsx
"use client";

import {
  useQuery,
} from "@tanstack/react-query";

import {
  getProjects,
} from "../api/get-projects";

import {
  projectQueryKeys,
} from "../api/project-query-keys";

export function useProjects() {
  return useQuery({
    queryKey:
      projectQueryKeys.list(),

    queryFn:
      ({ signal }) =>
        getProjects(
          signal,
        ),
  });
}
```

---

# 16. Project Query Hook

## File: `apps/web/src/features/projects/hooks/use-project.ts`

```tsx
"use client";

import {
  useQuery,
} from "@tanstack/react-query";

import {
  getProject,
} from "../api/get-project";

import {
  projectQueryKeys,
} from "../api/project-query-keys";

export function useProject(
  projectId: string,
) {
  return useQuery({
    queryKey:
      projectQueryKeys.detail(
        projectId,
      ),

    queryFn:
      ({ signal }) =>
        getProject(
          projectId,
          signal,
        ),

    enabled:
      Boolean(projectId),
  });
}
```

---

# 17. Create Mutation Hook

## File: `apps/web/src/features/projects/hooks/use-create-project.ts`

```tsx
"use client";

import {
  useMutation,
  useQueryClient,
} from "@tanstack/react-query";

import {
  createProject,
} from "../api/create-project";

import {
  projectQueryKeys,
} from "../api/project-query-keys";

export function useCreateProject() {
  const queryClient =
    useQueryClient();

  return useMutation({
    mutationFn:
      createProject,

    onSuccess:
      async () => {
        await queryClient
          .invalidateQueries({
            queryKey:
              projectQueryKeys.lists(),
          });
      },
  });
}
```

---

# 18. Update Mutation Hook

## File: `apps/web/src/features/projects/hooks/use-update-project.ts`

```tsx
"use client";

import {
  useMutation,
  useQueryClient,
} from "@tanstack/react-query";

import {
  updateProject,
} from "../api/update-project";

import {
  projectQueryKeys,
} from "../api/project-query-keys";

export function useUpdateProject(
  projectId: string,
) {
  const queryClient =
    useQueryClient();

  return useMutation({
    mutationFn:
      updateProject,

    onSuccess:
      async (project) => {
        queryClient.setQueryData(
          projectQueryKeys.detail(
            project.id,
          ),
          project,
        );

        await queryClient
          .invalidateQueries({
            queryKey:
              projectQueryKeys.lists(),
          });
      },
  });
}
```

---

# 19. Delete Mutation Hook

## File: `apps/web/src/features/projects/hooks/use-delete-project.ts`

```tsx
"use client";

import {
  useMutation,
  useQueryClient,
} from "@tanstack/react-query";

import {
  deleteProject,
} from "../api/delete-project";

import {
  projectQueryKeys,
} from "../api/project-query-keys";

export function useDeleteProject() {
  const queryClient =
    useQueryClient();

  return useMutation({
    mutationFn:
      deleteProject,

    onSuccess:
      async (
        _,
        projectId,
      ) => {
        queryClient.removeQueries({
          queryKey:
            projectQueryKeys.detail(
              projectId,
            ),
        });

        await queryClient
          .invalidateQueries({
            queryKey:
              projectQueryKeys.lists(),
          });
      },
  });
}
```

---

# 20. Project Card

## File: `apps/web/src/features/projects/components/project-card.tsx`

```tsx
import Link from "next/link";

import type {
  Project,
} from "@buildwise/api-contracts/projects";

import {
  Card,
  CardContent,
  CardHeader,
  CardTitle,
} from "@buildwise/design-system";

interface ProjectCardProps {
  project: Project;
}

export function ProjectCard({
  project,
}: ProjectCardProps) {
  return (
    <Card>
      <CardHeader>
        <CardTitle>
          <Link
            href={
              `/projects/${project.id}`
            }
          >
            {project.name}
          </Link>
        </CardTitle>
      </CardHeader>

      <CardContent>
        <dl>
          <div>
            <dt>Location</dt>
            <dd>
              {project.location ??
                "Not set"}
            </dd>
          </div>

          <div>
            <dt>Status</dt>
            <dd>
              {project.status}
            </dd>
          </div>
        </dl>
      </CardContent>
    </Card>
  );
}
```

---

# 21. Project List

## File: `apps/web/src/features/projects/components/project-list.tsx`

```tsx
"use client";

import {
  Alert,
  EmptyState,
  Skeleton,
} from "@buildwise/design-system";

import {
  useProjects,
} from "../hooks/use-projects";

import {
  ProjectCard,
} from "./project-card";

export function ProjectList() {
  const {
    data,
    error,
    isPending,
  } = useProjects();

  if (isPending) {
    return (
      <div
        aria-label="Loading projects"
      >
        <Skeleton />
        <Skeleton />
        <Skeleton />
      </div>
    );
  }

  if (error) {
    return (
      <Alert variant="error">
        We could not load your projects.
      </Alert>
    );
  }

  if (
    !data ||
    data.length === 0
  ) {
    return (
      <EmptyState
        title="No projects yet"
        description="Create your first BuildWise project."
      />
    );
  }

  return (
    <div
      className="
        grid
        gap-4
        md:grid-cols-2
        xl:grid-cols-3
      "
    >
      {data.map(
        (project) => (
          <ProjectCard
            key={project.id}
            project={project}
          />
        ),
      )}
    </div>
  );
}
```

For initial route rendering, server fetching may be preferable.

This client component sample demonstrates TanStack Query CRUD consistency.

---

# 22. Create Project Form

## File: `apps/web/src/features/projects/components/create-project-form.tsx`

```tsx
"use client";

import {
  zodResolver,
} from "@hookform/resolvers/zod";

import {
  useForm,
} from "react-hook-form";

import {
  createProjectSchema,
  type CreateProjectInput,
} from "@buildwise/api-contracts/projects";

import {
  Button,
  Input,
  Select,
} from "@buildwise/design-system";

import {
  ApiError,
} from "@/lib/api/api-error";

import {
  useCreateProject,
} from "../hooks/use-create-project";

interface CreateProjectFormProps {
  onSuccess?(): void;
}

export function CreateProjectForm({
  onSuccess,
}: CreateProjectFormProps) {
  const mutation =
    useCreateProject();

  const form =
    useForm<CreateProjectInput>({
      resolver:
        zodResolver(
          createProjectSchema,
        ),

      defaultValues: {
        name:
          "",

        location:
          "",

        currency:
          "GBP",

        unitSystem:
          "metric",
      },
    });

  const handleSubmit =
    form.handleSubmit(
      async (values) => {
        try {
          await mutation
            .mutateAsync(
              values,
            );

          form.reset();

          onSuccess?.();
        } catch (error) {
          if (
            error instanceof ApiError
          ) {
            form.setError(
              "root",
              {
                message:
                  error.message,
              },
            );

            return;
          }

          form.setError(
            "root",
            {
              message:
                "An unexpected error occurred.",
            },
          );
        }
      },
    );

  return (
    <form
      onSubmit={
        handleSubmit
      }
      className="space-y-4"
      noValidate
    >
      <div>
        <label
          htmlFor="project-name"
        >
          Project name
        </label>

        <Input
          id="project-name"
          autoComplete="off"
          aria-invalid={
            Boolean(
              form.formState
                .errors.name,
            )
          }
          {...form.register(
            "name",
          )}
        />

        {form.formState
          .errors.name && (
          <p role="alert">
            {
              form.formState
                .errors.name
                .message
            }
          </p>
        )}
      </div>

      <div>
        <label
          htmlFor="project-location"
        >
          Location
        </label>

        <Input
          id="project-location"
          {...form.register(
            "location",
          )}
        />
      </div>

      <div>
        <label
          htmlFor="project-currency"
        >
          Currency
        </label>

        <Select
          id="project-currency"
          {...form.register(
            "currency",
          )}
        >
          <option value="GBP">
            GBP
          </option>

          <option value="USD">
            USD
          </option>

          <option value="EUR">
            EUR
          </option>

          <option value="LKR">
            LKR
          </option>
        </Select>
      </div>

      {form.formState
        .errors.root && (
        <div
          role="alert"
          className="text-sm"
        >
          {
            form.formState
              .errors.root
              .message
          }
        </div>
      )}

      <Button
        type="submit"
        disabled={
          mutation.isPending
        }
      >
        {mutation.isPending
          ? "Creating…"
          : "Create project"}
      </Button>
    </form>
  );
}
```

The real BuildWise design-system field components should reduce repetitive label/error markup.

---

# 23. Create Project Dialog

## File: `apps/web/src/features/projects/components/create-project-dialog.tsx`

```tsx
"use client";

import {
  useState,
} from "react";

import {
  Button,
  Dialog,
  DialogContent,
  DialogHeader,
  DialogTitle,
  DialogTrigger,
} from "@buildwise/design-system";

import {
  CreateProjectForm,
} from "./create-project-form";

export function CreateProjectDialog() {
  const [open, setOpen] =
    useState(false);

  return (
    <Dialog
      open={open}
      onOpenChange={setOpen}
    >
      <DialogTrigger asChild>
        <Button>
          Create project
        </Button>
      </DialogTrigger>

      <DialogContent>
        <DialogHeader>
          <DialogTitle>
            Create project
          </DialogTitle>
        </DialogHeader>

        <CreateProjectForm
          onSuccess={() =>
            setOpen(false)
          }
        />
      </DialogContent>
    </Dialog>
  );
}
```

Accessible focus trapping/restoration should be handled by the underlying dialog primitive.

---

# 24. Projects Page

## File: `apps/web/src/app/(app)/projects/page.tsx`

```tsx
import {
  CreateProjectDialog,
} from "@/features/projects/components/create-project-dialog";

import {
  ProjectList,
} from "@/features/projects/components/project-list";

export default function ProjectsPage() {
  return (
    <main
      className="space-y-6"
    >
      <header
        className="
          flex
          items-center
          justify-between
          gap-4
        "
      >
        <div>
          <h1>
            Projects
          </h1>

          <p>
            Manage your BuildWise projects.
          </p>
        </div>

        <CreateProjectDialog />
      </header>

      <ProjectList />
    </main>
  );
}
```

This page remains a Server Component.

It composes Client Components only where needed.

---

# 25. Projects Loading Page

## File: `apps/web/src/app/(app)/projects/loading.tsx`

```tsx
import {
  Skeleton,
} from "@buildwise/design-system";

export default function Loading() {
  return (
    <div className="space-y-4">
      <Skeleton className="h-10 w-48" />

      <div
        className="
          grid
          gap-4
          md:grid-cols-2
          xl:grid-cols-3
        "
      >
        <Skeleton className="h-40" />
        <Skeleton className="h-40" />
        <Skeleton className="h-40" />
      </div>
    </div>
  );
}
```

---

# 26. Projects Error Page

## File: `apps/web/src/app/(app)/projects/error.tsx`

```tsx
"use client";

import {
  useEffect,
} from "react";

import {
  Button,
} from "@buildwise/design-system";

export default function ProjectsError({
  error,
  reset,
}: {
  error:
    Error & {
      digest?: string;
    };

  reset:
    () => void;
}) {
  useEffect(
    () => {
      // Capture with Sentry.
      void error;
    },
    [error],
  );

  return (
    <div role="alert">
      <h2>
        We could not load your projects.
      </h2>

      <p>
        Your saved data has not been changed.
      </p>

      <Button
        type="button"
        onClick={reset}
      >
        Try again
      </Button>
    </div>
  );
}
```

---

# 27. Project Overview Component

## File: `apps/web/src/features/projects/components/project-overview.tsx`

```tsx
import Link from "next/link";

import type {
  Project,
} from "@buildwise/api-contracts/projects";

import {
  Card,
  CardContent,
  CardHeader,
  CardTitle,
} from "@buildwise/design-system";

interface ProjectOverviewProps {
  project: Project;
}

export function ProjectOverview({
  project,
}: ProjectOverviewProps) {
  return (
    <div className="space-y-6">
      <header>
        <h1>
          {project.name}
        </h1>

        <p>
          {project.location ??
            "Location not set"}
        </p>
      </header>

      <div
        className="
          grid
          gap-4
          md:grid-cols-3
        "
      >
        <Card>
          <CardHeader>
            <CardTitle>
              Design
            </CardTitle>
          </CardHeader>

          <CardContent>
            <Link
              href={
                `/projects/${project.id}/2d`
              }
            >
              Open 2D Plan
            </Link>
          </CardContent>
        </Card>

        <Card>
          <CardHeader>
            <CardTitle>
              3D
            </CardTitle>
          </CardHeader>

          <CardContent>
            <Link
              href={
                `/projects/${project.id}/3d`
              }
            >
              Open 3D Model
            </Link>
          </CardContent>
        </Card>

        <Card>
          <CardHeader>
            <CardTitle>
              Commercial
            </CardTitle>
          </CardHeader>

          <CardContent>
            <Link
              href={
                `/projects/${project.id}/cost`
              }
            >
              Open Cost Estimate
            </Link>
          </CardContent>
        </Card>
      </div>
    </div>
  );
}
```

---

# 28. Server-Side Project Fetch

For project overview, prefer a server-side fetch.

## File: `apps/web/src/features/projects/api/get-project.server.ts`

```ts
import {
  projectSchema,
  type Project,
} from "@buildwise/api-contracts/projects";

import {
  ApiError,
} from "@/lib/api/api-error";

import type {
  ProblemDetails,
} from "@/lib/api/problem-details";

const API_URL =
  process.env.API_URL;

if (!API_URL) {
  throw new Error(
    "API_URL is required.",
  );
}

export async function getProjectForServer(
  projectId: string,
): Promise<Project> {
  const response =
    await fetch(
      `${API_URL}/api/v1/projects/${projectId}`,
      {
        cache:
          "no-store",

        headers: {
          // Add verified server-side auth
          // forwarding according to the
          // selected auth architecture.
        },
      },
    );

  if (!response.ok) {
    const problem =
      (await response.json())
      as ProblemDetails;

    throw new ApiError(
      problem,
    );
  }

  return projectSchema.parse(
    await response.json(),
  );
}
```

Auth forwarding is deliberately omitted because it depends on the chosen Clerk/server setup.

Never forward browser auth insecurely.

---

# 29. Project Detail Route

## File: `apps/web/src/app/(app)/projects/[projectId]/page.tsx`

```tsx
import {
  notFound,
} from "next/navigation";

import {
  ApiError,
} from "@/lib/api/api-error";

import {
  getProjectForServer,
} from "@/features/projects/api/get-project.server";

import {
  ProjectOverview,
} from "@/features/projects/components/project-overview";

interface ProjectPageProps {
  params: Promise<{
    projectId: string;
  }>;
}

export default async function ProjectPage({
  params,
}: ProjectPageProps) {
  const {
    projectId,
  } = await params;

  try {
    const project =
      await getProjectForServer(
        projectId,
      );

    return (
      <ProjectOverview
        project={project}
      />
    );
  } catch (error) {
    if (
      error instanceof ApiError &&
      error.status === 404
    ) {
      notFound();
    }

    throw error;
  }
}
```

---

# 30. Project Not Found Page

## File: `apps/web/src/app/(app)/projects/[projectId]/not-found.tsx`

```tsx
import Link from "next/link";

export default function ProjectNotFound() {
  return (
    <div>
      <h1>
        Project not found
      </h1>

      <p>
        The project does not exist or you do not have access to it.
      </p>

      <Link href="/projects">
        Back to projects
      </Link>
    </div>
  );
}
```

Do not reveal whether another tenant owns the project.

---

# 31. Edit Project Form

## File: `apps/web/src/features/projects/components/edit-project-form.tsx`

```tsx
"use client";

import {
  zodResolver,
} from "@hookform/resolvers/zod";

import {
  useForm,
} from "react-hook-form";

import type {
  Project,
  UpdateProjectInput,
} from "@buildwise/api-contracts/projects";

import {
  updateProjectSchema,
} from "@buildwise/api-contracts/projects";

import {
  Button,
  Input,
  Select,
} from "@buildwise/design-system";

import {
  ApiError,
} from "@/lib/api/api-error";

import {
  useUpdateProject,
} from "../hooks/use-update-project";

interface EditProjectFormProps {
  project: Project;
  onSuccess?(): void;
}

export function EditProjectForm({
  project,
  onSuccess,
}: EditProjectFormProps) {
  const mutation =
    useUpdateProject(
      project.id,
    );

  const form =
    useForm<UpdateProjectInput>({
      resolver:
        zodResolver(
          updateProjectSchema,
        ),

      defaultValues: {
        name:
          project.name,

        location:
          project.location,

        currency:
          project.currency,

        unitSystem:
          project.unitSystem,

        status:
          project.status,
      },
    });

  const handleSubmit =
    form.handleSubmit(
      async (changes) => {
        try {
          await mutation
            .mutateAsync({
              projectId:
                project.id,

              changes,
            });

          onSuccess?.();
        } catch (error) {
          form.setError(
            "root",
            {
              message:
                error instanceof ApiError
                  ? error.message
                  : "An unexpected error occurred.",
            },
          );
        }
      },
    );

  return (
    <form
      onSubmit={handleSubmit}
      className="space-y-4"
      noValidate
    >
      <div>
        <label
          htmlFor="edit-project-name"
        >
          Name
        </label>

        <Input
          id="edit-project-name"
          {...form.register(
            "name",
          )}
        />
      </div>

      <div>
        <label
          htmlFor="edit-project-location"
        >
          Location
        </label>

        <Input
          id="edit-project-location"
          {...form.register(
            "location",
          )}
        />
      </div>

      <div>
        <label
          htmlFor="edit-project-status"
        >
          Status
        </label>

        <Select
          id="edit-project-status"
          {...form.register(
            "status",
          )}
        >
          <option value="active">
            Active
          </option>

          <option value="archived">
            Archived
          </option>
        </Select>
      </div>

      {form.formState
        .errors.root && (
        <p role="alert">
          {
            form.formState
              .errors.root
              .message
          }
        </p>
      )}

      <Button
        type="submit"
        disabled={
          mutation.isPending
        }
      >
        {mutation.isPending
          ? "Saving…"
          : "Save changes"}
      </Button>
    </form>
  );
}
```

---

# 32. Edit Route

## File: `apps/web/src/app/(app)/projects/[projectId]/edit/page.tsx`

```tsx
import {
  getProjectForServer,
} from "@/features/projects/api/get-project.server";

import {
  EditProjectForm,
} from "@/features/projects/components/edit-project-form";

interface EditProjectPageProps {
  params: Promise<{
    projectId: string;
  }>;
}

export default async function EditProjectPage({
  params,
}: EditProjectPageProps) {
  const {
    projectId,
  } = await params;

  const project =
    await getProjectForServer(
      projectId,
    );

  return (
    <main
      className="
        mx-auto
        max-w-2xl
        space-y-6
      "
    >
      <h1>
        Edit project
      </h1>

      <EditProjectForm
        project={project}
      />
    </main>
  );
}
```

---

# 33. Delete Project Dialog

## File: `apps/web/src/features/projects/components/delete-project-dialog.tsx`

```tsx
"use client";

import {
  useRouter,
} from "next/navigation";

import {
  Alert,
  Button,
  Dialog,
  DialogContent,
  DialogDescription,
  DialogFooter,
  DialogHeader,
  DialogTitle,
  DialogTrigger,
} from "@buildwise/design-system";

import {
  ApiError,
} from "@/lib/api/api-error";

import {
  useDeleteProject,
} from "../hooks/use-delete-project";

interface DeleteProjectDialogProps {
  projectId: string;
  projectName: string;
}

export function DeleteProjectDialog({
  projectId,
  projectName,
}: DeleteProjectDialogProps) {
  const router =
    useRouter();

  const mutation =
    useDeleteProject();

  const handleDelete =
    async () => {
      try {
        await mutation
          .mutateAsync(
            projectId,
          );

        router.replace(
          "/projects",
        );

        router.refresh();
      } catch {
        // Mutation state renders the error.
      }
    };

  return (
    <Dialog>
      <DialogTrigger asChild>
        <Button
          variant="destructive"
        >
          Delete project
        </Button>
      </DialogTrigger>

      <DialogContent>
        <DialogHeader>
          <DialogTitle>
            Delete project?
          </DialogTitle>

          <DialogDescription>
            This will delete
            {" "}
            <strong>
              {projectName}
            </strong>
            .
          </DialogDescription>
        </DialogHeader>

        {mutation.error && (
          <Alert variant="error">
            {mutation.error instanceof ApiError
              ? mutation.error.message
              : "We could not delete the project."}
          </Alert>
        )}

        <DialogFooter>
          <Button
            type="button"
            variant="destructive"
            disabled={
              mutation.isPending
            }
            onClick={
              handleDelete
            }
          >
            {mutation.isPending
              ? "Deleting…"
              : "Delete permanently"}
          </Button>
        </DialogFooter>
      </DialogContent>
    </Dialog>
  );
}
```

Production BuildWise may prefer archive/retention rather than immediate permanent deletion.

---

# 34. Project Settings Actions

## File: `apps/web/src/features/projects/components/project-actions.tsx`

```tsx
import Link from "next/link";

import {
  Button,
} from "@buildwise/design-system";

import {
  DeleteProjectDialog,
} from "./delete-project-dialog";

interface ProjectActionsProps {
  projectId: string;
  projectName: string;
  canEdit: boolean;
  canDelete: boolean;
}

export function ProjectActions({
  projectId,
  projectName,
  canEdit,
  canDelete,
}: ProjectActionsProps) {
  return (
    <div
      className="
        flex
        gap-2
      "
    >
      {canEdit && (
        <Button asChild>
          <Link
            href={
              `/projects/${projectId}/edit`
            }
          >
            Edit
          </Link>
        </Button>
      )}

      {canDelete && (
        <DeleteProjectDialog
          projectId={
            projectId
          }
          projectName={
            projectName
          }
        />
      )}
    </div>
  );
}
```

The backend still enforces permission.

Frontend permission state is only for UX.

---

# 35. Root App Loading/Error Strategy

Recommended Next.js conventions:

```text
app/
├── loading.tsx
├── error.tsx
└── global-error.tsx
```

Use route-specific boundaries where a feature should fail independently.

Do not convert every error into a global crash.

---

# 36. 401 Handling

When API returns:

```text
401
```

do not repeatedly retry.

Recommended behaviour:

```text
session refresh if provider supports it
or
redirect/sign-in flow
```

Centralize this behaviour.

Do not implement 401 handling independently in every feature.

---

# 37. 403 Handling

Display:

```text
You do not have permission to perform this action.
```

Do not show:

```text
Unknown error.
```

For navigation-level access, render a dedicated forbidden surface.

---

# 38. 404 Handling

For project page:

```text
notFound()
```

For modal/inline mutations:

```text
This project is no longer available.
```

Then invalidate caches and navigate if needed.

---

# 39. 409 Conflict Handling

Critical for Building Model later.

Example:

```text
MODEL_VERSION_CONFLICT
```

UI:

```text
This project changed in another session.
Load the latest version before retrying this edit.
```

Do not automatically overwrite.

---

# 40. 422 Handling

For field errors:

```text
map to form fields
```

For domain validation:

```text
show inline domain explanation
```

Example:

```text
Wall length must be greater than 0.
```

---

# 41. 429 Handling

Differentiate:

```text
technical rate limit
```

from:

```text
AI credit limit
```

For AI credits:

```text
You have used all AI Design Credits for this billing period.
```

Offer:

- upgrade
- credit purchase

where appropriate.

---

# 42. 5xx Handling

Show safe message:

```text
We could not complete this request.
Your saved project data has not been changed.
```

Optionally show:

```text
Reference: traceId
```

Capture error telemetry.

---

# 43. Create Form Test

## File: `apps/web/src/features/projects/components/create-project-form.test.tsx`

```tsx
import {
  render,
  screen,
} from "@testing-library/react";

import userEvent
  from "@testing-library/user-event";

import {
  CreateProjectForm,
} from "./create-project-form";

describe(
  "CreateProjectForm",
  () => {
    it(
      "shows validation error when name is empty",
      async () => {
        const user =
          userEvent.setup();

        render(
          <CreateProjectForm />,
        );

        await user.click(
          screen.getByRole(
            "button",
            {
              name:
                "Create project",
            },
          ),
        );

        expect(
          await screen.findByRole(
            "alert",
          ),
        ).toHaveTextContent(
          "Project name is required",
        );
      },
    );
  },
);
```

---

# 44. Project Card Test

## File: `apps/web/src/features/projects/components/project-card.test.tsx`

```tsx
import {
  render,
  screen,
} from "@testing-library/react";

import {
  ProjectCard,
} from "./project-card";

describe(
  "ProjectCard",
  () => {
    it(
      "renders a link to the project",
      () => {
        render(
          <ProjectCard
            project={{
              id:
                "project_1",

              organizationId:
                "org_1",

              name:
                "Modern Family Home",

              location:
                "Nottingham",

              currency:
                "GBP",

              unitSystem:
                "metric",

              status:
                "active",

              createdAt:
                "2026-09-11T00:00:00.000Z",

              updatedAt:
                "2026-09-11T00:00:00.000Z",
            }}
          />,
        );

        expect(
          screen.getByRole(
            "link",
            {
              name:
                "Modern Family Home",
            },
          ),
        ).toHaveAttribute(
          "href",
          "/projects/project_1",
        );
      },
    );
  },
);
```

---

# 45. Playwright CRUD Test

## File: `apps/web/e2e/projects-crud.spec.ts`

```ts
import {
  expect,
  test,
} from "@playwright/test";

test.describe(
  "Projects CRUD",
  () => {
    test(
      "creates, edits and deletes a project",
      async ({ page }) => {
        await signInTestUser(
          page,
        );

        await page.goto(
          "/projects",
        );

        await page
          .getByRole(
            "button",
            {
              name:
                "Create project",
            },
          )
          .click();

        await page
          .getByLabel(
            "Project name",
          )
          .fill(
            "Test House",
          );

        await page
          .getByRole(
            "button",
            {
              name:
                "Create project",
            },
          )
          .click();

        await expect(
          page.getByText(
            "Test House",
          ),
        ).toBeVisible();

        await page
          .getByRole(
            "link",
            {
              name:
                "Test House",
            },
          )
          .click();

        await page
          .getByRole(
            "link",
            {
              name:
                "Edit",
            },
          )
          .click();

        await page
          .getByLabel("Name")
          .fill(
            "Updated House",
          );

        await page
          .getByRole(
            "button",
            {
              name:
                "Save changes",
            },
          )
          .click();

        await expect(
          page.getByText(
            "Updated House",
          ),
        ).toBeVisible();
      },
    );
  },
);
```

Production E2E should use a dedicated test tenant/database.

---

# 46. Reusability Pattern

A future CRUD should reuse:

```text
apiRequest
ApiError
ProblemDetails
query provider
design-system form fields
dialog
alert
empty state
loading patterns
query-key conventions
mutation conventions
```

Do not reinvent these for:

- Materials
- Suppliers
- Documents
- Assemblies

---

# 47. Material CRUD Example Mapping

The same pattern becomes:

```text
features/materials/
├── api/
├── hooks/
├── components/
└── schemas/
```

Endpoints:

```text
POST   /materials
GET    /materials
GET    /materials/:id
PATCH  /materials/:id
DELETE /materials/:id
```

Only the domain contracts change.

---

# 48. Do Not Reuse CRUD UI for Building Model Editing

A wall is not an ordinary admin CRUD record.

Do not create:

```text
Edit Wall Form
→ PATCH /walls/123
```

if the command architecture requires:

```text
model version
geometry validation
undo
derived quantity invalidation
```

The 2D editor uses the Building Model command API.

---

# 49. Server vs Client Rule Summary

Use Server Component when:

```text
read-only initial route
SEO/public
dashboard initial data
project overview
```

Use Client Component when:

```text
dialog
form
mutation
TanStack Query
PixiJS
Three.js
interactive editor
live collaboration
```

Do not use `"use client"` above the entire app.

---

# 50. Frontend CRUD Checklist

Every CRUD feature should normally have:

- [ ] shared schema
- [ ] API function
- [ ] query keys
- [ ] query hook where required
- [ ] mutation hook
- [ ] list component
- [ ] empty state
- [ ] loading state
- [ ] error state
- [ ] create form
- [ ] edit form
- [ ] delete confirmation
- [ ] route page
- [ ] accessibility
- [ ] permission-aware UI
- [ ] component tests
- [ ] Playwright coverage for critical flow
- [ ] no authoritative state duplicated in Zustand

---

# 51. CRUD Data Flow

Read:

```text
Route
 ↓
Server fetch or TanStack Query
 ↓
central API function
 ↓
typed API
 ↓
schema parse
 ↓
feature component
```

Mutation:

```text
Form
 ↓
Zod
 ↓
mutation hook
 ↓
API function
 ↓
backend
 ↓
typed response
 ↓
query cache update/invalidation
 ↓
UI updates
```

Errors:

```text
backend RFC 9457
 ↓
ApiError
 ↓
feature-specific UX
```

---

# 52. Final Frontend CRUD Rule

The reference pattern is:

```text
shared contracts
+
central API transport
+
feature-level API functions
+
query/mutation hooks
+
small reusable components
+
route-level loading/error boundaries
+
accessible forms/dialogs
```

Keep frontend state ownership clear:

```text
Backend/domain
→ authoritative project data

TanStack Query/server fetch
→ server state cache

React Hook Form
→ form state

Zustand/local state
→ ephemeral UI state

PixiJS/Three.js
→ disposable render state
```

This pattern should be used consistently throughout normal BuildWise SaaS CRUD features.
