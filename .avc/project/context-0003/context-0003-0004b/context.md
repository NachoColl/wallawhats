# Story: Apply Live Conversation Updates and Provider Status Changes in the UI

## Identity
- id: context-0003-0004b
- epic: context-0003 (WhatsApp Messaging and Conversation History)
- userType: staff, managers, owners, and administrators

## User Story
As staff, managers, owners, and administrators, I want the customer WhatsApp conversation view to apply live inbound and status-change events while keeping the current thread visible during connection interruptions so that I can monitor up-to-date message delivery state and continue working without losing customer context.

## Summary
This story implements the React conversation experience that consumes realtime messaging events and reflects provider-driven status transitions directly in the customer thread. It supports the epic goal of keeping WhatsApp communication history synchronized and visible inside customer records without requiring manual refresh during normal operation. The value to users is that they can see delivered, read, and failed updates inline, while connection problems degrade gracefully by preserving the current thread and prompting manual refetch when needed.

## Scope

### In Scope
- Apply realtime conversation events to the currently visible React threaded conversation UI.
- Update existing message entries in the UI when provider status changes such as delivered, read, and failed are received.
- Persist latest provider status changes in SQLite 3 for existing messages.
- Safely handle provider status callbacks that reference unknown providerMessageId values by logging and ignoring them.
- Prevent duplicate messages from appearing in the conversation view when live updates are applied.
- Show inline message status transitions in the conversation thread.
- Detect realtime connection interruption and show a non-blocking inline warning while keeping the current thread visible.
- Provide a manual refetch banner or similar recovery cue when the realtime connection drops.
- Throttle reconnect attempts after connection loss.
- Preserve the existing message list during reconnect and degraded realtime states.

### Out of Scope
- Establishing the realtime event transport itself; the server-side event streaming capability belongs to the dependency story context-0003-0004a.
- Initial conversation history retrieval and base threaded conversation rendering beyond what is necessary to apply live updates.
- Sending outbound WhatsApp messages and provider send flow handling.
- Inbound webhook signature validation and normalization logic.
- Group chat, broadcast, bulk messaging, templates, or non-WhatsApp channels.
- Admin or operator tools outside the authenticated CRM conversation experience for the specified user types.
- Advanced analytics, reporting, or audit log review interfaces.
- Cloud-scale connection management or public API behavior, which are not part of this local docker-based application.

## Acceptance Criteria
1. Provider status callbacks for delivered, read, and failed outcomes update the existing message row and persist the latest status in SQLite 3; unknown providerMessageId values are logged and ignored safely
2. The React conversation view applies live updates without duplicating messages, shows status transitions inline, and falls back to a manual refetch banner if the realtime connection drops
3. Connection loss shows a non-blocking inline warning rather than clearing the thread; reconnect attempts are throttled and the current message list remains visible

## Technical Notes
- This story spans both backend and frontend layers: provider status changes must update persisted message state in SQLite, and the React 18 + TypeScript UI must consume those changes and reconcile them into the visible thread.
- Because the project has a frontend and uses websocket in the stack, validators should expect client-side realtime subscription handling and UI state reconciliation rather than page refresh-based updates.
- The epic requires authenticated access to messaging history and realtime streams, so live conversation updates in the UI should only be available within authenticated CRM sessions.
- Data integrity is important here: message ordering and deduplication must remain stable when persisted backend state, realtime events, and any manual refetch behavior converge.
- In this docker-based local deployment, connection-loss handling should emphasize resilient local user experience, including visible warning state, throttled reconnect attempts, and preservation of already loaded thread data.
- Since the project does not expose a public API, technical validation should focus on internal application contracts between backend event/status processing and the React UI rather than third-party API consumer behavior.

## Dependencies
- context-0003-0004a — Required because this story consumes the realtime conversation event stream produced for authenticated application users.