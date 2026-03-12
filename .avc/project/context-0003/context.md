# Epic: WhatsApp Messaging and Conversation History

## Identity
- id: context-0003
- domain: messaging
- stories: 5

## Purpose
This epic enables operational users of the local-first CRM and appointment management system to handle one-to-one WhatsApp communication directly within customer records. It provides customer-linked conversation history, outbound messaging, inbound message capture, status visibility, and realtime updates so small-business teams can manage customer communication context and audit visibility without switching tools.

## Scope

### In Scope
- Implement outbound-whatsapp-messaging (FastAPI send endpoint to provider-compatible API).
- Implement inbound-webhook-processing (signature-validated WhatsApp callbacks normalized into local message records).
- Implement conversation-history-storage (SQLite 3 threads and message ordering linked to customers).
- Implement message-status-tracking (sent, delivered, read, failed provider event handling).
- Implement realtime-conversation-updates (WebSocket or server-sent events for inbound and outbound changes).
- Implement message-audit-events (send and status changes recorded in SQLite 3 audit log).
- Implement react-threaded-conversation-ui (message list, composer, loading and retry states).

### Out of Scope
- Group chats, broadcast messaging, campaigns, templates, or bulk outbound messaging are not included; this epic is limited to one-to-one WhatsApp messaging.
- Native mobile application messaging experiences are not included because the project context specifies no mobile app.
- Public API exposure for third-party consumers is not included because the project context specifies no public API.
- Cloud-scale availability targets, autoscaling, multi-region deployment, and enterprise-grade operational SLAs are not included for this docker-based local deployment.
- Messaging analytics, reporting dashboards, and business intelligence features beyond the specified audit log are not included.
- Non-WhatsApp communication channels are not included in this epic.
- Broader foundation capabilities and customer record management implemented by dependency epics are not reimplemented here.

## Features
- outbound-whatsapp-messaging (FastAPI send endpoint to provider-compatible API)
- inbound-webhook-processing (signature-validated WhatsApp callbacks normalized into local message records)
- conversation-history-storage (SQLite 3 threads and message ordering linked to customers)
- message-status-tracking (sent, delivered, read, failed provider event handling)
- realtime-conversation-updates (WebSocket or server-sent events for inbound and outbound changes)
- message-audit-events (send and status changes recorded in SQLite 3 audit log)
- react-threaded-conversation-ui (message list, composer, loading and retry states)

## Non-Functional Requirements
- Error handling: FastAPI send and webhook endpoints must reject invalid requests clearly, return developer-visible errors for provider failures and malformed payloads, and avoid leaving partial or ambiguous message state in storage. The React UI must show loading, retry, and failure states for conversation fetches and outbound sends.
- Data integrity: conversation threads, message records, status transitions, and audit log entries must persist correctly in SQLite and remain linked to the correct customer records supplied by the dependent customer records capability. Message ordering within a thread must remain stable across refreshes and realtime updates.
- Security: inbound webhook processing must validate provider signatures before accepting callbacks. Messaging history, send actions, and realtime streams must only be accessible to authenticated application users within the CRM.
- Retry safety: because the epic description includes retry-safe asynchronous work, webhook and provider status processing must tolerate duplicate callback delivery without creating incorrect duplicate state transitions or corrupting local records.
- Responsiveness: on the local docker deployment, conversation history loading, outbound send feedback, and realtime updates should be fast enough for interactive operational use and should minimize manual refresh.
- Accessibility: the React threaded conversation UI must support a responsive WCAG 2.1 AA-compliant experience, including readable message threads, usable composer controls, and visible loading/error/retry states.
- State consistency: persisted backend state, React Query data, and live event updates must converge so the UI reflects the same conversation and status data after sends, inbound messages, status changes, and page reloads.
- Auditability: message sends and provider status changes must be recorded in SQLite audit storage so teams can review messaging activity alongside the broader application audit visibility model.
- Local operability: the epic must run within the project's docker-based local environment using the declared stack and should not depend on cloud-only infrastructure.

## Dependencies

### Required
- context-0001 — Required because the epic description states that audit visibility for message sends and status changes integrates with Foundation.
- context-0002 — Required because conversation persistence and UI context are linked to customer records, and the description explicitly states customer-linked conversation persistence and synchronized customer record context.

### Optional
- (none)

## Success Criteria
- Authenticated users can open a customer record and view the persisted WhatsApp conversation history for that customer in the threaded conversation UI with stable message ordering.
- Sending an outbound WhatsApp message from the customer record creates the appropriate local record, calls the provider-compatible FastAPI send flow, and presents loading, success, and failure or retry states in the UI.
- Signed inbound webhook callbacks are validated, normalized, and persisted into the correct customer-linked conversation, while invalid signature callbacks are rejected and do not alter stored conversation data.
- Provider status events for sent, delivered, read, and failed are persisted, reflected in the conversation UI, and recorded in the SQLite audit log.
- Realtime conversation updates for inbound messages, outbound changes, and status transitions are delivered to authenticated users through WebSocket or server-sent events and keep the UI synchronized with persisted backend state.

## Stories Overview
- context-0003-0001: View Customer WhatsApp Conversation History
- context-0003-0002: Send an Outbound WhatsApp Message from the Customer Record
- context-0003-0003: Receive WhatsApp Inbound Messages via Signed Webhooks
- context-0003-0004a: Stream Realtime Conversation Events to Authenticated Application Users
- context-0003-0004b: Apply Live Conversation Updates and Provider Status Changes in the UI