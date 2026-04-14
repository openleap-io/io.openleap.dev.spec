# F-SD-003-01 — Billing Run

> **Conceptual Stack Layer:** Domain-Feature
> **Space:** Business
> **Owner:** SD Product Team
> **Companion files:** `F-SD-003-01.uvl`, `F-SD-003-01.aui.yaml`
> **Referenced by:** Suite Feature Catalog SS6
> **References:** `domain-specs/sd_bil-spec.md` (backend)

> **Meta Information**
> - **Version:** 2026-04-04
> - **Template:** `feature-spec.md` v1.0.0
> - **Template Compliance:** 100%
> - **Status:** DRAFT
> - **Feature ID:** `F-SD-003-01`
> - **Suite:** `sd`
> - **Node type:** LEAF
> - **Parent:** `F-SD-003` — Billing & Invoicing
> - **Companion UVL:** `F-SD-003-01.uvl`
> - **Companion AUI:** `F-SD-003-01.aui.yaml`

---

## ═══════════════════════════════════════════════
## PROBLEM SPACE
## ═══════════════════════════════════════════════

## 0. Feature Identity & Orientation

### 0.1 One-Line Summary
This feature lets a **billing clerk** execute a billing run to generate invoices for all due sales orders and deliveries.

### 0.2 Non-Goals
- Does not approve or post individual invoices — that is F-SD-003-02.
- Does not manage credit notes — that is F-SD-003-03.
- Does not post to FI directly — invoices are posted via FI suite integration after F-SD-003-02 approval.

### 0.3 Entry & Exit Points

**Entry points:**
- Billing menu → "Billing Run"
- Direct URL: `/sd/bil/runs`

**Exit points:**
- Run completed → navigate to F-SD-003-02 (Invoice Review) to review generated invoices
- Back to billing dashboard

### 0.4 Variability Points

| Variability Point | Model | Values | Default | Binding Time |
|---|---|---|---|---|
| Scheduled billing enabled | UVL attribute `Boolean scheduled_billing_enabled` | true / false | false | deploy |
| Billing run scope | UVL attribute `String billing_run_scope` | ALL, DELIVERED_ONLY, SELECTED | DELIVERED_ONLY | deploy |

---

## 1. User Goal & Scenarios

### 1.1 User Goal
Generate a batch of invoices for all delivered and unbilled orders so that revenue can be recognised and collected.

### 1.2 Scenarios

| # | Scenario | Precondition | Action | Expected Outcome |
|---|----------|-------------|--------|-----------------|
| S1 | Schedule billing run | At least one unbilled delivery | Enter run date and scope; click "Schedule" | Billing run created in SCHEDULED status |
| S2 | Execute run | Run is SCHEDULED | Click "Execute Now" | Run executes; invoices generated for all due deliveries |
| S3 | Review run results | Run COMPLETED | Click run row | Run detail shows count of invoices generated, errors |
| S4 | Post invoices | Run COMPLETED with reviewed invoices | Click "Post All" | All approved invoices posted to FI; `sd.bil.invoice.posted` published |

---

## 2. User Journey & Screen Layout

### 2.1 Sequence Diagram

```mermaid
sequenceDiagram
    actor Clerk as Billing Clerk
    participant BFF
    participant BIL as sd-bil-svc

    Clerk->>BFF: GET /sd/bil/runs
    BFF->>BIL: GET /api/sd/bil/v1/billing-runs
    BIL-->>BFF: 200 { runs[] }
    BFF-->>Clerk: Render billing runs list

    Clerk->>BFF: POST schedule billing run
    BFF->>BIL: POST /api/sd/bil/v1/billing-runs
    BIL-->>BFF: 201 { runId, status: SCHEDULED }
    BFF-->>Clerk: Run appears in list

    Clerk->>BFF: Execute run
    BFF->>BIL: POST /api/sd/bil/v1/billing-runs/{id}/execute
    BIL-->>BFF: 202 { status: RUNNING }
    BFF-->>Clerk: Progress indicator; poll for completion
```

### 2.2 Screen Layout

```
┌─────────────────────────────────────────────────────┐
│ Billing Runs                          [+ New Run]    │
├─────────────────────────────────────────────────────┤
│ ┌────────────┬────────────┬────────────┬──────────┐  │
│ │ Run ID     │ Date       │ Status     │ Invoices │  │
│ ├────────────┼────────────┼────────────┼──────────┤  │
│ │ RUN-0008   │ 2026-04-04 │ SCHEDULED  │ —        │ [Execute] │
│ │ RUN-0007   │ 2026-03-31 │ COMPLETED  │ 42       │ [View]    │
│ └────────────┴────────────┴────────────┴──────────┘  │
├─────────────────────────────────────────────────────┤
│ [EXT: extension zone]                                │
└─────────────────────────────────────────────────────┘
```

---

## 3. Interaction Requirements

### 3.1 Fields Table

| Field | Type | Required | Editable | Validation | i18n Key |
|---|---|---|---|---|---|
| Run Date | date picker | Yes | Yes | ≤ today | `F-SD-003-01.field.runDate` |
| Scope | select | Yes | Yes | ALL, DELIVERED_ONLY, SELECTED | `F-SD-003-01.field.scope` |

### 3.2 Actions Table

