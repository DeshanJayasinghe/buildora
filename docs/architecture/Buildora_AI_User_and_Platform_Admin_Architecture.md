# Buildora AI — User Roles & Platform Admin Architecture

> **Document purpose:** Define the customer-facing user model, organization/project authorization model, and internal Buildora Platform Admin structure.
>
> **Product:** Buildora AI
>
> **Repository:** `buildora-ai`
>
> **Status:** Architecture reference
>
> **Applies to:** Authentication, authorization, organizations, projects, billing, AI usage, support tooling, internal administration, and audit.

---

# 1. Purpose

Buildora AI serves both professional AEC users and non-professional clients.

The platform must clearly separate:

1. **Professional Persona** — what the person does in the construction/design process.
2. **Customer Access Role** — what the person is allowed to do inside an organization/project.
3. **Platform Admin Role** — what an internal Buildora team member is allowed to do across the SaaS platform.

These concepts must not be mixed.

A user's profession must not directly determine authorization.

Example:

```text
Professional Persona:
Quantity Surveyor

Organization Role:
Organization Member

Project Role:
Project Editor
```

Another example:

```text
Professional Persona:
Property Developer

Organization Role:
Organization Owner

Project Role:
Project Manager
```

Authorization is permission-based, not profession-based.

---

# 2. High-Level User Architecture

```text
Buildora AI
│
├── Customer Application
│   ├── Individual Users
│   ├── Organization Users
│   └── External Project Participants
│
│   Authorization
│   ├── Organization Roles
│   └── Project Roles / Permissions
│
└── Platform Admin
    ├── Platform Super Admin
    ├── Support Admin
    ├── Billing Admin
    └── Operations Admin
```

The customer application and Platform Admin authorization domains are separate.

An `Organization Owner` is **not** a Buildora Platform Admin.

A Buildora Platform Admin is **not automatically authorized to access customer project content**.

---

# 3. Customer User Personas

Professional personas describe how a user uses Buildora AI.

They are primarily for:

- onboarding,
- personalization,
- dashboards,
- feature recommendations,
- analytics,
- marketing,
- default workspace configuration.

They are **not** the primary authorization mechanism.

## 3.1 Architect / Building Designer

Typical use:

- create building projects,
- draw/edit 2D plans,
- review 3D models,
- use AI design assistance,
- manage room layouts,
- manage materials and finishes,
- review design changes,
- export plans/reports,
- collaborate with clients and consultants.

Typical important features:

```text
2D CAD
3D
AI Copilot
AI ChangeSets
Materials
Room/Space management
Documents
Reports
```

## 3.2 Quantity Surveyor / Estimator

Typical use:

- quantity takeoff,
- review element quantities,
- manage measurement rules,
- create BOQ,
- manage rates,
- prepare cost estimates,
- compare revisions,
- value-engineering analysis,
- cost reporting.

Typical important features:

```text
QS
BOQ
Cost
Rate Books
Assemblies
Estimate Versions
Revision Comparison
Reports
AI Cost Explanation
```

## 3.3 Contractor / Builder

Typical use:

- review project drawings,
- inspect model quantities,
- review BOQ,
- estimate construction cost,
- evaluate material alternatives,
- identify value-engineering opportunities,
- review construction documents,
- later manage project execution workflows.

## 3.4 Property Developer

Typical use:

- create/manage development projects,
- assess design feasibility,
- evaluate project cost,
- compare design alternatives,
- monitor changes,
- review reports,
- collaborate with architect/QS/contractor.

## 3.5 Homeowner / Client

Typical use:

- create an initial project,
- review proposed designs,
- view 2D and 3D,
- understand estimated costs,
- ask AI questions,
- review documents/reports,
- collaborate with professionals.

Homeowners should normally see a simplified experience compared with professional users.

## 3.6 Engineer / Consultant

Examples:

- structural engineer,
- MEP engineer,
- sustainability consultant,
- specialist consultant.

Typical use:

- review project geometry,
- access relevant drawings/documents,
- provide project information,
- collaborate with design/construction teams,
- later use discipline-specific Buildora modules.

## 3.7 Project Manager

Typical use:

- manage project access,
- coordinate team members,
- review project status,
- monitor documents,
- review design revisions,
- review costs,
- later manage schedule/procurement/change workflows.

