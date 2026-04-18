<!-- Template Meta
     Template-ID:   TPL-SUITE
     Version:       1.0.0
     Compliance:    100%
-->

# CRM Suite Specification

> **CAPABILITY MOVES (2026-04-18):** The following CRM-owned domains are being promoted to the platform tier because they are consumed cross-suite (see `T3_Domains/TKS/_tks_suite.md` ADR-TKS-005):
>
> - `crm.ntf` → `shared.ntf` (T2) — see `T2_SharedBusiness/domain-specs/shared_ntf-spec.md`
> - `crm.wf` → `shared.wf` (T2) — see `T2_SharedBusiness/domain-specs/shared_wf-spec.md`
> - `crm.search` → `tech.search` (T1) — see `T1_Platform/tech/domain-specs/tech_search-spec.md`
> - `crm.email` → `tech.email` (T1) — see `T1_Platform/tech/domain-specs/tech_email-spec.md`
> - `crm.sup` is superseded by `tks-tkt-svc` + `tks-kb-svc`; see its `§13 Migration`.
>
> Each origin spec is marked `DEPRECATED` in its meta header; routing-key bridges run for 60 days after the targets go live. New product compositions MUST NOT select the deprecated features; use the platform / TKS equivalents.

> **Conceptual Stack Layer:** Suite
> **Space:** Platform
> **Owner:** Domain Engineering Team
> **Schema alignment:** `suite-layer.schema.json`
> **Companion files:** `crm.catalog.uvl` (referenced in SS6)
> **Contains:** Domain/Service Specs, Platform-Feature Specs, Feature Catalog

> **Meta Information**
> - **Version:** 2026-04-03
> - **Template:** `suite-spec.md` v1.0.0
> - **Template Compliance:** 100% — fully compliant
> - **Author(s):** OpenLeap Architecture Team
> - **Status:** DRAFT
> - **Suite ID:** `crm` (pattern: `^[a-z]{2,4}$`)
> - **Suite Name:** Customer Relationship Management
> - **Description:** Comprehensive CRM capabilities for managing contacts, leads, opportunities, activities, and the full customer lifecycle across sales, marketing, and support.
> - **Semantic Version:** `1.0.0`
> - **Team:**
>   - Name: `team-crm`
>   - Email: `team-crm@openleap.io`
>   - Slack: `#team-crm`
> - **Bounded Contexts:** `bc:contact-management`, `bc:lead-management`, `bc:sales-pipeline`, `bc:activities`, `bc:product-pricing`, `bc:crm-search`, `bc:email-communication`, `bc:scheduling`, `bc:document-management`, `bc:reporting`, `bc:workflow-automation`, `bc:notifications`, `bc:integrations`, `bc:analytics`, `bc:marketing`, `bc:customer-support`, `bc:configure-price-quote`, `bc:customer-projects`

---

## Specification Guidelines

> **This specification MUST comply with the OpenLeap specification guidelines.**
>
> ### Non-Negotiables
> - Never invent facts. If required info is missing, add an **OPEN QUESTION** entry.
> - Preserve intent and decisions. Only change meaning when explicitly requested.
> - Keep the spec **self-contained**: no "see chat", no implicit context.
>
> ### Style Guide
> - Prefer short sentences and lists.
> - Use MUST/SHOULD/MAY for normative statements.
> - Keep terminology consistent with the Ubiquitous Language defined in SS1.
> - Avoid ambiguous words ("often", "maybe") unless explicitly noting uncertainty.

---

<!-- ═══════════════════════════════════════════════════════════════════
     SS0  SUITE IDENTITY & PURPOSE
     ═══════════════════════════════════════════════════════════════════ -->

## 0. Suite Identity & Purpose

### 0.1 Suite Identity

| Field | Value |
|-------|-------|
| id | `crm` |
| name | Customer Relationship Management |
| description | Manages the full customer lifecycle — from first contact through lead qualification, opportunity pursuit, deal closure, ongoing support, and account growth. Provides sales teams, marketing, and support agents with a unified 360° view of every customer interaction. |
| version | `1.0.0` |
| status | `DRAFT` |
| tier | T3 (Core Business Suite) |
| owner | `team-crm` |

### 0.2 Business Purpose

The CRM Suite exists to give every customer-facing role — sales representatives, account managers, marketing specialists, and support agents — a single platform for managing customer relationships end-to-end. It replaces fragmented spreadsheets, disconnected email threads, and siloed customer data with a unified system that captures every interaction, automates routine follow-ups, and surfaces actionable insights.

**Business value:**
- **Revenue acceleration:** Structured sales pipeline with stage-gated progression reduces deal cycle time by providing clear next-best-actions.
- **Customer retention:** 360° activity timeline ensures no customer interaction is lost when team members change; proactive alerts flag at-risk accounts.
- **Operational efficiency:** Workflow automation eliminates manual data entry, lead assignment, and follow-up scheduling; CPQ reduces quoting errors.
- **Data-driven decisions:** Built-in analytics and forecasting give management real-time visibility into pipeline health, conversion rates, and revenue projections.
- **Marketing alignment:** Integrated campaign management and lead nurturing close the gap between marketing-generated leads and sales-accepted opportunities.

### 0.3 In-Scope

- Contact and account master data management (persons and organizations)
- Lead capture, qualification, scoring, and conversion
- Opportunity pipeline management with stage progression and forecasting
- Activity tracking: tasks, meetings, calls, notes, and unified timeline
- Product catalog, price books, and discount rule management
- Configure-price-quote (CPQ) with approval workflows and PDF generation
- Email composition, template management, and open/click tracking
- Calendar synchronization with external providers (Google, Outlook)
- Document storage, template-based generation, and attachment to CRM records
- Workflow automation: assignment rules, escalation rules, and custom triggers
- Notification delivery across in-app, push, email, SMS, and webhook channels
- Dashboards, standard reports, custom report builder, and scheduled exports
- Sales forecasting, pipeline analytics, and predictive lead scoring
- Marketing campaign management, email campaigns, and lead nurturing
- Customer support: ticket management, SLA tracking, and knowledge base
- Customer project tracking and onboarding checklists
- Integration hub: webhooks, pre-built connectors, and third-party sync
- Global full-text search across all CRM entities with saved views and segments

### 0.4 Out-of-Scope

- **Business Partner master data ownership** — owned by `bp` suite; CRM syncs via events.
- **Financial transactions** — invoicing, payments, and accounting belong to `fi` suite.
- **Contract lifecycle management** — formal contract authoring and clause negotiation are out of scope; CRM only links to contracts.
- **HR / employee management** — internal employee data is managed by IAM and HR suites.
- **E-commerce storefronts** — public-facing product catalogs and shopping carts belong to `com` suite.
- **Project management beyond customer context** — internal project management belongs to `ps` suite; CRM covers only customer-facing projects and onboarding.
- **ERP integration orchestration** — owned by `dtp` suite; CRM publishes events that DTP consumes.

### 0.5 Target Users

| Role | Primary Use |
|------|-------------|
| Sales Representative | Daily pipeline management: create leads, progress opportunities, log activities, generate quotes |
| Account Manager | Relationship nurturing: 360° contact view, account health monitoring, renewal tracking |
| Sales Manager | Team oversight: pipeline dashboards, forecasting, win/loss analysis, assignment rules |
| Marketing Specialist | Campaign execution: lead nurturing, email campaigns, campaign ROI tracking |
| Support Agent | Ticket resolution: case management, SLA monitoring, knowledge base lookup |
| CRM Administrator | System configuration: workflows, notification rules, integrations, field customization |
| Executive / C-Level | Strategic decisions: revenue forecasts, pipeline health, conversion funnel analytics |

---

<!-- ═══════════════════════════════════════════════════════════════════
     SS1  UBIQUITOUS LANGUAGE
     ═══════════════════════════════════════════════════════════════════ -->

## 1. Ubiquitous Language

### 1.1 Glossary

| Term | Definition | Aliases |
|------|-----------|---------|
| Contact | A natural person with whom the organization has or seeks a business relationship. Contacts are always linked to at least one Account. The Contact aggregate is the most-accessed entity in the CRM. | Person, Individual |
| Account | An organization (company, institution, or household) that is a current or prospective customer. Accounts serve as the parent container for Contacts, Opportunities, and Support Tickets. | Company, Organization |
| Lead | An unqualified prospect that has expressed interest but has not yet been accepted by sales. Leads exist independently of Contacts and Accounts until conversion. | Prospect |
| Lead Conversion | The one-time process of promoting a qualified Lead into a Contact + Account + (optionally) Opportunity. Conversion is irreversible; the Lead record is archived with a back-reference. | Qualification Handoff |
| Opportunity | A potential revenue-generating deal linked to an Account. Opportunities progress through a configurable sequence of Stages and carry a weighted forecast value. | Deal |
| Stage | A named milestone in the Opportunity pipeline (e.g., Prospecting → Qualification → Proposal → Negotiation → Closed Won / Closed Lost). Stages have a sequence order, a default probability, and configurable entry criteria. | Pipeline Stage, Sales Stage |
| Activity | Any interaction with a Contact or Account: Task, Meeting, Call, Note, or Email. All activities appear on the unified Timeline. | Interaction, Touchpoint |
| Timeline | A reverse-chronological feed of all Activities, Events, and state changes for a given Contact, Account, Lead, or Opportunity. The Timeline is a read-only CQRS projection maintained by `crm.act`. | Activity Feed, History |
| Product | A good, service, or subscription offered to customers. Products have a type (PRODUCT, SERVICE, SUBSCRIPTION, BUNDLE), a list price, and belong to one or more Price Books. | Item, SKU |
| Price Book | A named collection of product prices. Each tenant has a default Price Book; additional Price Books support partner pricing, regional pricing, or promotional pricing. | Price List |
| Quote | A formal price proposal sent to a customer, containing line items derived from Products and a Price Book. Quotes follow an approval workflow and can be exported as PDF. | Proposal, Estimate |
| Campaign | A marketing initiative (email blast, event, webinar, ad campaign) designed to generate or nurture Leads. Campaigns track member lists, response status, and ROI. | Marketing Campaign |
| Ticket | A customer-reported issue or request tracked through to resolution. Tickets have priority, category, SLA targets, and an assigned agent. | Case, Support Request, Issue |
| SLA | Service Level Agreement — a set of time-based targets (first response, resolution) that apply to Tickets based on priority and customer tier. | Service Level |
| Workflow | An automated sequence of actions triggered by a CRM event or schedule. Workflows can send notifications, update fields, create tasks, or call external webhooks. | Automation Rule, Process Rule |
| Saved View | A reusable filter + column configuration persisted by `crm.search`. Saved Views can be private (user-scoped) or shared (team-scoped). | Segment, List View |
| Booking Link | A shareable URL that lets external parties book a meeting slot based on the CRM user's calendar availability. | Scheduling Link |

### 1.2 Boundary Test — CRM vs Related Suites

