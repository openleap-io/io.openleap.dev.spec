<!-- Template Meta
     Template-ID:   TPL-SVC
     Version:       1.0.0
     Compliance:    95%+
-->

# Contact Service — Domain Specification

> **Conceptual Stack Layer:** Domain / Service
> **Space:** Platform
> **Owner:** Domain Engineering Team

> **Meta Information**
> - **Version:** 2026-04-03
> - **Template:** `domain-service-spec.md` v1.0.0
> - **Template Compliance:** 95%+
> - **Status:** DRAFT
> - **Service ID:** `crm-contact-svc`
> - **Suite:** `crm`
> - **Domain:** `contact`
> - **Base Package:** `com.openleap.crm.contact`
> - **API Path:** `/api/crm/contact/v1`
> - **Port:** 8401
> - **DB Schema:** `crm_contact`
> - **Tier:** Core

---

## 0. Document Purpose & Scope

This document specifies the **Contact Service** (`crm-contact-svc`) within the CRM Suite.
It defines the domain model, business rules, use cases, REST API, events, data model, security, and quality attributes for the `contact` bounded context.

**Audience:** Backend developers, frontend developers, QA engineers, architects, product owners.

**In scope:** All domain logic within the `Contact Management` bounded context.
**Out of scope:** UI implementation (see feature specs), infrastructure provisioning, cross-suite orchestration.

**Related documents:**
- Suite Spec: `_crm_suite.md`
- OpenAPI: `contracts/http/crm/contact/openapi.yaml`
- Event Schemas: `contracts/events/crm/contact/*.schema.json`

---

## 1. Business Context

### 1.1 Purpose & Responsibility

The Contact Service is the master data service for all person and organization records in the CRM Suite. It owns the lifecycle of Contacts (natural persons), Accounts (organizations/companies), and the Relationships between them. It acts as the central reference point for all other CRM services — Leads convert into Contacts, Opportunities reference Accounts, Activities link to Contacts.

**Owns:**
- Contact (Person) aggregate
- Account (Organization) aggregate
- ContactRelationship aggregate
- Duplicate detection & merge logic

**Does NOT own:**
- Business Partner master data (owned by bp suite — synced via events)
- Activities/Timeline (owned by crm.act)
- Emails (owned by crm.email)

### 1.2 Business Value

Single source of truth for customer identity. Every CRM interaction starts with knowing who the customer is. Without accurate, deduplicated contact and account data, the entire sales, marketing, and support pipeline breaks down.

### 1.3 Stakeholders

| Stakeholder | Interest |
|------------|---------|
| Sales Representatives | Access contact details, log interactions |
| Account Managers | Monitor account health, manage relationships |
| Marketing | Target contacts for campaigns |
| Support Agents | Identify ticket reporters |
| CRM Administrators | Configure custom fields, merge rules |

---

## 2. Service Identity

| Field | Value |
|-------|-------|
| Service ID | `crm-contact-svc` |
| Suite | `crm` |
| Domain | `contact` |
| Bounded Context | Contact Management |
| Base Package | `com.openleap.crm.contact` |
| API Base Path | `/api/crm/contact/v1` |
| Port | 8401 |
| Database Schema | `crm_contact` |
| Tier | Core |
| Repository | `openleap-crm-contact-svc` |
| Team | `team-crm` |

---

## 3. Domain Model

### 3.x Aggregate: Contact

A natural person the organization has a business relationship with. Most-accessed entity in the CRM.

```
Contact (AggregateRoot)
├── contactId                      : ContactID (UUID) — PK
├── tenantId                       : TenantID (UUID) — RLS
├── firstName                      : String — required, max 100
├── lastName                       : String — required, max 100
├── email                          : EmailAddress (VO) — unique per tenant when present
├── phone                          : PhoneNumber (VO) — optional
├── mobilePhone                    : PhoneNumber (VO) — optional
├── title                          : String — job title, max 200
├── department                     : String — max 200
├── accountId                      : AccountID (UUID) — FK — primary account
├── ownerId                        : UserID (UUID) — assigned sales rep
├── leadSource                     : LeadSource (enum) — WEB, REFERRAL, CAMPAIGN, COLD_CALL, PARTNER, EVENT, OTHER
├── lifecycleStage                 : LifecycleStage (enum) — SUBSCRIBER, LEAD, MQL, SQL, OPPORTUNITY, CUSTOMER, EVANGELIST, OTHER
├── status                         : ContactStatus (enum) — ACTIVE, INACTIVE, BLOCKED
├── address                        : Address (VO) — optional
├── dateOfBirth                    : LocalDate — optional
├── linkedInUrl                    : URL — optional
├── tags                           : Set<Tag> — user-defined tags
├── customFields                   : Map<String, Object> — tenant-configurable
├── doNotEmail                     : boolean — GDPR opt-out
├── doNotCall                      : boolean — opt-out
├── lastActivityAt                 : Instant — denormalized from crm.act events
├── createdAt                      : Instant — auto
├── updatedAt                      : Instant — auto
├── version                        : Long — optimistic locking
```

