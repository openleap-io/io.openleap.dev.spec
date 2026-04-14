<!-- Template Meta
     Template-ID:   TPL-SVC
     Version:       1.0.0
     Compliance:    95%+
-->

# Calendar Service — Domain Specification

> **Conceptual Stack Layer:** Domain / Service
> **Space:** Platform
> **Owner:** Domain Engineering Team

> **Meta Information**
> - **Version:** 2026-04-03
> - **Template:** `domain-service-spec.md` v1.0.0
> - **Template Compliance:** 95%+
> - **Status:** DRAFT
> - **Service ID:** `crm-cal-svc`
> - **Suite:** `crm`
> - **Domain:** `cal`
> - **Base Package:** `com.openleap.crm.cal`
> - **API Path:** `/api/crm/cal/v1`
> - **Port:** 8412
> - **DB Schema:** `crm_calendar`
> - **Tier:** Extension

---

## 0. Document Purpose & Scope

This document specifies the **Calendar Service** (`crm-cal-svc`) within the CRM Suite.
It defines the domain model, business rules, use cases, REST API, events, data model, security, and quality attributes for the `Scheduling & Calendar` bounded context.

**Audience:** Backend developers, frontend developers, QA engineers, architects, product owners.

**In scope:** All domain logic within the `Scheduling & Calendar` bounded context.
**Out of scope:** UI implementation (see feature specs), infrastructure provisioning, cross-suite orchestration.

**Related documents:**
- Suite Spec: `_crm_suite.md`
- OpenAPI: `contracts/http/crm/cal/openapi.yaml`
- Event Schemas: `contracts/events/crm/cal/*.schema.json`

---

## 1. Business Context

### 1.1 Purpose & Responsibility

Provides bidirectional calendar synchronization with external providers (Google Calendar, Microsoft Outlook, CalDAV), scheduling features including availability checking, booking links for external parties, and meeting slot suggestions.

**Owns:**
- CalendarConnection (OAuth link to external calendar provider)
- SyncState (last sync timestamp, cursor, error state per connection)
- BookingLink (shareable scheduling URL with availability rules)
- AvailabilitySlot (computed free/busy slots for a user)

### 1.2 Business Value

Provides essential scheduling & calendar capabilities within the CRM, enabling users to manage scheduling & calendar workflows efficiently and with full audit trail.

---

## 2. Service Identity

| Field | Value |
|-------|-------|
| Service ID | `crm-cal-svc` |
| Suite | `crm` |
| Domain | `cal` |
| Bounded Context | Scheduling & Calendar |
| Base Package | `com.openleap.crm.cal` |
| API Base Path | `/api/crm/cal/v1` |
| Port | 8412 |
| Database Schema | `crm_calendar` |
| Tier | Extension |
| Repository | `openleap-crm-cal-svc` |
| Team | `team-crm` |

---

## 3. Domain Model

### 3.1 Aggregates

**CalendarConnection** (AggregateRoot)
- OAuth link to external calendar provider

**SyncState** (AggregateRoot)
- last sync timestamp, cursor, error state per connection

**BookingLink** (AggregateRoot)
- shareable scheduling URL with availability rules

**AvailabilitySlot** (AggregateRoot)
- computed free/busy slots for a user

---

## 4. Business Rules

### 4.1 Rule Catalog

| ID | Rule | Type |
|----|------|------|
| BR-CAL-001 | OAuth tokens stored encrypted at rest | POLICY |
| BR-CAL-002 | Sync conflict resolution: external calendar wins for time/location, CRM wins for CRM-specific metadata | POLICY |
| BR-CAL-003 | Booking links enforce minimum notice period and buffer between meetings | POLICY |
| BR-CAL-004 | Availability computation merges all connected calendars | POLICY |

---

## 5. Use Cases

Use cases are derived from the endpoint catalog below. Each write endpoint corresponds to a WRITE use case; each read endpoint corresponds to a READ use case.

| Endpoint | Use Case Type | Primary Aggregate |
|----------|--------------|------------------|
| `POST /calendar-connections` | WRITE | CalendarConnection |
| `GET /calendar-connections` | READ | CalendarConnection |
| `DELETE /calendar-connections/{id}` | WRITE | CalendarConnection |
| `POST /calendar-connections/{id}/sync` | WRITE | CalendarConnection |
| `GET /availability` | READ | CalendarConnection |
| `POST /booking-links` | WRITE | CalendarConnection |
| `GET /booking-links` | READ | CalendarConnection |
| `GET /booking-links/{id}` | READ | CalendarConnection |
| `POST /booking-links/{id}/book` | WRITE | CalendarConnection |

---

## 6. REST API

**Base Path:** `/api/crm/cal/v1`

**Authentication:** OAuth2/JWT (Bearer token)

### 6.1 Endpoint Summary

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/calendar-connections` | POST calendar-connections |
| `GET` | `/calendar-connections` | GET calendar-connections |
| `DELETE` | `/calendar-connections/{id}` | DELETE {id} |
| `POST` | `/calendar-connections/{id}/sync` | POST sync |
| `GET` | `/availability` | GET availability |
| `POST` | `/booking-links` | POST booking-links |
| `GET` | `/booking-links` | GET booking-links |
| `GET` | `/booking-links/{id}` | GET {id} |
| `POST` | `/booking-links/{id}/book` | POST book |

> Full request/response examples in companion OpenAPI: `contracts/http/crm/cal/openapi.yaml`

---

## 7. Events & Integration

### 7.1 Published Events

| Event | Routing Key |
|-------|------------|
| CalendarSynced | `crm.cal.calendar.synced` |
| BookingCreated | `crm.cal.booking.created` |
| BookingCancelled | `crm.cal.booking.cancelled` |

### 7.2 Consumed Events

This service consumes events from other CRM services as needed for its bounded context. See Suite Spec SS5 for the full event catalog.

---

## 8. Data Model

**Storage:** PostgreSQL 16, Schema: `crm_calendar`

Tables are derived from the aggregate model in §3. Each aggregate root maps to a primary table. Embedded value objects are stored as columns or JSONB. All tables include `tenant_id` for Row-Level Security.

Detailed DDL will be provided in the implementation phase. Key tables:

- `calendarconnections` — primary table for CalendarConnection aggregate
- `syncstates` — primary table for SyncState aggregate
- `bookinglinks` — primary table for BookingLink aggregate
- `availabilityslots` — primary table for AvailabilitySlot aggregate

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
| `F-CRM-008-01` | Calendar View |
| `F-CRM-008-02` | Calendar Sync Settings |
| `F-CRM-008-03` | Booking Link Manager |
| `F-CRM-008-04` | Availability Configurator |

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

- OpenAPI: `contracts/http/crm/cal/openapi.yaml`
- Event Schemas: `contracts/events/crm/cal/*.schema.json`