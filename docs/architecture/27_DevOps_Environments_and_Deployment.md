# Buildora AI — Deployment, Production & DevOps Architecture Plan

> **File name:** `27_DevOps_Environments_and_Deployment.md`
>
> **Purpose:** Canonical deployment, production operations, CI/CD, infrastructure, security, observability, backup, disaster recovery, scaling, and DevOps plan for Buildora AI.
>
> **Development model:** One-person SaaS development assisted by Codex Plus, Claude Pro, and Antigravity Pro.
>
> **Primary runtime architecture:** Docker + Cloudflare + AWS ECS/Fargate + ECR + managed PostgreSQL + managed Redis + Cloudflare R2 + Temporal Cloud + GitHub Actions.
>
> **Status:** Canonical DevOps source of truth unless superseded by an approved ADR.
>
> **Important:** Buildora AI should remain deployable as ordinary containers. Do not introduce Kubernetes, service mesh, multi-region active/active, or unnecessary microservices before measured production requirements justify them.

---

# 1. Purpose

Buildora AI is a production SaaS containing:

```text
Next.js Web Application
NestJS API
Python AI / CV Workers
BIM Workers
Temporal Workers
Future Render / GPU Workers
PostgreSQL
Redis
Object Storage
AI Provider Integrations
Stripe
Authentication Provider
```

The DevOps architecture must support:

- secure deployments,
- repeatable environments,
- safe database migrations,
- zero/low-downtime releases,
- fast rollback,
- background workflows,
- AI usage/cost controls,
- file processing,
- monitoring,
- disaster recovery,
- scaling,
- operational cost control,
- solo-developer maintainability.

The deployment architecture must not become more complicated than the product itself.

---

# 2. Core DevOps Principles

Buildora AI follows these principles.

## 2.1 Infrastructure is reproducible

Production infrastructure is defined using Infrastructure as Code.

No critical resource should exist only because it was manually clicked in a cloud console.

Manual emergency changes must later be reflected in IaC.

---

## 2.2 Build once, promote the same artifact

A release should follow:

```text
Commit
 ↓
Build immutable image
 ↓
Test image
 ↓
Deploy same image digest to staging
 ↓
Verify
 ↓
Promote same digest to production
```

Do not rebuild different production code after staging approval.

---

## 2.3 Deploy by immutable identifier

Use:

```text
Git commit SHA
container digest
release version
```

Do not deploy:

```text
latest
```

as the authoritative production reference.

---

## 2.4 Production secrets never live in Git

Use:

- cloud secret manager,
- encrypted environment configuration,
- short-lived CI credentials.

Never:

- `.env.production` in repository,
- AWS access keys in GitHub secrets if OIDC can be used,
- provider secret keys in frontend public variables.

---

## 2.5 Database migration is an explicit deployment step

Do not run migrations automatically at application startup.

Production deployment:

```text
validate migration
 ↓
run migration job
 ↓
verify
 ↓
deploy compatible application
```

---

## 2.6 Observability is part of the release

A production feature is incomplete if failures cannot be detected and diagnosed.

Every production deployable must have:

- structured logs,
- error reporting,
- health signal,
- metrics/traces where useful,
- release/version metadata.

---

## 2.7 External dependency failure must degrade gracefully

If OpenAI is down:

```text
2D editing
3D
QS
BOQ
Cost
```

should still work.

If Stripe is down:

existing active users should continue using already confirmed entitlements where safe.

If Redis is down:

critical project data must remain intact.

---

## 2.8 Kubernetes is not Phase 1

Do not deploy Buildora AI to Kubernetes merely because it may eventually scale.

Initial production target:

```text
AWS ECS/Fargate
```

Kubernetes/EKS is a future decision based on:

- service count,
- GPU orchestration,
- deployment complexity,
- custom networking,
- scale,
- team size.

---

# 3. Environment Strategy

Use four logical environments.

```text
Local
Preview / CI
Staging
Production
```

---

# 4. Local Environment

Purpose:

- development,
- unit/integration tests,
- local debugging.

Runs:

```text
Next.js
NestJS
PostgreSQL Docker
Redis Docker
Temporal dev server when needed
Python worker when needed
```

Local environment does not use production secrets.

Suggested URLs:

```text
Web       http://localhost:3000
API       http://localhost:4000
Temporal  http://localhost:8233
```

---

# 5. Preview / Pull Request Environment

Do not create a full expensive AWS environment for every trivial PR.

Use two preview levels.

## Level 1 — CI-only PR

For normal PRs:

```text
lint
typecheck
unit tests
integration tests
build
container build
migration test
```

No cloud deployment.

## Level 2 — Preview deployment

For:

- major UI,
- auth,
- editor,
- billing,
- migration-heavy PRs.

Optional preview contains:

```text
temporary application deployment
temporary database branch
test credentials
```

If using Neon, a dedicated database branch per selected PR is suitable.

Delete preview infrastructure automatically after PR close.

---

# 6. Staging Environment

Staging must be production-like.

It uses:

- same container build,
- same deployment runtime,
- same major DB version,
- same Redis class,
- same storage API,
- same Temporal workflow architecture.

But:

```text
different credentials
different database
different buckets
Stripe test mode
test auth environment
separate AI keys/budget
```

Staging is not a shared developer scratch database.

---

# 7. Production Environment

Production is isolated.

Required separation:

```text
production DB
production Redis
production storage bucket
production Stripe account/mode
production auth environment
production AI project/key
production Temporal namespace
production Sentry environment
```

Prefer a separate AWS production account from non-production before paid launch.

For early internal alpha, one account with strong environment separation is acceptable.

Before paid customers:

```text
Production AWS account
Non-production AWS account
```

is preferred.

---

# 8. Recommended V1 Production Topology

```text
                           INTERNET
                              │
                              ▼
                         CLOUDFLARE
                    DNS / WAF / CDN / TLS
                              │
                    ┌─────────┴─────────┐
                    │                   │
                    ▼                   ▼
             app.buildora...    api.buildora...
                    │                   │
                    └─────────┬─────────┘
                              ▼
                     AWS Application
                      Load Balancer
                              │
                 ┌────────────┴────────────┐
                 ▼                         ▼
           ECS Web Service           ECS API Service
           Next.js/Fargate           NestJS/Fargate
                                           │
           ┌───────────────────────────────┼──────────────────────┐
           │                               │                      │
           ▼                               ▼                      ▼
     Managed PostgreSQL              Managed Redis          Temporal Cloud
       Neon initially               private/TLS               │
                                                               ▼
                                                         ECS Workers
                                                   TS / Python / BIM

           ┌──────────────────────────────────────────────────────┐
           │                                                      │
           ▼                                                      ▼
     Cloudflare R2                                          External APIs
   private object storage                          OpenAI / Stripe / Clerk
```

Future:

```text
GPU render queue
    ↓
ECS EC2 GPU / AWS Batch GPU
```

Do not make GPU capacity permanently running before demand exists.

---

# 9. Why This Topology

It gives a solo developer:

- no Kubernetes cluster management,
- container portability,
- managed DB,
- managed workflow engine,
- managed object storage,
- simple autoscaling,
- clear deployment boundaries,
- low operational staffing requirements.

---

# 10. Primary AWS Region

Choose one primary region.

Criteria:

- customer geography,
- latency,
- service availability,
- data residency,
- cost.

Record as an ADR.

Do not hardcode a region into application domain logic.

For a UK-first deployment, a UK/EU region may be sensible, but the final choice must follow actual customer/data-residency requirements.

---

# 11. Domain Architecture

Suggested:

```text
app.buildora.example
api.buildora.example
```

Optional:

```text
status.buildora.example
docs.buildora.example
```

Admin should usually remain part of authenticated app:

```text
app.../admin
```

rather than a public separate origin unless security/operational reasons justify it.

---

# 12. Cloudflare Responsibilities

Use Cloudflare for:

- authoritative DNS,
- edge TLS,
- WAF,
- DDoS protection,
- CDN/static asset caching,
- basic bot/rate controls where appropriate,
- R2 storage.

Do not duplicate every Cloudflare control with another edge layer unless there is a reason.

---

# 13. Cloudflare Cache Policy

Authenticated app/API content should generally:

```text
BYPASS edge cache
```

Cache aggressively:

```text
/_next/static/*
immutable hashed JS/CSS
public static assets
versioned images
```

Potentially cache selected public marketing content.

Never cache:

- private API responses,
- billing state,
- project data,
- presigned URLs beyond intended policy.

---

# 14. Cloudflare Origin Protection

Do not leave the origin unnecessarily open.

Options include:

- restrict ALB ingress to Cloudflare IP ranges,
- automate range updates through IaC,
- use supported origin authentication controls.

Do not depend only on an obscure origin hostname as security.

---

# 15. Object Storage — Cloudflare R2

Use separate buckets:

```text
buildora-dev
buildora-staging
buildora-production
```

or equivalent isolated resources.

Do not mix environment prefixes inside one production bucket if separate buckets are feasible.

---

# 16. R2 Upload Architecture

Preferred upload flow:

```text
Browser
 ↓
API requests upload authorization
 ↓
API verifies:
 tenant
 project
 entitlement
 size/type policy
 ↓
API issues short-lived presigned PUT
 ↓
Browser uploads directly to R2
 ↓
Browser/API finalizes upload
 ↓
API verifies object metadata
 ↓
storage_objects = AVAILABLE
```

This prevents large files from consuming API container bandwidth unnecessarily.

---

# 17. R2 Presigned URL Rules

Presigned URLs are bearer credentials.

Use:

- short expiration,
- one object,
- one intended operation,
- restricted content type where practical.

Do not log complete presigned URLs.

Do not store them permanently.

Cloudflare R2 currently supports presigned S3 URLs for GET/HEAD/PUT/DELETE with bounded expiration.

---

# 18. Large File Uploads

Use multipart upload for large files.

Examples:

- IFC,
- DWG,
- point clouds later,
- video.

Single PUT is suitable for normal smaller files.

Multipart gives:

- resumability,
- parallel upload,
- retry of failed parts.

---

# 19. R2 Lifecycle Rules

Use lifecycle policies for temporary data.

Examples:

```text
temporary upload parts → expire
failed-processing staging files → expire
temporary renders → expire after policy
exports → expire if product policy permits
```

Do not apply automatic deletion to authoritative customer originals without explicit retention policy.

---

# 20. Container Registry

Use:

```text
Amazon ECR
```

Repositories:

```text
buildora-web
buildora-api
buildora-ai-worker
buildora-bim-worker
buildora-render-worker
```

Only create repositories when deployables exist.

---

# 21. Image Naming

Tag images:

```text
<git-sha>
release-<version>
```

