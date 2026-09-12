# Clerk Authentication and Organizations Setup

For the MVP, Clerk is recommended to reduce solo-development auth workload.

Important architectural rule:

> Clerk proves identity; BuildWise still owns application authorization and project-domain permissions.

Keep Clerk behind an auth adapter.

## 1. Install Clerk in Next.js

The current Clerk CLI can initialize a Next.js project:

```bash
cd apps/web
pnpm dlx clerk@latest init
```

If you already have a Clerk account/application and intentionally want to link it, authenticate through the CLI/dashboard as appropriate.

## 2. Verify

Run Clerk's diagnostic command:

```bash
pnpm dlx clerk@latest doctor
```

Run web app:

```bash
pnpm --filter web dev
```

Create a test user.

## 3. Environment

Clerk will require values such as:

```text
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=
CLERK_SECRET_KEY=
```

Never expose `CLERK_SECRET_KEY` to browser code.

## 4. Organizations

BuildWise has company/team workspaces.

Clerk Organizations can simplify:

- org membership
- invitations
- org switching

However BuildWise should keep a local organization mirror:

```text
organizations
organization_members
```

with external provider IDs.

Why:

- domain references stay stable
- audit/events are local
- provider replacement remains possible
- authorization can combine SaaS plan/project rules

## 5. API authentication

The API must verify tokens server-side.

Do not trust organization/user IDs merely because the browser sent them.

## 6. Authorization

Identity:

```text
Who are you?
```

Authorization:

```text
Can you edit Project X?
```

These are different.

Implement API guards/policies using BuildWise database relationships.

## 7. Initial roles

Start small:

```text
owner
admin
member
viewer
```

Professional role labels can be added separately from security roles.

## 8. Webhook sync

Later add Clerk webhook handling for:

- user created/updated
- org created/updated
- membership changed

Make webhook processing:

- signature verified
- idempotent
- auditable

## 9. Escape hatch

Do not make Building Model records depend on Clerk SDK types.

Use internal IDs and provider mappings.
