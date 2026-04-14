# F-CRM-017-04 — Quote PDF Generation

> **Template:** `feature-spec.md` v1.0.0
> **Template Compliance:** 95%+
> **Status:** DRAFT
> **Feature ID:** `F-CRM-017-04`
> **Suite:** `crm`
> **Node type:** LEAF
> **Parent:** `F-CRM-017`
> **Companion UVL:** `F-CRM-017-04.uvl`
> **Companion AUI:** `F-CRM-017-04.aui.yaml`

---

## 0. Feature Identity & Orientation

### 0.1 One-Line Summary

This feature lets a CRM user user generates a branded PDF proposal from a quote

### 0.2 Non-Goals

- This feature does NOT implement backend business logic (see domain spec for `crm.cpq`).
- This feature does NOT define the API contract (see `contracts/http/crm/` OpenAPI).

### 0.3 Entry & Exit Points

| Direction | Target | Trigger |
|-----------|--------|---------|
| Entry | This feature | Navigation menu, search result click, related record link |
| Exit | Related detail views | Click on related record |
| Exit | Create/Edit forms | Action button click |

### 0.4 Variability Points

| Attribute | Type | Binding Time | Default |
|-----------|------|-------------|---------|
| `F-CRM-017-04.enabled` | Boolean | deploy | true |
| `F-CRM-017-04.readOnly` | Boolean | runtime | false |

**Tree position:** `CRM_UI` → `F-CRM-017` → `F-CRM-017-04`

---

## 1. User Goal & Scenarios

### 1.1 User Goal

User generates a branded PDF proposal from a quote

### 1.2 Scenarios

**Scenario 1: Generate PDF for approved quote**
- **Outcome:** Click generate, preview PDF
- **Notes:** DocumentGenerated event, PDF stored in MinIO

**Scenario 2: Send PDF to customer**
- **Outcome:** Click Send, email composer opens with PDF attached
- **Notes:** Email sent with quote PDF

---

## 2. User Journey & Screen Layout

### 2.1 Happy Path Sequence

```mermaid
sequenceDiagram
    actor User
    participant BFF as CRM BFF
    participant SVC as crm.cpq
    User->>BFF: Navigate to Quote PDF Generation
    BFF->>SVC: POST /quotes/{id}/generate-pdf
    SVC-->>BFF: 200 OK (data)
    BFF-->>User: Render Quote PDF Generation
```

### 2.2 Screen Layout

```
┌─────────────────────────────────────────────────┐
│ [F-CRM-017-04] Quote PDF Generation                      │
├─────────────────────────────────────────────────┤
│ [display ] Preview generated PDF with company branding │
│ [dropdown] Select PDF template (if multiple available) │
│ [button  ] Generate PDF and attach to quote record │
├─────────────────────────────────────────────────┤
│ [EXT] Extension zone                            │
└─────────────────────────────────────────────────┘
```

---

## 3. Interaction Requirements

### 3.1 Zones

| Zone | Type | Priority | Description |
|------|------|----------|-------------|
| `pdfPreview` | `display` | high | Preview generated PDF with company branding |
| `templateSelector` | `dropdown` | high | Select PDF template (if multiple available) |
| `generateButton` | `button` | high | Generate PDF and attach to quote record |
| `extensionZone` | `feature-gated` | low | Extension point for product customization |

### 3.2 Actions


---

## 4. Edge Cases & Screen States

| State | Condition | Behaviour |
|-------|-----------|-----------|
| Loading | Data fetching in progress | Skeleton loader displayed |
| Empty | No records match criteria | Empty state illustration + CTA |
| Populated | Records available | Normal rendering |
| Error | Service unavailable | Error message with retry button |
| Partial | Some data loaded, some failed | Available data shown, failed sections show inline error |
| Read-only | `F-CRM-017-04.readOnly` = true | All edit controls disabled, view-only mode |

---

## 5. Backend Dependencies & BFF Contract

### 5.1 Service Calls

| Service | Endpoint | Tier | Is Mutation | Failure Mode |
|---------|----------|------|------------|-------------|
| `crm.cpq` | `POST /quotes/{id}/generate-pdf` | core | Yes | Show error toast, allow retry |

### 5.2 Feature Gating

| Mode | `F-CRM-017-04.enabled` | `F-CRM-017-04.readOnly` | Behaviour |
|------|------------------------|--------------------------|-----------|
| Full | true | false | All features available |
| Read-only | true | true | View only, mutations disabled |
| Excluded | false | — | Feature hidden from navigation |

---

## 6. Screen Contract (AUI)

See companion file: `contracts/aui/F-CRM-017-04.aui.yaml`

---

## 7. i18n, Permissions & Accessibility

### 7.1 Permissions

| Action | Required Role |
|--------|--------------|
| View | `crm-readonly`, `crm-sales-rep`, `crm-sales-manager`, `crm-admin` |

### 7.2 Accessibility

- All interactive elements must be keyboard-navigable
- ARIA labels on all form fields and action buttons
- Color is never the sole indicator of state (always paired with icon or text)
- Screen reader announcements for dynamic content updates

---

## 8. Acceptance Criteria

**AC-1: Generate PDF for approved quote**
- **Given** a user with appropriate permissions
- **When** generate pdf for approved quote
- **Then** click generate, preview pdf

**AC-2: Send PDF to customer**
- **Given** a user with appropriate permissions
- **When** send pdf to customer
- **Then** click send, email composer opens with pdf attached

---

## 9. Dependencies, Variability & Extension Points

### 9.1 Feature Dependencies

- Requires parent composition `F-CRM-017` to be selected
- Requires `F-CRM-006` (Global Search) for navigation and search integration

### 9.2 Extension Zones

| Zone | Interface | Default Behaviour |
|------|-----------|-------------------|
| `extensionZone` | Render custom components | Collapsed (hidden when empty) |

---

## 10. Change Log & Review

| Date | Version | Author | Changes |
|------|---------|--------|---------|
| 2026-04-03 | 1.0.0 | OpenLeap Architecture Team | Initial feature spec |