| CRM Term | Related Suite Term | Distinction |
|----------|-------------------|-------------|
| **Contact** (CRM) | **Business Partner** (BP) | A Contact is a CRM-specific view of a person enriched with sales context (lead source, lifecycle stage, last activity). A Business Partner is the canonical master-data record. CRM syncs from BP via `bp.partner.created` / `bp.partner.updated` events. CRM may create Contacts that do not yet exist in BP (e.g., cold outreach targets). |
| **Account** (CRM) | **Business Partner — Organization** (BP) | Same relationship as Contact ↔ BP but at the organization level. CRM Account adds sales-specific data (industry, annual revenue estimate, account tier). |
| **Opportunity** (CRM) | **Sales Order** (COM) | An Opportunity is a *potential* deal tracked for forecasting. A Sales Order is a *committed* commercial transaction. Opportunity closure MAY trigger Sales Order creation in COM via an integration event. |
| **Quote** (CRM) | **Sales Order** (COM) | A Quote is a proposal that may or may not be accepted. Quote acceptance MAY create a Sales Order in COM. |
| **Ticket** (CRM) | **Service Order** (OPS) | A Ticket is a customer-facing support case. A Service Order is an internal operational work item. Ticket resolution MAY trigger Service Order creation in OPS for field work. |
| **Product** (CRM) | **Article** (COM) | CRM Product is a simplified catalog entry for quoting and pricing. COM Article is the full commercial product with inventory, logistics, and marketplace attributes. CRM syncs product data from COM or maintains its own lightweight catalog. |
| **Campaign** (CRM) | *(no equivalent)* | Campaigns are CRM-native. No other suite manages marketing initiatives. |

---

<!-- ═══════════════════════════════════════════════════════════════════
     SS2  DOMAIN MODEL OVERVIEW
     ═══════════════════════════════════════════════════════════════════ -->

## 2. Domain Model Overview

### 2.1 Core Concepts

| Concept | Glossary Ref | Owning Service | Aggregate? | Notes |
|---------|-------------|----------------|-----------|-------|
| Contact | SS1 Contact | `crm.contact` | Yes — Aggregate Root | Most-accessed entity. Synced with BP. |
| Account | SS1 Account | `crm.contact` | Yes — Aggregate Root | Parent of Contacts and Opportunities. |
| ContactRelationship | — | `crm.contact` | Yes — Aggregate Root | Links Contacts to Accounts with role and date range. |
| Lead | SS1 Lead | `crm.lead` | Yes — Aggregate Root | Independent lifecycle until conversion. |
| LeadScoringRule | — | `crm.lead` | Yes — Aggregate Root | Configurable scoring criteria per tenant. |
| Opportunity | SS1 Opportunity | `crm.opp` | Yes — Aggregate Root | Carries line items and stage history. |
| OpportunityLineItem | — | `crm.opp` | Entity (child of Opportunity) | Product + quantity + price for a deal. |
| PipelineConfig | SS1 Stage | `crm.opp` | Yes — Aggregate Root | Defines stage sequence, probabilities, and entry criteria per tenant. |
| Task | SS1 Activity | `crm.act` | Yes — Aggregate Root | To-do with due date, priority, and assignee. |
| Meeting | SS1 Activity | `crm.act` | Yes — Aggregate Root | Scheduled interaction with attendees, location, and outcome. |
| Call | SS1 Activity | `crm.act` | Yes — Aggregate Root | Logged phone interaction with duration, direction, and outcome. |
| Note | SS1 Activity | `crm.act` | Yes — Aggregate Root | Free-text annotation attached to any CRM record. |
| Product | SS1 Product | `crm.prod` | Yes — Aggregate Root | Catalog entry with type, pricing, and tax class. |
| PriceBook | SS1 Price Book | `crm.prod` | Yes — Aggregate Root | Named collection of product prices. |
| DiscountRule | — | `crm.prod` | Yes — Aggregate Root | Conditional discount with type, value, and applicability rules. |
| EmailMessage | — | `crm.email` | Yes — Aggregate Root | Sent/received email with tracking metadata. |
| EmailTemplate | — | `crm.email` | Yes — Aggregate Root | Reusable template with merge fields and layout. |
| EmailAccount | — | `crm.email` | Yes — Aggregate Root | SMTP/IMAP connection configuration per user. |
| CalendarConnection | — | `crm.cal` | Yes — Aggregate Root | OAuth link to external calendar provider. |
| BookingLink | SS1 Booking Link | `crm.cal` | Yes — Aggregate Root | Shareable scheduling URL with availability rules. |
| Document | — | `crm.doc` | Yes — Aggregate Root | File metadata with MinIO storage reference. |
| DocumentTemplate | — | `crm.doc` | Yes — Aggregate Root | Merge-field template for PDF generation. |
| Dashboard | — | `crm.rpt` | Yes — Aggregate Root | User-configured layout of report widgets. |
| Report | — | `crm.rpt` | Yes — Aggregate Root | Data query definition with filters, grouping, and chart type. |
| Workflow | SS1 Workflow | `crm.wf` | Yes — Aggregate Root | Automation rule with trigger, conditions, and actions. |
| NotificationPreference | — | `crm.ntf` | Yes — Aggregate Root | Per-user channel preferences per event type. |
| Notification | — | `crm.ntf` | Yes — Aggregate Root | Delivered notification with read/unread state. |
| IntegrationConnection | — | `crm.int` | Yes — Aggregate Root | Webhook or connector configuration with OAuth tokens. |
| ForecastPeriod | — | `crm.ana` | Yes — Aggregate Root | Time-boxed revenue forecast with actual vs projected. |
| Campaign | SS1 Campaign | `crm.mkt` | Yes — Aggregate Root | Marketing initiative with member list and performance metrics. |
| CampaignMember | — | `crm.mkt` | Entity (child of Campaign) | Contact/Lead enrolled in a Campaign with response status. |
| Ticket | SS1 Ticket | `crm.sup` | Yes — Aggregate Root | Support case with priority, SLA, and resolution timeline. |
| SLAPolicy | SS1 SLA | `crm.sup` | Yes — Aggregate Root | Time-based response/resolution targets per priority. |
| KnowledgeArticle | — | `crm.sup` | Yes — Aggregate Root | Help article with categories, tags, and version history. |
| Quote | SS1 Quote | `crm.cpq` | Yes — Aggregate Root | Price proposal with line items, approval status, and PDF. |
| QuoteLineItem | — | `crm.cpq` | Entity (child of Quote) | Product + quantity + discount for a quote line. |
| ApprovalChain | — | `crm.cpq` | Yes — Aggregate Root | Configurable approval hierarchy for quotes above thresholds. |
| CustomerProject | — | `crm.prj` | Yes — Aggregate Root | Customer-facing project with milestones and tasks. |
| OnboardingChecklist | — | `crm.prj` | Yes — Aggregate Root | Template-based checklist for customer onboarding. |
| SavedView | SS1 Saved View | `crm.search` | Yes — Aggregate Root | Persisted filter + column set for search results. |

### 2.2 Shared Kernel Types

These value objects are shared across multiple CRM services:

| Type | Authoritative Attributes | Used By |
|------|-------------------------|---------|
| `Money` | `amount: BigDecimal`, `currency: CurrencyCode (ISO 4217)` | crm.opp, crm.prod, crm.cpq, crm.ana |
| `Address` | `street: String`, `city: String`, `state: String`, `postalCode: String`, `country: CountryCode (ISO 3166)` | crm.contact, crm.sup |
| `PhoneNumber` | `number: String`, `type: MOBILE/WORK/HOME/FAX`, `isPrimary: boolean` | crm.contact, crm.lead |
| `EmailAddress` | `address: String`, `type: WORK/PERSONAL`, `isPrimary: boolean` | crm.contact, crm.lead |
| `DateRange` | `from: LocalDate`, `to: LocalDate (nullable)` | crm.prod, crm.mkt, crm.cpq |
| `TenantId` | `value: UUID` | All services |
| `UserId` | `value: UUID` | All services |
| `CustomFields` | `fields: Map<String, Object>` | crm.contact, crm.lead, crm.opp, crm.sup |
| `Tag` | `name: String`, `color: String (hex)` | crm.contact, crm.lead, crm.opp |
| `Attachment` | `fileId: UUID`, `fileName: String`, `mimeType: String`, `sizeBytes: Long`, `storageUrl: String` | crm.email, crm.doc, crm.sup |

### 2.3 Bounded Context Map

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         CRM SUITE (crm)                                  │
│                                                                          │
│  ┌────────────┐    ┌────────────┐    ┌────────────┐    ┌────────────┐   │
│  │  contact   │───▶│    lead    │───▶│    opp     │◀───│   prod     │   │
│  │ (upstream) │    │(downstream)│    │(downstream)│    │ (upstream) │   │
│  └─────┬──────┘    └────────────┘    └─────┬──────┘    └────────────┘   │
│        │                                    │                            │
│        ▼                                    ▼                            │
│  ┌────────────┐    ┌────────────┐    ┌────────────┐    ┌────────────┐   │
│  │    act     │    │   search   │    │    cpq     │    │    doc     │   │
│  │ (consumer) │    │ (consumer) │    │(downstream)│    │ (consumer) │   │
│  └────────────┘    └────────────┘    └────────────┘    └────────────┘   │
│                                                                          │
│  ┌────────────┐    ┌────────────┐    ┌────────────┐    ┌────────────┐   │
│  │   email    │    │    cal     │    │     wf     │    │    ntf     │   │
│  │ (consumer) │    │ (consumer) │    │ (consumer) │    │ (consumer) │   │
│  └────────────┘    └────────────┘    └────────────┘    └────────────┘   │
│                                                                          │
│  ┌────────────┐    ┌────────────┐    ┌────────────┐    ┌────────────┐   │
│  │    rpt     │    │    ana     │    │    mkt     │    │    sup     │   │
│  │ (consumer) │    │ (consumer) │    │ (hybrid)   │    │ (hybrid)   │   │
│  └────────────┘    └────────────┘    └────────────┘    └────────────┘   │
│                                                                          │
│  ┌────────────┐    ┌────────────┐                                       │
│  │    int     │    │    prj     │                                       │
│  │ (consumer) │    │ (consumer) │                                       │
│  └────────────┘    └────────────┘                                       │
└─────────────────────────────────────────────────────────────────────────┘

Cross-suite relationships:
  crm.contact ←──ACL──→ bp (Business Partner) : Conformist — CRM conforms to BP's partner model
  crm.opp     ──event──→ com (Commerce)        : Published Language — opp.closed_won triggers order
  crm.cpq     ──event──→ com (Commerce)        : Published Language — quote.accepted triggers order
  crm.sup     ──event──→ ops (Operations)       : Published Language — ticket.escalated triggers service order
  crm.contact ←──event── iam (IAM)             : Conformist — principal events update CRM user records
  crm.prod    ←──ACL──→ com (Commerce)         : ACL — CRM translates COM articles into CRM products
