# ADR-XXXX — <Title>

> **Status:** Proposed | Accepted | Superseded by ADR-YYYY | Deprecated
> **Date:** YYYY-MM-DD
> **Deciders:** <who approved this>
> **Supersedes:** <ADR number, or "none">
> **Superseded by:** <ADR number, or "none">

---

## Context

What forces this decision? What is the problem, constraint, or conflict?

State facts, not preferences. If the decision resolves a contradiction between
existing documents, name both documents and quote the conflicting statements.

## Decision

The decision, stated in the active voice and specific enough to implement
against. Avoid "should" and "could" — an ADR says what *is*.

Include the concrete shape where it matters: table columns, type names,
package names, dependency directions, allowed and forbidden patterns.

## Alternatives considered

| Option | Pros | Cons | Why not chosen |
|---|---|---|---|
| A — ... | ... | ... | ... |
| B — ... | ... | ... | ... |

An ADR with no alternatives considered is usually not a real decision.

## Consequences

**Positive**

- ...

**Negative / accepted cost**

- ...

**Neutral**

- ...

Be honest about the negative consequences. An ADR that lists only benefits
is not trustworthy.

## Migration / implementation impact

- What must be built or changed?
- What existing documents or code does this affect?
- Is data migration required? If so, at what cost if deferred?
- Which sprint or phase owns the implementation?

## Rollback / exit strategy

If this decision proves wrong, what does reversing it cost, and what is the
earliest signal that it is wrong?

Some decisions are effectively irreversible once data exists. Say so plainly.

## Verification

How will we know this decision is being honoured in the code?

- CI rule, lint rule, or dependency check
- Test that would fail if violated
- Review checklist item

## References

- Related ADRs
- Architecture documents
- External sources, with the date consulted
