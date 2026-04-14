<!-- Template Meta
     Template-ID:   TPL-SVC
     Version:       1.0.0
     Compliance:    95%+
-->

# Lead Service — Domain Specification

> **Conceptual Stack Layer:** Domain / Service
> **Space:** Platform
> **Owner:** Domain Engineering Team

> **Meta Information**
> - **Version:** 2026-04-03
> - **Template:** `domain-service-spec.md` v1.0.0
> - **Template Compliance:** 95%+
> - **Status:** DRAFT
> - **Service ID:** `crm-lead-svc`
> - **Suite:** `crm`
> - **Domain:** `lead`
> - **Base Package:** `com.openleap.crm.lead`
> - **API Path:** `/api/crm/lead/v1`
> - **Port:** 8402
> - **DB Schema:** `crm_lead`
> - **Tier:** Core

---

## 0. Document Purpose & Scope

This document specifies the **Lead Service** (`crm-lead-svc`) within the CRM Suite.
It defines the domain model, business rules, use cases, REST API, events, data model, security, and quality attributes for the `lead` bounded context.

**Audience:** Backend developers, frontend developers, QA engineers, architects, product owners.

**In scope:** All domain logic within the `Lead Management` bounded context.
**Out of scope:** UI implementation (see feature specs), infrastructure provisioning, cross-suite orchestration.

**Related documents:**
- Suite Spec: `_crm_suite.md`
- OpenAPI: `contracts/http/crm/lead/openapi.yaml`
- Event Schemas: `contracts/events/crm/lead/*.schema.json`

---

## 1. Business Context

### 1.1 Purpose & Responsibility

The Lead Service manages the lifecycle of unqualified prospects from initial capture through qualification, scoring, and conversion. It owns the pre-sales funnel — everything that happens before a prospect becomes a Contact and Opportunity. It provides lead scoring (rule-based and ML-based), assignment rules, and the conversion saga that promotes qualified leads into the sales pipeline.

**Owns:**
- Lead aggregate
- LeadScoringRule aggregate
- Lead assignment logic
- Lead conversion orchestration (via Temporal saga)
- Lead import/bulk operations

**Does NOT own:**
- Contact/Account records (created during conversion, owned by crm.contact)
- Opportunity records (created during conversion, owned by crm.opp)
- Marketing campaign membership (owned by crm.mkt)

### 1.2 Business Value

Ensures no prospect falls through the cracks. Automated scoring and assignment route the hottest leads to the right sales reps immediately, while nurturing sequences keep cooler leads engaged. Conversion tracking provides clear handoff accountability between marketing and sales.

### 1.3 Stakeholders

| Stakeholder | Interest |
|------------|---------|
| Sales Representatives | Work assigned leads, qualify, convert |
| Sales Managers | Monitor lead pipeline, assignment rules |
| Marketing | Track lead source ROI, manage qualification criteria |
| CRM Administrators | Configure scoring rules, assignment rules |

---

## 2. Service Identity

| Field | Value |
|-------|-------|
| Service ID | `crm-lead-svc` |
| Suite | `crm` |
| Domain | `lead` |
| Bounded Context | Lead Management |
| Base Package | `com.openleap.crm.lead` |
| API Base Path | `/api/crm/lead/v1` |
| Port | 8402 |
| Database Schema | `crm_lead` |
| Tier | Core |
| Repository | `openleap-crm-lead-svc` |
| Team | `team-crm` |

---

## 3. Domain Model

### 3.x Aggregate: Lead

An unqualified prospect that has expressed interest but has not yet been accepted by sales. Leads exist independently of Contacts and Accounts until conversion.

```
Lead (AggregateRoot)
├── leadId                         : LeadID (UUID) — PK
├── tenantId                       : TenantID (UUID) — RLS
├── firstName                      : String — required, max 100
├── lastName                       : String — required, max 100
├── email                          : String — required, unique per tenant
├── phone                          : String — optional
├── company                        : String — optional, max 255
├── jobTitle                       : String — optional
├── source                         : LeadSource (enum) — WEB_FORM, LANDING_PAGE, REFERRAL, COLD_CALL, EVENT, PARTNER, IMPORT, API, OTHER
├── status                         : LeadStatus (enum) — NEW, CONTACTED, QUALIFIED, UNQUALIFIED, CONVERTED
├── score                          : Integer — 0-100, computed by scoring rules
├── rating                         : LeadRating (enum) — HOT, WARM, COLD — derived from score
├── assigneeId                     : UserID (UUID) — assigned sales rep
├── campaignId                     : CampaignID (UUID) — originating campaign, nullable
├── convertedContactId             : ContactID (UUID) — set after conversion, nullable
├── convertedAccountId             : AccountID (UUID) — set after conversion, nullable
├── convertedOpportunityId         : OpportunityID (UUID) — set after conversion, nullable
├── convertedAt                    : Instant — nullable
├── description                    : String — optional notes
├── website                        : URL — optional
├── address                        : Address (VO) — optional
├── tags                           : Set<Tag> — user-defined tags
├── customFields                   : Map<String, Object> — tenant-configurable
├── lastActivityAt                 : Instant — denormalized
├── createdAt                      : Instant — auto
├── updatedAt                      : Instant — auto
├── version                        : Long — optimistic locking
```

