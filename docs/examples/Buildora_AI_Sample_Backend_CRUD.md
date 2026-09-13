# Buildora AI — Sample Backend CRUD Reference

> **File name:** `Buildora_AI_Sample_Backend_CRUD.md`
>
> **Purpose:** Canonical sample CRUD implementation for Buildora AI backend development.
>
> **Example feature:** `Projects`
>
> **Stack:** NestJS + TypeScript + Zod + PostgreSQL + Drizzle ORM
>
> This sample follows the Buildora AI backend standards:
>
> - thin controllers
> - validated external input
> - application use cases
> - repository ports/adapters
> - organization/tenant scoping
> - explicit error codes
> - RFC 9457 Problem Details
> - structured architecture
> - reusable contracts
> - testability
> - no business logic in controllers
>
> Use this sample as a pattern for future CRUD modules such as:
>
> - materials
> - assemblies
> - documents
> - suppliers
> - rate libraries
> - project members
> - report templates
>
> Do **not** use simple CRUD patterns blindly for complex Building Model mutations. Building Model edits should use commands/versioning.

---

# 1. Feature Goal

Implement the following API:

```text
POST   /api/v1/projects
GET    /api/v1/projects
GET    /api/v1/projects/:projectId
PATCH  /api/v1/projects/:projectId
DELETE /api/v1/projects/:projectId
```

The API must be multi-tenant.

A user may only access projects belonging to their current organization.

---

# 2. Suggested Backend File Structure

```text
packages/
└── api-contracts/
    └── src/
        └── projects/
            ├── project.schema.ts
            ├── create-project.schema.ts
            ├── update-project.schema.ts
            └── index.ts

apps/
└── api/
    └── src/
        ├── common/
        │   ├── auth/
        │   │   └── actor.ts
        │   ├── errors/
        │   │   ├── app-error.ts
        │   │   ├── forbidden.error.ts
        │   │   ├── not-found.error.ts
        │   │   └── problem-details.filter.ts
        │   └── validation/
        │       └── zod-validation.pipe.ts
        ├── db/
        │   ├── client.ts
        │   └── schema/
        │       └── projects.ts
        └── modules/
            └── projects/
                ├── projects.module.ts
                ├── projects.controller.ts
                ├── errors/
                │   └── project-not-found.error.ts
                ├── application/
                │   ├── create-project.use-case.ts
                │   ├── list-projects.use-case.ts
                │   ├── get-project.use-case.ts
                │   ├── update-project.use-case.ts
                │   └── delete-project.use-case.ts
                ├── ports/
                │   └── project.repository.ts
                ├── policies/
                │   └── project-access.policy.ts
                └── infrastructure/
                    └── project.drizzle-repository.ts
```

---

# 3. Shared Project Contract

## File: `packages/api-contracts/src/projects/project.schema.ts`

```ts
import { z } from "zod";

export const projectStatusSchema = z.enum([
  "active",
  "archived",
]);

export const projectSchema = z.object({
  id: z.string().uuid(),

  // Display/support reference only. Never an FK target (ADR-004).
  publicId: z
    .string()
    .regex(/^[A-Z]{3}-[2-9A-HJ-NP-Z]{6}$/),

  organizationId: z.string().uuid(),
  name: z.string().min(1).max(120),
  location: z.string().max(250).nullable(),
  currency: z.enum(["GBP", "USD", "EUR", "LKR"]),
  unitSystem: z.enum(["metric", "imperial"]),
  status: projectStatusSchema,
  createdAt: z.string().datetime(),
  updatedAt: z.string().datetime(),
});

export type Project = z.infer<typeof projectSchema>;
```

The API contract deliberately uses ISO strings for timestamps.

Do not expose database `Date` objects directly as a shared HTTP contract.

---

# 4. Create Project Contract

## File: `packages/api-contracts/src/projects/create-project.schema.ts`

```ts
import { z } from "zod";

export const createProjectSchema = z.object({
  name: z
    .string()
    .trim()
    .min(1, "Project name is required.")
    .max(120, "Project name must be 120 characters or fewer."),

  location: z
    .string()
    .trim()
    .max(250, "Location must be 250 characters or fewer.")
    .optional(),

  currency: z
    .enum(["GBP", "USD", "EUR", "LKR"])
    .default("GBP"),

  unitSystem: z
    .enum(["metric", "imperial"])
    .default("metric"),
});

export type CreateProjectInput =
  z.infer<typeof createProjectSchema>;
```

---

# 5. Update Project Contract

## File: `packages/api-contracts/src/projects/update-project.schema.ts`

```ts
import { z } from "zod";

export const updateProjectSchema = z
  .object({
    name: z
      .string()
      .trim()
      .min(1)
      .max(120)
      .optional(),

    location: z
      .string()
      .trim()
      .max(250)
      .nullable()
      .optional(),

    currency: z
      .enum(["GBP", "USD", "EUR", "LKR"])
      .optional(),

    unitSystem: z
      .enum(["metric", "imperial"])
      .optional(),

    status: z
      .enum(["active", "archived"])
      .optional(),
  })
  .refine(
    (value) => Object.keys(value).length > 0,
    {
      message: "At least one field must be provided.",
    },
  );

export type UpdateProjectInput =
  z.infer<typeof updateProjectSchema>;
```

---

# 6. Contracts Barrel Export

## File: `packages/api-contracts/src/projects/index.ts`

```ts
export * from "./project.schema";
export * from "./create-project.schema";
export * from "./update-project.schema";
```

---

# 7. PostgreSQL / Drizzle Schema

## File: `apps/api/src/db/schema/projects.ts`

