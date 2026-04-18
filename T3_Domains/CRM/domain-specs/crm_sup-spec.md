<!-- Template Meta
     Template-ID:   TPL-SVC
     Version:       1.0.0
     Compliance:    95%+
-->

# Support Service — Domain Specification

> **Conceptual Stack Layer:** Domain / Service
> **Space:** Platform
> **Owner:** Domain Engineering Team

> **Meta Information**
> - **Version:** 2026-04-18
> - **Template:** `domain-service-spec.md` v1.0.0
> - **Template Compliance:** 95%+
> - **Status:** DEPRECATED — superseded by `tks-tkt-svc` (tickets + SLA + comments) and `tks-kb-svc` (knowledge). See `T3_Domains/TKS/_tks_suite.md` and §13 Migration below. Routing-key bridge `crm.sup.*` → `tks.tkt.*` active for 60 days; API path `/api/crm/sup/v1` returns HTTP 308 during grace period.
> - **Service ID:** `crm-sup-svc`
> - **Suite:** `crm`
> - **Domain:** `sup`
> - **Base Package:** `com.openleap.crm.sup`
> - **API Path:** `/api/crm/sup/v1`
> - **Port:** 8420
> - **DB Schema:** `crm_support`
> - **Tier:** Extension

---

## 0. Document Purpose & Scope

This document specifies the **Support Service** (`crm-sup-svc`) within the CRM Suite.
It defines the domain model, business rules, use cases, REST API, events, data model, security, and quality attributes for the `Tickets & Cases` bounded context.

**Audience:** Backend developers, frontend developers, QA engineers, architects, product owners.

**In scope:** All domain logic within the `Tickets & Cases` bounded context.
**Out of scope:** UI implementation (see feature specs), infrastructure provisioning, cross-suite orchestration.

**Related documents:**
- Suite Spec: `_crm_suite.md`
- OpenAPI: `contracts/http/crm/sup/openapi.yaml`
- Event Schemas: `contracts/events/crm/sup/*.schema.json`

---

## 1. Business Context

### 1.1 Purpose & Responsibility

Customer support ticket management — ticket creation, assignment, escalation, SLA tracking, and resolution. Includes knowledge base for self-service. Integrates with contacts for reporter identification and with workflow for auto-assignment and escalation.

**Owns:**
- Ticket (support case with priority, category, SLA targets, resolution timeline)
- SLAPolicy (time-based targets per priority — first response, resolution)
- KnowledgeArticle (help article with categories, tags, version history)
- TicketComment (internal or public comment on a ticket)

### 1.2 Business Value

Provides essential tickets & cases capabilities within the CRM, enabling users to manage tickets & cases workflows efficiently and with full audit trail.

---

## 2. Service Identity

| Field | Value |
|-------|-------|
| Service ID | `crm-sup-svc` |
| Suite | `crm` |
| Domain | `sup` |
| Bounded Context | Tickets & Cases |
| Base Package | `com.openleap.crm.sup` |
| API Base Path | `/api/crm/sup/v1` |
| Port | 8420 |
| Database Schema | `crm_support` |
| Tier | Extension |
| Repository | `openleap-crm-sup-svc` |
| Team | `team-crm` |

---

## 3. Domain Model

### 3.1 Aggregates

**Ticket** (AggregateRoot)
- support case with priority, category, SLA targets, resolution timeline

**SLAPolicy** (AggregateRoot)
- time-based targets per priority — first response, resolution

**KnowledgeArticle** (AggregateRoot)
- help article with categories, tags, version history

**TicketComment** (AggregateRoot)
- internal or public comment on a ticket

---

## 4. Business Rules

### 4.1 Rule Catalog

| ID | Rule | Type |
|----|------|------|
| BR-SUP-001 | SLA clock starts at ticket creation, pauses when status=WAITING_ON_CUSTOMER | POLICY |
| BR-SUP-002 | Escalation auto-triggers when SLA breach is imminent (80% of target time elapsed) | POLICY |
| BR-SUP-003 | Ticket reopening resets resolution SLA but not first-response SLA | POLICY |
| BR-SUP-004 | Knowledge articles require review approval before publishing | POLICY |
| BR-SUP-005 | Agent can view all tickets in their queue; customer portal shows only own tickets | POLICY |

---

## 5. Use Cases

Use cases are derived from the endpoint catalog below. Each write endpoint corresponds to a WRITE use case; each read endpoint corresponds to a READ use case.

| Endpoint | Use Case Type | Primary Aggregate |
|----------|--------------|------------------|
| `POST /tickets` | WRITE | Ticket |
| `GET /tickets` | READ | Ticket |
| `GET /tickets/{id}` | READ | Ticket |
| `PUT /tickets/{id}` | WRITE | Ticket |
| `POST /tickets/{id}/assign` | WRITE | Ticket |
| `POST /tickets/{id}/escalate` | WRITE | Ticket |
| `POST /tickets/{id}/resolve` | WRITE | Ticket |
| `POST /tickets/{id}/reopen` | WRITE | Ticket |
| `POST /tickets/{id}/comments` | WRITE | Ticket |
| `GET /tickets/{id}/comments` | READ | Ticket |
| `POST /sla-policies` | WRITE | Ticket |
| `GET /sla-policies` | READ | Ticket |
| `POST /knowledge-articles` | WRITE | Ticket |
| `GET /knowledge-articles` | READ | Ticket |
| `GET /knowledge-articles/search` | READ | Ticket |

---

## 6. REST API

**Base Path:** `/api/crm/sup/v1`

**Authentication:** OAuth2/JWT (Bearer token)