**Lifecycle:** NEW → CONTACTED → QUALIFIED → CONVERTED (terminal) or NEW → CONTACTED → UNQUALIFIED (terminal). Only QUALIFIED leads can be converted.

**Invariants:**
- Email must be unique per tenant
- Conversion is irreversible (status=CONVERTED is terminal)
- Only leads with status=QUALIFIED can be converted
- Score must be 0-100

### 3.x Aggregate: LeadScoringRule

A configurable rule that contributes points to a lead's score based on demographic or behavioral criteria.

```
LeadScoringRule (AggregateRoot)
├── ruleId                         : ScoringRuleID (UUID) — PK
├── tenantId                       : TenantID (UUID) — RLS
├── name                           : String — required
├── category                       : ScoringCategory (enum) — DEMOGRAPHIC, BEHAVIORAL, FIRMOGRAPHIC
├── field                          : String — the lead field to evaluate
├── operator                       : ComparisonOperator (enum) — EQUALS, CONTAINS, GREATER_THAN, LESS_THAN, IN, EXISTS
├── value                          : String — comparison value
├── points                         : Integer — positive or negative points
├── isActive                       : boolean
├── createdAt                      : Instant — auto
```

**Lifecycle:** Created → Active → Inactive. Rules are never deleted, only deactivated.

**Invariants:**
- Points can be negative (penalty rules)
- At least one active rule must exist per tenant

---

## 4. Business Rules

### 4.1 Rule Catalog

| ID | Name | Type | Error Code |
|----|------|------|-----------|
| BR-LEAD-001 | Unique Email Per Tenant | CONSTRAINT | `LEAD_EMAIL_DUPLICATE` |
| BR-LEAD-002 | Conversion Precondition | INVARIANT | `LEAD_NOT_QUALIFIED` |
| BR-LEAD-003 | Conversion Irreversibility | INVARIANT | `LEAD_ALREADY_CONVERTED` |
| BR-LEAD-004 | Score Recalculation on Update | POLICY | `N/A` |
| BR-LEAD-005 | Auto-Assignment on Create | POLICY | `N/A` |
| BR-LEAD-006 | Rating Derivation | COMPUTATION | `N/A` |

**BR-LEAD-001: Unique Email Per Tenant**
- **Type:** CONSTRAINT
- **Statement:** Lead email must be unique within the tenant. If a lead is created with an email matching an existing Contact, the system warns but does not block.
- **Error Code:** `LEAD_EMAIL_DUPLICATE`
- **Error Message:** "Lead with this email already exists."

**BR-LEAD-002: Conversion Precondition**
- **Type:** INVARIANT
- **Statement:** Only leads with status QUALIFIED can be converted. Attempting to convert a NEW, CONTACTED, or UNQUALIFIED lead returns an error.
- **Error Code:** `LEAD_NOT_QUALIFIED`
- **Error Message:** "Lead must be in QUALIFIED status to convert."

**BR-LEAD-003: Conversion Irreversibility**
- **Type:** INVARIANT
- **Statement:** Once a lead is converted, the conversion cannot be undone. The lead record is archived with back-references to the created Contact, Account, and Opportunity.
- **Error Code:** `LEAD_ALREADY_CONVERTED`
- **Error Message:** "Lead has already been converted."

**BR-LEAD-004: Score Recalculation on Update**
- **Type:** POLICY
- **Statement:** When lead fields change, the scoring engine re-evaluates all active scoring rules and updates the lead's score and rating.
- **Error Code:** `N/A`
- **Error Message:** "N/A"

**BR-LEAD-005: Auto-Assignment on Create**
- **Type:** POLICY
- **Statement:** New leads are automatically assigned based on tenant-configured assignment rules (round-robin, territory, source-based). If no rules match, the lead is assigned to the default queue.
- **Error Code:** `N/A`
- **Error Message:** "N/A"

