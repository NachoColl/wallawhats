# Story: Use Appointment Read Endpoints in Synchronized List and Calendar Views

## Identity
- id: context-0004-0004b
- epic: context-0004 (Appointment Scheduling and Operational Views)
- userType: staff, managers, owners, and administrators

## User Story
As staff, managers, owners, and administrators, I want to review upcoming appointments in list and calendar views that share the same date-range and status filters so that I can switch between operational views without losing context and stay aware of appointment changes as they happen.

## Summary
This story implements the frontend appointments screen in React 18 + TypeScript that reads appointment data from the appointment list and calendar endpoints and presents it in two synchronized operational views. It supports users who need to inspect upcoming appointments either as a paginated list or across a calendar date range while keeping the same filter selections when switching views. Within the epic, this is the UI consumption layer for operational appointment visibility, adding loading states, empty states, and realtime updates for appointment changes.

## Scope

### In Scope
- Build the React appointments screen that consumes the list read endpoint and renders a paginated list view of appointments.
- Build the React appointments screen that consumes the calendar read endpoint and renders appointments for a selected date range.
- Allow the user to toggle between list and calendar views from the same appointments screen.
- Preserve the selected date range and status filters when switching between list and calendar views.
- Render pagination controls in list view using the backend response's nextCursor value.
- Show skeleton placeholders while list or calendar data is loading.
- Show a clear empty state when no appointments match the current list filters or calendar date range.
- Update visible list rows and calendar cells in response to realtime appointment.created, appointment.rescheduled, and appointment.cancelled events without requiring a full page refresh.
- Use reusable list and calendar presentation components for operational appointment review.

### Out of Scope
- Implementing the backend read endpoints themselves; that belongs to dependency story context-0004-0004a.
- Creating, rescheduling, or cancelling appointments from this screen.
- Validation rules for booking availability, conflict prevention, or cancellation confirmation workflows handled by other stories in the epic.
- Public-facing scheduling flows or any public API behavior.
- Outbound notifications such as SMS, email, or WhatsApp for appointment events.
- Recurring appointments, waitlists, resource optimization, or multi-location scheduling behavior.
- Mobile-specific appointment experiences.
- Separate admin/operator-only tools outside the shared appointments screen described in this story.

## Acceptance Criteria
1. The React appointments screen consumes GET /api/appointments?view=list&from=&to=&cursor=&limit=&status= to render the list view with pagination controls based on nextCursor
2. The React appointments screen consumes GET /api/appointments/calendar?from=&to= to render the calendar view for the selected date range
3. The React appointments screen lets the user toggle between list and calendar views while preserving the selected date range and status filters
4. Both views show skeleton placeholders during load and render a clear empty state when no appointments match
5. Realtime appointment.created, appointment.rescheduled, and appointment.cancelled events update visible list rows and calendar cells without requiring a full page refresh

## Technical Notes
- This story primarily affects the React + TypeScript frontend and depends on existing backend appointment read endpoints exposed by the FastAPI-based application.
- Because the project is a local docker-based web application for a small team, the implementation should favor straightforward client-side state handling for filters, view mode, loading states, and realtime refresh behavior.
- The list view must honor the backend REST contract for from, to, cursor, limit, and status query parameters and use nextCursor to drive deterministic pagination.
- The calendar view must request appointment data for the currently selected date range using the dedicated calendar endpoint and keep that range synchronized with the shared screen state.
- The project stack includes websocket support and the epic requires realtime appointment updates, so the UI needs to apply appointment.created, appointment.rescheduled, and appointment.cancelled events to the currently visible data without a full page reload.
- Since appointments are customer-linked elsewhere in the epic, the frontend should present data in a way that remains consistent with operational appointment review rather than introducing separate customer-management behavior in this story.

## Dependencies
- context-0004-0004a: Provides the appointment list and calendar read endpoints consumed by this story's frontend appointments screen.