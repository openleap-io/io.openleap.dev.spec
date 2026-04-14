# F-SD-003-03 — Credit Note Management

> **Conceptual Stack Layer:** Domain-Feature
> **Space:** Business
> **Owner:** SD Product Team
> **Companion files:** `F-SD-003-03.uvl`, `F-SD-003-03.aui.yaml`
> **Referenced by:** Suite Feature Catalog SS6
> **References:** `domain-specs/sd_bil-spec.md` (backend)

> **Meta Information**
> - **Version:** 2026-04-04
> - **Template:** `feature-spec.md` v1.0.0
> - **Template Compliance:** 100%
> - **Status:** DRAFT
> - **Feature ID:** `F-SD-003-03`
> - **Suite:** `sd`
> - **Node type:** LEAF
> - **Parent:** `F-SD-003` — Billing & Invoicing
> - **Companion UVL:** `F-SD-003-03.uvl`
> - **Companion AUI:** `F-SD-003-03.aui.yaml`

---

## ═══════════════════════════════════════════════
## PROBLEM SPACE
## ═══════════════════════════════════════════════

## 0. Feature Identity & Orientation

### 0.1 One-Line Summary
This feature lets a **billing clerk** create and manage credit notes for customer returns and billing corrections.

### 0.2 Non-Goals
- Does not process physical returns — that is sd.ret.
- Does not approve invoices — that is F-SD-003-02.
- Does not post credit notes directly to FI — posting triggers the FI integration event.

### 0.3 Entry & Exit Points

**Entry points:**
- Billing menu → "Credit Notes"
- Direct URL: `/sd/bil/credit-notes`
- From return approval in sd.ret → auto-link to credit note draft

**Exit points:**
- Credit note posted → `sd.bil.credit-note.posted` published; FI suite processes
- Back to billing dashboard

### 0.4 Variability Points

| Variability Point | Model | Values | Default | Binding Time |
|---|---|---|---|---|
| Auto-create credit note on return | UVL attribute `Boolean auto_credit_on_return` | true / false | false | deploy |

---

## 1. User Goal & Scenarios

### 1.1 User Goal
Compensate customers for returned goods or billing errors by creating an approved and posted credit note that reduces the outstanding balance.

### 1.2 Scenarios

| # | Scenario | Precondition | Action | Expected Outcome |
|---|----------|-------------|--------|-----------------|
| S1 | Create credit note | Invoice in APPROVED or POSTED status | Open credit note form; link to original invoice | Credit note draft created |
| S2 | Link to return/complaint | Credit note in DRAFT status | Search and link sd.ret return document | Return reference added to credit note |
| S3 | Approve credit note | Credit note in DRAFT status; user has FINANCE_MANAGER role | Click "Approve" | Credit note status → APPROVED |
| S4 | Post credit note | Credit note APPROVED | Click "Post" | Credit note posted; `sd.bil.credit-note.posted` published; FI processes debit |

---

## 2. User Journey & Screen Layout

### 2.1 Sequence Diagram

```mermaid
sequenceDiagram
    actor Clerk as Billing Clerk
    participant BFF
    participant BIL as sd-bil-svc

    Clerk->>BFF: GET /sd/bil/credit-notes
    BFF->>BIL: GET /api/sd/bil/v1/credit-notes
    BIL-->>BFF: 200 { credit-notes[] }
    BFF-->>Clerk: Render credit note list

    Clerk->>BFF: POST new credit note
    BFF->>BIL: POST /api/sd/bil/v1/credit-notes
    BIL-->>BFF: 201 { creditNoteId, status: DRAFT }
    BFF-->>Clerk: Navigate to credit note detail

    Clerk->>BFF: Approve (FINANCE_MANAGER)
    BFF->>BIL: POST /api/sd/bil/v1/credit-notes/{id}/approve
    BIL-->>BFF: 200 { status: APPROVED }

    Clerk->>BFF: Post credit note
    BFF->>BIL: POST /api/sd/bil/v1/credit-notes/{id}/post
    BIL-->>BFF: 202 { status: POSTED }
    BFF-->>Clerk: Success toast; event published
```

