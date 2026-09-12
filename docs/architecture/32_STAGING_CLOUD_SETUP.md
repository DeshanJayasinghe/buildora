# Staging Environment Setup

Do not deploy directly from local development into production.

Create a staging environment before beta.

## Recommended solo-developer staging topology

```text
Cloudflare DNS/CDN
      │
      ├── Web deployment
      └── API deployment
              │
        Managed PostgreSQL
        Managed Redis
        R2/S3
        Temporal Cloud
```

You can deploy web/API to a platform that supports your chosen runtime, or to AWS containers.

Keep the application portable through Docker and environment configuration.

## 1. Create staging resources

Use explicit names:

```text
buildwise-staging-db
buildwise-staging-redis
buildwise-staging-files
```

## 2. Managed PostgreSQL

For an initial solo setup, Neon is a practical staging/production PostgreSQL option.

Create:

- staging project/branch
- production project/branch separately

Do not point staging at production.

## 3. Migrations

Deploy flow:

```text
CI passes
 ↓
apply staging migrations
 ↓
deploy API
 ↓
deploy web
 ↓
smoke tests
```

## 4. Auth

Create/use a separate staging Clerk instance/environment if practical.

## 5. Stripe

Use Stripe test mode only in staging.

## 6. AI

Use separate provider project/key if available.

Set strict spend/usage alerts.

## 7. Smoke tests

At minimum:

```text
sign in
create project
open project
save project
reload project
sign out
```

Later include editor/QS/AI flows.

## 8. Staging data

Use synthetic/non-sensitive test data.

Do not clone customer production data into staging by default.
