# Epic: Appointment Scheduling and Operational Views

## Identity
- id: context-0004
- domain: appointments
- stories: 5

## Purpose
This epic adds customer-linked appointment scheduling workflows to the local-first CRM and appointment management system so operational users can create, reschedule, cancel, and review appointments in one place with customer context. It benefits small-to-medium business operational teams by making scheduling actions validated, persisted in SQLite 3, visible in operational list and calendar views, and traceable through audit logging without introducing enterprise-scale infrastructure.

## Scope

### In Scope
- Implement appointment-creation (customer-linked booking with availability validation).
- Implement appointment-rescheduling (date/time update rules with conflict prevention).
- Implement appointment-cancellation (state transition with confirmation and audit logging).
- Implement calendar-view (date-based operational appointment display).
- Implement list-view (upcoming appointments with filters and pagination).
- Implement realtime-appointment-updates (WebSocket or server-sent events for booking changes).
- Implement appointment-audit-events (create, reschedule, cancel actions stored in SQLite 3).
- Build FastAPI REST endpoints for appointment creation, rescheduling, cancellation, and operational read access backed by SQLite 3 persistence using SQLModel or SQLAlchemy 2.x.
- Build React 18 + TypeScript UI flows using React Query, accessible forms and confirmation dialogs, and reusable list and calendar components with skeleton loading, inline validation, and clear destructive-action feedback at WCAG 2.1 AA level.
- Integrate appointment workflows with Customer Records so appointments are linked to customers and can be scheduled from customer context.
- Integrate appointment changes with Foundation audit logging for traceable create, reschedule, and cancel actions.

### Out of Scope
- Public-facing booking flows, self-service customer scheduling, or any public API exposure.
- SMS, email, WhatsApp, or other outbound notifications for appointment events.
- Recurring appointments, waitlists, resource optimization, or multi-location scheduling behavior not described in this epic.
- Mobile-specific appointment experiences, since the project context has no mobile app.
- Cloud-production concerns such as autoscaling, SLA targets, multi-region deployment, or enterprise observability, because this project is a local docker-based MVP.
- Any customer record management features beyond the customer-linking dependency needed for appointment workflows.

## Features
- appointment-creation (customer-linked booking with availability validation)
- appointment-rescheduling (date/time update rules with conflict prevention)
- appointment-cancellation (state transition with confirmation and audit logging)
- calendar-view (date-based operational appointment display)
- list-view (upcoming appointments with filters and pagination)
- realtime-appointment-updates (WebSocket or server-sent events for booking changes)
- appointment-audit-events (create, reschedule, cancel actions stored in SQLite 3)

## Non-Functional Requirements
- Appointment create, reschedule, and cancel operations must enforce the described business-rule validation in the FastAPI backend so invalid availability selections, conflicting bookings, and invalid state transitions are rejected before persistence.
- Appointment records and related audit events must persist correctly in SQLite 3, with create, reschedule, and cancel actions stored reliably so operational history remains traceable.
- Error handling must provide clear API responses for validation and persistence failures, and the frontend must surface inline validation, confirmation messaging, and destructive-action feedback clearly to users.
- Operational read endpoints for list and calendar views must perform adequately for a small-team local docker deployment, with reasonable response times for upcoming appointments and date-based appointment retrieval.
- Realtime appointment updates must publish booking changes to connected consumers over the selected transport (WebSocket or server-sent events) so list and calendar views can stay synchronized with persisted changes.
- List and calendar UI components must support skeleton loading states while server data is pending and remain consistent with React Query server-state caching behavior.
- Forms, dialogs, list views, and calendar views must satisfy the WCAG 2.1 AA accessibility requirement stated in the epic description, including accessible confirmation flows and clear validation feedback.
- Filtering and pagination in upcoming appointment views must behave deterministically so operational users can reliably review appointment data across repeated reads.
- The implementation should remain maintainable for a small team by using reusable frontend components and straightforward REST contracts rather than introducing unnecessary infrastructure complexity.

## Dependencies

### Required
- context-0001: Required for Foundation audit logging, which this epic uses for traceable appointment changes.
- context-0002: Required because appointments are customer-linked and this epic schedules and manages appointments from customer records.

### Optional
- (none)

## Success Criteria
- Users can create an appointment from customer context, and attempts to book unavailable or conflicting time slots are rejected with clear validation feedback.
- Users can reschedule an existing appointment, and only valid date/time changes that satisfy conflict-prevention rules are persisted in SQLite 3.
- Users can cancel an appointment only through a confirmation flow, and cancellation updates the appointment state and stores an audit event.
- Operational list and calendar views both load appointment data from backend read endpoints, including upcoming appointments with filters and pagination and date-based operational display.
- After create, reschedule, and cancel actions are persisted, realtime updates are emitted and stored audit events exist for each action type.

## Stories Overview
- context-0004-0001: Schedule an Appointment from the Customer Record
- context-0004-0002: Reschedule an Existing Appointment
- context-0004-0003: Cancel an Appointment with Confirmation
- context-0004-0004a: Provide Operational Appointment List and Calendar Read Endpoints
- context-0004-0004b: Use Appointment Read Endpoints in Synchronized List and Calendar Views