```

**DDD Patterns used:**
- **Customer-Supplier:** `crm.contact` (upstream) → `crm.lead`, `crm.opp`, `crm.act` (downstream consumers)
- **Conformist:** CRM conforms to BP's Business Partner model for master data sync
- **Anti-Corruption Layer (ACL):** CRM translates COM product data into its own lightweight Product model
- **Published Language:** CRM publishes well-defined events consumed by COM, OPS, and DTP
- **Shared Kernel:** Money, Address, PhoneNumber shared across CRM services (defined in `crm-shared-kernel`)


---

<!-- ═══════════════════════════════════════════════════════════════════
     SS3  SERVICE LANDSCAPE
     ═══════════════════════════════════════════════════════════════════ -->

## 3. Service Landscape

### 3.1 Service Catalog

#### 3.1.1 Core Services (6) — Mandatory, tightly integrated

| # | Service Name | Service ID | Domain | Port | DB Schema | Status | Spec Reference |
|---|-------------|-----------|--------|------|-----------|--------|---------------|
| 1 | Contact Service | `crm-contact-svc` | `contact` | 8401 | `crm_contact` | `draft` | `domain-specs/crm_contact-spec.md` |
| 2 | Lead Service | `crm-lead-svc` | `lead` | 8402 | `crm_lead` | `draft` | `domain-specs/crm_lead-spec.md` |
| 3 | Opportunity Service | `crm-opp-svc` | `opp` | 8403 | `crm_opportunity` | `draft` | `domain-specs/crm_opp-spec.md` |
| 4 | Activity Service | `crm-act-svc` | `act` | 8404 | `crm_activity` | `draft` | `domain-specs/crm_act-spec.md` |
| 5 | Product Catalog Service | `crm-prod-svc` | `prod` | 8405 | `crm_product` | `draft` | `domain-specs/crm_prod-spec.md` |
| 6 | Search Service | `crm-search-svc` | `search` | 8406 | Elasticsearch | `draft` | `domain-specs/crm_search-spec.md` |

#### 3.1.2 Extension Services (12) — Optional, event-driven, independently deployable

| # | Service Name | Service ID | Domain | Port | DB Schema | Status | Spec Reference |
|---|-------------|-----------|--------|------|-----------|--------|---------------|
| 7 | Email Service | `crm-email-svc` | `email` | 8411 | `crm_email` | `draft` | `domain-specs/crm_email-spec.md` |
| 8 | Calendar Service | `crm-cal-svc` | `cal` | 8412 | `crm_calendar` | `draft` | `domain-specs/crm_cal-spec.md` |
| 9 | Document Service | `crm-doc-svc` | `doc` | 8413 | `crm_document` + MinIO | `draft` | `domain-specs/crm_doc-spec.md` |
| 10 | Reporting Service | `crm-rpt-svc` | `rpt` | 8414 | `crm_reporting` | `draft` | `domain-specs/crm_rpt-spec.md` |
| 11 | Workflow Service | `crm-wf-svc` | `wf` | 8415 | `crm_workflow` | `draft` | `domain-specs/crm_wf-spec.md` |
| 12 | Notification Service | `crm-ntf-svc` | `ntf` | 8416 | `crm_notification` | `draft` | `domain-specs/crm_ntf-spec.md` |
| 13 | Integration Service | `crm-int-svc` | `int` | 8417 | `crm_integration` | `draft` | `domain-specs/crm_int-spec.md` |
| 14 | Analytics Service | `crm-ana-svc` | `ana` | 8418 | `crm_analytics` | `draft` | `domain-specs/crm_ana-spec.md` |
| 15 | Marketing Service | `crm-mkt-svc` | `mkt` | 8419 | `crm_marketing` | `draft` | `domain-specs/crm_mkt-spec.md` |
| 16 | Support Service | `crm-sup-svc` | `sup` | 8420 | `crm_support` | `draft` | `domain-specs/crm_sup-spec.md` |
| 17 | CPQ Service | `crm-cpq-svc` | `cpq` | 8421 | `crm_cpq` | `draft` | `domain-specs/crm_cpq-spec.md` |
| 18 | Project Service | `crm-prj-svc` | `prj` | 8422 | `crm_project` | `draft` | `domain-specs/crm_prj-spec.md` |

### 3.2 Service Tiers

| Tier | Services | Deployment | Scaling |
|------|----------|-----------|---------|
| **Core** (always deployed) | contact, lead, opp, act, prod, search | Kubernetes StatefulSet (search) / Deployment (rest) | Horizontal — 2+ replicas per service |
| **Extension** (opt-in per tenant) | email, cal, doc, rpt, wf, ntf, int, ana, mkt, sup, cpq, prj | Kubernetes Deployment, feature-gated | Scale-to-zero when disabled for tenant |

### 3.3 Technology Stack

| Concern | Technology | Notes |
|---------|-----------|-------|
| Runtime | Spring Boot 3.x, Java 21 | Hexagonal architecture (ports & adapters) |
| Database | PostgreSQL 16 with RLS | Schema-per-service, Row-Level Security |
| Search | Elasticsearch 8.x | CQRS read model for `crm.search` |
| Messaging | RabbitMQ 3.13 | Topic exchange per suite: `crm.events` |
| Object Storage | MinIO | Documents, email attachments, quote PDFs |
| Auth | Keycloak + OPAL | OAuth2/OIDC, fine-grained authorization |
| Orchestration | Temporal | Lead conversion saga, quote approval, campaign execution |
| API Gateway | Kong / Spring Cloud Gateway | Rate limiting, auth propagation, routing |
| Observability | Prometheus + Grafana + Jaeger | Metrics, dashboards, distributed tracing |

---

<!-- ═══════════════════════════════════════════════════════════════════
     SS4  INTEGRATION PATTERNS
     ═══════════════════════════════════════════════════════════════════ -->

## 4. Integration Patterns

### 4.1 Intra-Suite Communication Strategy

**Decision:** CRM uses a **hybrid** integration approach — synchronous REST for queries and commands that require immediate consistency, asynchronous events for eventual consistency and cross-service notifications.

**Rationale:** Core sales workflows (lead conversion, quote creation) require immediate feedback to the user. Extension services (search indexing, analytics aggregation, notification delivery) tolerate eventual consistency and benefit from loose coupling.

| Pattern | When Used | Examples |
|---------|-----------|---------|
| **REST (sync)** | Cross-aggregate queries, commands requiring immediate response | Lead conversion → create Contact + Opportunity; Quote → fetch Product pricing |
| **Events (async)** | State change notifications, CQRS projections, cross-service side effects | All aggregate state changes → search index; Activity created → timeline projection |
| **Saga (Temporal)** | Multi-step workflows with compensation | Lead conversion (3-step), quote approval (multi-level), campaign execution |

### 4.2 Communication Matrix

| Source | Target | Method | Purpose |
|--------|--------|--------|---------|
| `crm.lead` | `crm.contact` | REST (sync) | Lead conversion → create Contact |
| `crm.lead` | `crm.opp` | REST (sync) | Lead conversion → create Opportunity |
| `crm.opp` | `crm.prod` | REST (sync) | Fetch product/pricing for line items |
| `crm.cpq` | `crm.prod` | REST (sync) | Fetch product catalog for quotes |
| `crm.cpq` | `crm.opp` | REST (sync) | Link quote to opportunity |
| `crm.contact` | `bp-svc` | REST (sync) | Sync with Business Partner master data |
| `crm.mkt` | `crm.contact` | REST (sync) | Resolve campaign members to contacts |
| `crm.mkt` | `crm.lead` | REST (sync) | Resolve campaign members to leads |
| `crm.sup` | `crm.contact` | REST (sync) | Resolve ticket reporter to contact |
| `crm.*` | `crm.search` | Event (async) | Index updates via RabbitMQ |
| `crm.*` | `crm.act` | Event (async) | Activity timeline projection |
| `crm.*` | `crm.wf` | Event (async) | Workflow trigger evaluation |
| `crm.*` | `crm.ntf` | Event (async) | Notification delivery |
| `crm.*` | `crm.rpt` | Event (async) | Reporting data aggregation |
| `crm.*` | `crm.ana` | Event (async) | Analytics data collection |
| `crm.*` | `crm.int` | Event (async) | Webhook & connector forwarding |

### 4.3 Event Flows — Key Workflows

#### 4.3.1 Lead Conversion Saga (Temporal)

```
crm.lead                     crm.contact                   crm.opp
   │                              │                            │
   │  POST /contacts              │                            │
   │─────────────────────────────▶│                            │
   │  201 Created (contactId)     │                            │
   │◀─────────────────────────────│                            │
   │                              │                            │
   │  POST /opportunities         │                            │
   │──────────────────────────────┼───────────────────────────▶│
   │  201 Created (oppId)         │                            │
   │◀─────────────────────────────┼────────────────────────────│
   │                              │                            │
   │  Mark Lead CONVERTED         │                            │
   │  Publish lead.converted      │                            │
   │──────event──────────────────▶│──────event────────────────▶│
   │                        (index update)              (index update)
```

**Compensation:** If Opportunity creation fails, the Contact created in step 1 is NOT rolled back (Contact is still valid). The Lead remains in QUALIFIED state for retry. If Contact creation fails, the entire saga aborts and the Lead stays in its current state.

#### 4.3.2 Quote Approval Workflow

```
crm.cpq                      crm.wf                       crm.ntf
   │                              │                            │
   │  quote.submitted             │                            │
   │──────event──────────────────▶│                            │
   │                              │  Evaluate approval chain   │
   │                              │  Notify approvers          │
   │                              │──────event────────────────▶│
   │                              │                            │  (email/push)
   │  quote.approved / rejected   │                            │
   │◀─────event───────────────────│                            │
   │                              │                            │
   │  Generate PDF, update status │                            │
```

#### 4.3.3 Activity Timeline Projection

```
crm.contact / crm.lead / crm.opp / crm.email / crm.sup
   │
   │  *.*.{aggregate}.{action}
   │──────event──────────────────▶ crm.act (Timeline Projector)
                                      │
                                      │  Denormalize into timeline entry
                                      │  Store in crm_activity.timeline_events
                                      │
                                      │  timeline.entry.created
                                      │──────event──────────────────▶ crm.search
                                                                        (index)
```

---

<!-- ═══════════════════════════════════════════════════════════════════
     SS5  EVENT ARCHITECTURE
     ═══════════════════════════════════════════════════════════════════ -->

## 5. Event Architecture

### 5.1 Routing Key Pattern

```
{suite}.{domain}.{aggregate}.{action}
```

| Segment | Pattern | Example |
|---------|---------|---------|
| `suite` | `crm` (always) | `crm` |
| `domain` | lowercase domain short code | `contact`, `lead`, `opp` |
| `aggregate` | lowercase aggregate name | `contact`, `lead`, `opportunity` |
| `action` | past-tense verb | `created`, `updated`, `converted`, `closed_won` |

### 5.2 Payload Envelope

All CRM events follow the platform-standard envelope:

```json
{
  "eventId": "uuid-v4",
  "eventType": "crm.{domain}.{aggregate}.{action}",
  "timestamp": "ISO-8601",
  "version": 1,
  "source": "crm-{domain}-svc",
  "tenantId": "uuid",
  "correlationId": "uuid-v4",
  "causationId": "uuid-v4",
  "userId": "uuid",
  "schemaVersion": "1.0.0",
  "payload": { },
  "metadata": {
    "traceId": "string",
    "spanId": "string"
  }
}
```

### 5.3 Exchange & Queue Topology

| Exchange | Type | Routing | Bound Queues |
|----------|------|---------|-------------|
| `crm.events` | Topic | `crm.{domain}.{aggregate}.{action}` | Per-consumer queues with binding patterns |
| `crm.deadletter` | Fanout | — | `crm.deadletter.queue` |

**Consumer binding examples:**
- `crm.search`: `crm.#` (all CRM events for indexing)
- `crm.act`: `crm.*.*.created`, `crm.*.*.updated`, `crm.*.*.deleted` (timeline projection)
- `crm.ntf`: Selective bindings per configured notification rules
- `crm.wf`: Selective bindings per workflow trigger definitions
- `crm.ana`: `crm.#` (all events for analytics aggregation)

