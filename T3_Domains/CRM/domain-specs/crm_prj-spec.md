<!-- Template Meta
     Template-ID:   TPL-SVC
     Version:       1.0.0
     Compliance:    95%+
-->

# Project Service — Domain Specification

> **Conceptual Stack Layer:** Domain / Service
> **Space:** Platform
> **Owner:** Domain Engineering Team

> **Meta Information**
> - **Version:** 2026-04-03
> - **Template:** `domain-service-spec.md` v1.0.0
> - **Template Compliance:** 95%+
> - **Status:** DRAFT
> - **Service ID:** `crm-prj-svc`
> - **Suite:** `crm`
> - **Domain:** `prj`
> - **Base Package:** `com.openleap.crm.prj`
> - **API Path:** `/api/crm/prj/v1`
> - **Port:** 8422
> - **DB Schema:** `crm_project`
> - **Tier:** Extension

---

## 0. Document Purpose & Scope

This document specifies the **Project Service** (`crm-prj-svc`) within the CRM Suite.
It defines the domain model, business rules, use cases, REST API, events, data model, security, and quality attributes for the `Customer Projects & Onboarding` bounded context.

**Audience:** Backend developers, frontend developers, QA engineers, architects, product owners.

**In scope:** All domain logic within the `Customer Projects & Onboarding` bounded context.
**Out of scope:** UI implementation (see feature specs), infrastructure provisioning, cross-suite orchestration.

**Related documents:**
- Suite Spec: `_crm_suite.md`
- OpenAPI: `contracts/http/crm/prj/openapi.yaml`
- Event Schemas: `contracts/events/crm/prj/*.schema.json`

---

## 1. Business Context

### 1.1 Purpose & Responsibility

Manages customer-facing projects and onboarding checklists. Tracks post-sale delivery milestones, onboarding progress, and customer project tasks. Linked to accounts and opportunities for end-to-end customer lifecycle visibility.

**Owns:**
- CustomerProject (project with milestones, timeline, status, linked account/opportunity)
- ProjectMilestone (named checkpoint with due date and completion status)
- OnboardingChecklist (template-based checklist for customer onboarding)
- ChecklistItem (individual step in an onboarding checklist)
- ProjectTask (task within a customer project with assignee and due date)

### 1.2 Business Value

Provides essential customer projects & onboarding capabilities within the CRM, enabling users to manage customer projects & onboarding workflows efficiently and with full audit trail.

---

## 2. Service Identity

| Field | Value |
|-------|-------|
| Service ID | `crm-prj-svc` |
| Suite | `crm` |
| Domain | `prj` |
| Bounded Context | Customer Projects & Onboarding |
| Base Package | `com.openleap.crm.prj` |
| API Base Path | `/api/crm/prj/v1` |
| Port | 8422 |
| Database Schema | `crm_project` |
| Tier | Extension |
| Repository | `openleap-crm-prj-svc` |
| Team | `team-crm` |

---

## 3. Domain Model

### 3.1 Aggregates

**CustomerProject** (AggregateRoot)
- project with milestones, timeline, status, linked account/opportunity

**ProjectMilestone** (AggregateRoot)
- named checkpoint with due date and completion status

**OnboardingChecklist** (AggregateRoot)
- template-based checklist for customer onboarding

**ChecklistItem** (AggregateRoot)
- individual step in an onboarding checklist

**ProjectTask** (AggregateRoot)
- task within a customer project with assignee and due date

---

## 4. Business Rules

### 4.1 Rule Catalog

| ID | Rule | Type |
|----|------|------|
| BR-PRJ-001 | Project completion requires all milestones to be marked complete | POLICY |
| BR-PRJ-002 | Onboarding checklists can be created from templates | POLICY |
| BR-PRJ-003 | Project tasks follow same model as crm.act tasks but scoped to project | POLICY |
| BR-PRJ-004 | Projects linked to accounts inherit tenant isolation | POLICY |
| BR-PRJ-005 | Milestone due dates cannot be in the past on creation | CONSTRAINT |

---

## 5. Use Cases

Use cases are derived from the endpoint catalog below. Each write endpoint corresponds to a WRITE use case; each read endpoint corresponds to a READ use case.

| Endpoint | Use Case Type | Primary Aggregate |
|----------|--------------|------------------|
| `POST /projects` | WRITE | CustomerProject |
| `GET /projects` | READ | CustomerProject |
| `GET /projects/{id}` | READ | CustomerProject |
| `PUT /projects/{id}` | WRITE | CustomerProject |
| `POST /projects/{id}/milestones` | WRITE | CustomerProject |
| `POST /projects/{id}/milestones/{mid}/complete` | WRITE | CustomerProject |
| `POST /onboarding-checklists` | WRITE | CustomerProject |
| `GET /onboarding-checklists` | READ | CustomerProject |
| `POST /onboarding-checklists/{id}/items/{itemId}/complete` | WRITE | CustomerProject |
| `POST /projects/{id}/tasks` | WRITE | CustomerProject |
| `GET /projects/{id}/tasks` | READ | CustomerProject |

---

## 6. REST API

**Base Path:** `/api/crm/prj/v1`

**Authentication:** OAuth2/JWT (Bearer token)

### 6.1 Endpoint Summary

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/projects` | POST projects |
| `GET` | `/projects` | GET projects |
| `GET` | `/projects/{id}` | GET {id} |
| `PUT` | `/projects/{id}` | PUT {id} |
| `POST` | `/projects/{id}/milestones` | POST milestones |
| `POST` | `/projects/{id}/milestones/{mid}/complete` | POST complete |
| `POST` | `/onboarding-checklists` | POST onboarding-checklists |
| `GET` | `/onboarding-checklists` | GET onboarding-checklists |
| `POST` | `/onboarding-checklists/{id}/items/{itemId}/complete` | POST complete |
| `POST` | `/projects/{id}/tasks` | POST tasks |
| `GET` | `/projects/{id}/tasks` | GET tasks |

> Full request/response examples in companion OpenAPI: `contracts/http/crm/prj/openapi.yaml`

---

## 7. Events & Integration

### 7.1 Published Events

| Event | Routing Key |
|-------|------------|
| ProjectCreated | `crm.prj.project.created` |
| ProjectCompleted | `crm.prj.project.completed` |
| MilestoneReached | `crm.prj.milestone.reached` |
| ChecklistItemCompleted | `crm.prj.checklist_item.completed` |

### 7.2 Consumed Events

This service consumes events from other CRM services as needed for its bounded context. See Suite Spec SS5 for the full event catalog.

---

## 8. Data Model

**Storage:** PostgreSQL 16, Schema: `crm_project`

Tables are derived from the aggregate model in §3. Each aggregate root maps to a primary table. Embedded value objects are stored as columns or JSONB. All tables include `tenant_id` for Row-Level Security.

Detailed DDL will be provided in the implementation phase. Key tables:

- `customerprojects` — primary table for CustomerProject aggregate
- `projectmilestones` — primary table for ProjectMilestone aggregate
- `onboardingchecklists` — primary table for OnboardingChecklist aggregate
- `checklistitems` — primary table for ChecklistItem aggregate
- `projecttasks` — primary table for ProjectTask aggregate

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
| `F-CRM-018-01` | Project Board & List |
| `F-CRM-018-02` | Project Detail & Timeline |
| `F-CRM-018-03` | Onboarding Checklist |
| `F-CRM-018-04` | Project Task Management |

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

- OpenAPI: `contracts/http/crm/prj/openapi.yaml`
- Event Schemas: `contracts/events/crm/prj/*.schema.json`