Example:

```text
buildora-api:9f4e1cb...
buildora-api:release-1.2.0
```

ECS task definition should ultimately resolve/use immutable digest.

Avoid relying on:

```text
latest
```

---

# 22. Container Vulnerability Scanning

At minimum:

- scan on push,
- block known critical vulnerabilities according to policy,
- review base images regularly.

ECR supports image vulnerability scanning; enhanced scanning integrates with Amazon Inspector for OS and programming-language dependency findings.

Use enhanced scanning when budget/operational needs justify it.

---

# 23. Base Image Rules

Use small supported runtime images.

Examples conceptually:

```text
node:24-...
python:3.13-slim
```

Pin major/runtime versions.

Do not deploy development images containing:

- compilers unnecessarily,
- test fixtures,
- package manager caches,
- source-control metadata.

Use multi-stage Docker builds.

---

# 24. Next.js Web Dockerfile — Example

**File:** `apps/web/Dockerfile`

```dockerfile
# syntax=docker/dockerfile:1

FROM node:24-bookworm-slim AS base

ENV PNPM_HOME="/pnpm"
ENV PATH="$PNPM_HOME:$PATH"

RUN corepack enable

WORKDIR /workspace

FROM base AS dependencies

COPY package.json pnpm-lock.yaml pnpm-workspace.yaml ./
COPY apps/web/package.json apps/web/package.json
COPY packages/design-system/package.json packages/design-system/package.json
COPY packages/api-contracts/package.json packages/api-contracts/package.json

RUN pnpm install --frozen-lockfile

FROM dependencies AS build

COPY . .

RUN pnpm --filter web build

FROM node:24-bookworm-slim AS runtime

ENV NODE_ENV=production
ENV PORT=3000

WORKDIR /app

# Prefer Next.js standalone output.
COPY --from=build /workspace/apps/web/.next/standalone ./
COPY --from=build /workspace/apps/web/.next/static ./apps/web/.next/static
COPY --from=build /workspace/apps/web/public ./apps/web/public

USER node

EXPOSE 3000

CMD ["node", "apps/web/server.js"]
```

Configure Next.js:

```js
output: "standalone"
```

Verify exact standalone paths with the current workspace layout before copying this literally.

---

# 25. NestJS API Dockerfile — Example

**File:** `apps/api/Dockerfile`

```dockerfile
# syntax=docker/dockerfile:1

FROM node:24-bookworm-slim AS base

ENV PNPM_HOME="/pnpm"
ENV PATH="$PNPM_HOME:$PATH"

RUN corepack enable

WORKDIR /workspace

FROM base AS dependencies

COPY package.json pnpm-lock.yaml pnpm-workspace.yaml ./
COPY apps/api/package.json apps/api/package.json
COPY packages/api-contracts/package.json packages/api-contracts/package.json
COPY packages/qs-engine/package.json packages/qs-engine/package.json
COPY packages/cost-engine/package.json packages/cost-engine/package.json
COPY packages/building-model/package.json packages/building-model/package.json
COPY packages/units/package.json packages/units/package.json

RUN pnpm install --frozen-lockfile

FROM dependencies AS build

COPY . .

RUN pnpm --filter api build

FROM node:24-bookworm-slim AS runtime

ENV NODE_ENV=production
ENV PORT=4000

WORKDIR /app

COPY --from=build /workspace/apps/api/dist ./dist
COPY --from=build /workspace/node_modules ./node_modules
COPY --from=build /workspace/apps/api/package.json ./package.json

USER node

EXPOSE 4000

CMD ["node", "dist/main.js"]
```

Refine dependency copying when the actual workspace build output is known.

---

# 26. Python Worker Dockerfile — Example

**File:** `apps/ai-worker/Dockerfile`

```dockerfile
# syntax=docker/dockerfile:1

FROM python:3.13-slim AS runtime

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

WORKDIR /app

COPY --from=ghcr.io/astral-sh/uv:latest /uv /usr/local/bin/uv

COPY apps/ai-worker/pyproject.toml apps/ai-worker/uv.lock ./

RUN uv sync \
    --frozen \
    --no-dev \
    --no-install-project

COPY apps/ai-worker .

RUN uv sync \
    --frozen \
    --no-dev

RUN useradd --create-home buildora

USER buildora

CMD ["uv", "run", "python", "-m", "buildora_ai.worker"]
```

Pin the uv image version/digest in real production code.

Do not use `latest` in the final production Dockerfile.

---

# 27. `.dockerignore`

**File:** `.dockerignore`

```text
.git
.github
node_modules
**/node_modules
.next
**/.next
dist
**/dist
coverage
.env
.env.*
!.env.example
.venv
**/.venv
__pycache__
.pytest_cache
target
.temporal
playwright-report
test-results
docs
*.log
.DS_Store
```

If documentation is needed at runtime, adjust accordingly.

---

# 28. Container Runtime Rules

Containers must be:

- stateless,
- disposable,
- non-root where feasible,
- configured by environment/secrets,
- able to terminate gracefully.

Do not persist user files on ECS local filesystem.

Temporary files may use container ephemeral disk during processing but must not be authoritative.

---

# 29. Health Endpoints

API:

```text
GET /health/live
GET /health/ready
```

Web:

```text
GET /health
```

or a lightweight deployment health route.

---

# 30. Liveness

Liveness means:

```text
process is alive
```

Do not fail liveness because:

- OpenAI down,
- Stripe down,
- optional supplier API down.

Otherwise ECS may restart healthy application processes due to unrelated providers.

---

# 31. Readiness

Readiness asks whether traffic can be safely served.

API readiness may validate:

- application initialized,
- PostgreSQL available,
- critical configuration valid.

Redis may be included only if the API cannot function without it.

Do not include optional AI providers.

---

# 32. ECS Services

Initial services:

```text
buildora-web
buildora-api
```

Later:

```text
buildora-temporal-worker
buildora-ai-worker
buildora-bim-worker
```

Worker services do not require public ALB routes.

---

# 33. ECS Task Sizing

Start conservatively.

Measure:

- CPU,
- memory,
- event loop latency,
- request duration.

Do not allocate huge containers “for future scale”.

Adjust from metrics.

Web/API can start with small Fargate tasks and scale.

AI/BIM workers may require more memory.

---

# 34. Application Load Balancer

Use ALB for public ECS services.

Routing concept:

```text
Host: app.example
→ web target group

Host: api.example
→ API target group
```

Alternative:

separate ALBs if security/scaling later justify cost.

---

# 35. ALB Health Checks

Web target:

```text
/health
```

API target:

```text
/health/ready
```

Health endpoint must:

- respond quickly,
- not perform expensive queries,
- return stable status.

---

# 36. ECS Deployment Strategy — Initial

Start with ECS rolling deployments.

Configure:

- deployment circuit breaker,
- rollback,
- appropriate minimum/maximum healthy percentage,
- ALB health checks.

AWS ECS supports deployment failure detection using circuit breakers and CloudWatch alarms with rollback to the previous revision.

---

# 37. ECS Deployment Strategy — Later

Use ECS managed blue/green when production risk/traffic warrants it.

Blue/green supports:

- separate old/new revisions,
- traffic shifting,
- bake time,
- CloudWatch alarm rollback,
- lifecycle hooks.

Introduce when:

- user base grows,
- downtime risk increases,
- release validation needs live traffic.

Do not make blue/green a blocker for initial beta.

---

# 38. Graceful Shutdown

NestJS:

```text
enableShutdownHooks()
```

On SIGTERM:

1. stop accepting new work,
2. allow in-flight HTTP requests to finish within timeout,
3. stop Temporal worker polling,
4. finish/abandon activities safely,
5. close DB pool,
6. close Redis,
7. flush telemetry.

Do not abruptly terminate long-running financial/model operations.

---

# 39. Temporal Deployment

Use:

```text
Temporal Cloud
```

for production initially.

Reasons:

- less operations burden,
- workflow durability managed,
- suitable for one-person SaaS.

Local:

```text
temporal server start-dev
```

Production workers run inside Buildora AI compute environment.

---

# 40. Temporal Namespace Strategy

Separate:

```text
buildora-staging
buildora-production
```

Do not run staging and production workflows in the same namespace.

---

# 41. Temporal Task Queue Naming

Example:

```text
buildora-ai-v1
buildora-plan-processing-v1
buildora-bim-v1
buildora-reports-v1
buildora-render-v1
```

Do not create one queue per customer.

Queues reflect worker capability/deployment boundaries.

---

# 42. Temporal Workflow Compatibility

Workflow code must remain deterministic and replay-compatible.

Before changing workflow logic:

- test replay where available,
- preserve compatible workflow histories,
- use Temporal's current worker/workflow versioning mechanisms for incompatible changes.

Do not simply replace workflow code assuming no old workflow exists.

Long-running jobs may still be executing during deployment.

---

# 43. Temporal Worker Deployment

Worker deployment sequence:

```text
deploy new compatible worker
 ↓
verify polling/health
 ↓
allow new traffic/jobs
 ↓
drain/remove old worker
```

Do not stop every worker before the replacement is running.

---

# 44. AI Worker Architecture

Separate:

```text
orchestration
```

from:

```text
heavy compute
```

NestJS AI Gateway handles:

- provider routing,
- entitlements,
- usage/cost,
- tool policy.

Python workers handle:

- CV,
- ML,
- parsing,
- specialized BIM processing.

---

# 45. Render Worker Architecture

Photoreal rendering is later.

Do not run GPU render processes inside API Fargate tasks.

Recommended:

```text
Render request
 ↓
Temporal
 ↓
render queue
 ↓
GPU worker
 ↓
R2 output
```

Compute:

```text
ECS on EC2 GPU
or
AWS Batch GPU
```

Scale to zero when no render jobs where possible.

---

# 46. PostgreSQL — V1

For a solo developer, managed PostgreSQL is mandatory.

Initial recommendation:

```text
Neon
```

because it offers:

- managed Postgres,
- branching,
- preview isolation,
- autoscaling options,
- restore capabilities.

Keep Buildora AI on standard PostgreSQL semantics so migration to:

```text
AWS RDS / Aurora PostgreSQL
```

remains possible later.

---

# 47. PostgreSQL Environment Isolation

Use separate Neon projects or clearly isolated roots:

```text
buildora-staging
buildora-production
```

Production must not be a child/preview branch used for development experimentation.

---

# 48. Preview Database Branches

For selected PRs:

```text
create Neon branch
 ↓
run migrations
 ↓
run E2E
 ↓
destroy branch after PR
```

This is valuable for migration-heavy changes.

Neon supports isolated branches suitable for preview/E2E workflows.

---

