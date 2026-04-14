<!-- Template Meta
     Template-ID:   TPL-SVC
     Version:       1.0.0
     Compliance:    95%+
-->

# Marketing Service — Domain Specification

> **Conceptual Stack Layer:** Domain / Service
> **Space:** Platform
> **Owner:** Domain Engineering Team

> **Meta Information**
> - **Version:** 2026-04-03
> - **Template:** `domain-service-spec.md` v1.0.0
> - **Template Compliance:** 95%+
> - **Status:** DRAFT
> - **Service ID:** `crm-mkt-svc`
> - **Suite:** `crm`
> - **Domain:** `mkt`
> - **Base Package:** `com.openleap.crm.mkt`
> - **API Path:** `/api/crm/mkt/v1`
> - **Port:** 8419
> - **DB Schema:** `crm_marketing`
> - **Tier:** Extension

---

## 0. Document Purpose & Scope

This document specifies the **Marketing Service** (`crm-mkt-svc`) within the CRM Suite.
It defines the domain model, business rules, use cases, REST API, events, data model, security, and quality attributes for the `Campaigns & Lead Nurturing` bounded context.

**Audience:** Backend developers, frontend developers, QA engineers, architects, product owners.

**In scope:** All domain logic within the `Campaigns & Lead Nurturing` bounded context.
**Out of scope:** UI implementation (see feature specs), infrastructure provisioning, cross-suite orchestration.

**Related documents:**
- Suite Spec: `_crm_suite.md`
- OpenAPI: `contracts/http/crm/mkt/openapi.yaml`
- Event Schemas: `contracts/events/crm/mkt/*.schema.json`

---

## 1. Business Context

### 1.1 Purpose & Responsibility

Marketing campaign management — create campaigns, add members (contacts and leads), execute email campaigns, manage lead nurturing workflows, and track campaign performance and ROI.

**Owns:**
- Campaign (marketing initiative with type, dates, budget, member list)
- CampaignMember (contact or lead enrolled in a campaign with response status)
- NurturingSequence (multi-step email drip sequence for lead nurturing)
- NurturingStep (individual email in a nurturing sequence with delay and conditions)

### 1.2 Business Value

Provides essential campaigns & lead nurturing capabilities within the CRM, enabling users to manage campaigns & lead nurturing workflows efficiently and with full audit trail.

---

## 2. Service Identity

| Field | Value |
|-------|-------|
| Service ID | `crm-mkt-svc` |
| Suite | `crm` |
| Domain | `mkt` |
| Bounded Context | Campaigns & Lead Nurturing |
| Base Package | `com.openleap.crm.mkt` |
| API Base Path | `/api/crm/mkt/v1` |
| Port | 8419 |
| Database Schema | `crm_marketing` |
| Tier | Extension |
| Repository | `openleap-crm-mkt-svc` |
| Team | `team-crm` |

---

## 3. Domain Model

### 3.1 Aggregates

**Campaign** (AggregateRoot)
- marketing initiative with type, dates, budget, member list

**CampaignMember** (AggregateRoot)
- contact or lead enrolled in a campaign with response status

**NurturingSequence** (AggregateRoot)
- multi-step email drip sequence for lead nurturing

**NurturingStep** (AggregateRoot)
- individual email in a nurturing sequence with delay and conditions

---

## 4. Business Rules

### 4.1 Rule Catalog

| ID | Rule | Type |
|----|------|------|
| BR-MKT-001 | Campaign launch validates that all members have valid email addresses | POLICY |
| BR-MKT-002 | Members with doNotEmail=true are excluded from email campaigns | POLICY |
| BR-MKT-003 | Campaign ROI = (revenue from converted campaign leads - campaign cost) / campaign cost | POLICY |
| BR-MKT-004 | Nurturing sequences pause when lead status changes to QUALIFIED or CONVERTED | POLICY |
| BR-MKT-005 | A/B testing: max 3 variants per nurturing step | POLICY |

---

## 5. Use Cases

Use cases are derived from the endpoint catalog below. Each write endpoint corresponds to a WRITE use case; each read endpoint corresponds to a READ use case.

| Endpoint | Use Case Type | Primary Aggregate |
|----------|--------------|------------------|
| `POST /campaigns` | WRITE | Campaign |
| `GET /campaigns` | READ | Campaign |
| `GET /campaigns/{id}` | READ | Campaign |
| `PUT /campaigns/{id}` | WRITE | Campaign |
| `POST /campaigns/{id}/launch` | WRITE | Campaign |
| `POST /campaigns/{id}/members` | WRITE | Campaign |
| `GET /campaigns/{id}/members` | READ | Campaign |
| `GET /campaigns/{id}/performance` | READ | Campaign |
| `POST /nurturing-sequences` | WRITE | Campaign |
| `GET /nurturing-sequences` | READ | Campaign |
| `POST /nurturing-sequences/{id}/activate` | WRITE | Campaign |

---

## 6. REST API

**Base Path:** `/api/crm/mkt/v1`

**Authentication:** OAuth2/JWT (Bearer token)

### 6.1 Endpoint Summary

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/campaigns` | POST campaigns |
| `GET` | `/campaigns` | GET campaigns |
| `GET` | `/campaigns/{id}` | GET {id} |
| `PUT` | `/campaigns/{id}` | PUT {id} |
| `POST` | `/campaigns/{id}/launch` | POST launch |
| `POST` | `/campaigns/{id}/members` | POST members |
| `GET` | `/campaigns/{id}/members` | GET members |
| `GET` | `/campaigns/{id}/performance` | GET performance |
| `POST` | `/nurturing-sequences` | POST nurturing-sequences |
| `GET` | `/nurturing-sequences` | GET nurturing-sequences |
| `POST` | `/nurturing-sequences/{id}/activate` | POST activate |

> Full request/response examples in companion OpenAPI: `contracts/http/crm/mkt/openapi.yaml`

---

## 7. Events & Integration

### 7.1 Published Events

| Event | Routing Key |
|-------|------------|
| CampaignCreated | `crm.mkt.campaign.created` |
| CampaignLaunched | `crm.mkt.campaign.launched` |
| CampaignCompleted | `crm.mkt.campaign.completed` |
| MemberResponded | `crm.mkt.member.responded` |

### 7.2 Consumed Events

This service consumes events from other CRM services as needed for its bounded context. See Suite Spec SS5 for the full event catalog.

---

## 8. Data Model

**Storage:** PostgreSQL 16, Schema: `crm_marketing`

Tables are derived from the aggregate model in §3. Each aggregate root maps to a primary table. Embedded value objects are stored as columns or JSONB. All tables include `tenant_id` for Row-Level Security.

Detailed DDL will be provided in the implementation phase. Key tables:

- `campaigns` — primary table for Campaign aggregate
- `campaignmembers` — primary table for CampaignMember aggregate
- `nurturingsequences` — primary table for NurturingSequence aggregate
- `nurturingsteps` — primary table for NurturingStep aggregate

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
| `F-CRM-015-01` | Campaign List & Manager |
| `F-CRM-015-02` | Campaign Builder |
| `F-CRM-015-03` | Campaign Execution Monitor |
| `F-CRM-015-04` | Lead Nurturing Workflows |
| `F-CRM-015-05` | Campaign Performance Dashboard |

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

- OpenAPI: `contracts/http/crm/mkt/openapi.yaml`
- Event Schemas: `contracts/events/crm/mkt/*.schema.json`