## 3.8 External Client / Stakeholder

Typical use:

- view approved project information,
- view selected 2D/3D content,
- view selected reports,
- comment/review,
- approve decisions where enabled.

External users must not automatically see:

- internal margins,
- rate books,
- internal project notes,
- organization billing,
- confidential documents.

---

# 4. Recommended MVP Personas

The initial product should focus primarily on:

```text
Architect / Building Designer
Quantity Surveyor / Estimator
Contractor / Builder
Property Developer
Homeowner / Client
```

Other personas can use the platform through generic project roles until specialist modules are introduced.

---

# 5. Individual Users and Organizations

Buildora AI should support both individual and organization-based use.

## 5.1 Personal Workspace

An individual user can operate without first creating a traditional company organization.

Example:

```text
User
└── Personal Workspace
    └── Projects
```

Implementation recommendation:

A personal workspace may still be represented internally as a single-member organization/workspace so that tenancy architecture remains consistent.

## 5.2 Organization Workspace

Example:

```text
ABC Construction Ltd
├── Organization Owner
├── Organization Admins
├── Organization Members
└── Projects
```

An organization owns:

- projects,
- subscriptions,
- entitlements,
- AI credits,
- render credits,
- members,
- rate books,
- organization settings,
- organization-level audit records.

## 5.3 Multi-Organization Membership

A user may belong to multiple organizations.

Never store a single global `role` on the user and assume it applies everywhere.

---

# 6. Customer Authorization Model

Authorization should have two layers:

```text
Organization Authorization
        ↓
Project Authorization
```

Fine-grained permissions may further restrict sensitive capabilities.

---

# 7. Organization Roles

Recommended initial organization roles:

```text
ORGANIZATION_OWNER
ORGANIZATION_ADMIN
ORGANIZATION_MEMBER
```

## 7.1 Organization Owner

Typical permissions:

- manage organization settings,
- manage membership,
- manage subscription,
- manage billing,
- manage projects,
- access organization-wide usage,
- manage AI/credit usage policies,
- delete organization,
- transfer ownership.

## 7.2 Organization Admin

Typical permissions:

- create/manage projects,
- invite/manage members,
- manage project access,
- view organization usage,
- manage selected organization settings,
- optionally manage billing if permission is granted.

## 7.3 Organization Member

Normal organization participant.

Access depends primarily on project membership and project permissions.

---

# 8. Project Roles

Recommended project roles:

```text
PROJECT_MANAGER
PROJECT_EDITOR
PROJECT_VIEWER
EXTERNAL_CLIENT
```

## 8.1 Project Manager

Typical permissions:

- read project,
- edit project settings,
- edit Building Model,
- manage project members,
- access quantities,
- access costs when granted,
- manage documents,
- create reports,
- use AI,
- manage project workflows.

## 8.2 Project Editor

Typical permissions:

- read project,
- edit Building Model,
- upload/manage working documents,
- use AI tools,
- view QS,
- possibly view cost depending on explicit permission.

## 8.3 Project Viewer

Typical permissions:

- read project,
- view selected 2D/3D,
- view selected documents/reports,
- view quantities when allowed.

No model mutation.

## 8.4 External Client

Restricted external participant.

Possible permissions:

- view project summary,
- view approved drawings,
- view approved 3D,
- view selected reports,
- comment,
- approve/reject review items.

Sensitive access such as cost, internal documents and margins must be explicitly granted.

---

# 9. Fine-Grained Permissions

Recommended permission namespaces:

```text
organization.read
organization.manage
organization.delete

organization.members.read
organization.members.invite
organization.members.manage

organization.billing.read
organization.billing.manage
organization.usage.read

project.create
project.read
project.manage
project.delete

project.members.read
project.members.manage

model.read
model.edit
model.version.read
model.version.restore

ai.use
ai.modify
ai.approve_changes

qs.read
qs.run

boq.read
boq.edit

cost.read
cost.edit
cost.margin.read
cost.ratebook.read
cost.ratebook.manage

documents.read
documents.upload
documents.manage

reports.read
reports.create
reports.export

comments.read
comments.create

audit.read
```

---