> **ID strategy (locked — ADR-004).**
> Primary keys are native PostgreSQL `uuid`, generated in the application as
> UUIDv7 where a stable implementation is available (UUIDv4 is an acceptable
> fallback). Human-facing entities additionally carry a `public_id`
> (for example `PRJ-8K4D2A`) used only for display, URLs and support
> reference. **`public_id` is never a foreign-key target.**
>
> Do not copy prefixed `varchar` primary keys into new code.

```ts
import {
  pgEnum,
  pgTable,
  text,
  timestamp,
  uuid,
  varchar,
} from "drizzle-orm/pg-core";

export const projectStatusEnum =
  pgEnum("project_status", [
    "active",
    "archived",
  ]);

export const projects = pgTable("projects", {
  id: uuid("id").primaryKey(),

  // Display/support reference only. Never a foreign-key target.
  publicId: varchar("public_id", {
    length: 32,
  }).notNull(),

  organizationId: uuid(
    "organization_id",
  ).notNull(),

  createdByUserId: uuid(
    "created_by_user_id",
  ).notNull(),

  name: varchar("name", {
    length: 120,
  }).notNull(),

  location: varchar("location", {
    length: 250,
  }),

  currency: varchar("currency", {
    length: 3,
  }).notNull(),

  unitSystem: varchar("unit_system", {
    length: 16,
  }).notNull(),

  status: projectStatusEnum("status")
    .notNull()
    .default("active"),

  createdAt: timestamp(
    "created_at",
    { withTimezone: true },
  ).notNull().defaultNow(),

  updatedAt: timestamp(
    "updated_at",
    { withTimezone: true },
  ).notNull().defaultNow(),
});
```

Production schema should also add appropriate:

- foreign keys,
- indexes,
- organization relationships,
- project search indexes.

---

# 8. Database Client

## File: `apps/api/src/db/client.ts`

```ts
import { Pool } from "pg";
import { drizzle } from "drizzle-orm/node-postgres";

const connectionString =
  process.env.DATABASE_URL;

if (!connectionString) {
  throw new Error(
    "DATABASE_URL is required.",
  );
}

export const pgPool = new Pool({
  connectionString,
  max: 10,
});

export const db = drizzle({
  client: pgPool,
});
```

In the real Buildora AI application, environment values should be validated centrally during startup rather than validated independently here.

---

# 9. Authenticated Actor Type

## File: `apps/api/src/common/auth/actor.ts`

```ts
export type OrganizationRole =
  | "owner"
  | "admin"
  | "member"
  | "viewer";

export interface Actor {
  userId: string;
  organizationId: string;
  role: OrganizationRole;
}
```

The `organizationId` should come from verified authentication/session context.

Never trust an arbitrary organization ID from request body/query as the tenant context.

---

# 10. Request User Typing

## File: `apps/api/src/types/express.d.ts`

```ts
import type { Actor } from "@/common/auth/actor";

declare global {
  namespace Express {
    interface Request {
      actor: Actor;
    }
  }
}

export {};
```

Your authentication guard/middleware should populate:

```ts
request.actor
```

only after successful identity and organization verification.

---

# 11. Base Application Error

## File: `apps/api/src/common/errors/app-error.ts`

```ts
export interface AppErrorOptions {
  code: string;
  title: string;
  status: number;
  detail?: string;
  exposeDetail?: boolean;
  cause?: unknown;
}

export class AppError extends Error {
  readonly code: string;
  readonly title: string;
  readonly status: number;
  readonly exposeDetail: boolean;

  constructor(options: AppErrorOptions) {
    super(
      options.detail ?? options.title,
      { cause: options.cause },
    );

    this.name = this.constructor.name;
    this.code = options.code;
    this.title = options.title;
    this.status = options.status;
    this.exposeDetail =
      options.exposeDetail ??
      options.status < 500;
  }
}
```

---

# 12. Not Found Error

## File: `apps/api/src/common/errors/not-found.error.ts`

```ts
import { AppError } from "./app-error";

export class NotFoundError extends AppError {
  constructor(input: {
    code: string;
    title: string;
    detail?: string;
  }) {
    super({
      ...input,
      status: 404,
    });
  }
}
```

---

# 13. Forbidden Error

## File: `apps/api/src/common/errors/forbidden.error.ts`

```ts
import { AppError } from "./app-error";

export class ForbiddenError extends AppError {
  constructor(
    code: string,
    detail: string,
  ) {
    super({
      code,
      title: "Forbidden",
      detail,
      status: 403,
    });
  }
}
```

---

# 14. Project Not Found Error

## File: `apps/api/src/modules/projects/errors/project-not-found.error.ts`

```ts
import { NotFoundError } from "@/common/errors/not-found.error";

export class ProjectNotFoundError extends NotFoundError {
  constructor() {
    super({
      code: "PROJECT_NOT_FOUND",
      title: "Project not found",
      detail:
        "The requested project does not exist or is not accessible.",
    });
  }
}
```

Using the same not-found response for inaccessible tenant data avoids leaking whether another organization's project exists.

---

# 15. RFC 9457 Problem Details Filter

## File: `apps/api/src/common/errors/problem-details.filter.ts`

