# F-SD-002-02 — Shipment Execution

> **Conceptual Stack Layer:** Domain-Feature
> **Space:** Business
> **Owner:** SD Product Team
> **Companion files:** `F-SD-002-02.uvl`, `F-SD-002-02.aui.yaml`
> **Referenced by:** Suite Feature Catalog SS6
> **References:** `domain-specs/sd_shp-spec.md` (backend)

> **Meta Information**
> - **Version:** 2026-04-04
> - **Template:** `feature-spec.md` v1.0.0
> - **Template Compliance:** 100%
> - **Status:** DRAFT
> - **Feature ID:** `F-SD-002-02`
> - **Suite:** `sd`
> - **Node type:** LEAF
> - **Parent:** `F-SD-002` — Shipping & Delivery
> - **Companion UVL:** `F-SD-002-02.uvl`
> - **Companion AUI:** `F-SD-002-02.aui.yaml`

---

## ═══════════════════════════════════════════════
## PROBLEM SPACE
## ═══════════════════════════════════════════════

## 0. Feature Identity & Orientation

### 0.1 One-Line Summary
This feature lets a **shipping clerk** execute shipments: assign carriers, print shipping labels, post goods issue.

### 0.2 Non-Goals
- Does not plan deliveries — that is F-SD-002-01.
- Does not capture proof of delivery — that is F-SD-002-03.
- Does not trigger billing directly — goods issue event triggers billing eligibility in F-SD-003-01.

### 0.3 Entry & Exit Points

**Entry points:**
- Logistics menu → "Shipments"
- Direct URL: `/sd/shp/shipments`

**Exit points:**
- Goods issue posted → shipment status DISPATCHED; billing run eligible
- Navigate to F-SD-002-03 (Proof of Delivery) for tracking

### 0.4 Variability Points

| Variability Point | Model | Values | Default | Binding Time |
|---|---|---|---|---|
| Label print format | UVL attribute `String label_format` | ZPL, PDF, PNG | PDF | deploy |
| Carrier integration enabled | UVL attribute `Boolean carrier_integration_enabled` | true / false | false | deploy |

---

## 1. User Goal & Scenarios

### 1.1 User Goal
Execute outbound shipments by assigning a carrier, printing labels, and posting goods issue so that inventory is relieved and the customer's delivery is triggered.

### 1.2 Scenarios

| # | Scenario | Precondition | Action | Expected Outcome |
|---|----------|-------------|--------|-----------------|
| S1 | Create shipment | Delivery is PICKED | Click "Create Shipment" on delivery | Shipment document created in READY status |
| S2 | Assign carrier | Shipment in READY status | Select carrier from dropdown | Carrier and service level recorded on shipment |
| S3 | Print labels | Carrier assigned | Click "Print Labels" | Shipping labels generated in configured format |
| S4 | Post goods issue | Shipment in READY with carrier | Click "Post Goods Issue" | Inventory relieved; shipment status → DISPATCHED; `sd.shp.shipment.dispatched` published |
| S5 | Track shipment | Shipment DISPATCHED | View tracking section | Carrier tracking number displayed; status updates shown |

---

## 2. User Journey & Screen Layout

### 2.1 Sequence Diagram

```mermaid
sequenceDiagram
    actor Clerk as Shipping Clerk
    participant BFF
    participant SHP as sd-shp-svc

    Clerk->>BFF: GET /sd/shp/shipments
    BFF->>SHP: GET /api/sd/shp/v1/shipments
    SHP-->>BFF: 200 { shipments[] }
    BFF-->>Clerk: Render shipment list

    Clerk->>BFF: POST create shipment from delivery
    BFF->>SHP: POST /api/sd/shp/v1/shipments
    SHP-->>BFF: 201 { shipmentId, status: READY }
    BFF-->>Clerk: Navigate to shipment detail

    Clerk->>BFF: Assign carrier
    BFF->>SHP: PATCH /api/sd/shp/v1/shipments/{id}
    SHP-->>BFF: 200 { updated }

    Clerk->>BFF: Post goods issue
    BFF->>SHP: POST /api/sd/shp/v1/shipments/{id}/post-goods-issue
    SHP-->>BFF: 202 { status: DISPATCHED }
    BFF-->>Clerk: Success toast; shipment shows DISPATCHED
```

