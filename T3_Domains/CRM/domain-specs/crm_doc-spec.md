<!-- Template Meta
     Template-ID:   TPL-SVC
     Version:       1.0.0
     Compliance:    95%+
-->

# Document Service — Domain Specification

> **Conceptual Stack Layer:** Domain / Service
> **Space:** Platform
> **Owner:** Domain Engineering Team

> **Meta Information**
> - **Version:** 2026-04-03
> - **Template:** `domain-service-spec.md` v1.0.0
> - **Template Compliance:** 95%+
> - **Status:** DRAFT
> - **Service ID:** `crm-doc-svc`
> - **Suite:** `crm`
> - **Domain:** `doc`
> - **Base Package:** `com.openleap.crm.doc`
> - **API Path:** `/api/crm/doc/v1`
> - **Port:** 8413
> - **DB Schema:** `crm_document`
> - **Tier:** Extension

---

## 0. Document Purpose & Scope

This document specifies the **Document Service** (`crm-doc-svc`) within the CRM Suite.
It defines the domain model, business rules, use cases, REST API, events, data model, security, and quality attributes for the `Document Management` bounded context.

**Audience:** Backend developers, frontend developers, QA engineers, architects, product owners.

**In scope:** All domain logic within the `Document Management` bounded context.
**Out of scope:** UI implementation (see feature specs), infrastructure provisioning, cross-suite orchestration.

**Related documents:**
- Suite Spec: `_crm_suite.md`
- OpenAPI: `contracts/http/crm/doc/openapi.yaml`
- Event Schemas: `contracts/events/crm/doc/*.schema.json`

---

## 1. Business Context

### 1.1 Purpose & Responsibility

Manages document storage (MinIO), attachment linking to CRM records, and template-based PDF generation with merge fields. Supports version history and access control.

**Owns:**
- Document (file metadata + MinIO storage reference + version history)
- DocumentTemplate (merge-field template for PDF generation)
- DocumentLink (polymorphic link between a document and a CRM record)

### 1.2 Business Value

Provides essential document management capabilities within the CRM, enabling users to manage document management workflows efficiently and with full audit trail.

---

## 2. Service Identity

| Field | Value |
|-------|-------|
| Service ID | `crm-doc-svc` |
| Suite | `crm` |
| Domain | `doc` |
| Bounded Context | Document Management |
| Base Package | `com.openleap.crm.doc` |
| API Base Path | `/api/crm/doc/v1` |
| Port | 8413 |
| Database Schema | `crm_document` |
| Tier | Extension |
| Repository | `openleap-crm-doc-svc` |
| Team | `team-crm` |

---

## 3. Domain Model

### 3.1 Aggregates

**Document** (AggregateRoot)
- file metadata + MinIO storage reference + version history

**DocumentTemplate** (AggregateRoot)
- merge-field template for PDF generation

**DocumentLink** (AggregateRoot)
- polymorphic link between a document and a CRM record

---

## 4. Business Rules

### 4.1 Rule Catalog

| ID | Rule | Type |
|----|------|------|
| BR-DOC-001 | Max file size: 50MB per document | POLICY |
| BR-DOC-002 | Supported formats: PDF, DOCX, XLSX, PNG, JPG, CSV | POLICY |
| BR-DOC-003 | Documents stored in MinIO with tenant-prefixed paths | POLICY |
| BR-DOC-004 | Template merge fields resolve from contact, account, opportunity, quote data | POLICY |
| BR-DOC-005 | Virus scanning on upload via ClamAV integration | POLICY |

---

## 5. Use Cases

Use cases are derived from the endpoint catalog below. Each write endpoint corresponds to a WRITE use case; each read endpoint corresponds to a READ use case.

| Endpoint | Use Case Type | Primary Aggregate |
|----------|--------------|------------------|
| `POST /documents/upload` | WRITE | Document |
| `GET /documents` | READ | Document |
| `GET /documents/{id}` | READ | Document |
| `GET /documents/{id}/download` | READ | Document |
| `DELETE /documents/{id}` | WRITE | Document |
| `POST /document-templates` | WRITE | Document |
| `GET /document-templates` | READ | Document |
| `POST /document-templates/{id}/generate` | WRITE | Document |
| `GET /documents/by-entity` | READ | Document |

---

## 6. REST API

**Base Path:** `/api/crm/doc/v1`

**Authentication:** OAuth2/JWT (Bearer token)

### 6.1 Endpoint Summary

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/documents/upload` | POST upload |
| `GET` | `/documents` | GET documents |
| `GET` | `/documents/{id}` | GET {id} |
| `GET` | `/documents/{id}/download` | GET download |
| `DELETE` | `/documents/{id}` | DELETE {id} |
| `POST` | `/document-templates` | POST document-templates |
| `GET` | `/document-templates` | GET document-templates |
| `POST` | `/document-templates/{id}/generate` | POST generate |
| `GET` | `/documents/by-entity` | GET by-entity |

> Full request/response examples in companion OpenAPI: `contracts/http/crm/doc/openapi.yaml`

---

## 7. Events & Integration

### 7.1 Published Events

| Event | Routing Key |
|-------|------------|
| DocumentUploaded | `crm.doc.document.uploaded` |
| DocumentGenerated | `crm.doc.document.generated` |
| DocumentDeleted | `crm.doc.document.deleted` |

### 7.2 Consumed Events

This service consumes events from other CRM services as needed for its bounded context. See Suite Spec SS5 for the full event catalog.

---

## 8. Data Model

**Storage:** PostgreSQL 16, Schema: `crm_document`

Tables are derived from the aggregate model in §3. Each aggregate root maps to a primary table. Embedded value objects are stored as columns or JSONB. All tables include `tenant_id` for Row-Level Security.

Detailed DDL will be provided in the implementation phase. Key tables:

- `documents` — primary table for Document aggregate
- `documenttemplates` — primary table for DocumentTemplate aggregate
- `documentlinks` — primary table for DocumentLink aggregate

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
| `F-CRM-009-01` | Document Library |
| `F-CRM-009-02` | Document Upload & Attach |
| `F-CRM-009-03` | Document Template Generator |

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

- OpenAPI: `contracts/http/crm/doc/openapi.yaml`
- Event Schemas: `contracts/events/crm/doc/*.schema.json`