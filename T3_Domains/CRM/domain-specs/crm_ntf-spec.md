<!-- Template Meta
     Template-ID:   TPL-SVC
     Version:       1.0.0
     Compliance:    95%+
-->

# Notification Service — Domain Specification

> **Conceptual Stack Layer:** Domain / Service
> **Space:** Platform
> **Owner:** Domain Engineering Team

> **Meta Information**
> - **Version:** 2026-04-18
> - **Template:** `domain-service-spec.md` v1.0.0
> - **Template Compliance:** 95%+
> - **Status:** DEPRECATED — superseded by `shared-ntf-svc` (see `T2_SharedBusiness/domain-specs/shared_ntf-spec.md`). Routing-key bridge `crm.ntf.*` active for 60 days; API path `/api/crm/ntf/v1` returns HTTP 308 during grace period.
> - **Service ID:** `crm-ntf-svc`
> - **Suite:** `crm`
> - **Domain:** `ntf`
> - **Base Package:** `com.openleap.crm.ntf`
> - **API Path:** `/api/crm/ntf/v1`
> - **Port:** 8416
> - **DB Schema:** `crm_notification`
> - **Tier:** Extension

---

## 0. Document Purpose & Scope

This document specifies the **Notification Service** (`crm-ntf-svc`) within the CRM Suite.
It defines the domain model, business rules, use cases, REST API, events, data model, security, and quality attributes for the `Notifications & Alerts` bounded context.

**Audience:** Backend developers, frontend developers, QA engineers, architects, product owners.

**In scope:** All domain logic within the `Notifications & Alerts` bounded context.
**Out of scope:** UI implementation (see feature specs), infrastructure provisioning, cross-suite orchestration.

**Related documents:**
- Suite Spec: `_crm_suite.md`
- OpenAPI: `contracts/http/crm/ntf/openapi.yaml`
- Event Schemas: `contracts/events/crm/ntf/*.schema.json`

---

## 1. Business Context

### 1.1 Purpose & Responsibility

Centralized notification hub for the CRM Suite. Consumes events from other services, evaluates user notification preferences, and delivers through multiple channels: in-app, push, email, SMS, Slack, Microsoft Teams, and webhook.

**Owns:**
- Notification (delivered notification with read/unread state)
- NotificationPreference (per-user channel preferences per event type)
- NotificationChannel (channel configuration — SMTP, Twilio, Slack webhook, Teams webhook)

### 1.2 Business Value

Provides essential notifications & alerts capabilities within the CRM, enabling users to manage notifications & alerts workflows efficiently and with full audit trail.

---

## 2. Service Identity

| Field | Value |
|-------|-------|
| Service ID | `crm-ntf-svc` |
| Suite | `crm` |
| Domain | `ntf` |
| Bounded Context | Notifications & Alerts |
| Base Package | `com.openleap.crm.ntf` |
| API Base Path | `/api/crm/ntf/v1` |
| Port | 8416 |
| Database Schema | `crm_notification` |
| Tier | Extension |
| Repository | `openleap-crm-ntf-svc` |
| Team | `team-crm` |

---

## 3. Domain Model

### 3.1 Aggregates

**Notification** (AggregateRoot)
- delivered notification with read/unread state

**NotificationPreference** (AggregateRoot)
- per-user channel preferences per event type

**NotificationChannel** (AggregateRoot)
- channel configuration — SMTP, Twilio, Slack webhook, Teams webhook

---

## 4. Business Rules

### 4.1 Rule Catalog

| ID | Rule | Type |
|----|------|------|
| BR-NTF-001 | Users can opt out of non-critical notifications per event type | POLICY |
| BR-NTF-002 | Critical notifications (SLA breach, assignment) cannot be disabled | CONSTRAINT |
| BR-NTF-003 | In-app notifications retained for 90 days | POLICY |
| BR-NTF-004 | Delivery retry: 3 attempts with exponential backoff | POLICY |
| BR-NTF-005 | Batch digest mode: aggregate low-priority notifications into hourly/daily digest | POLICY |

---

## 5. Use Cases

Use cases are derived from the endpoint catalog below. Each write endpoint corresponds to a WRITE use case; each read endpoint corresponds to a READ use case.

