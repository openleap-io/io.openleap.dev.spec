<!-- Template Meta
     Template-ID:   TPL-SVC
     Version:       1.0.0
     Compliance:    95%+
-->

# Search Service — Domain Specification

> **Conceptual Stack Layer:** Domain / Service
> **Space:** Platform
> **Owner:** Domain Engineering Team

> **Meta Information**
> - **Version:** 2026-04-03
> - **Template:** `domain-service-spec.md` v1.0.0
> - **Template Compliance:** 95%+
> - **Status:** DRAFT
> - **Service ID:** `crm-search-svc`
> - **Suite:** `crm`
> - **Domain:** `search`
> - **Base Package:** `com.openleap.crm.search`
> - **API Path:** `/api/crm/search/v1`
> - **Port:** 8406
> - **DB Schema:** `Elasticsearch`
> - **Tier:** Core

---

## 0. Document Purpose & Scope

This document specifies the **Search Service** (`crm-search-svc`) within the CRM Suite.
It defines the domain model, business rules, use cases, REST API, events, data model, security, and quality attributes for the `Global CRM Search` bounded context.

**Audience:** Backend developers, frontend developers, QA engineers, architects, product owners.

**In scope:** All domain logic within the `Global CRM Search` bounded context.
**Out of scope:** UI implementation (see feature specs), infrastructure provisioning, cross-suite orchestration.

**Related documents:**
- Suite Spec: `_crm_suite.md`
- OpenAPI: `contracts/http/crm/search/openapi.yaml`
- Event Schemas: `contracts/events/crm/search/*.schema.json`

---

## 1. Business Context

### 1.1 Purpose & Responsibility

Provides global full-text search, faceted filtering, and saved views across all CRM entities. Pure CQRS read model — consumes events from all CRM services and projects them into Elasticsearch indices. Does not own any domain data.

**Owns:**
- SavedView (persisted filter + column configuration, user or team scoped)
- IndexSchema (Elasticsearch mapping definition per entity type — managed internally)

### 1.2 Business Value

Provides essential global crm search capabilities within the CRM, enabling users to manage global crm search workflows efficiently and with full audit trail.

---

## 2. Service Identity

| Field | Value |
|-------|-------|
| Service ID | `crm-search-svc` |
| Suite | `crm` |
| Domain | `search` |
| Bounded Context | Global CRM Search |
| Base Package | `com.openleap.crm.search` |
| API Base Path | `/api/crm/search/v1` |
| Port | 8406 |
| Database Schema | `Elasticsearch` |
| Tier | Core |
| Repository | `openleap-crm-search-svc` |
| Team | `team-crm` |

---

## 3. Domain Model

### 3.1 Aggregates

**SavedView** (AggregateRoot)
- persisted filter + column configuration, user or team scoped

**IndexSchema** (AggregateRoot)
- Elasticsearch mapping definition per entity type — managed internally

---

## 4. Business Rules

### 4.1 Rule Catalog

| ID | Rule | Type |
|----|------|------|
| BR-SEARCH-001 | All queries must include tenant_id filter (enforced at query layer) | CONSTRAINT |
| BR-SEARCH-002 | Search results return entity type + id + highlighted matches — detail fetched from owning service | POLICY |
| BR-SEARCH-003 | Saved views can be private (user) or shared (team) | POLICY |
| BR-SEARCH-004 | Reindex is admin-only and processes all events from offset 0 | POLICY |
| BR-SEARCH-005 | Maximum query result size: 10,000 records | POLICY |

---

## 5. Use Cases

Use cases are derived from the endpoint catalog below. Each write endpoint corresponds to a WRITE use case; each read endpoint corresponds to a READ use case.

| Endpoint | Use Case Type | Primary Aggregate |
|----------|--------------|------------------|
| `POST /search` | WRITE | SavedView |
| `GET /search/suggest` | READ | SavedView |
| `POST /saved-views` | WRITE | SavedView |
| `GET /saved-views` | READ | SavedView |
| `GET /saved-views/{id}` | READ | SavedView |
| `PUT /saved-views/{id}` | WRITE | SavedView |
| `DELETE /saved-views/{id}` | WRITE | SavedView |
| `POST /reindex` | WRITE | SavedView |

---

## 6. REST API

**Base Path:** `/api/crm/search/v1`

**Authentication:** OAuth2/JWT (Bearer token)

### 6.1 Endpoint Summary

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/search` | POST search |
| `GET` | `/search/suggest` | GET suggest |
| `POST` | `/saved-views` | POST saved-views |
| `GET` | `/saved-views` | GET saved-views |
| `GET` | `/saved-views/{id}` | GET {id} |
| `PUT` | `/saved-views/{id}` | PUT {id} |
| `DELETE` | `/saved-views/{id}` | DELETE {id} |
| `POST` | `/reindex` | POST reindex |

> Full request/response examples in companion OpenAPI: `contracts/http/crm/search/openapi.yaml`

---

## 7. Events & Integration

### 7.1 Published Events

This service does not publish domain events (pure consumer / infrastructure service).

### 7.2 Consumed Events

This service consumes events from other CRM services as needed for its bounded context. See Suite Spec SS5 for the full event catalog.

---

## 8. Data Model

**Storage:** Elasticsearch 8.x (CQRS read model — no PostgreSQL)

Elasticsearch indices are maintained as denormalized projections from domain events. Index schemas are defined per entity type (contacts, leads, opportunities, activities, products, tickets, etc.).

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
- Stateless application tier — all state in PostgreSQL / Elasticsearch
- Connection pooling via HikariCP (max 20 connections per pod)

---

## 11. Feature Dependencies

| Feature ID | Feature Name |
|-----------|-------------|
| `F-CRM-006-01` | Global Search Bar |
| `F-CRM-006-02` | Advanced Search & Filters |
| `F-CRM-006-03` | Saved Views & Segments |

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

- OpenAPI: `contracts/http/crm/search/openapi.yaml`
- Event Schemas: `contracts/events/crm/search/*.schema.json`