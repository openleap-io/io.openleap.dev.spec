<!-- Template Meta
     Template-ID:   TPL-SVC
     Version:       1.0.0
     Compliance:    95%+
-->

# Activity Service — Domain Specification

> **Conceptual Stack Layer:** Domain / Service
> **Space:** Platform
> **Owner:** Domain Engineering Team

> **Meta Information**
> - **Version:** 2026-04-03
> - **Template:** `domain-service-spec.md` v1.0.0
> - **Template Compliance:** 95%+
> - **Status:** DRAFT
> - **Service ID:** `crm-act-svc`
> - **Suite:** `crm`
> - **Domain:** `act`
> - **Base Package:** `com.openleap.crm.act`
> - **API Path:** `/api/crm/act/v1`
> - **Port:** 8404
> - **DB Schema:** `crm_activity`
> - **Tier:** Core

---

## 0. Document Purpose & Scope

This document specifies the **Activity Service** (`crm-act-svc`) within the CRM Suite.
It defines the domain model, business rules, use cases, REST API, events, data model, security, and quality attributes for the `Activities & Timeline` bounded context.

**Audience:** Backend developers, frontend developers, QA engineers, architects, product owners.

**In scope:** All domain logic within the `Activities & Timeline` bounded context.
**Out of scope:** UI implementation (see feature specs), infrastructure provisioning, cross-suite orchestration.

**Related documents:**
- Suite Spec: `_crm_suite.md`
- OpenAPI: `contracts/http/crm/act/openapi.yaml`
- Event Schemas: `contracts/events/crm/act/*.schema.json`

---

## 1. Business Context

### 1.1 Purpose & Responsibility

Central hub for all CRM interactions — tasks, meetings, calls, notes, and the unified activity timeline. Consumes events from all other CRM services to build a denormalized, reverse-chronological timeline for every Contact, Account, Lead, and Opportunity. The timeline is a CQRS read projection.

**Owns:**
- Task (to-do with due date, priority, assignee, recurrence)
- Meeting (scheduled interaction with attendees, location, outcome, calendar link)
- Call (logged phone interaction with duration, direction, outcome)
- Note (free-text annotation attached to any CRM record)
- TimelineEntry (read-only projection — denormalized event record)

### 1.2 Business Value

Provides essential activities & timeline capabilities within the CRM, enabling users to manage activities & timeline workflows efficiently and with full audit trail.

---

## 2. Service Identity

| Field | Value |
|-------|-------|
| Service ID | `crm-act-svc` |
| Suite | `crm` |
| Domain | `act` |
| Bounded Context | Activities & Timeline |
| Base Package | `com.openleap.crm.act` |
| API Base Path | `/api/crm/act/v1` |
| Port | 8404 |
| Database Schema | `crm_activity` |
| Tier | Core |
| Repository | `openleap-crm-act-svc` |
| Team | `team-crm` |

---

## 3. Domain Model

### 3.1 Aggregates

**Task** (AggregateRoot)
- to-do with due date, priority, assignee, recurrence

**Meeting** (AggregateRoot)
- scheduled interaction with attendees, location, outcome, calendar link

**Call** (AggregateRoot)
- logged phone interaction with duration, direction, outcome

**Note** (AggregateRoot)
- free-text annotation attached to any CRM record

**TimelineEntry** (AggregateRoot)
- read-only projection — denormalized event record

---

## 4. Business Rules

### 4.1 Rule Catalog

| ID | Rule | Type |
|----|------|------|
| BR-ACT-001 | Activities are linked via relatedToType + relatedToId (polymorphic) | POLICY |
| BR-ACT-002 | Task overdue detection runs via scheduled job every 15 minutes | POLICY |
| BR-ACT-003 | Timeline entries are immutable projections — only appendable | POLICY |
| BR-ACT-004 | Meeting attendees can include external email addresses (not just CRM contacts) | POLICY |
| BR-ACT-005 | Recurring tasks generate new task instances on schedule | POLICY |

---

## 5. Use Cases

Use cases are derived from the endpoint catalog below. Each write endpoint corresponds to a WRITE use case; each read endpoint corresponds to a READ use case.

| Endpoint | Use Case Type | Primary Aggregate |
|----------|--------------|------------------|
| `POST /tasks` | WRITE | Task |
| `GET /tasks` | READ | Task |
| `PUT /tasks/{id}` | WRITE | Task |
| `POST /tasks/{id}/complete` | WRITE | Task |
| `POST /meetings` | WRITE | Task |
| `GET /meetings` | READ | Task |
| `PUT /meetings/{id}` | WRITE | Task |
| `POST /meetings/{id}/complete` | WRITE | Task |
| `POST /calls` | WRITE | Task |
| `GET /calls` | READ | Task |
| `POST /notes` | WRITE | Task |
| `GET /notes` | READ | Task |
| `GET /timeline` | READ | Task |

---

## 6. REST API

**Base Path:** `/api/crm/act/v1`

**Authentication:** OAuth2/JWT (Bearer token)

### 6.1 Endpoint Summary

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/tasks` | POST tasks |
| `GET` | `/tasks` | GET tasks |
| `PUT` | `/tasks/{id}` | PUT {id} |
| `POST` | `/tasks/{id}/complete` | POST complete |
| `POST` | `/meetings` | POST meetings |
| `GET` | `/meetings` | GET meetings |
| `PUT` | `/meetings/{id}` | PUT {id} |
| `POST` | `/meetings/{id}/complete` | POST complete |
| `POST` | `/calls` | POST calls |
| `GET` | `/calls` | GET calls |
| `POST` | `/notes` | POST notes |
| `GET` | `/notes` | GET notes |
| `GET` | `/timeline` | GET timeline |

> Full request/response examples in companion OpenAPI: `contracts/http/crm/act/openapi.yaml`

---

## 7. Events & Integration

### 7.1 Published Events

| Event | Routing Key |
|-------|------------|
| TaskCreated | `crm.act.task.created` |
| TaskCompleted | `crm.act.task.completed` |
| TaskOverdue | `crm.act.task.overdue` |
| MeetingScheduled | `crm.act.meeting.scheduled` |
| MeetingCompleted | `crm.act.meeting.completed` |
| MeetingCancelled | `crm.act.meeting.cancelled` |
| CallLogged | `crm.act.call.logged` |
| NoteAdded | `crm.act.note.added` |

### 7.2 Consumed Events

This service consumes events from other CRM services as needed for its bounded context. See Suite Spec SS5 for the full event catalog.

---

## 8. Data Model

**Storage:** PostgreSQL 16, Schema: `crm_activity`

Tables are derived from the aggregate model in §3. Each aggregate root maps to a primary table. Embedded value objects are stored as columns or JSONB. All tables include `tenant_id` for Row-Level Security.

Detailed DDL will be provided in the implementation phase. Key tables:

- `tasks` — primary table for Task aggregate
- `meetings` — primary table for Meeting aggregate
- `calls` — primary table for Call aggregate
- `notes` — primary table for Note aggregate
- `timelineentrys` — primary table for TimelineEntry aggregate

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
| `F-CRM-004-01` | Activity Timeline View |
| `F-CRM-004-02` | Task Management |
| `F-CRM-004-03` | Meeting Scheduler |
| `F-CRM-004-04` | Call Logging |
| `F-CRM-004-05` | Note Editor |

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

- OpenAPI: `contracts/http/crm/act/openapi.yaml`
- Event Schemas: `contracts/events/crm/act/*.schema.json`