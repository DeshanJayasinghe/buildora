# Observability — Sentry and OpenTelemetry Setup

Do not add a full observability platform before the app runs, but add baseline error tracking before beta users.

## 1. Sentry

Recommended early coverage:

- Next.js
- NestJS
- Python workers

Use the current Sentry setup wizard/SDK for each framework.

Environment separation:

```text
development
staging
production
```

Never send secrets or full sensitive project documents into error metadata.

## 2. OpenTelemetry

Start server-side first.

OpenTelemetry JavaScript traces and metrics are stable; browser instrumentation should be introduced cautiously.

Trace important flows:

```text
HTTP request
 ↓
DB
 ↓
Temporal
 ↓
AI provider
 ↓
tool call
```

## 3. Correlation IDs

Every API request should have/request a correlation/trace identifier.

Important records should also include:

- user
- organization
- project
- model version
- workflow ID
- AI request ID

without leaking confidential content.

## 4. Key metrics

- API error rate
- p50/p95/p99 latency
- DB latency
- Redis failures
- workflow failures
- AI provider latency
- AI cost
- render duration
- upload failures
- model-load time
- 2D/3D interaction performance telemetry later.

## 5. Logs

Use structured JSON in production.

Never log:

- API keys
- auth tokens
- raw payment secrets
- complete private documents by default.

## 6. Alerts

Before beta:

- elevated API error rate
- DB unavailable
- repeated workflow failure
- billing webhook failure
- AI spend anomaly.