**BR-LEAD-006: Rating Derivation**
- **Type:** COMPUTATION
- **Statement:** Rating is derived from score: HOT (score >= 80), WARM (score 50-79), COLD (score < 50).
- **Error Code:** `N/A`
- **Error Message:** "N/A"

---

## 5. Use Cases

### 5.1 Use Case Catalog

| ID | Type | Trigger | Aggregate | REST | Events |
|----|------|---------|-----------|------|--------|
| CreateLead | WRITE | REST | Lead | `POST /leads` | LeadCreated, LeadAssigned |
| UpdateLead | WRITE | REST | Lead | `PUT /leads/{id}` | LeadUpdated, LeadScoreUpdated |
| QualifyLead | WRITE | REST | Lead | `POST /leads/{id}/qualify` | LeadQualified |
| DisqualifyLead | WRITE | REST | Lead | `POST /leads/{id}/disqualify` | LeadDisqualified |
| ConvertLead | WRITE | REST | Lead | `POST /leads/{id}/convert` | LeadConverted |
| GetLead | READ | REST | Lead | `GET /leads/{id}` | — |
| ListLeads | READ | REST | Lead | `GET /leads` | — |
| ImportLeads | WRITE | REST | Lead | `POST /leads/import` | LeadCreated (per lead) |
| AssignLead | WRITE | REST | Lead | `POST /leads/{id}/assign` | LeadAssigned |

### 5.x UC: CreateLead

- **Type:** WRITE
- **Trigger:** REST
- **Aggregate:** Lead
- **Operation:** `Lead.create`
- **REST:** `POST /leads`
- **Events:** LeadCreated, LeadAssigned
- **Description:** Captures a new lead. Runs scoring, duplicate detection, and auto-assignment.

### 5.x UC: UpdateLead

- **Type:** WRITE
- **Trigger:** REST
- **Aggregate:** Lead
- **Operation:** `Lead.update`
- **REST:** `PUT /leads/{id}`
- **Events:** LeadUpdated, LeadScoreUpdated
- **Description:** Updates lead fields. Triggers score recalculation.

### 5.x UC: QualifyLead

- **Type:** WRITE
- **Trigger:** REST
- **Aggregate:** Lead
- **Operation:** `Lead.qualify`
- **REST:** `POST /leads/{id}/qualify`
- **Events:** LeadQualified
- **Description:** Moves lead to QUALIFIED status. Validates qualification criteria.

### 5.x UC: DisqualifyLead

- **Type:** WRITE
- **Trigger:** REST
- **Aggregate:** Lead
- **Operation:** `Lead.disqualify`
- **REST:** `POST /leads/{id}/disqualify`
- **Events:** LeadDisqualified
- **Description:** Marks lead as UNQUALIFIED with a reason.

### 5.x UC: ConvertLead

- **Type:** WRITE
- **Trigger:** REST
- **Aggregate:** Lead
- **Operation:** `LeadConversionSaga.execute`
- **REST:** `POST /leads/{id}/convert`
- **Events:** LeadConverted
- **Description:** Temporal saga: creates Contact + Account + Opportunity from qualified lead.

### 5.x UC: GetLead

- **Type:** READ
- **Trigger:** REST
- **Aggregate:** Lead
- **Operation:** `LeadQuery.findById`
- **REST:** `GET /leads/{id}`
- **Description:** Retrieves a single lead by ID.

### 5.x UC: ListLeads

- **Type:** READ
- **Trigger:** REST
- **Aggregate:** Lead
- **Operation:** `LeadQuery.list`
- **REST:** `GET /leads`
- **Description:** Lists leads with pagination, sorting, filtering by status/source/score/assignee.

### 5.x UC: ImportLeads

- **Type:** WRITE
- **Trigger:** REST
- **Aggregate:** Lead
- **Operation:** `LeadImportService.import`
- **REST:** `POST /leads/import`
- **Events:** LeadCreated (per lead)
- **Description:** Bulk import from CSV/XLSX. Validates, deduplicates, scores, and assigns.

### 5.x UC: AssignLead

- **Type:** WRITE
- **Trigger:** REST
- **Aggregate:** Lead
- **Operation:** `Lead.assign`
- **REST:** `POST /leads/{id}/assign`
- **Events:** LeadAssigned
- **Description:** Manually assigns or reassigns a lead to a sales rep.

---

## 6. REST API

**Base Path:** `/api/crm/lead/v1`

