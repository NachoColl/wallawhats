# Project Brief

## 1. Overview

### Purpose

#### Core Purpose
The system helps small-to-medium businesses manage customer relationships and appointments by keeping customer records and one-to-one WhatsApp conversations tied together in one place.

#### Technology Stack Summary
The application uses a local-first stack with a Vite + React + TypeScript frontend, a FastAPI backend on Python 3.12, and SQLite as the primary local database.

#### Project Status
Implementation definition is in an MVP planning state with a default architecture selected for local deployment and development.

The product supports operational teams that need fast customer lookup, clear communication history, and reliable appointment handling without relying on complex enterprise tooling. The central experience combines customer profiles, WhatsApp chat context, and appointment scheduling in a single interface.

The system is optimized for small teams with mixed technical proficiency. It emphasizes responsive workflows, low operational overhead, and local deployment on localhost, with optional Docker Compose support for local service orchestration and testing parity.

The default backend choice is FastAPI because the technical considerations explicitly support async request handling, typed API contracts, and webhook processing in a local-first architecture. Express with TypeScript remains an alternative only when team TypeScript backend expertise is materially stronger than Python expertise.

## 2. Target Users

### Primary Users

#### Owners and Managers
Owners and managers oversee customer relationships, staff activity, and appointment operations. They need visibility into customer histories, message activity, appointment outcomes, and reminder-related operational follow-through so they can maintain service quality and business continuity.

These users require role-based access to operational actions, audit visibility for messages and appointment changes, and a simple interface that does not require technical administration knowledge. They often review customer context quickly during calls or walk-in interactions.

#### Front-Desk and Sales or Customer Service Staff
Front-desk and sales or customer service staff handle day-to-day customer communication and scheduling. They need fast search, immediate access to customer records, full WhatsApp conversation context, efficient appointment booking tools, and clear visibility into upcoming appointments that may need reminder follow-up.

These users have mixed technical proficiency, ranging from non-technical to moderately technical. The interface supports simple navigation, clear form behavior, responsive feedback, and low-friction access to the most common actions.

### Secondary Users

#### Business Administrators
Business administrators maintain user access, credential hygiene, and operational configuration for local deployments. Their needs focus on account management, role assignment, audit review, and secure handling of integration credentials.

These users require clear authentication controls, optional elevated protections such as two-factor authentication for admins, and access to operational logs without exposure to unnecessary technical complexity.

### Scope Clarifications

#### Appointment Reminders
Reliable appointment reminders are an operational expectation for target users. The current scope supports reminder readiness through appointment data, auditability, and async processing foundations, but does not include explicit outbound reminder delivery as a user-facing feature.

## 3. Core Features

### Customer Management

#### Customer Profiles
The system provides create, view, and update capabilities for customer profiles. Each profile stores contact details and internal notes that support service continuity across staff interactions.

#### Feature Status
Planned

#### Customer Search
The system provides fast customer search so staff can locate records quickly during calls, messages, or front-desk interactions. Search results lead directly to the unified customer record and conversation view.

#### Feature Status
Planned

### Messaging

#### One-to-One WhatsApp Messaging
The system supports sending and receiving one-to-one customer messages through a single business WhatsApp number. This keeps customer communication centralized and tied to the correct customer record.

#### Feature Status
Planned

#### Conversation History
The system displays each customer's full WhatsApp chat history alongside the customer profile. Staff can review previous exchanges before responding, which improves continuity and reduces context switching.

#### Feature Status
Planned

### Appointments

#### Appointment Scheduling
The system supports creating appointments for customers from the customer record. Scheduling includes date and time selection, availability validation, and confirmation of the booking.

#### Feature Status
Planned

#### Appointment Updates
The system supports rescheduling and cancellation of existing appointments. Changes are reflected in the customer record and the appointment views so staff can track the latest state of each booking.

#### Feature Status
Planned

#### Calendar and List Views
The system displays upcoming appointments in both calendar and list formats. This supports quick date-based planning and detailed operational review.

#### Feature Status
Planned

### Record Access and Context

#### Unified Customer Record
The system combines customer details, notes, conversation history, and appointment information in one place. This unified view reduces manual cross-referencing across separate tools.

#### Feature Status
Planned

#### Audit Visibility
The system records message sends and appointment changes in audit logs. This provides accountability and traceability for operational teams that prioritize clear records of customer interaction.

