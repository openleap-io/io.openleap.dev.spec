<!-- Template Meta
     Template-ID:   TPL-SVC
     Version:       1.1.0
     Compliance:    95%+
-->

# Channel Service — Domain Specification

> **Conceptual Stack Layer:** Domain / Service
> **Space:** Platform
> **Owner:** team-tks

> **Meta Information**
> - **Version:** 2026-04-18
> - **Template:** `domain-service-spec.md` v1.1.0
> - **Template Compliance:** 95%+
> - **Status:** DRAFT
> - **Service ID:** `tks-ch-svc`
> - **Suite:** `tks`
> - **Domain:** `ch`
> - **Base Package:** `io.openleap.tks.ch`
> - **API Path:** `/api/tks/ch/v1`
> - **Port:** 8501
> - **DB Schema:** `tks_ch`
> - **Tier:** T3 — Core Business Suite
> - **Bounded Context:** `bc:channels`

---

## 0. Document Purpose & Scope

This document specifies the **Channel Service** (`tks-ch-svc`): the polymorphic inbound/outbound gateway for TKS. Supports email, web form, chat, webhook, and API channels. Normalises inbound messages, resolves parties, deduplicates conversations, and produces events for `tks-tkt-svc`.

**In scope:** `Channel`, `Conversation`, `InboundMessage`, `OutboundReply`, `ChannelIdentity`.
**Out of scope:** Email transport (→ `tech.email`); AI triage (→ `tech.ai` called by this service).

**Related documents:**
- Suite Spec: `T3_Domains/TKS/_tks_suite.md`
- OpenAPI: `contracts/http/tks/ch/openapi.yaml`
- Event Schemas: `contracts/events/tks/ch/*.schema.json`

---

## 1. Business Context

### 1.1 Purpose & Responsibility

Own the per-tenant registry of inbound channels; accept and normalise inbound messages; match them to existing conversations; publish canonical events; dispatch outbound replies through the correct transport. Polymorphism: each channel type owns its adapter while sharing a common conversation schema.

**Owns:**
- `Channel` — root, polymorphic config
- `Conversation` — ordered thread of messages tied (eventually) to one ticket
- `InboundMessage` — raw + normalised inbound payload
- `OutboundReply` — queued outbound
- `ChannelIdentity` — party's handle in a channel (email address, chat user id, phone number)

### 1.2 Business Value

Single ingestion fabric: add a new channel type by writing an adapter. Deduplication and party matching happen once, in one place.

---

## 2. Service Identity

| Field | Value |
|-------|-------|
| Service ID | `tks-ch-svc` |
| Suite | `tks` |
| Domain | `ch` |
| Bounded Context | `bc:channels` |
| Base Package | `io.openleap.tks.ch` |
| API Base Path | `/api/tks/ch/v1` |
| Port | 8501 |
| Database Schema | `tks_ch` |
| Tier | T3 |
| Repository | `openleap-io/io.openleap.tks.ch` |
| Team | `team-tks` |

---

## 3. Domain Model

### 3.1 Aggregates

**Channel** (AggregateRoot, polymorphic)

| Attribute | Type | Notes |
|---|---|---|
| id | uuid | |
| tenantId | uuid | RLS |
| type | enum | `EMAIL`,`WEB_FORM`,`CHAT`,`WEBHOOK`,`API` |
| name | string | display |
| config | JSONB | per-type (email: `mailbox`, `tech_email_account_id`; web: `form_id`, `portal_scope`; webhook: `secret_ref`; chat: `provider`, `credential_ref`; api: `token_prefix`) |
| status | enum | `ACTIVE`,`DISABLED` |
| autoCreateTicket | bool | default true |
| aiTriageEnabled | bool | default false |
| defaultQueueId | uuid? | routing |
| version | int | |

**Conversation** (AggregateRoot)

| Attribute | Type | Notes |
|---|---|---|
| id | uuid | |
| tenantId | uuid | RLS |
| channelId | uuid | FK |
| ticketId | uuid? | set once converted |
| threadKey | string | per-channel natural key (message-id / form submission id / chat room id) |
| reporterPartyId | uuid? | → `shared.bp` |
| status | enum | `OPEN`,`CLOSED` |
| lastActivityAt | timestamp | |

**InboundMessage** (child entity of Conversation)

| Attribute | Type | Notes |
|---|---|---|
| id | uuid | |
| conversationId | uuid | FK |
| rawPayload | JSONB | full provider payload (encrypted at rest) |
| normalizedBody | text | stripped + formatted |
| subject | string? | |
| attachments | JSONB | metadata; bodies moved to `tech.dms` |
| providerMessageId | string | unique per channel (dedupe) |
| classification | enum | `NEW`,`FOLLOW_UP`,`SPAM`,`DUPLICATE` |
| aiTriageResult | JSONB? | `{category, priority, suggestedKbArticleId}` |
| receivedAt | timestamp | |

**OutboundReply** (AggregateRoot)

