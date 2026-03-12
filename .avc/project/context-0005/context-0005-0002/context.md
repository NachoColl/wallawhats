# Story: Change User Roles with Self-Modification Protection

## Identity
- id: context-0005-0002
- epic: context-0005 (Administration and Compliance Operations)
- userType: administrators

## User Story
As an administrators, I want to change another user's operational role from the protected admin interface while being blocked from changing my own role so that I can manage team permissions safely without accidentally removing my own administrative access.

## Summary
This story implements an administrator-only role management action for existing local user accounts in the admin console. It supports updating another user's operational role to one of the allowed roles, with backend enforcement of valid transitions and explicit protection against self-modification. Within the parent epic, this delivers a core protected administration workflow and ensures role changes are auditable, clearly validated, and reflected promptly in the React user management UI.

## Scope

### In Scope
- Administrator-only role update action for an existing user via `PATCH /api/admin/users/:userId/role`.
- Accepting a request body containing `{ role }` and returning the updated role payload on success.
- Restricting access so only authenticated users with the administrator role can perform the role change.
- Supporting only the explicit role values `administrator`, `owner`, `manager`, and `staff`.
- Returning explicit validation, authentication, authorization, and user-not-found errors for the role change endpoint.
- Preventing an administrator from modifying their own role when `:userId` matches the authenticated user's ID.
- Emitting an audit event for successful role changes including actor, target user, previous role, new role, and timestamp.
- Ensuring audit metadata for this action excludes credentials and session tokens.
- Updating the admin React user table behavior after a successful change by invalidating and refetching data.
- Disabling the edited user row while the role update is pending.
- Showing inline 403 messaging for unauthorized access in the React admin UI.
- Mapping 422 validation errors to the role control in the UI.

### Out of Scope
- Creating new user accounts or deactivating existing users; those belong to the dependent user account management story.
- Editing any user fields other than operational role.
- Allowing non-administrator roles such as owner, manager, or staff to perform role changes.
- Any self-service role changes or profile permission management by end users.
- Expansion of the role model beyond `administrator`, `owner`, `manager`, and `staff`.
- Broader identity and access management features such as SSO, external identity providers, SCIM, or public registration.
- Detailed audit log review screens or reporting interfaces beyond emitting the required audit event.
- Other admin error cases not specified here except where already covered by epic-level protected admin behavior.

## Acceptance Criteria
1. PATCH /api/admin/users/:userId/role (administrator role only; owner, manager, and staff receive 403 { error: "FORBIDDEN" }) accepts { role } and returns 200 { id, role, updatedAt }
2. role must be one of administrator|owner|manager|staff; invalid values return 422 { error: "VALIDATION_ERROR", field: "role" }
3. If :userId does not exist, return 404 { error: "USER_NOT_FOUND" }; unauthenticated requests return 401 { error: "UNAUTHENTICATED" }
4. If :userId matches the authenticated administrator's own user ID, return 403 { error: "CANNOT_MODIFY_SELF" } to prevent accidental privilege loss
5. On success, emit audit event user.role_changed with { actorId, targetUserId, previousRole, newRole, occurredAt } and exclude credentials or session tokens from metadata
6. After a successful 200 response, the React user table invalidates and refetches, disables the edited row while pending, shows inline 403 messaging for unauthorized access, and on 422 maps the returned error to the role control

## Technical Notes
- This story spans both the React/TypeScript admin console and the FastAPI backend, with the frontend handling inline error states and pending-row behavior and the backend enforcing authorization and validation.
- Because the project is a local Docker-deployed application with SQLite, the role update and corresponding audit record should preserve integrity for sensitive admin writes and avoid partially applied changes.
- The endpoint is part of a protected admin workflow in a system without a public API, so server-side authorization cannot rely only on UI restrictions; unauthenticated and non-administrator requests must be explicitly rejected.
- The parent epic requires auditable handling of sensitive administrative actions, so successful role changes must produce locally reviewable audit visibility with actor, action, and time while avoiding leakage of credentials or session tokens.
- Since the project has a frontend, the implementation should align with the epic's admin React console behavior, including data refetch after mutation, row-level pending state, and inline presentation of forbidden and validation errors.
- This story depends on existing user account records from the user management flow, so backend handling should resolve the target user reliably and return `USER_NOT_FOUND` when the referenced user does not exist.

## Dependencies
- context-0005-0001 — required because this story updates roles on existing user accounts created and managed by the user account management workflow.