#### Feature Status
Planned

### Administration

#### User and Role Management
The system allows administrators to create user accounts, update user details, assign roles, and review permission-sensitive changes. This supports controlled access for owners, managers, staff, and administrators in a small-team environment.

#### Feature Status
Planned

#### Integration Credential Management
The system provides administrator-controlled handling of local configuration values and credential hygiene for messaging integration setup. This keeps operational secrets managed separately from day-to-day business workflows.

#### Feature Status
Planned

## 4. User Workflows

### Customer Lookup and Record Access

The primary entry point for most daily actions is customer search. Staff use this workflow to move from a name, phone number, or known contact detail into the full customer context.

1. The user enters a customer name, phone number, or other identifying detail into the search bar.
2. The system displays matching customer results with key summary details.
3. The user selects a matching record.
4. The customer profile opens with contact information, internal notes, appointment summary, and WhatsApp conversation history.
5. The user chooses a follow-up action such as reviewing notes, sending a message, or managing an appointment.

### Customer Profile Creation and Update

Customer profile management supports both new customer intake and correction of existing records. This workflow keeps contact data, notes, and customer context accurate.

1. The user searches for an existing customer or selects the create customer action.
2. The system displays a customer form with contact details and notes fields.
3. The user enters or updates the customer information.
4. The system validates required fields and field formats.
5. The user saves the profile.
6. The system stores the customer record and opens the unified customer view.
7. The updated contact details and notes appear immediately in the customer record.

### WhatsApp Conversation Handling

The messaging workflow ties customer communication directly to the customer record. This ensures staff have context before replying and preserves a complete communication trail.

1. The user opens a customer record.
2. The system displays the full one-to-one WhatsApp conversation thread for that customer.
3. The user reviews prior messages and relevant notes.
4. The user composes and sends a new message, or an inbound message arrives through the webhook receiver.
5. The conversation view refreshes through WebSocket or server-sent events so the latest message appears without a full page reload.
6. The system records the message event in the audit log.

### Appointment Scheduling

The scheduling workflow supports booking an appointment directly from the customer context. This keeps operational actions connected to the correct customer record.

1. The user opens a customer record.
2. The user selects the schedule appointment action.
3. The system displays available date and time inputs.
4. The user chooses a date and time.
5. The system validates availability and appointment rules.
6. The user confirms the booking.
7. The appointment appears in the customer record and in the calendar or list view.

### Appointment Rescheduling and Cancellation

Appointment changes follow the same customer-centered workflow as appointment creation. This reduces friction for staff handling frequent changes.

1. The user opens the customer record or appointment view.
2. The user selects an existing appointment.
3. The user chooses to reschedule or cancel.
4. The system validates the requested change.
5. The appointment record updates to reflect the new date and time or cancellation state.
6. The updated appointment appears in the calendar or list view.
7. The system records the change in the audit log.

### Administrative User and Credential Management

Administrative workflows support secure access management and local integration setup. These actions remain restricted to authorized administrative users.

1. The administrator signs in with email and password credentials.
2. The system applies administrator-level access controls and optional two-factor authentication when enabled.
3. The administrator opens the users or settings area.
4. The administrator creates a user account, updates an existing account, or changes a role assignment.
5. The administrator saves the changes.
6. The system records the action in the audit log.
7. The updated permissions take effect for subsequent user actions.
8. The administrator reviews credential-related configuration values through the local setup workflow without exposing secrets in general operational views.

### Authentication and Role-Based Access

The authentication workflow controls access to customer records, messaging actions, and appointment management based on user role. This supports secure operation for owners, managers, and staff.

1. The user enters email and password credentials.
2. The system validates credentials using secure password hashing.
3. The system resolves the user's role.
4. The dashboard and allowed actions render according to role-based permissions.
5. Unauthorized actions are blocked with clear feedback.
6. Optional two-factor authentication is enforced for admin users when enabled.

### Error Handling and Recovery

Operational reliability depends on clear handling of messaging failures, webhook issues, and local database contention. The user experience includes visible recovery paths instead of silent failures.

1. A user action or inbound event triggers an error such as failed message send, invalid webhook signature, or SQLite write contention.
2. The system records the failure in structured logs and, where applicable, audit records.
3. The interface displays a clear error state with actionable guidance.
4. The user retries the action if retry is safe and supported.
5. The system refreshes the affected view after a successful retry or preserves the error state until corrective action is taken.
6. Operational users review logs for unresolved failures during debugging or support.

