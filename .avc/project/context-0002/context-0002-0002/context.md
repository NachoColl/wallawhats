# Story: Create Customer Profile with Contact Details and Notes

## Identity
- id: context-0002-0002
- epic: context-0002 (Customer Records and Unified Context)
- userType: staff, managers, owners, and administrators

## User Story
As a staff, managers, owners, and administrators, I want to register a new customer from the operational UI with core contact information, optional consent status, and internal notes so that the customer record is saved immediately and I can continue work from the newly opened unified customer view.

## Summary
This story implements the customer creation flow in the web application for operational users who need to add a customer during day-to-day CRM work. It covers submitting a create form in the React UI, validating the request in FastAPI, storing the new customer profile in SQLite, and then opening the newly created unified customer record. Within the parent epic, this story establishes the initial customer record that other customer-management views operate on.

## Scope

### In Scope
- A React-based operational UI flow for creating a new customer record.
- Submission of customer data containing fullName, phone, optional email, optional notes, and optional consentStatus.
- FastAPI handling of POST /api/customers.
- Validation that fullName is required and limited to 100 characters.
- Validation that phone is required and matches E.164 format.
- Validation that email is optional but must be a valid email format when provided.
- Returning a 422 validation response with fieldErrors for invalid input.
- Detecting an existing customer with the same normalized phone and returning a 409 conflict response with conflictingCustomerId.
- Requiring authentication for the endpoint while allowing all authenticated MVP roles listed in the story to create customers.
- Persisting the customer profile in SQLite and returning the created record fields defined by the acceptance criteria.
- Writing a customer.created audit event after successful creation with actorUserId, customerId, and createdAt.
- Excluding notes text from the audit payload.
- Invalidating the customer search cache in the React app after a successful create.
- Navigating to the unified customer record after a successful 201 response.
- Mapping 422 field errors to inline form messages and re-enabling submit in the UI.

### Out of Scope
- Updating an existing customer profile or internal notes after creation.
- Customer search result behavior, pagination behavior, or search UI implementation beyond cache invalidation after create.
- Detailed implementation of the unified customer record view beyond navigating to it after successful creation.
- Messaging summaries, appointment summaries, message composition, or appointment workflow actions.
- Public API concerns or third-party integrations.
- Authentication or authorization model changes beyond enforcing the authenticated access and role behavior stated in this story.
- Bulk import, deduplication beyond normalized-phone conflict detection, or advanced customer matching rules.
- Additional audit reporting beyond the specified customer.created event.
- Error-handling behaviors other than the specified 401, 409, and 422 cases and the successful 201 flow.

## Acceptance Criteria
1. POST /api/customers (any authenticated role: administrator, owner, manager, staff) accepts { fullName, phone, email?, notes?, consentStatus? } and returns 201 { id, fullName, phone, email, notes, consentStatus, createdAt }
2. fullName is required with max length 100; phone is required and must match E.164 format; email is optional but if provided must be a valid email format; validation failures return 422 { error: "VALIDATION_ERROR", fieldErrors: [...] }
3. If a customer with the same normalized phone already exists, POST /api/customers returns 409 { error: "PHONE_ALREADY_EXISTS", conflictingCustomerId }
4. Unauthenticated requests return 401 { error: "UNAUTHENTICATED" }; all authenticated roles can create customer records in the MVP
5. On success, FastAPI writes an audit event customer.created with fields { actorUserId, customerId, createdAt } and excludes notes text from the audit payload
6. After a successful 201 response, the React app invalidates the customer search cache, navigates to the unified customer record, and on 422 maps each field error to inline form messages while re-enabling submit

## Technical Notes
- This story spans the React + TypeScript frontend and the FastAPI backend, with customer data persisted in SQLite in the project's local Docker deployment.
- Backend validation should be enforced in FastAPI request handling, matching the epic requirements for phone and email validation and returning structured validation errors the frontend can display inline.
- Because the application has a frontend, this story includes UI state handling for submit, inline validation feedback, and post-create navigation to the unified customer record.
- The endpoint is an internal application REST endpoint rather than a public API, so implementation should focus on the app's authenticated user flows and minimal error payload exposure.
- The normalized-phone uniqueness rule and the audit event requirement make data integrity and correct server-side processing important before the UI transitions to the newly created customer view.
- Notes are part of the customer payload for storage and display, but the acceptance criteria explicitly require that notes text not be copied into the audit event payload.

## Dependencies
- context-0001-0002