```ts
import {
  ArgumentsHost,
  Catch,
  ExceptionFilter,
  HttpException,
  HttpStatus,
} from "@nestjs/common";

import type {
  Request,
  Response,
} from "express";

import { AppError } from "./app-error";

@Catch()
export class ProblemDetailsFilter
  implements ExceptionFilter
{
  catch(
    exception: unknown,
    host: ArgumentsHost,
  ): void {
    const http =
      host.switchToHttp();

    const request =
      http.getRequest<Request>();

    const response =
      http.getResponse<Response>();

    const traceId =
      request.headers[
        "x-request-id"
      ]?.toString() ?? "unknown";

    if (exception instanceof AppError) {
      response
        .status(exception.status)
        .type(
          "application/problem+json",
        )
        .json({
          type:
            `https://buildora.example/problems/` +
            exception.code
              .toLowerCase()
              .replaceAll("_", "-"),

          title: exception.title,
          status: exception.status,

          detail:
            exception.exposeDetail
              ? exception.message
              : undefined,

          instance:
            request.originalUrl,

          code:
            exception.code,

          traceId,
        });

      return;
    }

    if (
      exception instanceof HttpException
    ) {
      const status =
        exception.getStatus();

      response
        .status(status)
        .type(
          "application/problem+json",
        )
        .json({
          type: "about:blank",
          title:
            HttpStatus[status] ??
            "HTTP error",
          status,
          detail:
            status < 500
              ? "The request could not be completed."
              : undefined,
          instance:
            request.originalUrl,
          code: "HTTP_ERROR",
          traceId,
        });

      return;
    }

    // Production:
    // capture exception using structured logger
    // + Sentry/OpenTelemetry.

    response
      .status(500)
      .type(
        "application/problem+json",
      )
      .json({
        type:
          "https://buildora.example/problems/internal-error",
        title:
          "Internal server error",
        status: 500,
        detail:
          "An unexpected error occurred.",
        instance:
          request.originalUrl,
        code:
          "INTERNAL_ERROR",
        traceId,
      });
  }
}
```

---

# 16. Reusable Zod Validation Pipe

## File: `apps/api/src/common/validation/zod-validation.pipe.ts`

```ts
import {
  Injectable,
  PipeTransform,
} from "@nestjs/common";

import type {
  ZodType,
} from "zod";

import { AppError } from "@/common/errors/app-error";

@Injectable()
export class ZodValidationPipe<T>
  implements PipeTransform
{
  constructor(
    private readonly schema:
      ZodType<T>,
  ) {}

  transform(value: unknown): T {
    const result =
      this.schema.safeParse(value);

    if (result.success) {
      return result.data;
    }

    throw new AppError({
      code:
        "VALIDATION_FAILED",
      title:
        "Validation failed",
      detail:
        "One or more request fields are invalid.",
      status: 422,
    });
  }
}
```

A production version may attach structured field errors to Problem Details.

---

# 17. Project Repository Port

## File: `apps/api/src/modules/projects/ports/project.repository.ts`

```ts
export interface ProjectRecord {
  id: string;
  publicId: string;
  organizationId: string;
  createdByUserId: string;
  name: string;
  location: string | null;
  currency: string;
  unitSystem:
    | "metric"
    | "imperial";
  status:
    | "active"
    | "archived";
  createdAt: Date;
  updatedAt: Date;
}

export interface CreateProjectRecord {
  id: string;
  publicId: string;
  organizationId: string;
  createdByUserId: string;
  name: string;
  location?: string;
  currency: string;
  unitSystem:
    | "metric"
    | "imperial";
}

export interface UpdateProjectRecord {
  name?: string;
  location?: string | null;
  currency?: string;
  unitSystem?:
    | "metric"
    | "imperial";
  status?:
    | "active"
    | "archived";
}

export abstract class ProjectRepository {
  abstract create(
    input: CreateProjectRecord,
  ): Promise<ProjectRecord>;

  abstract listByOrganization(
    organizationId: string,
  ): Promise<ProjectRecord[]>;

  abstract findById(
    organizationId: string,
    projectId: string,
  ): Promise<ProjectRecord | null>;

  abstract update(
    organizationId: string,
    projectId: string,
    input: UpdateProjectRecord,
  ): Promise<ProjectRecord | null>;

  abstract delete(
    organizationId: string,
    projectId: string,
  ): Promise<boolean>;
}
```

Notice every tenant-sensitive method requires:

```text
organizationId
```

---

# 18. ID Generator Port

For portability/testability, generate IDs through an abstraction.

Two distinct concerns, deliberately kept separate (ADR-004):

```text
generateId()            → uuid primary key
generatePublicId(prefix) → human-facing reference, never an FK target
```

## File: `apps/api/src/common/id/id-generator.ts`

```ts
export abstract class IdGenerator {
  /** Primary key. UUIDv7 where available, else UUIDv4. */
  abstract generateId(): string;

  /**
   * Human-facing reference such as `PRJ-8K4D2A`.
   * Display/support only — never a foreign-key target.
   */
  abstract generatePublicId(
    prefix: string,
  ): string;
}
```

## File: `apps/api/src/common/id/crypto-id-generator.ts`

```ts
import {
  randomUUID,
} from "node:crypto";

import {
  Injectable,
} from "@nestjs/common";

import {
  IdGenerator,
} from "./id-generator";

// Unambiguous alphabet: no I, O, 0, 1.
const PUBLIC_ID_ALPHABET =
  "23456789ABCDEFGHJKLMNPQRSTUVWXYZ";

@Injectable()
export class CryptoIdGenerator
  extends IdGenerator
{
  generateId(): string {
    // Swap for a UUIDv7 implementation when one is
    // standardized in the project; the port does not change.
    return randomUUID();
  }

  generatePublicId(
    prefix: string,
  ): string {
    const bytes =
      new Uint8Array(6);

    crypto.getRandomValues(bytes);

    const body = Array.from(bytes)
      .map(
        (b) =>
          PUBLIC_ID_ALPHABET[
            b % PUBLIC_ID_ALPHABET.length
          ],
      )
      .join("");

    return `${prefix}-${body}`;
  }
}
```

> Uniqueness of `public_id` is enforced by a database unique index,
> not by generation alone. Retry on conflict.

---

# 19. Drizzle Repository Implementation

## File: `apps/api/src/modules/projects/infrastructure/project.drizzle-repository.ts`

```ts
import {
  Injectable,
} from "@nestjs/common";