### 5.4 Event Catalog

#### Contact Service (`crm.contact`)

| Event | Routing Key | Trigger | Key Payload Fields |
|-------|------------|---------|-------------------|
| ContactCreated | `crm.contact.contact.created` | New contact record | contactId, firstName, lastName, email, accountId |
| ContactUpdated | `crm.contact.contact.updated` | Contact fields changed | contactId, changedFields[] |
| ContactDeleted | `crm.contact.contact.deleted` | Contact soft-deleted | contactId |
| ContactMerged | `crm.contact.contact.merged` | Duplicate merge completed | survivorId, mergedIds[], fieldResolutions |
| AccountCreated | `crm.contact.account.created` | New account record | accountId, name, industry, type |
| AccountUpdated | `crm.contact.account.updated` | Account fields changed | accountId, changedFields[] |
| AccountDeleted | `crm.contact.account.deleted` | Account soft-deleted | accountId |
| RelationshipCreated | `crm.contact.relationship.created` | Contact linked to account | contactId, accountId, role |
| RelationshipRemoved | `crm.contact.relationship.removed` | Contact unlinked from account | contactId, accountId |

#### Lead Service (`crm.lead`)

| Event | Routing Key | Trigger | Key Payload Fields |
|-------|------------|---------|-------------------|
| LeadCreated | `crm.lead.lead.created` | New lead captured | leadId, firstName, lastName, email, source |
| LeadUpdated | `crm.lead.lead.updated` | Lead fields changed | leadId, changedFields[] |
| LeadQualified | `crm.lead.lead.qualified` | Lead meets qualification criteria | leadId, score, qualifiedBy |
| LeadConverted | `crm.lead.lead.converted` | Lead converted to Contact+Opp | leadId, contactId, accountId, opportunityId |
| LeadDisqualified | `crm.lead.lead.disqualified` | Lead marked as not viable | leadId, reason |
| LeadScoreUpdated | `crm.lead.lead.score_updated` | Scoring rule changed lead score | leadId, oldScore, newScore, triggeredBy |
| LeadAssigned | `crm.lead.lead.assigned` | Lead assigned to sales rep | leadId, assigneeId, previousAssigneeId |

#### Opportunity Service (`crm.opp`)

| Event | Routing Key | Trigger | Key Payload Fields |
|-------|------------|---------|-------------------|
| OpportunityCreated | `crm.opp.opportunity.created` | New opportunity opened | opportunityId, accountId, name, amount, stage |
| OpportunityUpdated | `crm.opp.opportunity.updated` | Opportunity fields changed | opportunityId, changedFields[] |
| OpportunityStageChanged | `crm.opp.opportunity.stage_changed` | Stage progression | opportunityId, fromStage, toStage, probability |
| OpportunityClosedWon | `crm.opp.opportunity.closed_won` | Deal won | opportunityId, accountId, amount, closedDate |
| OpportunityClosedLost | `crm.opp.opportunity.closed_lost` | Deal lost | opportunityId, accountId, lostReason, competitorId |
| OpportunityAmountChanged | `crm.opp.opportunity.amount_changed` | Deal value updated | opportunityId, oldAmount, newAmount |
| LineItemAdded | `crm.opp.lineitem.added` | Product added to deal | opportunityId, lineItemId, productId, quantity, unitPrice |
| LineItemRemoved | `crm.opp.lineitem.removed` | Product removed from deal | opportunityId, lineItemId |

#### Activity Service (`crm.act`)

| Event | Routing Key | Trigger | Key Payload Fields |
|-------|------------|---------|-------------------|
| TaskCreated | `crm.act.task.created` | New task | taskId, subject, dueDate, assigneeId, relatedTo |
| TaskCompleted | `crm.act.task.completed` | Task marked done | taskId, completedAt, completedBy |
| TaskOverdue | `crm.act.task.overdue` | Task passed due date | taskId, dueDate, assigneeId |
| MeetingScheduled | `crm.act.meeting.scheduled` | Meeting created | meetingId, subject, startTime, attendeeIds[] |
| MeetingCompleted | `crm.act.meeting.completed` | Meeting finished | meetingId, outcome, notes |
| MeetingCancelled | `crm.act.meeting.cancelled` | Meeting cancelled | meetingId, reason |
| CallLogged | `crm.act.call.logged` | Call recorded | callId, direction, duration, outcome, contactId |
| NoteAdded | `crm.act.note.added` | Note created | noteId, body, relatedTo |

#### Product Catalog Service (`crm.prod`)

| Event | Routing Key | Trigger | Key Payload Fields |
|-------|------------|---------|-------------------|
| ProductCreated | `crm.prod.product.created` | New product | productId, name, code, type, listPrice |
| ProductUpdated | `crm.prod.product.updated` | Product changed | productId, changedFields[] |
| ProductDeactivated | `crm.prod.product.deactivated` | Product disabled | productId |
| PriceBookCreated | `crm.prod.pricebook.created` | New price book | priceBookId, name, isDefault |
| PriceBookEntryUpdated | `crm.prod.pricebook.entry_updated` | Price changed | priceBookId, productId, oldPrice, newPrice |
| DiscountRuleCreated | `crm.prod.discountrule.created` | New discount rule | ruleId, name, type, value |

#### Email Service (`crm.email`)

| Event | Routing Key | Trigger | Key Payload Fields |
|-------|------------|---------|-------------------|
| EmailSent | `crm.email.message.sent` | Email dispatched | messageId, from, to[], subject, contactId |
| EmailReceived | `crm.email.message.received` | Inbound email synced | messageId, from, to[], subject, contactId |
| EmailOpened | `crm.email.message.opened` | Tracking pixel fired | messageId, contactId, openedAt |
| EmailClicked | `crm.email.message.clicked` | Link clicked | messageId, contactId, linkUrl, clickedAt |
| EmailBounced | `crm.email.message.bounced` | Delivery failed | messageId, contactId, bounceType, reason |

#### Calendar Service (`crm.cal`)

| Event | Routing Key | Trigger | Key Payload Fields |
|-------|------------|---------|-------------------|
| CalendarSynced | `crm.cal.connection.synced` | External calendar sync completed | connectionId, provider, eventsAdded, eventsUpdated |
| BookingCreated | `crm.cal.booking.created` | External party booked a slot | bookingId, bookingLinkId, guestEmail, startTime |
| BookingCancelled | `crm.cal.booking.cancelled` | Booking cancelled | bookingId, cancelledBy |

#### Workflow Service (`crm.wf`)

| Event | Routing Key | Trigger | Key Payload Fields |
|-------|------------|---------|-------------------|
| WorkflowTriggered | `crm.wf.workflow.triggered` | Rule matched, execution started | executionId, workflowId, triggerEvent |
| WorkflowCompleted | `crm.wf.workflow.completed` | All actions executed | executionId, workflowId, actionsExecuted |
| WorkflowFailed | `crm.wf.workflow.failed` | Action execution failed | executionId, workflowId, failedAction, error |

#### Notification Service (`crm.ntf`)

| Event | Routing Key | Trigger | Key Payload Fields |
|-------|------------|---------|-------------------|
| NotificationDelivered | `crm.ntf.notification.delivered` | Notification sent to channel | notificationId, userId, channel, eventType |
| NotificationRead | `crm.ntf.notification.read` | User read notification | notificationId, userId, readAt |

#### Document Service (`crm.doc`)

| Event | Routing Key | Trigger | Key Payload Fields |
|-------|------------|---------|-------------------|
| DocumentUploaded | `crm.doc.document.uploaded` | File uploaded | documentId, fileName, mimeType, relatedTo |
| DocumentGenerated | `crm.doc.document.generated` | Template rendered to PDF | documentId, templateId, relatedTo |
| DocumentDeleted | `crm.doc.document.deleted` | File removed | documentId |

#### Reporting Service (`crm.rpt`)

| Event | Routing Key | Trigger | Key Payload Fields |
|-------|------------|---------|-------------------|
| ReportGenerated | `crm.rpt.report.generated` | Report execution completed | reportId, format, rowCount |
| DashboardUpdated | `crm.rpt.dashboard.updated` | Dashboard layout changed | dashboardId, userId |

#### Integration Service (`crm.int`)

| Event | Routing Key | Trigger | Key Payload Fields |
|-------|------------|---------|-------------------|
| WebhookDispatched | `crm.int.webhook.dispatched` | Event forwarded to external URL | webhookId, targetUrl, eventType, statusCode |
| WebhookFailed | `crm.int.webhook.failed` | Webhook delivery failed | webhookId, targetUrl, error, retryCount |
| SyncCompleted | `crm.int.sync.completed` | External system sync finished | connectionId, provider, recordsSynced |

#### Analytics Service (`crm.ana`)

| Event | Routing Key | Trigger | Key Payload Fields |
|-------|------------|---------|-------------------|
| ForecastRecalculated | `crm.ana.forecast.recalculated` | Forecast model updated | periodId, totalForecast, confidence |
| ScoringModelTrained | `crm.ana.scoring.model_trained` | Predictive model retrained | modelId, accuracy, trainingSize |

#### Marketing Service (`crm.mkt`)

| Event | Routing Key | Trigger | Key Payload Fields |
|-------|------------|---------|-------------------|
| CampaignCreated | `crm.mkt.campaign.created` | New campaign | campaignId, name, type, startDate |
| CampaignLaunched | `crm.mkt.campaign.launched` | Campaign activated | campaignId, memberCount |
| CampaignCompleted | `crm.mkt.campaign.completed` | Campaign finished | campaignId, metrics |
| MemberResponded | `crm.mkt.member.responded` | Campaign member took action | campaignId, memberId, responseType |

#### Support Service (`crm.sup`)

| Event | Routing Key | Trigger | Key Payload Fields |
|-------|------------|---------|-------------------|
| TicketCreated | `crm.sup.ticket.created` | New support ticket | ticketId, contactId, subject, priority, category |
| TicketAssigned | `crm.sup.ticket.assigned` | Ticket assigned to agent | ticketId, agentId |
| TicketEscalated | `crm.sup.ticket.escalated` | SLA breach or manual escalation | ticketId, escalationLevel, reason |
| TicketResolved | `crm.sup.ticket.resolved` | Ticket marked resolved | ticketId, resolution, resolutionTime |
| TicketReopened | `crm.sup.ticket.reopened` | Customer re-opened ticket | ticketId, reason |
| SLABreached | `crm.sup.sla.breached` | Response/resolution SLA exceeded | ticketId, slaType, targetTime, actualTime |

#### CPQ Service (`crm.cpq`)

| Event | Routing Key | Trigger | Key Payload Fields |
|-------|------------|---------|-------------------|
| QuoteCreated | `crm.cpq.quote.created` | New quote | quoteId, opportunityId, totalAmount |
| QuoteSubmitted | `crm.cpq.quote.submitted` | Quote sent for approval | quoteId, approvalChainId |
| QuoteApproved | `crm.cpq.quote.approved` | All approvals completed | quoteId, approvedBy |
| QuoteRejected | `crm.cpq.quote.rejected` | Approver rejected | quoteId, rejectedBy, reason |
| QuoteAccepted | `crm.cpq.quote.accepted` | Customer accepted quote | quoteId, acceptedAt, signatureId |
| QuoteExpired | `crm.cpq.quote.expired` | Quote validity period ended | quoteId, expiryDate |

