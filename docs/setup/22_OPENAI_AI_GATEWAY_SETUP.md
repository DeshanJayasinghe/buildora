# OpenAI / Buildora AI Gateway Setup

Production AI should be accessed through the Buildora AI Gateway, never directly from random UI modules.

## 1. API key

Create an API project/key through your production AI provider account.

Local environment:

```text
OPENAI_API_KEY=
```

Never place this in:

- browser-exposed variables,
- Next.js public env variables,
- Git,
- screenshots,
- agent prompts.

## 2. Install SDK in API/AI Gateway

```bash
pnpm --filter api add openai
```

## 3. Create provider abstraction

Suggested conceptual interface:

```ts
interface AiProvider {
  respond(request: AiRequest): Promise<AiResponse>;
}
```

Implementation:

```text
OpenAiProvider
```

Domain packages must not import the OpenAI SDK.

## 4. Model aliases

Environment/configuration:

```text
AI_MODEL_FAST=
AI_MODEL_BALANCED=
AI_MODEL_ADVANCED=
AI_MODEL_EXPERT=
```

Do not hardcode product marketing names into domain logic.

## 5. Start with one simple request

Use server-side Responses API through the official SDK.

Before adding tools, verify:

- auth works,
- timeout works,
- request ID logged,
- token usage captured where returned,
- errors normalized.

## 6. Add router

Input signals:

- task type
- complexity
- affected elements
- context size
- plan limits
- financial/design risk

Output:

```text
model alias
reasoning level
max budget
tool policy
```

## 7. Add first read-only tool

Recommended:

```text
get_project_summary
```

Then:

```text
get_room
get_element
get_quantities
get_boq
get_cost
```

Only after read-only tools are reliable should AI propose geometry changes.

## 8. Change tools

AI generates structured:

```text
ProposedChangeSet
```

It never directly writes project tables.

## 9. Usage ledger

Record:

- user
- organization
- project
- model alias
- provider model
- task
- input/output usage
- cost
- credits reserved/charged
- latency
- success/failure

## 10. Testing

Use a fake provider in most tests.

Real API tests should be opt-in and budget-limited.
