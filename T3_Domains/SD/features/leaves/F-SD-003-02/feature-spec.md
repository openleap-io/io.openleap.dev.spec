# F-SD-003-02 — Invoice Review

> **Conceptual Stack Layer:** Domain-Feature
> **Space:** Business
> **Owner:** SD Product Team
> **Companion files:** `F-SD-003-02.uvl`, `F-SD-003-02.aui.yaml`
> **Referenced by:** Suite Feature Catalog SS6
> **References:** `domain-specs/sd_bil-spec.md` (backend)

> **Meta Information**
> - **Version:** 2026-04-04
> - **Template:** `feature-spec.md` v1.0.0
> - **Template Compliance:** 100%
> - **Status:** DRAFT
> - **Feature ID:** `F-SD-003-02`
> - **Suite:** `sd`
> - **Node type:** LEAF
> - **Parent:** `F-SD-003` — Billing & Invoicing
> - **Companion UVL:** `F-SD-003-02.uvl`
> - **Companion AUI:** `F-SD-003-02.aui.yaml`

---

## ═══════════════════════════════════════════════
## PROBLEM SPACE
## ═══════════════════════════════════════════════

## 0. Feature Identity & Orientation

### 0.1 One-Line Summary
This feature lets a **finance manager** review, approve, or hold generated invoices before posting to FI.

### 0.2 Non-Goals
- Does not execute billing runs — that is F-SD-003-01.
- Does not manage credit notes — that is F-SD-003-03.
- Does not post to FI directly (posting is triggered via F-SD-003-01 Post All or individually).

### 0.3 Entry & Exit Points

**Entry points:**
- Billing menu → "Invoices"
- Direct URL: `/sd/bil/invoices`
- From billing run results → "View Invoices"

**Exit points:**
- Invoice approved → eligible for posting in F-SD-003-01
- Invoice on hold → stays in HELD status until released

### 0.4 Variability Points

| Variability Point | Model | Values | Default | Binding Time |
|---|---|---|---|---|
| Dual-approval required | UVL attribute `Boolean dual_approval_required` | true / false | false | deploy |

---

## 1. User Goal & Scenarios

### 1.1 User Goal
Verify generated invoices for correctness before they are posted to accounting, ensuring only accurate and approved invoices reach the FI suite.

### 1.2 Scenarios

| # | Scenario | Precondition | Action | Expected Outcome |
|---|----------|-------------|--------|-----------------|
| S1 | Browse invoices | Billing run completed | Open invoice list | Paginated list with invoice number, customer, amount, status |
| S2 | Filter by status | Invoice list displayed | Select status = GENERATED | Only generated (unreviewed) invoices shown |
| S3 | Review invoice detail | Invoice list displayed | Click invoice row | Invoice detail with line items, amounts, sales order reference |
| S4 | Approve | Invoice in GENERATED status | Click "Approve" | Invoice status → APPROVED; eligible for posting |
| S5 | Put on hold | Invoice in GENERATED status | Click "Hold" with reason | Invoice status → HELD; excluded from bulk posting |

---

## 2. User Journey & Screen Layout

### 2.1 Sequence Diagram

```mermaid
sequenceDiagram
    actor FM as Finance Manager
    participant BFF
    participant BIL as sd-bil-svc

    FM->>BFF: GET /sd/bil/invoices
    BFF->>BIL: GET /api/sd/bil/v1/invoices?page=0&size=25
    BIL-->>BFF: 200 { invoices[], pagination }
    BFF-->>FM: Render invoice list

    FM->>BFF: Click invoice row
    BFF->>BIL: GET /api/sd/bil/v1/invoices/{id}
    BIL-->>BFF: 200 { invoice detail }
    BFF-->>FM: Render invoice detail

    FM->>BFF: Approve invoice
    BFF->>BIL: POST /api/sd/bil/v1/invoices/{id}/approve
    BIL-->>BFF: 200 { status: APPROVED }
    BFF-->>FM: Success toast; status badge updated
```

### 2.2 Screen Layout

```
┌─────────────────────────────────────────────────────┐
│ Invoices                                             │
├─────────────────────────────────────────────────────┤
│ [Status: All ▾]  [Run: All ▾]  [Customer: ___]      │
├──────────────┬──────────────┬────────────┬─────────┤
│ Invoice #    │ Customer     │ Status     │ Amount  │
├──────────────┼──────────────┼────────────┼─────────┤
│ INV-2026-001 │ Acme Corp    │ GENERATED  │ 250.00  │ [Approve] [Hold] │
│ INV-2026-002 │ Globex Inc   │ APPROVED   │ 800.00  │ [View]           │
├──────────────┴──────────────┴────────────┴─────────┤
│ [EXT: extension zone]                                │
├─────────────────────────────────────────────────────┤
│ Showing 1-25 of 42     [← Prev] [1] [2] [Next →]   │
└─────────────────────────────────────────────────────┘
```

---

## 3. Interaction Requirements

### 3.1 Fields Table

| Field | Type | Required | Editable | Validation | i18n Key |
|---|---|---|---|---|---|
| Status filter | select | No | Yes | ALL, GENERATED, APPROVED, HELD, POSTED | `F-SD-003-02.filter.status` |
| Run filter | select | No | Yes | All billing runs | `F-SD-003-02.filter.run` |
| Customer search | text | No | Yes | Min 2 chars | `F-SD-003-02.filter.customer` |
| Hold reason | text area | Conditional | Yes | Required when placing on hold | `F-SD-003-02.field.holdReason` |

### 3.2 Actions Table