### 2.2 Screen Layout

```
┌─────────────────────────────────────────────────────┐
│ Credit Notes                          [+ New]        │
├─────────────────────────────────────────────────────┤
│ ┌──────────────┬──────────────┬────────────┬───────┐ │
│ │ Credit Note# │ Customer     │ Status     │Amount │ │
│ ├──────────────┼──────────────┼────────────┼───────┤ │
│ │ CN-2026-001  │ Acme Corp    │ DRAFT      │-50.00 │ [Approve] │
│ │ CN-2026-002  │ Globex Inc   │ APPROVED   │-120.00│ [Post]    │
│ └──────────────┴──────────────┴────────────┴───────┘ │
├─────────────────────────────────────────────────────┤
│ [EXT: extension zone]                                │
└─────────────────────────────────────────────────────┘
```

---

## 3. Interaction Requirements

### 3.1 Fields Table

| Field | Type | Required | Editable | Validation | i18n Key |
|---|---|---|---|---|---|
| Original Invoice | search/select | Yes | Yes | Must be APPROVED or POSTED invoice | `F-SD-003-03.field.originalInvoice` |
| Credit Amount | number | Yes | Yes | Negative; ≤ invoice total | `F-SD-003-03.field.creditAmount` |
| Reason | select | Yes | Yes | RETURN, PRICE_CORRECTION, DUPLICATE | `F-SD-003-03.field.reason` |
| Return Reference | text | No | Yes | sd.ret document ID | `F-SD-003-03.field.returnRef` |

### 3.2 Actions Table

| Action | Trigger | Precondition | Effect |
|---|---|---|---|
| Create Credit Note | Button click | Original invoice selected | POST credit note; status DRAFT |
| Link Return | Field fill | Credit note DRAFT | Attach return reference |
| Approve | Button click | FINANCE_MANAGER role; credit note DRAFT | POST approve; status APPROVED |
| Post | Button click | Credit note APPROVED | POST post; async; event published |

### 3.3 Validation Messages

| Field | Condition | Message |
|---|---|---|
| Original Invoice | Not selected | "Please select the original invoice for this credit note." |
| Credit Amount | Exceeds invoice total | "Credit amount cannot exceed the original invoice total." |
| Credit Amount | > 0 | "Credit amount must be negative." |

---

## 4. Edge Cases & Screen States

### 4.1 Component States

| State | When | Behaviour |
|---|---|---|
| **Loading** | Awaiting API | Skeleton rows |
| **DRAFT** | Credit note created, not approved | Approve button shown; Post disabled |
| **APPROVED** | Credit note approved | Post button shown |
| **POSTED** | Credit note posted | Read-only; no actions |

### 4.2 Specific Edge Cases

| Case | Behaviour | Affected users |
|---|---|---|
| auto_credit_on_return = true | Credit note draft auto-created from return approval; clerk only needs to review and approve | BILLING_CLERK |
| Partial credit | Credit amount < invoice total; partial credit note allowed | BILLING_CLERK |

### 4.3 Attribute-Driven Behaviour Changes

| Attribute | Non-default value | Observable change |
|---|---|---|
| `auto_credit_on_return` | true | New button replaced by "Review Auto-Credits" section |

### 4.4 Connectivity
This feature requires a live connection.

---

## ═══════════════════════════════════════════════
## SOLUTION SPACE
## ═══════════════════════════════════════════════

## 5. Backend Dependencies & BFF Contract

### 5.1 Service Calls

| # | Service | Endpoint | Tier | isMutation | Failure Mode |
|---|---------|----------|------|------------|-------------|
| 1 | sd-bil-svc | `GET /api/sd/bil/v1/credit-notes` | T3 | No | Show error + retry |
| 2 | sd-bil-svc | `POST /api/sd/bil/v1/credit-notes` | T3 | Yes | Show error banner |
| 3 | sd-bil-svc | `POST /api/sd/bil/v1/credit-notes/{id}/approve` | T3 | Yes | Show error banner |
| 4 | sd-bil-svc | `POST /api/sd/bil/v1/credit-notes/{id}/post` | T3 | Yes | Show error banner |

