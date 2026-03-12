# Story: Email and Password Login with JWT Access Tokens

## Identity
- id: context-0001-0001
- epic: context-0001 (Foundation Services)
- userType: staff, managers, owners, and administrators

## User Story
As a staff, manager, owner, or administrator, I want to sign in through the React login screen using my email and password, and complete an administrator two-factor check when required, so that I can securely access the CRM and appointment workflows available to my role with a valid short-lived JWT session.

## Summary
This story implements the primary sign-in flow for operational users in the local-first CRM web application. It covers the React login interaction, FastAPI credential verification with argon2, JWT token issuance for authenticated API access, optional administrator-only two-factor handling, and lockout behavior for repeated failures. Within the Foundation Services epic, this story provides the entry point for all later role-aware navigation and protected backend access by establishing a valid authenticated session and clear frontend feedback for success and failure states.

## Scope

### In Scope
- Public login endpoint behavior for email and password authentication.
- Accepting login requests with email, password, and optional otpCode.
- Verifying stored passwords using argon2.
- Returning a short-lived JWT access token and authenticated user payload on successful login.
- Including JWT claims for user identity, role, issued-at time, and expiration time.
- Enforcing a 15-minute token lifetime and rejecting expired tokens with the defined 401 error.
- Supporting administrator-only two-factor authentication when enabled by local configuration.
- Returning the specified 401 errors when administrator OTP is required or invalid.
- Throttling repeated failed login attempts based on the same account or source IP within the specified time window.
- Returning the specified 429 lockout response including retryAfter.
- React login form behavior for in-flight submit disabling, inline validation handling for 422 responses, lockout messaging for 429 responses, and redirect after successful login.
- Redirecting authenticated users to a default screen appropriate to their role.

### Out of Scope
- Registration, password reset, password change, account invitation, or account provisioning flows.
- Authorization rules for protected features after login beyond token validity and role claim presence; broader role enforcement is handled by later authorization stories in the epic.
- Audit trail persistence and review interfaces, which are covered by the audit-focused story in this epic.
- Non-administrator two-factor flows, alternative MFA methods, backup codes, or recovery flows.
- Public API authentication patterns, third-party integrations, or mobile login behavior.
- Broader application screens for CRM records, appointments, or WhatsApp messaging beyond redirecting to the appropriate default destination after sign-in.
- Advanced cloud or distributed session concerns not relevant to the local Docker deployment.

## Acceptance Criteria
1. POST /api/auth/login (public) accepts { email, password, otpCode? } and returns 200 { accessToken, user: { id, name, email, role, requiresTwoFactor } } when credentials are valid and two-factor requirements are satisfied
2. The JWT access token contains claims { sub: userId, role: "administrator"|"owner"|"manager"|"staff", iat, exp } with a 15-minute lifetime; auth middleware rejects expired tokens with 401 { error: "TOKEN_EXPIRED" }
3. Passwords are verified using argon2; invalid credentials return 401 { error: "INVALID_CREDENTIALS" } without revealing whether the email exists
4. After 5 failed login attempts within 15 minutes for the same account or source IP, POST /api/auth/login returns 429 { error: "ACCOUNT_LOCKED", retryAfter: "<ISO-8601>" }
5. When administrator two-factor authentication is enabled and otpCode is missing or invalid, POST /api/auth/login returns 401 { error: "TWO_FACTOR_REQUIRED" } or 401 { error: "INVALID_OTP" } and no token is issued
6. The React login form shows inline field errors for 422 validation failures, a clear lockout message for 429, disables submit while the request is in flight, and redirects successful logins to the role-appropriate default screen

## Technical Notes
- This story spans both the React + TypeScript frontend and the FastAPI backend, with the frontend submitting credentials and managing session state while the backend performs verification and token issuance.
- The project is a local-first Docker deployment for a small team, so implementation should prioritize correct validation, predictable error contracts, and simple operational behavior rather than cloud-scale auth infrastructure.
- The backend must use argon2 for password verification and issue a REST-oriented JWT response contract that the React application can consume for authenticated requests.
- Since the epic requires structured 401, 422, and 429 responses that the frontend can interpret, backend error payloads for login and token expiry need to stay stable enough for inline validation, lockout messaging, and session handling.
- Because the project has a frontend but no public API, the contract is primarily internal between the React app and FastAPI services; UI state should reflect request-in-flight, success, validation failure, and lockout conditions clearly.
- Login throttling should work in the local MVP environment without assuming Redis is mandatory, since the epic explicitly states optional local Redis support must remain optional.

## Dependencies
- (none)