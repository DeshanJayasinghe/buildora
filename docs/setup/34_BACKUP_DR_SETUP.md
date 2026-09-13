# Backup and Disaster Recovery Setup

## 1. Define data classes

Authoritative:

- PostgreSQL domain/project data
- billing ledger
- audit log
- model versions
- file metadata

Blob:

- uploads
- IFC/DWG/PDF
- renders
- reports

Ephemeral:

- Redis caches/presence
- local generated artifacts

## 2. Database backup

Use managed backup/PITR where available.

Before production, document:

```text
RPO target
RTO target
```

Initial realistic SaaS targets may be modest; measure and improve.

## 3. Object storage

Enable appropriate object versioning/lifecycle/retention based on cost and customer requirements.

## 4. Restore drill

A backup you have never restored is not proven.

Quarterly or before major beta milestones:

```text
restore DB to isolated environment
validate row counts
open representative projects
validate model versions
validate BOQ/cost data
```

## 5. Redis

Do not rely on Redis restoration for authoritative application correctness.

## 6. Temporal

Production workflow persistence is handled by the Temporal service/cluster, but application recovery procedures must document worker redeploy/replay compatibility.

## 7. AI

AI provider is not a system of record.

Store the Buildora AI action/result records needed for audit and reproducibility.

## 8. Runbook

Create an incident checklist for:

- DB unavailable
- corrupted migration
- object storage unavailable
- auth provider outage
- AI provider outage
- Stripe webhook failure.
