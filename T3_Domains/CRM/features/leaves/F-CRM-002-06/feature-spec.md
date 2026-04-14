# F-CRM-002-06 — Lead Import

> **Template:** `feature-spec.md` v1.0.0
> **Template Compliance:** 95%+
> **Status:** DRAFT
> **Feature ID:** `F-CRM-002-06`
> **Suite:** `crm`
> **Node type:** LEAF
> **Parent:** `F-CRM-002`
> **Companion UVL:** `F-CRM-002-06.uvl`
> **Companion AUI:** `F-CRM-002-06.aui.yaml`

---

## 0. Feature Identity & Orientation

### 0.1 One-Line Summary

This feature lets a CRM user cRM Admin bulk-imports leads from CSV or Excel file

### 0.2 Non-Goals

- This feature does NOT implement backend business logic (see domain spec for `crm.lead`).
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
| `F-CRM-002-06.enabled` | Boolean | deploy | true |
| `F-CRM-002-06.readOnly` | Boolean | runtime | false |

**Tree position:** `CRM_UI` → `F-CRM-002` → `F-CRM-002-06`

---

## 1. User Goal & Scenarios

### 1.1 User Goal

CRM Admin bulk-imports leads from CSV or Excel file

### 1.2 Scenarios

**Scenario 1: Import 500 leads from CSV**
- **Outcome:** Upload, map columns, preview, confirm
- **Notes:** Leads created asynchronously, progress bar updates

**Scenario 2: Import with duplicate emails**
- **Outcome:** Duplicates flagged in preview
- **Notes:** User can skip or overwrite duplicates

**Scenario 3: Import with validation errors**
- **Outcome:** Error rows highlighted in red
- **Notes:** Downloadable error report for correction

---

## 2. User Journey & Screen Layout

### 2.1 Happy Path Sequence

```mermaid
sequenceDiagram
    actor User
    participant BFF as CRM BFF
    participant SVC as crm.lead
    User->>BFF: Navigate to Lead Import
    BFF->>SVC: POST /leads/import
    SVC-->>BFF: 200 OK (data)
    BFF-->>User: Render Lead Import
```

### 2.2 Screen Layout

```
┌─────────────────────────────────────────────────┐
│ [F-CRM-002-06] Lead Import                               │
├─────────────────────────────────────────────────┤
│ [upload  ] Drag-and-drop CSV/XLSX upload area with for │
│ [form    ] Column mapping: match file columns to lead  │
│ [table   ] Preview of first 10 rows with validation st │
│ [progress] Progress bar with count of imported, skippe │
│ [display ] Summary: X imported, Y duplicates skipped,  │
├─────────────────────────────────────────────────┤
│ [EXT] Extension zone                            │
└─────────────────────────────────────────────────┘
```

---

## 3. Interaction Requirements

### 3.1 Zones

| Zone | Type | Priority | Description |
|------|------|----------|-------------|
| `importUpload` | `upload` | high | Drag-and-drop CSV/XLSX upload area with format requirements |
| `importMapping` | `form` | high | Column mapping: match file columns to lead fields |
| `importPreview` | `table` | high | Preview of first 10 rows with validation status per row |
| `importProgress` | `progress` | high | Progress bar with count of imported, skipped, errored records |
| `importResult` | `display` | high | Summary: X imported, Y duplicates skipped, Z errors with downloadable error report |
| `extensionZone` | `feature-gated` | low | Extension point for product customization |

### 3.2 Actions

| Action | Trigger | Confirmation | Events |
|--------|---------|-------------|--------|
| Save | Button click | None (optimistic) | State change event |
| Cancel | Button click | Unsaved changes guard | None |

---

## 4. Edge Cases & Screen States

| State | Condition | Behaviour |
|-------|-----------|-----------|
| Loading | Data fetching in progress | Skeleton loader displayed |
| Empty | No records match criteria | Empty state illustration + CTA |
| Populated | Records available | Normal rendering |
| Error | Service unavailable | Error message with retry button |
| Partial | Some data loaded, some failed | Available data shown, failed sections show inline error |
| Read-only | `F-CRM-002-06.readOnly` = true | All edit controls disabled, view-only mode |

---

## 5. Backend Dependencies & BFF Contract

### 5.1 Service Calls

| Service | Endpoint | Tier | Is Mutation | Failure Mode |
|---------|----------|------|------------|-------------|
| `crm.lead` | `POST /leads/import` | core | Yes | Show error toast, allow retry |

### 5.2 Feature Gating

| Mode | `F-CRM-002-06.enabled` | `F-CRM-002-06.readOnly` | Behaviour |
|------|------------------------|--------------------------|-----------|
| Full | true | false | All features available |
| Read-only | true | true | View only, mutations disabled |
| Excluded | false | — | Feature hidden from navigation |

---

## 6. Screen Contract (AUI)

See companion file: `contracts/aui/F-CRM-002-06.aui.yaml`

---

## 7. i18n, Permissions & Accessibility

### 7.1 Permissions

| Action | Required Role |
|--------|--------------|
| View | `crm-readonly`, `crm-sales-rep`, `crm-sales-manager`, `crm-admin` |
| Create/Edit | `crm-sales-rep`, `crm-sales-manager`, `crm-admin` |
| Delete | `crm-sales-manager`, `crm-admin` |

### 7.2 Accessibility

- All interactive elements must be keyboard-navigable
- ARIA labels on all form fields and action buttons
- Color is never the sole indicator of state (always paired with icon or text)
- Screen reader announcements for dynamic content updates

---

## 8. Acceptance Criteria

**AC-1: Import 500 leads from CSV**
- **Given** a user with appropriate permissions
- **When** import 500 leads from csv
- **Then** upload, map columns, preview, confirm

**AC-2: Import with duplicate emails**
- **Given** a user with appropriate permissions
- **When** import with duplicate emails
- **Then** duplicates flagged in preview

**AC-3: Import with validation errors**
- **Given** a user with appropriate permissions
- **When** import with validation errors
- **Then** error rows highlighted in red

---

## 9. Dependencies, Variability & Extension Points

### 9.1 Feature Dependencies

- Requires parent composition `F-CRM-002` to be selected
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