**Lifecycle:** ACTIVE → INACTIVE → BLOCKED (or back to ACTIVE). Soft-delete sets status=BLOCKED and anonymizes PII after retention period.

**Invariants:**
- Email must be unique per tenant (when non-null)
- firstName + lastName required
- If doNotEmail=true, no marketing emails may be sent
- accountId must reference an existing Account in the same tenant

### 3.x Aggregate: Account

An organization (company, institution, household) that is a current or prospective customer. Parent container for Contacts and Opportunities.

```
Account (AggregateRoot)
├── accountId                      : AccountID (UUID) — PK
├── tenantId                       : TenantID (UUID) — RLS
├── name                           : String — required, max 255
├── type                           : AccountType (enum) — PROSPECT, CUSTOMER, PARTNER, COMPETITOR, OTHER
├── industry                       : String — max 100
├── website                        : URL — optional
├── phone                          : PhoneNumber (VO) — main phone
├── billingAddress                 : Address (VO) — optional
├── shippingAddress                : Address (VO) — optional
├── annualRevenue                  : Money (VO) — estimated
├── employeeCount                  : Integer — optional
├── ownerId                        : UserID (UUID) — assigned account manager
├── parentAccountId                : AccountID (UUID) — FK — parent in hierarchy, nullable
├── tier                           : AccountTier (enum) — ENTERPRISE, MID_MARKET, SMB, STARTUP
├── tags                           : Set<Tag> — user-defined tags
├── customFields                   : Map<String, Object> — tenant-configurable
├── bpPartnerId                    : UUID — synced from BP suite, nullable
├── createdAt                      : Instant — auto
├── updatedAt                      : Instant — auto
├── version                        : Long — optimistic locking
```

**Lifecycle:** Created → Active → Archived. Accounts are never hard-deleted due to referential integrity with Opportunities and Quotes.

**Invariants:**
- Name must be unique per tenant
- parentAccountId must not create cycles
- If bpPartnerId is set, sync updates from BP suite must not be overridden manually

### 3.x Aggregate: ContactRelationship

Links a Contact to an Account with a specific role (e.g., Decision Maker, Influencer, End User). Supports multiple contacts per account and multiple accounts per contact.

```
ContactRelationship (AggregateRoot)
├── relationshipId                 : RelationshipID (UUID) — PK
├── tenantId                       : TenantID (UUID) — RLS
├── contactId                      : ContactID (UUID) — FK
├── accountId                      : AccountID (UUID) — FK
├── role                           : RelationshipRole (enum) — DECISION_MAKER, INFLUENCER, END_USER, BILLING, TECHNICAL, EXECUTIVE_SPONSOR, OTHER
├── isPrimary                      : boolean — is this the contact's primary account?
├── startDate                      : LocalDate — when relationship started
├── endDate                        : LocalDate — nullable — when relationship ended
├── notes                          : String — optional
├── createdAt                      : Instant — auto
```

**Lifecycle:** Created → Active (endDate null) → Ended (endDate set).

**Invariants:**
- contactId + accountId + role must be unique
- Exactly one relationship per contact must be marked isPrimary=true

---

## 4. Business Rules

### 4.1 Rule Catalog

