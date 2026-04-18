<!-- Template Meta
     Template-ID:   TPL-SVC
     Version:       1.1.0
     Compliance:    95%+
-->

# Notification Service — Domain Specification

> **Conceptual Stack Layer:** Domain / Service
> **Space:** Platform
> **Owner:** Platform Shared-Business Team

> **Meta Information**
> - **Version:** 2026-04-18
> - **Template:** `domain-service-spec.md` v1.1.0
> - **Template Compliance:** 95%+
> - **Status:** DRAFT
> - **Service ID:** `shared-ntf-svc`
> - **Suite:** `shared`
> - **Domain:** `ntf`
> - **Base Package:** `io.openleap.shared.ntf`
> - **API Path:** `/api/shared/ntf/v1`
> - **Port:** 8416
> - **DB Schema:** `shared_notification`
> - **Tier:** T2 — Shared Enterprise Business
> - **Supersedes:** `crm-ntf-svc` (see §13 Migration)

---

## 0. Document Purpose & Scope

This document specifies the **Notification Service** (`shared-ntf-svc`) as a T2 Shared Enterprise Business capability. It defines the domain model, business rules, use cases, REST API, events, data model, security, and quality attributes for the `Notifications & Alerts` bounded context.

**Audience:** Backend developers, frontend developers, QA engineers, architects, product owners across every suite.

**In scope:** All domain logic within the `Notifications & Alerts` bounded context, consumed by any T3 suite (CRM, SD, FI, PS, SRV, OPS, TKS, HR, …).
**Out of scope:** UI implementation (see feature specs), infrastructure provisioning, suite-specific notification-authoring UIs.

**Related documents:**
- Suite Spec: `T2_SharedBusiness/_t2_suite.md`
- OpenAPI: `contracts/http/shared/ntf/openapi.yaml`
- Event Schemas: `contracts/events/shared/ntf/*.schema.json`
- Origin (deprecated): `T3_Domains/CRM/domain-specs/crm_ntf-spec.md`

---

## 1. Business Context

### 1.1 Purpose & Responsibility

Centralized notification hub for the **entire platform**. Consumes domain events from any suite, evaluates recipient notification preferences, and delivers through multiple channels: in-app, push, email, SMS, Slack, Microsoft Teams, and webhook.

**Owns:**
- Notification (delivered notification with read/unread state)
- NotificationPreference (per-principal channel preferences per event type)
- NotificationChannel (channel configuration — SMTP, Twilio, Slack webhook, Teams webhook)
- NotificationTemplate (per-event-type template with i18n variants — new in T2 to serve multi-suite consumers)

### 1.2 Business Value

Provides a single, reliable notification fabric across every business capability in the platform. Users configure their preferences once; every suite respects them. Critical events (SLA breach, assignment, compliance alerts) have guaranteed delivery paths.

---

## 2. Service Identity

| Field | Value |
|-------|-------|
| Service ID | `shared-ntf-svc` |
| Suite | `shared` |
| Domain | `ntf` |
| Bounded Context | Notifications & Alerts |
| Base Package | `io.openleap.shared.ntf` |
| API Base Path | `/api/shared/ntf/v1` |
| Port | 8416 |
| Database Schema | `shared_notification` |
| Tier | T2 — Shared Enterprise Business |
| Repository | `openleap-io/io.openleap.shared.ntf` |
| Team | `team-platform-shared` |

---

## 3. Domain Model

### 3.1 Aggregates

**Notification** (AggregateRoot)
- Delivered notification envelope: recipientPrincipalId, tenantId, eventType, channelId, title, body, payload (JSONB), readAt, deliveryStatus (`PENDING`|`DELIVERED`|`FAILED`|`READ`), retryCount, sourceEvent routingKey.

**NotificationPreference** (AggregateRoot)
- Per-principal preferences keyed by `{tenantId, principalId, eventType}`: enabled channels, digest mode (`IMMEDIATE`|`HOURLY`|`DAILY`|`OFF`), quiet hours (start/end), severity cutoff.

