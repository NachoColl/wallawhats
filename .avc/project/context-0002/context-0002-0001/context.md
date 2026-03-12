# Story: Search Customers by Name or Phone

## Identity
- id: context-0002-0001
- epic: context-0002 (Customer Records and Unified Context)
- userType: staff, managers, owners, and administrators

## User Story
As a staff, managers, owners, and administrators, I want to search for customers by entering a name or phone number in the search UI and page through matching results, so that I can quickly locate the correct customer and open that customer's unified record.

## Summary
This story implements the customer search interaction across the React frontend and FastAPI backend. Users enter a name or phone number into the search bar, receive paginated customer summary results from SQLite 3, and can use those results to open the unified customer record. Within the parent epic, this story provides the lookup workflow that supports access to customer profile context without exposing more than the summary fields needed in search results.

## Scope

### In Scope
- Implement a customer search UI for operational users.
- Allow users to type a customer name or phone number into the search bar.
- Debounce search input in the React frontend.
- Show a loading indicator while search requests are in progress.
- Render empty-state guidance when no customers match the query.
- Support keyboard navigation through the search results.
- Implement GET `/api/customers` with `q`, `cursor`, and `limit` query parameters.
- Search SQLite 3 customer data by `fullName` and normalized phone fields.
- Return the most recently updated customers when `q` is empty.
- Return paginated customer summary results containing only `id`, `fullName`, `phone`, `email`, `latestAppointmentAt`, and `latestMessageAt`.
- Use stable pagination ordering of `updatedAt DESC, id DESC`.
- Use an opaque base64 cursor encoding `(updatedAt, id)`.
- Apply default `limit=20` and maximum `limit=100`.
- Return the specified validation errors for invalid cursor and invalid limit values.
- Require authentication and allow authenticated administrator, owner, manager, and staff roles.
- Support opening a selected result into the unified customer record.

### Out of Scope
- Creating, updating, or deleting customer profiles, notes, or consent metadata.
- Displaying the full unified customer record itself beyond using search results to navigate to it.
- Appointment management or messaging workflows beyond returning `latestAppointmentAt` and `latestMessageAt` summary fields in search results.
- Search by fields other than `fullName` and normalized phone.
- Additional filters, sorting modes, fuzzy matching, ranking customization, bulk actions, deduplication, analytics, or segmentation features.
- Changes to the application's authentication or authorization model beyond honoring the listed authenticated roles and unauthenticated rejection behavior.
- Public API design or third-party integrations.
- Error scenarios not explicitly listed in the acceptance criteria when they belong to broader application handling outside this story.

## Acceptance Criteria
1. GET /api/customers?q=&cursor=&limit= (any authenticated role: administrator, owner, manager, staff) returns 200 { data: [{ id, fullName, phone, email, latestAppointmentAt, latestMessageAt }], nextCursor: string|null }
2. Search matches customer fullName and normalized phone fields in SQLite 3; empty q returns the most recently updated customers for quick access
3. Pagination uses stable sort updatedAt DESC, id DESC with an opaque base64 cursor encoding (updatedAt, id); default limit=20, max limit=100
4. Invalid cursor returns 422 { error: "INVALID_CURSOR" }; limit values that are non-integer, less than 1, or greater than 100 return 400 { error: "INVALID_PARAMETER", field: "limit" }
5. Unauthenticated requests return 401 { error: "UNAUTHENTICATED" }; all authenticated roles are permitted and no broader customer data than the summary fields is returned
6. The React search bar debounces input, shows a loading indicator while fetching, renders empty-state guidance when no records match, and allows keyboard navigation through results

## Technical Notes
- This story spans the React/TypeScript frontend and a FastAPI REST endpoint, with customer search results sourced from SQLite 3.
- The project is local-first and Docker-based, so implementation should favor responsive lookup behavior for operational users in a small-team environment.
- Backend pagination must be deterministic using `updatedAt DESC, id DESC` and cursor values derived from `(updatedAt, id)` to prevent duplicate or skipped records across pages.
- Input validation is required on query parameters, specifically for `cursor` and `limit`, with error responses matching the acceptance criteria so the frontend can handle them predictably.
- Because the project has a frontend and no public API, the endpoint contract is for internal application use and should return only the defined summary fields rather than broader customer data.
- Authentication is required, and authenticated internal roles listed in the story are permitted; unauthenticated requests must be rejected with the specified 401 error.

## Dependencies
- context-0001-0002