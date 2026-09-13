# Buildora AI Project Setup Pack

This pack is the step-by-step environment and repository setup companion to the Buildora AI Architecture Pack.

## Goal

Start with a clean development machine and reach a verified Buildora AI workspace containing:

```text
buildora/
├── apps/
│   ├── web/
│   ├── api/
│   ├── ai-worker/
│   ├── bim-worker/
│   └── render-worker/       # later
├── packages/
│   ├── design-system/
│   ├── model-session/
│   ├── building-model/
│   ├── cad-2d/
│   ├── engine-3d/
│   ├── qs-engine/
│   ├── cost-engine/
│   ├── ai-tools/
│   ├── api-contracts/
│   └── units/
├── native/
│   └── geometry-wasm/
├── infrastructure/
├── docs/
│   └── architecture/
├── AGENTS.md
├── CLAUDE.md
├── pnpm-workspace.yaml
├── turbo.json
└── package.json
```

## Recommended setup defaults

These are pragmatic defaults for a one-person SaaS implementation.

| Concern | Initial decision |
|---|---|
| Node | Node 24 LTS |
| Package manager | pnpm |
| Monorepo | Turborepo |
| Web | Next.js + React + TypeScript |
| API | NestJS + TypeScript |
| DB access | Drizzle ORM + node-postgres |
| Local DB | PostgreSQL in Docker |
| Production DB | Neon initially; AWS RDS/Aurora remains a later option |
| Cache | Redis |
| Auth | Clerk initially, behind an application auth adapter |
| Python | uv + FastAPI |
| Geometry | TypeScript first; Rust/WASM when justified |
| 2D | PixiJS |
| 3D | Three.js |
| BIM web | That Open + Fragments/web-ifc |
| BIM server | IfcOpenShell |
| Durable workflows | Temporal |
| Production AI | Buildora AI Gateway using provider APIs |
| File storage | Cloudflare R2 or S3 |
| Billing | Stripe |
| Unit tests | Vitest / pytest / cargo test |
| E2E | Playwright |
| Observability | Sentry + OpenTelemetry |

## Important

Do not install every optional technology on day one.

The correct setup order is documented in `01_SETUP_ORDER_AND_CHECKLIST.md`.

The minimum environment needed to start Milestone M0/M1 is:

1. Git
2. Node 24 LTS
3. pnpm
4. Docker Desktop
5. repository
6. Turborepo
7. Next.js
8. NestJS
9. PostgreSQL
10. Redis
11. CI

Python, Rust/WASM, BIM, Temporal, AI, rendering, PostGIS, and pgvector can be enabled when their milestone starts.

## Relationship with architecture documents

If this setup pack conflicts with the Buildora AI Architecture Pack:

1. a newer explicit ADR wins,
2. otherwise the Architecture Pack wins,
3. otherwise use this setup pack as the implementation default.

Never silently change a core architecture decision just because a tool's quickstart suggests a different structure.
