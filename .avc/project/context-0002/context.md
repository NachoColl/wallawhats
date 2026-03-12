# Epic: Customer Records and Unified Context

## Identity
- id: context-0002
- domain: customer-management
- stories: 4

## Purpose
This epic delivers the core customer-management workspace for a local-first CRM and appointment management system used by small-to-medium business operational teams. It enables staff to create, search, update, and review customer profiles together with notes, communication consent metadata, and a unified summary of related messaging and appointments so they can resolve calls, front-desk interactions, and follow-up work quickly from one interface.

## Scope

### In Scope
- Implement customer-profile-crud (FastAPI + SQLite 3 create/view/update for contacts and notes).
- Implement customer-search (name and phone lookup with cursor-based pagination).
- Implement unified-customer-record (customer details, notes, conversation summary, appointment summary in one view).
- Implement contact-validation (phone and email format validation in FastAPI request models).
- Implement consent-record-linking (customer communication consent metadata stored with profile context).
- Implement react-query-customer-caching (cached profile and search results with targeted invalidation).
- Implement accessible-customer-forms (inline validation and keyboard-friendly field flows).
- Build FastAPI REST endpoints backed by SQLite 3 using SQLModel or SQLAlchemy 2.x for the customer domain.
- Support read-heavy local-first retrieval patterns suitable for live calls and front-desk usage.
- Surface messaging and appointment summaries inside the customer workspace through integration points with the Messaging and Appointment epics.

### Out of Scope
- Full messaging management, conversation composition, or WhatsApp transport behavior beyond showing conversation summary data from the Messaging epic.
- Full appointment creation, rescheduling, cancellation, or calendar workflow behavior beyond showing appointment summary data from the Appointment epic.
- Public API design or third-party external integrations, since the project context indicates hasPublicAPI: false.
- Cloud deployment, autoscaling, uptime SLAs, or enterprise observability requirements, since the project is docker-based, local-first, and non-cloud.
- Mobile-specific interfaces or mobile app behaviors, since hasMobileApp: false.
- CI/CD pipeline work, release automation, or production infrastructure hardening, since hasCI_CD: false.
- Advanced analytics, deduplication, bulk import/export, or customer segmentation features not listed in the epic features.
- Authentication or authorization model changes except basic security handling relevant to customer data access patterns already present in the application foundation.

## Features
- customer-profile-crud (FastAPI + SQLite 3 create/view/update for contacts and notes)
- customer-search (name and phone lookup with cursor-based pagination)
- unified-customer-record (customer details, notes, conversation summary, appointment summary in one view)
- contact-validation (phone and email format validation in FastAPI request models)
- consent-record-linking (customer communication consent metadata stored with profile context)
- react-query-customer-caching (cached profile and search results with targeted invalidation)
- accessible-customer-forms (inline validation and keyboard-friendly field flows)

## Non-Functional Requirements
- Customer create, read, update, and search flows must behave correctly in a local Docker deployment using FastAPI, SQLite 3, and the existing web frontend stack.
- Data integrity must be preserved for customer profiles, notes, and consent metadata so updates persist accurately and the unified customer record reflects committed database state.
- SQLite access should be configured for the read-heavy local-first workload described in the epic, including SQLite WAL mode where used by the project architecture, to keep lookups responsive during live calls and front-desk workflows.
- Cursor-based pagination for customer search must be stable and deterministic so users do not see duplicate or skipped records while paging through name or phone search results.
- API validation must reject malformed phone and email inputs in FastAPI request models with clear, developer-visible error responses that the frontend can surface inline.
- Frontend forms must provide inline validation and keyboard-friendly navigation, and customer record layouts should target WCAG 2.1 AA accessibility expectations described in the epic.
- React Query caching must avoid stale or misleading customer data by using targeted invalidation after create or update operations that affect profile or search results.
- Error handling must provide clear user-visible feedback for failed search, retrieval, create, and update actions, while preserving form input where practical so staff can recover quickly.
- Basic security must protect customer contact details, notes, and consent metadata from accidental overexposure through unnecessary fields, verbose error payloads, or unsecured internal endpoints.
- Integration of conversation and appointment summaries into the unified customer record must fail gracefully when related data is unavailable, returning the customer record without corrupting core customer data.
- Response times should be reasonable for a small-team local deployment, especially for customer lookup and record loading during active operational use; optimization should prioritize perceived responsiveness over enterprise-scale throughput.

## Dependencies

### Required
- context-0001 - This epic depends on the prior foundational context needed before customer-management functionality can be implemented and integrated into the application.

### Optional
- Messaging epic integration for conversation summary data used inside the unified customer record.
- Appointment epic integration for booking summary data used inside the unified customer record.

## Success Criteria
- Users can create a customer profile with contact details, notes, and consent metadata, and the saved data persists correctly in SQLite and can be retrieved afterward without loss or mismatch.
- Users can search customers by name or phone with cursor-based pagination, and paging through results remains stable without duplicate or missing entries caused by ordering inconsistencies.
- Users can update customer profiles and internal notes, and the unified customer record reflects the latest committed values after targeted frontend cache invalidation.
- The unified customer record displays customer details, notes, conversation summary, and appointment summary together in one accessible view, with graceful fallback behavior if summary data is unavailable.
- Invalid phone or email inputs are rejected by FastAPI validation and surfaced in the frontend through inline, keyboard-accessible error states.

## Stories Overview
- context-0002-0001: Search Customers by Name or Phone
- context-0002-0002: Create Customer Profile with Contact Details and Notes
- context-0002-0003: Update Customer Profile and Internal Notes
- context-0002-0004: View the Unified Customer Record