### 6.1 Endpoint Summary

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/tickets` | POST tickets |
| `GET` | `/tickets` | GET tickets |
| `GET` | `/tickets/{id}` | GET {id} |
| `PUT` | `/tickets/{id}` | PUT {id} |
| `POST` | `/tickets/{id}/assign` | POST assign |
| `POST` | `/tickets/{id}/escalate` | POST escalate |
| `POST` | `/tickets/{id}/resolve` | POST resolve |
| `POST` | `/tickets/{id}/reopen` | POST reopen |
| `POST` | `/tickets/{id}/comments` | POST comments |
| `GET` | `/tickets/{id}/comments` | GET comments |
| `POST` | `/sla-policies` | POST sla-policies |
| `GET` | `/sla-policies` | GET sla-policies |
| `POST` | `/knowledge-articles` | POST knowledge-articles |
| `GET` | `/knowledge-articles` | GET knowledge-articles |
| `GET` | `/knowledge-articles/search` | GET search |

> Full request/response examples in companion OpenAPI: `contracts/http/crm/sup/openapi.yaml`

---

## 7. Events & Integration

### 7.1 Published Events

| Event | Routing Key |
|-------|------------|
| TicketCreated | `crm.sup.ticket.created` |
| TicketAssigned | `crm.sup.ticket.assigned` |
| TicketEscalated | `crm.sup.ticket.escalated` |
| TicketResolved | `crm.sup.ticket.resolved` |
| TicketReopened | `crm.sup.ticket.reopened` |
| SLABreached | `crm.sup.s_l_a.breached` |

### 7.2 Consumed Events

This service consumes events from other CRM services as needed for its bounded context. See Suite Spec SS5 for the full event catalog.

---

## 8. Data Model

**Storage:** PostgreSQL 16, Schema: `crm_support`

Tables are derived from the aggregate model in §3. Each aggregate root maps to a primary table. Embedded value objects are stored as columns or JSONB. All tables include `tenant_id` for Row-Level Security.

Detailed DDL will be provided in the implementation phase. Key tables:

- `tickets` — primary table for Ticket aggregate
- `slapolicys` — primary table for SLAPolicy aggregate
- `knowledgearticles` — primary table for KnowledgeArticle aggregate
- `ticketcomments` — primary table for TicketComment aggregate

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
| `F-CRM-016-01` | Ticket List & Queue |
| `F-CRM-016-02` | Ticket Detail & Resolution |
| `F-CRM-016-03` | Ticket Create/Edit |
| `F-CRM-016-04` | SLA Management Dashboard |
| `F-CRM-016-05` | Knowledge Base Browser |

---

## 12. Extension Points

- All domain events are available for extension service consumption
- Primary aggregates support `customFields` (JSONB) for tenant-specific extensions
- Integration Service (`crm.int`) can forward any event to external webhooks

---

## 13. Migration & Evolution

**This service is DEPRECATED.** It is being superseded by two new services in the TKS suite:

- `tks-tkt-svc` — takes over `Ticket`, `TicketComment`, `SLAPolicy` and their events
- `tks-kb-svc` — takes over `KnowledgeArticle` and its events

### 13.1 Data Mapping

| Origin (`crm_support`) | Target | Notes |
|---|---|---|
| `tickets.*` | `tks_tkt.tickets.*` | Field-for-field; `source='CH_EMAIL'` default for pre-TKS records |
| `ticketcomments.*` | `tks_tkt.ticket_comments.*` | Field-for-field; visibility preserved |
| `slapolicys.*` | `tks_tkt.sla_policies.*` | Field-for-field; business-hours profile pointer added (default = tenant default in `shared.cap`) |
| `knowledgearticles.*` | `tks_kb.articles` + first `tks_kb.article_versions` row | Existing body becomes version 1; `status` mapped DRAFT / APPROVED / PUBLISHED |

### 13.2 Event Routing-Key Bridge

For 60 days after `tks-tkt-svc` and `tks-kb-svc` go live:

| Legacy (crm.sup) | Re-emitted as |
|---|---|
| `crm.sup.ticket.created` | `tks.tkt.ticket.created` |
| `crm.sup.ticket.assigned` | `tks.tkt.ticket.assigned` |
| `crm.sup.ticket.escalated` | `tks.tkt.ticket.escalated` |
| `crm.sup.ticket.resolved` | `tks.tkt.ticket.resolved` |
| `crm.sup.ticket.reopened` | `tks.tkt.ticket.reopened` |
| `crm.sup.s_l_a.breached` | `tks.tkt.sla.breached` |

Bridge sits in the TKS event pipeline; new consumers subscribe only to `tks.*` keys.

### 13.3 API Redirects

`/api/crm/sup/v1/*` returns HTTP 308 to the corresponding `/api/tks/tkt/v1/*` or `/api/tks/kb/v1/*` path for 60 days. After the grace period, endpoints return 410 Gone.

### 13.4 Timeline

- 2026 Q3 (alongside TKS v1.0 GA): this service frozen (no new features, security patches only).
- 2026 Q3 + 60 days: routing-key bridge and HTTP 308 redirects disabled.
- Two minor versions after TKS v1.0 (target 2027 Q1): status = `RETIRED`; code and schema removed; data migration script `V0002__migrate_from_crm_support.sql` applied in `tks-tkt-svc` / `tks-kb-svc`.

### 13.5 Legacy House-keeping

- Flyway scripts: `V{version}__{description}.sql` (security patches only).
- No new feature work SHOULD land on this spec.
- Products composing new offerings MUST select `F-TKS-*` features instead of `F-CRM-016-*`.

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

- OpenAPI: `contracts/http/crm/sup/openapi.yaml`
- Event Schemas: `contracts/events/crm/sup/*.schema.json`