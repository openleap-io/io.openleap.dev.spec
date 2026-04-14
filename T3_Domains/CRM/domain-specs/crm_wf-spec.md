<!-- Template Meta
     Template-ID:   TPL-SVC
     Version:       1.0.0
     Compliance:    95%+
-->

# Workflow Service — Domain Specification

> **Conceptual Stack Layer:** Domain / Service
> **Space:** Platform
> **Owner:** Domain Engineering Team

> **Meta Information**
> - **Version:** 2026-04-03
> - **Template:** `domain-service-spec.md` v1.0.0
> - **Template Compliance:** 95%+
> - **Status:** DRAFT
> - **Service ID:** `crm-wf-svc`
> - **Suite:** `crm`
> - **Domain:** `wf`
> - **Base Package:** `com.openleap.crm.wf`
> - **API Path:** `/api/crm/wf/v1`
> - **Port:** 8415
> - **DB Schema:** `crm_workflow`
> - **Tier:** Extension

---

## 0. Document Purpose & Scope

This document specifies the **Workflow Service** (`crm-wf-svc`) within the CRM Suite.
It defines the domain model, business rules, use cases, REST API, events, data model, security, and quality attributes for the `Automation & Workflows` bounded context.

**Audience:** Backend developers, frontend developers, QA engineers, architects, product owners.

**In scope:** All domain logic within the `Automation & Workflows` bounded context.
**Out of scope:** UI implementation (see feature specs), infrastructure provisioning, cross-suite orchestration.

**Related documents:**
- Suite Spec: `_crm_suite.md`
- OpenAPI: `contracts/http/crm/wf/openapi.yaml`
- Event Schemas: `contracts/events/crm/wf/*.schema.json`

---

## 1. Business Context

### 1.1 Purpose & Responsibility

Rule-based automation engine for the CRM. Evaluates triggers (events, schedules, field changes), checks conditions, and executes actions (send notification, update field, create task, call webhook). Also manages assignment rules and escalation rules.

**Owns:**
- Workflow (automation rule: trigger + conditions + actions)
- WorkflowExecution (execution instance with step log)
- AssignmentRule (auto-assign leads/tickets based on criteria)
- EscalationRule (time-based escalation for overdue items)

### 1.2 Business Value

Provides essential automation & workflows capabilities within the CRM, enabling users to manage automation & workflows workflows efficiently and with full audit trail.

---

## 2. Service Identity

| Field | Value |
|-------|-------|
| Service ID | `crm-wf-svc` |
| Suite | `crm` |
| Domain | `wf` |
| Bounded Context | Automation & Workflows |
| Base Package | `com.openleap.crm.wf` |
| API Base Path | `/api/crm/wf/v1` |
| Port | 8415 |
| Database Schema | `crm_workflow` |
| Tier | Extension |
| Repository | `openleap-crm-wf-svc` |
| Team | `team-crm` |

---

## 3. Domain Model

### 3.1 Aggregates

**Workflow** (AggregateRoot)
- automation rule: trigger + conditions + actions

**WorkflowExecution** (AggregateRoot)
- execution instance with step log

**AssignmentRule** (AggregateRoot)
- auto-assign leads/tickets based on criteria

**EscalationRule** (AggregateRoot)
- time-based escalation for overdue items

---

## 4. Business Rules

### 4.1 Rule Catalog

| ID | Rule | Type |
|----|------|------|
| BR-WF-001 | Workflows execute asynchronously on event receipt | POLICY |
| BR-WF-002 | Max 10 actions per workflow | POLICY |
| BR-WF-003 | Workflow loops detected and terminated after 100 iterations | POLICY |
| BR-WF-004 | Assignment rules evaluated in priority order — first match wins | POLICY |
| BR-WF-005 | Escalation checks run every 5 minutes via scheduler | POLICY |

---

## 5. Use Cases

Use cases are derived from the endpoint catalog below. Each write endpoint corresponds to a WRITE use case; each read endpoint corresponds to a READ use case.

| Endpoint | Use Case Type | Primary Aggregate |
|----------|--------------|------------------|
| `POST /workflows` | WRITE | Workflow |
| `GET /workflows` | READ | Workflow |
| `GET /workflows/{id}` | READ | Workflow |
| `PUT /workflows/{id}` | WRITE | Workflow |
| `POST /workflows/{id}/activate` | WRITE | Workflow |
| `POST /workflows/{id}/deactivate` | WRITE | Workflow |
| `GET /workflow-executions` | READ | Workflow |
| `GET /workflow-executions/{id}` | READ | Workflow |
| `POST /assignment-rules` | WRITE | Workflow |
| `GET /assignment-rules` | READ | Workflow |
| `POST /escalation-rules` | WRITE | Workflow |
| `GET /escalation-rules` | READ | Workflow |

---

## 6. REST API

**Base Path:** `/api/crm/wf/v1`

**Authentication:** OAuth2/JWT (Bearer token)

### 6.1 Endpoint Summary

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/workflows` | POST workflows |
| `GET` | `/workflows` | GET workflows |
| `GET` | `/workflows/{id}` | GET {id} |
| `PUT` | `/workflows/{id}` | PUT {id} |
| `POST` | `/workflows/{id}/activate` | POST activate |
| `POST` | `/workflows/{id}/deactivate` | POST deactivate |
| `GET` | `/workflow-executions` | GET workflow-executions |
| `GET` | `/workflow-executions/{id}` | GET {id} |
| `POST` | `/assignment-rules` | POST assignment-rules |
| `GET` | `/assignment-rules` | GET assignment-rules |
| `POST` | `/escalation-rules` | POST escalation-rules |
| `GET` | `/escalation-rules` | GET escalation-rules |

> Full request/response examples in companion OpenAPI: `contracts/http/crm/wf/openapi.yaml`

---

## 7. Events & Integration

### 7.1 Published Events

| Event | Routing Key |
|-------|------------|
| WorkflowTriggered | `crm.wf.workflow.triggered` |
| WorkflowCompleted | `crm.wf.workflow.completed` |
| WorkflowFailed | `crm.wf.workflow.failed` |

### 7.2 Consumed Events

This service consumes events from other CRM services as needed for its bounded context. See Suite Spec SS5 for the full event catalog.

---

## 8. Data Model

**Storage:** PostgreSQL 16, Schema: `crm_workflow`

Tables are derived from the aggregate model in §3. Each aggregate root maps to a primary table. Embedded value objects are stored as columns or JSONB. All tables include `tenant_id` for Row-Level Security.

Detailed DDL will be provided in the implementation phase. Key tables:

- `workflows` — primary table for Workflow aggregate
- `workflowexecutions` — primary table for WorkflowExecution aggregate
- `assignmentrules` — primary table for AssignmentRule aggregate
- `escalationrules` — primary table for EscalationRule aggregate

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
| `F-CRM-011-01` | Workflow Builder |
| `F-CRM-011-02` | Workflow Template Library |
| `F-CRM-011-03` | Workflow Monitor & Logs |
| `F-CRM-011-04` | Assignment & Escalation Rules |

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

- OpenAPI: `contracts/http/crm/wf/openapi.yaml`
- Event Schemas: `contracts/events/crm/wf/*.schema.json`