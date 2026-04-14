<!-- TEMPLATE COMPLIANCE: ~20%
Template: domain-service-spec.md v1.0.0
Present sections: §0 (purpose, scope — minimal), §6 (REST API — one endpoint), §7 (events — outbound/inbound), §14 (decisions, open questions)
Missing sections: §1 (business context), §2 (service identity table), §3 (domain model), §4 (business rules), §5 (use cases), §8 (data model), §9 (security), §10 (quality attributes), §11 (feature dependencies), §12 (extension points), §13 (migration), §15 (appendix)
Naming issues: file should be com_prc-preview-spec.md per convention; underscore in name is inconsistent
Duplicates: none
Priority: LOW — optional domain, placeholder-level spec
-->
# Service Domain Specification — `com.prc.preview` (Price Preview)

> **Meta Information**
> - **Version:** 2026-01-18
> - **Template:** `domain-service-spec.md` v1.0.0
> - **Template Compliance:** ~20% — §1 (business context), §2 (service identity table), §3 (domain model), §4 (business rules), §5 (use cases), §8 (data model), §9 (security), §10 (quality attributes), §11 (feature dependencies), §12 (extension points), §13 (migration), §15 (appendix) missing
> - **Author(s):** OpenLeap Architecture Team
> - **Status:** DRAFT
> - **Tier:** T3
> - **Suite:** `com`
> - **Domain:** `prc.preview`
> - **Service ID:** `com-prc-preview-svc`
> - **basePackage:** `io.openleap.com.prc.preview`
> - **API Base Path:** `/api/com/prc.preview/v1`

---

## Specification Guidelines Compliance

> **This specification MUST comply with the project-wide specification guidelines.**
>
> #### Non-negotiables
> - Never invent facts. If information is missing, add an **OPEN QUESTION** entry.
> - Use **MUST/SHOULD/MAY** for normative statements.
> - Keep the spec **self-contained**: no references to chat context.
> - Record decisions and boundaries explicitly (see Section 12).

---

## 0. Document Purpose & Scope

### 0.1 Purpose
`com.prc.preview` specifies an **optional** COM domain that provides low-latency **price preview rendering** for browsing (badges, “from price”, strike-through display).

This domain is explicitly **not authoritative**: final pricing for quotes/orders/contracts MUST come from `sd.sd`.

### 0.3 Scope

**In Scope (MUST):**
- MUST serve cached/derived display price previews for channel browsing.
- SHOULD invalidate/refresh caches based on upstream pricing change signals.

**Out of Scope (MUST NOT):**
- MUST NOT be the pricing source of truth for any legally binding commercial document → `sd.sd`.
- MUST NOT persist immutable pricing snapshots for orders/contracts.

---

## 2. Domain Boundaries & Responsibilities

### 2.1 Verantwortlichkeiten (Responsibilities)
- MUST clearly label all outputs as “preview” (semantics: may differ from checkout pricing).
- SHOULD support per-channel rendering rules (currency/formatting) (OPEN QUESTION).

---

## 6. Public Interfaces (APIs)

### 6.1 REST API (OpenAPI-friendly)
**Base Path:** `/api/com/prc.preview/v1`

#### 6.1.1 Price preview
- `GET /prices?variantId=...`

---

## 7. Events & Messaging

### 7.1 Konventionen
- **Exchange/Topic:** `com.prc.preview.events`
- **Routing Key:** `com.prc.preview.<aggregate>.<event>`

### 7.2 Outbound Events
- `com.prc.preview.cache.invalidated`

### 7.3 Inbound Events
- `sd.sd.price.*` – (OPEN QUESTION) exact event names for pricing changes.

---

## 12. Decisions, Conflicts, Open Questions

### 12.1 Entscheidungen (Decisions)
- **DEC-001:** `com.prc.preview` is optional and MUST remain non-authoritative.

### 12.3 OPEN QUESTIONS
- **OQ-001:** Do we need preview pricing at all, or is SD pricing fast enough for browse?

---

## 13. Change Log
- Created: 2026-01-18