| ID | Name | Type | Error Code |
|----|------|------|-----------|
| BR-CONTACT-001 | Unique Email Per Tenant | CONSTRAINT | `CONTACT_EMAIL_DUPLICATE` |
| BR-CONTACT-002 | Account Name Uniqueness | CONSTRAINT | `ACCOUNT_NAME_DUPLICATE` |
| BR-CONTACT-003 | Primary Relationship Required | INVARIANT | `CONTACT_NO_PRIMARY_ACCOUNT` |
| BR-CONTACT-004 | Do Not Email Enforcement | POLICY | `CONTACT_DO_NOT_EMAIL` |
| BR-CONTACT-005 | Duplicate Detection Threshold | POLICY | `N/A` |
| BR-CONTACT-006 | Merge Survivor Rule | POLICY | `N/A` |
| BR-CONTACT-007 | No Cyclic Account Hierarchy | CONSTRAINT | `ACCOUNT_CYCLIC_HIERARCHY` |
| BR-CONTACT-008 | BP Sync Conflict Resolution | POLICY | `N/A` |

**BR-CONTACT-001: Unique Email Per Tenant**
- **Type:** CONSTRAINT
- **Statement:** A contact's email address must be unique within the tenant. Two contacts cannot share the same email.
- **Error Code:** `CONTACT_EMAIL_DUPLICATE`
- **Error Message:** "Contact with this email already exists."

**BR-CONTACT-002: Account Name Uniqueness**
- **Type:** CONSTRAINT
- **Statement:** Account name must be unique within the tenant.
- **Error Code:** `ACCOUNT_NAME_DUPLICATE`
- **Error Message:** "Account with this name already exists."

**BR-CONTACT-003: Primary Relationship Required**
- **Type:** INVARIANT
- **Statement:** Every contact must have exactly one primary account relationship.
- **Error Code:** `CONTACT_NO_PRIMARY_ACCOUNT`
- **Error Message:** "Contact must have a primary account."

**BR-CONTACT-004: Do Not Email Enforcement**
- **Type:** POLICY
- **Statement:** If doNotEmail is true, the system must prevent marketing and bulk email sends to this contact. Transactional emails (password reset, order confirmation) are exempt.
- **Error Code:** `CONTACT_DO_NOT_EMAIL`
- **Error Message:** "Contact has opted out of email communication."

**BR-CONTACT-005: Duplicate Detection Threshold**
- **Type:** POLICY
- **Statement:** Duplicate detection runs on create/update. Contacts with >80% similarity score (based on name + email + phone) are flagged as potential duplicates.
- **Error Code:** `N/A`
- **Error Message:** "N/A — advisory, not blocking."

**BR-CONTACT-006: Merge Survivor Rule**
- **Type:** POLICY
- **Statement:** When merging duplicate contacts, the oldest record (by createdAt) is the survivor. All relationships, activities, and references are transferred to the survivor. Merged records are archived.
- **Error Code:** `N/A`
- **Error Message:** "N/A — process rule."

**BR-CONTACT-007: No Cyclic Account Hierarchy**
- **Type:** CONSTRAINT
- **Statement:** parentAccountId must not create circular references in the account hierarchy tree.
- **Error Code:** `ACCOUNT_CYCLIC_HIERARCHY`
- **Error Message:** "Cannot set parent — would create circular reference."

**BR-CONTACT-008: BP Sync Conflict Resolution**
- **Type:** POLICY
- **Statement:** When BP sends an update for a contact/account with bpPartnerId, CRM accepts the update for master data fields (name, address, tax ID). CRM-only fields (leadSource, lifecycleStage, tags) are preserved.
- **Error Code:** `N/A`
- **Error Message:** "N/A — integration policy."

---

## 5. Use Cases

### 5.1 Use Case Catalog

| ID | Type | Trigger | Aggregate | REST | Events |
|----|------|---------|-----------|------|--------|
| CreateContact | WRITE | REST | Contact | `POST /contacts` | ContactCreated |
| UpdateContact | WRITE | REST | Contact | `PUT /contacts/{id}` | ContactUpdated |
| DeleteContact | WRITE | REST | Contact | `DELETE /contacts/{id}` | ContactDeleted |
| GetContact | READ | REST | Contact | `GET /contacts/{id}` | — |
| ListContacts | READ | REST | Contact | `GET /contacts` | — |
| CreateAccount | WRITE | REST | Account | `POST /accounts` | AccountCreated |
| UpdateAccount | WRITE | REST | Account | `PUT /accounts/{id}` | AccountUpdated |
| DeleteAccount | WRITE | REST | Account | `DELETE /accounts/{id}` | AccountDeleted |
| GetAccount | READ | REST | Account | `GET /accounts/{id}` | — |
| ListAccounts | READ | REST | Account | `GET /accounts` | — |
| CreateRelationship | WRITE | REST | ContactRelationship | `POST /contacts/{contactId}/relationships` | RelationshipCreated |
| RemoveRelationship | WRITE | REST | ContactRelationship | `DELETE /contacts/{contactId}/relationships/{relId}` | RelationshipRemoved |
| DetectDuplicates | READ | REST | Contact | `GET /contacts/{id}/duplicates` | — |
| MergeContacts | WRITE | REST | Contact | `POST /contacts/merge` | ContactMerged |
| SyncFromBP | WRITE | Message | Contact/Account | `N/A (event-driven)` | ContactUpdated, AccountUpdated |

