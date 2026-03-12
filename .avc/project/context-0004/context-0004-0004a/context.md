# Story: Provide Operational Appointment List and Calendar Read Endpoints

## Identity
- id: context-0004-0004a
- epic: context-0004 (Appointment Scheduling and Operational Views)
- userType: staff, managers, owners, and administrators

## User Story
As a staff, managers, owners, and administrators, I want to retrieve upcoming appointments through authenticated list and calendar FastAPI endpoints with date filtering and pagination so that I can review operational schedules in the formats needed for day-to-day planning and follow-up.

## Summary
This story implements the backend read contracts that operational appointment views depend on. It enables authenticated operational roles to fetch appointment data from SQLite 3 in either a paginated upcoming-list format or a date-range calendar format, with validation for request parameters and predictable response shapes. Within the parent epic, this provides the read-side foundation that list and calendar UI stories can consume to display synchronized appointment schedules.

## Scope

### In Scope
- FastAPI read endpoint for operational appointment list retrieval.
- FastAPI read endpoint for operational appointment calendar retrieval.
- Authentication gating so only authenticated operational roles can access these endpoints.
- SQLite 3-backed retrieval of appointment records for operational read use cases.
- List response format containing appointment summary fields: id, customerId, customerName, startAt, endAt, status, title.
- Calendar response format containing appointment summary fields: id, customerId, customerName, startAt, endAt, status, title.
- Date-range filtering using from and to query parameters.
- Cursor-based pagination for the list endpoint.
- Stable ascending sorting for paginated list results using startAt ASC, id ASC.
- Default and maximum limit behavior for the list endpoint.
- Validation and defined error responses for invalid cursor values, malformed limit values, and missing or invalid ISO-8601 from/to parameters.

### Out of Scope
- Any frontend list or calendar UI implementation; this story only provides backend read endpoints.
- Appointment creation, rescheduling, or cancellation workflows, which are covered by other stories in the epic.
- Realtime update transport or synchronization behavior beyond providing read data.
- Public or unauthenticated access; the project has no public API exposure for this feature.
- Additional appointment fields, expanded customer details, or detailed appointment drill-down responses beyond the acceptance-criteria payload shape.
- Notification behavior, audit-event creation, or other side effects not required for read endpoints.
- Recurring appointments, waitlists, resource optimization, or multi-location scheduling behavior.
- Admin/operator analytics, reporting exports, or dashboard metrics not described in this story.

## Acceptance Criteria
1. GET /api/appointments?view=list&from=&to=&cursor=&limit=&status= (any authenticated role: administrator, owner, manager, staff) returns 200 { data: [{ id, customerId, customerName, startAt, endAt, status, title }], nextCursor: string|null }
2. List pagination uses stable sort startAt ASC, id ASC with an opaque cursor encoding (startAt, id); default limit=20, max limit=100; invalid cursor returns 422 { error: "INVALID_CURSOR" } and malformed limit returns 400 { error: "INVALID_PARAMETER", field: "limit" }
3. GET /api/appointments/calendar?from=&to= (any authenticated role) returns 200 { data: [{ id, customerId, customerName, startAt, endAt, status, title }] } for the requested date range
4. If from/to parameters are missing or not valid ISO-8601 datetimes, the relevant endpoint returns 422 { error: "VALIDATION_ERROR", fieldErrors: [...] }

## Technical Notes
- This story primarily touches the FastAPI backend and SQLite persistence layer, exposing REST read endpoints for operational appointment data rather than frontend behavior.
- Because the project is a local docker-based application with a React frontend, these endpoints should provide deterministic contracts that frontend list and calendar views can consume via normal server-state fetching patterns.
- Authentication is required for operational roles named in the acceptance criteria, so endpoint access control should align with existing session/auth mechanisms and reject unauthenticated access outside this story's specified success cases.
- The list endpoint requires cursor pagination with stable ordering on startAt ASC and id ASC, which is important to prevent duplicates or skipped rows across repeated reads in SQLite-backed queries.
- Input validation is part of the contract: from/to must be valid ISO-8601 datetimes, limit must be parsed safely, and cursor values must be validated before query execution.
- Appointment records are customer-linked within this epic, so the read model must include both appointment summary data and customer-identifying fields needed by operational views.

## Dependencies
- context-0004-0001: Provides appointment creation flow whose persisted records must become visible in list and calendar read endpoints.
- context-0004-0002: Provides appointment rescheduling flow whose updated date/time data must be reflected by these read endpoints.
- context-0004-0003: Provides appointment cancellation flow whose status changes must be reflected in operational list and calendar results.