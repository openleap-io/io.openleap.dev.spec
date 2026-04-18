# F-PARAM-002-01 — Browse Translations

> **Conceptual Stack Layer:** Platform-Feature
> **Space:** Platform
> **Owner:** Platform Engineering Team
> **Companion files:** `F-PARAM-002-01.uvl`, `F-PARAM-002-01.aui.yaml`
> **Referenced by:** Suite Feature Catalog SS6
> **References:** `param_i18n-spec.md` (backend)

> **Meta Information**
> - **Version:** 2026-04-03
> - **Template:** `feature-spec.md` v1.0.0
> - **Template Compliance:** 100%
> - **Status:** DRAFT
> - **Feature ID:** `F-PARAM-002-01`
> - **Suite:** `param`
> - **Node type:** LEAF
> - **Parent:** `F-PARAM-002` — Translation Management
> - **Companion UVL:** `F-PARAM-002-01.uvl`
> - **Companion AUI:** `F-PARAM-002-01.aui.yaml`

---

## ═══════════════════════════════════════════════
## PROBLEM SPACE
## ═══════════════════════════════════════════════

## 0. Feature Identity & Orientation

### 0.1 One-Line Summary
This feature lets a **suite administrator** browse and search translation namespaces and their keys across locales so that they can find and verify translations.

### 0.2 Non-Goals
- Does not duplicate functionality of sibling features in F-PARAM-002.
- See composition spec `F-PARAM-002.md` for boundary rationale.

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
This feature lets a **suite administrator** browse and search translation namespaces and their keys across locales so that they can find and verify translations.

### 1.2 Scenarios
| # | Scenario | Precondition | Action | Expected Outcome |
|---|----------|-------------|--------|-----------------|
| S1 | Browse namespaces | Translations exist | Open translations list | List of namespaces with key count, locales |
| S2 | Search by namespace | List displayed | Type namespace pattern | Filtered namespaces |
| S3 | View bundle | Namespace selected | Click namespace | All keys with locale columns shown |
| S4 | Filter by locale | Bundle displayed | Select locale filter | Only selected locale values shown |
| S5 | Missing translation | Bundle displayed | Key has no value for locale | Cell shows [missing] indicator |

---

## 2. User Journey & Screen Layout

### 2.1 Sequence Diagram

```mermaid
sequenceDiagram
    actor Admin
    participant BFF
    participant SVC as param-i18n-svc

    Admin->>BFF: Open feature
    BFF->>SVC: GET /api/param/i18n/v1/translations
    SVC-->>BFF: 200 OK
    BFF-->>Admin: Render screen
```

### 2.2 Screen Layout
See companion AUI contract `F-PARAM-002-01.aui.yaml` for zone layout.

---

## 3. Interaction Requirements

### 3.1 Fields Table
| Field | Type | Required | Editable | Validation | i18n Key |
|---|---|---|---|---|---|
| Namespace search | text input | No | Yes | min 2 chars | `F-PARAM-002-01.search.namespace` |
| Type filter | select | No | Yes | CATALOG, MESSAGE, All | `F-PARAM-002-01.filter.type` |
| Locale filter | multi-select | No | Yes | Available locales | `F-PARAM-002-01.filter.locale` |

### 3.2 Actions Table
| Action | Trigger | Precondition | Effect |
|---|---|---|---|
| Search | Keystroke debounced | ≥ 2 chars | Filter namespace list |
| Select namespace | Row click | — | Show translation bundle |
| Navigate to edit | Click key | Write role | Navigate to F-PARAM-002-02 |

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
| Insufficient role | Action absent from DOM | Non-admin roles |
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
| 1 | param-i18n-svc | `GET /api/param/i18n/v1/translations` | T1 | No | Show error + retry |

### 5.2 BFF View-Model Shape
See domain spec `param_i18n-spec.md` §6 for response contract.

### 5.3 Feature-Gating Rules
| Mode | Behaviour |
|---|---|
| Full | All interactions available |
| Read-only | Mutation actions hidden |
| Excluded | Menu item hidden; direct URL returns 404 |

### 5.4 Caching Hints
BFF SHOULD cache read responses. Cache MUST be invalidated on relevant domain events.

### 5.5 i18n Keys
| Key | Default (en) |
|-----|-------------|
| `F-PARAM-002-01.title` | `Translations` |
| `F-PARAM-002-01.search.namespace` | `Search namespaces…` |
| `F-PARAM-002-01.missing` | `[missing]` |
| `F-PARAM-002-01.empty` | `No translations found.` |

---

## 6. AUI Screen Contract
See companion file `F-PARAM-002-01.aui.yaml`.

---

## ═══════════════════════════════════════════════
## BRIDGE ARTIFACTS
## ═══════════════════════════════════════════════

## 7. Permissions & Accessibility

### 7.1 Permission Matrix
| Action | PLATFORM_ADMIN | {SUITE}_ADMIN | TENANT_ADMIN | ANY_AUTHENTICATED |
|---|---|---|---|---|
| Read | ✓ | ✓ | ✓ | ✓ |
| Write | ✓ | ✓ (own scope) | — | — |

### 7.2 Accessibility
- All interactive elements MUST be keyboard-accessible.
- Forms MUST have proper ARIA labels and error associations.

---

## 8. Acceptance Criteria
| AC | Given | When | Then |
|----|-------|------|------|
| AC-01 | Translations exist | Admin opens list | Namespaces shown with key count |
| AC-02 | Admin searches | Types namespace prefix | Filtered results |
| AC-03 | Namespace selected | Admin clicks row | Bundle shown with all keys × locales |
| AC-04 | Key has no locale value | Bundle displayed | [missing] indicator shown |

---

## 9. Variability & Extension

### 9.1 Feature Dependencies
Requires IAM authentication (cross-suite).

### 9.2 Attributes
See SS0.4. Binding times: `deploy`, `runtime`.

### 9.3 Extension Points
| Extension Zone | Interface | Default Behaviour |
|---|---|---|
| `ext.customFields` | Additional fields in form | Hidden |

### 9.4 Companion UVL
See `uvl/leaves/F-PARAM-002-01.uvl`.

---
**END OF SPECIFICATION**
