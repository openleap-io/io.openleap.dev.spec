<!-- Template Meta
     Template-ID:   TPL-SVC
     Version:       1.0.0
     Compliance:    95%+
-->

# Email Service — Domain Specification

> **Conceptual Stack Layer:** Domain / Service
> **Space:** Platform
> **Owner:** Domain Engineering Team

> **Meta Information**
> - **Version:** 2026-04-18
> - **Template:** `domain-service-spec.md` v1.0.0
> - **Template Compliance:** 95%+
> - **Status:** DEPRECATED — superseded by `tech-email-svc` (see `T1_Platform/tech/domain-specs/tech_email-spec.md`). Routing-key bridge `crm.email.*` active for 60 days; API path `/api/crm/email/v1` returns HTTP 308 during grace period.
> - **Service ID:** `crm-email-svc`
> - **Suite:** `crm`
> - **Domain:** `email`
> - **Base Package:** `com.openleap.crm.email`
> - **API Path:** `/api/crm/email/v1`
> - **Port:** 8411
> - **DB Schema:** `crm_email`
> - **Tier:** Extension

---

## 0. Document Purpose & Scope

This document specifies the **Email Service** (`crm-email-svc`) within the CRM Suite.
It defines the domain model, business rules, use cases, REST API, events, data model, security, and quality attributes for the `Email Communication` bounded context.

**Audience:** Backend developers, frontend developers, QA engineers, architects, product owners.

**In scope:** All domain logic within the `Email Communication` bounded context.
**Out of scope:** UI implementation (see feature specs), infrastructure provisioning, cross-suite orchestration.

**Related documents:**
- Suite Spec: `_crm_suite.md`
- OpenAPI: `contracts/http/crm/email/openapi.yaml`
- Event Schemas: `contracts/events/crm/email/*.schema.json`

---

## 1. Business Context

### 1.1 Purpose & Responsibility

Handles email composition, sending (SMTP/SendGrid/SES), receiving (IMAP sync), template management with merge fields, and open/click tracking. Links all emails to CRM contacts and accounts for timeline enrichment.

**Owns:**
- EmailMessage (sent/received email with tracking metadata, attachments)
- EmailTemplate (reusable HTML template with merge fields)
- EmailAccount (SMTP/IMAP connection per user)
- EmailTracking (open/click events per message)

### 1.2 Business Value

Provides essential email communication capabilities within the CRM, enabling users to manage email communication workflows efficiently and with full audit trail.

---

## 2. Service Identity

| Field | Value |
|-------|-------|
| Service ID | `crm-email-svc` |
| Suite | `crm` |
| Domain | `email` |
| Bounded Context | Email Communication |
| Base Package | `com.openleap.crm.email` |
| API Base Path | `/api/crm/email/v1` |
| Port | 8411 |
| Database Schema | `crm_email` |
| Tier | Extension |
| Repository | `openleap-crm-email-svc` |
| Team | `team-crm` |

---

## 3. Domain Model

### 3.1 Aggregates

**EmailMessage** (AggregateRoot)
- sent/received email with tracking metadata, attachments

**EmailTemplate** (AggregateRoot)
- reusable HTML template with merge fields

**EmailAccount** (AggregateRoot)
- SMTP/IMAP connection per user

**EmailTracking** (AggregateRoot)
- open/click events per message

---

## 4. Business Rules

### 4.1 Rule Catalog

| ID | Rule | Type |
|----|------|------|
| BR-EMAIL-001 | Rate limit: 500 emails/hour per tenant (configurable) | POLICY |
| BR-EMAIL-002 | Max 50 recipients per email | POLICY |
| BR-EMAIL-003 | Tracking pixel injection is automatic for HTML emails | POLICY |
| BR-EMAIL-004 | Unsubscribe link mandatory in bulk/marketing emails | POLICY |
| BR-EMAIL-005 | IMAP sync runs every 5 minutes per connected account | POLICY |
| BR-EMAIL-006 | Bounce handling: soft bounce retries 3x, hard bounce marks contact | POLICY |

---

## 5. Use Cases

Use cases are derived from the endpoint catalog below. Each write endpoint corresponds to a WRITE use case; each read endpoint corresponds to a READ use case.

| Endpoint | Use Case Type | Primary Aggregate |
|----------|--------------|------------------|
| `POST /emails/send` | WRITE | EmailMessage |
| `POST /emails/send-template` | WRITE | EmailMessage |
| `GET /emails` | READ | EmailMessage |
| `GET /emails/{id}` | READ | EmailMessage |
| `GET /emails/{id}/tracking` | READ | EmailMessage |
| `POST /email-templates` | WRITE | EmailMessage |
| `GET /email-templates` | READ | EmailMessage |
| `PUT /email-templates/{id}` | WRITE | EmailMessage |
| `POST /email-accounts` | WRITE | EmailMessage |
| `GET /email-accounts` | READ | EmailMessage |
| `POST /email-accounts/{id}/sync` | WRITE | EmailMessage |

---

## 6. REST API

**Base Path:** `/api/crm/email/v1`

**Authentication:** OAuth2/JWT (Bearer token)

### 6.1 Endpoint Summary

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/emails/send` | POST send |
| `POST` | `/emails/send-template` | POST send-template |
| `GET` | `/emails` | GET emails |
| `GET` | `/emails/{id}` | GET {id} |
| `GET` | `/emails/{id}/tracking` | GET tracking |
| `POST` | `/email-templates` | POST email-templates |
| `GET` | `/email-templates` | GET email-templates |
| `PUT` | `/email-templates/{id}` | PUT {id} |
| `POST` | `/email-accounts` | POST email-accounts |
| `GET` | `/email-accounts` | GET email-accounts |
| `POST` | `/email-accounts/{id}/sync` | POST sync |

> Full request/response examples in companion OpenAPI: `contracts/http/crm/email/openapi.yaml`

---

## 7. Events & Integration

### 7.1 Published Events

| Event | Routing Key |
|-------|------------|
| EmailSent | `crm.email.email.sent` |
| EmailReceived | `crm.email.email.received` |
| EmailOpened | `crm.email.email.opened` |
| EmailClicked | `crm.email.email.clicked` |
| EmailBounced | `crm.email.email.bounced` |

### 7.2 Consumed Events

This service consumes events from other CRM services as needed for its bounded context. See Suite Spec SS5 for the full event catalog.

---

## 8. Data Model

**Storage:** PostgreSQL 16, Schema: `crm_email`

Tables are derived from the aggregate model in §3. Each aggregate root maps to a primary table. Embedded value objects are stored as columns or JSONB. All tables include `tenant_id` for Row-Level Security.

Detailed DDL will be provided in the implementation phase. Key tables:

- `emailmessages` — primary table for EmailMessage aggregate
- `emailtemplates` — primary table for EmailTemplate aggregate
- `emailaccounts` — primary table for EmailAccount aggregate
- `emailtrackings` — primary table for EmailTracking aggregate

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

This service handles **Confidential (PII)** data. GDPR requirements apply: right to access, rectification, erasure, and portability.

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
| `F-CRM-007-01` | Email Compose & Send |
| `F-CRM-007-02` | Email Inbox & Thread View |
| `F-CRM-007-03` | Email Template Manager |
| `F-CRM-007-04` | Email Tracking Dashboard |

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

- OpenAPI: `contracts/http/crm/email/openapi.yaml`
- Event Schemas: `contracts/events/crm/email/*.schema.json`