# F-SD-002-03 — Proof of Delivery

> **Conceptual Stack Layer:** Domain-Feature
> **Space:** Business
> **Owner:** SD Product Team
> **Companion files:** `F-SD-002-03.uvl`, `F-SD-002-03.aui.yaml`
> **Referenced by:** Suite Feature Catalog SS6
> **References:** `domain-specs/sd_dlv-spec.md` (backend)

> **Meta Information**
> - **Version:** 2026-04-04
> - **Template:** `feature-spec.md` v1.0.0
> - **Template Compliance:** 100%
> - **Status:** DRAFT
> - **Feature ID:** `F-SD-002-03`
> - **Suite:** `sd`
> - **Node type:** LEAF
> - **Parent:** `F-SD-002` — Shipping & Delivery
> - **Companion UVL:** `F-SD-002-03.uvl`
> - **Companion AUI:** `F-SD-002-03.aui.yaml`

---

## ═══════════════════════════════════════════════
## PROBLEM SPACE
## ═══════════════════════════════════════════════

## 0. Feature Identity & Orientation

### 0.1 One-Line Summary
This feature lets a **logistics coordinator** record and manage proof of delivery confirmations from customers or carriers.

### 0.2 Non-Goals
- Does not execute shipments — that is F-SD-002-02.
- Does not manage returns — that is sd.ret.
- Does not create credit notes for rejected deliveries — that is F-SD-003-03.

### 0.3 Entry & Exit Points

**Entry points:**
- Logistics menu → "Proof of Delivery"
- Direct URL: `/sd/dlv/pod`

**Exit points:**
- POD confirmed → delivery status updated to DELIVERED
- Rejection recorded → may trigger return flow in sd.ret

### 0.4 Variability Points

| Variability Point | Model | Values | Default | Binding Time |
|---|---|---|---|---|
| Signature capture required | UVL attribute `Boolean signature_required` | true / false | false | deploy |

---

## 1. User Goal & Scenarios

### 1.1 User Goal
Record confirmation that goods were received by the customer so that the delivery lifecycle is closed and billing finalization can proceed.

### 1.2 Scenarios

| # | Scenario | Precondition | Action | Expected Outcome |
|---|----------|-------------|--------|-----------------|
| S1 | Record POD | Shipment DISPATCHED | Enter confirmation details and click "Confirm" | Delivery status → DELIVERED; event published |
| S2 | Handle partial delivery | Not all items received by customer | Record partial quantities; flag remaining | Partial POD recorded; remainder flagged for follow-up |
| S3 | Handle rejection | Customer refuses delivery | Click "Record Rejection" with reason | Rejection recorded; delivery status → REJECTED; return flow optionally triggered |
| S4 | POD history | Delivery DELIVERED | Open POD history tab | All POD records for the delivery shown with timestamps |

---

## 2. User Journey & Screen Layout

### 2.1 Sequence Diagram

```mermaid
sequenceDiagram
    actor LC as Logistics Coordinator
    participant BFF
    participant DLV as sd-dlv-svc

    LC->>BFF: GET /sd/dlv/pod
    BFF->>DLV: GET /api/sd/dlv/v1/deliveries?status=DISPATCHED
    DLV-->>BFF: 200 { dispatched deliveries }
    BFF-->>LC: Render POD list

    LC->>BFF: Record POD for delivery
    BFF->>DLV: POST /api/sd/dlv/v1/proof-of-delivery
    DLV-->>BFF: 201 { podId, deliveryStatus: DELIVERED }
    BFF-->>LC: Success toast; delivery marked DELIVERED
```

### 2.2 Screen Layout

```
┌─────────────────────────────────────────────────────┐
│ Proof of Delivery                                    │
├─────────────────────────────────────────────────────┤
│ Deliveries Awaiting POD                              │
│ ┌──────────┬──────────────┬────────────┬──────────┐  │
│ │ Del. #   │ Customer     │ Dispatched │ Action   │  │
│ ├──────────┼──────────────┼────────────┼──────────┤  │
│ │ DEL-0021 │ Acme Corp    │ 2026-04-30 │ [Record] │  │
│ └──────────┴──────────────┴────────────┴──────────┘  │
├─────────────────────────────────────────────────────┤
│ Record POD — DEL-0021                                │
│ Received By: [________________]                      │
│ Date: [Date picker]                                  │
│ Notes: [________________]                            │
│ [EXT: extension zone]                                │
│ [Record Rejection]         [Confirm Delivery]        │
└─────────────────────────────────────────────────────┘
```

---

## 3. Interaction Requirements

### 3.1 Fields Table

| Field | Type | Required | Editable | Validation | i18n Key |
|---|---|---|---|---|---|
| Received By | text | Yes | Yes | Min 2 chars | `F-SD-002-03.field.receivedBy` |
| Receipt Date | date picker | Yes | Yes | Must be ≤ today | `F-SD-002-03.field.receiptDate` |
| Notes | text area | No | Yes | Max 500 chars | `F-SD-002-03.field.notes` |
| Rejection Reason | text area | Conditional | Yes | Required on rejection | `F-SD-002-03.field.rejectionReason` |

### 3.2 Actions Table