## 5. UI/UX Design

### Frontend Technology

#### Framework
React 18 with TypeScript on Vite 5 provides a fast local development experience, strong typing, and a component-based UI model that fits search-heavy views, customer record screens, and appointment workflows. This stack supports quick iteration and maintainable frontend structure for a small team.

#### UI Library
The interface uses a custom design system approach built on reusable React components. The design system focuses on forms, searchable lists, conversation panels, appointment views, and role-aware action controls rather than heavy visual customization.

#### State Management
React Query manages server state, caching, and background refresh for customer records, appointments, and message threads. This approach keeps the UI responsive while reducing unnecessary refetching and simplifying asynchronous data handling.

#### Build Tool
Vite 5 handles local builds, module resolution, and fast hot module replacement. This supports quick feedback during UI development and efficient bundling for local deployment.

### User Interface Requirements

#### Design Approach
The design uses a simple business application layout with emphasis on readability, low-friction data entry, and clear operational actions. Shared UI patterns cover search inputs, profile panels, threaded conversations, calendar views, form validation states, and audit-related activity displays.

#### Responsive Strategy
The application uses a responsive layout that supports desktop-first daily operations while remaining usable on tablets and smaller screens. Important workflows such as customer lookup, record access, conversation review, and appointment editing remain accessible without requiring separate mobile-specific experiences.

#### Visual Design
The visual design prioritizes clarity over branding complexity. High-contrast typography, consistent spacing, clear section grouping, and distinct action states help mixed-skill users navigate quickly during customer-facing interactions.

### User Experience Considerations

#### Navigation
Navigation centers on fast access to search, customer records, appointments, and role-appropriate actions. The interface uses predictable top-level navigation with contextual actions inside customer and appointment views so users can move from lookup to action with minimal steps.

#### Accessibility
The system targets WCAG 2.1 Level AA accessibility. Interactive elements support keyboard navigation, visible focus states, semantic labels, and readable contrast ratios so operational staff can use the interface reliably across varied environments.

#### Internationalization
The content model supports future localization of interface text, date formats, and appointment-related messaging. The initial interface uses consistent terminology so labels and workflows remain easy to translate later without structural changes.

#### Performance
The UI prioritizes fast customer search, quick record loading, and efficient background refresh of message and appointment data. Client-side rendering and cached queries reduce unnecessary waiting, while realtime updates keep active conversations current without repeated manual refresh.

#### Form Behavior
Forms use clear field labels, inline validation, and explicit confirmation for destructive actions such as appointment cancellation. Required fields and validation errors are presented close to the relevant input to reduce confusion.

#### Loading States
The system uses visible loading indicators for search, record loading, appointment actions, and message activity. Skeleton states or lightweight placeholders preserve layout stability while data loads.

#### Error Handling
User-facing errors appear with concise explanations and safe retry options where appropriate. Messaging failures, invalid appointment actions, and authorization denials use distinct feedback patterns so users understand whether to retry, correct input, or request assistance.

## 6. Technical Architecture

### Deployment Environment

#### Platform
The application runs in a local development environment on localhost. This supports small-team usage, rapid iteration, and low infrastructure overhead.

#### Hosting Type
The frontend and backend run as local processes, with optional Docker Compose for local orchestration. No cloud hosting is required in the current deployment model.

#### Infrastructure
The local setup includes a React frontend, a FastAPI backend, a SQLite database stored as a local file, and optional local containers for Redis and related tooling when async processing or debugging needs increase. Webhook testing uses an HTTPS tunnel such as ngrok for inbound callback delivery during development.

### Technology Stack

#### Backend
The default backend uses Python 3.12 with FastAPI 0.115 and SQLModel or SQLAlchemy 2.x. This choice supports asynchronous request handling, typed API contracts, webhook processing, and clear separation between API, service, and persistence layers.

Express 5 on Node.js 20 LTS remains an alternative only when backend implementation needs to align with a strongly TypeScript-centered team. The default architecture assumes FastAPI unless that decision rule is triggered.

#### Database
SQLite 3 serves as the primary local database and uses WAL mode with tuned PRAGMA settings for a read-heavy operational workload. The data model stores customers, notes, conversations, messages, appointments, users, roles, consent records, and audit entries.