**NotificationChannel** (AggregateRoot)
- Channel configuration per tenant: type (`IN_APP`|`EMAIL`|`PUSH`|`SMS`|`SLACK`|`TEAMS`|`WEBHOOK`), credentials ref (→ iam tenant secrets), status, rate limits.

**NotificationTemplate** (AggregateRoot)
- Template: eventType, channelType, locale, subject, bodyTemplate (Handlebars-like syntax), required payload fields. One template per (eventType, channelType, locale); falls back to default locale.

---

## 4. Business Rules

### 4.1 Rule Catalog

| ID | Rule | Type |
|----|------|------|
| BR-NTF-001 | Users MAY opt out of non-critical notifications per event type | POLICY |
| BR-NTF-002 | Critical notifications (severity = CRITICAL) MUST NOT be disabled by preference | CONSTRAINT |
| BR-NTF-003 | In-app notifications retained for 90 days then archived | POLICY |
| BR-NTF-004 | Delivery retry: up to 3 attempts with exponential backoff (1 min, 5 min, 30 min) | POLICY |
| BR-NTF-005 | Batch digest mode aggregates non-critical notifications into hourly/daily digest per recipient | POLICY |
| BR-NTF-006 | Quiet hours suppress non-critical delivery; critical bypasses quiet hours | POLICY |
| BR-NTF-007 | Template MUST exist for (eventType, channelType, recipient locale) or default-locale fallback; if neither exists, event goes to dead-letter queue | CONSTRAINT |
| BR-NTF-008 | Tenant isolation: notifications MUST NOT cross tenant boundaries; enforced by RLS on `tenant_id` | CONSTRAINT |

---

## 5. Use Cases

Derived from endpoint catalog below. WRITE endpoints = WRITE use cases; GET endpoints = READ use cases.

| Endpoint | Use Case Type | Primary Aggregate |
|----------|--------------|------------------|
| `GET /notifications` | READ | Notification |
| `GET /notifications/{id}` | READ | Notification |
| `POST /notifications/{id}/read` | WRITE | Notification |
| `POST /notifications/read-all` | WRITE | Notification |
| `GET /notification-preferences` | READ | NotificationPreference |
| `PUT /notification-preferences` | WRITE | NotificationPreference |
| `GET /notification-channels` | READ | NotificationChannel |
| `POST /notification-channels` | WRITE | NotificationChannel |
| `PUT /notification-channels/{id}` | WRITE | NotificationChannel |
| `DELETE /notification-channels/{id}` | WRITE | NotificationChannel |
| `GET /notification-templates` | READ | NotificationTemplate |
| `POST /notification-templates` | WRITE | NotificationTemplate |
| `PUT /notification-templates/{id}` | WRITE | NotificationTemplate |

---

## 6. REST API

**Base Path:** `/api/shared/ntf/v1`
**Authentication:** OAuth2 / JWT (Bearer). **Authorization:** scopes `shared.ntf:read`, `shared.ntf:write`, `shared.ntf:admin`.

