<!-- Template Meta
     Template-ID:   TPL-SVC
     Version:       1.0.0
     Compliance:    95%+
-->

# Product Catalog Service — Domain Specification

> **Conceptual Stack Layer:** Domain / Service
> **Space:** Platform
> **Owner:** Domain Engineering Team

> **Meta Information**
> - **Version:** 2026-04-03
> - **Template:** `domain-service-spec.md` v1.0.0
> - **Template Compliance:** 95%+
> - **Status:** DRAFT
> - **Service ID:** `crm-prod-svc`
> - **Suite:** `crm`
> - **Domain:** `prod`
> - **Base Package:** `com.openleap.crm.prod`
> - **API Path:** `/api/crm/prod/v1`
> - **Port:** 8405
> - **DB Schema:** `crm_product`
> - **Tier:** Core

---

## 0. Document Purpose & Scope

This document specifies the **Product Catalog Service** (`crm-prod-svc`) within the CRM Suite.
It defines the domain model, business rules, use cases, REST API, events, data model, security, and quality attributes for the `Products & Pricing` bounded context.

**Audience:** Backend developers, frontend developers, QA engineers, architects, product owners.

**In scope:** All domain logic within the `Products & Pricing` bounded context.
**Out of scope:** UI implementation (see feature specs), infrastructure provisioning, cross-suite orchestration.

**Related documents:**
- Suite Spec: `_crm_suite.md`
- OpenAPI: `contracts/http/crm/prod/openapi.yaml`
- Event Schemas: `contracts/events/crm/prod/*.schema.json`

---

## 1. Business Context

### 1.1 Purpose & Responsibility

Manages the product catalog, price books, and discount rules used by Opportunity line items and CPQ quotes. Provides a lightweight product model optimized for sales quoting — not a full commerce catalog.

**Owns:**
- Product (catalog entry with type, pricing, tax class, recurring flag)
- PriceBook (named collection of product prices — Standard, Enterprise, Partner)
- PriceBookEntry (product + unit price within a price book)
- DiscountRule (conditional discount — percentage, fixed amount, tiered)

### 1.2 Business Value

Provides essential products & pricing capabilities within the CRM, enabling users to manage products & pricing workflows efficiently and with full audit trail.

---

## 2. Service Identity

| Field | Value |
|-------|-------|
| Service ID | `crm-prod-svc` |
| Suite | `crm` |
| Domain | `prod` |
| Bounded Context | Products & Pricing |
| Base Package | `com.openleap.crm.prod` |
| API Base Path | `/api/crm/prod/v1` |
| Port | 8405 |
| Database Schema | `crm_product` |
| Tier | Core |
| Repository | `openleap-crm-prod-svc` |
| Team | `team-crm` |

---

## 3. Domain Model

### 3.1 Aggregates

**Product** (AggregateRoot)
- catalog entry with type, pricing, tax class, recurring flag

**PriceBook** (AggregateRoot)
- named collection of product prices — Standard, Enterprise, Partner

**PriceBookEntry** (AggregateRoot)
- product + unit price within a price book

**DiscountRule** (AggregateRoot)
- conditional discount — percentage, fixed amount, tiered

---

## 4. Business Rules

### 4.1 Rule Catalog

| ID | Rule | Type |
|----|------|------|
| BR-PROD-001 | Product code (SKU) must be unique per tenant | CONSTRAINT |
| BR-PROD-002 | Each tenant has exactly one default price book | POLICY |
| BR-PROD-003 | Price book entries must reference active products | CONSTRAINT |
| BR-PROD-004 | Discount rules are evaluated in priority order — first match wins | POLICY |
| BR-PROD-005 | Product deactivation is blocked if referenced in open opportunities or quotes | POLICY |

---

## 5. Use Cases

Use cases are derived from the endpoint catalog below. Each write endpoint corresponds to a WRITE use case; each read endpoint corresponds to a READ use case.

| Endpoint | Use Case Type | Primary Aggregate |
|----------|--------------|------------------|
| `POST /products` | WRITE | Product |
| `GET /products` | READ | Product |
| `GET /products/{id}` | READ | Product |
| `PUT /products/{id}` | WRITE | Product |
| `POST /price-books` | WRITE | Product |
| `GET /price-books` | READ | Product |
| `GET /price-books/{id}/entries` | READ | Product |
| `PUT /price-books/{id}/entries/{entryId}` | WRITE | Product |
| `POST /discount-rules` | WRITE | Product |
| `GET /discount-rules` | READ | Product |

---

## 6. REST API

**Base Path:** `/api/crm/prod/v1`

**Authentication:** OAuth2/JWT (Bearer token)

### 6.1 Endpoint Summary

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/products` | POST products |
| `GET` | `/products` | GET products |
| `GET` | `/products/{id}` | GET {id} |
| `PUT` | `/products/{id}` | PUT {id} |
| `POST` | `/price-books` | POST price-books |
| `GET` | `/price-books` | GET price-books |
| `GET` | `/price-books/{id}/entries` | GET entries |
| `PUT` | `/price-books/{id}/entries/{entryId}` | PUT {entryId} |
| `POST` | `/discount-rules` | POST discount-rules |
| `GET` | `/discount-rules` | GET discount-rules |

> Full request/response examples in companion OpenAPI: `contracts/http/crm/prod/openapi.yaml`

---

## 7. Events & Integration

### 7.1 Published Events

| Event | Routing Key |
|-------|------------|
| ProductCreated | `crm.prod.product.created` |
| ProductUpdated | `crm.prod.product.updated` |
| ProductDeactivated | `crm.prod.product.deactivated` |
| PriceBookCreated | `crm.prod.price_book.created` |
| PriceBookEntryUpdated | `crm.prod.price_book_entry.updated` |
| DiscountRuleCreated | `crm.prod.discount_rule.created` |

### 7.2 Consumed Events

This service consumes events from other CRM services as needed for its bounded context. See Suite Spec SS5 for the full event catalog.

---

## 8. Data Model

**Storage:** PostgreSQL 16, Schema: `crm_product`

Tables are derived from the aggregate model in §3. Each aggregate root maps to a primary table. Embedded value objects are stored as columns or JSONB. All tables include `tenant_id` for Row-Level Security.

Detailed DDL will be provided in the implementation phase. Key tables:

- `products` — primary table for Product aggregate
- `pricebooks` — primary table for PriceBook aggregate
- `pricebookentrys` — primary table for PriceBookEntry aggregate
- `discountrules` — primary table for DiscountRule aggregate

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

### 10.2 Availability: 99.9% (core service)

### 10.3 Scalability

- Horizontal scaling via Kubernetes Deployment (2+ replicas for core, scale-to-zero for extensions)
- Stateless application tier — all state in PostgreSQL
- Connection pooling via HikariCP (max 20 connections per pod)

---

## 11. Feature Dependencies

| Feature ID | Feature Name |
|-----------|-------------|
| `F-CRM-005-01` | Product Catalog Browser |
| `F-CRM-005-02` | Product Create/Edit |
| `F-CRM-005-03` | Price Book Management |
| `F-CRM-005-04` | Discount Rule Configuration |

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

- OpenAPI: `contracts/http/crm/prod/openapi.yaml`
- Event Schemas: `contracts/events/crm/prod/*.schema.json`