<!-- Template Meta
     Template-ID:   TPL-SVC
     Version:       1.1.0
     Compliance:    95%+
-->

# Knowledge Base Service — Domain Specification

> **Conceptual Stack Layer:** Domain / Service
> **Space:** Platform
> **Owner:** team-tks

> **Meta Information**
> - **Version:** 2026-04-18
> - **Template:** `domain-service-spec.md` v1.1.0
> - **Template Compliance:** 95%+
> - **Status:** DRAFT
> - **Service ID:** `tks-kb-svc`
> - **Suite:** `tks`
> - **Domain:** `kb`
> - **Base Package:** `io.openleap.tks.kb`
> - **API Path:** `/api/tks/kb/v1`
> - **Port:** 8502
> - **DB Schema:** `tks_kb`
> - **Tier:** T3
> - **Bounded Context:** `bc:knowledge`

---

## 0. Document Purpose & Scope

This document specifies the **Knowledge Base Service** (`tks-kb-svc`): versioned articles, approval workflow, visibility scopes, self-service + agent-facing suggest-on-create.

**In scope:** `Article`, `ArticleVersion`, `Category`, `Tag`, `ReviewRequest`.
**Out of scope:** Search index (→ `tech.search` indexes published articles); AI summary / suggestion (→ `tech.ai`).

---

## 1. Business Context

### 1.1 Purpose & Responsibility

Own authoring, versioning, approval, and publishing of knowledge articles. Serve agents (all visibility) and end customers (CUSTOMER + PUBLIC) via tenant-scoped reads. Notify reviewers on submission; notify readers (via `shared.ntf`) on publish.

**Owns:** `Article`, `ArticleVersion`, `Category`, `Tag`, `ReviewRequest`.

### 1.2 Business Value

First-response deflection: when a ticket is being created, suggest relevant articles to reporter / agent, reducing ticket volume and resolution time.

---

## 2. Service Identity

| Field | Value |
|-------|-------|
| Service ID | `tks-kb-svc` |
| Suite | `tks` |
| Domain | `kb` |
| Bounded Context | `bc:knowledge` |
| Base Package | `io.openleap.tks.kb` |
| API Base Path | `/api/tks/kb/v1` |
| Port | 8502 |
| Database Schema | `tks_kb` |
| Tier | T3 |
| Repository | `openleap-io/io.openleap.tks.kb` |
| Team | `team-tks` |

---

## 3. Domain Model

### 3.1 Aggregates

**Article** (AggregateRoot)

| Attribute | Type | Notes |
|---|---|---|
| id | uuid | |
| tenantId | uuid | RLS |
| slug | string | unique per tenant |
| title | string | 1..255 |
| currentVersionId | uuid? | → `ArticleVersion` |
| categoryIds | uuid[] | |
| tagIds | uuid[] | |
| visibility | enum | `INTERNAL`,`CUSTOMER`,`PUBLIC` |
| ownerPrincipalId | uuid | author |
| status | enum | `DRAFT`,`REVIEW`,`APPROVED`,`PUBLISHED`,`ARCHIVED` |
| createdAt, publishedAt, archivedAt | timestamp | |

**ArticleVersion** (child)

| Attribute | Type | Notes |
|---|---|---|
| id | uuid | |
| articleId | uuid | FK |
| number | int | monotonically increasing per article |
| authorPrincipalId | uuid | |
| body | text | markdown |
| summary | string? | AI-suggested, tenant-editable |
| attachmentDmsIds | uuid[] | |
| reviewRequestId | uuid? | |
| reviewerPrincipalId | uuid? | |
| approvedAt | timestamp? | |
| createdAt | timestamp | immutable |

**Category** (AggregateRoot)
- id, tenantId, parentId?, name, slug, visibility.

**Tag** (AggregateRoot)
- id, tenantId, name (unique per tenant).

**ReviewRequest** (AggregateRoot)

| Attribute | Type | Notes |
|---|---|---|
| id | uuid | |
| tenantId | uuid | |
| articleVersionId | uuid | FK |
| requesterPrincipalId | uuid | |
| approverPrincipalId | uuid? | may be resolved via policy |
| status | enum | `OPEN`,`APPROVED`,`REJECTED`,`WITHDRAWN` |
| decisionReason | text? | |
| decidedAt | timestamp? | |

---

## 4. Business Rules

### 4.1 Rule Catalog

