<!-- Template Meta
     Template-ID:   TPL-SVC
     Version:       1.1.0
     Compliance:    95%+
-->

# Search Service — Domain Specification

> **Conceptual Stack Layer:** Domain / Service
> **Space:** Platform
> **Owner:** Platform Infrastructure Team

> **Meta Information**
> - **Version:** 2026-04-18
> - **Template:** `domain-service-spec.md` v1.1.0
> - **Template Compliance:** 95%+
> - **Status:** DRAFT
> - **Service ID:** `tech-search-svc`
> - **Suite:** `tech`
> - **Domain:** `search`
> - **Base Package:** `io.openleap.tech.search`
> - **API Path:** `/api/tech/search/v1`
> - **Port:** 8406
> - **DB Schema:** Elasticsearch (CQRS read model — no PostgreSQL)
> - **Tier:** T1 — Platform & Technical Foundations
> - **Supersedes:** `crm-search-svc` (see §13 Migration)

---

## 0. Document Purpose & Scope

This document specifies the **Search Service** (`tech-search-svc`) as a T1 Platform infrastructure capability: a generic CQRS read-model search fabric consumed by any suite that needs full-text, faceted, or typeahead search.

**Audience:** Backend developers, frontend developers, architects, platform ops.

**In scope:** Index maintenance, query API, saved views, reindex jobs. Projections from domain events into Elasticsearch indices.
**Out of scope:** Domain data ownership (search is pure read side), business-level analytics (→ T4 BI).

**Related documents:**
- Suite Spec: `T1_Platform/tech/_tech_suite.md`
- OpenAPI: `contracts/http/tech/search/openapi.yaml`
- Event Schemas: `contracts/events/tech/search/*.schema.json` (saved-view events only)
- Origin (deprecated): `T3_Domains/CRM/domain-specs/crm_search-spec.md`

---

## 1. Business Context

### 1.1 Purpose & Responsibility

Global full-text search, faceted filtering, typeahead, and saved views across every indexable aggregate in the platform. Pure CQRS read model — consumes events from all suites and projects them into Elasticsearch indices. Owns no business data.

**Owns:**
- SavedView (persisted filter + column configuration, user- or team-scoped)
- IndexSchema (Elasticsearch mapping definition per aggregate type — registry-managed)
- IndexProjector (event → document transformation metadata per source event)

### 1.2 Business Value

One search fabric for every domain: agents, users, and products all get consistent global search, one-shot typeahead, and tenant-safe result scoping.

---

## 2. Service Identity

| Field | Value |
|-------|-------|
| Service ID | `tech-search-svc` |
| Suite | `tech` |
| Domain | `search` |
| Bounded Context | Platform Search |
| Base Package | `io.openleap.tech.search` |
| API Base Path | `/api/tech/search/v1` |
| Port | 8406 |
| Storage | Elasticsearch 9.x (primary), PostgreSQL schema `tech_search` (SavedView persistence) |
| Tier | T1 — Platform & Technical Foundations |
| Repository | `openleap-io/io.openleap.tech.search` |
| Team | `team-platform-infra` |

---

## 3. Domain Model

### 3.1 Aggregates

**SavedView** (AggregateRoot)
- id, tenantId, ownerPrincipalId, scope (`PRIVATE`|`TEAM`|`TENANT`), aggregateType, filters (JSONB), columns[], sortBy, sharedWithPrincipalIds[].

**IndexSchema** (AggregateRoot)
- aggregateType (e.g., `tks.tkt.ticket`, `crm.lead.lead`), mappings (JSONB), analyzers, version.

**IndexProjector** (AggregateRoot)
- routingKey subscription, aggregateType target, projector script reference, status.

---

## 4. Business Rules

### 4.1 Rule Catalog

| ID | Rule | Type |
|----|------|------|
| BR-SEARCH-001 | Every query MUST be filtered by caller's `tenant_id`; query layer strips client overrides | CONSTRAINT |
| BR-SEARCH-002 | Results return aggregate type + id + highlighted matches; detail fetched from owning service | POLICY |
| BR-SEARCH-003 | SavedView scope: PRIVATE only readable by owner; TEAM readable by sharedWithPrincipalIds; TENANT readable by any principal with `tech.search:read` | POLICY |
| BR-SEARCH-004 | Reindex is admin-only; processes all events from offset 0 for a given aggregate type | POLICY |
| BR-SEARCH-005 | Max query result size: 10 000 records (deep pagination requires scroll API) | CONSTRAINT |
| BR-SEARCH-006 | Index schema upgrades use zero-downtime alias-swap pattern (see ADR-SEARCH-002) | POLICY |

---

## 5. Use Cases

| Endpoint | Use Case Type | Primary Aggregate |
|----------|--------------|------------------|
| `POST /search` | READ (query) | — |
| `GET /search/suggest` | READ | — |
| `POST /saved-views` | WRITE | SavedView |
| `GET /saved-views` | READ | SavedView |
| `GET /saved-views/{id}` | READ | SavedView |
| `PUT /saved-views/{id}` | WRITE | SavedView |
| `DELETE /saved-views/{id}` | WRITE | SavedView |
| `POST /admin/reindex/{aggregateType}` | WRITE (admin) | IndexProjector |
| `GET /admin/index-schemas` | READ (admin) | IndexSchema |
| `PUT /admin/index-schemas/{aggregateType}` | WRITE (admin) | IndexSchema |

---

## 6. REST API

