# Story: View the Unified Customer Record

## Identity
- id: context-0002-0004
- epic: context-0002 (Customer Records and Unified Context)
- userType: staff, managers, owners, and administrators

## User Story
As a staff member, manager, owner, or administrator, I want to open a single customer workspace that shows profile details, internal notes, recent WhatsApp conversation context, and an appointment summary so that I can understand the customer's current situation and take action without switching between multiple screens.

## Summary
This story implements the read experience for a unified customer record in the web frontend and supporting backend retrieval for a single customer. It brings together core customer profile data with lightweight messaging and appointment summaries so operational users can handle calls, front-desk interactions, and follow-up work from one place. Within the parent epic, this is the consolidated customer view that builds on customer creation and search, using React Query to coordinate multiple reads while keeping the interface responsive in the project's local Docker deployment.

## Scope

### In Scope
- Implement viewing a single customer record by customer ID for authenticated internal roles.
- Return customer profile fields, internal notes, consent status, conversation summary, appointment summary, and timestamps from the customer record endpoint.
- Support 404 behavior when the customer does not exist and 401 behavior for unauthenticated access.
- Include lightweight conversation summary fields rather than full messaging history.
- Include lightweight appointment summary fields rather than full appointment details or workflow actions.
- Render a React customer record screen that shows profile information, notes, a conversation panel entry point, and appointment summary together in one layout.
- Show skeleton loading placeholders while the record view is loading.
- Show an inline retry state when record loading fails.
- Ensure keyboard reachability, visible focus states, and semantic section headings on the record page to align with WCAG 2.1 AA expectations.

### Out of Scope
- Creating, editing, or deleting customer profiles or notes; those belong to other customer CRUD stories.
- Customer search behavior, pagination, or lookup UX beyond arriving at a specific customer record.
- Full WhatsApp conversation management, message composition, or transport behavior.
- Full appointment workflows such as creation, rescheduling, cancellation, or calendar views.
- Returning full messaging payloads or full appointment payloads in this story's unified record response.
- Changes to authentication or authorization models beyond enforcing authenticated access and supported internal roles.
- Public API design or third-party external integrations, since the project does not expose a public API.
- Cloud-scale concerns such as distributed caching or autoscaling, which do not apply to this local-first docker deployment.

## Acceptance Criteria
1. GET /api/customers/:customerId (any authenticated role: administrator, owner, manager, staff) returns 200 { id, fullName, phone, email, notes, consentStatus, conversationSummary, appointmentSummary, createdAt, updatedAt }
2. If the customer does not exist, GET /api/customers/:customerId returns 404 { error: "CUSTOMER_NOT_FOUND" }; unauthenticated requests return 401 { error: "UNAUTHENTICATED" }
3. conversationSummary includes at minimum { latestMessageAt, unreadCount, lastMessagePreview } and appointmentSummary includes { nextAppointmentAt, activeAppointmentCount } without requiring the full messaging or appointment payloads
4. The React customer record screen renders profile, notes, conversation panel entry point, and appointment summary in a single layout with skeleton placeholders during load and an inline retry state on failure
5. All interactive controls on the record page are keyboard reachable with visible focus states and semantic section headings to satisfy WCAG 2.1 AA expectations

## Technical Notes
- This story touches both the React frontend and the FastAPI backend: the frontend renders the unified record screen, while the backend returns the customer record payload for a specific customer ID.
- Because the project uses React, TypeScript, and the epic explicitly calls for React Query caching, the record screen should use query-driven loading, failure, and retry states that avoid showing stale or misleading customer data.
- The backend is implemented in FastAPI with SQLite and SQLAlchemy/SQLModel patterns in this domain, so the record response must preserve committed database state for customer fields and notes in the local Docker environment.
- Since the system is local-first and read-heavy for live calls and front-desk use, retrieval should prioritize responsive record loading and graceful fallback for related summary data rather than requiring full messaging or appointment datasets.
- Authentication is required for internal roles, and basic security should avoid overexposing customer data beyond the fields defined in the acceptance criteria or returning verbose error payloads.
- The frontend must meet the epic's accessibility expectations by providing keyboard-friendly interaction, visible focus indication, semantic headings, and inline recoverable error handling on the record page.

## Dependencies
- context-0002-0001 - Customer search likely provides the user path to locate and open a specific customer record.
- context-0002-0002 - Existing customer creation provides the underlying customer data that this unified record view displays.