### 2.2 Screen Layout

```
┌─────────────────────────────────────────────────────┐
│ Shipments                             [+ New]        │
├─────────────────────────────────────────────────────┤
│ ┌──────────┬──────────────┬────────────┬──────────┐  │
│ │ Ship. #  │ Customer     │ Status     │ Carrier  │  │
│ ├──────────┼──────────────┼────────────┼──────────┤  │
│ │ SHP-0010 │ Acme Corp    │ READY      │ DHL      │ [Post GI] │
│ │ SHP-0009 │ Globex Inc   │ DISPATCHED │ UPS      │ [Track]   │
│ └──────────┴──────────────┴────────────┴──────────┘  │
├─────────────────────────────────────────────────────┤
│ [EXT: extension zone]                                │
└─────────────────────────────────────────────────────┘
```

---

## 3. Interaction Requirements

### 3.1 Fields Table

| Field | Type | Required | Editable | Validation | i18n Key |
|---|---|---|---|---|---|
| Carrier | select | Yes | Yes | Must be active carrier | `F-SD-002-02.field.carrier` |
| Service Level | select | Yes | Yes | Express, Standard, Economy | `F-SD-002-02.field.serviceLevel` |
| Tracking Number | text | No | Yes (if no integration) | Free text | `F-SD-002-02.field.trackingNumber` |

### 3.2 Actions Table

| Action | Trigger | Precondition | Effect |
|---|---|---|---|
| Create Shipment | Button click | Delivery in PICKED status | POST shipment |
| Assign Carrier | Select change | Shipment in READY | PATCH carrier |
| Print Labels | Button click | Carrier assigned | Generate labels in configured format |
| Post Goods Issue | Button click | Shipment READY; carrier assigned | POST goods issue; async processing |
| Track | Button click | Shipment DISPATCHED | Open tracking view |

### 3.3 Validation Messages

| Field | Condition | Message |
|---|---|---|
| Carrier | Not selected before Post GI | "Please assign a carrier before posting goods issue." |

---

## 4. Edge Cases & Screen States

### 4.1 Component States

| State | When | Behaviour |
|---|---|---|
| **Loading** | Awaiting API | Skeleton rows |
| **READY** | Shipment created, not dispatched | Post GI button enabled |
| **DISPATCHED** | GI posted | Post GI button hidden; Track button shown |
| **Error** | sd-shp-svc unavailable | Error banner with retry |

### 4.2 Specific Edge Cases

| Case | Behaviour | Affected users |
|---|---|---|
| GI post is async (202) | Spinner shown; status polled via event | SHIPPING_CLERK |
| Label print fails | Error toast; retry button in label panel | SHIPPING_CLERK |

### 4.3 Attribute-Driven Behaviour Changes

| Attribute | Non-default value | Observable change |
|---|---|---|
| `label_format` | ZPL | Labels sent directly to label printer; no PDF preview |
| `carrier_integration_enabled` | true | Carrier auto-assigned from routing rules; tracking number filled automatically |

### 4.4 Connectivity
This feature requires a live connection. Goods issue posting MUST NOT be retried automatically on network loss.

---

## ═══════════════════════════════════════════════
## SOLUTION SPACE
## ═══════════════════════════════════════════════

## 5. Backend Dependencies & BFF Contract

### 5.1 Service Calls

| # | Service | Endpoint | Tier | isMutation | Failure Mode |
|---|---------|----------|------|------------|-------------|
| 1 | sd-shp-svc | `GET /api/sd/shp/v1/shipments` | T3 | No | Show error + retry |
| 2 | sd-shp-svc | `POST /api/sd/shp/v1/shipments` | T3 | Yes | Show error banner |
| 3 | sd-shp-svc | `PATCH /api/sd/shp/v1/shipments/{id}` | T3 | Yes | Show error banner |
| 4 | sd-shp-svc | `POST /api/sd/shp/v1/shipments/{id}/post-goods-issue` | T3 | Yes | Show error banner |

