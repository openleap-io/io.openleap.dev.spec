<!-- Template Meta
     Template-ID:   TPL-SVC
     Version:       1.1.0
     Compliance:    95%+
-->

# CMDB Service — Domain Specification

> **Conceptual Stack Layer:** Domain / Service
> **Space:** Platform
> **Owner:** team-tks

> **Meta Information**
> - **Version:** 2026-04-18
> - **Template:** `domain-service-spec.md` v1.1.0
> - **Template Compliance:** 95%+
> - **Status:** DRAFT
> - **Service ID:** `tks-cmdb-svc`
> - **Suite:** `tks`
> - **Domain:** `cmdb`
> - **Base Package:** `io.openleap.tks.cmdb`
> - **API Path:** `/api/tks/cmdb/v1`
> - **Port:** 8503
> - **DB Schema:** `tks_cmdb`
> - **Tier:** T3
> - **Bounded Context:** `bc:cmdb`

---

## 0. Document Purpose & Scope

This document specifies the **CMDB Service** (`tks-cmdb-svc`): configuration items, typed attributes, typed relationships, ticket ↔ CI linkage, and impact analysis.

**In scope:** `ConfigurationItem`, `CiType`, `CiRelationship`, `CiAttribute`, `TicketCiLink`, impact graph query.
**Out of scope:** Financial asset accounting (→ `fi`); detailed asset procurement lifecycle (future `pur` suite); monitoring / telemetry (out of platform scope).

---

## 1. Business Context

### 1.1 Purpose & Responsibility

Own the per-tenant graph of things (servers, applications, services, subscriptions, documents, people-as-CI) that tickets can reference. Provide impact analysis so operators see what's affected when a CI is broken.

**Owns:** `ConfigurationItem`, `CiType`, `CiRelationship`, `CiAttribute`, `TicketCiLink`.

### 1.2 Business Value

Links tickets to affected assets; visualises blast radius; enables change-management coordination.

---

## 2. Service Identity

| Field | Value |
|-------|-------|
| Service ID | `tks-cmdb-svc` |
| Suite | `tks` |
| Domain | `cmdb` |
| Bounded Context | `bc:cmdb` |
| Base Package | `io.openleap.tks.cmdb` |
| API Base Path | `/api/tks/cmdb/v1` |
| Port | 8503 |
| Database Schema | `tks_cmdb` |
| Tier | T3 |
| Repository | `openleap-io/io.openleap.tks.cmdb` |
| Team | `team-tks` |

---

## 3. Domain Model

### 3.1 Aggregates

**CiType** (AggregateRoot)

| Attribute | Type | Notes |
|---|---|---|
| id | uuid | |
| tenantId | uuid | RLS |
| name | string | unique per tenant |
| description | text | |
| attributeSchema | JSON Schema | typed attributes per CI of this type |
| allowedRelationshipTypes | string[] | e.g. `["DEPENDS_ON","HOSTED_ON"]` |
| status | enum | `ACTIVE`,`ARCHIVED` |

**ConfigurationItem** (AggregateRoot)

| Attribute | Type | Notes |
|---|---|---|
| id | uuid | |
| tenantId | uuid | RLS |
| typeId | uuid | → CiType |
| name | string | |
| externalRef | string? | external ID (CMDB sync with vendor tools) |
| attributes | JSONB | validated against `CiType.attributeSchema` |
| ownerPartyId | uuid? | → `shared.bp` |
| status | enum | `PLANNED`,`ACTIVE`,`RETIRED` |
| createdAt, retiredAt | timestamp | |
| version | int | optimistic locking |

**CiRelationship** (AggregateRoot)

| Attribute | Type | Notes |
|---|---|---|
| id | uuid | |
| tenantId | uuid | |
| sourceCiId | uuid | |
| targetCiId | uuid | |
| relationshipType | string | e.g. `DEPENDS_ON` |
| strength | int? | 0..100 |
| createdAt | timestamp | |

**TicketCiLink** (AggregateRoot, join but owned here per ADR-TKS-002 discussion)

| Attribute | Type | Notes |
|---|---|---|
| id | uuid | |
| tenantId | uuid | |
| ticketId | uuid | → `tks.tkt.ticket` |
| ciId | uuid | → CI |
| linkReason | enum | `AFFECTED`,`CAUSED_BY`,`WORKAROUND_ON`,`INFORMATIONAL` |
| createdAt | timestamp | |
| createdByPrincipalId | uuid | |

---

## 4. Business Rules

### 4.1 Rule Catalog

