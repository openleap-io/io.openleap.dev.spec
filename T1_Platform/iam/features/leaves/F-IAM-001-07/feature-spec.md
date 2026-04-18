<!-- TEMPLATE COMPLIANCE: 100%
Template: feature-spec.md v1.0.0
-->

# F-IAM-001-07 — Credential Management

> **Conceptual Stack Layer:** Platform-Feature
> **Space:** Platform
> **Owner:** Domain Engineering Team
> **Companion files:** `F-IAM-001-07.uvl`, `F-IAM-001-07.aui.yaml`

> **Meta Information**
> - **Version:** 2026-04-03
> - **Template:** `feature-spec.md` v1.0.0
> - **Template Compliance:** 100%
> - **Author(s):** OpenLeap Architecture Team
> - **Status:** DRAFT
> - **Feature ID:** `F-IAM-001-07`
> - **Suite:** `iam`
> - **Node type:** LEAF
> - **Companion UVL:** `uvl/leaves/F-IAM-001-07.uvl`
> - **Companion AUI:** `contracts/aui/F-IAM-001-07.aui.yaml`

---

## PROBLEM SPACE

## 0. Feature Identity & Orientation

### 0.1 One-Line Summary
This feature lets a IAM_SYSTEM_ADMIN rotate, regenerate, or revoke credentials for service and system principals.

### 0.2 Non-Goals
This feature does not cover capabilities assigned to other leaf features in the same composition node.

### 0.3 Entry & Exit Points
**Entry:** IAM admin navigation menu, direct API call, principal detail page.
**Exit:** Success → toast → detail/list view. Failure → inline errors → stay on form.

### 0.4 Variability Points
| Variability | Modelled as | Default | Binding |
|---|---|---|---|
| Records per page | Attribute `pagination.pageSize` | 25 | `deploy` |

---

## 1. User Goal & Scenarios

### 1.1 The User Goal
Rotate, regenerate, or revoke credentials for service and system principals.

### 1.2 User Scenarios
**Scenario 1: Rotate API key**
> Old valid 24h, new returned once

**Scenario 2: Emergency revocation**
> Zero grace period


---

## 2. User Journey & Screen Layout

### 2.1 Happy-Path Flow
```mermaid
sequenceDiagram
    actor U as User
    participant UI as Feature UI
    participant BFF as BFF
    participant SVC as iam-principal-svc
    U->>UI: Navigate to feature
    UI->>BFF: Load reference data
    BFF-->>UI: View model
    U->>UI: Fill fields & submit
    UI->>BFF: Submit
    BFF->>SVC: API call
    alt Success
        SVC-->>BFF: 201/200
        BFF-->>UI: OK
        UI-->>U: Toast + navigate
    else Validation error
        SVC-->>BFF: 422
        UI-->>U: Inline errors
    else Unavailable
        SVC-->>BFF: 503
        UI-->>U: Error banner + retry
    end
```

### 2.2 Screen Layout
```
┌──────────────────────────────────────────────────┐
│  ← Back            Credential Management     [Cancel]    │
├──────────────────────────────────────────────────┤
│  SECTION: Core Details                            │
│  (Fields from §3.1)                               │
├──────────────────────────────────────────────────┤
│  SECTION: Extension Zone [EXT]                    │
├──────────────────────────────────────────────────┤
│                            [Cancel]  [Submit]     │
└──────────────────────────────────────────────────┘
```

---

## 3. Interaction Requirements

### 3.1 Form Fields
| Field | Service | Control | Req | Validation |
|---|---|---|:---:|---|
| Credential Type * | `iam-principal-svc` | Select | Y | API_KEY/CLIENT_SECRET/CERTIFICATE |
| Expiry | `iam-principal-svc` | Date picker |  | Default 90 days |
| Revoke Previous | `iam-principal-svc` | Checkbox |  | Default true, 24h grace |

### 3.2 Actions
| Action | Trigger | Precondition | Confirm | Role | Outcome |
|---|---|---|:---:|---|---|
| Submit | Button | Form valid | — | `IAM_SYSTEM_ADMIN` | Navigate + toast |
| Cancel | Button | — | If dirty | Any | Navigate back |

### 3.3 Unsaved Changes Guard
If dirty → dialog "You have unsaved changes. Leave and discard them?" → Leave / Stay.

---

## 4. Edge Cases & Screen States
| State | When | Behaviour |
|---|---|---|
| Loading | Awaiting response | Skeleton; controls disabled |
| Empty | No ref data | Error + retry |
| Error | Call failed | Inline message + retry |
| Populated | Data ready | Render normally |

---

## SOLUTION SPACE

## 5. Backend Dependencies & BFF Contract

### 5.1 Service Calls
| # | Service | Tier | Method | isMutation | Failure |
|---|---------|------|--------|:---:|---|
| 1 | iam-principal-svc | T1 | GET ref data | N | Retry 3x |
| 2 | iam-principal-svc | T1 | POST/PUT/PATCH | Y | Show error |

### 5.2 Feature-Gating
| Mode | Behaviour |
|---|---|
| full | All fields editable, submit enabled |
| read-only | All disabled, submit hidden |
| excluded | Entry point hidden |

---

## 6. Screen Contract (AUI)
See companion file: `contracts/aui/F-IAM-001-07.aui.yaml`

---

## 7. i18n, Permissions & Accessibility
| Action | Show when | Enable when | Role |
|---|---|---|---|
| Entry point | Always | Any role | Any |
| Submit | Always | Form valid | `IAM_SYSTEM_ADMIN` |

---

## 8. Acceptance Criteria
**AC-001:** Given role `IAM_SYSTEM_ADMIN`, when all required fields filled and submitted, then record created and toast shown.
**AC-002:** Given required field blank, when submit, then inline error on field with focus.
**AC-003:** Given role `IAM_VIEWER`, then submit action absent from DOM.

---

## 9. Dependencies, Variability & Extension Points
| Attribute | Type | Default | Binding |
|---|---|---|---|
| `pagination.pageSize` | Integer | 25 | `deploy` |

---

## 10. Change Log
| Date | Version | Author | Changes |
|------|---------|--------|---------|
| 2026-04-03 | 1.0 | Architecture Team | Initial specification |

---
**END OF SPECIFICATION**