#### Project Service (`crm.prj`)

| Event | Routing Key | Trigger | Key Payload Fields |
|-------|------------|---------|-------------------|
| ProjectCreated | `crm.prj.project.created` | New customer project | projectId, accountId, name |
| ProjectCompleted | `crm.prj.project.completed` | All milestones done | projectId, completedAt |
| MilestoneReached | `crm.prj.milestone.reached` | Project milestone completed | projectId, milestoneId, name |
| ChecklistItemCompleted | `crm.prj.checklist.item_completed` | Onboarding step done | checklistId, itemId, completedBy |

**Event count summary:** 88 distinct event types across 18 services.

---

<!-- ═══════════════════════════════════════════════════════════════════
     SS6  FEATURE CATALOG
     ═══════════════════════════════════════════════════════════════════ -->

## 6. Feature Catalog

### 6.1 Feature Tree

```
CRM_UI (root — optional group)
│
├── F-CRM-001  Contact & Account Management ──────── [mandatory]
│   ├── F-CRM-001-01  Contact List & Search
│   ├── F-CRM-001-02  Contact Detail View
│   ├── F-CRM-001-03  Contact Create/Edit Form
│   ├── F-CRM-001-04  Account List & Search
│   ├── F-CRM-001-05  Account Detail View
│   ├── F-CRM-001-06  Account Create/Edit Form
│   ├── F-CRM-001-07  Relationship Management
│   └── F-CRM-001-08  Duplicate Detection & Merge
│
├── F-CRM-002  Lead Management ──────────────────── [mandatory]
│   ├── F-CRM-002-01  Lead List & Pipeline View
│   ├── F-CRM-002-02  Lead Detail View
│   ├── F-CRM-002-03  Lead Create/Edit Form
│   ├── F-CRM-002-04  Lead Qualification & Scoring
│   ├── F-CRM-002-05  Lead Conversion Wizard
│   └── F-CRM-002-06  Lead Import
│
├── F-CRM-003  Opportunity Management ───────────── [mandatory]
│   ├── F-CRM-003-01  Opportunity List & Kanban
│   ├── F-CRM-003-02  Opportunity Detail View
│   ├── F-CRM-003-03  Opportunity Create/Edit Form
│   ├── F-CRM-003-04  Opportunity Line Items
│   ├── F-CRM-003-05  Stage Progression & Guidance
│   └── F-CRM-003-06  Win/Loss Analysis Dashboard
│
├── F-CRM-004  Activities & Timeline ────────────── [mandatory]
│   ├── F-CRM-004-01  Activity Timeline View
│   ├── F-CRM-004-02  Task Management
│   ├── F-CRM-004-03  Meeting Scheduler
│   ├── F-CRM-004-04  Call Logging
│   └── F-CRM-004-05  Note Editor
│
├── F-CRM-005  Product & Pricing ────────────────── [optional]
│   ├── F-CRM-005-01  Product Catalog Browser
│   ├── F-CRM-005-02  Product Create/Edit
│   ├── F-CRM-005-03  Price Book Management
│   └── F-CRM-005-04  Discount Rule Configuration
│
├── F-CRM-006  Global Search & Views ────────────── [mandatory]
│   ├── F-CRM-006-01  Global Search Bar
│   ├── F-CRM-006-02  Advanced Search & Filters
│   └── F-CRM-006-03  Saved Views & Segments
│
├── F-CRM-007  Email Communication ──────────────── [optional]
│   ├── F-CRM-007-01  Email Compose & Send
│   ├── F-CRM-007-02  Email Inbox & Thread View
│   ├── F-CRM-007-03  Email Template Manager
│   └── F-CRM-007-04  Email Tracking Dashboard
│
├── F-CRM-008  Calendar & Scheduling ────────────── [optional]
│   ├── F-CRM-008-01  Calendar View
│   ├── F-CRM-008-02  Calendar Sync Settings
│   ├── F-CRM-008-03  Booking Link Manager
│   └── F-CRM-008-04  Availability Configurator
│
├── F-CRM-009  Document Management ──────────────── [optional]
│   ├── F-CRM-009-01  Document Library
│   ├── F-CRM-009-02  Document Upload & Attach
│   └── F-CRM-009-03  Document Template Generator
│
├── F-CRM-010  Dashboards & Reporting ───────────── [optional]
│   ├── F-CRM-010-01  Dashboard Builder
│   ├── F-CRM-010-02  Standard Report Library
│   ├── F-CRM-010-03  Custom Report Builder
│   └── F-CRM-010-04  Report Scheduling & Export
│
├── F-CRM-011  Workflow Automation ──────────────── [optional]
│   ├── F-CRM-011-01  Workflow Builder
│   ├── F-CRM-011-02  Workflow Template Library
│   ├── F-CRM-011-03  Workflow Monitor & Logs
│   └── F-CRM-011-04  Assignment & Escalation Rules
│
├── F-CRM-012  Notifications & Alerts ───────────── [optional]
│   ├── F-CRM-012-01  Notification Center
│   ├── F-CRM-012-02  Notification Preferences
│   └── F-CRM-012-03  Channel Configuration
│
├── F-CRM-013  Integrations & Connectors ────────── [optional]
│   ├── F-CRM-013-01  Integration Hub
│   ├── F-CRM-013-02  Webhook Configuration
│   ├── F-CRM-013-03  Connector Marketplace
│   └── F-CRM-013-04  Integration Logs & Monitor
│
├── F-CRM-014  Analytics & Forecasting ──────────── [optional]
│   ├── F-CRM-014-01  Sales Forecast Dashboard
│   ├── F-CRM-014-02  Pipeline Analytics
│   ├── F-CRM-014-03  Activity Analytics
│   └── F-CRM-014-04  Predictive Scoring Setup
│
├── F-CRM-015  Marketing & Campaigns ────────────── [optional]
│   ├── F-CRM-015-01  Campaign List & Manager
│   ├── F-CRM-015-02  Campaign Builder
│   ├── F-CRM-015-03  Campaign Execution Monitor
│   ├── F-CRM-015-04  Lead Nurturing Workflows
│   └── F-CRM-015-05  Campaign Performance Dashboard
│
├── F-CRM-016  Customer Support ─────────────────── [optional]
│   ├── F-CRM-016-01  Ticket List & Queue
│   ├── F-CRM-016-02  Ticket Detail & Resolution
│   ├── F-CRM-016-03  Ticket Create/Edit
│   ├── F-CRM-016-04  SLA Management Dashboard
│   └── F-CRM-016-05  Knowledge Base Browser
│
├── F-CRM-017  Configure-Price-Quote (CPQ) ──────── [optional]
│   ├── F-CRM-017-01  Quote List
│   ├── F-CRM-017-02  Quote Builder
│   ├── F-CRM-017-03  Quote Approval Workflow
│   ├── F-CRM-017-04  Quote PDF Generation
│   └── F-CRM-017-05  Quote Acceptance Portal
│
└── F-CRM-018  Customer Projects ────────────────── [optional]
    ├── F-CRM-018-01  Project Board & List
    ├── F-CRM-018-02  Project Detail & Timeline
    ├── F-CRM-018-03  Onboarding Checklist
    └── F-CRM-018-04  Project Task Management
```

### 6.2 Feature Tree (Mermaid)

```mermaid
graph TD
    CRM_UI["CRM_UI<br/><small>optional group</small>"]

    CRM_UI --> F001["F-CRM-001<br/>Contact & Account<br/><small>mandatory</small>"]
    CRM_UI --> F002["F-CRM-002<br/>Lead Management<br/><small>mandatory</small>"]
    CRM_UI --> F003["F-CRM-003<br/>Opportunity Mgmt<br/><small>mandatory</small>"]
    CRM_UI --> F004["F-CRM-004<br/>Activities & Timeline<br/><small>mandatory</small>"]
    CRM_UI --> F005["F-CRM-005<br/>Product & Pricing<br/><small>optional</small>"]
    CRM_UI --> F006["F-CRM-006<br/>Global Search<br/><small>mandatory</small>"]
    CRM_UI --> F007["F-CRM-007<br/>Email<br/><small>optional</small>"]
    CRM_UI --> F008["F-CRM-008<br/>Calendar<br/><small>optional</small>"]
    CRM_UI --> F009["F-CRM-009<br/>Documents<br/><small>optional</small>"]
    CRM_UI --> F010["F-CRM-010<br/>Reporting<br/><small>optional</small>"]
    CRM_UI --> F011["F-CRM-011<br/>Workflow<br/><small>optional</small>"]
    CRM_UI --> F012["F-CRM-012<br/>Notifications<br/><small>optional</small>"]
    CRM_UI --> F013["F-CRM-013<br/>Integrations<br/><small>optional</small>"]
    CRM_UI --> F014["F-CRM-014<br/>Analytics<br/><small>optional</small>"]
    CRM_UI --> F015["F-CRM-015<br/>Marketing<br/><small>optional</small>"]
    CRM_UI --> F016["F-CRM-016<br/>Support<br/><small>optional</small>"]
    CRM_UI --> F017["F-CRM-017<br/>CPQ<br/><small>optional</small>"]
    CRM_UI --> F018["F-CRM-018<br/>Projects<br/><small>optional</small>"]

    F001 --> F001_01["F-CRM-001-01<br/>Contact List"]
    F001 --> F001_02["F-CRM-001-02<br/>Contact Detail"]
    F001 --> F001_03["F-CRM-001-03<br/>Contact Form"]
    F001 --> F001_04["F-CRM-001-04<br/>Account List"]
    F001 --> F001_05["F-CRM-001-05<br/>Account Detail"]
    F001 --> F001_06["F-CRM-001-06<br/>Account Form"]
    F001 --> F001_07["F-CRM-001-07<br/>Relationships"]
    F001 --> F001_08["F-CRM-001-08<br/>Dedup & Merge"]

    F002 --> F002_01["F-CRM-002-01<br/>Lead Pipeline"]
    F002 --> F002_02["F-CRM-002-02<br/>Lead Detail"]
    F002 --> F002_03["F-CRM-002-03<br/>Lead Form"]
    F002 --> F002_04["F-CRM-002-04<br/>Qualification"]
    F002 --> F002_05["F-CRM-002-05<br/>Conversion"]
    F002 --> F002_06["F-CRM-002-06<br/>Import"]

    F003 --> F003_01["F-CRM-003-01<br/>Opp Kanban"]
    F003 --> F003_02["F-CRM-003-02<br/>Opp Detail"]
    F003 --> F003_03["F-CRM-003-03<br/>Opp Form"]
    F003 --> F003_04["F-CRM-003-04<br/>Line Items"]
    F003 --> F003_05["F-CRM-003-05<br/>Stage Progress"]
    F003 --> F003_06["F-CRM-003-06<br/>Win/Loss"]

    F004 --> F004_01["F-CRM-004-01<br/>Timeline"]
    F004 --> F004_02["F-CRM-004-02<br/>Tasks"]
    F004 --> F004_03["F-CRM-004-03<br/>Meetings"]
    F004 --> F004_04["F-CRM-004-04<br/>Calls"]
    F004 --> F004_05["F-CRM-004-05<br/>Notes"]
```

