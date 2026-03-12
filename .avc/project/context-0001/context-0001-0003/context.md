# Story: Audit Trail for Security and Administrative Events

## Identity
- id: context-0001-0003
- epic: context-0001 (Foundation Services)
- userType: administrators, owners, and managers

## User Story
As administrators, owners, and managers, I want to review a paginated security audit trail of sign-in, authorization denial, account administration, and configuration-sensitive actions so that I can see who performed each action, when it happened, and investigate security-relevant activity with clear local audit visibility.

## Summary
This story implements the audit logging and audit review flow for security-relevant events in the local-first CRM's shared foundation services. FastAPI must write structured audit records into SQLite 3 for authentication, authorization, and permission-sensitive administrative actions, and expose a protected audit endpoint that authorized operational leads can browse from the React application. It supports the parent epic's goal of shared security foundations by giving role-appropriate users traceable visibility into sensitive activity while preserving strict access control and safe handling of secret data.

## Scope

### In Scope
- Persisting SQLite-backed audit events for the specified security-relevant actions.
- Ensuring each audit record contains the required structured fields for traceability.
- Excluding sensitive secret material and credential artifacts from audit metadata and stored event payloads.
- Providing a protected FastAPI endpoint to list security audit events for administrators, owners, and managers.
- Implementing cursor-based pagination for the audit listing with the defined default and maximum page sizes.
- Enforcing stable descending ordering by occurredAt and id for paginated audit retrieval.
- Returning defined error responses for invalid or tampered cursors, malformed limit values, unauthenticated access, and forbidden staff access.
- Building a React audit view that fetches paginated audit results with React Query.
- Showing skeleton loading rows in the React audit view while data is loading.
- Showing a retryable inline error state in the React audit view when the audit request fails.

### Out of Scope
- Broader customer, appointment, or WhatsApp workflow audit reporting beyond the security and administrative events named in this story.
- Full audit analytics, filtering, search, export, or reporting features not stated in the acceptance criteria.
- Audit access for staff users or unauthenticated users.
- Creation of new authentication or role models; this story depends on prior auth and authorization foundations.
- UI behavior beyond the audit list loading and error states, such as advanced sorting controls or bulk actions.
- External audit sinks, SIEM integrations, cloud log pipelines, or non-SQLite persistence.
- Recording prohibited secret material such as password hashes, raw JWTs, OTP secrets, or integration secrets under any circumstance.

## Acceptance Criteria
1. Security-relevant actions emit SQLite-backed audit events including login.success, login.failed, authorization.denied, user.created, user.role_changed, user.deactivated, and config.secret_updated
2. Each audit event stores { id, eventType, actorUserId?, targetUserId?, occurredAt, entityType?, entityId?, metadata } and must not include password hashes, raw JWTs, OTP secrets, or integration secrets
3. GET /api/audit/security?cursor=&limit= (administrator, owner, manager) returns 200 { data: [...], nextCursor: string|null } with default limit=20 and max limit=100
4. Cursor pagination uses stable sort occurredAt DESC, id DESC; the cursor encodes the last row's (occurredAt, id) pair, invalid or tampered cursors return 422 { error: "INVALID_CURSOR" }, and malformed limit values return 400 { error: "INVALID_PARAMETER", field: "limit" }
5. Staff role calling GET /api/audit/security returns 403 { error: "FORBIDDEN" }; unauthenticated callers receive 401 { error: "UNAUTHENTICATED" }
6. The React audit view uses React Query to fetch paginated results, shows skeleton rows while loading, and surfaces a retryable inline error state when the request fails

## Technical Notes
- This story spans FastAPI backend, SQLite persistence, and React frontend layers because it requires server-side audit event writing plus a session-aware audit review UI.
- The endpoint must rely on the authentication and RBAC foundation from context-0001-0002 so only administrator, owner, and manager roles can retrieve audit data, while staff and unauthenticated callers receive the specified structured errors.
- Because the project is a local-first Docker MVP using SQLite, implementation should favor predictable local data integrity, straightforward pagination behavior, and secure storage over cloud-scale logging patterns.
- The React frontend uses React Query in this epic, so paginated audit retrieval, loading skeletons, retry behavior, and inline error presentation should align with the shared session-aware UI and structured backend error contracts.
- Input validation is important on query parameters even without a public API: cursor integrity must be checked and malformed limit values must return the specified typed error responses for frontend handling.
- Audit payload design must preserve traceability while avoiding sensitive data leakage, specifically excluding password hashes, raw JWTs, OTP secrets, and integration secrets from stored metadata.

## Dependencies
- context-0001-0002 — provides the authenticated session behavior, role-aware navigation, and backend authorization needed to protect the audit endpoint and surface role-appropriate access in the UI.
