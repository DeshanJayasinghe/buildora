# Redis Setup

Redis is used for ephemeral/distributed concerns, not as the project source of truth.

## 1. Local Redis

Use Docker Compose.

Verify:

```bash
docker compose exec redis redis-cli ping
```

Expected:

```text
PONG
```

## 2. Add Node client

Redis currently recommends `node-redis` for Node.js.

```bash
pnpm --filter api add redis
```

## 3. Environment

```text
REDIS_URL=redis://localhost:6379
```

## 4. Create infrastructure adapter

Do not call Redis directly from domain packages.

Use infrastructure services for:

- cache
- rate limiting
- short-lived locks
- presence
- temporary idempotency keys

## 5. Initial uses

For M1, Redis may be used for:

- API rate limiting
- cache only when measured
- short-lived auth/session auxiliary state where required

Do not prematurely cache every query.

## 6. Later uses

- collaborative presence
- distributed locks
- temporary AI usage reservation
- task coordination where Temporal is not the right abstraction

## 7. Production

Use TLS for managed Redis when supported/required.

Do not expose Redis publicly.

## 8. Failure rule

Critical project data must survive Redis loss.

If deleting Redis loses a Building Model, BOQ, or payment ledger, the architecture is wrong.
