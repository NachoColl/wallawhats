# Story: Update Customer Profile and Internal Notes

## Identity
- id: context-0002-0003
- epic: context-0002 (Customer Records and Unified Context)
- userType: staff, managers, owners, and administrators

## User Story
As a staff, managers, owners, and administrators, I want to update an existing customer’s contact details, consent status, and internal notes directly from the customer record so that I can correct inaccurate information, preserve current internal context, and see the unified customer view reflect the saved changes immediately without a full page reload.

## Summary
This story implements editing of an existing customer record from the customer workspace, covering profile fields, consent status, and internal notes. It supports the epic’s goal of keeping customer information current and usable in one interface by persisting updates through FastAPI to SQLite and returning the updated record. The frontend behavior in this story ensures saved changes appear immediately and relevant cached customer data is refreshed without a full page reload.

## Scope

### In Scope
- Support updating an existing customer record through a PATCH endpoint.
- Allow partial updates for fullName, phone, email, notes, and consentStatus.
- Permit authenticated users in the roles administrator, owner, manager, and staff to perform the update.
- Return the updated customer data on successful save, including updatedAt.
- Apply the same validation rules used by customer creation for fullName, phone, and email.
- Return 404 when the specified customer does not exist.
- Return 401 for unauthenticated requests.
- Return 422 with fieldErrors for invalid submitted fields.
- Return 409 when a phone number is changed to one already used by another customer, including the conflictingCustomerId.
- Persist profile and notes changes through FastAPI into SQLite 3.
- Record a successful update audit event named customer.updated with actorUserId, customerId, changedFields, and updatedAt.
- Exclude note body diffs from audit metadata.
- After a successful update, invalidate the specific customer query and search result cache in the React app.
- Show updated values immediately in the UI after a successful response.
- Display in-context frontend error messaging for 403, 422, and 409 responses, with field-level mapping where applicable.

### Out of Scope
- Creating a new customer profile.
- Customer search implementation, pagination behavior, or search-result rendering beyond invalidating affected search caches after an update.
- Viewing the full unified customer record layout beyond keeping it current after edits.
- Messaging summary or appointment summary functionality beyond preserving consistency of the customer view after profile changes.
- Audit log browsing, reporting, or administrative audit tools.
- Authentication or authorization model changes beyond enforcing the response behavior and authenticated-role access described in the acceptance criteria.
- Delete, merge, deduplication, bulk edit, import, or export workflows.
- Public API design or external third-party integrations.

## Acceptance Criteria
1. PATCH /api/customers/:customerId (any authenticated role: administrator, owner, manager, staff) accepts { fullName?, phone?, email?, notes?, consentStatus? } and returns 200 { id, fullName, phone, email, notes, consentStatus, updatedAt }
2. If :customerId does not exist, the API returns 404 { error: "CUSTOMER_NOT_FOUND" }; unauthenticated requests return 401 { error: "UNAUTHENTICATED" }
3. Field validation matches create rules: fullName max 100, phone E.164, email valid format; invalid fields return 422 { error: "VALIDATION_ERROR", fieldErrors: [...] }
4. Changing phone to a value already used by another customer returns 409 { error: "PHONE_ALREADY_EXISTS", conflictingCustomerId }
5. On success, FastAPI writes audit event customer.updated with { actorUserId, customerId, changedFields, updatedAt } and excludes note body diffs from audit metadata
6. After a successful 200 response, the React app invalidates the specific customer query and search result cache, shows updated values immediately, and on 403/422/409 displays in-context error messaging with field-level mapping where applicable

## Technical Notes
- This story spans the React frontend and FastAPI backend: the UI updates customer data from the customer record view, and the backend persists the change and returns the updated record.
- Persistence is to SQLite in a local Docker deployment, so update behavior should reliably reflect committed database state in the returned payload and subsequent record views.
- Validation must remain consistent with the create-customer flow in this epic, especially for fullName length, E.164 phone format, and email format, so frontend inline errors can map to backend validation responses.
- Because the story explicitly requires cache invalidation after a successful update, frontend state management must refresh both the specific customer record data and affected search results to avoid stale values in the interface.
- The story introduces an audit requirement on successful updates; audit metadata must include actor and changed field tracking while avoiding exposure of note body diffs.
- Customer contact details and internal notes are sensitive operational data, so internal endpoint error responses should stay limited to the defined contracts and not expose unnecessary record details.

## Dependencies
- context-0002-0002 - Existing customer creation is a prerequisite because this story updates previously created customer records and reuses the create-flow validation rules.