| ID | Rule | Type |
|----|------|------|
| BR-CMDB-001 | `DEPENDS_ON` relationships MUST NOT form cycles | CONSTRAINT |
| BR-CMDB-002 | `CiRelationship.relationshipType` MUST appear in both endpoint CiType's `allowedRelationshipTypes` | CONSTRAINT |
| BR-CMDB-003 | Retiring a CI cascades relationships to `ARCHIVED_RELATIONSHIP` state (retained for audit) | POLICY |
| BR-CMDB-004 | `ConfigurationItem.attributes` MUST validate against `CiType.attributeSchema` | CONSTRAINT |
| BR-CMDB-005 | Impact-analysis graph traversal limited to depth 3 by default, max 5 (for perf) | POLICY |
| BR-CMDB-006 | `TicketCiLink` creation validates both ticket and CI belong to same tenant | CONSTRAINT |
| BR-CMDB-007 | Tenant isolation via RLS on `tenant_id` across all tables | CONSTRAINT |

---

## 5. Use Cases

| UC | Type | Trigger | Aggregate | Events | REST |
|----|------|---------|-----------|--------|------|
| DefineCiType | WRITE | REST (admin) | CiType | `tks.cmdb.ci-type.created` | `POST /ci-types` |
| UpdateCiType | WRITE | REST | CiType | `tks.cmdb.ci-type.updated` | `PATCH /ci-types/{id}` |
| CreateCi | WRITE | REST | ConfigurationItem | `tks.cmdb.ci.created` | `POST /cis` |
| UpdateCi | WRITE | REST | ConfigurationItem | `tks.cmdb.ci.updated` | `PATCH /cis/{id}` |
| RetireCi | WRITE | REST | ConfigurationItem | `tks.cmdb.ci.retired` | `POST /cis/{id}/retire` |
| CreateRelationship | WRITE | REST | CiRelationship | `tks.cmdb.relationship.created` | `POST /relationships` |
| RemoveRelationship | WRITE | REST | CiRelationship | `tks.cmdb.relationship.removed` | `DELETE /relationships/{id}` |
| LinkTicketToCi | WRITE | REST (from `tks-tkt-svc` or agent UI) | TicketCiLink | `tks.cmdb.ticket-ci-link.created` | `POST /ticket-ci-links` |
| UnlinkTicketFromCi | WRITE | REST | TicketCiLink | `tks.cmdb.ticket-ci-link.removed` | `DELETE /ticket-ci-links/{id}` |
| ImpactAnalysis | READ | REST | — | — | `GET /cis/{id}/impact?depth=3` |
| QueryCis | READ | REST | ConfigurationItem | — | `GET /cis?type=...&attribute.xxx=...` |
| GetCi | READ | REST | ConfigurationItem | — | `GET /cis/{id}` |
| ListTicketCiLinks | READ | REST | TicketCiLink | — | `GET /ticket-ci-links?ticketId=...` or `?ciId=...` |

---

## 6. REST API

**Base Path:** `/api/tks/cmdb/v1`
**Scopes:** `tks.cmdb:read`, `tks.cmdb:write`, `tks.cmdb:admin`.

