# Story: Stream Realtime Conversation Events to Authenticated Application Users

## Identity
- id: context-0003-0004a
- epic: context-0003 (WhatsApp Messaging and Conversation History)
- userType: staff, managers, owners, and administrators

## User Story
As a staff, managers, owners, and administrators user, I want an authenticated realtime channel that streams WhatsApp conversation message creation and status update events while I use the CRM so that the customer conversation view and related customer summary indicators stay current without manual refresh.

## Summary
This story implements the backend-facing realtime delivery mechanism that authenticated application users connect to in order to receive conversation event updates as messages are created or their delivery status changes. It supports the parent epic's goal of keeping customer-linked WhatsApp conversations synchronized with persisted backend state and visible in near real time inside the CRM. The value delivered is timely operational awareness for users working in customer records, while preserving authentication requirements and stable event structure for deterministic frontend updates.

## Scope

### In Scope
- Provide an authenticated realtime endpoint at GET /api/realtime/messages or WS /api/realtime/messages for application users.
- Allow any authenticated application role covered by this story's user type to connect to the realtime stream.
- Reject unauthenticated connection attempts with a 401 response body of { error: "UNAUTHENTICATED" }.
- Stream message.created events when newly persisted inbound or outbound messages become available.
- Stream message.status_updated events when later provider status transitions are persisted.
- Use a consistent event payload shape containing eventType, customerId, messageId, status?, sentAt, receivedAt?.
- Support frontend consumers updating the open conversation view and customer summary badges deterministically from the streamed payload.
- Ensure connection loss does not alter or invalidate existing persisted conversation data on the server.
- Allow clients that reconnect to resume receiving newly published events after re-establishing the realtime connection.

### Out of Scope
- Frontend UI logic that consumes the stream and applies updates to message lists or badges; that belongs to the corresponding UI story.
- Initial conversation history retrieval or message thread rendering; those are covered by conversation history and UI stories.
- Sending outbound WhatsApp messages or receiving inbound webhook payloads themselves; this story only streams events generated from those persisted actions.
- Backfilling missed events that occurred while a client was disconnected, beyond receiving newly published events after reconnection.
- Group chats, bulk messaging, non-WhatsApp channels, or public third-party realtime APIs.
- Mobile-specific realtime behavior, cloud-scale pub/sub architecture, or distributed multi-region delivery concerns, which are outside this local docker deployment.

## Acceptance Criteria
1. GET /api/realtime/messages or WS /api/realtime/messages (any authenticated role) streams events for message.created and message.status_updated scoped to authenticated application users; unauthenticated connection attempts receive 401 { error: "UNAUTHENTICATED" }
2. Event payloads include { eventType, customerId, messageId, status?, sentAt, receivedAt? } so the frontend can update the open conversation and customer summary badges deterministically
3. The realtime channel can publish message.created events for newly persisted inbound or outbound messages and message.status_updated events for later provider status transitions using the same payload shape
4. Connection loss does not invalidate existing conversation data on the server side and reconnecting clients can resume receiving newly published events after re-establishing GET /api/realtime/messages or WS /api/realtime/messages

## Technical Notes
- This story primarily touches backend realtime delivery in a local docker-based web application, likely bridging persisted messaging events from the FastAPI/SQLite messaging flows to a WebSocket or server-sent events channel.
- Because the project has a React frontend, the event contract must remain stable and deterministic so frontend state management can reconcile live updates with already-fetched conversation history and customer summary UI state.
- Authentication is required for access to messaging history and realtime streams per epic security requirements, so the realtime endpoint must enforce existing application auth/session rules for all listed application roles.
- The payload shape is intentionally minimal and should align with persisted message records and status transitions from inbound message processing and outbound/status tracking stories rather than introducing a separate divergent schema.
- Since the deployment is local-first and not cloud/public API oriented, technical concerns should focus on connection handling, auth validation, and consistency with stored message state rather than external consumer compatibility or internet-scale fanout.
- Reconnect behavior should preserve server-side data integrity: dropped client connections must not modify SQLite conversation records, and newly published events after reconnection should continue to reflect persisted backend changes accurately.

## Dependencies
- context-0003-0002 — Required because outbound message sends must exist and persist before this realtime channel can publish corresponding message.created events for outbound activity.
- context-0003-0003 — Required because inbound webhook processing must normalize and persist inbound messages before this realtime channel can publish corresponding message.created events for inbound activity.