### 6.3 Cross-Suite Feature Dependencies

| CRM Feature | Requires | Target Suite | Reason |
|-------------|----------|-------------|--------|
| F-CRM-001-01 Contact List | F-IAM-001 (Authentication) | IAM | User must be authenticated to access contacts |
| F-CRM-001-07 Relationships | F-BP-001 (Partner Registry) | BP | BP sync for master data reconciliation |
| F-CRM-003-04 Line Items | F-CRM-005-01 (Product Catalog) | CRM (intra) | Line items require products to be configured |
| F-CRM-007-01 Email Compose | F-CRM-001-02 (Contact Detail) | CRM (intra) | Email send requires resolved contact email |
| F-CRM-010-01 Dashboard Builder | F-CRM-006-01 (Global Search) | CRM (intra) | Dashboard widgets query via search service |
| F-CRM-014-01 Sales Forecast | F-CRM-003-01 (Opp Kanban) | CRM (intra) | Forecast requires opportunity pipeline data |
| F-CRM-015-01 Campaign Manager | F-CRM-001-01 (Contact List) | CRM (intra) | Campaigns target contacts as members |
| F-CRM-015-01 Campaign Manager | F-CRM-002-01 (Lead Pipeline) | CRM (intra) | Campaigns target leads as members |
| F-CRM-016-01 Ticket Queue | F-CRM-001-02 (Contact Detail) | CRM (intra) | Tickets reference reporter as contact |
| F-CRM-017-02 Quote Builder | F-CRM-005-01 (Product Catalog) | CRM (intra) | Quotes pull products and prices |
| F-CRM-017-02 Quote Builder | F-CRM-003-02 (Opp Detail) | CRM (intra) | Quotes linked to opportunities |

### 6.4 Feature Register

| Feature ID | Name | Parent | Status | Spec Reference |
|-----------|------|--------|--------|---------------|
| `F-CRM-001` | Contact & Account Management | CRM_UI | `draft` | `features/compositions/F-CRM-001.md` |
| `F-CRM-001-01` | Contact List & Search | F-CRM-001 | `draft` | `features/leaves/F-CRM-001-01/feature-spec.md` |
| `F-CRM-001-02` | Contact Detail View | F-CRM-001 | `draft` | `features/leaves/F-CRM-001-02/feature-spec.md` |
| `F-CRM-001-03` | Contact Create/Edit Form | F-CRM-001 | `draft` | `features/leaves/F-CRM-001-03/feature-spec.md` |
| `F-CRM-001-04` | Account List & Search | F-CRM-001 | `draft` | `features/leaves/F-CRM-001-04/feature-spec.md` |
| `F-CRM-001-05` | Account Detail View | F-CRM-001 | `draft` | `features/leaves/F-CRM-001-05/feature-spec.md` |
| `F-CRM-001-06` | Account Create/Edit Form | F-CRM-001 | `draft` | `features/leaves/F-CRM-001-06/feature-spec.md` |
| `F-CRM-001-07` | Relationship Management | F-CRM-001 | `draft` | `features/leaves/F-CRM-001-07/feature-spec.md` |
| `F-CRM-001-08` | Duplicate Detection & Merge | F-CRM-001 | `draft` | `features/leaves/F-CRM-001-08/feature-spec.md` |
| `F-CRM-002` | Lead Management | CRM_UI | `draft` | `features/compositions/F-CRM-002.md` |
| `F-CRM-002-01` | Lead List & Pipeline View | F-CRM-002 | `draft` | `features/leaves/F-CRM-002-01/feature-spec.md` |
| `F-CRM-002-02` | Lead Detail View | F-CRM-002 | `draft` | `features/leaves/F-CRM-002-02/feature-spec.md` |
| `F-CRM-002-03` | Lead Create/Edit Form | F-CRM-002 | `draft` | `features/leaves/F-CRM-002-03/feature-spec.md` |
| `F-CRM-002-04` | Lead Qualification & Scoring | F-CRM-002 | `draft` | `features/leaves/F-CRM-002-04/feature-spec.md` |
| `F-CRM-002-05` | Lead Conversion Wizard | F-CRM-002 | `draft` | `features/leaves/F-CRM-002-05/feature-spec.md` |
| `F-CRM-002-06` | Lead Import | F-CRM-002 | `draft` | `features/leaves/F-CRM-002-06/feature-spec.md` |
| `F-CRM-003` | Opportunity Management | CRM_UI | `draft` | `features/compositions/F-CRM-003.md` |
| `F-CRM-003-01` | Opportunity List & Kanban | F-CRM-003 | `draft` | `features/leaves/F-CRM-003-01/feature-spec.md` |
| `F-CRM-003-02` | Opportunity Detail View | F-CRM-003 | `draft` | `features/leaves/F-CRM-003-02/feature-spec.md` |
| `F-CRM-003-03` | Opportunity Create/Edit Form | F-CRM-003 | `draft` | `features/leaves/F-CRM-003-03/feature-spec.md` |
| `F-CRM-003-04` | Opportunity Line Items | F-CRM-003 | `draft` | `features/leaves/F-CRM-003-04/feature-spec.md` |
| `F-CRM-003-05` | Stage Progression & Guidance | F-CRM-003 | `draft` | `features/leaves/F-CRM-003-05/feature-spec.md` |
| `F-CRM-003-06` | Win/Loss Analysis Dashboard | F-CRM-003 | `draft` | `features/leaves/F-CRM-003-06/feature-spec.md` |
| `F-CRM-004` | Activities & Timeline | CRM_UI | `draft` | `features/compositions/F-CRM-004.md` |
| `F-CRM-004-01` | Activity Timeline View | F-CRM-004 | `draft` | `features/leaves/F-CRM-004-01/feature-spec.md` |
| `F-CRM-004-02` | Task Management | F-CRM-004 | `draft` | `features/leaves/F-CRM-004-02/feature-spec.md` |
| `F-CRM-004-03` | Meeting Scheduler | F-CRM-004 | `draft` | `features/leaves/F-CRM-004-03/feature-spec.md` |
| `F-CRM-004-04` | Call Logging | F-CRM-004 | `draft` | `features/leaves/F-CRM-004-04/feature-spec.md` |
| `F-CRM-004-05` | Note Editor | F-CRM-004 | `draft` | `features/leaves/F-CRM-004-05/feature-spec.md` |
| `F-CRM-005` | Product & Pricing | CRM_UI | `draft` | `features/compositions/F-CRM-005.md` |
| `F-CRM-005-01` | Product Catalog Browser | F-CRM-005 | `draft` | `features/leaves/F-CRM-005-01/feature-spec.md` |
| `F-CRM-005-02` | Product Create/Edit | F-CRM-005 | `draft` | `features/leaves/F-CRM-005-02/feature-spec.md` |
| `F-CRM-005-03` | Price Book Management | F-CRM-005 | `draft` | `features/leaves/F-CRM-005-03/feature-spec.md` |
| `F-CRM-005-04` | Discount Rule Configuration | F-CRM-005 | `draft` | `features/leaves/F-CRM-005-04/feature-spec.md` |
| `F-CRM-006` | Global Search & Views | CRM_UI | `draft` | `features/compositions/F-CRM-006.md` |
| `F-CRM-006-01` | Global Search Bar | F-CRM-006 | `draft` | `features/leaves/F-CRM-006-01/feature-spec.md` |
| `F-CRM-006-02` | Advanced Search & Filters | F-CRM-006 | `draft` | `features/leaves/F-CRM-006-02/feature-spec.md` |
| `F-CRM-006-03` | Saved Views & Segments | F-CRM-006 | `draft` | `features/leaves/F-CRM-006-03/feature-spec.md` |
| `F-CRM-007` | Email Communication | CRM_UI | `draft` | `features/compositions/F-CRM-007.md` |
| `F-CRM-007-01` | Email Compose & Send | F-CRM-007 | `draft` | `features/leaves/F-CRM-007-01/feature-spec.md` |
| `F-CRM-007-02` | Email Inbox & Thread View | F-CRM-007 | `draft` | `features/leaves/F-CRM-007-02/feature-spec.md` |
| `F-CRM-007-03` | Email Template Manager | F-CRM-007 | `draft` | `features/leaves/F-CRM-007-03/feature-spec.md` |
| `F-CRM-007-04` | Email Tracking Dashboard | F-CRM-007 | `draft` | `features/leaves/F-CRM-007-04/feature-spec.md` |
| `F-CRM-008` | Calendar & Scheduling | CRM_UI | `draft` | `features/compositions/F-CRM-008.md` |
| `F-CRM-008-01` | Calendar View | F-CRM-008 | `draft` | `features/leaves/F-CRM-008-01/feature-spec.md` |
| `F-CRM-008-02` | Calendar Sync Settings | F-CRM-008 | `draft` | `features/leaves/F-CRM-008-02/feature-spec.md` |
| `F-CRM-008-03` | Booking Link Manager | F-CRM-008 | `draft` | `features/leaves/F-CRM-008-03/feature-spec.md` |
| `F-CRM-008-04` | Availability Configurator | F-CRM-008 | `draft` | `features/leaves/F-CRM-008-04/feature-spec.md` |
| `F-CRM-009` | Document Management | CRM_UI | `draft` | `features/compositions/F-CRM-009.md` |
| `F-CRM-009-01` | Document Library | F-CRM-009 | `draft` | `features/leaves/F-CRM-009-01/feature-spec.md` |
| `F-CRM-009-02` | Document Upload & Attach | F-CRM-009 | `draft` | `features/leaves/F-CRM-009-02/feature-spec.md` |
| `F-CRM-009-03` | Document Template Generator | F-CRM-009 | `draft` | `features/leaves/F-CRM-009-03/feature-spec.md` |
| `F-CRM-010` | Dashboards & Reporting | CRM_UI | `draft` | `features/compositions/F-CRM-010.md` |
| `F-CRM-010-01` | Dashboard Builder | F-CRM-010 | `draft` | `features/leaves/F-CRM-010-01/feature-spec.md` |
| `F-CRM-010-02` | Standard Report Library | F-CRM-010 | `draft` | `features/leaves/F-CRM-010-02/feature-spec.md` |
| `F-CRM-010-03` | Custom Report Builder | F-CRM-010 | `draft` | `features/leaves/F-CRM-010-03/feature-spec.md` |
| `F-CRM-010-04` | Report Scheduling & Export | F-CRM-010 | `draft` | `features/leaves/F-CRM-010-04/feature-spec.md` |
| `F-CRM-011` | Workflow Automation | CRM_UI | `draft` | `features/compositions/F-CRM-011.md` |
| `F-CRM-011-01` | Workflow Builder | F-CRM-011 | `draft` | `features/leaves/F-CRM-011-01/feature-spec.md` |
| `F-CRM-011-02` | Workflow Template Library | F-CRM-011 | `draft` | `features/leaves/F-CRM-011-02/feature-spec.md` |
| `F-CRM-011-03` | Workflow Monitor & Logs | F-CRM-011 | `draft` | `features/leaves/F-CRM-011-03/feature-spec.md` |
| `F-CRM-011-04` | Assignment & Escalation Rules | F-CRM-011 | `draft` | `features/leaves/F-CRM-011-04/feature-spec.md` |
| `F-CRM-012` | Notifications & Alerts | CRM_UI | `draft` | `features/compositions/F-CRM-012.md` |
| `F-CRM-012-01` | Notification Center | F-CRM-012 | `draft` | `features/leaves/F-CRM-012-01/feature-spec.md` |
| `F-CRM-012-02` | Notification Preferences | F-CRM-012 | `draft` | `features/leaves/F-CRM-012-02/feature-spec.md` |
| `F-CRM-012-03` | Channel Configuration | F-CRM-012 | `draft` | `features/leaves/F-CRM-012-03/feature-spec.md` |
| `F-CRM-013` | Integrations & Connectors | CRM_UI | `draft` | `features/compositions/F-CRM-013.md` |
| `F-CRM-013-01` | Integration Hub | F-CRM-013 | `draft` | `features/leaves/F-CRM-013-01/feature-spec.md` |
| `F-CRM-013-02` | Webhook Configuration | F-CRM-013 | `draft` | `features/leaves/F-CRM-013-02/feature-spec.md` |
| `F-CRM-013-03` | Connector Marketplace | F-CRM-013 | `draft` | `features/leaves/F-CRM-013-03/feature-spec.md` |
| `F-CRM-013-04` | Integration Logs & Monitor | F-CRM-013 | `draft` | `features/leaves/F-CRM-013-04/feature-spec.md` |
| `F-CRM-014` | Analytics & Forecasting | CRM_UI | `draft` | `features/compositions/F-CRM-014.md` |
| `F-CRM-014-01` | Sales Forecast Dashboard | F-CRM-014 | `draft` | `features/leaves/F-CRM-014-01/feature-spec.md` |
| `F-CRM-014-02` | Pipeline Analytics | F-CRM-014 | `draft` | `features/leaves/F-CRM-014-02/feature-spec.md` |
| `F-CRM-014-03` | Activity Analytics | F-CRM-014 | `draft` | `features/leaves/F-CRM-014-03/feature-spec.md` |
| `F-CRM-014-04` | Predictive Scoring Setup | F-CRM-014 | `draft` | `features/leaves/F-CRM-014-04/feature-spec.md` |
| `F-CRM-015` | Marketing & Campaigns | CRM_UI | `draft` | `features/compositions/F-CRM-015.md` |
| `F-CRM-015-01` | Campaign List & Manager | F-CRM-015 | `draft` | `features/leaves/F-CRM-015-01/feature-spec.md` |
| `F-CRM-015-02` | Campaign Builder | F-CRM-015 | `draft` | `features/leaves/F-CRM-015-02/feature-spec.md` |
| `F-CRM-015-03` | Campaign Execution Monitor | F-CRM-015 | `draft` | `features/leaves/F-CRM-015-03/feature-spec.md` |
| `F-CRM-015-04` | Lead Nurturing Workflows | F-CRM-015 | `draft` | `features/leaves/F-CRM-015-04/feature-spec.md` |
| `F-CRM-015-05` | Campaign Performance Dashboard | F-CRM-015 | `draft` | `features/leaves/F-CRM-015-05/feature-spec.md` |
| `F-CRM-016` | Customer Support | CRM_UI | `draft` | `features/compositions/F-CRM-016.md` |
| `F-CRM-016-01` | Ticket List & Queue | F-CRM-016 | `draft` | `features/leaves/F-CRM-016-01/feature-spec.md` |
| `F-CRM-016-02` | Ticket Detail & Resolution | F-CRM-016 | `draft` | `features/leaves/F-CRM-016-02/feature-spec.md` |
| `F-CRM-016-03` | Ticket Create/Edit | F-CRM-016 | `draft` | `features/leaves/F-CRM-016-03/feature-spec.md` |
| `F-CRM-016-04` | SLA Management Dashboard | F-CRM-016 | `draft` | `features/leaves/F-CRM-016-04/feature-spec.md` |
| `F-CRM-016-05` | Knowledge Base Browser | F-CRM-016 | `draft` | `features/leaves/F-CRM-016-05/feature-spec.md` |
| `F-CRM-017` | Configure-Price-Quote (CPQ) | CRM_UI | `draft` | `features/compositions/F-CRM-017.md` |
| `F-CRM-017-01` | Quote List | F-CRM-017 | `draft` | `features/leaves/F-CRM-017-01/feature-spec.md` |
| `F-CRM-017-02` | Quote Builder | F-CRM-017 | `draft` | `features/leaves/F-CRM-017-02/feature-spec.md` |
| `F-CRM-017-03` | Quote Approval Workflow | F-CRM-017 | `draft` | `features/leaves/F-CRM-017-03/feature-spec.md` |
| `F-CRM-017-04` | Quote PDF Generation | F-CRM-017 | `draft` | `features/leaves/F-CRM-017-04/feature-spec.md` |
| `F-CRM-017-05` | Quote Acceptance Portal | F-CRM-017 | `draft` | `features/leaves/F-CRM-017-05/feature-spec.md` |
| `F-CRM-018` | Customer Projects | CRM_UI | `draft` | `features/compositions/F-CRM-018.md` |
| `F-CRM-018-01` | Project Board & List | F-CRM-018 | `draft` | `features/leaves/F-CRM-018-01/feature-spec.md` |
| `F-CRM-018-02` | Project Detail & Timeline | F-CRM-018 | `draft` | `features/leaves/F-CRM-018-02/feature-spec.md` |
| `F-CRM-018-03` | Onboarding Checklist | F-CRM-018 | `draft` | `features/leaves/F-CRM-018-03/feature-spec.md` |
| `F-CRM-018-04` | Project Task Management | F-CRM-018 | `draft` | `features/leaves/F-CRM-018-04/feature-spec.md` |