### 6.1 Endpoint Summary

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/ci-types` | Create CI type |
| `GET` | `/ci-types` | List |
| `GET` | `/ci-types/{id}` | Read |
| `PATCH` | `/ci-types/{id}` | Update |
| `POST` | `/cis` | Create CI |
| `GET` | `/cis` | List / query by typed attribute |
| `GET` | `/cis/{id}` | Read |
| `PATCH` | `/cis/{id}` | Update |
| `POST` | `/cis/{id}/retire` | Retire |
| `GET` | `/cis/{id}/impact` | Impact graph (depth 1-5) |
| `POST` | `/relationships` | Create edge |
| `DELETE` | `/relationships/{id}` | Remove |
| `GET` | `/relationships` | List for CI (query `ciId`) |
| `POST` | `/ticket-ci-links` | Link |
| `DELETE` | `/ticket-ci-links/{id}` | Unlink |
| `GET` | `/ticket-ci-links` | Filtered list |

---

## 7. Events & Integration

### 7.1 Pattern

`event_driven`.

### 7.2 Published Events

| Event | Routing Key |
|-------|------------|
| CiTypeCreated | `tks.cmdb.ci-type.created` |
| CiTypeUpdated | `tks.cmdb.ci-type.updated` |
| CiCreated | `tks.cmdb.ci.created` |
| CiUpdated | `tks.cmdb.ci.updated` |
| CiRetired | `tks.cmdb.ci.retired` |
| RelationshipCreated | `tks.cmdb.relationship.created` |
| RelationshipRemoved | `tks.cmdb.relationship.removed` |
| TicketCiLinkCreated | `tks.cmdb.ticket-ci-link.created` |
| TicketCiLinkRemoved | `tks.cmdb.ticket-ci-link.removed` |

### 7.3 Consumed Events

| Source | Routing Key | Purpose |
|---|---|---|
| `tks-tkt-svc` | `tks.tkt.ticket.resolved` | Update link status (historical record; link stays but flagged `resolved-at`) |
| `shared-bp-svc` | `shared.bp.party.erased` | Null out `ownerPartyId` references |

---

## 8. Data Model

**Storage:** PostgreSQL 16, schema `tks_cmdb`.

Tables: `ci_types`, `configuration_items`, `ci_relationships`, `ticket_ci_links`, `outbox_events`.

Graph queries via recursive CTE on `ci_relationships`; for tenants with > 100 k CIs the plan reserves an option to overlay a graph DB (ADR-CMDB-002 placeholder).

Indexes: `(tenant_id, type_id)`, `(tenant_id, status)`, `(tenant_id, source_ci_id)` + `(tenant_id, target_ci_id)` on relationships, GIN on `attributes`.

---

## 9. Security & Compliance

### 9.1 Access Control

| Role | Access Level |
|------|-------------|
| `cmdb-admin` | Manage CiTypes + all CIs |
| `cmdb-operator` | CRUD CIs, relationships, links |
| `support-agent` | Read + create TicketCiLink |
| `tenant-user` | Read own tenant CIs (no internal config secrets) |

### 9.2 Data Classification

Internal + selective Confidential (attributes may include credentials refs — stored as pointer to `iam.tenant` secrets, never inline).

### 9.3 Compliance

- **ISO 27001 A.8 asset inventory** — CMDB serves as the asset register.
- **DORA ICT asset lifecycle** — retirements audited; relationships retained for retrospective impact analysis.

### 9.4 DORA ICT Risk References

| Risk ID | Title | Treatment |
|---|---|---|
| `RISK-TKS-CMDB-001` | Stale CMDB misleads impact analysis during incident | Mitigate: freshness SLO (last-verified-at on each CI); periodic reconciliation jobs |
| `RISK-TKS-CMDB-002` | Cyclic dependency undermines graph queries | Mitigate: BR-CMDB-001 enforced + background detector |

### 9.5 SBOM

CycloneDX per build.

---

## 10. Quality Attributes

### 10.1 Performance

| Operation | p95 | p99 |
|---|---|---|
| `GET /cis/{id}` | < 60 ms | < 180 ms |
| `GET /cis/{id}/impact` (depth 3) | < 300 ms | < 1 s |
| `POST /cis` | < 120 ms | < 350 ms |
| Attribute query (indexed) | < 200 ms | < 500 ms |

### 10.2 Availability

99.9 %.

### 10.3 Scalability

Stateless; reads scale via read-replica; graph-query cache per (ciId, depth) with invalidation on relationship change.

### 10.5 SLI/SLO Reference

`SLO-TKS-CMDB-001`.

### 10.6 Resilience

| Aspect | Value |
|---|---|
| RTO | < 15 min |
| RPO | < 5 min |

---

## 11. Feature Dependencies

| Feature ID | Feature Name |
|-----------|-------------|
| `F-TKS-400` | Configuration Items |
| `F-TKS-410` | CI Relationships & Impact |
| `F-TKS-420` | Ticket ↔ CI Linkage |

---

## 12. Extension Points

- Custom attribute validators via `POST /ci-types/extensions/validators`.
- Impact-analysis scoring pluggable via extension hook `tks.cmdb.ext.impact-scoring`.
- Relationship-type catalog extensible per tenant (new edge types).

---

## 13. Migration & Evolution

- No predecessor; greenfield per TKS v1.
- v1.1: optional graph-DB overlay (Neo4j) for tenants > 100 k CIs.
- v1.2: reconciliation connectors (AWS/Azure/GCP inventory sync).

---

## 14. Decisions & Open Questions

| # | Question | Severity | Status |
|---|----------|----------|--------|
| Q-CMDB-001 | Ship ITIL-standard CiTypes out-of-box or leave to tenant? | Medium | Open — recommend shipping a "starter pack" library |
| Q-CMDB-002 | Tier promotion to T2 if another suite needs CIs (FI asset accounting)? | Medium | Deferred to ADR-TKS-002 |

### ADRs

- **ADR-CMDB-001** *(Proposed)*: JSONB attribute storage validated against per-type JSON Schema at write time.
- **ADR-CMDB-002** *(Proposed)*: Start with PostgreSQL recursive CTE for graph queries; reserve option to adopt graph DB overlay.

---

## 15. Appendix

### 15.1 Glossary

See `_tks_suite.md` SS1.

### 15.2 Change Log

| Date | Version | Author | Changes |
|------|---------|--------|---------|
| 2026-04-18 | 1.0.0 | OpenLeap Architecture Team | Initial CMDB Service spec |

### 15.3 Companion Files

- OpenAPI: `contracts/http/tks/cmdb/openapi.yaml`
- Event Schemas: `contracts/events/tks/cmdb/*.schema.json`
