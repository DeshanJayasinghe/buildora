# Buildora AI — Backend Engineering Standards & Reference Implementation

> **File name:** `Buildora_AI_Backend_Engineering_Standards.md`  
> **Audience:** Codex, Claude, human contributors, reviewers  
> **Applies to:** `apps/api`, backend workers, and shared domain/backend packages  
> **Primary stack:** NestJS + TypeScript + PostgreSQL + Drizzle + provider adapters
> (Redis and Temporal are **gated** — added only at their activation sprint, ADR-012)
> **Status:** Canonical coding standard unless superseded by an approved ADR
> **Subordinate to:** `docs/architecture/FINAL_SYSTEM_ARCHITECTURE.md` and ADR-001 … ADR-022.
> Where this document and an ADR disagree, **the ADR wins** — report the conflict.

---

# 1. Purpose

This document defines how Buildora AI backend code must be written so the codebase stays consistent while the SaaS grows from projects and tenancy into Building Model, QS, BOQ, cost, AI, BIM, documents, billing and enterprise workflows.

The main goals are:

- correctness,
- security,
- strong typing,
- reusability,
- clear boundaries,
- deterministic construction calculations,
- testability,
- observability,
- tenant isolation,
- stable API contracts,
- safe AI orchestration.

This is not only a style guide. It is the backend implementation contract.

---

# 2. Non-Negotiable Principles

1. **The Buildora AI Building Model is the source of truth.**
2. **Domain logic is framework-independent.**
3. **Controllers are thin.**
4. **Use cases/application services orchestrate work.**
5. **Repositories are persistence adapters, not business-logic containers.**
6. **LLMs never perform authoritative geometry, quantity, BOQ or cost arithmetic.**
7. **AI never directly mutates authoritative geometry.**
8. **Every tenant-owned query is tenant-scoped.**
9. **Money uses exact decimal-safe arithmetic.**
10. **Every external input is validated at runtime.**
11. **Errors use stable machine-readable codes.**
12. **Unexpected internal details never leak to clients.**
13. **Important changes are observable and auditable.**
14. **Concurrency is explicit through model/version checks.**
15. **Provider SDKs stay behind adapters.**
16. **Primary keys are `uuid`; `public_id` is display-only and never an FK target.** *(ADR-004)*
17. **`Level.elevationMm` is the single authoritative vertical fact.** Floor-to-floor and absolute z are derived, never stored. *(ADR-005)*
18. **A door or window IS its opening.** QS deduction uses one abstraction. *(frozen)*
19. **Element type identity is separate from cost-assembly identity.** *(ADR-007)*
20. **Tenant context is transaction-local.** `SET LOCAL` inside the transaction; never session-level. *(ADR-016)*
21. **The AI execution context uses a least-privilege database role** that cannot write canonical model tables. An import boundary alone is not a structural barrier. *(ADR-013)*
22. **Cost estimates pin their full lineage.** Historical estimates are never recomputed with current rates. *(ADR-015)*
23. **Every element query that feeds topology or QS filters on `element_class`.** FF&E never enters a construction quantity. *(ADR-019)*
24. **Finish, assembly and element type are three distinct axes.** Repainting creates a finish assignment, never a new assembly version. *(ADR-020)*
25. **Every user has a personal organization.** There is no user-owned project path. *(ADR-021)*
26. **Plans may be recurring or fixed-term.** An expired term archives the project; it is not a cancellation. *(ADR-021)*
27. **Feature code reads entitlements, never plan names.** *(ADR-021)*
28. **One set of engines for all customer segments.** No Home/Professional QS or cost duplication. *(ADR-021)*
29. **Every gated capability is checked server-side before the work happens.** Hiding a button is not access control. *(ADR-022)*
30. **Check order: feature flag → entitlement → credit reservation.** An unentitled request consumes no credit. *(ADR-022)*

---

# 3. Backend Architecture

Use this dependency direction:

```text
HTTP / Jobs / Webhooks
        ↓
Application Use Cases
        ↓
Domain / Building Model / QS / Cost
        ↓
Ports / Interfaces
        ↓
Infrastructure Adapters
        ↓
PostgreSQL / Redis / AI / Storage / Stripe / Temporal
```

Domain packages must not import:

- NestJS,
- Drizzle,
- PostgreSQL drivers,
- Redis SDK,
- OpenAI SDK,
- Stripe SDK,
- Three.js,
- PixiJS,
- React.

---

# 4. Repository Structure

```text
apps/
└── api/
    └── src/
        ├── main.ts
        ├── app.module.ts
        ├── config/
        ├── common/
        │   ├── audit/
        │   ├── auth/
        │   ├── errors/
        │   ├── http/
        │   ├── logging/
        │   ├── observability/
        │   └── validation/
        ├── db/
        │   ├── client.ts
        │   ├── schema/
        │   └── migrations/
        └── modules/
            ├── organizations/
            ├── projects/
            ├── building-model/
            ├── quantities/
            ├── boq/
            ├── cost/
            ├── documents/
            ├── ai/
            ├── billing/
            └── admin/

packages/
├── api-contracts/
├── units/
├── building-model/
├── model-session/     (browser working copy — frontend consumer)
├── qs-engine/
└── cost-engine/
```

> **`packages/domain` does not exist and must not be created** (ADR-011).
> **`packages/core` does not exist**; keep concepts with their owning module.
> **`packages/ai-tools` is not created yet** — gated to ~Sprint 27; if it stays a
> thin schema layer it merges into `api-contracts`.

Module folders are created when their sprint arrives, not up front (ADR-012).

---

# 5. Feature Module Structure

