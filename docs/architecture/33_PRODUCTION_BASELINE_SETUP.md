# Production Baseline Setup

Do not use this guide until staging is stable.

## 1. Production resources

Create isolated:

- production web
- production API
- production PostgreSQL
- production Redis
- production object storage
- production Temporal namespace/account
- production auth environment
- production Stripe mode
- production AI provider project/key

## 2. Domain/DNS

Use Cloudflare or equivalent for:

```text
app.example.com
api.example.com
```

A same-origin API proxy strategy is also acceptable if chosen deliberately.

## 3. TLS

All public production traffic is HTTPS only.

## 4. Database

Enable:

- automated backups
- point-in-time recovery where provider supports it
- connection pooling
- monitoring
- least-privilege credentials.

## 5. Deploy

Preferred:

```text
merge to main
 ↓
CI
 ↓
artifact/container build
 ↓
staging verification
 ↓
production promotion
 ↓
smoke tests
```

## 6. Migrations

Migrations must be backward-aware.

Avoid a deploy that requires new code and destructive DB change at exactly the same instant.

Use expand/migrate/contract for risky changes.

## 7. Object storage

Private by default.

Signed short-lived access.

## 8. Redis

TLS/private network.

No authoritative data only in Redis.

## 9. AI

Production key:

- server only
- budget limits
- usage ledger
- model aliases configured
- retries/timeouts
- escalation policy
- kill switch.

## 10. Billing

Verify webhook signatures and replay/idempotency.

## 11. Monitoring

Do not launch paid beta without:

- application error monitoring
- DB monitoring
- billing failure alerts
- AI spend anomaly alert
- backup confirmation.
