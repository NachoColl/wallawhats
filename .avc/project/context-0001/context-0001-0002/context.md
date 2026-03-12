# Story: Role-Aware Navigation and API Authorization

## Identity
- id: context-0001-0002
- epic: context-0001 (Foundation Services)
- userType: all authenticated users

## User Story
As all authenticated users, I want the application to show only the navigation options and actions allowed for my assigned role and to block unauthorized API access with clear responses so that I can use the CRM safely within my permissions and the system can prevent restricted operations.

## Summary
This story implements role-aware behavior across both the React frontend and FastAPI backend for authenticated users. It ensures administrator, owner, manager, and staff roles are enforced consistently by backend authorization dependencies and reflected in frontend navigation, restricted routes, and action availability. Within the Foundation Services epic, this provides the shared authorization layer that other product areas depend on, while giving users clear feedback when they cannot access a feature or when their session is no longer valid.

## Scope

### In Scope
- Enforcing JWT-based authentication on protected API endpoints using the Authorization: Bearer <JWT> header.
- Applying explicit RBAC rules for administrator, owner, manager, and staff roles on backend endpoints.
- Returning 401 for missing or invalid tokens on protected endpoints.
- Returning 403 for authenticated users who lack the required role for an endpoint.
- Exposing authenticated user identity and authorization context through GET /api/auth/me with id, name, email, role, and permissions: string[].
- Using the authenticated role and permissions in the React application to render role-aware navigation.
- Hiding administrator-only navigation items from non-administrators.
- Showing an inline permission message when a user reaches a restricted route instead of leaving the UI in a broken state.
- Clearing cached protected data in React Query and redirecting to login when the frontend receives a 401 caused by an invalid or expired token during navigation.
- Logging audit events for denied administrative actions when authorization dependencies reject access.

### Out of Scope
- Initial sign-in, credential verification, password hashing, and JWT issuance behavior, which are covered by the login story dependency.
- Administrator-only two-factor authentication, login throttling, and other auth hardening features not explicitly part of this story.
- Detailed implementation of customer, messaging, appointment, audit review, user-management, or credential-management business workflows beyond enforcing access rules for them.
- Creating new admin/operator interfaces beyond the role-aware shell behavior described here.
- Public API authorization patterns or third-party developer access, since the project has no public API.
- Cloud-scale authorization concerns such as distributed policy propagation or external identity providers.
- Fine-grained permission editing UX or role administration flows unless needed elsewhere; this story focuses on consuming assigned roles and permissions, not managing them.

## Acceptance Criteria
1. Protected endpoints require Authorization: Bearer <JWT>; requests without a valid token return 401 { error: "UNAUTHENTICATED" }
2. RBAC rules are explicit: administrator can access all endpoints; owner and manager can access customer, messaging, appointment, and audit review endpoints but not integration credential storage unless granted administrator role; staff can access customer, messaging, and appointment workflows but receive 403 { error: "FORBIDDEN" } on user-management and credential-management endpoints
3. GET /api/auth/me (any authenticated role) returns 200 { id, name, email, role, permissions: string[] } so the frontend can render role-aware navigation and disable forbidden actions before submission
4. FastAPI authorization dependencies return 403 { error: "FORBIDDEN" } for authenticated callers lacking the required role and log an audit event for denied administrative actions
5. The React shell hides administrator-only navigation items from non-administrators and shows an inline permission message instead of a broken action when a user lands on a restricted route
6. If a token becomes invalid or expires during navigation, React Query clears cached protected data and redirects the user to the login screen after receiving 401

## Technical Notes
- This story spans both frontend and backend layers in the local Docker-based stack: React + TypeScript on the client and FastAPI + Python on the server.
- It depends on the prior authentication story to provide valid short-lived JWTs; this story consumes those tokens for protected route access and session-aware UI behavior.
- Because the project has a frontend, validators should expect route guards, permission-based rendering, and client state updates driven by the authenticated user's role and permissions data.
- Backend authorization should use FastAPI dependency-based RBAC consistent with the epic, and error responses must match the project's structured auth contracts for 401 and 403 so the React app can respond predictably.
- Audit logging is relevant here specifically for denied administrative actions, with persistence aligned to the epic's SQLite-based audit visibility requirements.
- Since this is a local-first MVP without a public API or cloud deployment, the focus is on correct token handling, role enforcement, input/auth validation, and frontend/backend session alignment rather than external API consumers or distributed authorization concerns.

## Dependencies
- context-0001-0001 - Provides the email/password login flow and FastAPI-issued JWT access tokens required for authenticated role checks and session-aware frontend behavior.