# 10. Cost Visibility Is a Separate Permission

Do not assume:

```text
project.read = cost.read
```

Similarly:

```text
cost.read != cost.margin.read
```

A client may be allowed to see an estimated project total without seeing supplier cost, contractor markup, margin, or internal rates.

---

# 11. Example Customer Permission Matrix

| Capability | Org Owner | Org Admin | Project Manager | Project Editor | Project Viewer | External Client |
|---|---:|---:|---:|---:|---:|---:|
| View project | ✓ | ✓* | ✓ | ✓ | ✓ | ✓ |
| Edit model | ✓ | ✓* | ✓ | ✓ | — | — |
| Use AI Copilot | ✓ | ✓* | ✓ | ✓ | Optional | Optional |
| Commit AI changes | ✓ | ✓* | ✓ | ✓ | — | — |
| View QS | ✓ | ✓* | ✓ | ✓ | Optional | Optional |
| Run QS | ✓ | ✓* | ✓ | ✓ | — | — |
| View cost | ✓ | ✓* | ✓ | Permission | Permission | Permission |
| Edit cost | ✓ | ✓* | Permission | — | — | — |
| View internal margin | ✓ | Permission | Permission | — | — | — |
| Upload documents | ✓ | ✓* | ✓ | ✓ | — | Optional |
| Manage project members | ✓ | ✓* | ✓ | — | — | — |
| Manage organization members | ✓ | ✓ | — | — | — | — |
| Manage billing | ✓ | Permission | — | — | — | — |

`✓*` means subject to project-access policy.

---

# 12. Professional Persona vs Authorization Role

Never use:

```text
if user.profession == "ARCHITECT":
    allowModelEdit()
```

Use:

```text
if permissions.includes("model.edit"):
    allowModelEdit()
```

Personas may be stored separately for product customization.

---

# 13. Platform Admin Architecture

Buildora AI requires a separate internal administration capability for the Buildora operating team.

It is separate from customer organization administration.

---

# 14. Platform Admin Roles

Recommended initial roles:

```text
PLATFORM_SUPER_ADMIN
PLATFORM_SUPPORT_ADMIN
PLATFORM_BILLING_ADMIN
PLATFORM_OPERATIONS_ADMIN
```

---

# 15. Platform Super Admin

Typical capabilities:

- manage platform administrators,
- view/manage users,
- view/manage organizations,
- suspend accounts,
- manage subscriptions,
- manage billing corrections,
- manage credits,
- manage platform configuration,
- manage AI models/rate cards,
- manage feature flags,
- inspect jobs,
- inspect webhooks,
- inspect audit logs,
- perform restricted support operations.

All consequential actions must be audited.

---

# 16. Platform Support Admin

Typical capabilities:

- search users,
- search organizations,
- inspect project metadata,
- inspect membership issues,
- inspect failed jobs,
- view non-sensitive usage data,
- trigger safe retries,
- assist with account-access issues.

Support Admin should not automatically have billing-adjustment or unrestricted customer-content access.

---

# 17. Platform Billing Admin

Typical capabilities:

- inspect subscriptions,
- inspect invoices/payment state,
- inspect credit wallets,
- inspect credit ledger,
- perform controlled adjustments,
- process approved refunds,
- investigate webhook/reconciliation issues,
- view usage-to-cost records,
- review plan/entitlement state.

---

# 18. Platform Operations Admin

Typical capabilities:

- inspect background jobs,
- retry failed jobs,
- inspect workflow status,
- inspect provider failures,
- inspect AI usage,
- inspect storage/processing status,
- inspect system health,
- manage feature flags where authorized,
- inspect incidents.

---

# 19. Platform Admin Permissions

Recommended permission namespace:

```text
platform.dashboard.read

platform.users.read
platform.users.manage
platform.users.suspend

platform.organizations.read
platform.organizations.manage
platform.organizations.suspend

platform.projects.metadata.read

platform.subscriptions.read
platform.subscriptions.manage

platform.billing.read
platform.billing.adjust
platform.billing.refund

platform.credits.read
platform.credits.adjust

platform.ai_usage.read
platform.ai_models.read
platform.ai_models.manage
platform.rate_cards.read
platform.rate_cards.manage

platform.jobs.read
platform.jobs.retry
platform.jobs.cancel

platform.webhooks.read
platform.webhooks.retry

platform.feature_flags.read
platform.feature_flags.manage

platform.audit.read
platform.system_health.read

platform.support.access

platform.customer_content.read
platform.customer_content.download
```