import {
  and,
  desc,
  eq,
} from "drizzle-orm";

import {
  db,
} from "@/db/client";

import {
  projects,
} from "@/db/schema/projects";

import {
  ProjectRepository,
  type CreateProjectRecord,
  type ProjectRecord,
  type UpdateProjectRecord,
} from "../ports/project.repository";

@Injectable()
export class ProjectDrizzleRepository
  extends ProjectRepository
{
  async create(
    input: CreateProjectRecord,
  ): Promise<ProjectRecord> {
    const [row] = await db
      .insert(projects)
      .values({
        id:
          input.id,
        publicId:
          input.publicId,
        organizationId:
          input.organizationId,
        createdByUserId:
          input.createdByUserId,
        name:
          input.name,
        location:
          input.location,
        currency:
          input.currency,
        unitSystem:
          input.unitSystem,
      })
      .returning();

    if (!row) {
      throw new Error(
        "Project insert returned no row.",
      );
    }

    return row;
  }

  async listByOrganization(
    organizationId: string,
  ): Promise<ProjectRecord[]> {
    return db
      .select()
      .from(projects)
      .where(
        eq(
          projects.organizationId,
          organizationId,
        ),
      )
      .orderBy(
        desc(projects.updatedAt),
      );
  }

  async findById(
    organizationId: string,
    projectId: string,
  ): Promise<ProjectRecord | null> {
    const [row] = await db
      .select()
      .from(projects)
      .where(
        and(
          eq(
            projects.organizationId,
            organizationId,
          ),
          eq(
            projects.id,
            projectId,
          ),
        ),
      )
      .limit(1);

    return row ?? null;
  }

  async update(
    organizationId: string,
    projectId: string,
    input: UpdateProjectRecord,
  ): Promise<ProjectRecord | null> {
    const [row] = await db
      .update(projects)
      .set({
        ...input,
        updatedAt:
          new Date(),
      })
      .where(
        and(
          eq(
            projects.organizationId,
            organizationId,
          ),
          eq(
            projects.id,
            projectId,
          ),
        ),
      )
      .returning();

    return row ?? null;
  }

  async delete(
    organizationId: string,
    projectId: string,
  ): Promise<boolean> {
    const rows = await db
      .delete(projects)
      .where(
        and(
          eq(
            projects.organizationId,
            organizationId,
          ),
          eq(
            projects.id,
            projectId,
          ),
        ),
      )
      .returning({
        id:
          projects.id,
      });

    return rows.length > 0;
  }
}
```

---

# 20. Access Policy

## File: `apps/api/src/modules/projects/policies/project-access.policy.ts`

```ts
import {
  Injectable,
} from "@nestjs/common";

import {
  ForbiddenError,
} from "@/common/errors/forbidden.error";

import type {
  Actor,
} from "@/common/auth/actor";

@Injectable()
export class ProjectAccessPolicy {
  requireCanCreate(
    actor: Actor,
  ): void {
    if (
      actor.role === "viewer"
    ) {
      throw new ForbiddenError(
        "PROJECT_CREATE_FORBIDDEN",
        "You do not have permission to create projects.",
      );
    }
  }

  requireCanEdit(
    actor: Actor,
  ): void {
    if (
      actor.role === "viewer"
    ) {
      throw new ForbiddenError(
        "PROJECT_EDIT_FORBIDDEN",
        "You do not have permission to edit this project.",
      );
    }
  }

  requireCanDelete(
    actor: Actor,
  ): void {
    if (
      actor.role !== "owner" &&
      actor.role !== "admin"
    ) {
      throw new ForbiddenError(
        "PROJECT_DELETE_FORBIDDEN",
        "You do not have permission to delete this project.",
      );
    }
  }
}
```

---

# 21. Project Response Mapper

## File: `apps/api/src/modules/projects/mappers/project-response.mapper.ts`

```ts
import type {
  Project,
} from "@buildora/api-contracts/projects";

import type {
  ProjectRecord,
} from "../ports/project.repository";

export function toProjectResponse(
  project: ProjectRecord,
): Project {
  return {
    id:
      project.id,

    publicId:
      project.publicId,

    organizationId:
      project.organizationId,

    name:
      project.name,

    location:
      project.location,

    currency:
      project.currency as Project["currency"],

    unitSystem:
      project.unitSystem,

    status:
      project.status,

    createdAt:
      project.createdAt.toISOString(),

    updatedAt:
      project.updatedAt.toISOString(),
  };
}
```

Prefer stronger DB typing for currency in the final schema so casting is unnecessary.

---

# 22. Create Project Use Case

## File: `apps/api/src/modules/projects/application/create-project.use-case.ts`

```ts
import {
  Injectable,
} from "@nestjs/common";

import type {
  CreateProjectInput,
  Project,
} from "@buildora/api-contracts/projects";

import type {
  Actor,
} from "@/common/auth/actor";

import {
  IdGenerator,
} from "@/common/id/id-generator";

import {
  ProjectRepository,
} from "../ports/project.repository";

import {
  ProjectAccessPolicy,
} from "../policies/project-access.policy";

import {
  toProjectResponse,
} from "../mappers/project-response.mapper";

interface CreateProjectCommand {
  actor: Actor;
  input: CreateProjectInput;
}