Prefer feature-oriented organization.

Example:

```text
apps/api/src/modules/projects/
├── projects.module.ts
├── projects.controller.ts
├── application/
│   ├── create-project.use-case.ts
│   ├── get-project.use-case.ts
│   └── archive-project.use-case.ts
├── policies/
│   └── project-access.policy.ts
├── ports/
│   └── project.repository.ts
├── mappers/
│   └── project-response.mapper.ts
├── errors/
│   └── project-not-found.error.ts
└── infrastructure/
    └── project.drizzle-repository.ts
```

Avoid giant global folders such as:

```text
controllers/
services/
repositories/
```

with unrelated domains mixed together.

---

# 6. Naming Standards

## Files

Use kebab-case:

```text
create-project.use-case.ts
project-access.policy.ts
problem-details.filter.ts
project.drizzle-repository.ts
```

## Classes

```text
CreateProjectUseCase
ProjectAccessPolicy
ProjectDrizzleRepository
```

## Functions / variables

```text
createProject()
organizationId
modelVersion
```

## Database columns

Use snake_case:

```text
organization_id
created_at
model_version
```

## Routes

Prefer plural resource nouns:

```text
GET  /api/v1/projects
POST /api/v1/projects
GET  /api/v1/projects/:projectId
```

Explicit command endpoints are acceptable when modelling a command:

```text
POST /api/v1/projects/:projectId/archive
POST /api/v1/ai/actions/:actionId/approve
```

---

# 7. TypeScript Standards

Use strict TypeScript.

Avoid:

```ts
any
// @ts-ignore
as unknown as Something
```

unless isolated at an unavoidable boundary and documented.

Treat external data as:

```ts
unknown
```

until runtime validation succeeds.

Prefer discriminated unions for workflows and commands.

---

# 8. Critical Domain Value Types

Do not overuse primitive numbers for construction values.

**File:** `packages/units/src/length-mm.ts`

```ts
export type LengthMm = number & { readonly __brand: "LengthMm" };

export function lengthMm(value: number): LengthMm {
  if (!Number.isFinite(value) || value < 0) {
    throw new Error("Length must be a finite non-negative number.");
  }

  return value as LengthMm;
}
```

Use explicit value types for critical concepts such as:

- millimetres,
- model version,
- money,
- percentages,
- domain IDs where beneficial.

Do not brand every trivial string.

> **Do not brand a level's absolute height.** There is no such stored value —
> `absoluteZ` is derived at read time from `level.elevationMm + baseOffsetMm`
> (ADR-005). A branded type here would invite storing it.

---

# 9. Money Standard

Never perform authoritative monetary calculations with binary floating point.

API representation:

```json
{
  "amount": "12345.67",
  "currency": "GBP"
}
```

Database:

```text
numeric(...)
```

Internally use a decimal-safe library or exact integer minor units when appropriate.

Never use formatted currency strings as calculation input.

---

# 10. Runtime Validation

TypeScript is not runtime validation.

Validate:

- body,
- query,
- route parameters,
- environment variables,
- webhook payloads,
- Temporal/job payloads,
- AI tool arguments,
- external provider responses where required.

Shared request/response schemas belong in:

```text
packages/api-contracts
```

Recommended schema library: Zod.

**File:** `packages/api-contracts/src/projects/create-project.schema.ts`

```ts
import { z } from "zod";

export const createProjectSchema = z.object({
  name: z.string().trim().min(1).max(120),
  location: z.string().trim().max(250).optional(),
  currency: z.enum(["GBP", "USD", "EUR", "LKR"]),
  unitSystem: z.enum(["metric", "imperial"]),
});

export type CreateProjectInput = z.infer<typeof createProjectSchema>;
```

---

# 11. API Success Responses

Return the resource/result directly where possible.

Good:

```json
{
  "id": "project_123",
  "name": "Modern Family Home",
  "status": "active"
}
```

Avoid unnecessary wrappers:

```json
{
  "success": true,
  "data": {}
}
```

For collections use a documented pagination envelope.

---

# 12. Error Standard — RFC 9457 Problem Details

Buildora AI API errors should use `application/problem+json`.

Example:

```json
{
  "type": "https://buildora.example/problems/project-not-found",
  "title": "Project not found",
  "status": 404,
  "detail": "The requested project does not exist or is not accessible.",
  "instance": "/api/v1/projects/project_123",
  "code": "PROJECT_NOT_FOUND",
  "traceId": "01J..."
}
```

Never expose:

- stack traces,
- SQL,
- provider errors,
- filesystem paths,
- database table names,
- secrets.

---

# 13. Base Application Error

**File:** `apps/api/src/common/errors/app-error.ts`

```ts
export interface AppErrorOptions {
  code: string;
  title: string;
  status: number;
  detail?: string;
  exposeDetail?: boolean;
  cause?: unknown;
  metadata?: Record<string, unknown>;
}

export class AppError extends Error {
  readonly code: string;
  readonly title: string;
  readonly status: number;
  readonly exposeDetail: boolean;
  readonly metadata?: Record<string, unknown>;

  constructor(options: AppErrorOptions) {
    super(options.detail ?? options.title, {
      cause: options.cause,
    });

    this.name = this.constructor.name;
    this.code = options.code;
    this.title = options.title;
    this.status = options.status;
    this.exposeDetail = options.exposeDetail ?? options.status < 500;
    this.metadata = options.metadata;
  }
}
```

---

# 14. Specific Error Example

**File:** `apps/api/src/modules/projects/errors/project-not-found.error.ts`