### 5.x UC: CreateContact

- **Type:** WRITE
- **Trigger:** REST
- **Aggregate:** Contact
- **Operation:** `Contact.create`
- **REST:** `POST /contacts`
- **Events:** ContactCreated
- **Description:** Creates a new contact record with validation and duplicate check.

### 5.x UC: UpdateContact

- **Type:** WRITE
- **Trigger:** REST
- **Aggregate:** Contact
- **Operation:** `Contact.update`
- **REST:** `PUT /contacts/{id}`
- **Events:** ContactUpdated
- **Description:** Updates contact fields. Triggers duplicate re-check if name or email changed.

### 5.x UC: DeleteContact

- **Type:** WRITE
- **Trigger:** REST
- **Aggregate:** Contact
- **Operation:** `Contact.softDelete`
- **REST:** `DELETE /contacts/{id}`
- **Events:** ContactDeleted
- **Description:** Soft-deletes a contact (sets status=BLOCKED). Hard-delete after retention period.

### 5.x UC: GetContact

- **Type:** READ
- **Trigger:** REST
- **Aggregate:** Contact
- **Operation:** `ContactQuery.findById`
- **REST:** `GET /contacts/{id}`
- **Description:** Retrieves a single contact by ID with optional includes (account, relationships, recent activities).

### 5.x UC: ListContacts

- **Type:** READ
- **Trigger:** REST
- **Aggregate:** Contact
- **Operation:** `ContactQuery.list`
- **REST:** `GET /contacts`
- **Description:** Lists contacts with pagination, sorting, and filtering (by account, owner, status, tags, date range).

### 5.x UC: CreateAccount

- **Type:** WRITE
- **Trigger:** REST
- **Aggregate:** Account
- **Operation:** `Account.create`
- **REST:** `POST /accounts`
- **Events:** AccountCreated
- **Description:** Creates a new account (organization) record.

### 5.x UC: UpdateAccount

- **Type:** WRITE
- **Trigger:** REST
- **Aggregate:** Account
- **Operation:** `Account.update`
- **REST:** `PUT /accounts/{id}`
- **Events:** AccountUpdated
- **Description:** Updates account fields.

### 5.x UC: DeleteAccount

- **Type:** WRITE
- **Trigger:** REST
- **Aggregate:** Account
- **Operation:** `Account.softDelete`
- **REST:** `DELETE /accounts/{id}`
- **Events:** AccountDeleted
- **Description:** Soft-deletes an account. Blocked if account has open opportunities.

### 5.x UC: GetAccount

- **Type:** READ
- **Trigger:** REST
- **Aggregate:** Account
- **Operation:** `AccountQuery.findById`
- **REST:** `GET /accounts/{id}`
- **Description:** Retrieves a single account by ID with optional includes (contacts, opportunities, hierarchy).

### 5.x UC: ListAccounts

- **Type:** READ
- **Trigger:** REST
- **Aggregate:** Account
- **Operation:** `AccountQuery.list`
- **REST:** `GET /accounts`
- **Description:** Lists accounts with pagination, sorting, and filtering.

### 5.x UC: CreateRelationship

- **Type:** WRITE
- **Trigger:** REST
- **Aggregate:** ContactRelationship
- **Operation:** `ContactRelationship.create`
- **REST:** `POST /contacts/{contactId}/relationships`
- **Events:** RelationshipCreated
- **Description:** Links a contact to an account with a role.

### 5.x UC: RemoveRelationship

- **Type:** WRITE
- **Trigger:** REST
- **Aggregate:** ContactRelationship
- **Operation:** `ContactRelationship.remove`
- **REST:** `DELETE /contacts/{contactId}/relationships/{relId}`
- **Events:** RelationshipRemoved
- **Description:** Removes a contact-account relationship.