@Injectable()
export class CreateProjectUseCase {
  constructor(
    private readonly projects:
      ProjectRepository,

    private readonly access:
      ProjectAccessPolicy,

    private readonly ids:
      IdGenerator,
  ) {}

  async execute(
    command: CreateProjectCommand,
  ): Promise<Project> {
    this.access.requireCanCreate(
      command.actor,
    );

    const project =
      await this.projects.create({
        id:
          this.ids.generateId(),

        publicId:
          this.ids.generatePublicId("PRJ"),

        organizationId:
          command.actor.organizationId,

        createdByUserId:
          command.actor.userId,

        name:
          command.input.name,

        location:
          command.input.location,

        currency:
          command.input.currency,

        unitSystem:
          command.input.unitSystem,
      });

    return toProjectResponse(
      project,
    );
  }
}
```

In production add:

```text
AuditService.record(project.created)
```

after successful creation, preferably in the same application workflow.

---

# 23. List Projects Use Case

## File: `apps/api/src/modules/projects/application/list-projects.use-case.ts`

```ts
import {
  Injectable,
} from "@nestjs/common";

import type {
  Project,
} from "@buildora/api-contracts/projects";

import type {
  Actor,
} from "@/common/auth/actor";

import {
  ProjectRepository,
} from "../ports/project.repository";

import {
  toProjectResponse,
} from "../mappers/project-response.mapper";

@Injectable()
export class ListProjectsUseCase {
  constructor(
    private readonly projects:
      ProjectRepository,
  ) {}

  async execute(
    actor: Actor,
  ): Promise<Project[]> {
    const rows =
      await this.projects
        .listByOrganization(
          actor.organizationId,
        );

    return rows.map(
      toProjectResponse,
    );
  }
}
```

---

# 24. Get Project Use Case

## File: `apps/api/src/modules/projects/application/get-project.use-case.ts`

```ts
import {
  Injectable,
} from "@nestjs/common";

import type {
  Project,
} from "@buildora/api-contracts/projects";

import type {
  Actor,
} from "@/common/auth/actor";

import {
  ProjectRepository,
} from "../ports/project.repository";

import {
  ProjectNotFoundError,
} from "../errors/project-not-found.error";

import {
  toProjectResponse,
} from "../mappers/project-response.mapper";

@Injectable()
export class GetProjectUseCase {
  constructor(
    private readonly projects:
      ProjectRepository,
  ) {}

  async execute(input: {
    actor: Actor;
    projectId: string;
  }): Promise<Project> {
    const project =
      await this.projects.findById(
        input.actor.organizationId,
        input.projectId,
      );

    if (!project) {
      throw new ProjectNotFoundError();
    }

    return toProjectResponse(
      project,
    );
  }
}
```

---

# 25. Update Project Use Case

## File: `apps/api/src/modules/projects/application/update-project.use-case.ts`

```ts
import {
  Injectable,
} from "@nestjs/common";

import type {
  Project,
  UpdateProjectInput,
} from "@buildora/api-contracts/projects";

import type {
  Actor,
} from "@/common/auth/actor";

import {
  ProjectRepository,
} from "../ports/project.repository";

import {
  ProjectAccessPolicy,
} from "../policies/project-access.policy";

import {
  ProjectNotFoundError,
} from "../errors/project-not-found.error";

import {
  toProjectResponse,
} from "../mappers/project-response.mapper";

@Injectable()
export class UpdateProjectUseCase {
  constructor(
    private readonly projects:
      ProjectRepository,

    private readonly access:
      ProjectAccessPolicy,
  ) {}

  async execute(input: {
    actor: Actor;
    projectId: string;
    changes: UpdateProjectInput;
  }): Promise<Project> {
    this.access.requireCanEdit(
      input.actor,
    );

    const updated =
      await this.projects.update(
        input.actor.organizationId,
        input.projectId,
        input.changes,
      );

    if (!updated) {
      throw new ProjectNotFoundError();
    }

    return toProjectResponse(
      updated,
    );
  }
}
```

---

# 26. Delete Project Use Case

## File: `apps/api/src/modules/projects/application/delete-project.use-case.ts`

```ts
import {
  Injectable,
} from "@nestjs/common";

import type {
  Actor,
} from "@/common/auth/actor";

import {
  ProjectRepository,
} from "../ports/project.repository";

import {
  ProjectAccessPolicy,
} from "../policies/project-access.policy";

import {
  ProjectNotFoundError,
} from "../errors/project-not-found.error";

@Injectable()
export class DeleteProjectUseCase {
  constructor(
    private readonly projects:
      ProjectRepository,

    private readonly access:
      ProjectAccessPolicy,
  ) {}

  async execute(input: {
    actor: Actor;
    projectId: string;
  }): Promise<void> {
    this.access.requireCanDelete(
      input.actor,
    );

    const deleted =
      await this.projects.delete(
        input.actor.organizationId,
        input.projectId,
      );

    if (!deleted) {
      throw new ProjectNotFoundError();
    }
  }
}
```

For Buildora AI production, project deletion may eventually become:

```text
archive
→ retention period
→ asynchronous permanent deletion
```

rather than immediate hard deletion.

This sample uses direct delete to demonstrate CRUD.

---

# 27. Projects Controller

## File: `apps/api/src/modules/projects/projects.controller.ts`

```ts
import {
  Body,
  Controller,
  Delete,
  Get,
  HttpCode,
  Param,
  Patch,
  Post,
  Req,
  UsePipes,
} from "@nestjs/common";

import type {
  Request,
} from "express";

