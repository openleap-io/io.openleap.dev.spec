<!-- Template Meta
     Template-ID:   TPL-SVC
     Version:       1.1.0
     Compliance:    95%+
-->

# Workflow Service — Domain Specification

> **Conceptual Stack Layer:** Domain / Service
> **Space:** Platform
> **Owner:** Platform Shared-Business Team

> **Meta Information**
> - **Version:** 2026-04-18
> - **Template:** `domain-service-spec.md` v1.1.0
> - **Template Compliance:** 95%+
> - **Status:** DRAFT
> - **Service ID:** `auto-wf-svc`
> - **Suite:** `auto`
> - **Domain:** `wf`
> - **Base Package:** `io.openleap.auto.wf`
> - **API Path:** `/api/auto/wf/v1`
> - **Port:** 8415
> - **DB Schema:** `auto_workflow`
> - **Tier:** T2 — Common
> - **Supersedes:** `crm-wf-svc` (see §13 Migration)

---

## 0. Document Purpose & Scope

This document specifies the **Workflow Service** (`auto-wf-svc`) as a T2 Common capability: a suite-agnostic rule-based automation engine consumed by any T3 suite.

**Audience:** Backend developers, frontend developers, QA engineers, architects, product owners across every suite.

**In scope:** Triggers (events, schedules, field changes), conditions (predicates), actions (notify, update, create, webhook). Assignment rules and escalation rules as specialised workflow families.
**Out of scope:** Long-running stateful saga orchestration (defer to a future `ops.saga` service); domain-specific business rules (those stay inside their owning domain).

**Related documents:**
- Suite Spec: `T2_Common/_auto_suite.md`
- OpenAPI: `contracts/http/auto/wf/openapi.yaml`
- Event Schemas: `contracts/events/auto/wf/*.schema.json`
- Origin (deprecated): `T3_Domains/CRM/domain-specs/crm_wf-spec.md`

---

## 1. Business Context

### 1.1 Purpose & Responsibility

A generic, declarative rule engine. Evaluates triggers (domain events, cron schedules, field-change hints), checks conditions against payload + cached reference data, executes ordered actions: send notification (via `auto.ntf`), update an aggregate through its REST API, create a task in any domain, call an external webhook, or publish a further event.

**Owns:**
- Workflow (automation rule: trigger + conditions + actions)
- WorkflowExecution (execution instance with step log and outcome)
- AssignmentRule (auto-assign any work-item aggregate based on criteria and round-robin/load-balance strategies)
- EscalationRule (time-based escalation for aggregates referencing an SLA clock)
- WorkflowLibraryTemplate (shared library of reusable templates — new in T2)

### 1.2 Business Value

Every suite gets the same workflow primitives instead of rebuilding them. Business users configure automation once per tenant; any suite whose events match a trigger runs the rule.

---

## 2. Service Identity

| Field | Value |
|-------|-------|
| Service ID | `auto-wf-svc` |
| Suite | `auto` |
| Domain | `wf` |
| Bounded Context | Automation & Workflows |
| Base Package | `io.openleap.auto.wf` |
| API Base Path | `/api/auto/wf/v1` |
| Port | 8415 |
| Database Schema | `auto_workflow` |
| Tier | T2 — Common |
| Repository | `openleap-io/io.openleap.auto.wf` |
| Team | `team-platform-shared` |

---

## 3. Domain Model

### 3.1 Aggregates

**Workflow** (AggregateRoot)
- id, tenantId, name, description, trigger (`EVENT`|`SCHEDULE`|`FIELD_CHANGE`|`MANUAL`), triggerSpec (JSONB: routing-key pattern, cron expression, or field selector), conditions[] (predicates over payload + reference data), actions[] (ordered), status (`DRAFT`|`ACTIVE`|`PAUSED`|`ARCHIVED`), version.

**WorkflowExecution** (AggregateRoot)
- id, workflowId, triggeredBy (eventRef or scheduleRef), startedAt, finishedAt, status (`RUNNING`|`SUCCEEDED`|`FAILED`|`TERMINATED`), stepLog[] (per-action: name, input, output, latency, error?), correlationId.

**AssignmentRule** (AggregateRoot)
- id, tenantId, appliesTo (aggregate type, e.g. `tks.tkt.ticket`), criteria (JSONB predicate), strategy (`ROUND_ROBIN`|`LOAD_BALANCE`|`SKILL_BASED`|`CUSTOM_SCRIPT`), queueId or principalIds[], priority (int).