| Attribute | Type | Notes |
|---|---|---|
| id | uuid | |
| tenantId | uuid | RLS |
| conversationId | uuid | FK |
| ticketId | uuid | FK |
| channelId | uuid | FK |
| body | text | |
| attachmentDmsIds | uuid[] | |
| status | enum | `QUEUED`,`SENT`,`FAILED` |
| providerRef | string? | external id once sent |

**ChannelIdentity** (VO per Party per Channel)

| Attribute | Type | Notes |
|---|---|---|
| partyId | uuid | → `shared.bp` |
| channelType | enum | |
| handle | string | e.g. `alice@example.com`, `+49...`, `slack:U12345` |
| confidence | int | 0..100 |
| verifiedAt | timestamp? | |

---

## 4. Business Rules

### 4.1 Rule Catalog

| ID | Rule | Type |
|----|------|------|
| BR-CH-001 | Each `Channel` MUST be unique per `(tenantId, type, naturalKey)`; duplicate registration returns 409 | CONSTRAINT |
| BR-CH-002 | At least one `ACTIVE` channel per tenant for `autoCreateTicket=true` to operate | POLICY |
| BR-CH-003 | `InboundMessage` dedupe by `(channelId, providerMessageId)`; duplicates stored but not republished | CONSTRAINT |
| BR-CH-004 | Inbound sender MUST resolve to a `shared.bp` party; if none matches, create a provisional party with `source=tks.ch.inbound` | POLICY |
| BR-CH-005 | Outbound reply uses the same `channelId` as the originating inbound; cross-channel reply requires explicit agent switch | POLICY |
| BR-CH-006 | Attachments > 25 MB rejected at channel adapter; smaller ones uploaded to `tech.dms` | CONSTRAINT |
| BR-CH-007 | Spam classification → conversation status `CLOSED`; no ticket event fired | POLICY |
| BR-CH-008 | AI triage failure → ticket still created with `aiTriageFailed=true`; deferred rule picks up | POLICY |

---

## 5. Use Cases

### 5.1 Canonical Use Cases

| UC | Type | Trigger | Aggregate | Domain Operation | Events | REST |
|----|------|---------|-----------|------------------|--------|------|
| RegisterChannel | WRITE | REST | Channel | `Channel.register` | `tks.ch.channel.created` | `POST /channels` |
| DisableChannel | WRITE | REST | Channel | `Channel.disable` | `tks.ch.channel.disabled` | `POST /channels/{id}/disable` |
| IngestInbound | WRITE | Message (from `tech.email`, webhook, form) | Conversation | `Conversation.ingest` | `tks.ch.conversation.started`, `tks.ch.conversation.message-received` | — |
| SendReply | WRITE | REST | OutboundReply | `OutboundReply.queue` | `tks.ch.conversation.message-sent` | `POST /conversations/{id}/replies` |
| ListConversations | READ | REST | Conversation | query | — | `GET /conversations` |
| GetConversation | READ | REST | Conversation | query | — | `GET /conversations/{id}` |
| IngestWebhook | WRITE | REST (inbound webhook endpoint per channel) | Conversation | `Conversation.ingest` | `tks.ch.conversation.message-received` | `POST /webhooks/{channelId}` |
| SubmitWebForm | WRITE | REST (public portal) | Conversation | `Conversation.ingest` | `tks.ch.conversation.message-received` | `POST /web/{channelId}/submit` |

---

## 6. REST API

**Base Path:** `/api/tks/ch/v1`
**Authentication:** OAuth2 / JWT (internal endpoints); signed webhook (public inbound). Scopes `tks.ch:read`, `tks.ch:write`, `tks.ch:admin`.

### 6.1 Endpoint Summary

| Method | Path | Description |
|--------|------|-------------|
| `POST` | `/channels` | Register channel (admin) |
| `GET` | `/channels` | List channels |
| `GET` | `/channels/{id}` | Read |
| `PATCH` | `/channels/{id}` | Update config |
| `POST` | `/channels/{id}/disable` | Disable |
| `POST` | `/channels/{id}/enable` | Enable |
| `POST` | `/webhooks/{channelId}` | Public inbound webhook (HMAC-validated) |
| `POST` | `/web/{channelId}/submit` | Public web-form submit (captcha / session required) |
| `GET` | `/conversations` | List (filters: channelId, status, reporterPartyId) |
| `GET` | `/conversations/{id}` | Read + message log |
| `POST` | `/conversations/{id}/replies` | Queue outbound |
| `POST` | `/conversations/{id}/close` | Close (e.g. spam) |

---

## 7. Events & Integration

### 7.1 Pattern

`event_driven` + reactive ingestion. Follows suite pattern.

### 7.2 Published Events

| Event | Routing Key |
|-------|------------|
| ChannelCreated | `tks.ch.channel.created` |
| ChannelDisabled | `tks.ch.channel.disabled` |
| ConversationStarted | `tks.ch.conversation.started` |
| ConversationMessageReceived | `tks.ch.conversation.message-received` |
| ConversationMessageSent | `tks.ch.conversation.message-sent` |
| ConversationClosed | `tks.ch.conversation.closed` |