### 5.2 BFF View-Model Shape

```jsonc
{
  "shipments": [
    {
      "shipmentId": "shp-uuid",
      "shipmentNumber": "SHP-0010",
      "deliveryId": "del-uuid",
      "customerName": "Acme Corp",
      "status": "READY",
      "carrier": "DHL",
      "serviceLevel": "STANDARD",
      "trackingNumber": null
    }
  ],
  "_meta": { "allowedActions": ["postGoodsIssue", "printLabels"] }
}
```

### 5.3 Feature-Gating Rules

| Mode | Behaviour |
|---|---|
| Full | All shipment actions available |
| Read-only | Lists shown; Post GI and Print Labels disabled |
| Excluded | Menu item hidden; URL returns 404 |

### 5.4 Failure Modes

| Failure | User Experience |
|---------|----------------|
| sd-shp-svc down | Error banner; all actions disabled |
| GI async failure | Event-driven notification in notification center |

### 5.5 Caching Hints
BFF MUST NOT cache shipment state after Post GI action. Shipment list MAY be cached for 30 seconds.

### 5.6 i18n Keys

| Key | Default (en) |
|-----|-------------|
| `F-SD-002-02.title` | `Shipments` |
| `F-SD-002-02.field.carrier` | `Carrier` |
| `F-SD-002-02.field.serviceLevel` | `Service Level` |
| `F-SD-002-02.field.trackingNumber` | `Tracking Number` |
| `F-SD-002-02.action.postGI` | `Post Goods Issue` |
| `F-SD-002-02.action.printLabels` | `Print Labels` |
| `F-SD-002-02.action.track` | `Track Shipment` |

---

## 6. AUI Screen Contract

See companion file `F-SD-002-02.aui.yaml`.

---

## ═══════════════════════════════════════════════
## BRIDGE ARTIFACTS
## ═══════════════════════════════════════════════

## 7. Permissions & Accessibility

### 7.1 Permission Matrix

| Action | SHIPPING_CLERK | LOGISTICS_COORDINATOR | WAREHOUSE_MANAGER |
|---|---|---|---|
| View shipments | ✓ | ✓ | ✓ |
| Create shipment | ✓ | ✓ | — |
| Assign carrier | ✓ | ✓ | — |
| Print labels | ✓ | ✓ | — |
| Post goods issue | ✓ | ✓ | — |

### 7.2 Accessibility
- Post GI button MUST have `aria-label="Post Goods Issue for shipment {number}"`.
- DISPATCHED status badge MUST use `aria-label` not just colour.

---

## 8. Acceptance Criteria

| AC | Scenario | Given | When | Then |
|----|----------|-------|------|------|
| AC-01 | S1 | Delivery PICKED | Clerk creates shipment | SHP-xxxx created, status READY |
| AC-02 | S2 | Shipment READY | Clerk assigns carrier DHL | Carrier stored; labels enabled |
| AC-03 | S3 | Carrier assigned | Clerk clicks Print Labels | Labels generated in configured format |
| AC-04 | S4 | Carrier assigned | Clerk posts goods issue | Status → DISPATCHED; event published; inventory relieved |
| AC-05 | S5 | Shipment DISPATCHED | Clerk views tracking | Tracking number and status displayed |

---

## 9. Variability & Extension

### 9.1 Feature Dependencies
Requires IAM authentication. Requires F-SD-002-01 (Delivery Planning). Required by F-SD-002-03.

### 9.2 Attributes
See §0.4 variability points. Binding time: `deploy`.

### 9.3 Extension Points
| Extension Zone | Interface | Default Behaviour |
|---|---|---|
| `ext.carrierIntegration` | Carrier API adapter hook | Uses manual carrier entry (no extension) |

### 9.4 Companion UVL
See `uvl/leaves/F-SD-002-02.uvl`.

---

**END OF SPECIFICATION**