# 49. Database Pooling

Application production connection:

```text
pooled endpoint
```

Migration tasks:

```text
direct endpoint
```

where the provider recommends it.

Configuration:

```text
DATABASE_URL
DATABASE_DIRECT_URL
```

Do not use hundreds of connections per ECS task.

---

# 50. Database Migration Pipeline

Migration lifecycle:

```text
PR
 ↓
migration generated
 ↓
empty-DB migration test
 ↓
integration test
 ↓
staging branch/database
 ↓
migration applied
 ↓
application deploy
 ↓
production approval
 ↓
production migration job
 ↓
production application deploy
```

---

# 51. Never Run Migrations on Application Startup

Bad:

```text
container starts
→ drizzle migrate
→ API starts
```

Problems:

- multiple replicas race,
- long migration blocks deployment,
- failed migration creates restart loop.

Use a dedicated one-off migration task/job.

---

# 52. Migration Deployment Pattern

Safe deployment:

```text
1. Expand schema
2. Deploy compatible code
3. Backfill
4. Switch reads/writes
5. Contract old schema later
```

Never perform destructive schema change and dependent app code simultaneously if rollback would be impossible.

---

# 53. Database Backup / PITR

Before paid launch:

- enable provider-supported point-in-time restore,
- configure sufficient history/retention,
- configure snapshots/backup features supported by current plan,
- monitor backup capability.

Neon provides branch restore/point-in-time capabilities and snapshot functionality; exact retention/features depend on current plan and must be verified at deployment time.

---

# 54. Pre-Migration Snapshot

Before high-risk production migration:

```text
create provider snapshot/checkpoint
```

or ensure PITR point is available.

This is not a substitute for writing safe migrations.

---

# 55. Database Restore Drill

At least before beta, then periodically:

1. restore production-like backup into isolated environment,
2. apply migrations if needed,
3. verify project model,
4. verify QS/BOQ/cost,
5. verify billing ledger,
6. verify document references,
7. run integrity checks.

Record:

```text
restore start
restore complete
RTO achieved
issues
```

---

# 56. Managed Redis

Production Redis must be managed.

Recommended:

- AWS ElastiCache / managed Valkey/Redis if backend is AWS-private,
- or another trusted managed TLS Redis where operational simplicity wins.

Do not run a single unmanaged Redis container on ECS as production state infrastructure.

---

# 57. Redis Network Security

Redis:

- private network where possible,
- TLS,
- authentication,
- no public internet exposure.

---

# 58. Redis Failure

Application must treat Redis as non-authoritative.

Design:

```text
Redis unavailable
→ degraded cache/rate-limit/presence behaviour
→ authoritative data still safe
```

For operations where Redis is temporarily required, return controlled dependency error rather than corrupt state.

---

# 59. Secrets Architecture

Production secrets belong in:

```text
AWS Secrets Manager
```

or equivalent managed secret system.

Examples:

```text
DATABASE_URL
REDIS_URL
OPENAI_API_KEY
STRIPE_SECRET_KEY
STRIPE_WEBHOOK_SECRET
CLERK_SECRET_KEY
TEMPORAL credentials
R2_ACCESS_KEY_ID
R2_SECRET_ACCESS_KEY
SENTRY_DSN where server-private
```

---

# 60. Configuration vs Secret

Configuration:

```text
NODE_ENV
PORT
AI_MODEL_FAST
FEATURE_FLAG_DEFAULTS
LOG_LEVEL
```

Secret:

```text
password
API key
private token
signing secret
```

Do not put ordinary configuration into secret manager unnecessarily, but never expose secrets as plain IaC outputs.

---

# 61. ECS Secret Injection

ECS task definitions reference Secrets Manager/SSM.

Application receives secrets at runtime.

Do not bake secrets into:

- image,
- Docker layer,
- build args,
- `.next` browser bundle.

---

# 62. GitHub Actions Cloud Authentication

Use GitHub Actions OIDC to assume AWS IAM roles.

Do not store long-lived:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
```

in GitHub if OIDC can replace them.

GitHub/AWS both support OIDC federation for temporary deployment credentials.

---

# 63. GitHub Environment Protection

Create GitHub environments:

```text
staging
production
```

Production should require:

- manual approval initially,
- protected branch,
- limited deploy role.

---

# 64. CI/CD Repository Files

Recommended:

```text
.github/
└── workflows/
    ├── ci.yml
    ├── build-images.yml
    ├── deploy-staging.yml
    ├── deploy-production.yml
    ├── database-migrate.yml
    ├── security-scan.yml
    └── scheduled-maintenance.yml
```

Do not start with all separate workflows if one readable workflow is easier.

Split once complexity warrants it.

---

# 65. Pull Request CI Pipeline

PR pipeline:

```text
Checkout
 ↓
Node / pnpm setup
 ↓
Install --frozen-lockfile
 ↓
Lint
 ↓
Typecheck
 ↓
Unit tests
 ↓
Integration tests
 ↓
Python checks
 ↓
Rust checks if applicable
 ↓
Migration-from-zero
 ↓
Web build
 ↓
API build
 ↓
Docker build
 ↓
E2E selected flows
 ↓
Security/dependency scan
```

No deployment to production.

---

# 66. Monorepo Path Filtering

Avoid rebuilding everything unnecessarily.

Example:

```text
web-only change
→ web + affected shared packages

API change
→ API + affected packages

building-model package
→ web + API + QS/cost tests
```

Turborepo can compute affected task graph.

Correctness is more important than shaving a few CI minutes.

---

# 67. `ci.yml` Example

**File:** `.github/workflows/ci.yml`

```yaml
name: CI

on:
  pull_request:
  push:
    branches:
      - main

permissions:
  contents: read

jobs:
  typescript:
    runs-on: ubuntu-latest

    services:
      postgres:
        image: postgres:17
        env:
          POSTGRES_DB: buildora_test
          POSTGRES_USER: buildora
          POSTGRES_PASSWORD: buildora_test
        ports:
          - 5432:5432
        options: >-
          --health-cmd="pg_isready -U buildora -d buildora_test"
          --health-interval=5s
          --health-timeout=5s
          --health-retries=10

      redis:
        image: redis:7-alpine
        ports:
          - 6379:6379

    env:
      DATABASE_URL: postgresql://buildora:buildora_test@localhost:5432/buildora_test
      REDIS_URL: redis://localhost:6379

    steps:
      - uses: actions/checkout@v4

      - uses: pnpm/action-setup@v4
        with:
          run_install: false

      - uses: actions/setup-node@v4
        with:
          node-version: 24
          cache: pnpm

      - run: pnpm install --frozen-lockfile

      - run: pnpm lint
      - run: pnpm typecheck
      - run: pnpm test
      - run: pnpm build
```

Pin action versions/digests according to the repository security policy.

Add Python/Rust/E2E jobs as those components appear.

---

# 68. Main Branch Release Pipeline

On merge to `main`:

```text
CI
 ↓
Build images once
 ↓
Push ECR
 ↓
Record image digests
 ↓
Deploy staging
 ↓
Run migration
 ↓
Smoke test
 ↓
E2E critical tests
 ↓
Observe staging
 ↓
Production approval
 ↓
Production pre-check
 ↓
Migration
 ↓
Deploy same image digests
 ↓
Smoke test
 ↓
Observe
```

---

# 69. Build Metadata

Every deployable exposes/records:

```text
git SHA
release version
build timestamp
environment
```

Example internal endpoint:

```text
GET /version
```

Response:

```json
{
  "version": "1.4.0",
  "gitSha": "9f4e1cb...",
  "environment": "production"
}
```

Do not include secrets.

---

# 70. Release Versioning

Use:

```text
SemVer-like release tags
```

for product releases:

```text
v1.0.0
v1.1.0
v1.1.1
```

Git SHA remains the precise artifact identifier.

---

# 71. Staging Deploy

Staging can deploy automatically after successful merge to `main`.

Production should initially require explicit approval.

---

# 72. Production Deploy Frequency

Prefer small releases.

Do not accumulate weeks of unrelated changes into one giant deployment.

Small changes are:

- easier to review,
- easier to rollback,
- easier to diagnose.

---

# 73. Deployment Freeze

During:

- known provider incident,
- unresolved production data issue,
- billing reconciliation issue,

pause unrelated deployments.

Do not deploy through an active incident unless the deployment is the fix and risk is understood.

---

# 74. Production Deployment Workflow Example

**File:** `.github/workflows/deploy-production.yml`

```yaml
name: Deploy Production

on:
  workflow_dispatch:
    inputs:
      git_sha:
        description: Commit SHA already verified in staging
        required: true

permissions:
  contents: read
  id-token: write

jobs:
  deploy:
    environment: production
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ inputs.git_sha }}

      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ vars.AWS_DEPLOY_ROLE_ARN }}
          aws-region: ${{ vars.AWS_REGION }}

      - name: Resolve verified image digests
        run: |
          echo "Resolve image digest for the exact staged SHA here."

      - name: Run production migration task
        run: |
          echo "Trigger one-off migration task and wait for success."

      - name: Deploy ECS services
        run: |
          echo "Update task definitions to verified image digests."

      - name: Smoke test
        run: |
          echo "Run production smoke tests."
```

The production implementation should use tested deployment scripts/actions rather than placeholder shell text.

---

# 75. Deployment IAM

Separate roles:

```text
CI build role
staging deploy role
production deploy role
migration role
```

Production deploy role should have only permissions needed to:

- register task definitions,
- update approved ECS services,
- run approved migration tasks,
- read deployment state.

Do not give GitHub deployment role AWS AdministratorAccess.

---

# 76. Infrastructure as Code

Recommended:

```text
OpenTofu / Terraform-style IaC
```

because Buildora AI spans:

- AWS,
- Cloudflare,
- potentially Neon,
- other managed services.

Use an ADR to finalize Terraform vs OpenTofu.

The architecture is more important than the exact compatible engine.

---

# 77. Infrastructure Repository Layout

```text
infrastructure/
├── modules/
│   ├── network/
│   ├── ecr/
│   ├── ecs-cluster/
│   ├── ecs-service/
│   ├── alb/
│   ├── iam/
│   ├── redis/
│   ├── observability/
│   └── cloudflare/
├── environments/
│   ├── nonprod/
│   └── production/
├── scripts/
└── README.md
```

Do not duplicate entire Terraform trees for every environment if reusable modules can model differences clearly.

---

# 78. IaC State

Use encrypted remote state.

Requirements:

- remote storage,
- locking,
- restricted IAM,
- backup/versioning according to selected backend,
- no secrets exposed in outputs.

Do not commit Terraform/OpenTofu state into Git.

---

# 79. IaC Change Workflow

PR:

```text
fmt
validate
lint/security scan
plan
```

Production:

```text
review plan
manual approval
apply
```

Do not auto-apply infrastructure changes to production from every merge.

---

# 80. Cloud Resource Tagging

Tag every resource:

```text
Application=Buildora AI
Environment=Production
ManagedBy=IaC
Service=API
Owner=Buildora AI
```

Add cost-allocation tags where supported.

---

# 81. Networking

AWS VPC:

```text
public subnets
→ ALB