import {
  createProjectSchema,
  updateProjectSchema,
  type CreateProjectInput,
  type UpdateProjectInput,
} from "@buildora/api-contracts/projects";

import {
  ZodValidationPipe,
} from "@/common/validation/zod-validation.pipe";

import {
  CreateProjectUseCase,
} from "./application/create-project.use-case";

import {
  ListProjectsUseCase,
} from "./application/list-projects.use-case";

import {
  GetProjectUseCase,
} from "./application/get-project.use-case";

import {
  UpdateProjectUseCase,
} from "./application/update-project.use-case";

import {
  DeleteProjectUseCase,
} from "./application/delete-project.use-case";

@Controller("projects")
export class ProjectsController {
  constructor(
    private readonly createProject:
      CreateProjectUseCase,

    private readonly listProjects:
      ListProjectsUseCase,

    private readonly getProject:
      GetProjectUseCase,

    private readonly updateProject:
      UpdateProjectUseCase,

    private readonly deleteProject:
      DeleteProjectUseCase,
  ) {}

  @Post()
  async create(
    @Req() request: Request,

    @Body(
      new ZodValidationPipe(
        createProjectSchema,
      ),
    )
    input: CreateProjectInput,
  ) {
    return this.createProject.execute({
      actor:
        request.actor,

      input,
    });
  }

  @Get()
  async list(
    @Req() request: Request,
  ) {
    return this.listProjects.execute(
      request.actor,
    );
  }

  @Get(":projectId")
  async get(
    @Req() request: Request,

    @Param("projectId")
    projectId: string,
  ) {
    return this.getProject.execute({
      actor:
        request.actor,

      projectId,
    });
  }

  @Patch(":projectId")
  async update(
    @Req() request: Request,

    @Param("projectId")
    projectId: string,

    @Body(
      new ZodValidationPipe(
        updateProjectSchema,
      ),
    )
    changes: UpdateProjectInput,
  ) {
    return this.updateProject.execute({
      actor:
        request.actor,

      projectId,

      changes,
    });
  }

  @Delete(":projectId")
  @HttpCode(204)
  async delete(
    @Req() request: Request,

    @Param("projectId")
    projectId: string,
  ): Promise<void> {
    await this.deleteProject.execute({
      actor:
        request.actor,

      projectId,
    });
  }
}
```

Authentication guard omitted here for clarity.

The real module should attach the authenticated actor via a global or route-level guard.

---

# 28. Projects Module

## File: `apps/api/src/modules/projects/projects.module.ts`

```ts
import {
  Module,
} from "@nestjs/common";

import {
  ProjectsController,
} from "./projects.controller";

import {
  CreateProjectUseCase,
} from "./application/create-project.use-case";

import {
  ListProjectsUseCase,
} from "./application/list-projects.use-case";

import {
  GetProjectUseCase,
} from "./application/get-project.use-case";

import {
  UpdateProjectUseCase,
} from "./application/update-project.use-case";

import {
  DeleteProjectUseCase,
} from "./application/delete-project.use-case";

import {
  ProjectAccessPolicy,
} from "./policies/project-access.policy";

import {
  ProjectRepository,
} from "./ports/project.repository";

import {
  ProjectDrizzleRepository,
} from "./infrastructure/project.drizzle-repository";

import {
  IdGenerator,
} from "@/common/id/id-generator";

import {
  CryptoIdGenerator,
} from "@/common/id/crypto-id-generator";

@Module({
  controllers: [
    ProjectsController,
  ],

  providers: [
    CreateProjectUseCase,
    ListProjectsUseCase,
    GetProjectUseCase,
    UpdateProjectUseCase,
    DeleteProjectUseCase,

    ProjectAccessPolicy,

    {
      provide:
        ProjectRepository,

      useClass:
        ProjectDrizzleRepository,
    },

    {
      provide:
        IdGenerator,

      useClass:
        CryptoIdGenerator,
    },
  ],
})
export class ProjectsModule {}
```

---

# 29. App Module

## File: `apps/api/src/app.module.ts`

```ts
import {
  Module,
} from "@nestjs/common";

import {
  ProjectsModule,
} from "./modules/projects/projects.module";

@Module({
  imports: [
    ProjectsModule,
  ],
})
export class AppModule {}
```

---

# 30. Main Bootstrap

## File: `apps/api/src/main.ts`

```ts
import {
  NestFactory,
} from "@nestjs/core";

import {
  AppModule,
} from "./app.module";

import {
  ProblemDetailsFilter,
} from "./common/errors/problem-details.filter";

async function bootstrap(): Promise<void> {
  const app =
    await NestFactory.create(
      AppModule,
      {
        bufferLogs: true,
      },
    );

  app.setGlobalPrefix(
    "api/v1",
  );

  app.enableShutdownHooks();

  app.useGlobalFilters(
    new ProblemDetailsFilter(),
  );

  await app.listen(
    process.env.PORT ?? 4000,
  );
}