The architecture accounts for SQLite single-writer constraints by using a connection-per-thread strategy or a lightweight write queue. This protects reliability under small-team concurrent use.

#### Frontend
See Section 5 for the detailed frontend stack and interaction model.

#### Authentication
The default authentication approach uses email and password with argon2 hashing and short-lived JWT access tokens. Role-based access control governs messaging actions, appointment changes, user management, and administrative functions.

Secure session cookies remain an alternative only if local operational simplicity or browser-based session control proves more maintainable than token-based local authentication. The default architecture assumes JWT-based authentication.

### Architecture Patterns

The system follows a local-first web application architecture with a React client, a single FastAPI backend service, and a SQLite persistence layer. REST API endpoints handle customer, appointment, authentication, consent, and audit operations, while webhook endpoints process inbound WhatsApp events.

Realtime UI updates use WebSocket or server-sent events for message and appointment changes. Async processing uses an in-process queue or an optional local Redis container for background tasks such as webhook processing, retries, and reminder-ready job handling.

### Key Components

#### Customer Record Service
This component manages customer profile retrieval, updates, notes, and record linking across messages and appointments. It integrates with the database layer and provides the unified customer view consumed by the frontend.

#### Messaging Service
This component sends outbound WhatsApp messages, processes inbound webhook events, links messages to customer records, and publishes conversation updates to the UI. It integrates with the external WhatsApp service and the local audit logging mechanism.

#### Appointment Service
This component creates, updates, cancels, and retrieves appointments. It enforces availability rules, updates customer-linked appointment history, and feeds both calendar and list-based views.

#### Authentication and Authorization Service
This component handles login, password verification, token issuance, role resolution, and permission checks. It protects customer data and operational actions according to RBAC rules.

#### Administrative Configuration Service
This component manages user accounts, role assignments, and local integration configuration metadata. It supports administrator workflows while keeping permission-sensitive operations auditable.

#### Webhook Receiver
This component accepts inbound external callbacks, validates signatures, normalizes payloads, and routes events into message processing flows. It is exposed locally through ngrok during development.

#### Audit Logging Component
This component records message sends, appointment changes, authentication-relevant events, and administrative actions. It supports operational accountability and privacy-related traceability.

## 7. Integration Points

### External Services

#### WhatsApp Business API-Compatible Provider

##### Purpose
This integration enables one-to-one messaging through a single business WhatsApp number. It is the core channel that connects customer communication with internal customer records.

##### Data Exchanged
The system exchanges outbound message content, inbound message payloads, sender and recipient identifiers, message timestamps, delivery status events, read status events, message identifiers, and conversation linkage data tied to customer records.

Expected inbound event types include message received, message delivered, message read, and message status failure. Expected outbound operations include sending text messages and correlating provider message IDs to local customer conversation records.

##### Authentication
Authentication uses provider-issued API credentials stored securely in local environment variables. Webhook callbacks use signature validation to confirm authenticity.

#### ngrok

##### Purpose
ngrok exposes local webhook endpoints through an HTTPS tunnel so external messaging callbacks reach the local backend during development and testing.

##### Data Exchanged
The system exchanges inbound webhook requests and related callback traffic through the tunnel. No application business logic depends on ngrok beyond local callback delivery.

##### Authentication
Access depends on ngrok tunnel configuration and account controls, while the application still validates webhook signatures at the backend boundary.

### Internal Dependencies

#### Realtime Update Channel
The frontend depends on WebSocket or server-sent event channels for live message and appointment updates. This keeps customer conversation views and appointment displays synchronized with backend events.

#### Async Task Processing
The backend depends on an in-process queue or an optional local Redis container for deferred processing such as retries, webhook handling, and reminder-ready jobs. This reduces blocking during user-facing operations.

## 8. Security & Compliance

### Security Measures

#### Authentication & Authorization
The system authenticates users with email and password credentials protected by argon2 hashing. It issues short-lived JWTs and supports optional two-factor authentication for administrative users.

Role-based access control distinguishes owners and managers from staff and administrators. Permissions restrict messaging actions, appointment modifications, user management, and configuration functions according to role.

#### Data Protection
Sensitive credentials such as WhatsApp API tokens are stored in local environment variables with operational guidance for secure handling on developer or office-managed machines. Customer data, message history, and appointment data are protected for webhook traffic through HTTPS tunnels such as ngrok during local integration testing.

