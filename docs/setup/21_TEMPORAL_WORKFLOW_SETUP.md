# Temporal Workflow Setup

Temporal should be added when BuildWise starts long-running/multi-step operations such as plan recognition and BIM processing.

## 1. Install Temporal CLI on macOS

```bash
brew install temporal
```

## 2. Start local dev server

Use persistent local workflow history:

```bash
mkdir -p .temporal
temporal server start-dev --db-filename .temporal/buildwise.db
```

Default service:

```text
localhost:7233
```

Default Web UI:

```text
http://localhost:8233
```

Add `.temporal/` to `.gitignore`.

## 3. Verify

In another terminal:

```bash
temporal operator namespace list
temporal workflow list
```

## 4. Add TypeScript SDK

Create an application/worker package or worker app when needed:

```bash
pnpm add @temporalio/client @temporalio/worker @temporalio/workflow @temporalio/activity
```

Use package placement based on your actual worker architecture.

## 5. First workflow

Do not begin with plan recognition.

Create a trivial workflow:

```text
Start
 ↓
Activity: echo/health
 ↓
Complete
```

Verify it appears in the Temporal UI.

## 6. First real BuildWise workflow

Later:

```text
Upload plan
 ↓
Validate file
 ↓
Extract vectors/image
 ↓
Recognize
 ↓
Persist recognition result
 ↓
WAIT for user verification
 ↓
Convert to Building Model draft
 ↓
Validate
 ↓
Complete
```

## 7. Workflow rule

Temporal Workflows must be deterministic.

Network/database/AI calls belong in Activities.

## 8. Production

For a solo developer, prefer Temporal Cloud before self-hosting a production Temporal cluster unless cost/requirements justify operating it yourself.
