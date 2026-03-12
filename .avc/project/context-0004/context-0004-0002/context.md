# Story: Reschedule an Existing Appointment

## Identity
- id: context-0004-0002
- epic: context-0004 (Appointment Scheduling and Operational Views)
- userType: staff, managers, owners, and administrators

## User Story
As a staff, managers, owners, and administrators, I want to move an existing scheduled appointment to a new date or time so that I can keep the customer appointment accurate without losing the customer link or the appointment’s traceable history.

## Summary
This story implements the rescheduling workflow for an existing appointment in the operational CRM, allowing authenticated operational users to update appointment timing while preserving the linked customer record and auditability. It fits within the appointment scheduling epic by covering the validated update path between initial booking and cancellation, with FastAPI enforcing appointment-state and availability rules before SQLite persistence. The user value is the ability to correct or adjust schedules reliably while keeping list, calendar, and appointment detail views synchronized after the change.

## Scope

### In Scope
- Add the authenticated appointment reschedule operation for existing appointments.
- Accept a new start and end datetime for an appointment reschedule request.
- Validate that the target appointment exists before attempting the update.
- Validate that only appointments currently in scheduled state can be rescheduled.
- Validate ISO-8601 datetime input and enforce that endAt is later than startAt.
- Check appointment availability and reject conflicting reschedule attempts.
- Persist the updated appointment time values in SQLite-backed storage through the FastAPI backend.
- Return the updated appointment payload on successful reschedule.
- Record an audit event for successful reschedules including previous and new datetime values and actor context.
- Support frontend handling after success and common error cases, including cache invalidation, inline field error display, permission feedback, and stale-record messaging.

### Out of Scope
- Creating new appointments; that belongs to the appointment creation story.
- Cancelling appointments or confirmation flows for cancellation.
- Rescheduling cancelled appointments beyond rejecting the attempt with the specified invalid-state response.
- Public-facing booking or any unauthenticated access.
- Notification delivery by WhatsApp, SMS, or email after rescheduling.
- Recurring appointment logic, waitlists, multi-location coordination, or resource optimization.
- Broad customer record management changes beyond preserving the existing customer link on the appointment.
- Detailed list and calendar UI implementation beyond the cache invalidation and error-handling behaviors specified here.
- Cloud-scale infrastructure concerns not relevant to this local docker-based application.

## Acceptance Criteria
1. PATCH /api/appointments/:appointmentId/reschedule (any authenticated role: administrator, owner, manager, staff) accepts { startAt, endAt } and returns 200 { id, customerId, startAt, endAt, status, updatedAt }
2. Only appointments in scheduled state can be rescheduled; attempting to reschedule a cancelled appointment returns 409 { error: "INVALID_APPOINTMENT_STATE" }
3. New startAt and endAt values must be valid ISO-8601 datetimes with endAt > startAt; validation failures return 422 { error: "VALIDATION_ERROR", fieldErrors: [...] }
4. Availability conflicts return 409 { error: "APPOINTMENT_CONFLICT", conflictingAppointmentId }; nonexistent appointmentId returns 404 { error: "APPOINTMENT_NOT_FOUND" }
5. On success, emit audit event appointment.rescheduled with { actorUserId, appointmentId, previousStartAt, previousEndAt, newStartAt, newEndAt, occurredAt }
6. After a successful 200 response, the React app invalidates appointment detail, calendar, and list caches; on 403 shows an inline permission banner, on 422 maps field errors inline, and on 404 shows a stale-record warning with navigation back to the current list

## Technical Notes
- This story spans both FastAPI backend logic and the React + TypeScript frontend because it defines a REST PATCH update flow plus client-side cache invalidation and inline error handling.
- Backend validation is central in this project and epic: appointment state checks, datetime validation, and conflict detection must be enforced before updating the SQLite 3 appointment row.
- Because the project is a local docker-based MVP with no public API, focus should remain on clear REST behavior, straightforward persistence, and authenticated internal-user access rather than external API hardening or cloud-scale concerns.
- The appointment must remain linked to its existing customer record during rescheduling, so the update should change timing fields without breaking customer association established by the appointment creation flow.
- Successful reschedules must integrate with the foundation audit logging dependency so the appointment.rescheduled event is stored with both previous and new time values for traceability.
- Since the frontend uses server-state synchronization patterns in this epic, React client behavior should invalidate appointment detail, list, and calendar data so operational views reflect the persisted change consistently.

## Dependencies
- context-0004-0001: Required because an appointment must already exist before it can be rescheduled, and this story updates the appointment created by the scheduling flow.