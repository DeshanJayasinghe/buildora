# ADR-016 — Tenant Isolation and the RLS Production Gate

> **Status:** Accepted
> **Date:** 2026-09-12
> **Deciders:** Product Owner / CTO
> **Supersedes:** none
> **Superseded by:** none

---

## Context

Organization is the tenant boundary. Cross-tenant data disclosure is the one
defect class that can end a B2B SaaS business outright.

Application-level scoping is the primary control, and it is well specified. But
a single missing `WHERE organization_id = ?` in one repository method is a
breach. PostgreSQL Row-Level Security turns that same mistake into an empty
result set.

The database architecture deferred RLS to "before paid production beta" with no
owning sprint and no design work scheduled — which in practice means it does not
get built.

A first draft of this decision proposed simply enabling RLS in Sprint 05. That
was refined during review: forcing a poorly-tested session-variable configuration
into every development path merely to satisfy a sprint number creates its own
failure mode, particularly around pooled connections.

## Decision

Tenant isolation is a **non-negotiable requirement** with two defences applied
on different schedules.

### From Sprint 05 — mandatory, no exceptions

1. Every tenant-owned table carries `organization_id` where required (direct
   tagging preferred on high-risk and high-volume tables, even when derivable
   from `project_id`).
2. Every repository method **explicitly scopes by organization**:
   `findProjectById(organizationId, projectId)` — never `findById(projectId)`
   for tenant-owned data.
3. **Negative cross-tenant integration tests are mandatory** — for each
   tenant-owned repository, organization A must not be able to read or write
   organization B's rows.
4. **RLS schema and policies must be designed** and written, not deferred to
   discovery later.
5. The **transaction/session-context helper is implemented and tested** — the
   mechanism that sets `app.current_organization_id` for a unit of work.
6. That helper's behaviour is **verified against the actual pooled PostgreSQL
   connection model** in use, since session variables and connection pooling
   interact in ways that must be proven rather than assumed.

### Hard gate — before the first external beta

**RLS must be enabled, tested and verified before any external user accesses
production.** Production beta is not possible without it.

This is a release gate, not an aspiration.

### Deliberately not required

RLS enforcement need not be switched on in every local development path from
Sprint 05 if doing so would mean shipping a poorly-understood session-variable
configuration. The requirement is that it is **designed, implemented, and proven
against the real connection model** — with the enforcement gate landing before
external exposure.

The distinction matters: the goal is working isolation, not a checkbox.

### The normative runtime recipe

> **Added 2026-09-12 after independent review.** Session-scoped tenant context is
> **unsafe under transaction pooling** (Neon's pooled endpoint, PgBouncer in
> transaction mode). A connection is returned to the pool between transactions, so
> a `SET` from one request can leak into another — or vanish before the query that
> depends on it.

Every tenant-scoped operation **must** follow this shape:

```text
BEGIN
  SELECT set_config('app.current_organization_id', $1, true)   ← true = LOCAL
  ...all tenant queries for this unit of work, on THIS transaction...
COMMIT
```

Binding rules:

1. **`SET LOCAL` / `set_config(..., true)` only.** Never session-level `SET`.
2. **Every scoped query runs inside that same transaction.** A query issued
   outside it has no tenant context and must fail closed, never fall back.
3. **The runtime database role is non-owner and non-superuser, and must not hold
   `BYPASSRLS`.** PostgreSQL exempts table owners from RLS by default — a test
   suite running as the owner will appear to pass while enforcing nothing.
4. **Decide per table where `FORCE ROW LEVEL SECURITY` is required**, so even an
   owning role is subject to policy.
5. **CI integration tests run with RLS actually enabled**, from Sprint 05, using
   the exact driver and pooling mode used in production — not a direct connection.

### Application scoping is never optional

RLS is defence in depth. It does not license relaxing application-level scoping.
Both layers are required.

### Cross-tenant access is release-blocking

Any cross-tenant read or write discovered at any time is a release-blocking
defect, not a bug to triage.

## Alternatives considered

| Option | Pros | Cons | Why not chosen |
|---|---|---|---|
| A — Application scoping only | Simple; no pooling complications | One missed `WHERE` is a breach; no safety net | Unacceptable for B2B multi-tenant |
| B — RLS fully enforced from Sprint 05 everywhere | Strongest earliest | Risks a poorly-tested session/pooling configuration baked into every dev path | Refined during review |
| C — Design + test from Sprint 05, enforce before external beta (chosen) | Both layers proven; pooling interaction verified deliberately; cheap while few tables exist | Requires discipline to honour the gate | — |
| D — RLS "before paid beta", undated | Nominally planned | No owning sprint means it does not happen; retrofitting across 60 tables is painful | The situation being corrected |

## Consequences

**Positive**

- Two independent defences against the highest-severity defect class.
- Designing RLS at Sprint 05 (few tables) is far cheaper than at Sprint 37 (many).
- The pooling interaction is discovered deliberately in development rather than
  during a beta incident.

**Negative / accepted cost**

- Session-context plumbing on every database interaction.
- RLS policies must be maintained as tables are added.
- Some debugging complexity when a policy silently filters rows.

**Neutral**

- Admin and support access paths need explicit, audited elevation.

## Migration / implementation impact

No migration.

Affects: `docs/architecture/15_Database_and_Data_Architecture.md` §13–§15, §169–§171,
`docs/planning/*` Sprint 05, `apps/api` repository layer and database module.

Owned by: Phase 3 / Sprint 05, with the enforcement gate in the
production-readiness phase.

## Rollback / exit strategy

Not reversible in the safety direction. Removing tenant isolation is never an
acceptable outcome.

## Verification

- Test (mandatory, per repository): organization A cannot read organization B's rows.
- Test (mandatory, per repository): organization A cannot write organization B's rows.
- Test: the session-context helper sets and clears correctly across pooled
  connection reuse.
- Test: with RLS enabled, a deliberately unscoped query returns zero rows rather
  than another tenant's data.
- Release gate: RLS enabled and verified in production before first external user.
- Review: any repository method on a tenant-owned table without an organization
  parameter is a BLOCKER.

## References

- `AGENTS.md` §24, §25, §81
- `CLAUDE.md` §5.6, §21, §29
- `docs/architecture/15_Database_and_Data_Architecture.md` §10–§15
- ADR-003, ADR-004