void bootstrap();
```

---

# 31. Example API Calls

## Create

```http
POST /api/v1/projects
Content-Type: application/json
```

```json
{
  "name": "Modern Family Home",
  "location": "Nottingham",
  "currency": "GBP",
  "unitSystem": "metric"
}
```

Response:

```http
201 Created
```

```json
{
  "id": "019a3f2c-8d41-7c3e-9b02-4f7a1c6e5d88",
  "publicId": "PRJ-8K4D2A",
  "organizationId": "019a3f2c-1111-7000-8000-000000000001",
  "name": "Modern Family Home",
  "location": "Nottingham",
  "currency": "GBP",
  "unitSystem": "metric",
  "status": "active",
  "createdAt": "2026-09-11T00:00:00.000Z",
  "updatedAt": "2026-09-11T00:00:00.000Z"
}
```

---

# 32. List

```http
GET /api/v1/projects
```

Response:

```json
[
  {
    "id": "019a3f2c-8d41-7c3e-9b02-4f7a1c6e5d88",
    "publicId": "PRJ-8K4D2A",
    "organizationId": "019a3f2c-1111-7000-8000-000000000001",
    "name": "Modern Family Home",
    "location": "Nottingham",
    "currency": "GBP",
    "unitSystem": "metric",
    "status": "active",
    "createdAt": "2026-09-11T00:00:00.000Z",
    "updatedAt": "2026-09-11T00:00:00.000Z"
  }
]
```

For large project counts, replace this with cursor pagination.

---

# 33. Read

```http
GET /api/v1/projects/019a3f2c-8d41-7c3e-9b02-4f7a1c6e5d88
```

If inaccessible:

```http
404 Not Found
Content-Type: application/problem+json
```

```json
{
  "type": "https://buildora.example/problems/project-not-found",
  "title": "Project not found",
  "status": 404,
  "detail": "The requested project does not exist or is not accessible.",
  "code": "PROJECT_NOT_FOUND",
  "traceId": "..."
}
```

---

# 34. Update

```http
PATCH /api/v1/projects/019a3f2c-8d41-7c3e-9b02-4f7a1c6e5d88
Content-Type: application/json
```

```json
{
  "name": "Modern Family Home V2",
  "status": "archived"
}
```

Response:

```http
200 OK
```

with updated resource.

---

# 35. Delete

```http
DELETE /api/v1/projects/019a3f2c-8d41-7c3e-9b02-4f7a1c6e5d88
```

Response:

```http
204 No Content
```

---

# 36. Unit Test — Create Use Case

## File: `apps/api/src/modules/projects/application/create-project.use-case.spec.ts`

```ts
import {
  describe,
  expect,
  it,
  vi,
} from "vitest";

import {
  CreateProjectUseCase,
} from "./create-project.use-case";

describe(
  "CreateProjectUseCase",
  () => {
    it(
      "creates a project inside the actor organization",
      async () => {
        const projects = {
          create:
            vi.fn().mockResolvedValue({
              id:
                "019a3f2c-0000-7000-8000-000000000001",

              publicId:
                "PRJ-8K4D2A",

              organizationId:
                "org_1",

              createdByUserId:
                "user_1",

              name:
                "House",

              location:
                null,

              currency:
                "GBP",

              unitSystem:
                "metric",

              status:
                "active",

              createdAt:
                new Date(
                  "2026-09-11T00:00:00Z",
                ),

              updatedAt:
                new Date(
                  "2026-09-11T00:00:00Z",
                ),
            }),
        };

        const access = {
          requireCanCreate:
            vi.fn(),
        };

        const ids = {
          generateId:
            vi.fn()
              .mockReturnValue(
                "019a3f2c-0000-7000-8000-000000000001",
              ),

          generatePublicId:
            vi.fn()
              .mockReturnValue(
                "PRJ-8K4D2A",
              ),
        };

        const useCase =
          new CreateProjectUseCase(
            projects as never,
            access as never,
            ids as never,
          );

        const result =
          await useCase.execute({
            actor: {
              userId:
                "user_1",

              organizationId:
                "org_1",

              role:
                "owner",
            },

            input: {
              name:
                "House",

              currency:
                "GBP",

              unitSystem:
                "metric",
            },
          });

        expect(
          projects.create,
        ).toHaveBeenCalledWith(
          expect.objectContaining({
            organizationId:
              "org_1",

            createdByUserId:
              "user_1",
          }),
        );

        expect(
          result.id,
        ).toBe(
          "019a3f2c-0000-7000-8000-000000000001",
        );

        expect(
          result.publicId,
        ).toBe("PRJ-8K4D2A");
      },
    );
  },
);
```

---

# 37. Integration Test — Tenant Isolation

## File: `apps/api/test/projects/project-tenant-isolation.e2e-spec.ts`

```ts
describe(
  "Project tenant isolation",
  () => {
    it(
      "does not allow organization B to read organization A project",
      async () => {
        const project =
          await createProjectForOrganization(
            "org_a",
          );

        const response =
          await request(app.getHttpServer())
            .get(
              `/api/v1/projects/${project.id}`,
            )
            .set(
              "Authorization",
              tokenFor("org_b"),
            );

        expect(
          response.status,
        ).toBe(404);

        expect(
          response.body.code,
        ).toBe(
          "PROJECT_NOT_FOUND",
        );
      },
    );
  },
);
```

Cross-tenant tests are mandatory for SaaS resources.

---

# 38. Update Test Cases

Test:

```text
owner can update
admin can update
member can update
viewer cannot update
other organization gets 404
empty PATCH gets 422
invalid currency gets 422
```

---

# 39. Delete Test Cases

Test:

```text
owner can delete
admin can delete
member cannot delete
viewer cannot delete
other tenant receives 404
deleting missing project returns 404
```

---

# 40. Production Improvements to Add

This sample intentionally focuses on the CRUD architecture.

The production Projects module should additionally implement:

- audit events,
- cursor pagination,
- project archive/retention workflow,
- project search,
- observability spans,
- structured logging,
- metrics,
- idempotency where needed,
- soft/hard deletion policy,
- background deletion of project assets,
- database foreign keys,
- indexes,
- OpenAPI annotations,
- Clerk/auth guard integration,
- organization membership verification.

---

# 41. Pattern to Reuse for Other Simple CRUDs

Use this same structure for ordinary CRUD domains:

```text
shared schema
↓
controller
↓
use case
↓
policy
↓
repository port
↓
repository implementation
↓
database
```

Examples:

```text
materials
suppliers
rate libraries
project notes
document categories
report templates
```

---

# 42. Do Not Use This CRUD Pattern for Building Model Commands

Do not implement:

```text
PATCH /walls/:id
```

as a simple Drizzle update if wall mutation must:

- check model version,
- validate geometry,
- update relationships,
- invalidate quantities,
- create audit record,
- support undo/redo.

Instead use:

```text
POST /projects/:projectId/model/commands
```

with a versioned command.

## The shape it takes instead

This is **not** a CRUD endpoint. It is a command endpoint with optimistic
concurrency, an idempotency key, and an atomic cascade.

### Request

```http
POST /api/v1/projects/:projectId/model/commands
Content-Type: application/json
Idempotency-Key: 6f1c8e2a-...        ← REQUIRED (ADR-002, §30)
```

```json
{
  "baseVersion": 42,
  "command": {
    "type": "MOVE_WALL",
    "commandSchemaVersion": 1,
    "wallId": "019a3f2c-...",
    "endXMm": 6400,
    "endYMm": 4500
  }
}
```

### Contract

```ts
// packages/api-contracts/src/model/move-wall.schema.ts
import { z } from "zod";