### 7.3 Consumed Events

| Source | Routing Key | Handler Purpose |
|--------|-------------|-----------------|
| `tech-email-svc` | `tech.email.email.received` | Create / append InboundMessage for email channels bound to that mailbox |
| `tech-email-svc` | `tech.email.email.bounced` | Mark outbound as failed |
| `shared-bp-svc` | `shared.bp.party.merged` | Update `ChannelIdentity` on surviving party |
| `shared-bp-svc` | `shared.bp.party.erased` | GDPR anonymise inbound payloads + identities |
| `tks-tkt-svc` | `tks.tkt.ticket.resolved` | Close conversation if configured `autoCloseOnResolve` |

---

## 8. Data Model

**Storage:** PostgreSQL 16, schema `tks_ch`. Raw payload column encrypted at rest with platform KMS key.

Key tables: `channels`, `conversations`, `inbound_messages`, `outbound_replies`, `channel_identities`, `outbox_events`.

Indexes: `(tenant_id, channel_id, status, last_activity_at DESC)` on conversations; unique `(channel_id, provider_message_id)` on inbound_messages.

---

## 9. Security & Compliance

### 9.1 Access Control

| Role | Access Level |
|------|-------------|
| `tenant-admin` | Manage channels |
| `support-agent` | Read conversations; send replies |
| `public` (unauth) | Submit via signed webhook / captcha-protected web form |
| `automation-sender` | Internal use via ingestion bus |

### 9.2 Data Classification

Confidential — raw inbound payloads MAY contain PII; encrypted at rest; redacted in logs.

### 9.3 Compliance

- **GDPR erasure** — payloads purged on `shared.bp.party.erased`; inbound raw retained max 90 days by default.
- **CAN-SPAM / EU ePrivacy** — outbound replies inherit deliverability policy from `tech.email`; public channels require captcha for unauth submission.

### 9.4 DORA ICT Risk References

| Risk ID | Title | Treatment |
|---|---|---|
| `RISK-TKS-CH-001` | Mass inbound storm (DoS) | Mitigate: per-channel rate limit + queue depth alarm + adaptive shedding |
| `RISK-TKS-CH-002` | Provider outage blocks intake | Mitigate: offline queue with replay; operator notification |

### 9.5 SBOM

CycloneDX per build; channel adapter jars reported individually.

---

## 10. Quality Attributes

### 10.1 Performance

| Operation | p95 | p99 |
|---|---|---|
| Inbound webhook ack | < 100 ms | < 300 ms |
| Conversation lookup | < 80 ms | < 200 ms |
| Outbound queue → provider submit | < 2 s | < 8 s |

### 10.2 Availability

99.9 %. Offline buffer retains inbound during downstream outages.

### 10.3 Scalability

Adapter workers scale horizontally per channel type.

### 10.5 SLI/SLO Reference

`SLO-TKS-CH-001` — availability 99.9%, webhook ack p95 < 150 ms.

### 10.6 Resilience

| Aspect | Value |
|---|---|
| RTO | < 15 min |
| RPO | < 5 min |
| Buffer | persisted inbound queue; replay supported |

---

## 11. Feature Dependencies

| Feature ID | Feature Name |
|-----------|-------------|
| `F-TKS-200` | Multi-Channel Intake |
| `F-TKS-210` | Email Channel Adapter |
| `F-TKS-220` | Web Form / Portal |
| `F-TKS-230` | Webhook Channel |
| `F-TKS-240` | Chat Channel (placeholder) |
| `F-TKS-510` | AI Auto-Triage (calls this service's ingest path) |

---

## 12. Extension Points

- New channel-type adapters register via `POST /channels/extensions/types`.
- Pre-ingest extension hook `tks.ch.ext.pre-ingest` (fail-open) for tenant-side filters (e.g., spam classifiers).
- Outbound pre-send hook for content transformation.

---

## 13. Migration & Evolution

- No direct predecessor; `crm.sup` had no channel abstraction (email arrived indirectly via `crm.email`).
- v1.1 adds SMS channel type; v1.2 adds official chat provider adapters.

---

## 14. Decisions & Open Questions

| # | Question | Severity | Status |
|---|----------|----------|--------|
| Q-CH-001 | Default chat provider at GA? | Low | Open — recommend none in v1 (placeholder only) |
| Q-CH-002 | Web-form tenant branding in `tks.ch` or a new `ui.portal` repo? | Medium | Open |

### ADRs

- **ADR-CH-001** *(Proposed)*: Polymorphic single-service adapter model (supports ADR-TKS-003).

---

## 15. Appendix

### 15.1 Glossary

See `_tks_suite.md` SS1.

### 15.2 Change Log

| Date | Version | Author | Changes |
|------|---------|--------|---------|
| 2026-04-18 | 1.0.0 | OpenLeap Architecture Team | Initial Channel Service spec |

### 15.3 Companion Files

- OpenAPI: `contracts/http/tks/ch/openapi.yaml`
- Event Schemas: `contracts/events/tks/ch/*.schema.json`
