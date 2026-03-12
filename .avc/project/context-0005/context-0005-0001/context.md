# Story: Create and Deactivate User Accounts

## Identity
- id: context-0005-0001
- epic: context-0005 (Administration and Compliance Operations)
- userType: administrators

## User Story
As an administrator, I want to create local user accounts for operational staff and deactivate existing accounts when needed so that the business can control who has system access without risking accidental administrator lockout.

## Summary
This story implements protected admin workflows for provisioning and deactivating local user accounts in the CRM's administration interface. It supports the parent epic's goal of administrator-controlled account management by combining backend authorization and validation with frontend admin-console behaviors for refresh and inline error handling. The value to administrators is fast, auditable control over staff access using the local application stack, while preserving account integrity and preventing self-deactivation.

## Scope

### In Scope
- Administrator-only creation of local user accounts through a protected backend endpoint.
- Storage of created account records in SQLite through the FastAPI backend.
- Password hashing with argon2 for newly created accounts.
- Validation of user creation input for required name, maximum name length, valid and unique email, allowed role values, and local password minimum policy.
- Explicit backend responses for unauthenticated, forbidden, validation, and duplicate-email cases during account creation.
- Administrator-only deactivation of an existing user account through a protected backend endpoint.
- Self-deactivation protection that rejects attempts by an authenticated administrator to deactivate their own account.
- Successful deactivate response including inactive state and deactivation timestamp.
- Audit event emission for successful user creation and deactivation, including required audit fields and exclusion of sensitive secret material.
- React admin user list invalidation and refetch after successful create or deactivate actions.
- Inline admin UI handling for 403 permission errors, 422 field validation errors, and 404 stale-row responses.

### Out of Scope
- Public registration, self-service sign-up, or any customer-facing identity flows.
- Login, session management, JWT issuance, OTP setup, password reset, or broader authentication features beyond enforcing authenticated and authorized access to these admin actions.
- Editing user details other than deactivation.
- Reactivation of deactivated users.
- Role-change workflows beyond selecting an allowed role value at account creation; ongoing role changes are handled by a different story in this epic.
- Owner, manager, or staff administrative screens or permissions to perform these write actions.
- Expanded identity or enterprise IAM features such as SSO, SCIM, or external identity provider integration.
- Bulk user import/export, bulk deactivation, or advanced lifecycle management.
- Audit log review UI; this story covers emitting audit events, not viewing them.
- Handling of admin settings, integration credential metadata, or privacy operations covered by other stories in the epic.

## Acceptance Criteria
1. POST /api/admin/users (administrator role only; owner, manager, and staff receive 403 { error: "FORBIDDEN" }) accepts { name, email, role, password } and returns 201 { id, name, email, role, active, createdAt }
2. name is required max 100, email must be valid and unique, role must be one of administrator|owner|manager|staff, and password must meet the local minimum policy; validation failures return 422 { error: "VALIDATION_ERROR", fieldErrors: [...] }
3. Duplicate email returns 409 { error: "EMAIL_ALREADY_EXISTS" }; unauthenticated requests return 401 { error: "UNAUTHENTICATED" }
4. POST /api/admin/users/:userId/deactivate (administrator role only) returns 200 { id, active: false, deactivatedAt } and rejects self-deactivation where :userId matches the authenticated administrator with 403 { error: "CANNOT_DEACTIVATE_SELF" }
5. On successful create or deactivate, emit audit events user.created and user.deactivated with fields { actorId, targetUserId, occurredAt, role? } and exclude password hashes, raw passwords, JWTs, and OTP secrets
6. After a successful 201 or 200 response, the React admin user list invalidates and refetches; on 403 it shows an inline permission banner, on 422 maps field errors to inputs, and on 404 stale-row responses remove or refresh the affected row

## Technical Notes
- This story spans both the React frontend and FastAPI backend because the acceptance criteria require protected admin UI behavior as well as REST endpoints and SQLite-backed writes.
- The project is a local Docker-deployed web application, so implementation should favor straightforward local data flow, server-side validation, and reliable session/auth checks rather than cloud-specific infrastructure.
- Authorization must strictly enforce administrator-only access for create and deactivate actions, consistent with the epic's requirement that permission-sensitive writes are restricted and fail clearly with explicit errors.
- User-account writes should preserve SQLite data integrity for sensitive admin operations; create and deactivate paths should avoid partial updates and support auditable outcomes.
- Because the project has a frontend, the admin console should handle mutation state and list refresh behavior predictably, including inline mapping of validation errors and stale-row handling after backend responses.
- Security hygiene is especially relevant here: passwords must be hashed with argon2 before storage, and audit events or API responses must not leak password hashes, raw passwords, JWTs, or OTP secrets.

## Dependencies
- context-0001-0002: Required upstream functionality likely provides the authentication and authorization foundation needed to identify the current user, enforce administrator-only access, and detect self-deactivation attempts.