| Action | Trigger | Precondition | Effect |
|---|---|---|---|
| Confirm Delivery | Button click | receivedBy and receiptDate filled | POST POD; delivery → DELIVERED |
| Record Rejection | Button click | Rejection reason filled | POST rejection; delivery → REJECTED |
| View History | Tab click | Delivery has POD records | Load and display POD history |

### 3.3 Validation Messages

| Field | Condition | Message |
|---|---|---|
| Received By | Empty | "Please enter the name of the recipient." |
| Receipt Date | In the future | "Receipt date cannot be in the future." |
| Rejection Reason | Empty on rejection | "Please provide a reason for rejection." |

---

## 4. Edge Cases & Screen States

### 4.1 Component States

| State | When | Behaviour |
|---|---|---|
| **Loading** | Awaiting API | Skeleton rows |
| **Empty** | No deliveries awaiting POD | "No deliveries are currently awaiting proof of delivery." |
| **Error** | sd-dlv-svc unavailable | Error banner with retry |

### 4.2 Specific Edge Cases

| Case | Behaviour | Affected users |
|---|---|---|
| Partial delivery | Coordinator records received qty per line; remainder auto-flagged | LOGISTICS_COORDINATOR |
| signature_required = true | Signature capture widget shown; Confirm disabled without signature | LOGISTICS_COORDINATOR |

### 4.3 Attribute-Driven Behaviour Changes

| Attribute | Non-default value | Observable change |
|---|---|---|
| `signature_required` | true | Signature pad widget shown; Confirm disabled without captured signature |

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
| 1 | sd-dlv-svc | `GET /api/sd/dlv/v1/deliveries?status=DISPATCHED` | T3 | No | Show error + retry |
| 2 | sd-dlv-svc | `POST /api/sd/dlv/v1/proof-of-delivery` | T3 | Yes | Show error banner |

### 5.2 BFF View-Model Shape

```jsonc
{
  "awaitingPod": [
    { "deliveryId": "del-uuid", "deliveryNumber": "DEL-0021", "customerName": "Acme Corp", "dispatchedAt": "2026-04-30T08:00:00Z" }
  ]
}
```

### 5.3 Feature-Gating Rules

| Mode | Behaviour |
|---|---|
| Full | All POD actions available |
| Read-only | History visible; Confirm and Reject disabled |
| Excluded | Menu item hidden; URL returns 404 |

### 5.4 Failure Modes

| Failure | User Experience |
|---------|----------------|
| sd-dlv-svc down | Error banner; all actions disabled |

### 5.5 Caching Hints
BFF MUST NOT cache the POD form. Awaiting POD list MAY be cached for 60 seconds.

### 5.6 i18n Keys

| Key | Default (en) |
|-----|-------------|
| `F-SD-002-03.title` | `Proof of Delivery` |
| `F-SD-002-03.field.receivedBy` | `Received By` |
| `F-SD-002-03.field.receiptDate` | `Receipt Date` |
| `F-SD-002-03.field.notes` | `Notes` |
| `F-SD-002-03.field.rejectionReason` | `Rejection Reason` |
| `F-SD-002-03.action.confirm` | `Confirm Delivery` |
| `F-SD-002-03.action.reject` | `Record Rejection` |

---

## 6. AUI Screen Contract

See companion file `F-SD-002-03.aui.yaml`.

---

## ═══════════════════════════════════════════════
## BRIDGE ARTIFACTS
## ═══════════════════════════════════════════════

## 7. Permissions & Accessibility

### 7.1 Permission Matrix

| Action | LOGISTICS_COORDINATOR | CUSTOMER_SERVICE | SHIPPING_CLERK |
|---|---|---|---|
| View awaiting POD list | ✓ | ✓ | ✓ |
| Confirm delivery | ✓ | ✓ | — |
| Record rejection | ✓ | ✓ | — |
| View history | ✓ | ✓ | ✓ |

### 7.2 Accessibility
- Confirm and Reject buttons MUST have `aria-label` with delivery reference.
- Date picker MUST support keyboard entry.

---

## 8. Acceptance Criteria

| AC | Scenario | Given | When | Then |
|----|----------|-------|------|------|
| AC-01 | S1 | Shipment DISPATCHED | LC confirms POD | Delivery → DELIVERED; `sd.dlv.delivery.pod-recorded` published |
| AC-02 | S2 | Partial receipt | LC records partial qty | Partial POD recorded; remainder flagged |
| AC-03 | S3 | Customer rejects | LC records rejection | Delivery → REJECTED; return flow optionally triggered |
| AC-04 | S4 | Delivery DELIVERED | LC views history tab | All POD records shown with timestamps |
| AC-05 | signature_required | signature_required = true | LC attempts confirm without signature | Confirm button disabled; signature prompt shown |

---

## 9. Variability & Extension

### 9.1 Feature Dependencies
Requires IAM authentication. Requires F-SD-002-02 (Shipment Execution).

### 9.2 Attributes
See §0.4 variability points. Binding time: `deploy`.

### 9.3 Extension Points
| Extension Zone | Interface | Default Behaviour |
|---|---|---|
| `ext.podFields` | Additional POD fields (e.g., condition notes) | Hidden (no extension) |

### 9.4 Companion UVL
See `uvl/leaves/F-SD-002-03.uvl`.

---

**END OF SPECIFICATION**
