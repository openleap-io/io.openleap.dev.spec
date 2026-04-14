# F-PARAM-004-03 — Unit Conversion Tool

> **Conceptual Stack Layer:** Platform-Feature
> **Space:** Platform
> **Owner:** Platform Engineering Team
> **Companion files:** `F-PARAM-004-03.uvl`, `F-PARAM-004-03.aui.yaml`

> **Meta Information**
> - **Version:** 2026-04-03
> - **Template:** `feature-spec.md` v1.0.0
> - **Template Compliance:** 100%
> - **Status:** DRAFT
> - **Feature ID:** `F-PARAM-004-03`
> - **Suite:** `param`
> - **Node type:** LEAF
> - **Parent:** `F-PARAM-004` — Unit Management
> - **Companion UVL:** `F-PARAM-004-03.uvl`
> - **Companion AUI:** `F-PARAM-004-03.aui.yaml`

---

## ═══════════════════════════════════════════════
## PROBLEM SPACE
## ═══════════════════════════════════════════════

## 0. Feature Identity & Orientation

### 0.1 One-Line Summary
This feature lets a **domain engineer or support staff** perform interactive unit conversions with dimensional analysis validation so that they can verify conversion correctness and test custom units.

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
This feature lets a **domain engineer or support staff** perform interactive unit conversions with dimensional analysis validation so that they can verify conversion correctness and test custom units.

### 1.2 Scenarios
| # | Scenario | Precondition | Action | Expected Outcome |
|---|----------|-------------|--------|-----------------|
| S1 | Simple conversion | Units exist | Enter 100 km → m | Result: 100000 m |
| S2 | Temperature conversion | Affine units | Enter 25 °C → K | Result: 298.15 K |
| S3 | Incompatible units | Different dimensions | Enter 5 kg → m | Error: dimensional mismatch |
| S4 | Prefix formatting | Conversion done | Toggle prefix format | Result displayed as 100 km instead of 100000 m |

---

## 2. User Journey & Screen Layout

### 2.1 Sequence Diagram

```mermaid
sequenceDiagram
    actor Admin
    participant BFF
    participant SVC as param-si-svc

    Admin->>BFF: Open feature
    BFF->>SVC: POST /api/param/si/v1/conversions
    SVC-->>BFF: 200 OK
    BFF-->>Admin: Render screen
```

### 2.2 Screen Layout
See companion AUI contract `F-PARAM-004-03.aui.yaml` for zone layout.

---

## 3. Interaction Requirements

### 3.1 Fields Table
| Field | Type | Required | Editable | Validation | i18n Key |
|---|---|---|---|---|---|
| Value | number input | Yes | Yes | valid number | `F-PARAM-004-03.field.value` |
| From Unit | typeahead/select | Yes | Yes | existing unit | `F-PARAM-004-03.field.fromUnit` |
| To Unit | typeahead/select | Yes | Yes | existing unit | `F-PARAM-004-03.field.toUnit` |
| Use Prefix | toggle | No | Yes | — | `F-PARAM-004-03.field.usePrefix` |

### 3.2 Actions Table
| Action | Trigger | Precondition | Effect |
|---|---|---|---|
| Convert | Button click or Enter | All fields filled | Show conversion result |
| Swap | Swap button | Units selected | Swap from/to units |

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
| 1 | param-si-svc | `POST /api/param/si/v1/conversions` | T1 | No | Show error + retry |

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
| `F-PARAM-004-03.title` | `Unit Conversion Tool` |
| `F-PARAM-004-03.action.convert` | `Convert` |
| `F-PARAM-004-03.action.swap` | `Swap` |
| `F-PARAM-004-03.result` | `{value} {fromUnit} = {result} {toUnit}` |
| `F-PARAM-004-03.error.incompatible` | `Cannot convert: dimensional mismatch.` |

---

## 6. AUI Screen Contract
See companion file `F-PARAM-004-03.aui.yaml`.

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
| AC-01 | Compatible units | Admin converts 100 km → m | Result: 100000 m |
| AC-02 | Temperature | Admin converts 25 °C → K | Result: 298.15 K |
| AC-03 | Incompatible | Admin converts kg → m | Error: dimensional mismatch |
| AC-04 | Prefix toggle | Admin toggles | Result reformatted |

---

## 9. Variability & Extension

### 9.1 Feature Dependencies
Requires IAM authentication.

### 9.2 Extension Points
| Extension Zone | Interface | Default Behaviour |
|---|---|---|
| `ext.customFields` | Additional fields | Hidden |

### 9.4 Companion UVL
See `uvl/leaves/F-PARAM-004-03.uvl`.

---
**END OF SPECIFICATION**