private subnets
→ ECS tasks
→ Redis
```

API/web ECS tasks should normally not receive public IP addresses.

---

# 82. Outbound Internet

Private ECS tasks need outbound access for:

- AI provider,
- Stripe,
- auth provider,
- R2 API,
- Temporal Cloud.

Use the chosen AWS egress architecture.

Be aware NAT Gateways can materially affect cost.

For a low-volume startup, evaluate:

- NAT Gateway cost,
- architecture alternatives,
- VPC endpoints for AWS services where useful.

Do not optimize networking complexity before measuring the cost.

---

# 83. Security Groups

ALB:

```text
443 from trusted edge/origin policy
```

ECS:

```text
only ALB → web/API ports
```

Redis:

```text
only ECS security group
```

No public Redis/Postgres ingress.

---

# 84. Database Network

If Neon is external managed Postgres:

- TLS required,
- provider access controls if available,
- strong credentials,
- pooled endpoint.

If later moving to RDS:

```text
private subnets
security group only from ECS
```

---

# 85. Secret Rotation

Document rotation for:

- AI provider key,
- Stripe secret,
- auth secret,
- R2 key,
- DB credential,
- Temporal credential.

Rotation should not require code changes.

Use overlapping/dual-key period where provider supports it.

---

# 86. Feature Flags and Deployments

Deploying code does not always mean enabling feature.

Use feature flags for high-risk modules:

```text
AI_DESIGN
PLAN_RECOGNITION
IFC_IMPORT
BILLING_TOPUPS
PHOTOREAL_RENDERING
```

Release:

```text
deploy dark
 ↓
verify
 ↓
enable internal tenant
 ↓
beta tenants
 ↓
broader rollout
```

---

# 87. Database Feature Compatibility

Feature flag does not solve incompatible DB rollback.

Always ensure database migration strategy is backward-compatible during rollout.

---

# 88. Rollback Strategy

Application rollback:

```text
previous ECS task definition/image digest
```

Database rollback:

prefer:

```text
forward-compatible migration
+
forward fix
```

rather than automatically reversing data migrations.

High-risk destructive incidents may require:

```text
PITR/snapshot restore
```

which is an incident-level action.

---

# 89. Deployment Rollback Trigger

Rollback if:

- health checks fail,
- error rate spikes materially,
- critical user workflow fails,
- payment flow broken,
- model save/reload broken,
- migration incompatibility found.

Do not roll back solely because of a non-critical cosmetic bug.

---

# 90. ECS Automatic Rollback

Use:

- deployment circuit breaker,
- CloudWatch alarms where useful.

Alarms can monitor:

```text
HTTP 5xx
target health
latency
```

Use carefully to avoid rollback loops due to unrelated external incidents.

---

# 91. Smoke Tests

After deployment:

```text
GET web
GET API live
GET API ready
auth handshake
DB read
basic project read
```

Staging smoke additionally:

```text
create test project
update
delete/archive
```

Production smoke uses dedicated safe synthetic tenant/data.

Do not mutate real customer projects during smoke tests.

---

# 92. Critical E2E Tests

Staging before production:

```text
Sign in
Create project
Draw/save/reload
Open 3D
Run QS
Open BOQ
Calculate cost
AI read-only request
Upload test file
Billing test-mode flow
```

Run a smaller non-destructive subset in production.

---

# 93. Canary / Blue-Green Future

When traffic warrants:

```text
new revision receives small traffic
 ↓
observe
 ↓
increase
 ↓
full cutover
```

Suitable for:

- API,
- web.

Workers require workflow/task compatibility rather than HTTP traffic shifting.

---

# 94. Observability Architecture

Use:

```text
Sentry
+
OpenTelemetry
+
Cloud provider metrics/logs
```

Responsibilities:

Sentry:

- application exceptions,
- release tracking,
- frontend/browser errors,
- backend errors.

OpenTelemetry:

- distributed traces,
- service metrics,
- context propagation.

CloudWatch/AWS:

- ECS/ALB infrastructure metrics/log transport where applicable.

---

# 95. OpenTelemetry Scope

Instrument server-side first.

Current OpenTelemetry JavaScript status:

```text
Traces  stable
Metrics stable
Logs    still evolving/development
```

Therefore:

- use OTel strongly for traces/metrics,
- use structured application logger for logs,
- do not depend on browser OTel for critical production diagnostics initially.

---

# 96. Trace Propagation

Trace across:

```text
Browser/API request
 ↓
NestJS
 ↓
PostgreSQL
 ↓
Redis
 ↓
Temporal workflow/activity
 ↓