export const moveWallCommandSchema = z.object({
  type: z.literal("MOVE_WALL"),
  commandSchemaVersion: z.literal(1),
  wallId: z.string().uuid(),

  // Millimetres. Never screen pixels (ADR — units).
  endXMm: z.number().int(),
  endYMm: z.number().int(),
});

export const submitCommandSchema = z.object({
  baseVersion: z.number().int().nonnegative(),
  command: z.discriminatedUnion("type", [
    moveWallCommandSchema,
    // …CREATE_WALL, CREATE_DOOR, DELETE_ELEMENT
  ]),
});
```

### Use case — what makes it different from CRUD

```ts
@Injectable()
export class SubmitModelCommandUseCase {
  async execute(input: SubmitCommandInput): Promise<CommitResult> {
    this.access.requireCanEditModel(input.actor);

    // Replay a retried request rather than reapplying it (§30).
    const replayed = await this.idempotency.find(
      input.actor.organizationId,
      input.idempotencyKey,
    );
    if (replayed) return replayed;

    return this.db.transaction(async (tx) => {
      // Tenant context is transaction-local (ADR-016).
      await this.tenant.bind(tx, input.actor.organizationId);

      // Lock the project row, then verify the version.
      const current = await this.model
        .withTx(tx)
        .lockProjectVersion(input.actor.organizationId, input.projectId);

      if (current.version !== input.baseVersion) {
        throw new ModelVersionConflictError(current.version);
      }

      // Domain decides — this is framework-free code in packages/building-model.
      const result = applyCommand(current.model, input.command);
      if (!result.ok) throw new InvalidGeometryError(result.errors);

      // The cascade commits atomically WITH the command:
      //   hosted openings · bounded rooms · junction neighbours
      const version = await this.model.withTx(tx).commit({
        organizationId: input.actor.organizationId,
        projectId: input.projectId,
        baseVersion: input.baseVersion,
        command: input.command,
        changeItems: result.changeItems,
      });

      await this.audit.withTx(tx).record({ /* … */ });
      await this.idempotency.withTx(tx).store(input.idempotencyKey, version);

      return version;
    });
  }
}
```

### Conflict response

```http
409 Conflict
Content-Type: application/problem+json
```

```json
{
  "type": "https://buildora.example/problems/model-version-conflict",
  "title": "Model version conflict",
  "status": 409,
  "detail": "The project changed after this edit was started.",
  "code": "MODEL_VERSION_CONFLICT",
  "currentVersion": 43,
  "traceId": "01J..."
}
```

### What CRUD does not give you

| Requirement | Plain `PATCH /walls/:id` | Command endpoint |
|---|---|---|
| Reject stale writes | ✗ last write wins | ✓ `baseVersion` check |
| Safe retry | ✗ applies twice | ✓ idempotency key |
| Atomic cascade | ✗ partial update possible | ✓ one transaction |
| Undo / audit | ✗ no record of intent | ✓ command journal |
| Derived invalidation | ✗ silent | ✓ change items drive it |
| AI proposal path | ✗ none | ✓ same pipeline (ADR-013) |

**The AI mutation path ends here too.** An approved ChangeSet is committed through
this exact use case — AI never gets a separate write path.

---

# 43. Backend CRUD Checklist

For every future CRUD:

- [ ] shared request schema
- [ ] shared response schema
- [ ] thin controller
- [ ] application use case
- [ ] policy/authorization
- [ ] tenant scope
- [ ] repository port
- [ ] repository adapter
- [ ] stable errors
- [ ] RFC 9457 error response
- [ ] tests
- [ ] database migration
- [ ] audit events where required
- [ ] no provider SDK leakage
- [ ] no raw SQL in controllers
- [ ] no unvalidated request data

---

# 44. Final Backend CRUD Flow

```text
Browser
  ↓
HTTP API
  ↓
Authentication
  ↓
Runtime validation
  ↓
Controller
  ↓
Use Case
  ↓
Authorization Policy
  ↓
Repository Port
  ↓
Drizzle Repository
  ↓
PostgreSQL
  ↓
Response Mapper
  ↓
Typed JSON Response
```

Errors flow through:

```text
AppError
  ↓
ProblemDetailsFilter
  ↓
application/problem+json
```

This is the reference CRUD architecture Buildora AI should use for normal SaaS resources.
