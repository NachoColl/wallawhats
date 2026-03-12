# Story: Schedule an Appointment from the Customer Record

## Identity
- id: context-0004-0001
- epic: context-0004 (Appointment Scheduling and Operational Views)
- userType: staff, managers, owners, and administrators

## User Story
As a staff, managers, owners, and administrators, I want to book an appointment directly from a customer's unified record with validated start and end times so that I can schedule customer work in context and have the appointment stored, visible, and traceable without leaving the customer record.

## Summary
This story implements the appointment-creation flow from within a customer's record in the CRM UI. It covers submitting scheduling details to the FastAPI backend, enforcing required-field and availability validation before persisting the appointment in SQLite 3, and updating the React interface so the new booking appears in customer and upcoming appointment views. Within the parent epic, this is the initial booking workflow that establishes customer-linked appointments and the audit trail needed for later operational review and follow-on actions like rescheduling or cancellation.

## Scope

### In Scope
- Allow authenticated operational roles (administrator, owner, manager, staff) to create an appointment from a specific customer record.
- Call the customer-linked appointment creation endpoint using the customer identifier in the route.
- Accept and validate scheduling payload fields: startAt, endAt, optional title, and optional notes.
- Enforce backend validation that startAt and endAt are ISO-8601 datetimes and that endAt is later than startAt.
- Reject booking attempts that violate the MVP appointment conflict rule and return the specified conflict response.
- Return the specified success response for a newly scheduled appointment, including persisted appointment fields and scheduled status.
- Return the specified error responses for missing customer and unauthenticated requests.
- Write the appointment.created audit event with the required metadata while excluding free-text notes from audit metadata.
- In the React app, invalidate the customer appointment summary and upcoming appointments queries after success.
- Show the newly created appointment in the UI after successful creation.
- Map 422 and 409 API errors to inline form feedback and re-enable form submission after those failures.

### Out of Scope
- Rescheduling or cancelling existing appointments; those belong to separate stories in this epic.
- Operational list-view and calendar-view implementation beyond the query invalidation and UI refresh needed to reflect a new booking.
- Realtime appointment broadcasting behavior after creation; the epic includes realtime updates, but this story's acceptance criteria focus on create persistence, audit logging, and frontend query invalidation.
- Public/self-service booking flows or any unauthenticated customer-facing scheduling.
- Notification sending by WhatsApp, SMS, or email.
- Recurring appointments, waitlists, multi-location scheduling, or advanced resource management.
- Customer record management beyond using an existing customer context and handling the case where the customer does not exist.
- Audit logging of free-text notes content, which is explicitly excluded.

## Acceptance Criteria
1. POST /api/customers/:customerId/appointments (any authenticated role: administrator, owner, manager, staff) accepts { startAt, endAt, title?, notes? } and returns 201 { id, customerId, startAt, endAt, status: "scheduled", title, notes, createdAt }
2. startAt and endAt are required ISO-8601 datetimes with endAt > startAt; invalid date order or missing fields returns 422 { error: "VALIDATION_ERROR", fieldErrors: [...] }
3. If the selected time conflicts with an existing active appointment according to the MVP availability rule, return 409 { error: "APPOINTMENT_CONFLICT", conflictingAppointmentId }
4. If :customerId does not exist, return 404 { error: "CUSTOMER_NOT_FOUND" }; unauthenticated requests return 401 { error: "UNAUTHENTICATED" }
5. On success, write audit event appointment.created with { actorUserId, customerId, appointmentId, startAt, endAt, occurredAt } and exclude free-text notes from audit metadata
6. After a successful 201 response, the React app invalidates the customer appointment summary and upcoming appointments queries, shows the new booking in the UI, and on 422/409 maps errors inline while re-enabling submit

## Technical Notes
- This story spans both frontend and backend layers in the project's React + TypeScript UI and FastAPI REST backend, with persistence in SQLite 3 for the appointment record.
- Because the project is a local docker-based MVP with no public API, the focus should be on straightforward authenticated server-side validation, clear REST error contracts, and reliable local persistence rather than cloud-scale concerns.
- The endpoint is customer-linked, so this story depends on existing customer-record capabilities to supply a valid customerId and render the scheduling interaction from customer context.
- Backend validation must enforce required datetime fields and conflict prevention before writing to SQLite, matching the epic requirement that invalid availability selections be rejected before persistence.
- The React frontend should integrate the create flow with existing server-state management behavior by invalidating affected queries and surfacing inline validation/conflict feedback accessibly in the form.
- Audit logging is part of the foundation context for this project and must record appointment.created metadata without including free-text notes, which has both privacy and traceability implications.

## Dependencies
- context-0002-0004: Required to schedule from an existing customer record and to provide the customer context referenced by :customerId