| Action | Trigger | Precondition | Effect |
|---|---|---|---|
| Approve | Button click | Invoice in GENERATED status | POST approve; status → APPROVED |
| Hold | Button click | Invoice in GENERATED status | POST hold with reason; status → HELD |
| Release Hold | Button click | Invoice in HELD status | POST release; status → GENERATED |
| View Detail | Row click | — | Navigate to invoice detail |

### 3.3 Validation Messages

| Field | Condition | Message |
|---|---|---|
| Hold Reason | Empty when holding | "Please provide a reason for placing this invoice on hold." |

---

## 4. Edge Cases & Screen States

### 4.1 Component States

| State | When | Behaviour |
|---|---|---|
| **Loading** | Awaiting API | Skeleton rows |
| **Empty** | No invoices match filter | "No invoices found. Adjust your filters." |
| **Error** | sd-bil-svc unavailable | Error banner with retry |

### 4.2 Specific Edge Cases

| Case | Behaviour | Affected users |
|---|---|---|
| dual_approval_required = true | Invoice requires two distinct FINANCE_MANAGER approvals | FINANCE_MANAGER |
| Invoice already POSTED | Approve/Hold buttons hidden; read-only view | All users |

### 4.3 Attribute-Driven Behaviour Changes

| Attribute | Non-default value | Observable change |
|---|---|---|
| `dual_approval_required` | true | Approval status shows "1/2 Approvals"; second approval button shown |

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
| 1 | sd-bil-svc | `GET /api/sd/bil/v1/invoices` | T3 | No | Show error + retry |
| 2 | sd-bil-svc | `GET /api/sd/bil/v1/invoices/{id}` | T3 | No | Show error + retry |
| 3 | sd-bil-svc | `POST /api/sd/bil/v1/invoices/{id}/approve` | T3 | Yes | Show error banner |
| 4 | sd-bil-svc | `POST /api/sd/bil/v1/invoices/{id}/hold` | T3 | Yes | Show error banner |

### 5.2 BFF View-Model Shape

```jsonc
{
  "invoices": [
    {
      "invoiceId": "inv-uuid",
      "invoiceNumber": "INV-2026-001",
      "customerName": "Acme Corp",
      "status": "GENERATED",
      "totalAmount": 250.00,
      "currency": "EUR",
      "billingRunId": "run-uuid"
    }
  ],
  "pagination": { "page": 0, "size": 25, "totalElements": 42, "totalPages": 2 }
}
```

### 5.3 Feature-Gating Rules

| Mode | Behaviour |
|---|---|
| Full | Browse, approve, hold |
| Read-only | Browse only; Approve/Hold hidden |
| Excluded | Menu item hidden; URL returns 404 |

### 5.4 Failure Modes

| Failure | User Experience |
|---------|----------------|
| sd-bil-svc down | Error banner; all actions disabled |

### 5.5 Caching Hints
BFF MAY cache invoice list for 30 seconds. Cache MUST be invalidated on `sd.bil.invoice.approved` or `sd.bil.invoice.held`.

### 5.6 i18n Keys

| Key | Default (en) |
|-----|-------------|
| `F-SD-003-02.title` | `Invoices` |
| `F-SD-003-02.filter.status` | `Status` |
| `F-SD-003-02.filter.run` | `Billing Run` |
| `F-SD-003-02.filter.customer` | `Customer` |
| `F-SD-003-02.field.holdReason` | `Hold Reason` |
| `F-SD-003-02.action.approve` | `Approve` |
| `F-SD-003-02.action.hold` | `Hold` |

---

## 6. AUI Screen Contract

See companion file `F-SD-003-02.aui.yaml`.

---

## ═══════════════════════════════════════════════
## BRIDGE ARTIFACTS
## ═══════════════════════════════════════════════

## 7. Permissions & Accessibility

### 7.1 Permission Matrix

| Action | FINANCE_MANAGER | BILLING_CLERK | SALES_MANAGER |
|---|---|---|---|
| Browse invoices | ✓ | ✓ | — |
| Review detail | ✓ | ✓ | — |
| Approve | ✓ | — | — |
| Hold / Release | ✓ | — | — |

### 7.2 Accessibility
- Status badges MUST use `aria-label` not just colour.
- Approve and Hold buttons MUST include invoice number in `aria-label`.

---

## 8. Acceptance Criteria

| AC | Scenario | Given | When | Then |
|----|----------|-------|------|------|
| AC-01 | S1 | Billing run completed | FM opens invoice list | Paginated list displayed |
| AC-02 | S2 | Invoice list | FM filters by GENERATED | Only generated invoices shown |
| AC-03 | S3 | Invoice list | FM clicks invoice row | Invoice detail with lines and amounts |
| AC-04 | S4 | Invoice GENERATED | FM clicks Approve | Status → APPROVED |
| AC-05 | S5 | Invoice GENERATED | FM clicks Hold with reason | Status → HELD |
| AC-06 | Role gate | BILLING_CLERK | Attempts approve | Approve button absent |

---

## 9. Variability & Extension

### 9.1 Feature Dependencies
Requires IAM authentication. Requires F-SD-003-01 (Billing Run). Required by F-SD-003-03.

### 9.2 Attributes
See §0.4 variability points. Binding time: `deploy`.

### 9.3 Extension Points
| Extension Zone | Interface | Default Behaviour |
|---|---|---|
| `ext.invoiceReviewActions` | Additional review actions (e.g., dispute button) | Hidden (no extension) |

### 9.4 Companion UVL
See `uvl/leaves/F-SD-003-02.uvl`.

---

**END OF SPECIFICATION**
