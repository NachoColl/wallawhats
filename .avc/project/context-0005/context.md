# Epic: Administration and Compliance Operations

## Identity
- id: context-0005
- domain: administration
- stories: 5

## Purpose
This epic provides protected administration workflows for a local-first CRM and appointment management system used by small-to-medium business operational teams. It enables administrator users to manage local user accounts, role changes, integration credential metadata, and privacy-related customer operations through a dedicated admin experience, while supporting auditable handling of sensitive actions appropriate to a Docker-deployed small-team application.

## Scope

### In Scope
- user-account-management (administrator-controlled create/update/deactivate local users)
- role-assignment (administrator-only owner/manager/staff role changes with self-protection)
- integration-credential-metadata (administrator-only local configuration and secret hygiene workflows)
- customer-data-export (GDPR/CCPA access request export of customer-related data)
- customer-data-deletion (privacy deletion workflow with auditable execution)
- consent-and-retention-operations (consent status and retention-ready data handling)
- admin-react-console (protected user and settings screens with React Query and inline errors)

### Out of Scope
- Public self-service registration, customer-facing account management, or non-admin identity flows.
- Any role model expansion beyond owner/manager/staff role changes and administrator-controlled actions stated in this epic.
- Legal policy authoring, regulatory certification, or guarantees of GDPR/CCPA legal compliance beyond the operational capabilities explicitly described here.
- Exact retention schedules, deletion policies, consent schema details, or privacy export file formats not specified in the epic data.
- Cloud-hosted administration features, production SRE capabilities, or enterprise IAM features such as SSO, SCIM, or external identity provider integration.
- Mobile-specific administration experiences.
- Broader customer lifecycle features unrelated to export, deletion, consent, or retention operations.

## Features
- user-account-management (administrator-controlled create/update/deactivate local users)
- role-assignment (administrator-only owner/manager/staff role changes with self-protection)
- integration-credential-metadata (administrator-only local configuration and secret hygiene workflows)
- customer-data-export (GDPR/CCPA access request export of customer-related data)
- customer-data-deletion (privacy deletion workflow with auditable execution)
- consent-and-retention-operations (consent status and retention-ready data handling)
- admin-react-console (protected user and settings screens with React Query and inline errors)

## Non-Functional Requirements
- Authorization enforcement must reliably restrict permission-sensitive write actions in this epic to administrator-controlled flows, consistent with the epic description and protected admin interface behavior.
- Sensitive operations must fail safely: forbidden, validation, stale-data, and destructive-action errors must be surfaced explicitly in the admin UI and returned clearly by backend endpoints rather than failing silently.
- Data integrity must be preserved for user, role, configuration metadata, consent, export, and deletion operations; writes in SQLite should not leave partially updated records for sensitive admin actions.
- Auditable actions described in the epic must produce locally reviewable audit visibility sufficient for a small team to understand who performed the action, what action occurred, and when it happened.
- Secret hygiene workflows must avoid unnecessary exposure of secret values in API responses or UI rendering, since this epic includes integration credential metadata and secret hygiene workflows.
- Privacy operations must operate only on intended customer-related data and avoid including unrelated customer records in export or deletion handling.
- Protected admin screens should remain usable in a local Docker deployment with reasonable response times for common user lists, settings retrieval, and administrative writes expected in a small-team CRM.
- The implementation must work within the existing local stack described by the project context: React, TypeScript, FastAPI, SQLite, Docker, REST, and related local services, without requiring cloud-only infrastructure.
- Basic security hygiene must be maintained for admin endpoints and screens, including prevention of unauthorized access and minimization of sensitive data leakage in normal responses.
- Accessibility should be maintained for the admin console workflows because the epic description explicitly references protected screens with inline errors and an admin interface used for operational tasks.

## Dependencies

### Required
- context-0001
- context-0002
- context-0003
- context-0004

### Optional
- (none)

## Success Criteria
- All listed administration features are implemented in a way that allows administrator-controlled execution of the intended workflows through the protected system interface.
- Unauthorized or forbidden attempts for sensitive admin actions are blocked and surfaced clearly, including role-related self-protection cases and other explicit error states described by the epic.
- User account management and role assignment flows complete without leaving inconsistent account or role data in SQLite.
- Customer data export and customer data deletion can be performed for privacy operations with auditable execution and without affecting unrelated customer records.
- The admin React console provides protected user and settings screens using React Query and presents inline error handling for forbidden, validation, stale-data, and destructive-action scenarios.

## Stories Overview
- context-0005-0001: Create and Deactivate User Accounts
- context-0005-0002: Change User Roles with Self-Modification Protection
- context-0005-0003: Manage Messaging Integration Credential Metadata
- context-0005-0004: Export Customer Data for Privacy Requests
- context-0005-0005: Delete Customer Data for Privacy Requests