| ID | Rule | Type |
|----|------|------|
| BR-KB-001 | Publishing requires `ReviewRequest.status = APPROVED` for the current version | CONSTRAINT |
| BR-KB-002 | `PUBLIC` articles MUST NOT reference `INTERNAL` CIs in body; validator runs on submit | CONSTRAINT |
| BR-KB-003 | Archived articles remain readable for 5 years; hard-delete thereafter | POLICY |
| BR-KB-004 | A new version supersedes the current; previous versions kept forever for audit | CONSTRAINT |
| BR-KB-005 | Slug unique per tenant; slug change creates a permanent redirect entry | CONSTRAINT |
| BR-KB-006 | `CUSTOMER` visibility readable by authenticated tenant reporters; `PUBLIC` readable unauthenticated (per portal route); `INTERNAL` requires `tks.kb:read-internal` | CONSTRAINT |
| BR-KB-007 | Review approver MUST NOT be the version author (four-eyes) | CONSTRAINT |
| BR-KB-008 | Tenant isolation via RLS on `tenant_id` | CONSTRAINT |

---

## 5. Use Cases

| UC | Type | Trigger | Aggregate | Events | REST |
|----|------|---------|-----------|--------|------|
| CreateArticle | WRITE | REST | Article | `tks.kb.article.drafted` | `POST /articles` |
| UpdateDraft | WRITE | REST | ArticleVersion | `tks.kb.article.drafted` | `PATCH /articles/{id}` |
| SubmitForReview | WRITE | REST | ReviewRequest | `tks.kb.article.submitted`, `tks.kb.review.requested` | `POST /articles/{id}/submit` |
| ReviewArticle | WRITE | REST | ReviewRequest | `tks.kb.article.approved` \| `tks.kb.review.completed` | `POST /review-requests/{id}/decide` |
| PublishArticle | WRITE | REST (post-approve) | Article | `tks.kb.article.published` | `POST /articles/{id}/publish` |
| ArchiveArticle | WRITE | REST | Article | `tks.kb.article.archived` | `POST /articles/{id}/archive` |
| GetArticle | READ | REST | Article | — | `GET /articles/{id}` or `GET /articles/by-slug/{slug}` |
| ListArticles | READ | REST | Article | — | `GET /articles` (filters: category, tag, visibility, status) |
| SuggestOnCreate | READ | REST (from channels / agent UI) | — | — | `POST /articles/suggest` (uses `tech.search` + `tech.ai`) |
| SearchArticles | READ | REST | — | — | `GET /articles/search?q=...` (delegates to `tech.search`) |
| CreateCategory | WRITE | REST | Category | — | `POST /categories` |
| CreateTag | WRITE | REST | Tag | — | `POST /tags` |

---

## 6. REST API

**Base Path:** `/api/tks/kb/v1`
**Scopes:** `tks.kb:read`, `tks.kb:read-internal`, `tks.kb:write`, `tks.kb:review`, `tks.kb:admin`.