Customer-content permissions are highly sensitive.

---

# 20. Customer Content Access Policy

Platform administration must not imply unrestricted customer-content access.

Normal support may inspect metadata such as:

```text
Project ID
Project Name
Organization ID
Owner
Current Model Version
Last Updated
Storage Usage
Job Status
Subscription
Entitlements
AI Usage Totals
```

This is different from:

```text
Building Model
Uploaded Drawings
Contracts
Specifications
Cost Data
AI Conversations
Private Documents
```

Customer-content access must require explicit permission and audit.

---

# 21. Support Access / Break-Glass Model

Future controlled support access:

```text
Support Admin
        ↓
requests customer-content access
        ↓
policy check
        ↓
time-limited support session
        ↓
access logged
        ↓
automatic expiry
```

For MVP, it is acceptable to disable customer-content access until this workflow exists.

---

# 22. Platform Admin Audit Events

Examples:

```text
PLATFORM_USER_SUSPENDED
PLATFORM_USER_REACTIVATED
PLATFORM_ORGANIZATION_SUSPENDED
PLATFORM_SUBSCRIPTION_CHANGED
PLATFORM_CREDIT_ADJUSTED
PLATFORM_REFUND_TRIGGERED
PLATFORM_JOB_RETRIED
PLATFORM_JOB_CANCELLED
PLATFORM_FEATURE_FLAG_CHANGED
PLATFORM_AI_RATE_CARD_CHANGED
PLATFORM_ADMIN_ROLE_CHANGED
PLATFORM_CUSTOMER_CONTENT_ACCESSED
PLATFORM_CUSTOMER_FILE_DOWNLOADED
```

Audit records should include actor, action, target, organization/project where applicable, timestamp, reason, metadata, and request/trace ID.

---

# 23. Platform Admin UI Structure

```text
/admin
├── dashboard
├── users
│   └── [userId]
├── organizations
│   └── [organizationId]
├── projects
│   └── [projectId]
├── subscriptions
├── billing
├── credits
├── ai-usage
├── ai-models
├── rate-cards
├── jobs
├── webhooks
├── feature-flags
├── audit
├── system-health
└── settings
```

Not every route must be implemented in the MVP.

---

# 24. Recommended MVP Admin Panel

Initial:

```text
/admin/dashboard
/admin/users
/admin/organizations
/admin/projects
/admin/subscriptions
/admin/credits
/admin/ai-usage
/admin/jobs
/admin/webhooks
/admin/audit
```

Later:

```text
AI model management
Rate card management
Feature flags
Advanced billing tools
System health dashboards
Support-access workflow
```

---

# 25. Platform Admin Dashboard

Useful initial metrics:

```text
Total Users
Active Users
Total Organizations
Active Organizations
Total Projects
Paid Subscriptions
MRR / ARR
AI Jobs Today
AI Provider Cost
AI Credits Consumed
Failed Jobs
Failed Webhooks
Storage Usage
```

---

# 26. User Management

Possible data:

```text
User ID
Name
Email
Account Status
Created Date
Last Login
Organizations
Usage Summary
Support Flags
```

Actions:

```text
Suspend
Reactivate
View memberships
View support history
```

Avoid casual destructive deletion.

---

# 27. Organization Management

Possible data:

```text
Organization ID
Name
Owner
Members
Plan
Subscription Status
Project Count
Storage Usage
AI Usage
Credit Balances
Created At
Status
```

---

# 28. Project Administration

Default platform-admin project view should be metadata-first:

```text
Project ID
Name
Organization
Status
Model Version
Element Count
Storage Usage
Last Activity
Processing Jobs
```

Do not automatically expose full customer project content.

---

# 29. Billing Administration

Internal billing area:

```text
Plans
Subscriptions
Invoices / Provider References
Entitlements
Wallets
Credit Ledger
Reservations
Usage Events
Refunds / Adjustments
Webhook Events
```

