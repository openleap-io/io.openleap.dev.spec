<!-- Template Meta
     Template-ID:   TPL-SVC
     Version:       1.0.0
     Compliance:    95%+
-->

# Reporting Service — Domain Specification

> **Conceptual Stack Layer:** Domain / Service
> **Space:** Platform
> **Owner:** Domain Engineering Team

> **Meta Information**
> - **Version:** 2026-04-03
> - **Template:** `domain-service-spec.md` v1.0.0
> - **Template Compliance:** 95%+
> - **Status:** DRAFT
> - **Service ID:** `crm-rpt-svc`
> - **Suite:** `crm`
> - **Domain:** `rpt`
> - **Base Package:** `com.openleap.crm.rpt`
> - **API Path:** `/api/crm/rpt/v1`
> - **Port:** 8414
> - **DB Schema:** `crm_reporting`
> - **Tier:** Extension

---

## 0. Document Purpose & Scope

This document specifies the **Reporting Service** (`crm-rpt-svc`) within the CRM Suite.
It defines the domain model, business rules, use cases, REST API, events, data model, security, and quality attributes for the `Dashboards & Reports` bounded context.

**Audience:** Backend developers, frontend developers, QA engineers, architects, product owners.

**In scope:** All domain logic within the `Dashboards & Reports` bounded context.
**Out of scope:** UI implementation (see feature specs), infrastructure provisioning, cross-suite orchestration.

**Related documents:**
- Suite Spec: `_crm_suite.md`
- OpenAPI: `contracts/http/crm/rpt/openapi.yaml`
- Event Schemas: `contracts/events/crm/rpt/*.schema.json`

---

## 1. Business Context

### 1.1 Purpose & Responsibility

Provides configurable dashboards, standard report library, custom report builder, and scheduled report export. Consumes events from all CRM services to maintain materialized aggregates for fast reporting.

**Owns:**
- Dashboard (user-configured layout of report widgets)
- Report (data query definition with filters, grouping, chart type)
- ReportSchedule (cron-based report execution with delivery via email/webhook)
- ReportExecution (execution history with result cache)

### 1.2 Business Value

Provides essential dashboards & reports capabilities within the CRM, enabling users to manage dashboards & reports workflows efficiently and with full audit trail.

---

## 2. Service Identity

| Field | Value |
|-------|-------|
| Service ID | `crm-rpt-svc` |
| Suite | `crm` |
| Domain | `rpt` |
| Bounded Context | Dashboards & Reports |
| Base Package | `com.openleap.crm.rpt` |
| API Base Path | `/api/crm/rpt/v1` |
| Port | 8414 |
| Database Schema | `crm_reporting` |
| Tier | Extension |
| Repository | `openleap-crm-rpt-svc` |
| Team | `team-crm` |

---

## 3. Domain Model

### 3.1 Aggregates

**Dashboard** (AggregateRoot)
- user-configured layout of report widgets

**Report** (AggregateRoot)
- data query definition with filters, grouping, chart type

**ReportSchedule** (AggregateRoot)
- cron-based report execution with delivery via email/webhook

**ReportExecution** (AggregateRoot)
- execution history with result cache

---

## 4. Business Rules

### 4.1 Rule Catalog

| ID | Rule | Type |
|----|------|------|
| BR-RPT-001 | Standard reports are system-defined and read-only (users can clone to customize) | POLICY |
| BR-RPT-002 | Custom reports query via crm.search Elasticsearch indices | POLICY |
| BR-RPT-003 | Report execution timeout: 30 seconds | POLICY |
| BR-RPT-004 | Export formats: CSV, XLSX, PDF | POLICY |
| BR-RPT-005 | Dashboards support max 12 widgets in a grid layout | POLICY |

---

## 5. Use Cases

Use cases are derived from the endpoint catalog below. Each write endpoint corresponds to a WRITE use case; each read endpoint corresponds to a READ use case.

| Endpoint | Use Case Type | Primary Aggregate |
|----------|--------------|------------------|
| `POST /dashboards` | WRITE | Dashboard |
| `GET /dashboards` | READ | Dashboard |
| `GET /dashboards/{id}` | READ | Dashboard |
| `PUT /dashboards/{id}` | WRITE | Dashboard |
| `POST /reports` | WRITE | Dashboard |
| `GET /reports` | READ | Dashboard |
| `GET /reports/{id}` | READ | Dashboard |
| `POST /reports/{id}/execute` | WRITE | Dashboard |
| `GET /reports/{id}/export` | READ | Dashboard |
| `POST /report-schedules` | WRITE | Dashboard |
| `GET /report-schedules` | READ | Dashboard |

---

## 6. REST API

**Base Path:** `/api/crm/rpt/v1`

**Authentication:** OAuth2/JWT (Bearer token)

### 6.1 Endpoint Summary

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/dashboards` | POST dashboards |
| `GET` | `/dashboards` | GET dashboards |
| `GET` | `/dashboards/{id}` | GET {id} |
| `PUT` | `/dashboards/{id}` | PUT {id} |
| `POST` | `/reports` | POST reports |
| `GET` | `/reports` | GET reports |
| `GET` | `/reports/{id}` | GET {id} |
| `POST` | `/reports/{id}/execute` | POST execute |
| `GET` | `/reports/{id}/export` | GET export |
| `POST` | `/report-schedules` | POST report-schedules |
| `GET` | `/report-schedules` | GET report-schedules |

> Full request/response examples in companion OpenAPI: `contracts/http/crm/rpt/openapi.yaml`

---

## 7. Events & Integration

### 7.1 Published Events

| Event | Routing Key |
|-------|------------|
| ReportGenerated | `crm.rpt.report.generated` |
| DashboardUpdated | `crm.rpt.dashboard.updated` |

### 7.2 Consumed Events

This service consumes events from other CRM services as needed for its bounded context. See Suite Spec SS5 for the full event catalog.

---

## 8. Data Model

**Storage:** PostgreSQL 16, Schema: `crm_reporting`

Tables are derived from the aggregate model in §3. Each aggregate root maps to a primary table. Embedded value objects are stored as columns or JSONB. All tables include `tenant_id` for Row-Level Security.

Detailed DDL will be provided in the implementation phase. Key tables:

- `dashboards` — primary table for Dashboard aggregate
- `reports` — primary table for Report aggregate
- `reportschedules` — primary table for ReportSchedule aggregate
- `reportexecutions` — primary table for ReportExecution aggregate

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
| `F-CRM-010-01` | Dashboard Builder |
| `F-CRM-010-02` | Standard Report Library |
| `F-CRM-010-03` | Custom Report Builder |
| `F-CRM-010-04` | Report Scheduling & Export |

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

- OpenAPI: `contracts/http/crm/rpt/openapi.yaml`
- Event Schemas: `contracts/events/crm/rpt/*.schema.json`