### 6.5 Variability Summary

| Metric | Value |
|--------|-------|
| Total composition nodes | 18 |
| Total leaf features | 81 |
| Mandatory compositions | 4 (F-CRM-001, F-CRM-002, F-CRM-003, F-CRM-004) |
| Mandatory leaf features | 28 (all leaves under mandatory compositions + F-CRM-006) |
| Optional compositions | 14 |
| Optional leaf features | 53 |
| Cross-suite `requires` | 2 (IAM, BP) |
| Intra-suite `requires` | 9 |
| Attributes (total across leaves) | ~162 (est. 2 per leaf) |
| Binding times used | `compile`, `deploy`, `runtime` |

**Mandatory group rationale:** Contact & Account Management (F-CRM-001), Lead Management (F-CRM-002), Opportunity Management (F-CRM-003), and Activities & Timeline (F-CRM-004) form the irreducible core of any CRM deployment. Without contacts, leads, opportunities, and activity tracking, no other CRM capability delivers value. Global Search (F-CRM-006) is also mandatory as every CRM screen depends on it.

> **Note:** F-CRM-006 (Global Search & Views) is marked mandatory because it provides the search infrastructure all other features depend on. Products that include CRM_UI MUST include F-CRM-006.

---

<!-- ═══════════════════════════════════════════════════════════════════
     SS7  CROSS-CUTTING CONCERNS
     ═══════════════════════════════════════════════════════════════════ -->

## 7. Cross-Cutting Concerns

### 7.1 Compliance

| Regulation | Requirement | Implementation |
|-----------|-------------|----------------|
| GDPR (EU 2016/679) | Right to access, rectification, erasure, portability for personal data | Contact/Lead data export API; soft-delete with 30-day purge; anonymization for analytics; consent tracking on marketing communications |
| DSGVO (German GDPR) | Stricter consent requirements for email marketing; data processing agreements | Double opt-in for email campaigns; DPA templates in document service; Auftragsverarbeitung records |
| ePrivacy Directive | Cookie consent for email tracking pixels | Tracking pixel opt-out per contact; unsubscribe link mandatory in all marketing emails |
| ISO 27001 | Information security management | Encryption at rest (AES-256) and in transit (TLS 1.3); access control via Keycloak; audit logging |
| SOC 2 Type II | Trust service criteria: security, availability, confidentiality | Penetration testing; uptime monitoring; data classification per field; incident response procedures |

### 7.2 Security

| Aspect | Approach |
|--------|---------|
| **Authentication** | OAuth2 / OIDC via T1 `iam-svc` (Keycloak). All CRM API calls require valid Bearer token. |
| **Authorization** | RBAC via Keycloak roles + OPAL fine-grained policies. CRM defines roles: `crm-admin`, `crm-sales-rep`, `crm-sales-manager`, `crm-marketing`, `crm-support-agent`, `crm-readonly`. Field-level visibility rules via OPAL. |
| **Data Classification** | **Confidential**: Contact PII (email, phone, address), email message bodies, document contents. **Internal**: Opportunity amounts, forecast data, pipeline metrics. **Public**: Product catalog (when shared via quote acceptance portal). |

### 7.3 Multi-Tenancy

| Aspect | Value |
|--------|-------|
| **Model** | `shared_schema` |
| **Isolation** | Row-Level Security via `tenant_id` on all tables. All PostgreSQL queries filtered by RLS policy. |
| **Tenant ID Propagation** | JWT claim `tenant_id` → Spring Security `TenantContext` thread-local → propagated in event envelope `tenantId` field and `X-Tenant-ID` HTTP header for inter-service calls. |

**Rules:**
- All queries MUST include `tenant_id` filter (enforced by RLS — no application-level bypass).
- Cross-tenant data access is forbidden at the API level.
- Elasticsearch indices use tenant-field filtering (not index-per-tenant) for cost efficiency.
- MinIO uses tenant-prefixed bucket paths: `crm-docs/{tenantId}/`.
- RabbitMQ events always carry `tenantId`; consumers MUST validate tenant context.

### 7.4 Audit

**Audit Requirements:**
- All state changes on aggregates MUST be audit-logged (who, when, what, old value, new value).
- All API calls MUST be logged with request ID, user ID, tenant ID, HTTP method, path, and response status.
- Login/logout events and permission changes are logged by IAM suite (CRM consumes for timeline enrichment).
- Email open/click tracking events are audit-logged for compliance with ePrivacy regulations.

**Retention Policies:**

| Entity / Data Class | Retention Period | Legal Basis | Action After Expiry |
|--------------------|-----------------|-------------|-------------------|
| Contact / Account data | Indefinite (until deletion request) | Legitimate interest / Contract | `anonymize` on GDPR erasure request |
| Lead data | 2 years after disqualification | Legitimate interest | `anonymize` |
| Opportunity data | 7 years after close | HGB §257 (commercial records) | `archive` |
| Email messages | 3 years | Business correspondence | `delete` |
| Email tracking events | 1 year | ePrivacy compliance | `delete` |
| Audit log entries | 90 days (hot) + 2 years (cold) | Internal security policy | `delete` |
| Support tickets | 5 years after resolution | Customer service records | `archive` |
| Documents | Until explicit deletion by user | User-controlled | `delete` |
| Analytics aggregates | 3 years | Business intelligence | `anonymize` |
| Quote PDFs | 10 years | HGB §257 (commercial documents) | `archive` |

---

<!-- ═══════════════════════════════════════════════════════════════════
     SS8  EXTERNAL INTERFACES
     ═══════════════════════════════════════════════════════════════════ -->

## 8. External Interfaces

### 8.1 Outbound Interfaces (CRM → Other Suites)