### 5.x UC: DetectDuplicates

- **Type:** READ
- **Trigger:** REST
- **Aggregate:** Contact
- **Operation:** `DuplicateDetectionService.detect`
- **REST:** `GET /contacts/{id}/duplicates`
- **Description:** Finds potential duplicate contacts based on similarity scoring.

### 5.x UC: MergeContacts

- **Type:** WRITE
- **Trigger:** REST
- **Aggregate:** Contact
- **Operation:** `MergeService.merge`
- **REST:** `POST /contacts/merge`
- **Events:** ContactMerged
- **Description:** Merges two or more duplicate contacts into a survivor record.

### 5.x UC: SyncFromBP

- **Type:** WRITE
- **Trigger:** Message
- **Aggregate:** Contact/Account
- **Operation:** `BPSyncService.handlePartnerEvent`
- **REST:** `N/A (event-driven)`
- **Events:** ContactUpdated, AccountUpdated
- **Description:** Handles bp.partner.created/updated events to sync master data.

---

## 6. REST API

**Base Path:** `/api/crm/contact/v1`

**Authentication:** OAuth2/JWT (Bearer token)

### 6.1 Endpoint Summary

| Method | Path | Description | Required Scope |
|--------|------|-------------|---------------|
| `POST` | `/contacts` | Create contact | `crm.contact:write` |
| `GET` | `/contacts` | List contacts | `crm.contact:read` |
| `GET` | `/contacts/{id}` | Get contact | `crm.contact:read` |
| `PUT` | `/contacts/{id}` | Update contact | `crm.contact:write` |
| `DELETE` | `/contacts/{id}` | Delete contact | `crm.contact:write` |
| `GET` | `/contacts/{id}/duplicates` | Find duplicates | `crm.contact:read` |
| `POST` | `/contacts/merge` | Merge contacts | `crm.contact:admin` |
| `POST` | `/contacts/{contactId}/relationships` | Create relationship | `crm.contact:write` |
| `GET` | `/contacts/{contactId}/relationships` | List relationships | `crm.contact:read` |
| `DELETE` | `/contacts/{contactId}/relationships/{relId}` | Remove relationship | `crm.contact:write` |
| `POST` | `/accounts` | Create account | `crm.contact:write` |
| `GET` | `/accounts` | List accounts | `crm.contact:read` |
| `GET` | `/accounts/{id}` | Get account | `crm.contact:read` |
| `PUT` | `/accounts/{id}` | Update account | `crm.contact:write` |
| `DELETE` | `/accounts/{id}` | Delete account | `crm.contact:write` |
| `GET` | `/accounts/{id}/contacts` | List account contacts | `crm.contact:read` |
| `GET` | `/accounts/{id}/hierarchy` | Get account hierarchy | `crm.contact:read` |

> Full request/response examples in companion OpenAPI: `contracts/http/crm/contact/openapi.yaml`

---

## 7. Events & Integration

### 7.1 Published Events

| Event | Routing Key | Key Payload Fields |
|-------|------------|-------------------|
| ContactCreated | `crm.contact.contact.created` | contactId, firstName, lastName, email, accountId, ownerId |
| ContactUpdated | `crm.contact.contact.updated` | contactId, changedFields[], previousValues{} |
| ContactDeleted | `crm.contact.contact.deleted` | contactId |
| ContactMerged | `crm.contact.contact.merged` | survivorId, mergedIds[], fieldResolutions{} |
| AccountCreated | `crm.contact.account.created` | accountId, name, type, industry, ownerId |
| AccountUpdated | `crm.contact.account.updated` | accountId, changedFields[] |
| AccountDeleted | `crm.contact.account.deleted` | accountId |
| RelationshipCreated | `crm.contact.relationship.created` | contactId, accountId, role, isPrimary |
| RelationshipRemoved | `crm.contact.relationship.removed` | contactId, accountId |

### 7.2 Consumed Events

| Event | Source | Purpose |
|-------|--------|---------|
| `bp.partner.partner.created` | BP | Create or update local Contact/Account from BP master data |
| `bp.partner.partner.updated` | BP | Update local Contact/Account fields from BP sync |
| `iam.principal.principal.deactivated` | IAM | Reassign contacts owned by deactivated user |

---

## 8. Data Model

**Storage:** PostgreSQL 16, Schema: `crm_contact`

### 8.x Table: `contacts`