### 6.1 Endpoint Summary

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/articles` | Create draft |
| `GET` | `/articles` | List |
| `GET` | `/articles/{id}` | Read (current version) |
| `GET` | `/articles/by-slug/{slug}` | Read by slug |
| `GET` | `/articles/{id}/versions` | List versions |
| `GET` | `/articles/{id}/versions/{n}` | Read specific version |
| `PATCH` | `/articles/{id}` | Update draft (new version auto-created) |
| `POST` | `/articles/{id}/submit` | Submit for review |
| `POST` | `/articles/{id}/publish` | Publish (requires APPROVED) |
| `POST` | `/articles/{id}/archive` | Archive |
| `GET` | `/articles/search` | Search (delegates to `tech.search`) |
| `POST` | `/articles/suggest` | Suggest-on-create (agent + inbound flow) |
| `POST` | `/review-requests/{id}/decide` | Approve / reject |
| `GET` | `/review-requests` | List (for reviewer) |
| `POST` | `/categories` | Create |
| `GET` | `/categories` | List |
| `POST` | `/tags` | Create |
| `GET` | `/tags` | List |

---

## 7. Events & Integration

### 7.1 Pattern

`event_driven` (producer) + `sync_api` (consumer of `tech.search`, `tech.ai`).

### 7.2 Published Events

| Event | Routing Key |
|-------|------------|
| ArticleDrafted | `tks.kb.article.drafted` |
| ArticleSubmitted | `tks.kb.article.submitted` |
| ArticleApproved | `tks.kb.article.approved` |
| ArticlePublished | `tks.kb.article.published` |
| ArticleArchived | `tks.kb.article.archived` |
| ReviewRequested | `tks.kb.review.requested` |
| ReviewCompleted | `tks.kb.review.completed` |

### 7.3 Consumed Events

| Source | Routing Key | Purpose |
|---|---|---|
| `iam.principal` | `iam.principal.principal.deleted` | Reassign authored articles to tenant admin |
| `shared.bp` | `shared.bp.party.erased` | n/a (articles don't reference parties directly, only principals) |

---

## 8. Data Model

**Storage:** PostgreSQL 16, schema `tks_kb`.

Tables: `articles`, `article_versions`, `categories`, `tags`, `article_tag` (join), `article_category` (join), `review_requests`, `slug_redirects`, `outbox_events`.

Indexes: unique `(tenant_id, slug)` on articles; `(tenant_id, status, visibility)` filter; FTS trigger publishing to `tech.search` via outbox.

---

## 9. Security & Compliance

### 9.1 Access Control

| Role | Access Level |
|------|-------------|
| `kb-author` | CRUD own drafts |
| `kb-reviewer` | Decide on assigned reviews |
| `kb-admin` | Full CRUD, category/tag mgmt |
| `tenant-user` | Read per visibility rules |
| `public` | Read `PUBLIC` articles via portal |

### 9.2 Data Classification

- `PUBLIC` — Public.
- `CUSTOMER` — Internal (tenant + portal-scoped).
- `INTERNAL` — Confidential (agents only).

### 9.3 Compliance

- **GDPR**: articles do not store PII by policy; validator flags PII tokens in submission.
- **Accessibility**: markdown rendering MUST produce WCAG-2.2-AA output (delegated to UI layer; spec requires headings + alt-text presence in body validator).

### 9.4 DORA ICT Risk References

| Risk ID | Title | Treatment |
|---|---|---|
| `RISK-TKS-KB-001` | Public article leaks internal info | Mitigate: BR-KB-002 validator + four-eyes review |

### 9.5 SBOM

CycloneDX per build; 5-year retention.

---

## 10. Quality Attributes

### 10.1 Performance

| Operation | p95 | p99 |
|---|---|---|
| `GET /articles/{id}` | < 60 ms | < 150 ms |
| `GET /articles/search` | < 200 ms | < 500 ms |
| `POST /articles/suggest` | < 500 ms | < 1.5 s |

### 10.2 Availability

99.9 %. Read path cached via CDN for `PUBLIC`.

### 10.3 Scalability

Stateless; horizontal replicas; reads dominated by agent + portal.

### 10.5 SLI/SLO Reference

`SLO-TKS-KB-001`.

### 10.6 Resilience

| Aspect | Value |
|---|---|
| RTO | < 15 min |
| RPO | < 5 min |

---

## 11. Feature Dependencies

| Feature ID | Feature Name |
|---|---|
| `F-TKS-300` | Knowledge Articles |
| `F-TKS-310` | KB Versioning & Approval |
| `F-TKS-320` | KB Search & Suggest |

---

## 12. Extension Points

- Custom body validators via `POST /articles/extensions/validators`.
- Pre-publish extension hook `tks.kb.ext.pre-publish` (fail-closed) for tenant compliance.

---

## 13. Migration & Evolution

- Supersedes `crm.sup.KnowledgeArticle`. Migration mapping: `crm_support.knowledgearticles` → `tks_kb.articles` + first `article_version` row per record.
- v1.1 adds multi-locale article versions.

---

## 14. Decisions & Open Questions

| # | Question | Severity | Status |
|---|----------|----------|--------|
| Q-KB-001 | Do we bundle a WYSIWYG editor or stick with markdown in v1? | Medium | Open — plan markdown-only in v1 |
| Q-KB-002 | Are attached videos stored via `tech.dms` or external (YouTube/Vimeo)? | Low | Open |

### ADRs

- **ADR-KB-001** *(Proposed)*: Versioned, immutable `ArticleVersion`; current version pointer lives on `Article`.
- **ADR-KB-002** *(Proposed)*: Four-eyes approval enforced in BR-KB-007.

---

## 15. Appendix

### 15.1 Glossary

See `_tks_suite.md` SS1.

### 15.2 Change Log

| Date | Version | Author | Changes |
|------|---------|--------|---------|
| 2026-04-18 | 1.0.0 | OpenLeap Architecture Team | Initial KB Service spec |

### 15.3 Companion Files

- OpenAPI: `contracts/http/tks/kb/openapi.yaml`
- Event Schemas: `contracts/events/tks/kb/*.schema.json`
