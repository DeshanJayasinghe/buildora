# Setup Order and Master Checklist

Follow this order. Do not jump straight into AI, BIM, or rendering.

## Stage A — Machine

- [ ] Install/update Xcode Command Line Tools on macOS
- [ ] Install Homebrew if you use it
- [ ] Install Git
- [ ] Install Node 24 LTS
- [ ] Install pnpm
- [ ] Install Docker Desktop
- [ ] Verify terminal, Git, Node, pnpm, Docker
- [ ] Configure GitHub SSH
- [ ] Install/use Codex, Claude Code, and Antigravity

Verification:

```bash
git --version
node --version
pnpm --version
docker --version
docker compose version
```

Node should be a supported 24.x LTS release.

## Stage B — Repository foundation

- [ ] Create private GitHub repository
- [ ] Clone locally
- [ ] Copy architecture docs into `docs/architecture/`
- [ ] Add `AGENTS.md`
- [ ] Add `CLAUDE.md`
- [ ] Create pnpm workspace
- [ ] Configure Turborepo
- [ ] Create `.editorconfig`
- [ ] Create `.gitignore`
- [ ] Create `.env.example`

## Stage C — First applications

- [ ] `apps/web` — Next.js
- [ ] `apps/api` — NestJS
- [ ] health endpoints
- [ ] shared TypeScript config
- [ ] shared lint/typecheck scripts
- [ ] `pnpm dev`
- [ ] `pnpm build`

## Stage D — Local infrastructure

- [ ] PostgreSQL
- [ ] Redis
- [ ] Docker Compose
- [ ] DB connection health
- [ ] migration tooling
- [ ] local reset procedure

## Stage E — SaaS foundation

- [ ] Clerk authentication
- [ ] local user mirror
- [ ] organization/workspace model
- [ ] authorization guard
- [ ] project CRUD
- [ ] audit event baseline

## Stage F — Domain engine foundation

- [ ] `packages/units`
- [ ] `packages/domain`
- [ ] `packages/building-model`
- [ ] command model
- [ ] model version
- [ ] first Wall entity
- [ ] unit tests

## Stage G — Visual engines

- [ ] PixiJS 2D package
- [ ] Three.js 3D package
- [ ] renderer adapters
- [ ] simple wall fixture
- [ ] 2D/3D sync
- [ ] save/reload test

## Stage H — Construction intelligence

- [ ] materials
- [ ] assemblies
- [ ] QS engine
- [ ] BOQ engine
- [ ] cost engine
- [ ] decimal-safe money
- [ ] golden fixtures

## Stage I — AI

- [ ] OpenAI/provider API key
- [ ] AI Gateway
- [ ] model aliases
- [ ] usage ledger
- [ ] typed tools
- [ ] first read-only tools
- [ ] proposed ChangeSet tool
- [ ] approval pipeline

## Stage J — Import/BIM/workflows

- [ ] Python uv
- [ ] FastAPI worker
- [ ] Temporal
- [ ] plan upload pipeline
- [ ] IFC/web BIM packages
- [ ] IfcOpenShell worker
- [ ] pgvector
- [ ] document RAG

## Stage K — Commercial/production

- [ ] R2/S3
- [ ] Stripe
- [ ] credit ledger
- [ ] Sentry
- [ ] OpenTelemetry
- [ ] staging
- [ ] backups
- [ ] production
- [ ] alerts
- [ ] production readiness checklist

## Do not block M1 on these

The following are later:

- OpenCascade
- ODA SDK
- Blender GPU workers
- PostGIS site intelligence
- advanced IFC roundtrip
- mobile/LiDAR
- structural/MEP
- supplier marketplace