| Target Suite | Interface Type | Interface Name | Description |
|-------------|---------------|----------------|-------------|
| COM | `event` | `crm.opp.opportunity.closed_won` | Signals deal closure; COM may create Sales Order |
| COM | `event` | `crm.cpq.quote.accepted` | Signals quote acceptance; COM may create Sales Order |
| OPS | `event` | `crm.sup.ticket.escalated` | Signals ticket escalation; OPS may create Service Order |
| DTP | `event` | `crm.int.webhook.dispatched` | External system webhook notifications |
| DTP | `event` | `crm.int.sync.completed` | Signals external data sync completion |
| Platform | `api` | `GET /api/crm/search/v1/query` | Platform services may search CRM data (read-only) |

### 8.2 Inbound Interfaces (Other Suites → CRM)

| Source Suite | Interface Type | Interface Name | Description |
|-------------|---------------|----------------|-------------|
| BP | `event` | `bp.partner.partner.created` | New Business Partner → CRM creates/updates Contact or Account |
| BP | `event` | `bp.partner.partner.updated` | BP master data change → CRM updates matching Contact/Account |
| IAM | `event` | `iam.principal.principal.created` | New user → CRM may create internal Contact record |
| IAM | `event` | `iam.principal.principal.deactivated` | User deactivated → CRM reassigns open activities and leads |
| COM | `event` | `com.catalog.article.updated` | Product data change → CRM updates matching Product |
| COM | `event` | `com.order.order.created` | Order placed → CRM updates Opportunity to Closed Won (if linked) |
| Platform | `api` | CRM consumes `ref-data-svc` | Countries, currencies, units of measure for validation |

### 8.3 External Context Mapping

| External Suite | DDD Pattern | Rationale |
|---------------|------------|-----------|
| BP (Business Partner) | **Conformist** | CRM conforms to BP's partner model. BP is the authoritative source for master data. CRM stores a local copy with CRM-specific enrichments (lead source, lifecycle stage). |
| COM (Commerce) | **Anti-Corruption Layer** | CRM translates COM's Article model into its own lightweight Product aggregate. Different bounded contexts, different concerns (CRM cares about pricing for quotes; COM cares about inventory and fulfillment). |
| IAM | **Conformist** | CRM conforms to IAM's principal and role models. No translation needed. |
| OPS (Operations) | **Published Language** | CRM publishes well-defined escalation events. OPS decides how to handle them — no coupling back. |
| DTP (Data Transfer Platform) | **Published Language** | CRM publishes integration events. DTP orchestrates external system sync. |

---

<!-- ═══════════════════════════════════════════════════════════════════
     SS9  ARCHITECTURE DECISIONS
     ═══════════════════════════════════════════════════════════════════ -->

## 9. Architecture Decisions

### ADR-CRM-001: Core vs Extension Service Split

**Status:** Accepted  
**Date:** 2026-02-23

**Context:** The CRM suite has 18 services. Not all deployments need all 18. Customers in early CRM adoption need contacts, leads, and opportunities but may not need marketing automation or CPQ.

**Decision:** Split services into Core (6, always deployed) and Extension (12, opt-in per tenant). Core services use synchronous REST for tight integration. Extension services are event-driven and can be deployed/undeployed independently.

**Consequences:**
- **Positive:** Smaller initial footprint; extension services scale independently; feature-gating aligns with licensing tiers.
- **Negative:** Core services cannot depend on extension services (no sync calls to email, workflow, etc.); event replay needed when enabling a previously disabled extension.

### ADR-CRM-002: CQRS for Search via Elasticsearch

**Status:** Accepted  
**Date:** 2026-02-23

**Context:** CRM users expect instant, cross-entity search (contacts + leads + opportunities + activities in one query). Querying 5+ PostgreSQL databases with JOINs is unacceptable for latency and scalability.

**Decision:** Dedicated Search Service (`crm.search`) maintains Elasticsearch indices as denormalized CQRS read models. All CRM services publish domain events; Search Service projects them into indices.

**Consequences:**
- **Positive:** Sub-100ms search across all entities; faceted filtering; saved views; no cross-database queries.
- **Negative:** Eventual consistency (search results may lag 1-3 seconds after writes); index rebuild needed for schema changes; Elasticsearch operational overhead.

### ADR-CRM-003: Temporal for Lead Conversion Saga

**Status:** Accepted  
**Date:** 2026-03-15

**Context:** Lead conversion creates a Contact, an Account (if new), and an Opportunity in three separate services. If step 2 or 3 fails, the user expects consistent rollback or clear partial-success messaging.

**Decision:** Use Temporal workflow orchestration for the lead conversion saga. The saga has compensation logic for each step.

**Consequences:**
- **Positive:** Reliable multi-service orchestration with retry, timeout, and compensation; full execution history for debugging.
- **Negative:** Temporal infrastructure dependency; saga-aware testing needed; slightly increased latency (~200ms overhead).

### ADR-CRM-004: Tenant-Field Elasticsearch (not Index-per-Tenant)

**Status:** Accepted  
**Date:** 2026-02-23

**Context:** With potentially hundreds of tenants, creating separate ES indices per tenant leads to shard explosion and high operational cost.

**Decision:** Use shared indices with a `tenant_id` field. All queries include a mandatory `tenant_id` filter term.

**Consequences:**
- **Positive:** Predictable shard count; simpler index management; better resource utilization.
- **Negative:** Noisy-neighbor risk (mitigated by per-tenant query rate limiting); no per-tenant index lifecycle.

### ADR-CRM-005: MinIO for Document and Attachment Storage

**Status:** Accepted  
**Date:** 2026-03-01

**Context:** CRM needs to store email attachments, quote PDFs, and user-uploaded documents. PostgreSQL BLOB storage is inefficient for large files.

**Decision:** Use MinIO (S3-compatible) for binary object storage. Metadata stored in PostgreSQL; files in MinIO with tenant-prefixed paths.

**Consequences:**
- **Positive:** Scalable object storage; S3 API compatibility for future cloud migration; streaming upload/download.
- **Negative:** Two storage systems to operate; referential integrity between PostgreSQL metadata and MinIO objects must be maintained via application logic.

---

<!-- ═══════════════════════════════════════════════════════════════════
     SS10  ROADMAP
     ═══════════════════════════════════════════════════════════════════ -->

## 10. Roadmap

| Phase | Timeframe | Items |
|-------|-----------|-------|
| Phase 1 — Core CRM | Weeks 1–12 | `crm.contact` (Contact + Account), `crm.lead` (Lead Pipeline), `crm.opp` (Opportunity Pipeline), `crm.act` (Tasks, Notes, Activity Timeline), `crm.search` (Elasticsearch integration), Vue.js Frontend: Contact/Lead/Opportunity views, Kanban boards. Features: F-CRM-001, F-CRM-002, F-CRM-003, F-CRM-004, F-CRM-006. |
| Phase 2 — Essential Extensions | Weeks 13–20 | `crm.email` (Email integration, templates, tracking), `crm.wf` (Workflow engine, basic automation), `crm.ntf` (Notifications), `crm.prod` (Product catalog). Features: F-CRM-005, F-CRM-007, F-CRM-011, F-CRM-012. |
| Phase 3 — Sales Acceleration | Weeks 21–28 | `crm.cpq` (Quotes), `crm.doc` (Document generation, e-signature), `crm.rpt` (Dashboards, KPIs), Lead conversion Temporal Saga. Features: F-CRM-009, F-CRM-010, F-CRM-017. |
| Phase 4 — Advanced | Weeks 29+ | `crm.mkt` (Marketing automation, campaigns), `crm.sup` (Support tickets, SLA), `crm.ana` (Predictive scoring, forecasting), `crm.int` (Webhooks, 3rd-party connectors), `crm.prj` (Customer onboarding projects), `crm.cal` (Calendar sync: Google, Outlook). Features: F-CRM-008, F-CRM-013, F-CRM-014, F-CRM-015, F-CRM-016, F-CRM-018. |

---

<!-- ═══════════════════════════════════════════════════════════════════
     SS11  APPENDIX
     ═══════════════════════════════════════════════════════════════════ -->

## 11. Appendix

### 11.1 Change Log

| Date | Version | Author | Changes |
|------|---------|--------|---------|
| 2026-02-23 | 0.1.0 | OpenLeap Architecture Team | Initial CRM suite architecture draft (~35% compliance) |
| 2026-04-03 | 1.0.0 | OpenLeap Architecture Team | Full rewrite to suite-spec.md v1.0.0 template. 100% compliance. Added SS0-SS11, feature tree (18 compositions, 78 leaves), 88 events, full UBL glossary, ADRs, cross-suite context mapping. |

### 11.2 Review & Approval

**Status:** DRAFT

**Reviewers:**

| Role | Name | Date | Status |
|------|------|------|--------|
| Suite Architect | TBD | YYYY-MM-DD | [ ] Reviewed |
| Domain Lead (contact) | TBD | YYYY-MM-DD | [ ] Reviewed |
| Domain Lead (lead) | TBD | YYYY-MM-DD | [ ] Reviewed |
| Domain Lead (opp) | TBD | YYYY-MM-DD | [ ] Reviewed |
| Domain Lead (act) | TBD | YYYY-MM-DD | [ ] Reviewed |
| Product Owner | TBD | YYYY-MM-DD | [ ] Reviewed |

**Approval:**

| Role | Name | Date | Approved |
|------|------|------|----------|
| Suite Architect | TBD | YYYY-MM-DD | [ ] |
| Engineering Manager | TBD | YYYY-MM-DD | [ ] |

---

## Authoring Checklist

> Before moving to REVIEW status, verify:

- [x] Suite ID follows pattern `^[a-z]{2,4}$` (SS0) — `crm`
- [x] Business purpose is at least 50 characters (SS0)
- [x] In-scope and out-of-scope are concrete and mutually exclusive (SS0)
- [x] UBL glossary has entries for every core concept (SS1) — 15 entries
- [x] Every glossary definition is at least 20 characters (SS1)
- [x] UBL boundary test demonstrates vocabulary distinction from at least one related suite (SS1) — 7 boundary tests
- [x] Every core concept in SS2 has a glossary_ref back to SS1 (SS2)
- [x] Shared kernel types define authoritative attributes (SS2)
- [x] Bounded context map uses valid DDD patterns (SS2)
- [x] Service catalog lists all services with status and spec reference (SS3) — 18 services
- [x] Integration pattern decision has rationale (SS4)
- [x] Event flows cover all major intra-suite workflows (SS4) — 3 key workflows
- [x] Routing key pattern is documented with segments and examples (SS5)
- [x] Payload envelope matches platform standard (SS5)
- [x] Event catalog lists every published event (SS5) — 88 events
- [x] Feature tree is complete with mandatory/optional annotations (SS6) — 18 compositions, 81 leaves
- [x] Cross-suite feature dependencies are listed (SS6) — 11 dependencies
- [ ] Companion `crm.catalog.uvl` is created and matches SS6 tree (SS6)
- [x] Compliance requirements list all applicable regulations (SS7) — 5 regulations
- [x] Multi-tenancy model is specified (SS7)
- [x] Retention policies have legal basis (SS7) — 10 retention rules
- [x] External interfaces document all cross-suite communication (SS8) — 6 outbound, 6 inbound
- [x] External context mapping uses valid DDD patterns (SS8) — 5 mappings
- [x] All ADRs have ID pattern `ADR-CRM-NNN` (SS9) — 5 ADRs
- [x] Roadmap covers at least the next two phases (SS10) — 4 phases