```ts
import { AppError } from "@/common/errors/app-error";

export class ProjectNotFoundError extends AppError {
  constructor() {
    super({
      code: "PROJECT_NOT_FOUND",
      title: "Project not found",
      detail: "The requested project does not exist or is not accessible.",
      status: 404,
    });
  }
}
```

Stable machine codes matter more than changing human sentences.

---

# 15. Standard Error Codes

Examples:

```text
VALIDATION_FAILED
UNAUTHENTICATED
FORBIDDEN
PROJECT_NOT_FOUND
ORGANIZATION_NOT_FOUND
MODEL_VERSION_CONFLICT
INVALID_MODEL_COMMAND
INVALID_GEOMETRY
RATE_NOT_FOUND
INSUFFICIENT_AI_CREDITS
AI_PROVIDER_UNAVAILABLE
DOCUMENT_PROCESSING_FAILED
FILE_TOO_LARGE
UNSUPPORTED_FILE_TYPE
SUBSCRIPTION_REQUIRED
FEATURE_NOT_ENTITLED
IDEMPOTENCY_CONFLICT
```

Treat error codes as part of the API contract.

---

# 16. Global Exception Filter

**File:** `apps/api/src/common/errors/problem-details.filter.ts`

```ts
import {
  ArgumentsHost,
  Catch,
  ExceptionFilter,
  HttpException,
  HttpStatus,
} from "@nestjs/common";
import type { Request, Response } from "express";
import { AppError } from "./app-error";

@Catch()
export class ProblemDetailsFilter implements ExceptionFilter {
  catch(exception: unknown, host: ArgumentsHost): void {
    const http = host.switchToHttp();
    const request = http.getRequest<Request>();
    const response = http.getResponse<Response>();

    const traceId =
      request.headers["x-request-id"]?.toString() ?? "unknown";

    if (exception instanceof AppError) {
      response
        .status(exception.status)
        .type("application/problem+json")
        .json({
          type: `https://buildora.example/problems/${exception.code
            .toLowerCase()
            .replaceAll("_", "-")}`,
          title: exception.title,
          status: exception.status,
          detail: exception.exposeDetail ? exception.message : undefined,
          instance: request.originalUrl,
          code: exception.code,
          traceId,
        });

      return;
    }

    if (exception instanceof HttpException) {
      const status = exception.getStatus();

      response
        .status(status)
        .type("application/problem+json")
        .json({
          type: "about:blank",
          title: HttpStatus[status] ?? "HTTP error",
          status,
          detail:
            status < 500
              ? "The request could not be completed."
              : undefined,
          instance: request.originalUrl,
          code: "HTTP_ERROR",
          traceId,
        });

      return;
    }

    // Log the original exception with the structured logger.

    response
      .status(500)
      .type("application/problem+json")
      .json({
        type: "https://buildora.example/problems/internal-error",
        title: "Internal server error",
        status: 500,
        detail: "An unexpected error occurred.",
        instance: request.originalUrl,
        code: "INTERNAL_ERROR",
        traceId,
      });
  }
}
```

Production implementation should inject/use the standard logger for the original exception.

---

# 17. Validation Error Shape

Recommended:

```json
{
  "type": "https://buildora.example/problems/validation-failed",
  "title": "Validation failed",
  "status": 422,
  "detail": "One or more request fields are invalid.",
  "code": "VALIDATION_FAILED",
  "traceId": "...",
  "errors": [
    {
      "path": "name",
      "message": "Required"
    }
  ]
}
```

Recommended convention:

```text
400 → malformed/syntactically invalid request
422 → structurally parsed but semantically invalid input
```

Use consistently.

---

# 18. Application Bootstrap

**File:** `apps/api/src/main.ts`

```ts
import { NestFactory } from "@nestjs/core";
import { AppModule } from "./app.module";
import { ProblemDetailsFilter } from "./common/errors/problem-details.filter";

async function bootstrap(): Promise<void> {
  const app = await NestFactory.create(AppModule, {
    bufferLogs: true,
  });

  app.setGlobalPrefix("api/v1");
  app.enableShutdownHooks();
  app.useGlobalFilters(new ProblemDetailsFilter());

  await app.listen(process.env.PORT ?? 4000);
}

void bootstrap();
```

Production bootstrap should also configure:

- validated configuration,
- structured logger,
- request IDs,
- secure proxy settings,
- body limits,
- allowed CORS origins only where needed,
- OpenAPI exposure policy,
- graceful shutdown.

---

# 19. Environment Validation

**File:** `apps/api/src/config/env.schema.ts`

```ts
import { z } from "zod";

export const envSchema = z.object({
  NODE_ENV: z
    .enum(["development", "test", "staging", "production"])
    .default("development"),

  PORT: z.coerce.number().int().positive().default(4000),

  DATABASE_URL: z.string().min(1),

  // Runtime role must be non-owner, non-superuser, without BYPASSRLS (ADR-016).
  // A separate least-privilege role is used for AI-facing execution (ADR-013).
  DATABASE_URL_AI: z.string().min(1).optional(),

  // Gated (ADR-012). Optional until its activation sprint.
  REDIS_URL: z.string().min(1).optional(),

  OPENAI_API_KEY: z.string().min(1).optional(),

  AI_MODEL_FAST: z.string().min(1).optional(),
  AI_MODEL_BALANCED: z.string().min(1).optional(),
  AI_MODEL_ADVANCED: z.string().min(1).optional(),
  AI_MODEL_EXPERT: z.string().min(1).optional(),
});
```

Fail startup if required configuration is invalid.

---

# 20. Thin Controller Example

**File:** `apps/api/src/modules/projects/projects.controller.ts`

