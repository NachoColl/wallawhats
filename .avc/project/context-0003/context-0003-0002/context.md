# Story: Send an Outbound WhatsApp Message from the Customer Record

## Identity
- id: context-0003-0002
- epic: context-0003 (WhatsApp Messaging and Conversation History)
- userType: staff, managers, owners, and administrators

## User Story
As a staff, managers, owners, and administrators, I want to send a one-to-one outbound WhatsApp text message directly from a customer record so that I can contact the customer from the CRM, keep the conversation attached to the correct customer history, and immediately see the send result in the conversation thread.

## Summary
This story implements the outbound messaging action within the customer record so authenticated operational users can send a WhatsApp text without leaving the CRM. It covers request validation, provider send execution, local SQLite persistence, audit logging, and the UI behavior around sending and refresh. Within the parent epic, this is the core outbound path that complements stored conversation history and status visibility, giving teams a unified and auditable messaging workflow.

## Scope

### In Scope
- Add the authenticated outbound message send endpoint for a specific customer record.
- Accept a text payload for one-to-one outbound WhatsApp messaging.
- Validate that text is present after trimming and within the allowed 1-4096 character length.
- Return the specified success payload for a created outbound message, including sent status and provider message identifier.
- Check that the referenced customer exists before or as part of send processing and return the specified not-found error when it does not.
- Call the provider-compatible send flow and handle synchronous provider send failure with the specified 502 response.
- Persist the outbound message in SQLite 3 without creating duplicate local rows when the provider send fails synchronously.
- Allow all authenticated MVP roles listed in the story to send messages.
- Emit the message.sent audit event with the specified metadata fields and without message text.
- Update the React conversation experience after send by invalidating and refetching the conversation query.
- Disable the composer submit button while a send is in progress.
- Show inline validation feedback for 422 errors.
- Show an in-context permission banner for 403 responses.
- Show a retryable toast for 502 and network failures.

### Out of Scope
- Inbound webhook processing and signature validation for inbound WhatsApp messages.
- Realtime push delivery of outbound changes or status updates; this belongs to the realtime stories in the epic.
- Group chats, broadcast messaging, templates, campaigns, or bulk outbound messaging.
- Message status transitions beyond the initial successful send response covered here; later provider-driven delivered, read, or failed updates are handled by other epic work.
- Admin-only controls or role-specific restrictions beyond the MVP rule that all authenticated roles can send.
- Public API exposure for third parties.
- Non-text message types, attachments, media, or rich content.
- Full conversation history rendering itself, which depends on the prior story for viewing customer WhatsApp conversation history.

## Acceptance Criteria
1. POST /api/customers/:customerId/messages (any authenticated role: administrator, owner, manager, staff) accepts { text } and returns 201 { id, customerId, direction: "outbound", text, status: "sent", providerMessageId, sentAt }
2. text is required, trimmed, and must be 1-4096 characters; empty or oversized content returns 422 { error: "VALIDATION_ERROR", field: "text" }
3. If :customerId does not exist, return 404 { error: "CUSTOMER_NOT_FOUND" }; if provider send fails synchronously, return 502 { error: "PROVIDER_SEND_FAILED" } with no duplicate local message row created
4. Unauthenticated requests return 401 { error: "UNAUTHENTICATED" }; all authenticated roles can send messages in the MVP
5. On success, emit audit event message.sent with fields { actorUserId, customerId, messageId, providerMessageId, sentAt } and do not include message text in audit metadata
6. After a successful 201 response, the React conversation query invalidates and refetches; while sending, the composer submit button is disabled, 422 errors are shown inline, 403 displays an in-context permission banner, and 502/network failures show a retryable toast

## Technical Notes
- This story touches both frontend and backend layers in the local docker-based stack: React/TypeScript for the composer interaction and feedback states, FastAPI/Python for validation and send orchestration, and SQLite via SQLAlchemy for message persistence.
- Because the project has no public API and is an authenticated CRM workflow, the endpoint should be treated as an internal application API, but it still needs strict input validation and clear 401/404/422/502 response handling.
- The story depends on customer-linked conversation history, so outbound messages must be stored against the correct customer/thread model established by the conversation history capability, with stable data suitable for later thread reloads and status updates.
- Auditability is a stated epic requirement, so the send path must create the specified audit event while excluding message text from audit metadata to reduce unnecessary sensitive-content exposure.
- UI state management should align with the epic's state consistency requirement: after a successful send, React conversation data should be invalidated and refetched so persisted backend state and visible thread state converge.
- Since the deployment is local-first and docker-based rather than cloud-scale, technical concerns here center on request validation, auth/session enforcement, DB consistency, and user-visible retry/error handling rather than distributed scaling concerns.

## Dependencies
- context-0003-0001 — Required because outbound send success is expected to update and refetch the existing customer conversation view, which depends on the conversation history capability being in place.