<!-- Template Meta
     Template-ID:   TPL-SVC
     Version:       1.1.0
     Compliance:    95%+
-->

# Email Service — Domain Specification

> **Conceptual Stack Layer:** Domain / Service
> **Space:** Platform
> **Owner:** Platform Infrastructure Team

> **Meta Information**
> - **Version:** 2026-04-18
> - **Template:** `domain-service-spec.md` v1.1.0
> - **Template Compliance:** 95%+
> - **Status:** DRAFT
> - **Service ID:** `tech-email-svc`
> - **Suite:** `tech`
> - **Domain:** `email`
> - **Base Package:** `io.openleap.tech.email`
> - **API Path:** `/api/tech/email/v1`
> - **Port:** 8411
> - **DB Schema:** `tech_email`
> - **Tier:** T1 — Platform & Technical Foundations
> - **Supersedes:** `crm-email-svc` (see §13 Migration)

---

## 0. Document Purpose & Scope

This document specifies the **Email Service** (`tech-email-svc`) as a T1 Platform infrastructure capability: email transport (SMTP send, IMAP receive), template management, and open/click tracking. Consumed by any suite that sends or receives email (notifications, ticket channels, marketing).

**Audience:** Backend developers, integration engineers, platform ops.

**In scope:** Outbound send (SMTP / SendGrid / SES / Mailgun), inbound sync (IMAP / MS Graph / webhook), template management with merge fields, tracking, bounce handling, rate limits.
**Out of scope:** Business-level campaign orchestration (→ crm.mkt for marketing), per-suite composition UIs (handled by consuming suite), archival (→ tech.dms).

**Related documents:**
- Suite Spec: `T1_Platform/tech/_tech_suite.md`
- OpenAPI: `contracts/http/tech/email/openapi.yaml`
- Event Schemas: `contracts/events/tech/email/*.schema.json`
- Origin (deprecated): `T3_Domains/CRM/domain-specs/crm_email-spec.md`

---

## 1. Business Context

### 1.1 Purpose & Responsibility

Email transport layer for the entire platform. Composes and sends outbound mail, ingests inbound mail, resolves the sender/recipient to a `shared.bp` Party when possible, fires events so downstream consumers (ticket channel adapter, marketing, notification) can react. Tracks delivery, opens, clicks, bounces.

**Owns:**
- EmailMessage (sent or received message with headers, body, attachments, tracking metadata)
- EmailTemplate (reusable template with merge fields, HTML + plain-text variants, locale)
- EmailAccount (per-tenant or per-principal SMTP / IMAP / OAuth mailbox)
- EmailTrackingEvent (open / click / bounce / complaint)
- EmailSuppression (hard-bounce list, spam-complaint list — per tenant)

### 1.2 Business Value

Single transport fabric for every email-driven flow: ticket channel, notifications, marketing, transactional. Deliverability managed centrally (SPF/DKIM/DMARC, suppression lists, rate limits).

---

## 2. Service Identity

| Field | Value |
|-------|-------|
| Service ID | `tech-email-svc` |
| Suite | `tech` |
| Domain | `email` |
| Bounded Context | Email Transport |
| Base Package | `io.openleap.tech.email` |
| API Base Path | `/api/tech/email/v1` |
| Port | 8411 |
| Database Schema | `tech_email` |
| Tier | T1 — Platform & Technical Foundations |
| Repository | `openleap-io/io.openleap.tech.email` |
| Team | `team-platform-infra` |

---

## 3. Domain Model

### 3.1 Aggregates

**EmailMessage** (AggregateRoot)
- id, tenantId, direction (`INBOUND`|`OUTBOUND`), headers (JSONB), from, to[], cc[], bcc[], subject, bodyHtml, bodyText, attachmentDmsIds[] (→ tech.dms), status (`QUEUED`|`SENT`|`DELIVERED`|`BOUNCED`|`RECEIVED`|`FAILED`), providerMessageId, tracking pixel token, sourceAccountId, threadId, relatedPartyIds[] (→ shared.bp).

**EmailTemplate** (AggregateRoot)
- id, tenantId, slug, locale, subject template, bodyHtml template, bodyText template, mergeFields[], category (e.g., `transactional`|`notification`|`marketing`), version, status.

**EmailAccount** (AggregateRoot)
- id, tenantId, ownerScope (`TENANT`|`PRINCIPAL`), name, transport (SMTP | SES | SendGrid | Mailgun | MS-Graph | IMAP), credentials ref (→ iam tenant secrets), inboundPolling (boolean), inboundWebhookUrl, status.

