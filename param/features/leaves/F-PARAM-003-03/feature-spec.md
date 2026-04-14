# F-PARAM-003-03 — Configuration Audit Trail

> **Conceptual Stack Layer:** Platform-Feature
> **Space:** Platform
> **Owner:** Platform Engineering Team
> **Companion files:** `F-PARAM-003-03.uvl`, `F-PARAM-003-03.aui.yaml`

> **Meta Information**
> - **Version:** 2026-04-03
> - **Template:** `feature-spec.md` v1.0.0
> - **Template Compliance:** 100%
> - **Status:** DRAFT
> - **Feature ID:** `F-PARAM-003-03`
> - **Suite:** `param`
> - **Node type:** LEAF
> - **Parent:** `F-PARAM-003` — Configuration Management
> - **Companion UVL:** `F-PARAM-003-03.uvl`
> - **Companion AUI:** `F-PARAM-003-03.aui.yaml`

---

## ═══════════════════════════════════════════════
## PROBLEM SPACE
## ═══════════════════════════════════════════════

## 0. Feature Identity & Orientation

### 0.1 One-Line Summary
This feature lets a **platform administrator** view the change history of configuration entries so that compliance auditors can trace who changed what and when.

### 0.2 Non-Goals
- Does not duplicate functionality of sibling features in F-PARAM-003.
- See composition spec `F-PARAM-003.md` for boundary rationale.

### 0.3 Entry & Exit Points
**Entry points:**
- Platform Administration menu → linked from parent composition
- Direct URL or navigation from sibling feature

**Exit points:**
- Back to parent composition view or Platform Administration dashboard

### 0.4 Variability Points
| Variability Point | Model | Values | Default | Binding Time |
|---|---|---|---|---|
| Pagination page size | UVL attribute | 10, 25, 50, 100 | 25 | runtime |

---

## 1. User Goal & Scenarios

### 1.1 User Goal
This feature lets a **platform administrator** view the change history of configuration entries so that compliance auditors can trace who changed what and when.

### 1.2 Scenarios
| # | Scenario | Precondition | Action | Expected Outcome |
|---|----------|-------------|--------|-----------------|
| S1 | View audit trail | Config entry selected | Open audit tab | Chronological list of changes |
| S2 | Filter by date range | Audit list displayed | Set date range | Only changes in range shown |
| S3 | View change detail | Audit list displayed | Click entry | Previous/new value diff, reason, admin |

---

## 2. User Journey & Screen Layout

### 2.1 Sequence Diagram

```mermaid
sequenceDiagram
    actor Admin
    participant BFF
    participant SVC as param-cfg-svc

    Admin->>BFF: Open feature
    BFF->>SVC: GET /api/param/cfg/v1/configs/{scope}/{key}/audit
    SVC-->>BFF: 200 OK
    BFF-->>Admin: Render screen
```

### 2.2 Screen Layout
See companion AUI contract `F-PARAM-003-03.aui.yaml` for zone layout.

---

## 3. Interaction Requirements

### 3.1 Fields Table
| Field | Type | Required | Editable | Validation | i18n Key |
|---|---|---|---|---|---|
| Date From | date picker | No | Yes | — | `F-PARAM-003-03.filter.dateFrom` |
| Date To | date picker | No | Yes | must be after Date From | `F-PARAM-003-03.filter.dateTo` |
| Changed By | text input | No | Yes | — | `F-PARAM-003-03.filter.changedBy` |

### 3.2 Actions Table
| Action | Trigger | Precondition | Effect |
|---|---|---|---|
| Filter | Date/user change | — | Filter audit entries |
| View detail | Row click | — | Show previous/new value diff |

### 3.3 Validation Messages
| Field | Condition | Message |
|---|---|---|
| Required fields | Empty on submit | "{Label} is required." |
| API 422 | BR violated | Error message from backend |

---

## 4. Edge Cases & Screen States

### 4.1 Component States
| State | When | Behaviour |
|---|---|---|
| **Loading** | Awaiting response | Skeleton; controls disabled |
| **Empty** | No data matches | Message + CTA |
| **Error** | Service unavailable | Inline message + retry button |
| **Populated** | Data ready | Render normally |

### 4.2 Specific Edge Cases
| Case | Behaviour | Affected users |
|---|---|---|
| Insufficient role | Mutation actions absent from DOM | Non-admin roles |
| Concurrent edit (412) | Banner: "Updated by another user. Reload." | Concurrent editors |

### 4.3 Attribute-Driven Behaviour Changes
| Attribute | Non-default value | Observable change |
|---|---|---|
| `pagination.pageSize` | 10 | Shorter list, more pages |

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
| 1 | param-cfg-svc | `GET /api/param/cfg/v1/configs/{scope}/{key}/audit` | T1 | No | Show error + retry |

### 5.2 BFF View-Model Shape
See domain spec `param_cfg-spec.md` §6 for response contract.

### 5.3 Feature-Gating Rules
| Mode | Behaviour |
|---|---|
| Full | All interactions available |
| Read-only | Mutation actions hidden |
| Excluded | Menu item hidden; direct URL returns 404 |

### 5.5 i18n Keys
| Key | Default (en) |
|-----|-------------|
| `F-PARAM-003-03.title` | `Configuration Audit Trail` |
| `F-PARAM-003-03.column.changedAt` | `Changed At` |
| `F-PARAM-003-03.column.changedBy` | `Changed By` |
| `F-PARAM-003-03.column.reason` | `Reason` |
| `F-PARAM-003-03.empty` | `No audit entries found.` |

---

## 6. AUI Screen Contract
See companion file `F-PARAM-003-03.aui.yaml`.

---

## ═══════════════════════════════════════════════
## BRIDGE ARTIFACTS
## ═══════════════════════════════════════════════

## 7. Permissions & Accessibility

### 7.1 Permission Matrix
| Action | PLATFORM_ADMIN | {SUITE}_ADMIN | TENANT_ADMIN | ANY_AUTHENTICATED |
|---|---|---|---|---|
| Read | ✓ | ✓ | ✓ | ✓ |
| Write | ✓ | PLATFORM_ADMIN | — | — |

### 7.2 Accessibility
- All interactive elements MUST be keyboard-accessible.
- Forms MUST have proper ARIA labels.

---

## 8. Acceptance Criteria
| AC | Given | When | Then |
|----|-------|------|------|
| AC-01 | Config has changes | Admin opens audit | Chronological change list |
| AC-02 | Date range set | Admin filters | Only changes in range |
| AC-03 | Entry clicked | Admin views detail | Previous/new value diff shown |

---

## 9. Variability & Extension

### 9.1 Feature Dependencies
Requires IAM authentication.

### 9.2 Extension Points
| Extension Zone | Interface | Default Behaviour |
|---|---|---|
| `ext.customFields` | Additional fields | Hidden |

### 9.4 Companion UVL
See `uvl/leaves/F-PARAM-003-03.uvl`.

---
**END OF SPECIFICATION**