### 6.1 Endpoint Summary

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/notifications` | List current principal's notifications (filter by status, eventType, date range) |
| `GET` | `/notifications/{id}` | Retrieve single notification |
| `POST` | `/notifications/{id}/read` | Mark read |
| `POST` | `/notifications/read-all` | Mark all read for current principal |
| `GET` | `/notification-preferences` | Read current principal's preferences |
| `PUT` | `/notification-preferences` | Replace preferences |
| `GET` | `/notification-channels` | List tenant channels (admin) |
| `POST` | `/notification-channels` | Register a tenant channel |
| `PUT` | `/notification-channels/{id}` | Update channel config |
| `DELETE` | `/notification-channels/{id}` | Soft-delete (disable) channel |
| `GET` | `/notification-templates` | List templates (filter by eventType, locale) |
| `POST` | `/notification-templates` | Create/version template |
| `PUT` | `/notification-templates/{id}` | Update template |

Full payloads in `contracts/http/shared/ntf/openapi.yaml`.

---

## 7. Events & Integration

### 7.1 Published Events

| Event | Routing Key |
|-------|------------|
| NotificationDelivered | `shared.ntf.notification.delivered` |
| NotificationRead | `shared.ntf.notification.read` |
| NotificationFailed | `shared.ntf.notification.failed` |
| NotificationPreferenceChanged | `shared.ntf.preference.changed` |
| NotificationChannelCreated | `shared.ntf.channel.created` |
| NotificationChannelDisabled | `shared.ntf.channel.disabled` |

### 7.2 Consumed Events

Any domain event from any suite is a candidate source. Routing by explicit subscription rules (`NotificationTemplate.eventType`). Notable currently-wired sources:

- `tks.tkt.*` — ticket lifecycle events (TKS suite)
- `tks.sla.breached`, `tks.sla.approaching-breach`
- `crm.lead.*`, `crm.opp.*`, `crm.sup.*` (while crm.sup runs; see Migration §13)
- `sd.ord.*`, `fi.ap.*`, `fi.ar.*`, `iam.audit.alert.threshold-exceeded`

### 7.3 Routing-Key Bridge (Migration)

For 60 days after the promotion PR merges, `shared-ntf-svc` SHALL also publish every outbound event under its legacy `crm.ntf.*` routing key so existing CRM consumers continue to work. Bridge SHALL be removed after grace period ends (see §13).

---

## 8. Data Model

**Storage:** PostgreSQL 16, Schema: `shared_notification`.

All tables carry `tenant_id` (RLS policy: caller's JWT `tenant_id` claim must match row).

Key tables:

- `notifications` — Notification aggregate (primary + indexes on recipient, status, created_at)
- `notification_preferences` — unique on (tenant_id, principal_id, event_type)
- `notification_channels` — channel registry per tenant
- `notification_templates` — unique on (tenant_id, event_type, channel_type, locale); default tenant = platform-defaults

DDL details deferred to implementation repo (`io.openleap.shared.ntf`).

---

## 9. Security & Compliance

### 9.1 Access Control

| Role | Access Level |
|------|-------------|
| `platform-admin` | Full CRUD on templates + channels |
| `tenant-admin` | CRUD on own-tenant channels, templates, test delivery |
| `tenant-user` | Read own notifications + preferences; write own preferences |
| `service-account` (inter-service) | Publish events via internal queue — no REST access |

### 9.2 Data Classification

- **Confidential** — notification payload may contain references to PII or business data. Payload SHOULD be truncated / masked in logs.
- **Internal** — preferences and channel configuration.

### 9.3 Compliance

- **GDPR**: right-to-erasure supported via `DELETE /notifications?principalId=X` (admin) and automatic purge on principal deletion event `iam.principal.principal.deleted`. 90-day retention enforces data minimisation.
- **DORA**: notifications about ICT incidents (per `iam.audit.alert.threshold-exceeded`) MUST be delivered via at least two channels to at least one tenant-admin; see ADR-NTF-002 (placeholder).

---

## 10. Quality Attributes

### 10.1 Performance Targets

| Operation | p95 | p99 |
|-----------|-----|-----|
| `GET /notifications` (page 50) | < 80 ms | < 200 ms |
| `POST /notifications/{id}/read` | < 50 ms | < 150 ms |
| Event → delivered (in-app) | < 2 s | < 5 s |
| Event → delivered (email/SMS/push) | < 30 s | < 90 s |

### 10.2 Availability

99.9 % (T2 shared service — critical path for user experience and compliance notifications).

### 10.3 Scalability

- Horizontal Kubernetes Deployment (≥ 3 replicas)
- Template evaluation offloaded to worker pool
- Per-channel delivery workers with channel-specific rate limits
- Outbox pattern on publisher side; at-least-once delivery (see ADR-013 platform-wide)

---

## 11. Feature Dependencies

| Feature ID | Feature Name |
|-----------|-------------|
| `F-SHARED-NTF-001` | Notification Center (in-app) |
| `F-SHARED-NTF-002` | Notification Preferences |
| `F-SHARED-NTF-003` | Channel Configuration (admin) |
| `F-SHARED-NTF-004` | Template Management |
| `F-SHARED-NTF-005` | Digest & Quiet Hours |

Consumed by feature specs from multiple suites that require notifications (e.g., `F-TKS-120` SLA & Escalation, `F-CRM-*`, `F-SD-*`).

---

## 12. Extension Points

- Every outbound event available for extension subscribers (`shared.ntf.ext.*` extension-event channel for tenant-custom notification hooks).
- `NotificationChannel.type` is an open enum: new channel-type adapters register via extension API `POST /notification-channels/extensions`.
- `NotificationTemplate.bodyTemplate` supports custom helper registration via `POST /notification-templates/helpers`.

---

## 13. Migration & Evolution

### 13.1 Promotion from `crm-ntf-svc`

- **Spec:** `T3_Domains/CRM/domain-specs/crm_ntf-spec.md` is marked **DEPRECATED** and points here.
- **Routing keys:** new keys `shared.ntf.*`; old keys `crm.ntf.*` re-emitted by this service for a 60-day grace period.
- **API:** new path `/api/shared/ntf/v1`; legacy path `/api/crm/ntf/v1` proxied to new endpoints for 60 days (HTTP 308 redirect).
- **Data:** schema renamed `crm_notification` → `shared_notification` via Flyway migration; tenant boundaries unchanged.
- **Port:** stays at 8416 (no client-side change during migration).

### 13.2 Post-Migration

- All CRM-internal references to `crm-ntf-svc` refactored to `shared-ntf-svc` within 60 days.
- Origin spec (`crm_ntf-spec.md`) retired (status `RETIRED`) two minor versions after this spec reaches `ACTIVE`.

---

## 14. Decisions & Open Questions

| # | Question | Severity | Status |
|---|----------|----------|--------|
| Q-NTF-001 | Should NotificationTemplate live in `param.ref` (platform reference data) instead? | Medium | Open — revisit after one year of use |
| Q-NTF-002 | DORA dual-channel delivery for ICT incidents — which channels count as "independent"? | Medium | Open |
| Q-NTF-003 | Do we need WebSocket push for in-app notifications at T2 tier, or is that a product-tier concern? | Low | Open |

### ADRs

- **ADR-NTF-001** *(Proposed)*: Promote to T2 Shared. Rationale: notification is a cross-suite capability; keeping it in CRM would violate the "suite = UBL boundary" rule when non-CRM suites (TKS, SD, FI) consume it. Consequences: one-time migration cost (spec + routing-key bridge + path proxy); long-term clarity and shared-kernel status.
- **ADR-NTF-002** *(Proposed)*: DORA-driven dual-channel requirement for CRITICAL notifications. To be refined with Q-NTF-002.

---

## 15. Appendix

### 15.1 Glossary

See `T2_SharedBusiness/_t2_suite.md` SS1 for the shared-business UBL.

### 15.2 Change Log

| Date | Version | Author | Changes |
|------|---------|--------|---------|
| 2026-04-18 | 1.0.0 | OpenLeap Architecture Team | Initial promotion spec from crm.ntf (supersedes crm-ntf-svc) |

### 15.3 Companion Files

- OpenAPI: `contracts/http/shared/ntf/openapi.yaml`
- Event Schemas: `contracts/events/shared/ntf/*.schema.json`
- Origin (deprecated): `T3_Domains/CRM/domain-specs/crm_ntf-spec.md`