**EmailTrackingEvent** (AggregateRoot)
- id, messageId, type (`OPEN`|`CLICK`|`BOUNCE`|`COMPLAINT`|`UNSUBSCRIBE`), occurredAt, metadata.

**EmailSuppression** (AggregateRoot)
- tenantId, address, reason (`HARD_BOUNCE`|`COMPLAINT`|`MANUAL`), createdAt, expiresAt.

---

## 4. Business Rules

### 4.1 Rule Catalog

| ID | Rule | Type |
|----|------|------|
| BR-EMAIL-001 | Default rate limit 500 mails/hour per tenant; per-account overrides honored; platform-wide cap 10 000/hour | POLICY |
| BR-EMAIL-002 | Max 50 recipients per message (To + Cc + Bcc); bulk sends require a campaign ID and MUST use batch API | CONSTRAINT |
| BR-EMAIL-003 | Tracking pixel injected automatically for HTML outbound except when `trackingOptOut=true` | POLICY |
| BR-EMAIL-004 | Unsubscribe link MUST be present in marketing category; enforcement by template validator | CONSTRAINT |
| BR-EMAIL-005 | IMAP sync default interval 5 minutes per account, configurable 1 min – 1 h | POLICY |
| BR-EMAIL-006 | Bounce handling: soft bounce retries 3× exponential (1 min / 5 min / 30 min); hard bounce → `EmailSuppression` entry | POLICY |
| BR-EMAIL-007 | Suppression list enforced pre-send; suppressed recipients dropped with `EmailSuppressed` event | CONSTRAINT |
| BR-EMAIL-008 | Inbound message that resolves to an active `shared.bp` party and a subscribing consumer produces exactly one `EmailReceived` event (idempotent by providerMessageId) | CONSTRAINT |
| BR-EMAIL-009 | Tenant isolation on all aggregates via RLS on `tenant_id` | CONSTRAINT |

---

## 5. Use Cases

| Endpoint | Use Case Type | Primary Aggregate |
|----------|--------------|------------------|
| `POST /emails/send` | WRITE | EmailMessage |
| `POST /emails/send-template` | WRITE | EmailMessage |
| `POST /emails/send-batch` | WRITE (bulk) | EmailMessage |
| `GET /emails` | READ | EmailMessage |
| `GET /emails/{id}` | READ | EmailMessage |
| `GET /emails/{id}/tracking` | READ | EmailTrackingEvent |
| `POST /email-templates` | WRITE | EmailTemplate |
| `GET /email-templates` | READ | EmailTemplate |
| `PUT /email-templates/{id}` | WRITE | EmailTemplate |
| `POST /email-accounts` | WRITE | EmailAccount |
| `GET /email-accounts` | READ | EmailAccount |
| `POST /email-accounts/{id}/sync` | WRITE | EmailAccount |
| `GET /suppressions` | READ | EmailSuppression |
| `POST /suppressions` | WRITE | EmailSuppression |
| `DELETE /suppressions/{address}` | WRITE | EmailSuppression |

---

## 6. REST API

**Base Path:** `/api/tech/email/v1`
**Authentication:** OAuth2 / JWT. Scopes `tech.email:read`, `tech.email:write`, `tech.email:admin`.

See `contracts/http/tech/email/openapi.yaml` for full payloads.

---

## 7. Events & Integration

### 7.1 Published Events

| Event | Routing Key |
|-------|------------|
| EmailQueued | `tech.email.email.queued` |
| EmailSent | `tech.email.email.sent` |
| EmailDelivered | `tech.email.email.delivered` |
| EmailReceived | `tech.email.email.received` |
| EmailOpened | `tech.email.email.opened` |
| EmailClicked | `tech.email.email.clicked` |
| EmailBounced | `tech.email.email.bounced` |
| EmailComplaint | `tech.email.email.complaint` |
| EmailSuppressed | `tech.email.email.suppressed` |
| EmailAccountConnected | `tech.email.account.connected` |
| EmailAccountSyncFailed | `tech.email.account.sync-failed` |

### 7.2 Consumed Events

- `iam.principal.principal.deleted` → enqueue GDPR cleanup of messages owned by principal.
- `shared.bp.party.merged` → reattribute historical messages to surviving party.

### 7.3 Routing-Key Bridge (Migration)

For 60 days after promotion, outbound events also emitted under legacy `crm.email.*` keys.

---

## 8. Data Model

**Storage:** PostgreSQL 16, schema `tech_email`. RLS on `tenant_id`. Attachments stored via `tech.dms`.

Key tables: `email_messages`, `email_message_headers` (normalized), `email_templates`, `email_accounts`, `email_tracking_events`, `email_suppressions`.

---

## 9. Security & Compliance

### 9.1 Access Control