AI provider
```

Record application identifiers as span attributes carefully:

```text
organizationId
projectId
workflowId
aiJobId
```

Do not put:

- full prompts,
- document text,
- secrets,

into trace attributes.

---

# 97. Structured Logging

Production log example:

```json
{
  "level": "info",
  "message": "Model command committed",
  "service": "api",
  "environment": "production",
  "release": "9f4e1cb",
  "traceId": "...",
  "organizationId": "...",
  "projectId": "...",
  "modelVersion": 43
}
```

---

# 98. Log Levels

```text
DEBUG → development / temporary diagnosis
INFO  → meaningful lifecycle
WARN  → recoverable anomaly
ERROR → failed operation
FATAL → process cannot safely continue
```

Do not log every ordinary request at huge verbosity if cost/noise becomes high.

---

# 99. Never Log

- passwords,
- bearer tokens,
- database passwords,
- Stripe secrets,
- AI keys,
- full private documents,
- presigned URLs,
- payment details,
- full raw AI prompts by default.

---

# 100. Sentry Environments

Separate:

```text
development
staging
production
```

Attach release:

```text
git SHA / release version
```

so error regression can be linked to deployment.

---

# 101. Source Maps

Upload source maps securely during CI.

Do not expose private source maps publicly unless intentionally configured.

---

# 102. Core Infrastructure Metrics

Monitor:

## ECS

- CPU
- memory
- task restarts
- desired/running count.

## ALB

- request count
- target response time
- 4xx
- 5xx
- unhealthy targets.

## Database

- connections
- query latency
- storage
- compute utilization
- slow queries.

## Redis

- memory
- connections
- latency
- evictions.

## Temporal

- workflow failures
- activity failures
- task queue schedule-to-start latency
- worker polling health.

---

# 103. Product/Domain Metrics

Monitor:

```text
project open failures
model commit failures
model conflicts
QS failures
cost calculation failures
upload processing failures
IFC failures
AI job failures
billing reservation failures
webhook failures
```

Infrastructure green does not mean product healthy.

---

# 104. AI Metrics

Monitor:

```text
requests/model
tokens/model
cached token ratio
reasoning token usage
AI cost
AI latency
tool failures
escalation rate
cost/user
cost/org
cost/plan
```

Alerts:

```text
unexpected spend spike
single job cost anomaly
provider 5xx spike
```

---

# 105. Billing Metrics

Monitor:

```text
checkout failures
payment failures
webhook lag
duplicate-event attempts
credit settlement errors
negative-balance invariant
reconciliation mismatches
```

Billing alerts are release-critical.

---

# 106. Suggested Initial SLOs

Do not overpromise public SLA before enough evidence.

Internal targets can begin as:

```text
API availability target       99.9%
Core API p95                  < 500 ms for ordinary CRUD/read
Model command p95             measured target after baseline
Critical webhook processing   < defined minutes
```

Do not apply the same latency SLO to:

- AI generation,
- BIM import,
- rendering.

Those are asynchronous workflows.

---

# 107. Error Budget

Once real SLOs exist:

```text
availability target
→ allowable failure budget
```

If error budget is exhausted:

prioritize reliability over new features.

Do not invent complex SRE process before user volume requires it.

---

# 108. Alerting Philosophy

Alert on:

```text
user-impacting symptoms
```

not every metric fluctuation.

Page/high priority:

- production unavailable,
- database inaccessible,
- billing processing broken,
- data-integrity invariant failure.

Lower priority:

- one failed non-critical render,
- small CPU spike.

---

# 109. Alert Channels

Initial:

```text
email
mobile push / incident app
```

As team grows:

```text
Slack/Teams
PagerDuty/Opsgenie equivalent
```

A solo developer should still have a reliable urgent channel.

---

# 110. Production Dashboards

Create dashboards:

## Platform

- web/API health,
- latency,
- error rate,
- ECS tasks,
- DB,
- Redis.

## AI

- jobs,
- latency,
- model distribution,
- tokens/cost.

## Billing

- checkout,
- webhooks,
- payment failures,
- credits.

## Workflows

- plan recognition,
- IFC,
- reports,
- renders.

---

# 111. Security Pipeline

CI security:

- dependency audit,
- secret scan,
- container scan,
- IaC scan,
- code analysis as practical.

Do not block all development on noisy low-confidence scanners.

Define severity policy.

---

# 112. Dependency Updates

Use automated dependency PRs:

```text
Dependabot / Renovate
```

Policy:

- patch/minor reviewed regularly,
- major upgrades intentionally planned,
- framework upgrades tested in staging.

Do not allow AI coding agents to upgrade unrelated core dependencies opportunistically.

---

# 113. SBOM

Before larger/enterprise customers, generate Software Bill of Materials for releases.

Possible standard:

```text
CycloneDX / SPDX
```

Container/image tooling can generate it.

---

# 114. Container Signing — Later

As supply-chain maturity increases:

- sign container images,
- verify signatures in deployment.

Do not make this a blocker for MVP if it materially delays delivery, but design CI so immutable digests are already used.

---

# 115. Git Protection

`main`:

- no force push,
- required CI,
- PR for significant changes,
- protected production workflow.

As a solo developer, emergency direct commit may be technically possible but should be exceptional and documented.

---

# 116. Branch Strategy

Keep simple:

```text
main
feature/*
fix/*
```

No permanent complex GitFlow unless team/process eventually requires it.

---

# 117. Environment Configuration

Use explicit environment names:

```text
development
test
staging
production
```

Never infer production from hostname alone.

---

# 118. Feature Configuration

Provider model IDs, rates, credit rules, limits should be:

- environment/configuration,
- database plan/rate records,

not Docker-image constants where business admins may need changes.

---

# 119. AI Provider Kill Switch

Operational control:

```text
AI_PROVIDER_ENABLED
AI_DESIGN_ENABLED
AI_EXPERT_ROUTING_ENABLED
```

If provider cost/outage becomes dangerous:

disable expensive optional actions while keeping deterministic product operational.

---

# 120. Render Kill Switch

```text
RENDERING_ENABLED
```

Useful if GPU queue/provider fails or runaway cost appears.

---

# 121. Upload Kill Switch

If malicious-file incident occurs:

```text
PLAN_UPLOAD_ENABLED=false
```

while existing project access continues.

---

# 122. Backup Architecture

Authoritative systems:

```text
PostgreSQL
Object Storage
Temporal workflow state
Billing/provider state
```

Each needs a recovery strategy.

---

# 123. PostgreSQL Backup

Use provider:

- PITR/history,
- snapshots,
- retention appropriate to plan.

Track backup capability as production monitoring/checklist.

---

# 124. Object Storage Recovery

R2 provides high durability, but durability is not the same as protection from accidental application deletion.

Use:

- application soft-delete/finalization workflow,
- lifecycle policies,
- bucket locks/retention controls where product/legal needs justify them,
- optional independent backup/export strategy for critical customer originals.

Do not rely solely on hardware durability to recover application mistakes.

---

# 125. File Deletion Grace

Recommended:

```text
DB marks object pending deletion
 ↓
grace period
 ↓
background physical deletion
```

Especially for:

- original plan files,
- BIM files,
- reports.

Exact retention depends on privacy/product policy.

---

# 126. Disaster Recovery Objectives

Before paid launch define:

```text
RPO
RTO
```

Example initial internal targets could be:

```text
RPO: minutes-hours depending provider backup window
RTO: few hours
```

Do not advertise a contractual SLA until tested.

---

# 127. Disaster Scenario — Bad Database Migration

Response:

1. stop deployment,
2. disable affected writes if required,
3. assess schema/data,
4. forward-fix if safe,
5. restore snapshot/PITR only if necessary,
6. reconcile external side effects,
7. validate,
8. reopen traffic.

Do not blindly run migration `down` on production data.

---

# 128. Disaster Scenario — Database Outage

Application:

- fail readiness,
- stop mutations,
- show controlled maintenance/dependency error.

Do not buffer authoritative writes only in Redis.

After recovery:

- verify connections,
- integrity checks,
- process queued safe workflows.

---

# 129. Disaster Scenario — Redis Outage

Expected:

- cache misses,
- rate-limit/presence degradation.

Authoritative data remains in PostgreSQL.

After recovery:

- repopulate cache naturally,
- no restore of core project state needed.

---

# 130. Disaster Scenario — R2 Outage

Existing Building Model/BOQ remains accessible if it does not require blob fetch.

File-dependent operations:

```text
temporarily unavailable
```

Do not corrupt DB metadata by marking objects permanently failed because provider is temporarily unavailable.

---

# 131. Disaster Scenario — OpenAI Outage

Disable/queue/retry AI depending operation.

Keep:

- project editing,
- 2D/3D,
- QS,
- BOQ,
- cost.

Reserved credits for failed jobs are released according to billing policy.

---

# 132. Disaster Scenario — Stripe Outage

Existing confirmed entitlements continue from internal state.

Disable:

- new checkout,
- top-ups,
- plan changes.

Webhook reconciliation catches delayed changes later.

---

# 133. Disaster Scenario — Temporal Cloud Issue

Do not start duplicate workflows blindly.

Use stable workflow IDs/idempotency.

When service resumes:

- workers reconnect,
- workflows continue.

User-facing processing jobs remain in controlled state.

---

# 134. Incident Severity

Suggested:

## SEV-1

- production unavailable,
- cross-tenant data exposure,
- financial double charging,
- irreversible data corruption.

## SEV-2

- major workflow unavailable,
- large user segment affected,
- AI/render spending runaway.

## SEV-3

- limited feature issue,
- workaround exists.

## SEV-4

- cosmetic/minor.

---

# 135. Incident Workflow

```text
Detect
 ↓
Declare severity
 ↓
Stabilize
 ↓
Communicate
 ↓
Investigate
 ↓
Fix / Rollback
 ↓
Validate
 ↓
Resolve
 ↓
Postmortem
```

Do not focus on perfect root cause before stabilizing customer impact.

---

# 136. Incident Runbook Template

```text
Incident:
Start:
Severity:
Affected services:
Customer impact:

Immediate mitigations:
- ...

Current hypothesis:
- ...

Actions:
- ...

Rollback decision:
- ...

Data integrity:
- ...

Billing impact:
- ...

Resolution:
- ...

Follow-up:
- ...
```

---

# 137. Postmortem

For SEV-1/SEV-2:

Document:

- timeline,
- impact,
- detection,
- root cause,
- contributing factors,
- what went well,
- what failed,
- action items.

Do not turn postmortems into blame documents.

---

# 138. Deployment Failure Runbook

If new ECS tasks fail health:

1. ECS circuit breaker/rollback where configured,
2. inspect application logs,
3. check secrets/config,
4. check migration compatibility,
5. inspect ALB target health,
6. compare previous task definition,
7. restore previous image/task revision.

---

# 139. Database Migration Runbook

Before:

- CI migration test,
- staging migration,
- backup/PITR assurance,
- estimate lock/runtime.

During:

- run one migration job,
- capture logs,
- block application deploy if failed.

After:

- verify schema,
- smoke query,
- deploy app.

---

# 140. Secrets Incident Runbook

If a secret leaks:

1. revoke/rotate immediately,
2. identify exposure window,
3. inspect provider logs,
4. invalidate sessions/tokens as needed,
5. update secret manager,
6. redeploy/restart services if needed,
7. document incident.

Do not wait for normal deployment cadence.

---

# 141. Scaling Strategy — Web/API

First scale:

```text
horizontal ECS tasks
```

based on:

- CPU,
- memory,
- ALB request metrics.

Do not vertically scale forever.

---

# 142. Minimum Production Replica Count

For paid production, consider at least:

```text
2 API tasks
2 Web tasks
```

across availability zones if cost permits.

This avoids a single task being the entire service.

Early low-cost beta may deliberately run one task but accepts reduced availability.

Document the trade-off.

---

# 143. Autoscaling

Example policy:

API:

```text
min 2
max N
target CPU/memory/request load
```

Web:

similar.

Workers:

scale from:

- queue backlog,
- Temporal schedule-to-start latency,
- CPU/memory.

---

# 144. Worker Scaling

Do not autoscale workers only by CPU if they are waiting on external APIs.

Better signals may include:

```text
Temporal task queue latency
outstanding jobs
processing duration
```

---

# 145. AI Provider Rate Limits

Autoscaling workers cannot overcome provider rate limits.

AI Gateway needs:

- concurrency control,
- provider quotas,
- backpressure,
- retries.

Do not spawn hundreds of workers that immediately hit provider 429s.

---

# 146. Database Scaling

Order:

```text
optimize queries/indexes
 ↓
adjust compute/autoscaling
 ↓
connection pooling
 ↓
read replica for read-heavy workload
 ↓
specialized analytics store
 ↓
only later consider sharding
```

Do not shard early.

---

# 147. Redis Scaling

Track:

- memory,
- eviction,
- latency.

Scale managed instance/cluster when evidence requires it.

Do not use Redis Cluster complexity before necessary.

---

# 148. Object Storage Scaling

R2 handles object scaling.

Application concerns:

- prefix/key design,
- upload concurrency,
- lifecycle,
- processing worker throughput.

Do not create one bucket per customer.

---

# 149. Large Model Scaling

When project models become large:

- send deltas,
- compress payloads,
- binary formats where useful,
- Fragments for BIM,
- object-store large derivatives,
- incremental QS/cost recalculation.

This is application architecture, not a reason to add Kubernetes.

---

# 150. CDN Strategy

Use Cloudflare CDN for immutable/static content.

Do not put private project objects behind public CDN cache without proper signed/access strategy.

---

# 151. Cost Management

Tag cloud resources.

Track monthly:

```text
AWS ECS
ALB
NAT/egress
Redis
Neon
R2
Temporal
Sentry
AI providers
Stripe fees
```

AI is likely a major variable cost but infrastructure cost can still creep through idle resources.

---

# 152. AWS Cost Watch

Common startup cost traps:

- NAT Gateway,
- oversized Fargate tasks,
- idle GPU instances,
- duplicate ALBs,
- verbose CloudWatch logs,
- oversized Redis.

Review monthly.

---

# 153. GPU Cost Control

GPU workers:

```text
scale to zero
queue work
use spot where retry-safe
on-demand fallback for SLA if needed
```

Never keep high-end GPU machines idle for occasional rendering.

---

# 154. Log Cost Control

Set retention.

Example:

```text
staging logs → shorter
production application logs → policy-based
audit records → DB retention policy
```

Do not retain high-volume debug logs indefinitely.

---

# 155. AI Cost Control

DevOps needs alarms from billing/AI usage:

```text
daily provider cost
hourly anomaly
single job cost
cost by model
```

Provider billing dashboards alone are too delayed for product guardrails.

---

# 156. Staging Cost Control

Staging:

- lower task counts,
- scale down when safe,
- smaller DB compute,
- no permanent GPU.

Do not make staging so different that production behavior cannot be validated.

---

# 157. Preview Cost Control

Preview environments:

- created only when useful,
- automatic expiration,
- database branches removed,
- no long-running workers by default.

---

# 158. Production Security — Edge

Cloudflare:

- TLS,
- WAF managed rules,
- rate limits,
- bot controls where useful.

Do not depend on WAF to fix application authorization bugs.

---

# 159. Production Security — Application

- secure headers,
- session security,
- CORS minimal,
- request body limits,
- upload limits,
- auth,
- authorization,
- tenant scoping,
- RFC 9457 safe errors.

---

# 160. Production Security — Infrastructure

- least privilege IAM,
- no public DB/Redis,
- secret manager,
- OIDC CI,
- MFA on cloud admin account,
- CloudTrail/audit logging,
- separate prod/nonprod where feasible.

---

# 161. AWS Root Account

- MFA,
- no everyday use,
- no root access keys,
- protected recovery details.

Use IAM/SSO administrative identity for normal work.

---

# 162. Break-Glass Access

Create documented emergency admin access.

Use only for:

- identity outage,
- severe production recovery.

Audit its use.

---

# 163. Dependency Egress

Workers/API should only call required external providers.

At maturity consider egress policy/domain controls.

Do not over-engineer network firewalls during MVP if they become brittle.

---

# 164. Supply Chain Security

CI:

- pinned lockfiles,
- frozen installs,
- trusted package registries,
- dependency scanning,
- immutable container images.

Do not run unknown third-party scripts in production build blindly.

---

# 165. CI Secrets

PRs from untrusted forks should not receive production/staging secrets.

GitHub environment secrets/permissions must reflect trust level.

---

# 166. Database Credentials

Use different credentials for:

```text
application
migration
admin
```

Rotate periodically.

Do not expose direct production DB URL to frontend or client-side tools.

---

# 167. Production Data Access

As solo developer, you may need support access.

Still use:

- audited admin access,
- read-only role for investigation where possible,
- explicit temporary elevation for writes.

Avoid routine manual editing of customer rows.

---

# 168. Manual Production SQL

Emergency-only.

If executed:

- ticket/incident reference,
- exact command captured,
- backup/transaction,
- peer/AI review where practical,
- follow-up migration/code fix.

Do not make manual SQL part of normal operations.

---

# 169. Cron / Scheduled Jobs

Prefer:

- Temporal schedules,
- EventBridge schedules,
- managed scheduled tasks.

Do not rely on a cron process inside one API container.

Jobs:

- reconciliation,
- credit expiry,
- cleanup,
- integrity checks,
- notifications.

---

# 170. Scheduled Job Idempotency

Every scheduled financial/cleanup job must tolerate rerun.

Example:

```text
monthly credit grant
```

has database uniqueness guard.

---

# 171. Maintenance Tasks

One-off maintenance runs as:

```text
ECS task
```

or controlled admin workflow.

Examples:

- data backfill,
- re-embedding documents,
- cost-rate import,
- integrity reconciliation.

---

# 172. Admin Scripts

Put maintained scripts in:

```text
scripts/
```

or dedicated internal CLI.

Scripts must:

- require environment explicitly,
- default against accidental production,
- validate tenant/resource,
- log operation.

---

# 173. Environment Safety

Dangerous scripts require:

```text
--environment production
--confirm <identifier>
```

Do not infer production from current shell silently.

---

# 174. Data Backfill Deployment

Large data change:

```text
add new schema
 ↓
deploy compatible application
 ↓
run resumable batch backfill
 ↓
monitor
 ↓
switch read path
 ↓
remove old field later
```

---

# 175. Application Compatibility Window

During rolling deployment, old and new tasks may run simultaneously.

Therefore releases must maintain compatibility during the deployment window.

This is essential for:

- DB schema,
- event payloads,
- Redis keys,
- APIs.

---

# 176. Internal Event Versioning

Outbox/event payloads include:

```text
event_version
```

Workers must handle active versions during transition.

Do not change an event schema in place if queued messages may still exist.

---

# 177. Temporal Activity Payload Versioning

Long-running workflows may call activities after a deployment.

Keep payloads backward compatible or use versioned activities/workflows.

---

# 178. API Compatibility

Frontend and API may not deploy at exactly the same millisecond.

Keep at least one-release compatibility for critical interfaces where rolling independently.

Alternatively deploy them as a coordinated release.

---

# 179. Frontend Deployment

Next.js web image contains only public build-time config intended for browser.

Sensitive server config is injected at runtime if server-side Next.js routes require it.

Be careful:

```text
NEXT_PUBLIC_*
```

is browser-visible.

---

# 180. Static Asset Caching

Next hashed static assets:

```text
cache long
immutable
```

HTML/dynamic application:

```text
no unintended shared cache
```

---

# 181. Browser Error Monitoring

Sentry captures:

- unhandled JS errors,
- route failures,
- editor crashes,
- 3D WebGL context issues where instrumented.

Do not send full Building Model/customer documents as error payloads.

---

# 182. Source Map Security

Sentry upload during CI.

Prefer hidden/private source maps rather than public hosting.

---

# 183. 2D/3D Performance Telemetry

Later track:

```text
project element count
load duration
FPS bucket
memory
renderer initialization
context loss
```

Use aggregated metrics.

Do not record detailed customer geometry.

---

# 184. Service Ownership

Even as one person, define ownership in docs.

```text
web         → frontend
api         → SaaS/domain orchestration
ai-worker   → CV/ML
bim-worker  → BIM
render      → rendering
```

This prevents architecture drift when AI agents generate code.

---

# 185. Runbook Directory

Recommended:

```text
docs/operations/
├── README.md
├── deploy.md
├── rollback.md
├── database-migration.md
├── database-restore.md
├── ai-provider-outage.md
├── stripe-outage.md
├── redis-outage.md
├── object-storage-outage.md
├── temporal-outage.md
├── secret-rotation.md
└── incident-response.md
```

---

# 186. Production Readiness Review

Before first paid launch review:

## Infrastructure

- [ ] IaC.
- [ ] isolated production environment.
- [ ] ECS web/API.
- [ ] worker deployment.
- [ ] production DB.
- [ ] managed Redis.
- [ ] R2.
- [ ] Temporal Cloud.
- [ ] Cloudflare.

## CI/CD

- [ ] immutable artifacts.
- [ ] staging promotion.
- [ ] production approval.
- [ ] OIDC AWS auth.
- [ ] migration job.
- [ ] rollback.

## Security

- [ ] least privilege.
- [ ] private DB/Redis.
- [ ] secret manager.
- [ ] WAF.
- [ ] vulnerability scans.
- [ ] dependency scans.

## Reliability

- [ ] health endpoints.
- [ ] minimum replicas decision.
- [ ] deployment rollback.
- [ ] DB backup.
- [ ] restore test.
- [ ] incident runbooks.

## Observability

- [ ] Sentry.
- [ ] OTel.
- [ ] logs.
- [ ] platform dashboard.
- [ ] AI dashboard.
- [ ] billing dashboard.
- [ ] alerts.

---

# 187. Production Launch Checklist

## Pre-Deployment

- [ ] CI green.
- [ ] staging green.
- [ ] E2E green.
- [ ] migration reviewed.
- [ ] backup/PITR confirmed.
- [ ] image scan reviewed.
- [ ] release version created.
- [ ] same image digest staged.
- [ ] feature flags configured.
- [ ] AI spend controls set.

## Deployment

- [ ] run migration.
- [ ] verify migration.
- [ ] deploy API.
- [ ] deploy web.
- [ ] deploy workers.
- [ ] verify ECS target health.
- [ ] verify Temporal worker polling.

## Smoke

- [ ] web opens.
- [ ] sign-in.
- [ ] project opens.
- [ ] model read.
- [ ] save synthetic project.
- [ ] 3D opens.
- [ ] QS.
- [ ] billing status.
- [ ] AI low-cost smoke.
- [ ] file upload smoke.

## Observe

- [ ] error rate.
- [ ] p95 latency.
- [ ] DB connections.
- [ ] 5xx.
- [ ] worker failures.
- [ ] AI errors.
- [ ] billing errors.

---

# 188. Release Rollback Checklist

If rollback required:

- [ ] stop further promotion.
- [ ] identify whether DB schema remains backward compatible.
- [ ] revert ECS task definition/images.
- [ ] confirm old task health.
- [ ] run smoke.
- [ ] verify billing/model integrity.
- [ ] keep new DB expansion if safe.
- [ ] create forward-fix ticket/ADR.

Do not automatically restore DB merely because application code rolled back.

---

# 189. Database Restore Checklist

Emergency only:

- [ ] declare incident.
- [ ] stop risky writes.
- [ ] identify restore timestamp/snapshot.
- [ ] preserve current state if possible.
- [ ] restore into preview/isolation first where feasible.
- [ ] validate target state.
- [ ] perform approved restore.
- [ ] restart/repoint services.
- [ ] reconcile Stripe.
- [ ] reconcile credit ledger.
- [ ] reconcile Temporal jobs.
- [ ] reconcile object storage.
- [ ] integrity tests.
- [ ] reopen traffic.

---

# 190. Scaling Gates

Do not introduce architecture based on user-count guesses.

## Add read replica when

- DB read load is material,
- reporting/admin competes with writes.

## Add Rust/WASM when

- measured geometry hotspots exist.

## Add Go service when

- profiling identifies a throughput service where TypeScript is insufficient.

## Add Kubernetes when

- ECS operational limits genuinely hurt development/scale.

## Add multi-region when

- latency/residency/availability requirements justify complexity.

---

# 191. Kubernetes Decision Checklist

Do **not** adopt Kubernetes unless several are true:

- [ ] many independent services.
- [ ] complex service autoscaling.
- [ ] multiple GPU pools.
- [ ] custom scheduling.
- [ ] platform team/operational capacity.
- [ ] ECS limitations documented.
- [ ] cost justified.
- [ ] migration plan.

One million users by itself is not an automatic reason to use Kubernetes.

---

# 192. Multi-Region Decision Checklist

Only after:

- [ ] clear latency requirement.
- [ ] data residency requirement.
- [ ] tested single-region DR.
- [ ] database regional strategy.
- [ ] billing consistency strategy.
- [ ] Building Model write-consistency strategy.

Do not deploy active/active databases casually.

---

# 193. Production Database Migration to RDS/Aurora — Future

Neon → RDS/Aurora may be justified by:

- AWS private networking requirements,
- enterprise procurement,
- compliance/data residency,
- predictable sustained workload,
- custom replication/operations needs.

Because Buildora AI uses standard PostgreSQL, migration should remain feasible.

Do not use provider-specific database features in core domain without an ADR.

---

# 194. Backup Independence — Mature Stage

For higher-value enterprise customers, consider independent logical backups/export in addition to provider PITR.

Example:

```text
periodic pg_dump / managed export
→ encrypted backup storage
```

Test restore.

Do not assume provider-level backup alone satisfies every enterprise requirement.

---

# 195. Data Residency — Future

If Buildora AI sells into regulated/enterprise markets:

- choose customer home region,
- regional object storage,
- regional DB,
- regional processing.

AI/document provider data residency must also be reviewed.

This is a product/legal architecture project, not a simple DNS change.

---

# 196. Compliance Readiness

As business matures, operational controls needed for:

```text
SOC 2
ISO 27001
```

may include:

- access reviews,
- change control,
- incident logs,
- audit evidence,
- vulnerability management,
- backup evidence,
- vendor inventory.

Do not claim certifications before obtaining them.

---

# 197. Vendor Inventory

Maintain:

```text
AWS
Cloudflare
Neon
Temporal
OpenAI
Stripe
Clerk
Sentry
GitHub
```

Record:

- purpose,
- data handled,
- region,
- credentials owner,
- outage impact,
- termination/export plan.

---

# 198. Vendor Exit Strategy

Critical providers should have an abstraction/exit route.

Examples:

```text
Postgres → standard PostgreSQL
R2 → S3 API compatible abstraction
OpenAI → AI provider adapter
Stripe → billing provider boundary
Clerk → internal user IDs + identity mapping
```

Do not build unnecessary multi-provider failover now.

Maintain portability.

---

# 199. DevOps Documentation Rules

Infrastructure changes require documentation update.

If deployment behaviour changes:

```text
docs/operations/
```

must be updated in same PR.

Do not leave runbooks describing obsolete architecture.

---

# 200. Codex DevOps Instruction

Add to `AGENTS.md`:

```text
Before modifying infrastructure, deployment workflows, Dockerfiles,
cloud configuration, migrations, production environment settings,
observability, secrets, backups or CI/CD, read:

docs/architecture/27_DevOps_Environments_and_Deployment.md

Do not:
- introduce Kubernetes without an ADR,
- deploy mutable "latest" images,
- run production migrations at app startup,
- commit secrets,
- add public database/Redis access,
- create long-lived AWS credentials for GitHub Actions,
- bypass staging for production releases,
- make provider-specific infrastructure assumptions silently.
```

---

# 201. Recommended Repository Layout

```text
buildora/
├── apps/
│   ├── web/
│   ├── api/
│   ├── ai-worker/
│   ├── bim-worker/
│   └── render-worker/
│
├── infrastructure/
│   ├── modules/
│   ├── environments/
│   ├── scripts/
│   └── README.md
│
├── .github/
│   └── workflows/
│       ├── ci.yml
│       ├── deploy-staging.yml
│       └── deploy-production.yml
│
├── scripts/
│   ├── smoke-test.sh
│   ├── migration-status.sh
│   └── integrity-check.sh
│
└── docs/
    ├── architecture/
    │   └── 27_DevOps_Environments_and_Deployment.md
    └── operations/
        ├── deploy.md
        ├── rollback.md
        ├── database-restore.md
        └── incident-response.md
```

---

# 202. Initial Implementation Phases

## DevOps Phase A — Local / CI

Implement first:

- [ ] Dockerfiles.
- [ ] local Compose.
- [ ] GitHub CI.
- [ ] frozen installs.
- [ ] build/test.
- [ ] migration testing.

## DevOps Phase B — Staging

- [ ] ECR.
- [ ] ECS cluster.
- [ ] ALB.
- [ ] Cloudflare DNS.
- [ ] staging secrets.
- [ ] staging DB.
- [ ] staging Redis.
- [ ] staging R2.
- [ ] Temporal staging.
- [ ] automated staging deployment.

## DevOps Phase C — Production

- [ ] separate production environment/account.
- [ ] production resources.
- [ ] GitHub protected environment.
- [ ] OIDC.
- [ ] backup.
- [ ] alerts.
- [ ] production deployment.
- [ ] rollback.

## DevOps Phase D — Scale

Only after usage:

- [ ] worker autoscaling.
- [ ] blue/green.
- [ ] GPU compute.
- [ ] read replica.
- [ ] advanced observability.
- [ ] enterprise controls.

---

# 203. Day-1 Production Minimum

For a low-volume paid beta:

```text
Cloudflare
1–2 Web ECS tasks
1–2 API ECS tasks
managed PostgreSQL
managed Redis
R2
Temporal Cloud
Sentry
GitHub Actions
```

If budget is tight, one task per service can be accepted temporarily with documented reduced availability.

Do not pretend a single-replica beta has high availability.

---

# 204. Day-2 Production Operations

After launch:

Daily:

- error dashboard,
- billing/webhook failures,
- AI spend,
- workflow failures.

Weekly:

- dependency vulnerabilities,
- slow queries,
- ECS utilization,
- DB connections,
- user-impacting incidents.

Monthly:

- restore/backup health,
- cloud cost,
- AI cost,
- capacity,
- access review,
- old secrets,
- stale preview resources.

---

# 205. Operational Automation

Automate:

```text
preview cleanup
old image cleanup
expired temporary objects
credit reconciliation
billing reconciliation
integrity checks
backup check
dependency update PRs
```

Do not automate destructive production actions without guardrails.

---

# 206. ECR Image Retention

Keep:

- current production,
- recent rollback releases,
- selected milestone releases.

Expire old unreferenced images according to lifecycle policy.

Never delete the only image for the active task definition.

---

# 207. Database Branch Cleanup

Preview DB branches should:

- include PR number,
- include expiry metadata,
- be removed on PR close,
- have periodic orphan cleanup.

Prevent surprise managed-database cost.

---

# 208. Cloudflare R2 Cleanup

Temporary processing prefix example:

```text
tmp/
```

Apply lifecycle expiration.

Authoritative originals:

```text
org/{orgId}/project/{projectId}/documents/...
```

are controlled by application deletion/retention policy, not generic temporary lifecycle.

---

# 209. Deployment Change Categories

## Low Risk

- UI text,
- CSS,
- non-critical frontend.

## Medium Risk

- API code,
- ordinary schema additive migration,
- worker logic.

## High Risk

- model persistence,
- billing,
- authentication,
- destructive migration,
- credit ledger,
- tenant permissions.

High-risk releases require stronger staging/smoke/manual review.

---

# 210. Production Approval Checklist for High-Risk Changes

- [ ] Claude architecture review.
- [ ] Codex implementation tests.
- [ ] tenant tests.
- [ ] migration test.
- [ ] staging data test.
- [ ] rollback plan.
- [ ] backup/PITR verified.
- [ ] manual production approval.

---

# 211. Maintenance Windows

Normal Buildora AI releases should avoid downtime.

For rare maintenance requiring degraded access:

- announce when user base justifies it,
- set maintenance mode,
- prevent writes,
- complete operation,
- integrity test,
- restore writes.

Do not use maintenance windows as an excuse for unsafe ordinary migrations.

---

# 212. Maintenance Mode

Feature:

```text
READ_ONLY_MODE
```

or:

```text
MAINTENANCE_MODE
```

Can be used during severe recovery.

Keep login/status/billing support access where safe.

---

# 213. Read-Only Recovery Mode

During uncertain DB/model integrity:

Allow:

- project read,
- report download,
- billing contact.

Block:

- model writes,
- AI model mutations,
- destructive operations.

Useful during incident investigation.

---

# 214. API Timeouts

Set timeouts appropriate to endpoint type.

Synchronous HTTP:

```text
seconds, not minutes
```

Long work:

```text
202 Accepted
→ background workflow
```

Do not keep ALB/API request open during:

- IFC import,
- plan recognition,
- photoreal render,
- long AI design.

---

# 215. Background Job Status

Frontend polls/subscribes to:

```text
QUEUED
RUNNING
NEEDS_REVIEW
COMPLETED
FAILED
```

Job state persists in PostgreSQL/business table.

Temporal provides orchestration history.

---

# 216. Retry Policy

Retry transient:

- provider 429,
- selected provider 5xx,
- network timeout.

Do not retry:

- validation,
- permission,
- insufficient credits,
- deterministic geometry failure.

Use bounded exponential backoff.

---

# 217. Dead Letter / Failed Jobs

Temporal preserves failed workflows; application should expose:

- failed status,
- error code,
- retry/admin action.

Do not silently loop forever.

---

# 218. AI Backpressure

If AI provider quota is saturated:

```text
queue
throttle
return controlled status
```

Do not let every web request independently retry until provider failure worsens.

---

# 219. Upload Backpressure

Limit:

- concurrent uploads per user/org,
- processing jobs per org,
- file size.

Upload can succeed while processing queues if capacity is busy.

---

# 220. Render Backpressure

Render queue can be explicitly capacity-limited.

Show estimated status/order if product later needs it.

Do not autoscale GPU infinitely from malicious requests.

---

# 221. Production Data Fixtures

Use a dedicated synthetic production smoke tenant.

Example:

```text
Buildora AI System Test
```

No real customer data.

Cleanup or reset deliberately.

---

# 222. Release Verification Data

Keep one known tiny project fixture to validate:

```text
2D
3D
QS
BOQ
cost
```

after critical deployments.

Do not modify it in normal customer workflows.

---

# 223. Backup Integrity Data

A known integrity fixture helps verify restored environments.

Check:

- expected model version,
- expected wall count,
- expected QS,
- expected cost.

---

# 224. DevOps Metrics for the Solo Developer

Track:

```text
deploy frequency
deployment failure rate
rollback count
mean time to recover
CI duration
staging failures
production incidents
cloud spend
```

You do not need a formal DORA program, but these metrics reveal process quality.

---

# 225. CI Performance

Keep PR CI fast enough that you use it.

Strategies:

- dependency caching,
- Turborepo cache,
- job parallelism,
- affected-package tests.

Do not skip correctness tests simply to make CI green faster.

---

# 226. Remote Build Cache

Turborepo remote cache can be introduced when CI build time justifies it.

Ensure cache access cannot leak production secrets/artifacts.

---

# 227. Production Change Log

Every production release records:

```text
release version
Git SHA
images
migration version
deploy time
operator/workflow
major changes
rollback target
```

---

# 228. Release Notes

Customer-visible release notes are separate from operational deployment logs.

Do not expose internal security vulnerabilities before patched.

---

# 229. Infrastructure Drift

Periodically run IaC plan against production.

Unexpected drift:

- investigate,
- import/manual change into IaC,
- or revert manual change.

Do not allow permanent hidden console configuration.

---

# 230. Provider Status Dependencies

Maintain links/runbook references for:

- AWS,
- Cloudflare,
- Neon,
- Temporal,
- OpenAI,
- Stripe,
- auth provider,
- Sentry.

During incident, verify provider status before random code changes.

---

# 231. External Dependency Timeouts

Every outbound client must have:

- connection timeout,
- request timeout,
- retry policy.

Do not let hanging provider connections exhaust API workers.

---

# 232. Circuit Breaker — Application Level

For chronically failing optional provider:

- temporarily stop requests,
- fail fast,
- recover after probe.

Use where measured failure patterns justify it.

Do not blindly add complex circuit-breaker libraries everywhere.

---

# 233. Production Configuration Validation

Every service must fail fast if mandatory configuration missing.

API should not boot with:

```text
DATABASE_URL undefined
```

Workers should not poll queues if required credentials are invalid.

---

# 234. Environment Startup Check

At boot log safe metadata:

```text
service
environment
release
node/python version
feature-config version
```

Do not log secrets.

---

# 235. Time Synchronization

Cloud runtime clocks are managed, but application should use UTC/system time and provider timestamps.

Never trust browser clock for billing/model version operations.

---

# 236. File Processing Isolation

Treat user uploads as untrusted.

Heavy parsers:

- Python/BIM worker,
- resource limits,
- isolated container,
- timeout,
- file-size limits.

Do not parse unknown DWG/IFC libraries inside public API request process.

---

# 237. BIM Worker Resource Limits

IFC can be memory-heavy.

Set:

- memory limit,
- job timeout,
- max file size,
- concurrency.

Scale worker separately from API.

---

# 238. Python ML Worker Resources

CV inference may require:

- more CPU/memory,
- later GPU.

Do not give entire API task GPU/memory resources because one feature needs them.

---

# 239. Rollout of New AI Model

Do not switch all production traffic instantly.

Use configuration:

```text
AI_MODEL_BALANCED
```

Rollout:

```text
internal tenant
→ small percentage / selected users
→ compare quality/cost
→ broaden
```

Keep old model configured for rollback where provider supports it.

---

# 240. AI Rate Card Deployment

Provider price changes:

- create new versioned rate card,
- effective timestamp,
- deploy/configure,
- do not rewrite old records.

This is data/config rollout, not necessarily application deployment.

---

# 241. Billing Plan Deployment

New plan version:

- insert versioned catalog data,
- map Stripe price,
- test entitlement resolution,
- activate at effective time.

Do not edit historical plan row in place.

---

# 242. Database Reference Data Deployment

System materials/rates/classification catalogs should use:

- versioned import/migration,
- checksum,
- source version.

Do not update production catalogs manually through SQL.

---

# 243. Disaster Recovery Exercise Schedule

Before launch:

```text
one full exercise
```

After launch:

```text
quarterly or risk-based
```

At minimum simulate:

- database restore,
- application rollback,
- secret rotation.

---

# 244. Security Review Schedule

Before beta:

- architecture review,
- dependency scan,
- tenant/IDOR tests.

Post-launch:

- recurring dependency review,
- annual/major-release penetration test when commercially appropriate,
- enterprise customer security review.

---

# 245. Infrastructure Cost Budget

Set monthly internal budget alerts for:

```text
AWS
Neon
Cloudflare
Temporal
Sentry
AI provider
```

AI budget is managed additionally through product usage controls.

---

# 246. Cost Anomaly Runbook

If infrastructure spend spikes:

1. identify service/tag,
2. check traffic/attack,
3. check autoscaling,
4. check runaway worker/job,
5. stop optional expensive capability if necessary,
6. preserve core product,
7. investigate root cause.

---

# 247. Autoscaling Cost Guardrails

Set maximum task count.

Autoscaling without maximum can become a financial incident during:

- bot attack,
- bug,
- retry storm.

WAF/rate limits provide upstream protection.

---

# 248. Retry Storm Protection

Retries must be:

- bounded,
- jittered,
- categorized.

Do not let:

```text
API retry
+
Temporal retry
+
provider SDK retry
```

multiply into uncontrolled request volume.

Define one owner for retry policy at each layer.

---

# 249. Rate Limits

Apply by:

- IP for public/auth endpoints,
- user/org for authenticated expensive features,
- provider quotas internally.

High-cost AI/render limits belong inside product/backend, not only Cloudflare.

---

# 250. Deployment Architecture Maturity Stages

## Stage 1 — Development

```text
Local Docker
GitHub CI
```

## Stage 2 — Private Alpha

```text
Staging cloud
single production-like environment
```

## Stage 3 — Paid Beta

```text
isolated production
ECS/Fargate
backup
alerts
billing
two replicas where feasible
```

## Stage 4 — Growth

```text
autoscaling
blue/green
GPU workers
read replica
advanced dashboards
```

## Stage 5 — Enterprise

```text
regionalization
SSO
enhanced security/compliance
dedicated resources where justified
```

---

# 251. What Not to Deploy Initially

Do not start with:

```text
Kubernetes/EKS
Kafka
service mesh
self-hosted Temporal cluster
self-hosted PostgreSQL
self-hosted Redis
self-hosted Sentry
multi-region active/active
five separate AWS accounts
24/7 GPU fleet
separate DB per tenant
```

These may become valid later.

They are not the right starting point for a one-person SaaS.

---

# 252. Current Platform Facts to Re-Verify Before Implementation

As of September 2026:

- AWS ECS supports rolling deployments with deployment failure detection/rollback, and managed blue/green strategies can use traffic shifting, bake time and CloudWatch alarm rollback.
- GitHub Actions supports OIDC federation to AWS so workflows can obtain short-lived AWS credentials rather than storing long-lived AWS keys.
- Amazon ECR offers vulnerability scanning, including enhanced scanning through Amazon Inspector.
- Cloudflare R2 exposes an S3-compatible API and supports presigned URLs; presigned URLs should be treated as bearer tokens.
- R2 supports lifecycle rules for expiration/storage-class transitions and is strongly consistent.
- Neon supports isolated branching and point-in-time restore/backup workflows, with exact retention/features depending on plan.
- OpenTelemetry JavaScript traces and metrics are stable; browser instrumentation and logs require more caution.
- Temporal is designed for durable workflow execution that resumes through application/infrastructure failures.

Always verify exact provider capabilities, pricing and CLI/API syntax immediately before infrastructure implementation.

---

# 253. Reference Documentation

AWS ECS:

- ECS rolling deployment / circuit breaker:
  https://docs.aws.amazon.com/AmazonECS/latest/developerguide/deployment-type-ecs.html

- ECS blue/green:
  https://docs.aws.amazon.com/AmazonECS/latest/developerguide/blue-green-deployment-implementation.html

AWS ECR:

- Image scanning:
  https://docs.aws.amazon.com/AmazonECR/latest/userguide/image-scanning.html

GitHub:

- AWS OIDC:
  https://docs.github.com/en/actions/how-tos/secure-your-work/security-harden-deployments/oidc-in-aws

Cloudflare R2:

- R2:
  https://developers.cloudflare.com/r2/

- Presigned URLs:
  https://developers.cloudflare.com/r2/api/s3/presigned-urls/

- Lifecycle rules:
  https://developers.cloudflare.com/r2/buckets/object-lifecycles/

Neon:

- Branching:
  https://neon.com/docs/guides/branching-intro

- Backup / restore documentation:
  https://neon.com/docs/

Temporal:

- Documentation:
  https://docs.temporal.io/

OpenTelemetry:

- JavaScript:
  https://opentelemetry.io/docs/languages/js/

Sentry:

- Next.js / NestJS:
  https://docs.sentry.io/

---

# 254. Master DevOps Checklist

## Foundation

- [ ] Dockerized web.
- [ ] Dockerized API.
- [ ] Dockerized workers.
- [ ] health endpoints.
- [ ] graceful shutdown.
- [ ] IaC initialized.

## CI

- [ ] lint.
- [ ] typecheck.
- [ ] unit.
- [ ] integration.
- [ ] migration test.
- [ ] build.
- [ ] container build.
- [ ] security scan.

## Staging

- [ ] ECR.
- [ ] ECS.
- [ ] ALB.
- [ ] Cloudflare.
- [ ] staging DB.
- [ ] staging Redis.
- [ ] R2.
- [ ] Temporal Cloud.
- [ ] secrets.
- [ ] deployment pipeline.
- [ ] smoke.
- [ ] E2E.

## Production

- [ ] separate production isolation.
- [ ] OIDC deploy role.
- [ ] production secrets.
- [ ] production DB.
- [ ] production Redis.
- [ ] production R2.
- [ ] production Temporal.
- [ ] production auth.
- [ ] production Stripe.
- [ ] production AI provider.
- [ ] backup/PITR.
- [ ] restore test.
- [ ] monitoring.
- [ ] alerts.
- [ ] rollback tested.

## Security

- [ ] least privilege.
- [ ] non-root containers.
- [ ] no public DB/Redis.
- [ ] WAF.
- [ ] secrets manager.
- [ ] image scanning.
- [ ] dependency scanning.
- [ ] tenant tests.

## Operations

- [ ] deploy runbook.
- [ ] rollback runbook.
- [ ] DB restore runbook.
- [ ] provider outage runbooks.
- [ ] incident severity.
- [ ] postmortem template.
- [ ] cost budgets.
- [ ] vendor inventory.

---

# 255. Final DevOps Architecture

```text
                    GITHUB
                      │
             PR → CI / Tests
                      │
                Merge to main
                      │
              Build containers
                      │
                     ECR
                      │
              Deploy STAGING
                      │
           Migration + E2E + Smoke
                      │
              Production approval
                      │
         Promote SAME image digests
                      │
             Migration job
                      │
                ECS Production
                      │
       ┌──────────────┼────────────────┐
       │              │                │
       ▼              ▼                ▼
     Web/API        Workers         Temporal
       │              │                │
       └──────────────┼────────────────┘
                      │
       ┌──────────────┼──────────────────┐
       ▼              ▼                  ▼
   PostgreSQL       Redis               R2
       │
       ▼
  Backups / PITR

Edge:
Cloudflare DNS / WAF / CDN

Observability:
Sentry + OpenTelemetry + AWS metrics

External:
OpenAI / Stripe / Auth Provider
```

---

# 256. Final Principles

Buildora AI DevOps must preserve these rules:

1. **Containers are immutable and stateless.**
2. **One artifact is built and promoted across environments.**
3. **Production deploys use immutable image SHA/digest, never mutable `latest`.**
4. **GitHub Actions uses short-lived OIDC cloud credentials.**
5. **Production secrets are injected at runtime.**
6. **Database migrations are explicit deployment jobs, never app-startup side effects.**
7. **Schema changes remain backward compatible during rolling releases.**
8. **Staging is required before production.**
9. **Production has a tested rollback path.**
10. **Database backups are proven by restore tests.**
11. **R2/files are private by default.**
12. **Redis holds no irreplaceable state.**
13. **AI/Stripe/provider outages do not destroy core deterministic functionality.**
14. **Long operations use Temporal/background workers, not long HTTP requests.**
15. **Workers scale independently from the API.**
16. **AI/GPU capacity is cost-controlled.**
17. **Observability ships with every production feature.**
18. **Security follows least privilege.**
19. **Infrastructure changes are code-reviewed and reproducible.**
20. **Kubernetes is introduced only after ECS limitations are measured.**
21. **Multi-region is introduced only after real latency/residency/availability needs exist.**
22. **Production operations remain simple enough for a one-person SaaS to manage safely.**

---

# 257. Final Rule

For Buildora AI, the correct production philosophy is:

> **Use managed infrastructure for undifferentiated operations, keep application architecture portable, make every release reproducible, and add operational complexity only when real customer load or enterprise requirements pay for it.**

The goal is not to build an impressive DevOps platform.

The goal is to make:

```text
Buildora AI
```

safe to deploy, easy to recover, economical to operate, and able to scale without forcing a rewrite.