### 5.2 BFF View-Model Shape

```jsonc
{
  "creditNotes": [
    {
      "creditNoteId": "cn-uuid",
      "creditNoteNumber": "CN-2026-001",
      "customerName": "Acme Corp",
      "status": "DRAFT",
      "creditAmount": -50.00,
      "currency": "EUR",
      "originalInvoiceId": "inv-uuid",
      "returnReference": null
    }
  ]
}
```

### 5.3 Feature-Gating Rules

| Mode | Behaviour |
|---|---|
| Full | Create, approve, post |
| Read-only | List and detail only |
| Excluded | Menu item hidden; URL returns 404 |

### 5.4 Failure Modes

| Failure | User Experience |
|---------|----------------|
| sd-bil-svc down | Error banner; all actions disabled |

### 5.5 Caching Hints
BFF MUST NOT cache credit note form. List MAY cache 30 seconds. Invalidated on `sd.bil.credit-note.posted`.

### 5.6 i18n Keys

| Key | Default (en) |
|-----|-------------|
| `F-SD-003-03.title` | `Credit Notes` |
| `F-SD-003-03.field.originalInvoice` | `Original Invoice` |
| `F-SD-003-03.field.creditAmount` | `Credit Amount` |
| `F-SD-003-03.field.reason` | `Reason` |
| `F-SD-003-03.field.returnRef` | `Return Reference` |
| `F-SD-003-03.action.approve` | `Approve` |
| `F-SD-003-03.action.post` | `Post Credit Note` |

---

## 6. AUI Screen Contract

See companion file `F-SD-003-03.aui.yaml`.

---

## ═══════════════════════════════════════════════
## BRIDGE ARTIFACTS
## ═══════════════════════════════════════════════

## 7. Permissions & Accessibility

### 7.1 Permission Matrix

| Action | BILLING_CLERK | FINANCE_MANAGER | CUSTOMER_SERVICE |
|---|---|---|---|
| View credit notes | ✓ | ✓ | ✓ |
| Create credit note | ✓ | ✓ | — |
| Approve credit note | — | ✓ | — |
| Post credit note | — | ✓ | — |

### 7.2 Accessibility
- Credit amount MUST display with sign indicator for screen readers: "Credit: -50.00 EUR".
- Post button MUST be `aria-disabled` when credit note not yet APPROVED.

---

## 8. Acceptance Criteria

| AC | Scenario | Given | When | Then |
|----|----------|-------|------|------|
| AC-01 | S1 | Invoice APPROVED | Clerk creates credit note | CN draft created linked to invoice |
| AC-02 | S2 | CN DRAFT | Clerk links return document | Return reference stored on CN |
| AC-03 | S3 | CN DRAFT | FM approves | Status → APPROVED |
| AC-04 | S4 | CN APPROVED | FM posts | Status → POSTED; `sd.bil.credit-note.posted` published |
| AC-05 | Role gate | BILLING_CLERK | Attempts approve | Approve button absent |
| AC-06 | auto_credit | auto_credit_on_return = true | Return approved | CN draft auto-created; clerk sees it in list |

---

## 9. Variability & Extension

### 9.1 Feature Dependencies
Requires IAM authentication. Requires F-SD-003-02 (Invoice Review) for the approval workflow.

### 9.2 Attributes
See §0.4 variability points. Binding time: `deploy`.

### 9.3 Extension Points
| Extension Zone | Interface | Default Behaviour |
|---|---|---|
| `ext.creditNoteFields` | Additional credit note fields (e.g., dispute reference) | Hidden (no extension) |

### 9.4 Companion UVL
See `uvl/leaves/F-SD-003-03.uvl`.

---

**END OF SPECIFICATION**
