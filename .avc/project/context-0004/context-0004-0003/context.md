# Story: Cancel an Appointment with Confirmation

## Identity
- id: context-0004-0003
- epic: context-0004 (Appointment Scheduling and Operational Views)
- userType: staff, managers, owners, and administrators

## User Story
As a staff, manager, owner, or administrator, I want to cancel an existing appointment from either the customer record or an operational appointment view only after confirming the destructive action so that I can accurately stop a scheduled appointment, keep operational views up to date, and preserve an audit trail of the cancellation.

## Summary
This story implements the appointment-cancellation workflow for authenticated operational users working in either customer context or broader appointment views. It adds the backend state transition to cancelled, enforces the expected API responses for missing or already-cancelled appointments, and requires an explicit confirmation step in the React UI before the destructive action is submitted. Within the parent epic, this completes the cancellation portion of the appointment lifecycle and ensures the result is persisted, audited, and reflected in list, calendar, and customer-related views.

## Scope

### In Scope
- Allow cancellation of an existing appointment from the customer record UI.
- Allow cancellation of an existing appointment from an operational appointment view.
- Provide an authenticated FastAPI cancellation endpoint for appointment state transition to cancelled.
- Accept an optional cancellation reason in the request payload.
- Return the cancelled appointment response including id, cancelled status, cancelledAt timestamp, and reason when provided.
- Return defined error responses for appointment-not-found and already-cancelled cases.
- Enforce unauthenticated request rejection.
- Allow all authenticated MVP roles named in the story to perform cancellation.
- Create an audit event for successful cancellation with actor user ID, appointment ID, cancelledAt, and optional reason.
- Ensure audit payloads do not include unrelated customer PII beyond linked IDs.
- Require a confirmation dialog in the React UI before sending the cancellation request.
- Disable the confirm action while the cancellation request is pending.
- Invalidate appointment list, calendar, and customer summary caches after successful cancellation.
- Show the specified UI handling for 403, 404, and network/500 error outcomes.

### Out of Scope
- Appointment creation or rescheduling behavior.
- Public booking or unauthenticated cancellation flows.
- Restrictive role-based authorization beyond the MVP rule that all authenticated roles can cancel appointments.
- Notification delivery by SMS, email, WhatsApp, or other outbound channels after cancellation.
- Recurring appointment cancellation logic, waitlists, or other advanced scheduling features not described in this story.
- Admin reporting, analytics, or broader audit-history browsing beyond emitting the cancellation audit event.
- Realtime synchronization behavior itself unless covered elsewhere in the epic; this story specifically requires cache invalidation and audit emission for the cancellation flow.
- Detailed handling of every possible backend error code beyond the UI responses explicitly listed in the acceptance criteria.

## Acceptance Criteria
1. POST /api/appointments/:appointmentId/cancel (any authenticated role: administrator, owner, manager, staff) accepts { reason? } and returns 200 { id, status: "cancelled", cancelledAt, reason }
2. If the appointment does not exist, return 404 { error: "APPOINTMENT_NOT_FOUND" }; if it is already cancelled, return 409 { error: "APPOINTMENT_ALREADY_CANCELLED" }
3. Unauthenticated requests return 401 { error: "UNAUTHENTICATED" }; all authenticated roles can cancel appointments in the MVP
4. On success, emit audit event appointment.cancelled with { actorUserId, appointmentId, cancelledAt, reason? } and do not include unrelated customer PII beyond linked IDs
5. The React UI requires a confirmation dialog before submission, disables the confirm button while the request is pending, and after a successful 200 invalidates list, calendar, and customer summary caches
6. On 403 the UI shows an in-context permission message, on 404 removes stale appointment rows from the current view with a warning, and on network/500 displays a retryable toast

## Technical Notes
- This story spans both FastAPI backend and React + TypeScript frontend layers because it defines a REST cancellation endpoint and a confirmation-based UI flow in customer and operational views.
- The backend should persist the appointment state transition and cancellation metadata in SQLite, consistent with the epic requirement that appointment lifecycle actions remain traceable in local docker deployment.
- Because the project has a frontend but no public API, the contract is an internal authenticated REST endpoint used by the app UI; request validation and clear error responses are important for inline feedback and stale-row handling.
- Authentication is required, and MVP authorization is intentionally broad for this story: administrator, owner, manager, and staff can all cancel appointments, while unauthenticated requests must be rejected.
- The frontend should integrate with existing server-state management patterns implied by the epic, including pending-state button disabling and invalidation of list, calendar, and customer summary caches after success.
- Audit logging is a hard requirement from the epic dependency chain, and the emitted cancellation event must minimize exposed data by including linked IDs and avoiding unrelated customer PII.

## Dependencies
- context-0004-0001: Required because appointment creation establishes the appointment records later targeted by cancellation, and this story depends on the shared appointment workflow foundation within the epic.