```ts
import { Body, Controller, Get, Param, Post, Req } from "@nestjs/common";
import type { Request } from "express";
import {
  createProjectSchema,
  type CreateProjectInput,
} from "@buildora/api-contracts/projects";
import { CreateProjectUseCase } from "./application/create-project.use-case";
import { GetProjectUseCase } from "./application/get-project.use-case";

@Controller("projects")
export class ProjectsController {
  constructor(
    private readonly createProject: CreateProjectUseCase,
    private readonly getProject: GetProjectUseCase,
  ) {}

  @Post()
  async create(@Req() request: Request, @Body() body: unknown) {
    const input: CreateProjectInput = createProjectSchema.parse(body);

    return this.createProject.execute({
      actor: request.user,
      input,
    });
  }

  @Get(":projectId")
  async findOne(
    @Req() request: Request,
    @Param("projectId") projectId: string,
  ) {
    return this.getProject.execute({
      actor: request.user,
      projectId,
    });
  }
}
```

In the final codebase, wrap Zod validation in a reusable pipe/decorator so controllers do not repeat `.parse()`.

---

# 21. Use Case Example

**File:** `apps/api/src/modules/projects/application/create-project.use-case.ts`

```ts
import { Injectable } from "@nestjs/common";
import type { CreateProjectInput } from "@buildora/api-contracts/projects";
import { ProjectRepository } from "../ports/project.repository";
import { OrganizationAccessPolicy } from "../policies/organization-access.policy";
import { AuditService } from "@/common/audit/audit.service";

interface ExecuteParams {
  actor: {
    userId: string;
    organizationId: string;
  };
  input: CreateProjectInput;
}

@Injectable()
export class CreateProjectUseCase {
  constructor(
    private readonly projects: ProjectRepository,
    private readonly accessPolicy: OrganizationAccessPolicy,
    private readonly audit: AuditService,
  ) {}

  async execute(params: ExecuteParams) {
    await this.accessPolicy.requireProjectCreationPermission(params.actor);

    const project = await this.projects.create({
      organizationId: params.actor.organizationId,
      createdByUserId: params.actor.userId,
      name: params.input.name,
      location: params.input.location,
      currency: params.input.currency,
      unitSystem: params.input.unitSystem,
    });

    await this.audit.record({
      actorUserId: params.actor.userId,
      organizationId: params.actor.organizationId,
      action: "project.created",
      targetType: "project",
      targetId: project.id,
    });

    return project;
  }
}
```

---

# 22. Repository Port

**File:** `apps/api/src/modules/projects/ports/project.repository.ts`

```ts
export interface CreateProjectRecord {
  organizationId: string;
  createdByUserId: string;
  name: string;
  location?: string;
  currency: string;
  unitSystem: "metric" | "imperial";
}

export interface ProjectRecord {
  id: string;
  organizationId: string;
  name: string;
  location?: string;
  currency: string;
  unitSystem: "metric" | "imperial";
  createdAt: Date;
}

export abstract class ProjectRepository {
  abstract create(input: CreateProjectRecord): Promise<ProjectRecord>;

  abstract findAccessibleById(
    organizationId: string,
    projectId: string,
  ): Promise<ProjectRecord | null>;
}
```

---

# 23. Drizzle Repository Example

**File:** `apps/api/src/modules/projects/infrastructure/project.drizzle-repository.ts`

```ts
import { Injectable } from "@nestjs/common";
import { and, eq } from "drizzle-orm";
import { db } from "@/db/client";
import { projects } from "@/db/schema/projects";
import {
  ProjectRepository,
  type CreateProjectRecord,
  type ProjectRecord,
} from "../ports/project.repository";

@Injectable()
export class ProjectDrizzleRepository extends ProjectRepository {
  async create(input: CreateProjectRecord): Promise<ProjectRecord> {
    const [row] = await db
      .insert(projects)
      .values({
        organizationId: input.organizationId,
        createdByUserId: input.createdByUserId,
        name: input.name,
        location: input.location,
        currency: input.currency,
        unitSystem: input.unitSystem,
      })
      .returning();

    if (!row) {
      throw new Error("Database did not return created project.");
    }

    return row;
  }

  async findAccessibleById(
    organizationId: string,
    projectId: string,
  ): Promise<ProjectRecord | null> {
    const [row] = await db
      .select()
      .from(projects)
      .where(
        and(
          eq(projects.id, projectId),
          eq(projects.organizationId, organizationId),
        ),
      )
      .limit(1);

    return row ?? null;
  }
}
```

Do not let Drizzle query objects leak outside infrastructure/application layers.

---

# 24. Authentication vs Authorization

Authentication:

```text
Who are you?
```

Authorization:

```text
Can you perform this action on this resource?
```

Frontend visibility is not authorization.

Every protected backend operation must authorize independently.

---

# 25. Tenant Isolation

Every tenant-sensitive operation must be scoped by the authenticated organization context.

Prefer:

```ts
findProject(organizationId, projectId)
```

Avoid:

```ts
findProject(projectId)
```

unless the repository itself guarantees tenant scoping through another trusted relationship.

Do not accept `organizationId` from request body as proof of tenancy.

## Row-Level Security runtime contract (ADR-016)

Application scoping is the **primary** control. RLS is defence in depth. Both are
required — RLS does not license dropping the `WHERE` clause.

Tenant context is **transaction-local**, never session-level:

```ts
await db.transaction(async (tx) => {
  // `true` = LOCAL. Session-level SET leaks across pooled connections.
  await tx.execute(
    sql`SELECT set_config('app.current_organization_id', ${organizationId}, true)`,
  );

  // Every tenant query for this unit of work runs on THIS transaction.
  return repository.withTx(tx).findProject(organizationId, projectId);
});
```

