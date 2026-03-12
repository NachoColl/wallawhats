# Story: Receive WhatsApp Inbound Messages via Signed Webhooks

## Identity
- id: context-0003-0003
- epic: context-0003 (WhatsApp Messaging and Conversation History)
- userType: staff, managers, owners, and administrators

## User Story
As a staff, managers, owners, and administrators, I want inbound WhatsApp provider callbacks to be securely accepted, validated, normalized, and attached to the correct customer conversation so that new customer messages appear in the CRM with accurate customer context and realtime visibility without manual refresh or tool switching.

## Summary
This story implements the backend flow for receiving inbound WhatsApp messages from the provider through a locally exposed webhook endpoint, validating the callback signature, transforming the provider payload into the application's message model, and linking the message to the correct customer record. It supports the epic goal of keeping one-to-one WhatsApp conversation history complete and customer-linked inside the CRM. The story also ensures inbound messages are audit-visible and pushed to open conversation views through realtime updates, while keeping webhook handling responsive by delegating processing away from the immediate HTTP request path.

## Scope

### In Scope
- Expose a public provider callback endpoint at POST /api/webhooks/whatsapp for inbound WhatsApp webhooks.
- Validate the provider signature before any inbound message processing occurs.
- Reject invalid signatures with the specified 401 error response and ensure no message record is stored.
- Validate inbound payload shape and required provider identifiers.
- Reject malformed or incomplete payloads with the specified 422 error response.
- Normalize valid inbound provider payloads into the application's local message representation.
- Persist normalized inbound messages in SQLite 3.
- Link inbound messages to the correct customer conversation using customer phone number or existing conversation mapping.
- Return the specified 202 accepted response for valid inbound message payloads.
- Delegate webhook processing to an in-process queue or optional local Redis-backed worker so callback handling is retry-safe and does not block on realtime UI delivery.
- Emit the audit event message.received with the specified fields after successful processing.
- Exclude raw webhook secrets and unrelated provider credentials from audit event contents.
- Publish a WebSocket or server-sent event after inbound message persistence so open React conversation panels can update without full page reload.
- Support local development exposure via ngrok, consistent with the project's local docker environment.

### Out of Scope
- Outbound WhatsApp sending flows, provider send requests, and send retry UI.
- Conversation history browsing UI itself beyond the realtime event needed to update an already open panel.
- Provider delivery, read, sent, or failed status processing handled by message status tracking stories.
- Group chat, broadcast, campaign, template, or bulk messaging support.
- Public third-party API design beyond the provider-facing webhook callback required for WhatsApp integration.
- Cloud deployment concerns such as autoscaling or managed queue infrastructure.
- Reimplementation of customer record management or customer lookup foundations outside linking by phone number or conversation mapping.
- Admin/operator reporting, analytics, or dashboards related to inbound messaging volume.
- Broader error-state UX handling in the React application outside receiving the published realtime event.

## Acceptance Criteria
1. POST /api/webhooks/whatsapp (public provider callback) validates the provider signature before processing; invalid signature returns 401 { error: "INVALID_WEBHOOK_SIGNATURE" } and no message is stored
2. A valid inbound message payload is normalized into a SQLite 3 message record linked by customer phone number or conversation mapping and returns 202 { accepted: true, messageId, customerId }
3. If the payload is malformed or missing required provider identifiers, POST /api/webhooks/whatsapp returns 422 { error: "INVALID_WEBHOOK_PAYLOAD" }
4. Webhook processing is delegated to an in-process queue or optional local Redis-backed worker for retry-safe persistence and realtime publication so the HTTP callback does not block on UI fan-out
5. Successful processing emits audit event message.received with { customerId, messageId, providerMessageId, occurredAt } and excludes raw webhook secrets or unrelated provider credentials
6. When a new inbound message is persisted, the backend publishes a WebSocket or server-sent event so any open React conversation panel for that customer updates without full page reload

## Technical Notes
- This story primarily touches the FastAPI backend, SQLite persistence, audit logging, and the realtime event path used by the React frontend.
- Because the project is a local-first docker deployment and the description references ngrok, the webhook must work with locally exposed callback URLs rather than assuming cloud-hosted infrastructure.
- The project stack includes Redis and WebSocket support, so the acceptance criteria's in-process queue or optional local Redis-backed worker aligns with available local infrastructure for non-blocking webhook handling.
- Since hasFrontend is true, the backend event publication must produce a customer-scoped realtime update that the React conversation UI can consume to refresh an open thread without a full reload.
- Although the project has no public API for third-party consumers, this webhook is a provider-facing public callback and therefore needs strict signature validation and payload validation before persisting any data.
- Customer linkage and conversation history integrity depend on existing customer and conversation-view capabilities, so message ordering, correct customer association, and duplicate-safe processing should remain consistent with the epic's data integrity and retry-safety requirements.

## Dependencies
- context-0003-0001 — Required because inbound messages must attach to an existing customer conversation history view and data model established by viewing customer WhatsApp conversation history.