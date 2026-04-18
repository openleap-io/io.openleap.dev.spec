# F-PARAM-004-01 — Browse Units & Prefixes

> **Conceptual Stack Layer:** Platform-Feature
> **Space:** Platform
> **Owner:** Platform Engineering Team
> **Companion files:** `F-PARAM-004-01.uvl`, `F-PARAM-004-01.aui.yaml`

> **Meta Information**
> - **Version:** 2026-04-03
> - **Template:** `feature-spec.md` v1.0.0
> - **Template Compliance:** 100%
> - **Status:** DRAFT
> - **Feature ID:** `F-PARAM-004-01`
> - **Suite:** `param`
> - **Node type:** LEAF
> - **Parent:** `F-PARAM-004` — Unit Management
> - **Companion UVL:** `F-PARAM-004-01.uvl`
> - **Companion AUI:** `F-PARAM-004-01.aui.yaml`

---

## ═══════════════════════════════════════════════
## PROBLEM SPACE
## ═══════════════════════════════════════════════

## 0. Feature Identity & Orientation

### 0.1 One-Line Summary
This feature lets a **platform administrator or domain engineer** browse the SI unit catalog including base units, derived units, custom units, and SI prefixes so that they can understand available units.

### 0.2 Non-Goals
- Does not duplicate functionality of sibling features in F-PARAM-004.
- See composition spec `F-PARAM-004.md` for boundary rationale.

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
This feature lets a **platform administrator or domain engineer** browse the SI unit catalog including base units, derived units, custom units, and SI prefixes so that they can understand available units.

### 1.2 Scenarios
| # | Scenario | Precondition | Action | Expected Outcome |
|---|----------|-------------|--------|-----------------|
| S1 | Browse units | Units seeded | Open unit list | All units with name, symbol, dimension, base/custom flag |
| S2 | Search by name | List displayed | Type search | Filtered units |
| S3 | Filter base only | List displayed | Toggle base-only | Only 7 SI base units shown |
| S4 | View prefixes | Tab available | Click Prefixes tab | SI prefix table with name, symbol, power |
| S5 | View unit detail | Unit selected | Click unit row | Dimensional signature, conversion factor, definition |

---

## 2. User Journey & Screen Layout

### 2.1 Sequence Diagram

```mermaid
sequenceDiagram
    actor Admin
    participant BFF
    participant SVC as param-si-svc

    Admin->>BFF: Open feature
    BFF->>SVC: GET /api/param/si/v1/units
    SVC-->>BFF: 200 OK
    BFF-->>Admin: Render screen
```

### 2.2 Screen Layout
See companion AUI contract `F-PARAM-004-01.aui.yaml` for zone layout.

---

## 3. Interaction Requirements

### 3.1 Fields Table
| Field | Type | Required | Editable | Validation | i18n Key |
|---|---|---|---|---|---|
| Search | text input | No | Yes | min 2 chars | `F-PARAM-004-01.search` |
| Base only | toggle | No | Yes | — | `F-PARAM-004-01.filter.baseOnly` |
| Tab | tab selector | No | Yes | Units, Prefixes | `F-PARAM-004-01.tab` |

### 3.2 Actions Table
| Action | Trigger | Precondition | Effect |
|---|---|---|---|
| Search | Keystroke debounced | ≥ 2 chars | Filter unit list |
| Toggle base-only | Toggle change | — | Filter list |
| Switch tab | Tab click | — | Show units or prefixes |
| Select unit | Row click | — | Show unit detail panel |

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
| 1 | param-si-svc | `GET /api/param/si/v1/units` | T1 | No | Show error + retry |

### 5.2 BFF View-Model Shape
See domain spec `param_si-spec.md` §6 for response contract.

### 5.3 Feature-Gating Rules
| Mode | Behaviour |
|---|---|
| Full | All interactions available |
| Read-only | Mutation actions hidden |
| Excluded | Menu item hidden; direct URL returns 404 |

### 5.5 i18n Keys
| Key | Default (en) |
|-----|-------------|
| `F-PARAM-004-01.title` | `Units of Measure` |
| `F-PARAM-004-01.tab.units` | `Units` |
| `F-PARAM-004-01.tab.prefixes` | `Prefixes` |
| `F-PARAM-004-01.empty` | `No units found.` |

---

## 6. AUI Screen Contract
See companion file `F-PARAM-004-01.aui.yaml`.

---

## ═══════════════════════════════════════════════
## BRIDGE ARTIFACTS
## ═══════════════════════════════════════════════

## 7. Permissions & Accessibility

### 7.1 Permission Matrix
| Action | PLATFORM_ADMIN | {SUITE}_ADMIN | TENANT_ADMIN | ANY_AUTHENTICATED |
|---|---|---|---|---|
| Read | ✓ | ✓ | ✓ | ✓ |
| Write | ✓ | ANY_AUTHENTICATED | — | — |

### 7.2 Accessibility
- All interactive elements MUST be keyboard-accessible.
- Forms MUST have proper ARIA labels.

---

## 8. Acceptance Criteria
| AC | Given | When | Then |
|----|-------|------|------|
| AC-01 | Units seeded | Admin opens list | Units shown with symbol, dimension |
| AC-02 | Admin searches | Types name | Filtered results |
| AC-03 | Base-only toggled | Toggle on | Only 7 base units |
| AC-04 | Prefixes tab | Admin clicks | Prefix table shown |
| AC-05 | Unit selected | Admin clicks row | Detail panel with conversion factor |

---

## 9. Variability & Extension

### 9.1 Feature Dependencies
Requires IAM authentication.

### 9.2 Extension Points
| Extension Zone | Interface | Default Behaviour |
|---|---|---|
| `ext.customFields` | Additional fields | Hidden |

### 9.4 Companion UVL
See `uvl/leaves/F-PARAM-004-01.uvl`.

---
**END OF SPECIFICATION**