| Action | Trigger | Precondition | Effect |
|---|---|---|---|
| Schedule Run | Button click | Run date and scope set | POST billing run; status SCHEDULED |
| Execute Now | Button click | Run in SCHEDULED status | POST execute; async run |
| View Results | Row click | Run COMPLETED | Navigate to run detail |
| Post All | Button in run detail | All invoices reviewed | POST invoices to FI |

### 3.3 Validation Messages

| Field | Condition | Message |
|---|---|---|
| Run Date | In the future | "Billing run date cannot be in the future." |

---

## 4. Edge Cases & Screen States

### 4.1 Component States

| State | When | Behaviour |
|---|---|---|
| **Loading** | Awaiting API | Skeleton rows |
| **RUNNING** | Run executing | Progress indicator; Execute button disabled |
| **COMPLETED** | Run finished | Invoice count shown; View Results enabled |
| **ERROR** | Run failed | Error badge; error details in run detail |

### 4.2 Specific Edge Cases

| Case | Behaviour | Affected users |
|---|---|---|
| No due deliveries | Run completes with 0 invoices; warning shown | BILLING_CLERK |
| scheduled_billing_enabled = true | Runs triggered automatically by scheduler; manual execution still possible | BILLING_CLERK |

### 4.3 Attribute-Driven Behaviour Changes

| Attribute | Non-default value | Observable change |
|---|---|---|
| `scheduled_billing_enabled` | true | "Schedule" button replaced with "Run Now"; automated runs listed |
| `billing_run_scope` | ALL | Scope field pre-selected to ALL |

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
| 1 | sd-bil-svc | `GET /api/sd/bil/v1/billing-runs` | T3 | No | Show error + retry |
| 2 | sd-bil-svc | `POST /api/sd/bil/v1/billing-runs` | T3 | Yes | Show error banner |
| 3 | sd-bil-svc | `POST /api/sd/bil/v1/billing-runs/{id}/execute` | T3 | Yes | Show error banner |

### 5.2 BFF View-Model Shape

```jsonc
{
  "runs": [
    {
      "runId": "run-uuid",
      "runNumber": "RUN-0008",
      "runDate": "2026-04-04",
      "status": "SCHEDULED",
      "invoiceCount": null,
      "errorCount": null
    }
  ]
}
```

### 5.3 Feature-Gating Rules

| Mode | Behaviour |
|---|---|
| Full | Schedule, execute, view results |
| Read-only | List and view only; Schedule/Execute disabled |
| Excluded | Menu item hidden; URL returns 404 |

### 5.4 Failure Modes

| Failure | User Experience |
|---------|----------------|
| sd-bil-svc down | Error banner; all actions disabled |
| Run execution error | Run status = ERROR; details in run detail |

### 5.5 Caching Hints
BFF MAY cache run list for 30 seconds. Cache invalidated on `sd.bil.billing-run.completed`.

### 5.6 i18n Keys

| Key | Default (en) |
|-----|-------------|
| `F-SD-003-01.title` | `Billing Runs` |
| `F-SD-003-01.field.runDate` | `Run Date` |
| `F-SD-003-01.field.scope` | `Scope` |
| `F-SD-003-01.action.schedule` | `Schedule Run` |
| `F-SD-003-01.action.execute` | `Execute Now` |
| `F-SD-003-01.action.postAll` | `Post All Invoices` |

---

## 6. AUI Screen Contract

See companion file `F-SD-003-01.aui.yaml`.

---

## ═══════════════════════════════════════════════
## BRIDGE ARTIFACTS
## ═══════════════════════════════════════════════

## 7. Permissions & Accessibility

### 7.1 Permission Matrix

| Action | BILLING_CLERK | FINANCE_MANAGER | SALES_MANAGER |
|---|---|---|---|
| View billing runs | ✓ | ✓ | — |
| Schedule run | ✓ | ✓ | — |
| Execute run | ✓ | ✓ | — |
| Post invoices | — | ✓ | — |

### 7.2 Accessibility
- Run status MUST use `aria-label` not just colour badge.
- Execute button MUST be disabled with `aria-disabled` when run is RUNNING.

---

## 8. Acceptance Criteria

| AC | Scenario | Given | When | Then |
|----|----------|-------|------|------|
| AC-01 | S1 | Unbilled deliveries exist | Clerk schedules run | RUN-xxxx created SCHEDULED |
| AC-02 | S2 | Run SCHEDULED | Clerk executes | Run → RUNNING; async processing starts |
| AC-03 | S3 | Run COMPLETED | Clerk views results | Invoice count and errors shown |
| AC-04 | S4 | All invoices reviewed | Finance manager posts all | Invoices posted to FI; events published |
| AC-05 | Role gate | BILLING_CLERK | Attempts Post All | Action disabled; FINANCE_MANAGER required |

---

## 9. Variability & Extension

### 9.1 Feature Dependencies
Requires IAM authentication. Requires F-SD-001 (order references on invoices). Required by F-SD-003-02.

### 9.2 Attributes
See §0.4 variability points. Binding time: `deploy`.

### 9.3 Extension Points
| Extension Zone | Interface | Default Behaviour |
|---|---|---|
| `ext.billingRunFilters` | Additional run scope filters | Hidden (no extension) |

### 9.4 Companion UVL
See `uvl/leaves/F-SD-003-01.uvl`.

---

**END OF SPECIFICATION**