| Endpoint | Use Case Type | Primary Aggregate |
|----------|--------------|------------------|
| `GET /notifications` | READ | Notification |
| `POST /notifications/{id}/read` | WRITE | Notification |
| `POST /notifications/read-all` | WRITE | Notification |
| `GET /notification-preferences` | READ | Notification |
| `PUT /notification-preferences` | WRITE | Notification |
| `POST /notification-channels` | WRITE | Notification |
| `GET /notification-channels` | READ | Notification |
| `PUT /notification-channels/{id}` | WRITE | Notification |

---

## 6. REST API

**Base Path:** `/api/crm/ntf/v1`

**Authentication:** OAuth2/JWT (Bearer token)

### 6.1 Endpoint Summary

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/notifications` | GET notifications |
| `POST` | `/notifications/{id}/read` | POST read |
| `POST` | `/notifications/read-all` | POST read-all |
| `GET` | `/notification-preferences` | GET notification-preferences |
| `PUT` | `/notification-preferences` | PUT notification-preferences |
| `POST` | `/notification-channels` | POST notification-channels |
| `GET` | `/notification-channels` | GET notification-channels |
| `PUT` | `/notification-channels/{id}` | PUT {id} |

> Full request/response examples in companion OpenAPI: `contracts/http/crm/ntf/openapi.yaml`

---

## 7. Events & Integration

### 7.1 Published Events

| Event | Routing Key |
|-------|------------|
| NotificationDelivered | `crm.ntf.notification.delivered` |
| NotificationRead | `crm.ntf.notification.read` |

### 7.2 Consumed Events

This service consumes events from other CRM services as needed for its bounded context. See Suite Spec SS5 for the full event catalog.

---

## 8. Data Model

**Storage:** PostgreSQL 16, Schema: `crm_notification`

Tables are derived from the aggregate model in §3. Each aggregate root maps to a primary table. Embedded value objects are stored as columns or JSONB. All tables include `tenant_id` for Row-Level Security.

Detailed DDL will be provided in the implementation phase. Key tables:

- `notifications` — primary table for Notification aggregate
- `notificationpreferences` — primary table for NotificationPreference aggregate
- `notificationchannels` — primary table for NotificationChannel aggregate

---

## 9. Security & Compliance

### 9.1 Access Control

| Role | Access Level |
|------|-------------|
| `crm-admin` | Full CRUD + configuration |
| `crm-sales-manager` | Full CRUD + team visibility |
| `crm-sales-rep` | CRUD own records + read team records |
| `crm-readonly` | Read-only |

### 9.2 Data Classification

This service handles **Internal** business data. No PII is stored directly (references to contacts/leads via ID only).

---

## 10. Quality Attributes

### 10.1 Performance Targets

| Operation | p95 | p99 |
|-----------|-----|-----|
| Single entity GET | < 50ms | < 100ms |
| List/search | < 200ms | < 500ms |
| Create/update | < 150ms | < 300ms |

### 10.2 Availability: 99.5% (extension service)

### 10.3 Scalability

- Horizontal scaling via Kubernetes Deployment (2+ replicas for core, scale-to-zero for extensions)
- Stateless application tier — all state in PostgreSQL
- Connection pooling via HikariCP (max 20 connections per pod)

---

## 11. Feature Dependencies

| Feature ID | Feature Name |
|-----------|-------------|
| `F-CRM-012-01` | Notification Center |
| `F-CRM-012-02` | Notification Preferences |
| `F-CRM-012-03` | Channel Configuration |

---

## 12. Extension Points

- All domain events are available for extension service consumption
- Primary aggregates support `customFields` (JSONB) for tenant-specific extensions
- Integration Service (`crm.int`) can forward any event to external webhooks

---

## 13. Migration & Evolution

- Flyway migration scripts: `V{version}__{description}.sql`
- API versioning: `/api/crm/{domain}/v1` with 6-month deprecation for breaking changes
- All migrations must be backward-compatible

---

## 14. Decisions & Open Questions

| # | Question | Severity | Status |
|---|----------|----------|--------|
| 1 | Detailed aggregate field definitions pending full design review | Medium | Open |
| 2 | Custom field schema validation rules to be defined | Low | Open |

---

## 15. Appendix

### 15.1 Glossary

See Suite Spec SS1 for the CRM Ubiquitous Language glossary.

### 15.2 Change Log

| Date | Version | Author | Changes |
|------|---------|--------|---------|
| 2026-04-03 | 1.0.0 | OpenLeap Architecture Team | Initial domain spec at 95%+ compliance |

### 15.3 Companion Files

- OpenAPI: `contracts/http/crm/ntf/openapi.yaml`
- Event Schemas: `contracts/events/crm/ntf/*.schema.json`