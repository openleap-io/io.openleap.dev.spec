<!-- Template Meta
     Template-ID:   TPL-SVC
     Version:       1.0.0
     Compliance:    95%+
-->

# CPQ Service — Domain Specification

> **Conceptual Stack Layer:** Domain / Service
> **Space:** Platform
> **Owner:** Domain Engineering Team

> **Meta Information**
> - **Version:** 2026-04-03
> - **Template:** `domain-service-spec.md` v1.0.0
> - **Template Compliance:** 95%+
> - **Status:** DRAFT
> - **Service ID:** `crm-cpq-svc`
> - **Suite:** `crm`
> - **Domain:** `cpq`
> - **Base Package:** `com.openleap.crm.cpq`
> - **API Path:** `/api/crm/cpq/v1`
> - **Port:** 8421
> - **DB Schema:** `crm_cpq`
> - **Tier:** Extension

---

## 0. Document Purpose & Scope

This document specifies the **CPQ Service** (`crm-cpq-svc`) within the CRM Suite.
It defines the domain model, business rules, use cases, REST API, events, data model, security, and quality attributes for the `Configure-Price-Quote` bounded context.

**Audience:** Backend developers, frontend developers, QA engineers, architects, product owners.

**In scope:** All domain logic within the `Configure-Price-Quote` bounded context.
**Out of scope:** UI implementation (see feature specs), infrastructure provisioning, cross-suite orchestration.

**Related documents:**
- Suite Spec: `_crm_suite.md`
- OpenAPI: `contracts/http/crm/cpq/openapi.yaml`
- Event Schemas: `contracts/events/crm/cpq/*.schema.json`

---

## 1. Business Context

### 1.1 Purpose & Responsibility

Configure-Price-Quote engine — create quotes linked to opportunities, add product line items with pricing and discounts, manage multi-level approval workflows, generate PDF proposals, and handle customer acceptance with e-signature.

**Owns:**
- Quote (price proposal with line items, approval status, validity period, PDF reference)
- QuoteLineItem (product + quantity + unit price + discount per line)
- ApprovalChain (configurable approval hierarchy — per discount threshold or total amount)
- QuoteAcceptance (customer acceptance record with signature and timestamp)

### 1.2 Business Value

Provides essential configure-price-quote capabilities within the CRM, enabling users to manage configure-price-quote workflows efficiently and with full audit trail.

---

## 2. Service Identity

| Field | Value |
|-------|-------|
| Service ID | `crm-cpq-svc` |
| Suite | `crm` |
| Domain | `cpq` |
| Bounded Context | Configure-Price-Quote |
| Base Package | `com.openleap.crm.cpq` |
| API Base Path | `/api/crm/cpq/v1` |
| Port | 8421 |
| Database Schema | `crm_cpq` |
| Tier | Extension |
| Repository | `openleap-crm-cpq-svc` |
| Team | `team-crm` |

---

## 3. Domain Model

### 3.1 Aggregates

**Quote** (AggregateRoot)
- price proposal with line items, approval status, validity period, PDF reference

**QuoteLineItem** (AggregateRoot)
- product + quantity + unit price + discount per line

**ApprovalChain** (AggregateRoot)
- configurable approval hierarchy — per discount threshold or total amount

**QuoteAcceptance** (AggregateRoot)
- customer acceptance record with signature and timestamp

---

## 4. Business Rules

### 4.1 Rule Catalog

| ID | Rule | Type |
|----|------|------|
| BR-CPQ-001 | Quote amount = sum of (quantity × unit price - line discount) for all line items | POLICY |
| BR-CPQ-002 | Quotes exceeding discount threshold require approval per approval chain | POLICY |
| BR-CPQ-003 | Quote validity: configurable per tenant (default 30 days) | POLICY |
| BR-CPQ-004 | PDF generation uses document template service | POLICY |
| BR-CPQ-005 | Accepted quotes cannot be modified — create new version instead | CONSTRAINT |
| BR-CPQ-006 | Quote versioning: v1 → v2 on edit after submission | POLICY |

