# Story: Manage Messaging Integration Credential Metadata

## Identity
- id: context-0005-0003
- epic: context-0005 (Administration and Compliance Operations)
- userType: administrators

## User Story
As administrators, I want to view and update WhatsApp integration configuration metadata in the admin settings area, including phoneNumberId, webhook verify token alias references, and secret rotation intent, so that I can maintain local messaging integration settings safely without exposing raw secret values in everyday operational views.

## Summary
This story implements a protected admin settings workflow for reading and updating WhatsApp integration metadata in the local CRM. It supports the parent epic's goal of administrator-only operational control over sensitive configuration while preserving auditability and secret hygiene. The workflow gives administrators a clear way to confirm configuration status, update non-secret metadata, and trigger controlled secret-reference changes without leaking API tokens or signing secrets through the UI or API responses.

## Scope

### In Scope
- Administrator-only GET access to current WhatsApp integration metadata through the admin API endpoint.
- Administrator-only PATCH updates for WhatsApp integration metadata in the admin settings workflow.
- Editable request fields limited to phoneNumberId, webhookVerifyTokenAlias, and rotateSecret as defined in the story acceptance criteria.
- Response payloads that expose provider and configuration status fields while withholding raw API tokens and signing secrets.
- Explicit 401 handling for unauthenticated requests, 403 handling for authenticated non-admin users, and 422 validation handling for invalid field formats.
- Audit event emission for workflows that change credential references or rotateSecret state.
- React settings screen behavior that shows secret presence only as a boolean status and performs inline validation for editable metadata fields.
- In-context permission messaging on 403 within the settings screen.
- Query invalidation and refetch after a successful update so visible configuration status matches current server state.

### Out of Scope
- Returning, displaying, or editing raw WhatsApp API tokens, signing secrets, or other secret material.
- Configuration workflows for messaging providers other than WhatsApp.
- General messaging operations such as webhook delivery behavior, message sending, or integration validation beyond the status fields explicitly returned by the API.
- Customer-facing, staff-facing, or public configuration experiences outside the protected admin settings area.
- Audit log viewing screens or broader audit reporting beyond emitting the required event.
- Secret storage backend design, cloud secret-management services, or infrastructure changes not implied by the local controlled workflow.
- Additional admin settings features unrelated to WhatsApp integration credential metadata.

## Acceptance Criteria
1. GET /api/admin/integrations/whatsapp (administrator role only; all other authenticated roles receive 403 { error: "FORBIDDEN" }) returns 200 { providerName, phoneNumberId?, webhookConfigured, lastValidatedAt?, secretConfigured: boolean } without returning raw API tokens or signing secrets
2. PATCH /api/admin/integrations/whatsapp (administrator role only) accepts { phoneNumberId?, webhookVerifyTokenAlias?, rotateSecret?: boolean } and returns 200 { providerName, phoneNumberId, webhookConfigured, updatedAt, secretConfigured }
3. Invalid field formats return 422 { error: "VALIDATION_ERROR", fieldErrors: [...] }; unauthenticated requests return 401 { error: "UNAUTHENTICATED" }
4. Any workflow that changes credential references or rotateSecret state emits audit event config.secret_updated with { actorId, integration: "whatsapp", occurredAt, rotated: boolean } and must not include raw secret values in the audit record
5. The React settings screen masks secret presence as boolean status only, uses inline validation for editable metadata fields, and on 403 shows an in-context permission message rather than navigating away abruptly
6. After a successful 200 response, the integration settings query invalidates and refetches so the visible configuration status reflects the latest server state

## Technical Notes
- This story spans the protected React admin console and FastAPI backend, aligned with the project's React/TypeScript frontend and REST-based backend stack.
- The story explicitly states that non-secret metadata is persisted in SQLite 3, so backend writes for fields like phoneNumberId and related metadata should preserve local data integrity for this admin configuration record.
- Because the project is Docker-deployed and local-first, the workflow should rely on clear local request/response behavior and controlled local secret-reference handling rather than cloud-only secret infrastructure.
- The epic requires strong authorization and safe failure behavior for sensitive admin actions, so admin-only enforcement and explicit 401, 403, and 422 responses are part of the implementation contract for both API and UI behavior.
- Secret hygiene is central to this story: responses, UI rendering, and audit records must avoid exposing raw secret values while still indicating whether secret configuration exists.
- The epic references protected React screens with React Query and inline errors, so the frontend should refresh integration settings after successful mutation to avoid stale status in the admin interface.

## Dependencies
- context-0001-0003