| Role | Access Level |
|------|-------------|
| `platform-admin` | Manage global rate limits, platform-default templates |
| `tenant-admin` | Manage tenant accounts, templates, suppressions |
| `automation-sender` (service account) | Send on behalf of tenant |
| `tenant-user` | Send from own-owned account; read own messages |

### 9.2 Data Classification

Confidential — email bodies MAY contain PII. Bodies stored encrypted at rest. Logs MUST NOT include bodies; only message IDs and metadata.

### 9.3 Compliance

- **GDPR Art. 17** (erasure): `DELETE /emails?partyId=X` + automatic on `shared.bp.party.erased` event.
- **GDPR Art. 20** (portability): `GET /emails?partyId=X` returns JSON export.
- **CAN-SPAM / GDPR marketing**: unsubscribe link enforcement via template validator (BR-EMAIL-004).
- **DORA ICT third-party**: transport provider credentials classified as critical third-party dependency; SPOC to platform-ops for outages.

---

## 10. Quality Attributes

### 10.1 Performance Targets

| Operation | p95 | p99 |
|-----------|-----|-----|
| `POST /emails/send` (accepted, queued) | < 80 ms | < 200 ms |
| Queue → provider submit | < 3 s | < 10 s |
| Provider webhook → tracking event | < 2 s | < 5 s |

### 10.2 Availability

99.9 %. Graceful degradation: queue persists if provider outage; retries on recovery.

### 10.3 Scalability

- API tier stateless, horizontal.
- Dedicated worker pools per transport provider (circuit breakers, provider-specific back-pressure).
- Inbound pollers shard by tenant hash.

---

## 11. Feature Dependencies

| Feature ID | Feature Name |
|-----------|-------------|
| `F-TECH-EMAIL-001` | Outbound Compose |
| `F-TECH-EMAIL-002` | Inbound Sync & Thread |
| `F-TECH-EMAIL-003` | Template Editor |
| `F-TECH-EMAIL-004` | Tracking & Deliverability Dashboard |
| `F-TECH-EMAIL-005` | Suppression Management |

Consumed by features across suites (e.g., `F-TKS-210` Email Channel Adapter, marketing features in `crm.mkt`).

---

## 12. Extension Points

- New transport providers via `POST /email-accounts/extensions/transports`.
- Custom merge-field resolvers via `POST /email-templates/extensions/merge-resolvers`.
- Extension events `tech.email.ext.pre-send`, `tech.email.ext.post-send` for tenant enrichment / scanning.

---

## 13. Migration & Evolution

### 13.1 Promotion from `crm-email-svc`

- Origin `T3_Domains/CRM/domain-specs/crm_email-spec.md` marked DEPRECATED.
- Routing keys: new `tech.email.*`; old `crm.email.*` re-emitted 60 days.
- API path `/api/tech/email/v1`; `/api/crm/email/v1` 308-redirected.
- Schema `crm_email` renamed to `tech_email`. Data migrated verbatim; tenant boundaries unchanged.
- Port stays 8411.
- Marketing-specific flows that stayed in `crm.mkt` now call `tech-email-svc` directly.

### 13.2 Post-Migration

Bridge removed after 60 days. Origin retired two minor versions later.

---

## 14. Decisions & Open Questions

| # | Question | Severity | Status |
|---|----------|----------|--------|
| Q-EMAIL-001 | Do we keep built-in IMAP pollers or require webhook-only providers in v2? | Medium | Open |
| Q-EMAIL-002 | Separate `tech.mail.archive` service for legal holds, or extend `tech.dms`? | Medium | Open |
| Q-EMAIL-003 | Provider connector set at GA: SES + SendGrid + SMTP + MS-Graph only? | Low | Open |

### ADRs

- **ADR-EMAIL-001** *(Proposed)*: Promote to T1. Rationale: transport is platform infrastructure; marketing semantics stay in `crm.mkt`.
- **ADR-EMAIL-002** *(Proposed)*: Attachments stored in `tech.dms`; `EmailMessage` references by DMS document id.

---

## 15. Appendix

### 15.1 Glossary

See `T1_Platform/tech/_tech_suite.md` SS1.

### 15.2 Change Log

| Date | Version | Author | Changes |
|------|---------|--------|---------|
| 2026-04-18 | 1.0.0 | OpenLeap Architecture Team | Initial promotion spec from crm.email (supersedes crm-email-svc) |

### 15.3 Companion Files

- OpenAPI: `contracts/http/tech/email/openapi.yaml`
- Event Schemas: `contracts/events/tech/email/*.schema.json`
- Origin (deprecated): `T3_Domains/CRM/domain-specs/crm_email-spec.md`