| Column | Type | Constraints |
|--------|------|------------|
| `contact_id` | `UUID` | PK, DEFAULT gen_random_uuid() |
| `tenant_id` | `UUID` | NOT NULL, RLS |
| `first_name` | `VARCHAR(100)` | NOT NULL |
| `last_name` | `VARCHAR(100)` | NOT NULL |
| `email` | `VARCHAR(255)` |  |
| `phone` | `VARCHAR(50)` |  |
| `mobile_phone` | `VARCHAR(50)` |  |
| `title` | `VARCHAR(200)` |  |
| `department` | `VARCHAR(200)` |  |
| `account_id` | `UUID` | FK → accounts |
| `owner_id` | `UUID` | NOT NULL |
| `lead_source` | `VARCHAR(50)` |  |
| `lifecycle_stage` | `VARCHAR(50)` | DEFAULT 'OTHER' |
| `status` | `VARCHAR(20)` | DEFAULT 'ACTIVE' |
| `do_not_email` | `BOOLEAN` | DEFAULT false |
| `do_not_call` | `BOOLEAN` | DEFAULT false |
| `date_of_birth` | `DATE` |  |
| `linkedin_url` | `VARCHAR(500)` |  |
| `last_activity_at` | `TIMESTAMPTZ` |  |
| `custom_fields` | `JSONB` | DEFAULT '{}' |
| `created_at` | `TIMESTAMPTZ` | DEFAULT now() |
| `updated_at` | `TIMESTAMPTZ` | DEFAULT now() |
| `version` | `BIGINT` | DEFAULT 1 |

**Indexes:**
- `UNIQUE(tenant_id, email) WHERE email IS NOT NULL`
- `INDEX(tenant_id, account_id)`
- `INDEX(tenant_id, owner_id)`
- `INDEX(tenant_id, status)`
- `INDEX(tenant_id, last_name, first_name)`
- `INDEX(tenant_id, last_activity_at DESC)`

### 8.x Table: `accounts`

| Column | Type | Constraints |
|--------|------|------------|
| `account_id` | `UUID` | PK, DEFAULT gen_random_uuid() |
| `tenant_id` | `UUID` | NOT NULL, RLS |
| `name` | `VARCHAR(255)` | NOT NULL |
| `type` | `VARCHAR(50)` | DEFAULT 'PROSPECT' |
| `industry` | `VARCHAR(100)` |  |
| `website` | `VARCHAR(500)` |  |
| `phone` | `VARCHAR(50)` |  |
| `annual_revenue_amount` | `NUMERIC(18,2)` |  |
| `annual_revenue_currency` | `VARCHAR(3)` |  |
| `employee_count` | `INTEGER` |  |
| `owner_id` | `UUID` | NOT NULL |
| `parent_account_id` | `UUID` | FK → accounts, nullable |
| `tier` | `VARCHAR(50)` |  |
| `bp_partner_id` | `UUID` | synced from BP suite |
| `custom_fields` | `JSONB` | DEFAULT '{}' |
| `created_at` | `TIMESTAMPTZ` | DEFAULT now() |
| `updated_at` | `TIMESTAMPTZ` | DEFAULT now() |
| `version` | `BIGINT` | DEFAULT 1 |

**Indexes:**
- `UNIQUE(tenant_id, name)`
- `INDEX(tenant_id, type)`
- `INDEX(tenant_id, owner_id)`
- `INDEX(tenant_id, parent_account_id)`
- `INDEX(tenant_id, bp_partner_id) WHERE bp_partner_id IS NOT NULL`

### 8.x Table: `addresses`

| Column | Type | Constraints |
|--------|------|------------|
| `address_id` | `UUID` | PK |
| `tenant_id` | `UUID` | NOT NULL, RLS |
| `entity_type` | `VARCHAR(20)` | CONTACT or ACCOUNT |
| `entity_id` | `UUID` | NOT NULL |
| `address_type` | `VARCHAR(20)` | BILLING, SHIPPING, MAILING, HOME |
| `street` | `VARCHAR(500)` |  |
| `city` | `VARCHAR(200)` |  |
| `state` | `VARCHAR(200)` |  |
| `postal_code` | `VARCHAR(20)` |  |
| `country` | `VARCHAR(3)` | ISO 3166 |

**Indexes:**
- `INDEX(tenant_id, entity_type, entity_id)`

