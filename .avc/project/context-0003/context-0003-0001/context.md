# Story: View Customer WhatsApp Conversation History

## Identity
- id: context-0003-0001
- epic: context-0003 (WhatsApp Messaging and Conversation History)
- userType: staff, managers, owners, and administrators

## User Story
As a staff, managers, owners, and administrators, I want to open a selected customer record and review that customer's full one-to-one WhatsApp conversation history in an ordered conversation panel so that I can understand prior communication context before replying and make better scheduling or customer-service decisions.

## Summary
This story implements the read path for customer-linked WhatsApp conversations inside the CRM customer record view. It covers an authenticated API for fetching paginated message history from SQLite through FastAPI and a React conversation panel that renders the thread with direction and status visibility. Within the parent epic, this is the foundational history-view experience that lets operational users see message context and audit-relevant status details before sending new messages or reacting to inbound updates.

## Scope

### In Scope
- Authenticated retrieval of WhatsApp message history for a specific customer via `GET /api/customers/:customerId/messages?cursor=&limit=`.
- Returning paginated message records with the documented fields: `id`, `direction`, `text`, `providerMessageId?`, `status`, `sentAt`, `receivedAt?`, and `actorUserId?`.
- Stable reverse-chronological ordering using `sentAt DESC, id DESC`.
- Cursor-based pagination where the cursor encodes `(sentAt, id)`.
- Default and maximum page size handling for `limit`.
- Validation and error responses for invalid cursor values and malformed limit parameters.
- Error handling for missing customers and unauthenticated access.
- Rendering the React conversation panel for the selected customer.
- Displaying message direction and delivery/read/failure status in the UI.
- Preserving scroll position while older pages are loaded into the thread.
- Showing an empty-state prompt when no conversation exists for the customer.
- Showing skeleton placeholders during loading and an inline retry action for failed loads without collapsing the surrounding customer record layout.

### Out of Scope
- Sending outbound WhatsApp messages, composer behavior, and send retry flows; those belong to separate messaging stories in this epic.
- Receiving inbound WhatsApp webhooks, signature validation, and provider callback normalization.
- Realtime push delivery of new messages or status changes over WebSocket or server-sent events.
- Audit log writing details beyond what is already implied by stored message history; this story is about viewing conversation history, not implementing audit event generation.
- Group chats, broadcast messaging, templates, bulk messaging, or non-WhatsApp channels.
- Public third-party API exposure, since the project has no public API.
- Admin/operator-only tooling outside the authenticated CRM customer record experience for the listed application roles.
- Broader customer record creation or management behavior outside resolving an existing `customerId` for conversation lookup.

## Acceptance Criteria
1. GET /api/customers/:customerId/messages?cursor=&limit= (any authenticated role: administrator, owner, manager, staff) returns 200 { data: [{ id, direction, text, providerMessageId?, status, sentAt, receivedAt?, actorUserId? }], nextCursor: string|null }
2. Messages are sorted by sentAt DESC, id DESC for pagination stability; the cursor encodes (sentAt, id), default limit=50, max limit=100
3. Invalid cursor returns 422 { error: "INVALID_CURSOR" }; malformed limit values return 400 { error: "INVALID_PARAMETER", field: "limit" }
4. If :customerId does not exist, the API returns 404 { error: "CUSTOMER_NOT_FOUND" }; unauthenticated requests return 401 { error: "UNAUTHENTICATED" }
5. The React conversation panel shows message direction, delivery/read/failure status, preserves scroll position while older pages load, and renders an empty-state prompt when no conversation exists
6. Loading states use skeleton message placeholders and failed loads show an inline retry action without collapsing the surrounding customer record layout

## Technical Notes
- This story spans both backend and frontend layers in the project's React + TypeScript frontend and FastAPI + SQLite backend, with the API reading customer-linked message records and the UI rendering them within the customer record interface.
- Because the application is local-first and docker-deployed rather than cloud-distributed, the key technical concerns are correct local persistence, pagination stability, responsive fetch behavior, and consistent UI state rather than cloud scaling concerns.
- Authentication is required for message history access, and the acceptance criteria explicitly allow authenticated application roles `administrator`, `owner`, `manager`, and `staff`; unauthenticated requests must be rejected with the specified 401 error shape.
- Conversation records are customer-linked and depend on existing customer data from a separate capability, so backend lookup must validate the customer exists and preserve correct association between customer records and message history.
- The epic requires stable message ordering and state consistency, so the API's `sentAt DESC, id DESC` ordering and encoded cursor must be implemented in a way that avoids duplicate or missing messages across paginated loads.
- Since the project has a frontend but no public API, the API contract here is internal to the application but still needs strict parameter validation and predictable error responses so the React conversation panel can show loading, empty, and retry states reliably.

## Dependencies
- context-0002-0004 — Required because this story reads conversation history for a selected customer record and depends on customer record context being available for `customerId` resolution and UI integration.