# ADR-004 — UUID Primary Keys with Separate Public Identifiers

> **Status:** Accepted
> **Date:** 2026-09-12
> **Deciders:** Product Owner / CTO
> **Supersedes:** none
> **Superseded by:** none

---

## Context

Two incompatible ID strategies existed in the documentation:

- `docs/architecture/15_Database_and_Data_Architecture.md` §7 mandated native
  `UUID` primary keys (UUIDv7 preferred, UUIDv4 fallback).
- `docs/examples/Buildora_AI_Sample_Backend_CRUD.md` implemented
  `varchar(64)` primary keys populated with `prj_<uuid>` values.

The database document acknowledged the discrepancy and deferred it to an ADR
that had never been written.

This matters disproportionately because implementation agents pattern-match
working examples over prose. Shipping the first migration with varchar primary
keys would mean rewriting every table, foreign key and index in a populated
database to correct it.

## Decision

**Primary keys are native PostgreSQL `uuid`.**

- Generated in the application as **UUIDv7** where a stable, standardized
  implementation is available in the project.
- **UUIDv4** is an acceptable fallback.
- ID strategies are not mixed casually across tables.

**Human-facing entities additionally carry `public_id`**, a short display
reference:

```text
database primary key   019a3f2c-... (uuid)
public reference       PRJ-8K4D2A   (varchar)
```

Rules for `public_id`:

- Display, URL and support reference only.
- **Never a foreign-key target.** Relational integrity uses the `uuid` PK only.
- Unique per tenant, enforced by a database unique index — generation alone is
  not sufficient. Retry on conflict.
- Generated from an unambiguous alphabet (no `I`, `O`, `0`, `1`).

**Other identifier types** (unchanged from the database architecture):

- `BIGINT` for monotonic counters — model version, sequence numbers, high-volume
  event sequence.
- `NUMERIC` for money, rates and exact percentages.

The prefixed-varchar primary key pattern is **withdrawn**.
`docs/examples/Buildora_AI_Sample_Backend_CRUD.md` has been corrected. No
document in this repository now shows a prefixed varchar primary key.

## Alternatives considered

| Option | Pros | Cons | Why not chosen |
|---|---|---|---|
| A — `uuid` PK only | 16 bytes; index-friendly; standard | Opaque in logs and support conversations | Loses human-facing readability |
| B — `varchar(64)` prefixed PK | Self-describing everywhere | ~4× storage; larger indexes on every FK; non-standard; invites string-typed IDs and accidental parsing | Cost paid on every index and join, forever |
| C — `uuid` PK + `public_id` (chosen) | Both benefits; display identity decoupled from relational identity | One extra column and unique index per user-facing table | — |

UUIDv7 is preferred over UUIDv4 because time-ordered keys reduce B-tree page
splits and improve insert locality — relevant for the high-volume journal and
audit tables.

## Consequences

**Positive**

- Compact, standard, index-friendly primary keys.
- Support and URLs still get a readable reference.
- Display identity can change (rebranding, format change) without touching
  relational integrity.

**Negative / accepted cost**

- One extra column and unique index on user-facing tables.
- Two generation paths in the ID port (`generateId`, `generatePublicId`).

**Neutral**

- UUIDv7 availability is a library choice, not an architecture change; the port
  abstracts it.

## Migration / implementation impact

**No data migration — this is decided before the first migration exists.**
That timing is the entire point of the ADR.

Documents corrected as part of this decision:

- `docs/examples/Buildora_AI_Sample_Backend_CRUD.md` — schema and ID generator
  rewritten to `uuid` + `public_id`.
- `docs/architecture/15_Database_and_Data_Architecture.md` §7 — marked LOCKED,
  deferral note replaced with the resolution.

Owned by: Sprint 00 (documentation), first migration in Sprint 02/04.

## Rollback / exit strategy

**Effectively irreversible once data exists.** Changing the primary key type of
a populated schema requires rewriting every table and foreign key. Reversal is
only realistic before the first production migration.

## Verification

- Review: any `varchar` primary key on a new table is a BLOCKER.
- Review: any foreign key referencing `public_id` is a BLOCKER.
- Test: `public_id` uniqueness conflict is handled by retry, not by failure.
- Grep gate: `id: varchar(` and `prj_` must not appear in repository documentation
  or code.

## References

- `docs/architecture/15_Database_and_Data_Architecture.md` §7, §8
- `docs/examples/Buildora_AI_Sample_Backend_CRUD.md` §17, §18
- OD-01 in `docs/architecture/OPEN_ARCHITECTURE_DECISIONS.md`
