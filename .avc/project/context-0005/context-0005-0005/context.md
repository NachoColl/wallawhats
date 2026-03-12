# Story: Delete Customer Data for Privacy Requests

## Identity
- id: context-0005-0005
- epic: context-0005 (Administration and Compliance Operations)
- userType: administrators and owners

## User Story
As administrators and owners, I want to execute a confirmed customer deletion workflow for GDPR or CCPA privacy requests so that customer-related data is removed or anonymized according to the local MVP retention policy and the action is recorded with audit traceability.

## Summary
This story implements the protected admin workflow for deleting a customer's data in response to a privacy request. It adds a destructive-action flow in the React admin experience and a FastAPI-backed deletion endpoint that validates confirmation text, enforces role restrictions, performs deletion or anonymization of linked customer data, and records an audit event. Within the parent epic, this is the privacy deletion counterpart to customer data export and is intended to give authorized operational administrators a local, auditable way to fulfill deletion requests safely.

## Scope

### In Scope
- A protected deletion action for a specific customer privacy request, available only to authorized administrators and owners.
- Backend endpoint `POST /api/privacy/customers/:customerId/delete` that accepts `confirmationText` and returns a success payload with `customerId`, `status: "deleted"`, and `deletedAt`.
- Authorization enforcement so manager and staff roles receive forbidden responses for this deletion action.
- Authentication enforcement so unauthenticated requests receive the specified unauthenticated response.
- Server-side validation that `confirmationText` exactly matches the required server-provided phrase.
- Not-found handling when the target `customerId` does not exist.
- Deletion workflow behavior that removes or anonymizes customer profile, notes, consent-linked data, message linkage, and appointment linkage according to the local retention implementation for the MVP.
- Post-deletion behavior ensuring follow-up searches no longer return the deleted customer.
- Audit emission for successful deletion using the `customer.deleted` event with the specified payload fields and without raw message text or PII in the audit payload.
- React destructive-action confirmation dialog for the delete flow.
- Frontend query invalidation for customer search and open customer queries only after a successful 200 response.
- Frontend redirect away from the deleted customer's record after successful deletion.
- Inline UI handling for 403, 422, and 404 server responses.

### Out of Scope
- Customer data export behavior beyond the dependency on the export story.
- Legal policy definition, exact regulatory interpretation, or broader guarantees of GDPR/CCPA compliance beyond this operational deletion workflow.
- Any deletion behavior for non-customer entities or unrelated records not described in the acceptance criteria.
- Expansion of who can perform the action beyond administrators and owners.
- Public-facing, self-service, or customer-initiated deletion flows.
- Mobile-specific deletion experiences.
- Additional admin reporting, dashboards, or audit review screens beyond emitting the required audit event.
- Detailed retention schedule authoring or configurable policy management beyond using the local MVP retention implementation already referenced by the story.
- Handling of error responses other than the explicitly specified 401, 403, 404, and 422 states, unless required by shared platform behavior.
- Query invalidation before successful completion, bulk deletion workflows, or multi-customer deletion operations.

## Acceptance Criteria
1. POST /api/privacy/customers/:customerId/delete (administrator or owner role; manager and staff receive 403 { error: "FORBIDDEN" }) accepts { confirmationText } and returns 200 { customerId, status: "deleted", deletedAt }
2. confirmationText must exactly match a required server-provided phrase; mismatches return 422 { error: "INVALID_CONFIRMATION", field: "confirmationText" }
3. If :customerId does not exist, return 404 { error: "CUSTOMER_NOT_FOUND" }; unauthenticated requests return 401 { error: "UNAUTHENTICATED" }
4. The deletion workflow removes or anonymizes customer profile, notes, consent-linked data, message linkage, and appointment linkage according to the local retention implementation and ensures follow-up searches no longer return the deleted customer
5. On success, emit audit event customer.deleted with { actorId, customerId, deletedAt } and do not retain deleted raw message text or PII in the audit payload
6. The React delete flow requires a destructive-action confirmation dialog, invalidates customer search and open customer queries only after the server returns 200, redirects away from the deleted record, and shows inline handling for 403, 422, and 404 responses

## Technical Notes
- This story touches both the React frontend and FastAPI backend in the local Docker deployment, with REST-based request/response handling and protected admin UI behavior.
- Because the project has a frontend, validators should expect a destructive confirmation dialog, inline error rendering, redirect behavior, and careful client-side state invalidation for customer detail/search views after successful deletion.
- Backend implementation must enforce role-based authorization and authentication on the deletion endpoint; security-sensitive failures should be explicit and match the acceptance criteria and epic non-functional requirements.
- The deletion operation likely spans multiple customer-linked records in SQLite via SQLAlchemy-backed persistence, so the workflow should preserve integrity and avoid partial sensitive updates for profile, notes, consent-linked data, message linkage, and appointment linkage.
- Audit visibility is required by the epic, so successful deletions should create a locally reviewable audit record containing actor, action, and timestamp while minimizing sensitive data exposure by excluding raw message text and PII from the audit payload.
- This story depends on the broader privacy operations area established by the export story, so validators should consider consistency of customer identification and privacy workflow handling across export and deletion without assuming cloud infrastructure or public API concerns.

## Dependencies
- context-0005-0004 - Related privacy request workflow for customer data export; likely establishes adjacent admin privacy handling and customer-targeted operational patterns.