Every manual financial adjustment requires authorization, reason, idempotency, immutable ledger entry, and audit.

---

# 30. AI Usage Administration

Track:

```text
AI Job
Provider
Model Alias
Provider Model
Input Tokens
Cached Tokens
Output Tokens
Provider Cost
Customer Credits Charged
Organization
User
Project
Operation Type
Latency
Status
```

Useful views:

```text
Cost by Organization
Cost by Plan
Cost by AI Operation
Cost by Model
High-Cost Customers
Failed AI Jobs
Credit vs Provider-Cost Margin
```

---

# 31. AI Model & Rate Card Administration

Conceptual configuration:

```text
Logical Alias
Provider
Provider Model ID
Input Rate
Cached Input Rate
Output Rate
Effective From
Effective To
Verified At
Source Reference
```

Changes must be audited.

Historical AI cost must continue using the rate version effective at the time of usage.

---

# 32. Jobs / Workflow Administration

Admin should inspect:

```text
job_id
job_type
organization_id
project_id
status
attempt
started_at
completed_at
duration
error_code
workflow_id
```

Actions may include safe Retry and Cancel.

Retries must not duplicate credit charges, model mutations, imports, or billing actions.

---

# 33. Webhook Administration

Inspect:

```text
Provider
Event ID
Event Type
Received At
Processing Status
Attempts
Last Error
```

Retries must use existing idempotency controls.

---

# 34. Feature Flags

Feature flags are platform configuration, not customer authorization.

Examples:

```text
AI_DESIGN_GENERATION
PLAN_RECOGNITION
IFC_IMPORT
EXPERIMENTAL_WEBGPU
NEW_COST_ENGINE
```

Feature flags must not be used as a substitute for permission checks.

---

# 35. Platform Admin Frontend Deployment

For MVP, keep the Platform Admin UI inside:

```text
apps/web
```

Example:

```text
apps/web/
└── app/
    ├── (customer)/
    └── admin/
```

Do not create `apps/admin` initially.

---

# 36. Future Admin App Extraction

A separate `apps/admin` may be justified later when:

- a dedicated support/operations team exists,
- deployment cadence differs,
- stronger network isolation is required,
- the internal application becomes significantly larger.

The backend authorization model should allow this without redesign.

---

# 37. Backend Module Structure

Conceptual API structure:

```text
apps/api/src/modules/
├── identity/
├── organizations/
├── projects/
├── model/
├── documents/
├── qs/
├── cost/
├── ai/
├── billing/
└── platform-admin/
```

Platform Admin should orchestrate existing domain use cases rather than bypassing domain rules.

---

# 38. Authentication vs Authorization

Authentication answers:

```text
Who are you?
```

Authorization answers:

```text
What are you allowed to do here?
```

Buildora remains authoritative for organization membership, project membership, customer permissions, and platform-admin permissions.

---

# 39. Platform Admin Authentication Security

Platform Admin should require stronger controls.

Recommended:

```text
MFA required
shorter sessions
secure session/device handling
audit logging
restricted role assignment
```

Future:

```text
SSO
IP/network restrictions
step-up authentication
support-access approval
```

---

# 40. Conceptual Data Model

Customer authorization:

```text
users

organizations

organization_memberships
- user_id
- organization_id
- role

projects

project_memberships
- user_id
- project_id
- role
```

Platform authorization:

```text
platform_admin_memberships
- user_id
- platform_role
- status

platform_admin_audit_events
```

---

# 41. Avoid Global User Role

Avoid:

```text
users.role = 'ADMIN'
```

Use contextual authorization:

```text
organization_memberships.role
project_memberships.role
platform_admin_memberships.role
```

---

# 42. Authorization Evaluation

```text
Authenticated User
        ↓
Is Platform Admin route?
        │
   yes ─┴─ no
    ↓       ↓
Platform    Organization
Permissions Membership
            ↓
        Project Membership
            ↓
        Fine Permissions
```

Never infer platform access from customer roles.

---

# 43. Authorization Rules

Every sensitive request must be evaluated server-side.

Frontend hiding is UX, not authorization.

---

# 44. API Boundary Recommendation

Customer routes conceptually:

```text
/api/organizations/...
/api/projects/...
/api/model/...
/api/qs/...
/api/cost/...
```

