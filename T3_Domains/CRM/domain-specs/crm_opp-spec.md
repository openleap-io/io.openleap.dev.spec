<!-- Template Meta
     Template-ID:   TPL-SVC
     Version:       1.0.0
     Compliance:    95%+
-->

# Opportunity Service — Domain Specification

> **Conceptual Stack Layer:** Domain / Service
> **Space:** Platform
> **Owner:** Domain Engineering Team

> **Meta Information**
> - **Version:** 2026-04-03
> - **Template:** `domain-service-spec.md` v1.0.0
> - **Template Compliance:** 95%+
> - **Status:** DRAFT
> - **Service ID:** `crm-opp-svc`
> - **Suite:** `crm`
> - **Domain:** `opp`
> - **Base Package:** `com.openleap.crm.opp`
> - **API Path:** `/api/crm/opp/v1`
> - **Port:** 8403
> - **DB Schema:** `crm_opportunity`
> - **Tier:** Core

---

## 0. Document Purpose & Scope

This document specifies the **Opportunity Service** (`crm-opp-svc`) within the CRM Suite.
It defines the domain model, business rules, use cases, REST API, events, data model, security, and quality attributes for the `Sales Pipeline` bounded context.

**Audience:** Backend developers, frontend developers, QA engineers, architects, product owners.

**In scope:** All domain logic within the `Sales Pipeline` bounded context.
**Out of scope:** UI implementation (see feature specs), infrastructure provisioning, cross-suite orchestration.

**Related documents:**
- Suite Spec: `_crm_suite.md`
- OpenAPI: `contracts/http/crm/opp/openapi.yaml`
- Event Schemas: `contracts/events/crm/opp/*.schema.json`

---

## 1. Business Context

### 1.1 Purpose & Responsibility

Manages the sales pipeline — opportunities from creation through stage progression to close (won or lost). Owns opportunity lifecycle, line items, stage configuration, and win/loss tracking. Provides forecast data to the analytics service.

**Owns:**
- Opportunity (deal with stages, line items, forecast value)
- OpportunityLineItem (product + quantity + price per deal)
- PipelineConfig (stage sequence, probabilities, entry criteria per tenant)
- StageHistory (audit trail of stage transitions)

### 1.2 Business Value

Provides essential sales pipeline capabilities within the CRM, enabling users to manage sales pipeline workflows efficiently and with full audit trail.

---

## 2. Service Identity

| Field | Value |
|-------|-------|
| Service ID | `crm-opp-svc` |
| Suite | `crm` |
| Domain | `opp` |
| Bounded Context | Sales Pipeline |
| Base Package | `com.openleap.crm.opp` |
| API Base Path | `/api/crm/opp/v1` |
| Port | 8403 |
| Database Schema | `crm_opportunity` |
| Tier | Core |
| Repository | `openleap-crm-opp-svc` |
| Team | `team-crm` |

---

## 3. Domain Model

### 3.1 Aggregates

**Opportunity** (AggregateRoot)
- deal with stages, line items, forecast value

**OpportunityLineItem** (AggregateRoot)
- product + quantity + price per deal

**PipelineConfig** (AggregateRoot)
- stage sequence, probabilities, entry criteria per tenant

**StageHistory** (AggregateRoot)
- audit trail of stage transitions

---

## 4. Business Rules

### 4.1 Rule Catalog

| ID | Rule | Type |
|----|------|------|
| BR-OPP-001 | Stage progression must follow configured sequence (no skipping unless admin override) | CONSTRAINT |
| BR-OPP-002 | Closed Won/Lost are terminal — reopening creates a new opportunity | POLICY |
| BR-OPP-003 | Line items require valid productId from crm.prod | POLICY |
| BR-OPP-004 | Opportunity amount = sum of line item totals | POLICY |
| BR-OPP-005 | Probability auto-set from stage config but can be overridden | POLICY |

---

## 5. Use Cases

