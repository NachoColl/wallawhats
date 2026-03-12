# Questions & Answers

Sponsor-call questionnaire answers captured during project brief generation.

## Mission Statement

Help small-to-medium businesses manage customer relationships and appointments by keeping customer records and one-to-one WhatsApp conversations tied together in one place.

## Initial Scope

- Users can create, view, and update customer profiles with contact details and notes
- Users can send and receive one-to-one customer messages through a single business WhatsApp number
- Users can view each customer's full WhatsApp chat history alongside their customer record
- Users can schedule, reschedule, and cancel appointments for customers
- Users can view upcoming appointments in a calendar or list format
- Users can search for customers and quickly access their records and conversation history

## Target Users

Small-to-medium business staff who manage customer relationships: business owners/managers (decision-makers) and front-desk or sales/CS agents who need quick access to customer records, chat history, and appointment scheduling. Users typically have mixed technical proficiency (non-technical to moderately technical) and need a simple, responsive UI for searching customers, viewing one-to-one WhatsApp conversations, and booking/rescheduling appointments. Teams are small (single-user up to a few dozen concurrent users) and prioritize speed of lookup, clear audit trails of conversations, and reliable appointment reminders. Primary workflows center on customer lookup, message context during calls, and calendar-driven appointment management rather than advanced IT administration features.

## Deployment Target

MVP runs entirely on the developer or customer's local machine: a FastAPI (Python) or Express (Node) backend on localhost, a Vite + React frontend in dev mode, and a local SQLite database file (or SQLite in a lightweight container) with optional Docker Compose manifest for parity. This approach incurs zero cloud costs during MVP development and supports local webhook tunnels (ngrok) for WhatsApp integration and testing. The repo should include dev, staging (Docker Compose), and production-ready deployment notes — ready to migrate to Vercel + managed API, Railway, or Render when scaling to production.

## Technical Considerations

Frontend: Vite + React with TypeScript for fastest iteration, client-side search UI and calendar views; use React Query or SWR for caching and background refresh. Backend: FastAPI (async Python) with SQLModel/SQLAlchemy + Alembic or Express (TypeScript) with Prisma for schema-first SQLite access; choose the stack that matches team skills. Database: SQLite (local file) configured with WAL mode and tuned PRAGMA settings for a 60/40 read/write workload; be mindful of SQLite's single-writer constraints—use a connection-per-thread strategy or a lightweight write queue for concurrent writes and plan migration to PostgreSQL for heavier loads. Integration and realtime: implement webhook receivers (test locally via ngrok), WebSocket or server-sent events for UI updates, and an in-process queue or local Redis container for async tasks; use Docker Compose locally to improve production parity, enable easy addition of Redis/Postgres for later. Development practices: automated tests (unit + integration), E2E (Playwright), linting, and a simple CI that builds and runs tests; keep logs structured and use local log aggregation during debugging.

## Security & Compliance Requirements

Authentication with secure credential storage: email/password with bcrypt/argon2 hashing and short-lived JWTs or session cookies, optional 2FA for admins; implement RBAC for owners/managers vs staff to restrict appointment and messaging actions. Protect sensitive data: encrypt secrets (WhatsApp API tokens) in environment variables locally and recommend SQLCipher or disk-level encryption for SQLite if storing messages/PII on devices; all webhook endpoints validated with signatures and HTTPS tunnels (ngrok) during dev and TLS in production. Privacy and compliance: implement data export/deletion workflows, retention policy and consent tracking to meet GDPR/CCPA expectations for customer PII and message history, and maintain audit logs of message sends/appointment changes. Defensive measures: input validation/sanitization, rate limiting on APIs, secure headers (CSP/HSTS), dependency security scanning, and clear operational guidance to rotate credentials and move secrets to a real secrets manager when migrating out of the local MVP.