**EscalationRule** (AggregateRoot)
- id, tenantId, appliesTo, slaClockRef, thresholds[] ({pctConsumed, action}), status.

**WorkflowLibraryTemplate** (AggregateRoot)
- id, name, description, suiteHint, template (Workflow blueprint with placeholders). Platform maintains defaults; tenants clone-and-customize.

---

## 4. Business Rules

### 4.1 Rule Catalog

| ID | Rule | Type |
|----|------|------|
| BR-WF-001 | Workflows execute asynchronously on trigger; REST endpoint `execute` MUST return 202 Accepted | POLICY |
| BR-WF-002 | Max 10 actions per workflow (hard limit); greater requires sub-workflow composition | CONSTRAINT |
| BR-WF-003 | Workflow loops detected by repeated (workflowId, correlationId) and terminated after 100 iterations with `WorkflowFailed{reason=LOOP}` event | CONSTRAINT |
| BR-WF-004 | Assignment rules evaluated in descending priority; first match wins | POLICY |
| BR-WF-005 | Escalation checks run every 60 s via scheduler (configurable per tenant, min 60 s, max 1 h) | POLICY |
| BR-WF-006 | Actions that mutate other aggregates MUST call the target service's REST API with caller's original tenant context (no privilege escalation) | CONSTRAINT |
| BR-WF-007 | Cross-suite writes by a workflow are permitted BUT each target write is audit-logged with the workflow-execution id | POLICY |
| BR-WF-008 | Tenant isolation: workflows MUST NOT trigger on events from other tenants; enforced at subscription layer | CONSTRAINT |

---

## 5. Use Cases

| Endpoint | Use Case Type | Primary Aggregate |
|----------|--------------|------------------|
| `POST /workflows` | WRITE | Workflow |
| `GET /workflows` | READ | Workflow |
| `GET /workflows/{id}` | READ | Workflow |
| `PUT /workflows/{id}` | WRITE | Workflow |
| `POST /workflows/{id}/activate` | WRITE | Workflow |
| `POST /workflows/{id}/pause` | WRITE | Workflow |
| `POST /workflows/{id}/execute` | WRITE (manual trigger) | Workflow |
| `GET /workflow-executions` | READ | WorkflowExecution |
| `GET /workflow-executions/{id}` | READ | WorkflowExecution |
| `POST /assignment-rules` | WRITE | AssignmentRule |
| `GET /assignment-rules` | READ | AssignmentRule |
| `POST /escalation-rules` | WRITE | EscalationRule |
| `GET /escalation-rules` | READ | EscalationRule |
| `GET /library-templates` | READ | WorkflowLibraryTemplate |
| `POST /library-templates/{id}/clone` | WRITE | Workflow |

---

## 6. REST API

**Base Path:** `/api/auto/wf/v1`
**Authentication:** OAuth2 / JWT (Bearer). **Authorization:** scopes `auto.wf:read`, `auto.wf:write`, `auto.wf:admin`.

Endpoint table above is the canonical summary. Full payloads + request/response schemas in `contracts/http/auto/wf/openapi.yaml`.

---

## 7. Events & Integration

### 7.1 Published Events

| Event | Routing Key |
|-------|------------|
| WorkflowTriggered | `auto.wf.workflow.triggered` |
| WorkflowCompleted | `auto.wf.workflow.completed` |
| WorkflowFailed | `auto.wf.workflow.failed` |
| AssignmentPerformed | `auto.wf.assignment.performed` |
| EscalationFired | `auto.wf.escalation.fired` |

### 7.2 Consumed Events

Any domain event across the platform is a potential trigger. Notable subscriptions (v1):

- `tks.tkt.ticket.created` → run assignment rules
- `tks.sla.approaching-breach` / `tks.sla.breached` → run escalation rules
- `crm.lead.created` → lead-routing
- `sd.ord.order-placed` → downstream follow-up workflows

### 7.3 Routing-Key Bridge (Migration)

For 60 days after promotion, `auto-wf-svc` SHALL also publish outbound events under legacy `crm.wf.*` keys.

---

## 8. Data Model

**Storage:** PostgreSQL 16, Schema: `auto_workflow`. Multi-tenant RLS on `tenant_id`.

Key tables: `workflows`, `workflow_executions`, `workflow_execution_steps`, `assignment_rules`, `escalation_rules`, `workflow_library_templates`.

---

## 9. Security & Compliance

### 9.1 Access Control

