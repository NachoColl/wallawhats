# Story: Export Customer Data for Privacy Requests

## Identity
- id: context-0005-0004
- epic: context-0005 (Administration and Compliance Operations)
- userType: administrators and owners

## User Story
As a administrators and owners, I want to request and monitor a structured export of a specific customer's stored profile, internal notes, consent records, conversation history metadata, appointment history, and customer-scoped audit metadata so that I can fulfill privacy access requests using the system's stored customer data without exposing secrets or unrelated records.

## Summary
This story implements a protected privacy export workflow in the admin experience for authorized business users. It allows administrators and owners to queue a customer-specific export job, check that job's status, and access the export when it is ready. Within the parent epic's compliance operations scope, this delivers an auditable, role-restricted way to respond to customer access requests using data composed from the local application's SQLite-backed records.

## Scope

### In Scope
- A protected backend endpoint to request export generation for a specific customer.
- Role enforcement allowing only administrator and owner users to request exports.
- Explicit 401, 403, and 404 handling for unauthenticated requests, forbidden roles, and unknown customers during export request creation.
- Returning an asynchronous export job response with exportJobId, customerId, and queued status.
- Composition of a structured customer export from SQLite 3 domain tables through FastAPI.
- Inclusion in the export of customer profile fields, internal notes, consent records, conversation history metadata, appointment history, and relevant audit entries scoped to the customer.
- Explicit exclusion from the export of password hashes, user session tokens, integration secrets, and unrelated tenant data.
- A protected backend endpoint to retrieve export job status by exportJobId.
- Returning export job status information and an optional downloadUrl when the export is complete.
- 404 handling for unknown export jobs.
- Emitting the specified audit event for successful export generation requests.
- React privacy workflow support for queued, ready, and failed export states.
- Inline permission feedback in the UI for 403 responses.
- Retry action support in the UI for failed export jobs.

### Out of Scope
- Customer-facing self-service privacy request submission, tracking, or download experiences.
- Any privacy deletion workflow, retention execution, or consent editing behavior covered by other stories in the epic.
- Role management changes or broader authorization model updates beyond enforcing administrator or owner access and forbidding manager and staff access for this workflow.
- Exact export file schema, formatting conventions, or archival/lifecycle behavior beyond producing a downloadable structured payload.
- Bulk exports, tenant-wide exports, or exporting data for multiple customers in one action.
- Exposure or management of password hashes, session tokens, or integration secrets.
- Admin features unrelated to privacy export, such as user account management or integration credential metadata administration.
- Cloud-only background processing or storage services not aligned with the local Docker-based application stack.

## Acceptance Criteria
1. POST /api/privacy/customers/:customerId/export (administrator or owner role; manager and staff receive 403 { error: "FORBIDDEN" }) returns 202 { exportJobId, customerId, status: "queued" }
2. If :customerId does not exist, return 404 { error: "CUSTOMER_NOT_FOUND" }; unauthenticated requests return 401 { error: "UNAUTHENTICATED" }
3. The export payload includes customer profile fields, internal notes, consent records, conversation history metadata, appointment history, and relevant audit entries scoped to that customer, excluding password hashes, user session tokens, integration secrets, and unrelated tenant data
4. GET /api/privacy/exports/:exportJobId (administrator or owner role) returns 200 { exportJobId, status, downloadUrl? } when complete or 404 { error: "EXPORT_NOT_FOUND" } for unknown jobs
5. Successful export generation emits audit event customer.export_requested with { actorId, customerId, exportJobId, occurredAt }
6. The React privacy workflow shows queued, ready, and failed states; unauthorized 403 responses show an inline permission banner and failed jobs present a retry action

## Technical Notes
- This story touches both the React frontend and FastAPI backend: the admin privacy UI must initiate export requests and present job state, while the backend composes customer-scoped export content from SQLite-backed records.
- Because the project is a local-first Docker deployment with no cloud infrastructure, export job handling and downloadable payload delivery should fit within local application services and persistence/runtime patterns already available in the stack.
- The project has a frontend but no public API, so validators should expect protected internal admin REST endpoints and UI state handling rather than external developer-facing API concerns.
- Authorization and explicit failure handling are required by the epic: unauthenticated requests must return 401, forbidden manager/staff attempts must return 403, and unknown customer or export job identifiers must return the specified 404 errors without silent failure.
- Data minimization is central to this story: the export must be limited to the requested customer's allowed data categories and must exclude password hashes, session tokens, integration secrets, and unrelated tenant data.
- The epic requires auditable handling of sensitive actions, so the export request must create the specified locally reviewable audit event with actor, customer, export job, and timestamp information.

## Dependencies
- context-0002-0004
- context-0003-0001
- context-0004-0004