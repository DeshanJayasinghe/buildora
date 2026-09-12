# First AI Copilot Vertical Slice

Only start once Building Model, QS, BOQ and Cost are trustworthy.

## Phase 1 — Read-only AI

Tools:

```text
get_project_summary
get_room
get_element
get_quantities
get_boq
get_cost
```

User example:

```text
Why is this wall expensive?
```

AI retrieves data using tools and explains it.

Do not let AI invent rates or quantities.

## Phase 2 — Document tool

Add:

```text
search_project_documents
```

only after RAG provenance is working.

## Phase 3 — Proposed geometry change

Add:

```text
propose_move_wall
```

AI returns a ChangeSet.

It does not commit.

## Phase 4 — Preview

System applies ChangeSet to draft/sandbox model.

Run:

- geometry validation
- quantities
- BOQ
- cost.

Show:

```text
Before
After
Cost impact
Quantity impact
Warnings
```

## Phase 5 — Approval

User presses:

```text
Apply
```

Only then commit domain command(s).

## AI routing

Start simple:

```text
FAST
BALANCED
ADVANCED
EXPERT
```

Use configuration aliases.

Do not overbuild routing ML.

## Testing

Use fake AI outputs for deterministic tests.

Contract-test:

- tool schemas
- malformed tool arguments
- unauthorized project
- stale model version
- rejected ChangeSet
- failed recalculation.

## Acceptance

AI cannot directly mutate authoritative model rows or override deterministic calculations.
