# F-CRM-002-01 — Lead List & Pipeline View

> **Template:** `feature-spec.md` v1.0.0
> **Template Compliance:** 95%+
> **Status:** DRAFT
> **Feature ID:** `F-CRM-002-01`
> **Suite:** `crm`
> **Node type:** LEAF
> **Parent:** `F-CRM-002`
> **Companion UVL:** `F-CRM-002-01.uvl`
> **Companion AUI:** `F-CRM-002-01.aui.yaml`

---

## 0. Feature Identity & Orientation

### 0.1 One-Line Summary

This feature lets a CRM user sales Rep views leads in table or Kanban pipeline grouped by status

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
| `F-CRM-002-01.enabled` | Boolean | deploy | true |
| `F-CRM-002-01.readOnly` | Boolean | runtime | false |

**Tree position:** `CRM_UI` → `F-CRM-002` → `F-CRM-002-01`

---

## 1. User Goal & Scenarios

### 1.1 User Goal

Sales Rep views leads in table or Kanban pipeline grouped by status

### 1.2 Scenarios

**Scenario 1: View leads in table mode**
- **Outcome:** Paginated table with sorting and filters
- **Notes:** Click row opens Lead Detail

**Scenario 2: Switch to Kanban view**
- **Outcome:** Cards grouped by status columns
- **Notes:** Drag card to change status

**Scenario 3: Filter by rating=HOT**
- **Outcome:** Only hot leads displayed in both views
- **Notes:** Badge count updates

---

## 2. User Journey & Screen Layout

### 2.1 Happy Path Sequence

```mermaid
sequenceDiagram
    actor User
    participant BFF as CRM BFF
    participant SVC as crm.lead
    User->>BFF: Navigate to Lead List & Pipeline View
    BFF->>SVC: GET /leads
    SVC-->>BFF: 200 OK (data)
    BFF-->>User: Render Lead List & Pipeline View
```

### 2.2 Screen Layout

```
┌─────────────────────────────────────────────────┐
│ [F-CRM-002-01] Lead List & Pipeline View                 │
├─────────────────────────────────────────────────┤
│ [table   ] Paginated table: name, company, source, sta │
│ [kanban  ] Columns: New, Contacted, Qualified, Unquali │
│ [control ] Toggle between Table and Kanban views  │
│ [form    ] Filter by status, source, rating, assignee, │
├─────────────────────────────────────────────────┤
│ [EXT] Extension zone                            │
└─────────────────────────────────────────────────┘
```

---

## 3. Interaction Requirements

### 3.1 Zones

| Zone | Type | Priority | Description |
|------|------|----------|-------------|
| `leadTableView` | `table` | high | Paginated table: name, company, source, status, score, rating badge, assignee, created date |
| `leadKanbanView` | `kanban` | high | Columns: New, Contacted, Qualified, Unqualified. Cards show name, score, company |
| `viewToggle` | `control` | high | Toggle between Table and Kanban views |
| `leadFilterPanel` | `form` | high | Filter by status, source, rating, assignee, score range, date range |
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
| Read-only | `F-CRM-002-01.readOnly` = true | All edit controls disabled, view-only mode |

---

## 5. Backend Dependencies & BFF Contract

### 5.1 Service Calls

| Service | Endpoint | Tier | Is Mutation | Failure Mode |
|---------|----------|------|------------|-------------|
| `crm.lead` | `GET /leads` | core | No | Show error toast, allow retry |

### 5.2 Feature Gating

| Mode | `F-CRM-002-01.enabled` | `F-CRM-002-01.readOnly` | Behaviour |
|------|------------------------|--------------------------|-----------|
| Full | true | false | All features available |
| Read-only | true | true | View only, mutations disabled |
| Excluded | false | — | Feature hidden from navigation |

---

## 6. Screen Contract (AUI)

See companion file: `contracts/aui/F-CRM-002-01.aui.yaml`

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

**AC-1: View leads in table mode**
- **Given** a user with appropriate permissions
- **When** view leads in table mode
- **Then** paginated table with sorting and filters

**AC-2: Switch to Kanban view**
- **Given** a user with appropriate permissions
- **When** switch to kanban view
- **Then** cards grouped by status columns

**AC-3: Filter by rating=HOT**
- **Given** a user with appropriate permissions
- **When** filter by rating=hot
- **Then** only hot leads displayed in both views

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