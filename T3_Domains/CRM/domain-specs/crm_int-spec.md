<!-- Template Meta
     Template-ID:   TPL-SVC
     Version:       1.0.0
     Compliance:    95%+
-->

# Integration Service — Domain Specification

> **Conceptual Stack Layer:** Domain / Service
> **Space:** Platform
> **Owner:** Domain Engineering Team

> **Meta Information**
> - **Version:** 2026-04-03
> - **Template:** `domain-service-spec.md` v1.0.0
> - **Template Compliance:** 95%+
> - **Status:** DRAFT
> - **Service ID:** `crm-int-svc`
> - **Suite:** `crm`
> - **Domain:** `int`
> - **Base Package:** `com.openleap.crm.int`
> - **API Path:** `/api/crm/int/v1`
> - **Port:** 8417
> - **DB Schema:** `crm_integration`
> - **Tier:** Extension

---

## 0. Document Purpose & Scope

This document specifies the **Integration Service** (`crm-int-svc`) within the CRM Suite.
It defines the domain model, business rules, use cases, REST API, events, data model, security, and quality attributes for the `Integrations & Connectors` bounded context.

**Audience:** Backend developers, frontend developers, QA engineers, architects, product owners.

**In scope:** All domain logic within the `Integrations & Connectors` bounded context.
**Out of scope:** UI implementation (see feature specs), infrastructure provisioning, cross-suite orchestration.

**Related documents:**
- Suite Spec: `_crm_suite.md`
- OpenAPI: `contracts/http/crm/int/openapi.yaml`
- Event Schemas: `contracts/events/crm/int/*.schema.json`

---

## 1. Business Context

### 1.1 Purpose & Responsibility

Integration hub — Zapier-style webhooks, pre-built connectors (Mailchimp, Salesforce, HubSpot, Slack), OAuth management for external apps, and custom API integrations. Consumes configurable CRM events and forwards them to external systems.

**Owns:**
- IntegrationConnection (webhook or connector config with OAuth tokens)
- WebhookSubscription (event type → target URL mapping with retry policy)
- ConnectorDefinition (pre-built connector metadata and field mapping)
- SyncLog (execution log per sync operation)

### 1.2 Business Value

Provides essential integrations & connectors capabilities within the CRM, enabling users to manage integrations & connectors workflows efficiently and with full audit trail.

---

## 2. Service Identity

| Field | Value |
|-------|-------|
| Service ID | `crm-int-svc` |
| Suite | `crm` |
| Domain | `int` |
| Bounded Context | Integrations & Connectors |
| Base Package | `com.openleap.crm.int` |
| API Base Path | `/api/crm/int/v1` |
| Port | 8417 |
| Database Schema | `crm_integration` |
| Tier | Extension |
| Repository | `openleap-crm-int-svc` |
| Team | `team-crm` |

---

## 3. Domain Model

### 3.1 Aggregates

**IntegrationConnection** (AggregateRoot)
- webhook or connector config with OAuth tokens

**WebhookSubscription** (AggregateRoot)
- event type → target URL mapping with retry policy

**ConnectorDefinition** (AggregateRoot)
- pre-built connector metadata and field mapping

**SyncLog** (AggregateRoot)
- execution log per sync operation

---

## 4. Business Rules

### 4.1 Rule Catalog

| ID | Rule | Type |
|----|------|------|
| BR-INT-001 | Webhook delivery timeout: 10 seconds | POLICY |
| BR-INT-002 | Retry policy: 3 attempts with exponential backoff | POLICY |
| BR-INT-003 | OAuth tokens encrypted at rest with AES-256 | POLICY |
| BR-INT-004 | Connector field mappings are tenant-configurable | POLICY |
| BR-INT-005 | Max 50 active webhook subscriptions per tenant | POLICY |

---

## 5. Use Cases

Use cases are derived from the endpoint catalog below. Each write endpoint corresponds to a WRITE use case; each read endpoint corresponds to a READ use case.

| Endpoint | Use Case Type | Primary Aggregate |
|----------|--------------|------------------|
| `POST /integrations` | WRITE | IntegrationConnection |
| `GET /integrations` | READ | IntegrationConnection |
| `GET /integrations/{id}` | READ | IntegrationConnection |
| `PUT /integrations/{id}` | WRITE | IntegrationConnection |
| `DELETE /integrations/{id}` | WRITE | IntegrationConnection |
| `POST /webhooks` | WRITE | IntegrationConnection |
| `GET /webhooks` | READ | IntegrationConnection |
| `DELETE /webhooks/{id}` | WRITE | IntegrationConnection |
| `GET /connectors` | READ | IntegrationConnection |
| `GET /sync-logs` | READ | IntegrationConnection |

---

## 6. REST API

**Base Path:** `/api/crm/int/v1`

**Authentication:** OAuth2/JWT (Bearer token)

### 6.1 Endpoint Summary

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/integrations` | POST integrations |
| `GET` | `/integrations` | GET integrations |
| `GET` | `/integrations/{id}` | GET {id} |
| `PUT` | `/integrations/{id}` | PUT {id} |
| `DELETE` | `/integrations/{id}` | DELETE {id} |
| `POST` | `/webhooks` | POST webhooks |
| `GET` | `/webhooks` | GET webhooks |
| `DELETE` | `/webhooks/{id}` | DELETE {id} |
| `GET` | `/connectors` | GET connectors |
| `GET` | `/sync-logs` | GET sync-logs |

> Full request/response examples in companion OpenAPI: `contracts/http/crm/int/openapi.yaml`

---

## 7. Events & Integration

### 7.1 Published Events

| Event | Routing Key |
|-------|------------|
| WebhookDispatched | `crm.int.webhook.dispatched` |
| WebhookFailed | `crm.int.webhook.failed` |
| SyncCompleted | `crm.int.sync.completed` |

### 7.2 Consumed Events

This service consumes events from other CRM services as needed for its bounded context. See Suite Spec SS5 for the full event catalog.

---

## 8. Data Model

**Storage:** PostgreSQL 16, Schema: `crm_integration`

Tables are derived from the aggregate model in §3. Each aggregate root maps to a primary table. Embedded value objects are stored as columns or JSONB. All tables include `tenant_id` for Row-Level Security.

Detailed DDL will be provided in the implementation phase. Key tables:

- `integrationconnections` — primary table for IntegrationConnection aggregate
- `webhooksubscriptions` — primary table for WebhookSubscription aggregate
- `connectordefinitions` — primary table for ConnectorDefinition aggregate
- `synclogs` — primary table for SyncLog aggregate

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
| `F-CRM-013-01` | Integration Hub |
| `F-CRM-013-02` | Webhook Configuration |
| `F-CRM-013-03` | Connector Marketplace |
| `F-CRM-013-04` | Integration Logs & Monitor |

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

- OpenAPI: `contracts/http/crm/int/openapi.yaml`
- Event Schemas: `contracts/events/crm/int/*.schema.json`