Platform Admin:

```text
/api/platform-admin/...
```

---

# 45. Tenant Isolation

Normal customer repositories stay tenant-scoped.

Platform Admin cross-tenant operations should use clearly separate query/application paths.

Do not weaken ordinary repositories to support admin search.

---

# 46. Internal Support Queries

Prefer explicit cross-tenant query services such as:

```text
PlatformOrganizationQueryService
PlatformUserQueryService
```

Avoid generic flags such as:

```text
skipTenantCheck=true
```

throughout normal repositories.

---

# 47. Suspension Model

Prefer:

```text
ACTIVE
SUSPENDED
PENDING_DELETION
```

over destructive deletion.

---

# 48. Deletion

Deletion should be a governed workflow considering retention, storage cleanup, billing records, legal requirements, and audit history.

Platform Admin must not perform casual hard deletes.

---

# 49. Impersonation

Do not implement unrestricted admin impersonation initially.

If introduced later:

- explicit action,
- limited duration,
- prominent indicator,
- restricted operations,
- full audit.

---

# 50. Architecture Invariants

1. Professional persona is not authorization.
2. Organization roles are separate from project roles.
3. Platform Admin roles are separate from customer roles.
4. Organization remains the customer tenant boundary.
5. A user may belong to multiple organizations.
6. Project access is explicitly evaluated.
7. Cost visibility is independently permissioned.
8. Platform Admin does not automatically receive customer-content access.
9. Cross-tenant platform actions are explicit and audited.
10. Financial changes always use the billing ledger.
11. Admin retries remain idempotent.
12. Frontend visibility is not authorization.
13. Backend checks every sensitive permission.
14. Customer and platform-admin audit trails are preserved.
15. No global ambiguous `users.role = ADMIN` model.
16. Platform admin permissions follow least privilege.
17. Internal admin UI remains inside `apps/web` initially.
18. Separate `apps/admin` is deferred until real need.
19. Customer data access by Buildora staff is controlled and auditable.
20. Platform administration reuses domain use cases rather than bypassing domain rules.

---

# 51. Recommended Initial Implementation Order

```text
Identity
    ↓
Organization Membership
    ↓
Project Membership
    ↓
Permission Evaluation
    ↓
Customer RBAC
    ↓
Audit Foundation
    ↓
Basic Platform Admin Authorization
    ↓
Users / Organizations Admin Search
```

Then add admin capabilities alongside their domains:

```text
Billing ships → Billing Admin
AI ships → AI Usage Admin
Workers ship → Job Admin
Feature Flags ship → Feature Flag Admin
```

---

# 52. MVP Recommendation

Customer authorization MVP:

```text
Organization Owner
Organization Admin
Organization Member

Project Manager
Project Editor
Project Viewer
External Client

fine-grained cost permissions
```

Platform Admin MVP:

```text
Platform Super Admin
Platform Support Admin

Dashboard
Users
Organizations
Projects metadata
Audit
```

When billing arrives:

```text
Platform Billing Admin
Subscriptions
Credits
Ledger
Webhooks
```

When AI/workflows arrive:

```text
Platform Operations Admin
AI Usage
Jobs
Provider/Rate Configuration
```

---

# 53. Final Structure

```text
                           BUILDORA AI
                                │
             ┌──────────────────┴──────────────────┐
             │                                     │
             ▼                                     ▼
      CUSTOMER PLATFORM                      PLATFORM ADMIN
             │                                     │
     Professional Persona                     Internal Roles
             │                                     │
             ▼                                     ▼
      Organization Role                    Super Admin
             │                             Support Admin
             ▼                             Billing Admin
        Project Role                       Operations Admin
             │                                     │
             ▼                                     ▼
      Fine Permissions                    Platform Permissions
             │                                     │
             └──────────────────┬──────────────────┘
                                ▼
                        Server Authorization
                                │
                                ▼
                    Domain / Application Use Cases
                                │
                                ▼
                         Authoritative Data
```

---

# 54. Final Rule

> **Customer roles determine what users may do inside their own organizations and projects. Platform Admin roles determine what Buildora staff may do across the service. Neither role system implicitly grants permissions in the other.**

This separation must remain intact as Buildora AI grows.