**Authentication:** OAuth2/JWT (Bearer token)

### 6.1 Endpoint Summary

| Method | Path | Description | Required Scope |
|--------|------|-------------|---------------|
| `POST` | `/leads` | Create lead | `crm.lead:write` |
| `GET` | `/leads` | List leads | `crm.lead:read` |
| `GET` | `/leads/{id}` | Get lead | `crm.lead:read` |
| `PUT` | `/leads/{id}` | Update lead | `crm.lead:write` |
| `DELETE` | `/leads/{id}` | Delete lead | `crm.lead:write` |
| `POST` | `/leads/{id}/qualify` | Qualify lead | `crm.lead:write` |
| `POST` | `/leads/{id}/disqualify` | Disqualify lead | `crm.lead:write` |
| `POST` | `/leads/{id}/convert` | Convert lead | `crm.lead:write` |
| `POST` | `/leads/{id}/assign` | Assign lead | `crm.lead:write` |
| `POST` | `/leads/import` | Bulk import | `crm.lead:admin` |
| `GET` | `/leads/scoring-rules` | List scoring rules | `crm.lead:admin` |
| `POST` | `/leads/scoring-rules` | Create scoring rule | `crm.lead:admin` |
| `PUT` | `/leads/scoring-rules/{id}` | Update scoring rule | `crm.lead:admin` |

> Full request/response examples in companion OpenAPI: `contracts/http/crm/lead/openapi.yaml`

---

## 7. Events & Integration

### 7.1 Published Events

| Event | Routing Key | Key Payload Fields |
|-------|------------|-------------------|
| LeadCreated | `crm.lead.lead.created` | leadId, firstName, lastName, email, source, assigneeId |
| LeadUpdated | `crm.lead.lead.updated` | leadId, changedFields[] |
| LeadQualified | `crm.lead.lead.qualified` | leadId, score, qualifiedBy |
| LeadConverted | `crm.lead.lead.converted` | leadId, contactId, accountId, opportunityId |
| LeadDisqualified | `crm.lead.lead.disqualified` | leadId, reason |
| LeadScoreUpdated | `crm.lead.lead.score_updated` | leadId, oldScore, newScore, triggeredBy |
| LeadAssigned | `crm.lead.lead.assigned` | leadId, assigneeId, previousAssigneeId |

### 7.2 Consumed Events

| Event | Source | Purpose |
|-------|--------|---------|
| `crm.mkt.member.responded` | Marketing | Update lead score based on campaign engagement |

---

## 8. Data Model

**Storage:** PostgreSQL 16, Schema: `crm_lead`

### 8.x Table: `leads`

| Column | Type | Constraints |
|--------|------|------------|
| `lead_id` | `UUID` | PK |
| `tenant_id` | `UUID` | NOT NULL, RLS |
| `first_name` | `VARCHAR(100)` | NOT NULL |
| `last_name` | `VARCHAR(100)` | NOT NULL |
| `email` | `VARCHAR(255)` | NOT NULL |
| `phone` | `VARCHAR(50)` |  |
| `company` | `VARCHAR(255)` |  |
| `job_title` | `VARCHAR(200)` |  |
| `source` | `VARCHAR(50)` | NOT NULL |
| `status` | `VARCHAR(20)` | DEFAULT 'NEW' |
| `score` | `INTEGER` | DEFAULT 0 |
| `rating` | `VARCHAR(10)` | DEFAULT 'COLD' |
| `assignee_id` | `UUID` |  |
| `campaign_id` | `UUID` |  |
| `converted_contact_id` | `UUID` |  |
| `converted_account_id` | `UUID` |  |
| `converted_opportunity_id` | `UUID` |  |
| `converted_at` | `TIMESTAMPTZ` |  |
| `description` | `TEXT` |  |
| `website` | `VARCHAR(500)` |  |
| `last_activity_at` | `TIMESTAMPTZ` |  |
| `custom_fields` | `JSONB` | DEFAULT '{}' |
| `created_at` | `TIMESTAMPTZ` | DEFAULT now() |
| `updated_at` | `TIMESTAMPTZ` | DEFAULT now() |
| `version` | `BIGINT` | DEFAULT 1 |

**Indexes:**
- `UNIQUE(tenant_id, email)`
- `INDEX(tenant_id, status)`
- `INDEX(tenant_id, assignee_id)`
- `INDEX(tenant_id, source)`
- `INDEX(tenant_id, score DESC)`
- `INDEX(tenant_id, created_at DESC)`

### 8.x Table: `lead_scoring_rules`

