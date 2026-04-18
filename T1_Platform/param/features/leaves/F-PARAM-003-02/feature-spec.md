# F-PARAM-003-02 — Edit Configuration Entry

> **Conceptual Stack Layer:** Platform-Feature
> **Space:** Platform
> **Owner:** Platform Engineering Team
> **Companion files:** `F-PARAM-003-02.uvl`, `F-PARAM-003-02.aui.yaml`

> **Meta Information**
> - **Version:** 2026-04-03
> - **Template:** `feature-spec.md` v1.0.0
> - **Template Compliance:** 100%
> - **Status:** DRAFT
> - **Feature ID:** `F-PARAM-003-02`
> - **Suite:** `param`
> - **Node type:** LEAF
> - **Parent:** `F-PARAM-003` — Configuration Management
> - **Companion UVL:** `F-PARAM-003-02.uvl`
> - **Companion AUI:** `F-PARAM-003-02.aui.yaml`

---

## ═══════════════════════════════════════════════
## PROBLEM SPACE
## ═══════════════════════════════════════════════

## 0. Feature Identity & Orientation

### 0.1 One-Line Summary
This feature lets a **platform or tenant administrator** create or update a runtime configuration entry with mandatory audit trail so that system behaviour can be changed without redeployment.

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
This feature lets a **platform or tenant administrator** create or update a runtime configuration entry with mandatory audit trail so that system behaviour can be changed without redeployment.

### 1.2 Scenarios
| # | Scenario | Precondition | Action | Expected Outcome |
|---|----------|-------------|--------|-----------------|
| S1 | Edit global config | Entry exists | Change value, enter reason, save | Entry updated; audit log created; event published |
| S2 | Create tenant config | Admin is tenant admin | Fill form, submit | Tenant-scoped entry created |
| S3 | Edit well-known key | Key is well-known | Edit value | Value changes; key and scope are protected |
| S4 | Invalid value type | Value doesn't match configType | Submit | 422 validation error |

---

## 2. User Journey & Screen Layout

### 2.1 Sequence Diagram

```mermaid
sequenceDiagram
    actor Admin
    participant BFF
    participant SVC as param-cfg-svc

    Admin->>BFF: Open feature
    BFF->>SVC: PUT /api/param/cfg/v1/configs/{scope}/{key}
    SVC-->>BFF: 200 OK
    BFF-->>Admin: Render screen
```

### 2.2 Screen Layout
See companion AUI contract `F-PARAM-003-02.aui.yaml` for zone layout.

---

## 3. Interaction Requirements

### 3.1 Fields Table
| Field | Type | Required | Editable | Validation | i18n Key |
|---|---|---|---|---|---|
| Scope | select | Yes | No (after create) | GLOBAL, TENANT | `F-PARAM-003-02.field.scope` |
| Key | text input | Yes | No (after create) | max 200, naming convention | `F-PARAM-003-02.field.key` |
| Config Type | select | Yes | Yes | RUN_FLAG, FEATURE_FLAG, PARAMETER | `F-PARAM-003-02.field.configType` |
| Value | text input | Yes | Yes | max 2000, type-dependent | `F-PARAM-003-02.field.value` |
| Description | textarea | No | Yes | max 500 | `F-PARAM-003-02.field.description` |
| Change Reason | text input | Yes | Yes | max 500, not blank | `F-PARAM-003-02.field.changeReason` |

### 3.2 Actions Table
| Action | Trigger | Precondition | Effect |
|---|---|---|---|
| Save | Form submit | Form valid + role | Create/update config; audit log; event |
| Cancel | Button click | — | Discard, return to browse |

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
| 1 | param-cfg-svc | `PUT /api/param/cfg/v1/configs/{scope}/{key}` | T1 | Yes | Show error + retry |

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
| `F-PARAM-003-02.title` | `Edit Configuration` |
| `F-PARAM-003-02.action.save` | `Save` |
| `F-PARAM-003-02.field.changeReason` | `Reason for change` |
| `F-PARAM-003-02.error.wellKnown` | `This is a protected key. Scope and key cannot be changed.` |

---

## 6. AUI Screen Contract
See companion file `F-PARAM-003-02.aui.yaml`.

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
| AC-01 | Config exists | Admin edits value + reason, saves | Updated; audit log created; event published |
| AC-02 | New tenant config | Admin fills form, submits | Tenant-scoped entry created |
| AC-03 | Well-known key | Admin edits | Scope and key fields are read-only |
| AC-04 | Change reason blank | Admin submits | 422: change reason required |

---

## 9. Variability & Extension

### 9.1 Feature Dependencies
Requires IAM authentication.

### 9.2 Extension Points
| Extension Zone | Interface | Default Behaviour |
|---|---|---|
| `ext.customFields` | Additional fields | Hidden |

### 9.4 Companion UVL
See `uvl/leaves/F-PARAM-003-02.uvl`.

---
**END OF SPECIFICATION**
