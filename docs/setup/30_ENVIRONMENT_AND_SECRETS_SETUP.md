# Environment Variables and Secrets Setup

## 1. Principle

No secret is committed to Git.

Commit:

```text
.env.example
```

Do not commit:

```text
.env
.env.local
.env.production
```

## 2. Local variables

Examples:

```text
WEB_URL=http://localhost:3000
API_URL=http://localhost:4000

DATABASE_URL=postgresql://buildwise:local-development-only@localhost:5432/buildwise
REDIS_URL=redis://localhost:6379

CLERK_SECRET_KEY=
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=

OPENAI_API_KEY=

R2_ACCOUNT_ID=
R2_ACCESS_KEY_ID=
R2_SECRET_ACCESS_KEY=
R2_BUCKET=

STRIPE_SECRET_KEY=
STRIPE_WEBHOOK_SECRET=
```

## 3. Public variables

Only variables explicitly intended for browser exposure may use framework public prefixes.

Never place:

- secret API keys
- DB passwords
- Stripe secret keys
- storage secrets

in public variables.

## 4. Environment validation

Every deployable should validate required configuration during startup.

Fail fast with a clear configuration error.

Do not allow:

```text
undefined
```

to travel into infrastructure clients.

## 5. Separate environments

Use different credentials/resources for:

```text
local
staging
production
```

Production credentials must never be copied into local `.env`.

## 6. Rotation

Design provider adapters so keys can be rotated without schema changes.

## 7. AI agent safety

Do not paste real production secrets into Codex/Claude/Antigravity conversations.

Configure secrets through local/cloud secret stores.