For local data at rest, the system uses SQLCipher or disk-level encryption when devices store customer messages or personally identifiable information. Input validation and sanitization protect against malformed or unsafe user input, and consent records support privacy-sensitive handling of customer communication data.

#### Infrastructure Security
API endpoints use rate limiting, secure headers such as CSP and HSTS where applicable to the local web application context, and dependency security scanning. Webhook endpoints require signature validation before processing inbound data.

Structured logging supports monitoring and debugging without relying on unstructured console output. Local operational guidance includes credential rotation, restricted access to environment files, and device-level protection for machines that store customer data.

### Compliance Requirements

#### GDPR
The system supports customer data export and deletion workflows, consent tracking, retention policy enforcement, and audit records for data-affecting actions. Customer records and message history are managed in a way that supports traceable privacy operations.

#### CCPA
The system supports access and deletion requests for customer personal data and maintains clear records of operational actions related to customer information. Auditability and retention controls help document handling of customer communication and appointment history.

#### Audit Logging
Audit logs capture message sends, appointment changes, authentication events, user management actions, and relevant administrative changes. These records support accountability, operational review, and compliance-related investigation.

### Local Security Constraints

#### Deployment Alignment
The current security model is local-first and localhost-oriented. It assumes local processes, local configuration files, local SQLite storage, and ngrok-based HTTPS tunneling only for webhook development and testing.

#### Future Migration Considerations
If the deployment target changes from localhost to a hosted environment, secret management, transport security, and infrastructure controls require a separate architecture review. These hosted-environment controls are outside the current deployment definition.

## 9. Success Criteria

### Key Outcomes

#### Operational Effectiveness
The system enables staff to find a customer record quickly, review WhatsApp history in context, and complete appointment actions without switching tools. Search and record access remain fast enough to support live customer interactions.

#### Messaging Reliability
Inbound and outbound WhatsApp conversations remain correctly linked to the associated customer record. Message history appears in the unified record with accurate ordering and clear traceability.

#### Appointment Accuracy
Users can create, reschedule, and cancel appointments successfully, and those changes appear consistently in both customer records and appointment views. Calendar and list displays reflect the current appointment state.

### Technical Goals

#### Application Quality
The codebase includes automated unit, integration, and end-to-end tests, along with linting and build validation. Structured logs support debugging of messaging, webhook, and appointment flows.

#### Local Reliability
The local deployment model runs consistently on localhost with SQLite configured for WAL mode and tuned for mixed read and write activity. The architecture handles small-team concurrency without frequent write failures or data inconsistency.

#### Realtime Behavior
Conversation and appointment views update through WebSocket or server-sent events with minimal manual refresh. Users see inbound messages and appointment changes promptly during active workflows.

### User Experience Goals

#### Usability
Users with mixed technical proficiency can search customers, review message context, and manage appointments through a simple and responsive interface. Common tasks require few steps and present clear feedback.

#### Accessibility
The interface meets WCAG 2.1 Level AA expectations for keyboard access, focus visibility, labeling, and readable contrast. Users can complete essential workflows without relying on pointer-only interactions.

#### Error Recovery
Errors in message sending, appointment validation, webhook processing, or authorization are visible and understandable. Users receive clear recovery guidance and safe retry paths where applicable.

### Definition of Done

#### Checklist
- [ ] Customer search works on local SQLite data with fast record lookup.
- [ ] Users can create, view, and update customer profiles with contact details and notes.
- [ ] WhatsApp messages send and receive through a WhatsApp Business API-compatible provider.
- [ ] Inbound and outbound WhatsApp messages link to the correct customer records.
- [ ] Full customer conversation history appears alongside the unified customer record.
- [ ] Users can schedule, reschedule, and cancel appointments from the customer context.
- [ ] Upcoming appointments appear in both calendar and list views.
- [ ] Role-based access control restricts messaging, appointment, and administrative actions correctly.
- [ ] Administrative users can create accounts, assign roles, and review audit-relevant changes.
- [ ] Webhook signature validation protects inbound messaging events.
- [ ] Audit logs capture message events, appointment changes, authentication events, and administrative actions.
- [ ] Realtime updates refresh conversation and appointment views without full page reloads.
- [ ] Unit, integration, and Playwright end-to-end tests pass for core customer, messaging, and appointment workflows.
- [ ] The application runs locally on localhost with documented setup and optional Docker Compose support.