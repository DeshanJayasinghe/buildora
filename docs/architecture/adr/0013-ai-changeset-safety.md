# ADR-013 — AI ChangeSet Safety Architecture

> **Status:** Accepted
> **Date:** 2026-09-12
> **Deciders:** Product Owner / CTO
> **Supersedes:** none
> **Superseded by:** none

---

## Context

Buildora AI lets users modify buildings through natural language. The building
model drives quantities, BOQ and cost — and ultimately real construction
decisions. An LLM that could write geometry directly would be able to corrupt
the authoritative model, break quantity lineage, and produce cost figures with
no traceable origin.

Additionally, uploaded construction documents are untrusted input flowing into a
tool-calling agent. A PDF containing instruction-shaped text is a real attack path.

## Decision

### AI proposes; the domain commits

```text
User request
  → AI Gateway
  → structured ChangeSet          (ai_change_sets / ai_change_items)
  → draft / sandbox               (model_drafts, relative to base version)
  → geometry validation
  → QS / cost impact computation
  → preview presented to user
  → user approval
  → baseVersion RE-CHECK
  → transactional commit via domain commands
  → new model version
```

**AI writes only to `ai_change_sets` and `ai_change_items`.** Only the domain
commit path writes `building_elements`.

This is enforced at **two** levels, and the distinction matters:

| Level | Mechanism | What it actually prevents |
|---|---|---|
| Code | The AI module has no import of model repositories; enforced by CI boundary checks | An *intended* dependency |
| **Database** | The AI execution context uses a **least-privilege PostgreSQL role** that can write draft/change-set tables but has **no INSERT/UPDATE/DELETE grant on canonical model tables** | An *accidental* one |

> **Amended 2026-09-12 after independent review.** This ADR previously claimed the
> module boundary alone was a *structural* barrier. It is not. In a modular
> monolith all modules share one process and, by default, one database credential.
> An import rule stops a deliberate dependency; it does nothing if a generic
> database handle is injected into a shared service during a refactor — the
> credential still has permission to execute the SQL.
>
> The code boundary is necessary but not sufficient. **The database privilege is
> the actual structural barrier.** Tests must assert the privilege, not only the
> import graph.

Explicitly forbidden:

```text
LLM → direct UPDATE building_elements
LLM → raw SQL against any database
LLM → arbitrary database read access
```

### baseVersion is re-checked at commit

A ChangeSet is created against `base_model_version`, but the user may approve it
minutes later after other edits have landed. The commit path **re-checks
`baseVersion` at approval time** and either rebases or fails with
`MODEL_VERSION_CONFLICT` (ADR-002). Validation at creation time alone is
insufficient.

### Tools are typed and scoped

- Every tool has a runtime-validated input and output schema.
- Every tool is scoped to the caller's organization and project — tool
  authorization is checked server-side, never inferred from the conversation.
- Tool returns are narrow and paginated. Whole-model dumps are forbidden, for
  both cost and context-quality reasons.

### Untrusted content handling

- Uploaded and project documents are **data, never instructions**.
- Document content is explicitly delimited in prompts and the model is instructed
  to treat it as untrusted data.
- **Document contents never directly trigger tool invocation.**
- Any ChangeSet derived from document context requires **human approval** — it
  may not be auto-applied under any policy.
- RAG retrieval is tenant- and project-filtered before or together with semantic
  ranking; cross-tenant retrieval is a release-blocking defect.

### Determinism boundary

The LLM is never authoritative for:

```text
geometry        quantities      BOQ totals
financial arithmetic            tax arithmetic
billing balances                credit balances
```

AI may explain, summarize, recommend, classify and propose. Deterministic
engines calculate authoritative results (ADR-015).

### Confidence is first-class

Recognition and AI-derived results carry confidence, provenance and verification
status. Low-confidence results require user review and are never silently
converted into authoritative geometry.

## Alternatives considered

| Option | Pros | Cons | Why not chosen |
|---|---|---|---|
| A — AI writes directly with validation | Fewer moving parts; faster | No preview; no audit of intent vs effect; a validation gap corrupts the model | Unacceptable risk to the authoritative model |
| B — AI proposes, domain commits (chosen) | Model cannot be corrupted by AI; full lineage; user sees impact before commit | More infrastructure: change sets, drafts, preview | — |
| C — AI in a fully separate sandbox project | Maximum isolation | Cannot show real QS/cost impact against the live model | Loses the product's key AI value |

## Consequences

**Positive**

- The authoritative model cannot be corrupted by a model error or a prompt attack.
- Users see what changes and what it costs before committing.
- Full lineage: user → AI job → provider calls → tools → ChangeSet → approval →
  command → model version.
- AI features can be disabled independently without affecting deterministic work.

**Negative / accepted cost**

- More infrastructure than direct mutation: change sets, change items, drafts,
  preview computation.
- Preview requires running QS/cost against a draft, which costs compute.

**Neutral**

- Partial (per-item) approval is supported by the data model; MVP policy is
  all-or-nothing, revisitable without schema change.

## Migration / implementation impact

No migration.

Affects: `docs/architecture/10_AI_Gateway_and_Model_Routing.md` and
`11_AI_Copilot_Tool_Architecture.md` (written when Phase 12 begins),
`15_Database_and_Data_Architecture.md` §45–§47.

Owned by: Phase 13 / Sprint 29 (AI change proposals & approval).

## Rollback / exit strategy

**Effectively irreversible in the safety direction** — relaxing it after launch
would expose the model to corruption with no audit trail. Treat as permanent.

## Verification

- Test: the AI module has no write path to `building_elements` (repository
  surface does not expose one).
- Test: ChangeSet approved against a stale `baseVersion` returns
  `MODEL_VERSION_CONFLICT`.
- Test: a document containing instruction-shaped text does not cause tool invocation.
- Test: cross-tenant RAG retrieval returns nothing.
- Test: tool called with another organization's project ID is rejected.
- Review: any LLM-generated SQL reaching a database is a BLOCKER.

## References

- `AGENTS.md` §16, §17, §19, §40, §41
- `CLAUDE.md` §25, §26, §31
- ADR-001, ADR-002, ADR-010, ADR-015
