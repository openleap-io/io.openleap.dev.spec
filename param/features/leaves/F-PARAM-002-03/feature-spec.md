# F-PARAM-002-03 — Seed Translation Bundles

> **Conceptual Stack Layer:** Platform-Feature
> **Space:** Platform
> **Owner:** Platform Engineering Team
> **Companion files:** `F-PARAM-002-03.uvl`, `F-PARAM-002-03.aui.yaml`

> **Meta Information**
> - **Version:** 2026-04-03
> - **Template:** `feature-spec.md` v1.0.0
> - **Template Compliance:** 100%
> - **Status:** DRAFT
> - **Feature ID:** `F-PARAM-002-03`
> - **Suite:** `param`
> - **Node type:** LEAF
> - **Parent:** `F-PARAM-002` — Translation Management
> - **Companion UVL:** `F-PARAM-002-03.uvl`
> - **Companion AUI:** `F-PARAM-002-03.aui.yaml`

---

## ═══════════════════════════════════════════════
## PROBLEM SPACE
## ═══════════════════════════════════════════════

## 0. Feature Identity & Orientation

### 0.1 One-Line Summary
This feature lets a **suite administrator** bulk-seed translation bundles from JSON payloads so that initial translations for new features or locales can be loaded efficiently.

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
This feature lets a **suite administrator** bulk-seed translation bundles from JSON payloads so that initial translations for new features or locales can be loaded efficiently.

### 1.2 Scenarios
| # | Scenario | Precondition | Action | Expected Outcome |
|---|----------|-------------|--------|-----------------|
| S1 | Seed bundle | Valid JSON payload | Upload and confirm | Translations seeded; summary shown |
| S2 | Validation errors | Missing default locale | Upload | 422 with error details |
| S3 | Partial update | Some keys exist | Upload with overwrite=true | Existing keys updated, new keys created |

---

## 2. User Journey & Screen Layout

### 2.1 Sequence Diagram

```mermaid
sequenceDiagram
    actor Admin
    participant BFF
    participant SVC as param-i18n-svc

    Admin->>BFF: Open feature
    BFF->>SVC: POST /api/param/i18n/v1/translations/seed
    SVC-->>BFF: 200 OK
    BFF-->>Admin: Render screen
```

### 2.2 Screen Layout
See companion AUI contract `F-PARAM-002-03.aui.yaml` for zone layout.

---

## 3. Interaction Requirements

### 3.1 Fields Table
| Field | Type | Required | Editable | Validation | i18n Key |
|---|---|---|---|---|---|
| Namespace | text input | Yes | Yes | valid namespace pattern | `F-PARAM-002-03.field.namespace` |
| Payload | JSON editor | Yes | Yes | valid seed schema | `F-PARAM-002-03.field.payload` |
| Overwrite | checkbox | No | Yes | — | `F-PARAM-002-03.field.overwrite` |

### 3.2 Actions Table
| Action | Trigger | Precondition | Effect |
|---|---|---|---|
| Seed | Form submit | Valid payload + ownership | Bulk create translations |
| Validate | Pre-submit | — | Dry-run validation, show errors |

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
| 1 | param-i18n-svc | `POST /api/param/i18n/v1/translations/seed` | T1 | Yes | Show error + retry |

### 5.2 BFF View-Model Shape
See domain spec `param_i18n-spec.md` §6 for response contract.

### 5.3 Feature-Gating Rules
| Mode | Behaviour |
|---|---|
| Full | All interactions available |
| Read-only | Mutation actions hidden |
| Excluded | Menu item hidden; direct URL returns 404 |

### 5.5 i18n Keys
| Key | Default (en) |
|-----|-------------|
| `F-PARAM-002-03.title` | `Seed Translation Bundle` |
| `F-PARAM-002-03.action.seed` | `Seed` |
| `F-PARAM-002-03.action.validate` | `Validate` |
| `F-PARAM-002-03.seed.success` | `{count} translations seeded.` |

---

## 6. AUI Screen Contract
See companion file `F-PARAM-002-03.aui.yaml`.

---

## ═══════════════════════════════════════════════
## BRIDGE ARTIFACTS
## ═══════════════════════════════════════════════

## 7. Permissions & Accessibility

### 7.1 Permission Matrix
| Action | PLATFORM_ADMIN | {SUITE}_ADMIN | TENANT_ADMIN | ANY_AUTHENTICATED |
|---|---|---|---|---|
| Read | ✓ | ✓ | ✓ | ✓ |
| Write | ✓ | {SUITE}_ADMIN | — | — |

### 7.2 Accessibility
- All interactive elements MUST be keyboard-accessible.
- Forms MUST have proper ARIA labels.

---

## 8. Acceptance Criteria
| AC | Given | When | Then |
|----|-------|------|------|
| AC-01 | Valid payload | Admin seeds | Translations created |
| AC-02 | Missing default | Admin seeds | 422 error |
| AC-03 | Overwrite enabled | Admin seeds existing keys | Keys updated |

---

## 9. Variability & Extension

### 9.1 Feature Dependencies
Requires IAM authentication.

### 9.2 Extension Points
| Extension Zone | Interface | Default Behaviour |
|---|---|---|
| `ext.customFields` | Additional fields | Hidden |

### 9.4 Companion UVL
See `uvl/leaves/F-PARAM-002-03.uvl`.

---
**END OF SPECIFICATION**