Binding rules:

1. `SET LOCAL` / `set_config(..., true)` only.
2. A query issued outside that transaction has no tenant context and must **fail
   closed**, never fall back to unscoped access.
3. The runtime role is **non-owner, non-superuser, without `BYPASSRLS`**.
   PostgreSQL exempts table owners from RLS by default — a test suite running as
   the owner passes while enforcing nothing.
4. Integration tests run with RLS **enabled**, using the production driver and
   pooling mode, from Sprint 05.

---

# 26. Policy Example

**File:** `apps/api/src/modules/projects/policies/project-access.policy.ts`

```ts
import { AppError } from "@/common/errors/app-error";

export class ProjectAccessPolicy {
  requireCanEdit(role: "owner" | "admin" | "member" | "viewer"): void {
    if (role === "viewer") {
      throw new AppError({
        code: "PROJECT_EDIT_FORBIDDEN",
        title: "Project editing is not allowed",
        detail: "Your role does not allow editing this project.",
        status: 403,
      });
    }
  }
}
```

Centralize policy behaviour rather than scattering role checks.

---

# 27. Transactions

Transaction boundaries belong around business operations.

Example:

```text
Approve AI action
→ validate version
→ commit model changes
→ increment model version
→ create audit event
→ mark AI action applied
```

should be atomic where practical.

Do not hold a DB transaction open while waiting for:

- AI provider,
- email,
- object storage,
- external supplier,
- long Temporal activity.

Use outbox/workflow patterns for cross-system reliability when required.

---

# 28. Optimistic Concurrency

Every Building Model mutation should carry `baseVersion`.

Example:

```json
{
  "baseVersion": 42,
  "command": {
    "type": "MOVE_WALL",
    "wallId": "wall_123"
  }
}
```

If current model version is 43:

```text
409 Conflict
MODEL_VERSION_CONFLICT
```

Never silently overwrite newer model state.

---

# 29. Command Pattern

Model changes are explicit commands.

Example:

```ts
export interface MoveWallCommand {
  type: "MOVE_WALL";
  wallId: string;
  startXmm: number;
  startYmm: number;
  endXmm: number;
  endYmm: number;
  baseVersion: number;
}
```

Command handler responsibilities:

- load model,
- validate version,
- validate permission,
- execute domain operation,
- validate geometry,
- persist,
- increment version,
- audit.

---

# 30. Idempotency

## Required for every externally submitted mutation

A client cannot know whether a timed-out request committed. Retrying a **semantic**
mutation without an idempotency key applies it twice, and both journal entries look
individually valid.

```text
MOVE_WALL_BY(deltaX = 100mm)  commits, response lost
client retries                 wall moves 200mm
```

Therefore:

- Every externally submitted mutation carries a **client-generated idempotency key**,
  unique within organization/project (or command stream).
- The original response and resulting version are persisted for replay, so a retry
  returns the first result rather than reapplying.
- Internal deterministic cascade items are covered by the parent command
  transaction and need no separate key.

Also required for: Stripe webhooks, credit reservation, report/render requests,
purchase orders.

Do not add idempotency complexity to ordinary read requests.

---

# 31. Database Standards

Typical business table fields:

```text
id
created_at
updated_at
```

Tenant-owned tables:

```text
organization_id
```

Use soft deletion only where product/legal behaviour requires it.

Financial ledgers, usage ledgers and audit records should generally use immutable/append-style patterns.

---

# 32. Migration Standards

- migrations are committed,
- migrations are reviewed,
- never edit an already-applied production migration,
- destructive changes require staged rollout,
- production does not use ad-hoc schema push.

For dangerous changes use:

```text
expand
→ migrate/backfill
→ deploy new readers/writers
→ contract old schema
```

---

# 33. Query Standards

Avoid:

- N+1,
- unbounded lists,
- `SELECT *` in high-volume paths when only a few columns are required,
- client-controlled arbitrary sorting over unindexed fields.

Use indexes based on real filters and measured performance.

---

# 34. Pagination

Prefer cursor pagination for large changing collections.

Example:

```http
GET /api/v1/projects?limit=25&cursor=...
```

```json
{
  "items": [],
  "page": {
    "nextCursor": "...",
    "hasMore": true
  }
}
```

Server enforces maximum page size.

---

# 35. Logging

Use structured logging.

Include useful context:

```text
traceId
userId
organizationId
projectId
modelVersion
workflowId
providerRequestId
```

Never log:

- passwords,
- tokens,
- API keys,
- payment secrets,
- full private documents,
- unredacted sensitive AI inputs by default.

Avoid `console.log` in production application code.

---

# 36. Request/Trace IDs

Every request should have a trace/request ID.

Propagate through:

- API,
- DB spans,
- Redis,
- Temporal,
- AI provider calls,
- internal workers.

Return a support reference when useful.

---

# 37. Audit Logging

Audit logs are separate from debug/application logs.

Audit:

- membership changes,
- role changes,
- project deletion,
- model commits,
- AI action approval,
- rate override,
- BOQ/report export,
- subscription/credit adjustment.

Audit records should identify:

- actor,
- tenant,
- resource,
- action,
- timestamp,
- before/after metadata where appropriate.

---

# 38. Health Endpoints

Use:

```text
/health/live
/health/ready
```

Liveness:

```text
process can run
```

Readiness:

```text
critical dependencies usable
```

Do not fail liveness because optional AI is unavailable.

---

# 39. Redis