Use cases are derived from the endpoint catalog below. Each write endpoint corresponds to a WRITE use case; each read endpoint corresponds to a READ use case.

| Endpoint | Use Case Type | Primary Aggregate |
|----------|--------------|------------------|
| `POST /opportunities` | WRITE | Opportunity |
| `GET /opportunities` | READ | Opportunity |
| `GET /opportunities/{id}` | READ | Opportunity |
| `PUT /opportunities/{id}` | WRITE | Opportunity |
| `POST /opportunities/{id}/stage` | WRITE | Opportunity |
| `POST /opportunities/{id}/line-items` | WRITE | Opportunity |
| `DELETE /opportunities/{id}/line-items/{itemId}` | WRITE | Opportunity |
| `GET /opportunities/{id}/stage-history` | READ | Opportunity |
| `GET /pipeline-config` | READ | Opportunity |
| `PUT /pipeline-config` | WRITE | Opportunity |

---

## 6. REST API

**Base Path:** `/api/crm/opp/v1`

**Authentication:** OAuth2/JWT (Bearer token)

### 6.1 Endpoint Summary

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/opportunities` | POST opportunities |
| `GET` | `/opportunities` | GET opportunities |
| `GET` | `/opportunities/{id}` | GET {id} |
| `PUT` | `/opportunities/{id}` | PUT {id} |
| `POST` | `/opportunities/{id}/stage` | POST stage |
| `POST` | `/opportunities/{id}/line-items` | POST line-items |
| `DELETE` | `/opportunities/{id}/line-items/{itemId}` | DELETE {itemId} |
| `GET` | `/opportunities/{id}/stage-history` | GET stage-history |
| `GET` | `/pipeline-config` | GET pipeline-config |
| `PUT` | `/pipeline-config` | PUT pipeline-config |

> Full request/response examples in companion OpenAPI: `contracts/http/crm/opp/openapi.yaml`

---

## 7. Events & Integration

### 7.1 Published Events

| Event | Routing Key |
|-------|------------|
| OpportunityCreated | `crm.opp.opportunity.created` |
| OpportunityUpdated | `crm.opp.opportunity.updated` |
| OpportunityStageChanged | `crm.opp.opportunity_stage.changed` |
| OpportunityClosedWon | `crm.opp.opportunity_closed.won` |
| OpportunityClosedLost | `crm.opp.opportunity_closed.lost` |
| OpportunityAmountChanged | `crm.opp.opportunity_amount.changed` |
| LineItemAdded | `crm.opp.line_item.added` |
| LineItemRemoved | `crm.opp.line_item.removed` |

### 7.2 Consumed Events

This service consumes events from other CRM services as needed for its bounded context. See Suite Spec SS5 for the full event catalog.

---

## 8. Data Model

**Storage:** PostgreSQL 16, Schema: `crm_opportunity`

Tables are derived from the aggregate model in §3. Each aggregate root maps to a primary table. Embedded value objects are stored as columns or JSONB. All tables include `tenant_id` for Row-Level Security.

Detailed DDL will be provided in the implementation phase. Key tables:

- `opportunitys` — primary table for Opportunity aggregate
- `opportunitylineitems` — primary table for OpportunityLineItem aggregate
- `pipelineconfigs` — primary table for PipelineConfig aggregate
- `stagehistorys` — primary table for StageHistory aggregate

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
| `F-CRM-003-01` | Opportunity List & Kanban |
| `F-CRM-003-02` | Opportunity Detail View |
| `F-CRM-003-03` | Opportunity Create/Edit Form |
| `F-CRM-003-04` | Opportunity Line Items |
| `F-CRM-003-05` | Stage Progression & Guidance |
| `F-CRM-003-06` | Win/Loss Analysis Dashboard |

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

- OpenAPI: `contracts/http/crm/opp/openapi.yaml`
- Event Schemas: `contracts/events/crm/opp/*.schema.json`