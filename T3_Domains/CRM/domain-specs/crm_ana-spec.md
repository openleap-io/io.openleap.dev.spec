<!-- Template Meta
     Template-ID:   TPL-SVC
     Version:       1.0.0
     Compliance:    95%+
-->

# Analytics Service — Domain Specification

> **Conceptual Stack Layer:** Domain / Service
> **Space:** Platform
> **Owner:** Domain Engineering Team

> **Meta Information**
> - **Version:** 2026-04-03
> - **Template:** `domain-service-spec.md` v1.0.0
> - **Template Compliance:** 95%+
> - **Status:** DRAFT
> - **Service ID:** `crm-ana-svc`
> - **Suite:** `crm`
> - **Domain:** `ana`
> - **Base Package:** `com.openleap.crm.ana`
> - **API Path:** `/api/crm/ana/v1`
> - **Port:** 8418
> - **DB Schema:** `crm_analytics`
> - **Tier:** Extension

---

## 0. Document Purpose & Scope

This document specifies the **Analytics Service** (`crm-ana-svc`) within the CRM Suite.
It defines the domain model, business rules, use cases, REST API, events, data model, security, and quality attributes for the `BI & Predictive Analytics` bounded context.

**Audience:** Backend developers, frontend developers, QA engineers, architects, product owners.

**In scope:** All domain logic within the `BI & Predictive Analytics` bounded context.
**Out of scope:** UI implementation (see feature specs), infrastructure provisioning, cross-suite orchestration.

**Related documents:**
- Suite Spec: `_crm_suite.md`
- OpenAPI: `contracts/http/crm/ana/openapi.yaml`
- Event Schemas: `contracts/events/crm/ana/*.schema.json`

---

## 1. Business Context

### 1.1 Purpose & Responsibility

Business intelligence and predictive analytics for CRM. Maintains materialized aggregates for sales forecasting, pipeline analytics, activity analytics, and predictive lead scoring. Consumes all CRM events to build analytical models.

**Owns:**
- ForecastPeriod (time-boxed revenue forecast with actual vs projected, per team/rep)
- PipelineSnapshot (point-in-time pipeline state for trend analysis)
- ActivityMetrics (aggregated activity counts per user/team/period)
- ScoringModel (ML model metadata — training date, accuracy, feature importance)

### 1.2 Business Value

Provides essential bi & predictive analytics capabilities within the CRM, enabling users to manage bi & predictive analytics workflows efficiently and with full audit trail.

---

## 2. Service Identity

| Field | Value |
|-------|-------|
| Service ID | `crm-ana-svc` |
| Suite | `crm` |
| Domain | `ana` |
| Bounded Context | BI & Predictive Analytics |
| Base Package | `com.openleap.crm.ana` |
| API Base Path | `/api/crm/ana/v1` |
| Port | 8418 |
| Database Schema | `crm_analytics` |
| Tier | Extension |
| Repository | `openleap-crm-ana-svc` |
| Team | `team-crm` |

---

## 3. Domain Model

### 3.1 Aggregates

**ForecastPeriod** (AggregateRoot)
- time-boxed revenue forecast with actual vs projected, per team/rep

**PipelineSnapshot** (AggregateRoot)
- point-in-time pipeline state for trend analysis

**ActivityMetrics** (AggregateRoot)
- aggregated activity counts per user/team/period

**ScoringModel** (AggregateRoot)
- ML model metadata — training date, accuracy, feature importance

---

## 4. Business Rules

### 4.1 Rule Catalog

| ID | Rule | Type |
|----|------|------|
| BR-ANA-001 | Forecasts recalculate nightly and on-demand | POLICY |
| BR-ANA-002 | Pipeline snapshots taken daily at midnight UTC | POLICY |
| BR-ANA-003 | Activity analytics aggregated hourly | POLICY |
| BR-ANA-004 | Predictive scoring model retrained weekly with last 6 months of data | POLICY |
| BR-ANA-005 | Forecast = sum of (opportunity amount × stage probability) per period | POLICY |

---

## 5. Use Cases

Use cases are derived from the endpoint catalog below. Each write endpoint corresponds to a WRITE use case; each read endpoint corresponds to a READ use case.

| Endpoint | Use Case Type | Primary Aggregate |
|----------|--------------|------------------|
| `GET /forecasts` | READ | ForecastPeriod |
| `GET /forecasts/{periodId}` | READ | ForecastPeriod |
| `POST /forecasts/recalculate` | WRITE | ForecastPeriod |
| `GET /pipeline-analytics` | READ | ForecastPeriod |
| `GET /pipeline-analytics/trend` | READ | ForecastPeriod |
| `GET /activity-analytics` | READ | ForecastPeriod |
| `GET /activity-analytics/leaderboard` | READ | ForecastPeriod |
| `GET /scoring-models` | READ | ForecastPeriod |
| `POST /scoring-models/train` | WRITE | ForecastPeriod |

---

## 6. REST API

**Base Path:** `/api/crm/ana/v1`

**Authentication:** OAuth2/JWT (Bearer token)

### 6.1 Endpoint Summary

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/forecasts` | GET forecasts |
| `GET` | `/forecasts/{periodId}` | GET {periodId} |
| `POST` | `/forecasts/recalculate` | POST recalculate |
| `GET` | `/pipeline-analytics` | GET pipeline-analytics |
| `GET` | `/pipeline-analytics/trend` | GET trend |
| `GET` | `/activity-analytics` | GET activity-analytics |
| `GET` | `/activity-analytics/leaderboard` | GET leaderboard |
| `GET` | `/scoring-models` | GET scoring-models |
| `POST` | `/scoring-models/train` | POST train |

> Full request/response examples in companion OpenAPI: `contracts/http/crm/ana/openapi.yaml`

---

## 7. Events & Integration

### 7.1 Published Events

| Event | Routing Key |
|-------|------------|
| ForecastRecalculated | `crm.ana.forecast.recalculated` |
| ScoringModelTrained | `crm.ana.scoring_model.trained` |

### 7.2 Consumed Events

This service consumes events from other CRM services as needed for its bounded context. See Suite Spec SS5 for the full event catalog.

---

## 8. Data Model

**Storage:** PostgreSQL 16, Schema: `crm_analytics`

Tables are derived from the aggregate model in §3. Each aggregate root maps to a primary table. Embedded value objects are stored as columns or JSONB. All tables include `tenant_id` for Row-Level Security.

Detailed DDL will be provided in the implementation phase. Key tables:

- `forecastperiods` — primary table for ForecastPeriod aggregate
- `pipelinesnapshots` — primary table for PipelineSnapshot aggregate
- `activitymetrics` — primary table for ActivityMetrics aggregate
- `scoringmodels` — primary table for ScoringModel aggregate

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
| `F-CRM-014-01` | Sales Forecast Dashboard |
| `F-CRM-014-02` | Pipeline Analytics |
| `F-CRM-014-03` | Activity Analytics |
| `F-CRM-014-04` | Predictive Scoring Setup |

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

- OpenAPI: `contracts/http/crm/ana/openapi.yaml`
- Event Schemas: `contracts/events/crm/ana/*.schema.json`