---

## 5. Use Cases

Use cases are derived from the endpoint catalog below. Each write endpoint corresponds to a WRITE use case; each read endpoint corresponds to a READ use case.

| Endpoint | Use Case Type | Primary Aggregate |
|----------|--------------|------------------|
| `POST /quotes` | WRITE | Quote |
| `GET /quotes` | READ | Quote |
| `GET /quotes/{id}` | READ | Quote |
| `PUT /quotes/{id}` | WRITE | Quote |
| `POST /quotes/{id}/submit` | WRITE | Quote |
| `POST /quotes/{id}/approve` | WRITE | Quote |
| `POST /quotes/{id}/reject` | WRITE | Quote |
| `POST /quotes/{id}/generate-pdf` | WRITE | Quote |
| `GET /quotes/{id}/pdf` | READ | Quote |
| `POST /quotes/{id}/send` | WRITE | Quote |
| `POST /quotes/{id}/accept` | WRITE | Quote |
| `POST /approval-chains` | WRITE | Quote |
| `GET /approval-chains` | READ | Quote |

---

## 6. REST API

**Base Path:** `/api/crm/cpq/v1`

**Authentication:** OAuth2/JWT (Bearer token)

### 6.1 Endpoint Summary

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/quotes` | POST quotes |
| `GET` | `/quotes` | GET quotes |
| `GET` | `/quotes/{id}` | GET {id} |
| `PUT` | `/quotes/{id}` | PUT {id} |
| `POST` | `/quotes/{id}/submit` | POST submit |
| `POST` | `/quotes/{id}/approve` | POST approve |
| `POST` | `/quotes/{id}/reject` | POST reject |
| `POST` | `/quotes/{id}/generate-pdf` | POST generate-pdf |
| `GET` | `/quotes/{id}/pdf` | GET pdf |
| `POST` | `/quotes/{id}/send` | POST send |
| `POST` | `/quotes/{id}/accept` | POST accept |
| `POST` | `/approval-chains` | POST approval-chains |
| `GET` | `/approval-chains` | GET approval-chains |

> Full request/response examples in companion OpenAPI: `contracts/http/crm/cpq/openapi.yaml`

---

## 7. Events & Integration

### 7.1 Published Events

| Event | Routing Key |
|-------|------------|
| QuoteCreated | `crm.cpq.quote.created` |
| QuoteSubmitted | `crm.cpq.quote.submitted` |
| QuoteApproved | `crm.cpq.quote.approved` |
| QuoteRejected | `crm.cpq.quote.rejected` |
| QuoteAccepted | `crm.cpq.quote.accepted` |
| QuoteExpired | `crm.cpq.quote.expired` |

### 7.2 Consumed Events

This service consumes events from other CRM services as needed for its bounded context. See Suite Spec SS5 for the full event catalog.

---

## 8. Data Model

**Storage:** PostgreSQL 16, Schema: `crm_cpq`

Tables are derived from the aggregate model in §3. Each aggregate root maps to a primary table. Embedded value objects are stored as columns or JSONB. All tables include `tenant_id` for Row-Level Security.

Detailed DDL will be provided in the implementation phase. Key tables:

- `quotes` — primary table for Quote aggregate
- `quotelineitems` — primary table for QuoteLineItem aggregate
- `approvalchains` — primary table for ApprovalChain aggregate
- `quoteacceptances` — primary table for QuoteAcceptance aggregate

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
| `F-CRM-017-01` | Quote List |
| `F-CRM-017-02` | Quote Builder |
| `F-CRM-017-03` | Quote Approval Workflow |
| `F-CRM-017-04` | Quote PDF Generation |
| `F-CRM-017-05` | Quote Acceptance Portal |

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

- OpenAPI: `contracts/http/crm/cpq/openapi.yaml`
- Event Schemas: `contracts/events/crm/cpq/*.schema.json`