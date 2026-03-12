# Epic: Foundation Services

## Identity
- id: context-0001
- domain: infrastructure
- stories: 3

## Purpose
This epic establishes the shared security and session foundation for the local-first CRM and appointment management system. It enables operational users to sign in with local accounts, receive role-appropriate access in the React application and FastAPI backend, and preserves audit visibility for security-relevant events on a localhost Docker deployment used by a small team.

## Scope

### In Scope
- Implement JWT-based authentication with FastAPI token issuance for local account sign-in.
- Implement argon2 password hashing and email/password verification for local accounts.
- Implement role-based authorization for administrator, owner, manager, and staff roles in FastAPI dependencies.
- Implement optional administrator-only two-factor authentication when enabled by local configuration.
- Implement throttling for failed login attempts on localhost deployment.
- Implement audit logging for authentication events, authorization denials, and administrative security events stored in SQLite 3.
- Implement structured FastAPI error responses for 401, 403, 422, and 429 cases.
- Implement session-aware React UI behavior using React Query route guards, permission-based rendering, and inline/toast error patterns.
- Provide shared auth, permission, and error-handling behavior that other epics can consume.

### Out of Scope
- Customer record management, appointment scheduling flows, and WhatsApp conversation workflows beyond the shared foundation services needed to support them.
- Public API authentication patterns, third-party API consumer auth, or external developer access, because the project context indicates there is no public API.
- Mobile-specific authentication or session handling, because the project has no mobile application.
- SSO, OAuth providers, social login, or enterprise identity federation.
- Cloud production concerns such as multi-region deployment, autoscaling, managed secret stores, or uptime/SLA commitments, because this project is local-first and Docker-based.
- External audit pipeline integrations, SIEM forwarding, or enterprise security monitoring.
- Making Redis mandatory for MVP operation; optional local Redis support described in the epic must remain optional.
- CI/CD automation, release pipelines, or deployment orchestration beyond local Docker usage.

## Features
- jwt-authentication (short-lived JWT access tokens with FastAPI token issuance)
- argon2-password-hashing (email/password verification for local accounts)
- rbac-authorization (administrator, owner, manager, staff roles enforced in FastAPI dependencies)
- optional-admin-2fa (administrator-only second factor when enabled in local configuration)
- login-rate-limiting (throttled failed sign-in attempts on localhost deployment)
- audit-logging (authentication, authorization denial, and administrative security events stored in SQLite 3)
- structured-error-contracts (typed FastAPI responses for 401/403/422/429 cases)
- session-aware-react-ui (React Query route guards, permission-based rendering, inline and toast errors)

## Non-Functional Requirements
- Authentication and authorization must behave consistently across protected FastAPI routes so that requests without a valid JWT receive 401 responses and requests lacking sufficient role permissions receive 403 responses using the defined structured error contracts.
- Password handling must use argon2 hashing for local accounts and must never store plaintext passwords or reversible password representations in SQLite.
- JWT access tokens must be short-lived, as stated in the epic, to reduce exposure from token leakage in a local Docker environment.
- Login rate limiting must reliably throttle repeated failed sign-in attempts on localhost and return a typed 429 response that the frontend can interpret and display.
- Optional administrator 2FA must be controlled by local configuration and must apply only to administrator users when enabled.
- Audit logging must persist authentication events, authorization denials, and administrative security events in SQLite 3 with sufficient integrity for local audit visibility and troubleshooting.
- Structured error responses for 401, 403, 422, and 429 must be predictable enough for the React frontend to map them into route guards, inline validation feedback, and toast notifications without ambiguous parsing.
- Frontend session state must stay aligned with backend auth state so expired or invalid sessions are reflected promptly in navigation, guarded routes, and permission-based rendering.
- User-facing auth and permission feedback must be accessible and understandable in the React UI, consistent with the epic description's WCAG 2.1 AA-compliant feedback surfaces.
- Error handling should prioritize correctness, data integrity, and developer-visible diagnosis appropriate for a small-team local MVP, without assuming enterprise observability or cloud-scale resilience.
- The foundation services must continue to support MVP operation without requiring Redis, because the epic description states Redis is optional rather than mandatory.

## Dependencies

### Required
- (none)

### Optional
- (none)

## Success Criteria
- Valid local account credentials produce FastAPI-issued short-lived JWT access tokens, while invalid email/password attempts are rejected with structured auth errors.
- Protected backend routes enforce administrator, owner, manager, and staff permissions correctly, returning structured 401 or 403 responses for unauthorized access attempts.
- Repeated failed login attempts are throttled on localhost and surfaced to users through clear frontend inline or toast feedback based on the 429 error contract.
- Authentication events, authorization denials, and administrative security events are recorded in SQLite 3 and remain available for local audit review.
- The React application applies session-aware route guards and permission-based rendering so users only access UI areas allowed for their authenticated role, and expired or invalid sessions are handled clearly.

## Stories Overview
- context-0001-0001: Email and Password Login with JWT Access Tokens
- context-0001-0002: Role-Aware Navigation and API Authorization
- context-0001-0003: Audit Trail for Security and Administrative Events