Valid uses:

- cache,
- rate limiting,
- distributed lock where required,
- presence,
- temporary reservations.

Never make Redis the only source for:

- Building Model,
- subscription,
- credits,
- BOQ,
- project permission.

---

# 40. Cache Standard

Do not cache blindly.

Every cache requires:

- reason,
- key format,
- TTL,
- invalidation strategy,
- stale-data acceptance.

Good versioned key:

```text
project:{projectId}:model:{modelVersion}:summary
```

---

# 41. Temporal

Use Temporal for durable multi-step workflows:

- plan recognition,
- document processing,
- IFC import,
- rendering,
- report generation,
- long AI workflows.

Temporal Workflow code must remain deterministic.

Network/database/provider calls belong in Activities.

Do not use Temporal for normal CRUD.

---

# 42. Events

Use explicit domain/application events for meaningful changes.

Example:

```ts
export interface BuildingModelVersionCommitted {
  type: "building-model.version-committed";
  projectId: string;
  version: number;
  affectedElementIds: string[];
}
```

Possible consumers:

- derived-data invalidation,
- QS recalculation,
- search/index updates.

Avoid a chaotic global event bus.

---

# 43. AI Provider Boundary

Only provider adapters import provider SDKs.

```text
AiGateway
    ↓
AiProvider
    ↓
OpenAiProvider
```

Never import the OpenAI SDK inside:

- QS,
- cost,
- building model,
- projects,
- repositories.

## The import rule is not the security barrier (ADR-013)

An import boundary stops an **intended** dependency. It does nothing against an
**accidental** one: in a modular monolith every module shares the process and, by
default, one database credential. If a generic `db` handle is injected into a
shared service during a refactor, TypeScript will not stop it — the credential
still has permission to execute the SQL.

Two levels are required:

| Level | Mechanism | Prevents |
|---|---|---|
| Code | AI module imports no model repository; CI boundary check | Intended dependency |
| **Database** | AI-facing context uses a **least-privilege role** with no INSERT/UPDATE/DELETE grant on canonical model tables | Accidental one |

Tests must assert the **database privilege**, not only the import graph.

---

# 44. AI Model Routing

Application code uses aliases:

```text
FAST_MODEL
BALANCED_MODEL
ADVANCED_MODEL
EXPERT_MODEL
```

Actual provider model IDs live in configuration.

Routing considers:

- task type,
- complexity,
- context,
- affected elements,
- risk,
- plan entitlement,
- credit budget.

---

# 45. AI Tool Standard

Every tool has:

- stable name,
- purpose,
- validated input,
- authorization,
- timeout,
- output schema,
- audit policy.

**File:** `packages/ai-tools/src/get-project-summary.tool.ts`

```ts
import { z } from "zod";

export const getProjectSummaryInput = z.object({
  projectId: z.string().min(1),
});

export type GetProjectSummaryInput =
  z.infer<typeof getProjectSummaryInput>;
```

Never allow model-generated SQL.

---

# 46. AI Mutation Standard

AI mutation path:

```text
AI proposal
→ structured ChangeSet
→ validate
→ draft/sandbox model
→ geometry check
→ QS/cost delta
→ preview
→ user approval
→ normal domain command
```

Example:

```ts
export interface MoveWallChange {
  type: "MOVE_WALL";
  wallId: string;
  deltaXmm: number;
  deltaYmm: number;
  baseVersion: number;
}
```

AI has no direct DB update path.

---

# 47. Usage and Credit Accounting

Capture:

- tenant,
- user,
- project,
- task,
- model alias,
- provider model,
- provider request ID,
- token/usage metadata,
- estimated provider cost,
- credit reserved,
- credit settled,
- duration,
- result status.

Credit calculations are deterministic backend logic.

---

# 48. File Upload Security

Never trust:

- filename,
- extension,
- MIME header alone,
- client object key.

Validate:

- permission,
- maximum size,
- supported type,
- content/file signature where practical.

Generate object keys server-side.

Files private by default.

---

# 49. Webhooks

Every webhook:

1. verifies signature,
2. validates payload,
3. records provider event ID,
4. is idempotent,
5. responds quickly,
6. enqueues long processing.

Do not trust a webhook because its URL is obscure.

---

# 50. REST Security

Use:

- HTTPS,
- standard HTTP methods/status codes,
- server-side authorization,
- strict validation,
- size limits,
- content-type validation,
- rate limiting,
- generic client errors,
- secure headers/gateway controls,
- audit logs.

Never rely on frontend sequencing for workflow security.

Every endpoint validates the current workflow state.

---

# 51. Rate Limiting

Prioritize:

- auth,
- upload,
- AI,
- reports,
- public links,
- expensive conversion.

Return:

```text
429 Too Many Requests
```

with `Retry-After` where appropriate.

---

# 52. Secrets

Secrets live only in:

- environment variables,
- secret manager.

Never in:

- Git,
- logs,
- API responses,
- public Next.js variables,
- agent prompts/screenshots.

---

# 53. HTTP Status Guidelines

```text
200 OK
201 Created
202 Accepted
204 No Content
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
409 Conflict
413 Content Too Large
415 Unsupported Media Type
422 Unprocessable Content
429 Too Many Requests
500 Internal Server Error
502/503 Upstream/service unavailable where appropriate
```

Never return HTTP 200 with an error body.

---

# 54. Long-Running Operations

Return `202 Accepted` and a job/workflow identifier when a job can take meaningful time.

Example:

```json
{
  "jobId": "job_123",
  "status": "queued"
}
```

Do not hold an HTTP connection open for a multi-minute plan import.

---