**Base Path:** `/api/tech/search/v1`
**Authentication:** OAuth2 / JWT. Scopes `tech.search:read`, `tech.search:write`, `tech.search:admin`.

See `contracts/http/tech/search/openapi.yaml` for full payloads.

---

## 7. Events & Integration

### 7.1 Published Events

| Event | Routing Key |
|-------|------------|
| SavedViewCreated | `tech.search.saved-view.created` |
| SavedViewShared | `tech.search.saved-view.shared` |
| ReindexCompleted | `tech.search.reindex.completed` |
| IndexSchemaMigrated | `tech.search.index.schema-migrated` |

### 7.2 Consumed Events

Subscribes to domain events across suites (explicit registry in `IndexProjector`). Initial set: `crm.*`, `tks.*`, `sd.ord.*`, `fi.*`, `shared.bp.*`, `iam.principal.*`.

### 7.3 Routing-Key Bridge (Migration)

For 60 days after promotion, outbound keys also emitted under legacy `crm.search.*` for CRM consumers of `SavedView*` events.

---

## 8. Data Model

**Primary storage:** Elasticsearch 9.x. Indices per aggregate type, aliased for zero-downtime upgrades.
**Secondary (PostgreSQL):** schema `tech_search` — tables `saved_views`, `index_schemas`, `index_projectors` for state that needs ACID semantics + RLS.

---

## 9. Security & Compliance

### 9.1 Access Control

| Role | Access Level |
|------|-------------|
| `platform-admin` | Index schemas, reindex, projectors |
| `tenant-admin` | Manage tenant-scoped SavedViews |
| `tenant-user` | Query, own SavedViews |

### 9.2 Data Classification

- Index payloads inherit classification of the source aggregate. Confidential fields (PII) are masked in index via projector config.
- Tenant isolation at query layer MUST hold (BR-SEARCH-001).

### 9.3 Compliance

GDPR erasure propagates from source domain: deletion events (`*.deleted`, `iam.principal.principal.deleted`) trigger index deletion with max 60-second lag.

---

## 10. Quality Attributes

### 10.1 Performance Targets

| Operation | p95 | p99 |
|-----------|-----|-----|
| `POST /search` (result set ≤ 50) | < 150 ms | < 400 ms |
| `GET /search/suggest` (typeahead) | < 50 ms | < 120 ms |
| Event → indexed | < 2 s | < 10 s |

### 10.2 Availability

99.9 % — degraded mode returns cached typeahead if Elasticsearch unavailable.

### 10.3 Scalability

Elasticsearch cluster with ≥ 3 master, ≥ 3 data nodes per environment. Query layer stateless, horizontally scaled.

---

## 11. Feature Dependencies

| Feature ID | Feature Name |
|-----------|-------------|
| `F-TECH-SEARCH-001` | Global Search Bar |
| `F-TECH-SEARCH-002` | Advanced Search & Filters |
| `F-TECH-SEARCH-003` | Saved Views & Segments |
| `F-TECH-SEARCH-004` | Typeahead / Suggest |

Consumed by features from multiple suites (e.g., `F-TKS-320` KB Search & Suggest, `F-CRM-006-*`).

---

## 12. Extension Points

- New aggregate type indexed via `PUT /admin/index-schemas/{aggregateType}` + registering an `IndexProjector`.
- Custom analyzers registered per tenant via `POST /admin/index-schemas/analyzers`.
- Extension events `tech.search.ext.pre-index`, `tech.search.ext.post-index` for tenant-specific enrichment.

---

## 13. Migration & Evolution

### 13.1 Promotion from `crm-search-svc`

- Origin spec `T3_Domains/CRM/domain-specs/crm_search-spec.md` marked DEPRECATED.
- API path `/api/tech/search/v1`; `/api/crm/search/v1` returns HTTP 308 for 60 days.
- Elasticsearch indices renamed from `crm_*` to `<suite>_<domain>_*` via alias-swap; tenant boundary unchanged.
- CRM-specific IndexSchemas migrate unchanged (aggregate types already suite-qualified).
- Port stays 8406.

### 13.2 Post-Migration

Bridge removed after 60 days. Origin retired two minor versions later.

---

## 14. Decisions & Open Questions

| # | Question | Severity | Status |
|---|----------|----------|--------|
| Q-SEARCH-001 | Do we support vector / semantic search in v1 (shared infra with `tech.ai`)? | Medium | Open |
| Q-SEARCH-002 | Multi-region Elasticsearch for tenants with data-residency requirements? | Medium | Open |

### ADRs

- **ADR-SEARCH-001** *(Proposed)*: Promote to T1 (platform infrastructure). Rationale: search is a foundational capability consumed by every suite; moving it out of CRM ends the "CRM-prefixed but CRM-agnostic" ambiguity.
- **ADR-SEARCH-002** *(Proposed)*: Zero-downtime schema evolution via alias-swap (`_v1` / `_v2` indices behind a single alias).

---

## 15. Appendix

### 15.1 Glossary

See `T1_Platform/tech/_tech_suite.md` SS1.

### 15.2 Change Log

| Date | Version | Author | Changes |
|------|---------|--------|---------|
| 2026-04-18 | 1.0.0 | OpenLeap Architecture Team | Initial promotion spec from crm.search (supersedes crm-search-svc) |

### 15.3 Companion Files

- OpenAPI: `contracts/http/tech/search/openapi.yaml`
- Event Schemas: `contracts/events/tech/search/*.schema.json`
- Origin (deprecated): `T3_Domains/CRM/domain-specs/crm_search-spec.md`
