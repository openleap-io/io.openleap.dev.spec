# F-CRM-010-04 — Report Scheduling & Export

> **Template:** `feature-spec.md` v1.0.0
> **Template Compliance:** 95%+
> **Status:** DRAFT
> **Feature ID:** `F-CRM-010-04`
> **Suite:** `crm`
> **Node type:** LEAF
> **Parent:** `F-CRM-010`
> **Companion UVL:** `F-CRM-010-04.uvl`
> **Companion AUI:** `F-CRM-010-04.aui.yaml`

---

## 0. Feature Identity & Orientation

### 0.1 One-Line Summary

This feature lets a CRM user user schedules automated report delivery

### 0.2 Non-Goals

- This feature does NOT implement backend business logic (see domain spec for `crm.rpt`).
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
| `F-CRM-010-04.enabled` | Boolean | deploy | true |
| `F-CRM-010-04.readOnly` | Boolean | runtime | false |

**Tree position:** `CRM_UI` → `F-CRM-010` → `F-CRM-010-04`

---

## 1. User Goal & Scenarios

### 1.1 User Goal

User schedules automated report delivery

### 1.2 Scenarios

**Scenario 1: Schedule weekly pipeline report**
- **Outcome:** Every Monday 8am, PDF to sales-team@company.com
- **Notes:** Schedule created, next run shown

**Scenario 2: View delivery history**
- **Outcome:** List of past deliveries with status
- **Notes:** Re-download past reports

---

## 2. User Journey & Screen Layout

### 2.1 Happy Path Sequence

```mermaid
sequenceDiagram
    actor User
    participant BFF as CRM BFF
    participant SVC as crm.rpt
    User->>BFF: Navigate to Report Scheduling & Export
    BFF->>SVC: POST /report-schedules
    SVC-->>BFF: 200 OK (data)
    BFF-->>User: Render Report Scheduling & Export
```

### 2.2 Screen Layout

```
┌─────────────────────────────────────────────────┐
│ [F-CRM-010-04] Report Scheduling & Export                │
├─────────────────────────────────────────────────┤
│ [form    ] Report selection, frequency (daily/weekly/m │
│ [table   ] Active schedules: report name, frequency, n │
│ [table   ] Past deliveries: date, status, download lin │
├─────────────────────────────────────────────────┤
│ [EXT] Extension zone                            │
└─────────────────────────────────────────────────┘
```

---

## 3. Interaction Requirements

### 3.1 Zones

| Zone | Type | Priority | Description |
|------|------|----------|-------------|
| `scheduleForm` | `form` | high | Report selection, frequency (daily/weekly/monthly), recipients, format (PDF/CSV/XLSX) |
| `scheduleList` | `table` | high | Active schedules: report name, frequency, next run, recipients |
| `deliveryHistory` | `table` | high | Past deliveries: date, status, download link |
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
| Read-only | `F-CRM-010-04.readOnly` = true | All edit controls disabled, view-only mode |

---

## 5. Backend Dependencies & BFF Contract

### 5.1 Service Calls

| Service | Endpoint | Tier | Is Mutation | Failure Mode |
|---------|----------|------|------------|-------------|
| `crm.rpt` | `POST /report-schedules` | core | Yes | Show error toast, allow retry |

### 5.2 Feature Gating

| Mode | `F-CRM-010-04.enabled` | `F-CRM-010-04.readOnly` | Behaviour |
|------|------------------------|--------------------------|-----------|
| Full | true | false | All features available |
| Read-only | true | true | View only, mutations disabled |
| Excluded | false | — | Feature hidden from navigation |

---

## 6. Screen Contract (AUI)

See companion file: `contracts/aui/F-CRM-010-04.aui.yaml`

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

**AC-1: Schedule weekly pipeline report**
- **Given** a user with appropriate permissions
- **When** schedule weekly pipeline report
- **Then** every monday 8am, pdf to sales-team@company.com

**AC-2: View delivery history**
- **Given** a user with appropriate permissions
- **When** view delivery history
- **Then** list of past deliveries with status

---

## 9. Dependencies, Variability & Extension Points

### 9.1 Feature Dependencies

- Requires parent composition `F-CRM-010` to be selected
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