# 55. Date and Time

Persist timezone-aware UTC timestamps.

API format:

```text
ISO 8601
```

Example:

```text
2026-09-11T01:23:45.123Z
```

Do not persist display-formatted local dates as authoritative timestamps.

---

# 56. Derived Data Provenance

Quantities/BOQ/cost results must reference the inputs that created them.

Record the **full lineage** (ADR-015). A cost estimate pins **all** of:

```text
projectId
modelVersion
quantityRunId            ← required
boqVersionId
rateBookVersionId
costAssumptionSetVersion ← required
qsRuleSetVersion
costEngineVersion
createdAt
```

Never present a BOQ from model version 12 as current if model version is 14.

**Never recompute a historical estimate with today's rate book and call it the same
estimate.** Reproduction uses the versions it pinned; an update creates a new version.

> **Scope of the guarantee.** Recalculation guarantees **numeric equivalence** from
> the pinned tuple. A regenerated PDF is *not* guaranteed byte-identical, because
> that also depends on template, renderer, fonts, locale and timezone. **Issued
> reports are preserved as immutable, checksummed artifacts in object storage**
> rather than regenerated on demand.

---

# 57. Recalculation Strategy

On model commit:

```text
identify affected elements
→ invalidate dependent results
→ recalculate cheaply if possible
or
→ enqueue durable recalculation
```

Correctness first.

Optimize to deltas only after tests prove equivalence.

---

# 58. External Provider Adapters

Use adapters for:

- auth,
- AI,
- Stripe,
- storage,
- email,
- supplier pricing,
- BIM services.

Application code consumes internal interfaces, not provider classes.

---

# 58.1 Gated Infrastructure

Redis and Temporal are **not available from day one** (ADR-012). Code written
before their activation sprint must not assume them.

| Service | Activation | Before that, use |
|---|---|---|
| Redis | ~Sprint 27 (AI rate limiting) | In-process cache; no distributed lock |
| Temporal | ~Sprint 31 (upload pipeline) | A `processing_jobs` table + in-process worker |

Design the **interface** (job creation, status polling) so adopting Temporal later
is an implementation change behind an existing port — not an architecture change.

Do not add a managed service without naming its consumer.

---

# 59. Retry Policy

Retry only transient failures.

Potential retries:

- selected 429,
- network timeout,
- selected 5xx.

Do not retry:

- 4xx validation,
- authorization,
- insufficient credits,
- deterministic geometry failure.

Use bounded exponential backoff and jitter for external providers.

---

# 60. Graceful Degradation

If AI is down, users must still be able to:

- open projects,
- edit 2D,
- view 3D,
- calculate QS,
- generate BOQ/cost from deterministic engines.

Provider outage should not collapse unrelated product capabilities.

---

# 61. Testing Strategy

## Domain unit tests

High volume:

- units,
- building commands,
- geometry,
- QS,
- cost,
- ledgers.

## Application tests

- use cases,
- authorization policies,
- transactions,
- error mapping.

## Integration tests

- PostgreSQL,
- Redis,
- API,
- tenant isolation.

## Contract tests

- schemas,
- tools,
- webhooks.

## E2E

Critical SaaS workflows.

---

# 62. Test Naming

Good:

```ts
it("subtracts hosted window area from wall net area", ...)
```

Bad:

```ts
it("test1", ...)
```

Name tests by observable behaviour.

---

# 63. QS Unit Test Example

**File:** `packages/qs-engine/src/wall-quantity.test.ts`

```ts
import { describe, expect, it } from "vitest";
import { calculateNetWallAreaM2 } from "./wall-quantity";

describe("calculateNetWallAreaM2", () => {
  it("subtracts opening area from gross wall area", () => {
    const result = calculateNetWallAreaM2({
      wallLengthMm: 5000,
      wallHeightMm: 2700,
      openings: [
        {
          widthMm: 1000,
          heightMm: 2100,
        },
      ],
    });

    expect(result).toBe(11.4);
  });
});
```

---

# 64. Tenant Isolation Test

Create:

```text
Organization A → project X
Organization B → user Y
```

Attempt:

```text
GET project X as user Y
```

Expected:

```text
404 or 403 according to deliberate anti-enumeration policy
```

This test is mandatory for tenant-sensitive repositories/endpoints.

---

# 65. Error Test Standard

Error tests assert:

- HTTP status,
- machine code,
- absence of sensitive details,
- validation details when intended.

Avoid asserting complete human text unless it is intentionally stable.

---

# 66. Coding Style

Prefer:

- small cohesive functions,
- explicit inputs,
- immutable calculation functions,
- meaningful names,
- early validation.

Avoid:

- God services,
- generic `UtilsService`,
- deep inheritance,
- hidden global state,
- magic booleans,
- 500+ line services,
- “helper” dumping grounds.

Bad:

```ts
processProject(project, true, false, true);
```

Good:

```ts
recalculateProjectCost({
  project,
  includeWaste: true,
  includeContingency: false,
});
```

---

# 67. Comments

Comments explain **why**, assumptions and constraints.

Bad:

```ts
// increment version
version += 1;
```

Good:

```ts
// Every committed command advances the model version so stale editor
// sessions can be rejected through optimistic concurrency.
version += 1;
```

---

# 68. TODOs

Good:

```ts
// TODO(BW-431): replace StaticRateProvider after supplier rates are introduced.
```

Bad:

```ts
// TODO fix later
```

---

# 69. OpenAPI / API Documentation

Every public endpoint should define:

- auth requirement,
- request schema,
- response schema,
- error codes,
- pagination,
- side effects,
- idempotency requirement.