| Role | Access Level |
|------|-------------|
| `platform-admin` | Manage library templates (platform-default) |
| `tenant-admin` | CRUD all workflows + rules for own tenant |
| `automation-designer` (per-tenant) | CRUD workflows + rules |
| `tenant-user` | Read + manual-execute workflows granted explicitly |
| `service-account` | Subscribe to events + invoke actions internally |

### 9.2 Data Classification

Internal. Workflow definitions and execution logs may reference PII (through aggregate IDs) but do not store it directly.

### 9.3 DORA / Audit

Every workflow execution step is audit-logged (§7.1 `WorkflowCompleted` event carries the full step log). Steps that mutate other aggregates are traceable via correlationId to the calling audit record in `iam.audit`.

---

## 10. Quality Attributes

### 10.1 Performance Targets

| Operation | p95 | p99 |
|-----------|-----|-----|
| Trigger → first action start | < 500 ms | < 2 s |
| Single-action workflow end-to-end | < 1 s | < 5 s |
| `GET /workflows` (page 50) | < 120 ms | < 300 ms |

### 10.2 Availability

99.9 %. Execution engine resilient via outbox + retry; API tier 2+ replicas.

### 10.3 Scalability

- Stateless API tier (horizontal).
- Worker pool scales on queue depth.
- Per-tenant rate limits; per-workflow concurrency caps.
- Dead-letter queue for poisoned executions.

---

## 11. Feature Dependencies

| Feature ID | Feature Name |
|-----------|-------------|
| `F-AUTO-WF-001` | Workflow Builder |
| `F-AUTO-WF-002` | Workflow Template Library |
| `F-AUTO-WF-003` | Workflow Monitor & Logs |
| `F-AUTO-WF-004` | Assignment & Escalation Rules |

Consumed by feature specs from multiple suites that require automation (e.g., `F-TKS-120` SLA & Escalation, `F-TKS-130` Queue & Assignment, `F-CRM-*`).

---

## 12. Extension Points

- `Workflow.actions[]` is an open registry; new action types register via `POST /workflows/extensions/action-types`.
- Condition predicate language supports custom functions via `POST /workflows/extensions/predicate-fns`.
- Extension events `auto.wf.ext.pre-execute`, `auto.wf.ext.post-execute` published so tenant-specific logic can observe every execution.

---

## 13. Migration & Evolution

### 13.1 Promotion from `crm-wf-svc`

- Origin spec `T3_Domains/CRM/domain-specs/crm_wf-spec.md` marked DEPRECATED.
- Routing keys: new `auto.wf.*`; old `crm.wf.*` re-emitted 60 days.
- API path `/api/auto/wf/v1`; `/api/crm/wf/v1` returns HTTP 308 during grace period.
- Schema `crm_workflow` renamed to `auto_workflow` via Flyway.
- Port stays 8415.
- `AssignmentRule.appliesTo` enum extended from CRM-only aggregates to include `tks.tkt.ticket`, `sd.ord.order`, etc.

### 13.2 Post-Migration

All CRM-internal callers refactored within 60 days. Bridge removed. Origin spec retired two minor versions after this reaches ACTIVE.

---

## 14. Decisions & Open Questions

| # | Question | Severity | Status |
|---|----------|----------|--------|
| Q-WF-001 | Do we adopt Temporal.io as the engine or keep the bespoke rule evaluator? | High | Open — impacts ADR-WF-001 |
| Q-WF-002 | Should WorkflowLibraryTemplate sit in `param.ref` instead? | Low | Open |
| Q-WF-003 | Cross-suite writes audit format — reuse `iam.audit` schema or define a local one? | Medium | Open |

### ADRs

- **ADR-WF-001** *(Proposed)*: Promote to T2 Shared. Rationale: every T3 suite needs automation; leaving it in CRM forces other suites to reimplement the wheel or violate the UBL boundary. Consequences: migration cost + engine breadth (aggregate-type registry).
- **ADR-WF-002** *(Proposed)*: Cross-suite writes permitted through REST with audit, not via shared DB. Preserves bounded-context autonomy.

---

## 15. Appendix

### 15.1 Glossary

See `T2_Common/_auto_suite.md` SS1.

### 15.2 Change Log

| Date | Version | Author | Changes |
|------|---------|--------|---------|
| 2026-04-18 | 1.0.0 | OpenLeap Architecture Team | Initial promotion spec from crm.wf (supersedes crm-wf-svc) |

### 15.3 Companion Files

- OpenAPI: `contracts/http/auto/wf/openapi.yaml`
- Event Schemas: `contracts/events/auto/wf/*.schema.json`
- Origin (deprecated): `T3_Domains/CRM/domain-specs/crm_wf-spec.md`