| Column | Type | Constraints |
|--------|------|------------|
| `rule_id` | `UUID` | PK |
| `tenant_id` | `UUID` | NOT NULL, RLS |
| `name` | `VARCHAR(255)` | NOT NULL |
| `category` | `VARCHAR(50)` | NOT NULL |
| `field` | `VARCHAR(100)` | NOT NULL |
| `operator` | `VARCHAR(50)` | NOT NULL |
| `value` | `VARCHAR(500)` |  |
| `points` | `INTEGER` | NOT NULL |
| `is_active` | `BOOLEAN` | DEFAULT true |
| `created_at` | `TIMESTAMPTZ` | DEFAULT now() |

**Indexes:**
- `INDEX(tenant_id, is_active)`

---

## 9. Security & Compliance

### 9.1 Data Classification

| Data | Classification | Rationale |
|------|---------------|-----------|
| Lead name, email, phone | **Confidential (PII)** | Personal data subject to GDPR |
| Lead score, source | **Internal** | Business data |

### 9.2 Role-Permission Matrix

| Role | Permissions |
|------|-----------|
| `crm-admin` | Full CRUD + scoring rules + import + assignment rules |
| `crm-sales-manager` | Full CRUD + assign leads + view all leads |
| `crm-sales-rep` | CRUD assigned leads + qualify/convert + read all leads |
| `crm-marketing` | Create leads + read all leads + update source/campaign fields |
| `crm-readonly` | Read-only access |

---

## 10. Quality Attributes

### 10.1 Performance Targets

| Endpoint | p95 | p99 |
|----------|-----|-----|
| `GET /leads/{id}` | p95 < 50ms | p99 < 100ms |
| `GET /leads (list)` | p95 < 150ms | p99 < 300ms |
| `POST /leads/{id}/convert` | p95 < 2s | p99 < 5s (saga) |

### 10.2 Availability

**Target:** 99.9% (core service)

### 10.3 Scalability

- Horizontal scaling via Kubernetes Deployment (2+ replicas)
- Stateless application tier — all state in PostgreSQL
- Connection pooling via HikariCP (max 20 connections per pod)

---

## 11. Feature Dependencies

| Feature ID | Feature Name | Endpoints Used |
|-----------|-------------|---------------|
| `F-CRM-002-01` | Lead List & Pipeline View | `GET /leads` |
| `F-CRM-002-02` | Lead Detail View | `GET /leads/{id}` |
| `F-CRM-002-03` | Lead Create/Edit Form | `POST /leads, PUT /leads/{id}` |
| `F-CRM-002-04` | Lead Qualification & Scoring | `POST /leads/{id}/qualify, GET /leads/scoring-rules` |
| `F-CRM-002-05` | Lead Conversion Wizard | `POST /leads/{id}/convert` |
| `F-CRM-002-06` | Lead Import | `POST /leads/import` |

---

## 12. Extension Points

### 12.1 Extension Events

All domain events listed in §7 are available for extension services to consume.

### 12.2 Custom Fields

All primary aggregates support `customFields` (JSONB) for tenant-specific extensions without schema changes.

### 12.3 Webhook Hooks

The Integration Service (`crm.int`) can subscribe to any event from this service and forward it to external webhooks.

---

## 13. Migration & Evolution

### 13.1 Data Migration

- Flyway migration scripts in `src/main/resources/db/migration/`
- Naming convention: `V{version}__{description}.sql`
- All migrations must be backward-compatible (no column drops without deprecation period)

### 13.2 API Versioning

- URL path versioning: `/api/crm/{domain}/v1`
- Breaking changes require new version (v2) with 6-month deprecation of v1
- Non-breaking additions (new optional fields, new endpoints) allowed in current version

---

## 14. Decisions & Open Questions

### 14.1 Open Questions

| # | Question | Severity | Status |
|---|----------|----------|--------|
| 1 | Should lead support field-level audit trail (not just aggregate-level)? | Medium | Open |
| 2 | Should custom fields support computed/formula fields? | Low | Open |

---

## 15. Appendix

### 15.1 Glossary

See Suite Spec SS1 for the CRM Ubiquitous Language glossary.

### 15.2 Change Log

| Date | Version | Author | Changes |
|------|---------|--------|---------|
| 2026-04-03 | 1.0.0 | OpenLeap Architecture Team | Initial domain spec at 95%+ compliance |

### 15.3 Companion Files

- OpenAPI: `contracts/http/crm/lead/openapi.yaml`
- Event Schemas: `contracts/events/crm/lead/*.schema.json`