Generate OpenAPI where practical.

---

# 70. Health and Shutdown

Enable graceful shutdown.

Close:

- DB pool,
- Redis,
- workers,
- telemetry exporters.

Stop accepting new traffic before tearing down critical resources.

---

# 71. Performance Rules

Measure before optimizing.

Track:

- DB query latency,
- API p95/p99,
- model load time,
- serialization size,
- QS execution time,
- AI provider latency.

Do not send the entire project model over every small endpoint.

---

# 72. Common Backend Anti-Patterns — Forbidden

Do not normalize these patterns:

```text
Controller contains SQL
Controller calls OpenAI
Repository contains business policy
QS imports OpenAI SDK
Domain imports NestJS
Money uses float
Tenant ID trusted from request body
Authorization only in frontend
AI-generated SQL executed directly
LLM calculates authoritative quantities
Stack traces returned to client
catch(error) { return null }
Errors swallowed silently
Redis becomes project database
Provider objects leak into domain
One global GodService
varchar primary key on a new table
public_id used as a foreign-key target
floor-to-floor or absolute z stored as a column
a second Opening element for a door/window hole
element type conflated with cost assembly
session-level SET for tenant context
tests running as the table owner (RLS silently bypassed)
model mutation accepted without an idempotency key
cost estimate persisted without quantity run + assumption set
element query feeding QS without an element_class filter
furniture stored only in a renderer scene
GLB/GLTF binary in a relational column
finish change creating a new assembly version
`if (plan === "Pro")` in feature code
a second QS or cost engine for a customer segment
gated capability enforced only by hiding a UI control
feature flag used to grant paid access (flags are rollout, not entitlement)
credit reserved before the entitlement check
projects owned directly by a user instead of an organization
```

---

# 73. Sample Project Creation Flow

```text
HTTP request
    ↓
Authentication
    ↓
Runtime validation
    ↓
ProjectsController
    ↓
CreateProjectUseCase
    ↓
Authorization policy
    ↓
ProjectRepository
    ↓
PostgreSQL
    ↓
Audit record
    ↓
Response mapper
    ↓
HTTP 201
```

---

# 74. Sample Model Command Flow

```text
POST model command
    ↓
authenticate
    ↓
validate schema
    ↓
authorize project
    ↓
load current model/version
    ↓
verify baseVersion
    ↓
execute domain command
    ↓
validate geometry
    ↓
transaction
        save elements
        increment model version
        audit
    ↓
publish version-committed event
    ↓
invalidate/recalculate derived data
```

---

# 75. Sample Error Flow

```text
Domain detects stale model
    ↓
ModelVersionConflictError
    ↓
ProblemDetailsFilter
    ↓
409 application/problem+json
```

Example:

```json
{
  "type": "https://buildora.example/problems/model-version-conflict",
  "title": "Model version conflict",
  "status": 409,
  "detail": "The project changed after this edit was started.",
  "code": "MODEL_VERSION_CONFLICT",
  "traceId": "01J..."
}
```

---

# 76. Sample AI Read Flow

```text
Copilot request
    ↓
AiGateway
    ↓
model router
    ↓
selected model alias
    ↓
tool: get_cost
    ↓
authorize project
    ↓
deterministic CostReadService
    ↓
structured result
    ↓
AI explanation
    ↓
usage ledger
```

---

# 77. Sample AI Mutation Flow

```text
User: "Move kitchen wall 1m"
    ↓
AI proposes move_wall
    ↓
validated ChangeSet
    ↓
draft model
    ↓
geometry validation
    ↓
QS/cost impact
    ↓
preview
    ↓
user approval
    ↓
normal model command pipeline
```

AI never bypasses the normal command path.

---

# 78. Backend Pull Request Checklist

Before merge:

- [ ] correct bounded context
- [ ] controller thin
- [ ] runtime input validation
- [ ] server-side authorization
- [ ] tenant scoping
- [ ] no duplicated source of truth
- [ ] deterministic domain logic isolated
- [ ] provider SDK behind adapter
- [ ] stable error code
- [ ] RFC 9457 error behaviour
- [ ] structured logging
- [ ] audit event where required
- [ ] transaction boundary correct
- [ ] concurrency/version considered
- [ ] idempotency key required for external mutations
- [ ] uuid PK; public_id not an FK target
- [ ] derived lineage pinned where applicable
- [ ] tenant context transaction-local
- [ ] migration reviewed
- [ ] tests added
- [ ] secrets absent
- [ ] `pnpm verify` passes

---

# 79. References

Keep implementation aligned with current official guidance:

- NestJS Exception Filters  
  https://docs.nestjs.com/exception-filters
- NestJS Validation  
  https://docs.nestjs.com/techniques/validation
- RFC 9457 — Problem Details for HTTP APIs  
  https://www.rfc-editor.org/rfc/rfc9457.html
- OWASP REST Security Cheat Sheet  
  https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html
- PostgreSQL  
  https://www.postgresql.org/docs/
- Drizzle ORM  
  https://orm.drizzle.team/docs/
- Temporal  
  https://docs.temporal.io/
- OpenTelemetry JavaScript  
  https://opentelemetry.io/docs/languages/js/

---

# 80. Final Backend Rule

When multiple implementations are possible, prefer the one that preserves:

```text
Correctness
→ Security
→ Domain clarity
→ Testability
→ Observability
→ Maintainability
→ Performance
```

The backend is consistent when every important operation is:

> **authenticated, authorized, validated, executed through an explicit use case/domain path, persisted safely, observable, auditable where necessary, and returned through a stable API contract without leaking infrastructure details.**
