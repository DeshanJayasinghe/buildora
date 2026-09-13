# Stripe Billing, Plans and Credit Ledger Setup

Do not integrate billing before user/project flows are stable, but design usage accounting early.

## 1. Stripe CLI

On macOS:

```bash
brew install stripe/stripe-cli/stripe
stripe login
```

## 2. SDK

API:

```bash
pnpm --filter api add stripe
```

## 3. Environment

```text
STRIPE_SECRET_KEY=
STRIPE_WEBHOOK_SECRET=
STRIPE_PRICE_HOME=
STRIPE_PRICE_PRO=
STRIPE_PRICE_BUSINESS=
```

Publishable key may be exposed only through the correct browser-safe variable when client-side Stripe components are used.

## 4. Plans

Initial conceptual plans:

```text
Free
Home
Pro
Business
```

Do not encode entitlements purely by Stripe price ID.

Map external price/product IDs to internal plan definitions.

## 5. Webhook endpoint

Implement signature verification.

Events should update internal subscription state idempotently.

## 6. Usage ledger

Buildora AI needs its own usage accounting for:

- AI design credits
- render credits
- possibly storage limits

Use append-style ledger entries.

Do not only store:

```text
credits_remaining = 17
```

without an audit trail.

## 7. Reservation

For an expensive AI/render operation:

```text
check entitlement
 ↓
reserve credits
 ↓
run operation
 ↓
settle actual charge
 ↓
release unused reservation
```

Handle failures idempotently.

## 8. Development webhooks

Use Stripe CLI forwarding to local API.

Example pattern:

```bash
stripe listen --forward-to localhost:4000/api/v1/billing/webhook
```

Use the current CLI output webhook signing secret only in local env.

## 9. Never trust client subscription status

API checks internal entitlement state for protected actions.
