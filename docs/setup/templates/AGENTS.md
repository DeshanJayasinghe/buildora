# BuildWise Agent Instructions

BuildWise architecture is documented under:

```text
docs/architecture/
```

Before implementing any feature:

1. Read `docs/architecture/00_README.md`.
2. Read the master product reference.
3. Read the architecture guide for the module being changed.
4. Read applicable ADRs.
5. Inspect current code before creating a new package/service.

Critical rules:

- The BuildWise Building Model is the source of truth.
- Never duplicate authoritative 2D / 3D / BIM / QS / BOQ / cost state.
- 2D and 3D are adapters/views over the Building Model.
- IFC is interoperability, not the live domain database.
- AI does not perform authoritative quantity or cost arithmetic.
- AI geometry changes use structured, validated ChangeSets.
- Financial calculations are decimal-safe.
- Canonical construction geometry uses millimetres unless an ADR changes it.
- Tenant/project authorization is enforced server-side.
- Tests are mandatory.
- Architectural changes require an ADR or explicit approval.

Before completing an implementation:

- run relevant tests,
- run lint,
- run typecheck,
- run build where applicable,
- report changed files,
- report assumptions and incomplete items.