### 8.x Table: `contact_relationships`

| Column | Type | Constraints |
|--------|------|------------|
| `relationship_id` | `UUID` | PK |
| `tenant_id` | `UUID` | NOT NULL, RLS |
| `contact_id` | `UUID` | FK → contacts, NOT NULL |
| `account_id` | `UUID` | FK → accounts, NOT NULL |
| `role` | `VARCHAR(50)` | NOT NULL |
| `is_primary` | `BOOLEAN` | DEFAULT false |
| `start_date` | `DATE` |  |
| `end_date` | `DATE` |  |
| `notes` | `TEXT` |  |
| `created_at` | `TIMESTAMPTZ` | DEFAULT now() |

**Indexes:**
- `UNIQUE(tenant_id, contact_id, account_id, role)`
- `INDEX(tenant_id, account_id)`

### 8.x Table: `tags`

| Column | Type | Constraints |
|--------|------|------------|
| `tag_id` | `UUID` | PK |
| `tenant_id` | `UUID` | NOT NULL, RLS |
| `entity_type` | `VARCHAR(20)` | CONTACT or ACCOUNT |
| `entity_id` | `UUID` | NOT NULL |
| `name` | `VARCHAR(100)` | NOT NULL |
| `color` | `VARCHAR(7)` | hex |

**Indexes:**
- `INDEX(tenant_id, entity_type, entity_id)`
- `UNIQUE(tenant_id, entity_type, entity_id, name)`

---

## 9. Security & Compliance

### 9.1 Data Classification

| Data | Classification | Rationale |
|------|---------------|-----------|
| Contact name, email, phone, address | **Confidential (PII)** | Personal data subject to GDPR |
| Account name, industry, revenue | **Internal** | Business data |
| Tags, custom fields | **Internal** | May contain PII — treated as Confidential when uncertain |

### 9.2 Role-Permission Matrix

| Role | Permissions |
|------|-----------|
| `crm-admin` | Full CRUD + merge + bulk operations + custom field config |
| `crm-sales-manager` | Full CRUD + view all contacts in team |
| `crm-sales-rep` | CRUD own contacts + read team contacts |
| `crm-marketing` | Read all contacts + update marketing fields (lifecycle, tags, doNotEmail) |
| `crm-support-agent` | Read contacts + update support-related fields |
| `crm-readonly` | Read-only access to contacts and accounts |

---

## 10. Quality Attributes

### 10.1 Performance Targets

| Endpoint | p95 | p99 |
|----------|-----|-----|
| `GET /contacts/{id}` | p95 < 50ms | p99 < 100ms |
| `GET /contacts (list)` | p95 < 150ms | p99 < 300ms |
| `POST /contacts` | p95 < 100ms | p99 < 200ms |
| `POST /contacts/merge` | p95 < 500ms | p99 < 1s |

### 10.2 Availability

**Target:** 99.9% (core service — highest priority)

### 10.3 Scalability

- Horizontal scaling via Kubernetes Deployment (2+ replicas)
- Stateless application tier — all state in PostgreSQL
- Connection pooling via HikariCP (max 20 connections per pod)

---

## 11. Feature Dependencies

| Feature ID | Feature Name | Endpoints Used |
|-----------|-------------|---------------|
| `F-CRM-001-01` | Contact List & Search | `GET /contacts, GET /accounts` |
| `F-CRM-001-02` | Contact Detail View | `GET /contacts/{id}, GET /contacts/{id}/relationships` |
| `F-CRM-001-03` | Contact Create/Edit Form | `POST /contacts, PUT /contacts/{id}` |
| `F-CRM-001-04` | Account List & Search | `GET /accounts` |
| `F-CRM-001-05` | Account Detail View | `GET /accounts/{id}, GET /accounts/{id}/contacts` |
| `F-CRM-001-06` | Account Create/Edit Form | `POST /accounts, PUT /accounts/{id}` |
| `F-CRM-001-07` | Relationship Management | `POST/DELETE /contacts/{id}/relationships` |
| `F-CRM-001-08` | Duplicate Detection & Merge | `GET /contacts/{id}/duplicates, POST /contacts/merge` |

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
| 1 | Should contact support field-level audit trail (not just aggregate-level)? | Medium | Open |
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

- OpenAPI: `contracts/http/crm/contact/openapi.yaml`
- Event Schemas: `contracts/events/crm/contact/*.schema.json`