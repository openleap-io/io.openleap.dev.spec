# F-CRM-006-02 — Advanced Search & Filters

> **Template:** `feature-spec.md` v1.0.0
> **Template Compliance:** 95%+
> **Status:** DRAFT
> **Feature ID:** `F-CRM-006-02`
> **Suite:** `crm`
> **Node type:** LEAF
> **Parent:** `F-CRM-006`
> **Companion UVL:** `F-CRM-006-02.uvl`
> **Companion AUI:** `F-CRM-006-02.aui.yaml`

---

## 0. Feature Identity & Orientation

### 0.1 One-Line Summary

This feature lets a CRM user user builds complex search queries with multiple filter criteria

### 0.2 Non-Goals

- This feature does NOT implement backend business logic (see domain spec for `crm.search`).
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
| `F-CRM-006-02.enabled` | Boolean | deploy | true |
| `F-CRM-006-02.readOnly` | Boolean | runtime | false |

**Tree position:** `CRM_UI` → `F-CRM-006` → `F-CRM-006-02`

---

## 1. User Goal & Scenarios

### 1.1 User Goal

User builds complex search queries with multiple filter criteria

### 1.2 Scenarios

**Scenario 1: Search contacts where industry=Technology AND tier=Enterprise**
- **Outcome:** Builder with two filter rows
- **Notes:** Results filtered to matching contacts

**Scenario 2: Search across all entities for keyword**
- **Outcome:** All entity types selected
- **Notes:** Grouped results with counts per type

---

## 2. User Journey & Screen Layout

### 2.1 Happy Path Sequence

```mermaid
sequenceDiagram
    actor User
    participant BFF as CRM BFF
    participant SVC as crm.search
    User->>BFF: Navigate to Advanced Search & Filters
    BFF->>SVC: POST /search
    SVC-->>BFF: 200 OK (data)
    BFF-->>User: Render Advanced Search & Filters
```

### 2.2 Screen Layout

```
┌─────────────────────────────────────────────────┐
│ [F-CRM-006-02] Advanced Search & Filters                 │
├─────────────────────────────────────────────────┤
│ [form    ] Add filter rows: field, operator, value wit │
│ [chips   ] Toggle which entity types to include in res │
│ [control ] Sort by relevance, name, date created, last │
├─────────────────────────────────────────────────┤
│ [EXT] Extension zone                            │
└─────────────────────────────────────────────────┘
```

---

## 3. Interaction Requirements

### 3.1 Zones

| Zone | Type | Priority | Description |
|------|------|----------|-------------|
| `advancedFilterBuilder` | `form` | high | Add filter rows: field, operator, value with AND/OR logic |
| `entityTypeSelector` | `chips` | high | Toggle which entity types to include in results |
| `sortOptions` | `control` | high | Sort by relevance, name, date created, last modified |
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
| Read-only | `F-CRM-006-02.readOnly` = true | All edit controls disabled, view-only mode |

---

## 5. Backend Dependencies & BFF Contract

### 5.1 Service Calls

| Service | Endpoint | Tier | Is Mutation | Failure Mode |
|---------|----------|------|------------|-------------|
| `crm.search` | `POST /search` | core | Yes | Show error toast, allow retry |

### 5.2 Feature Gating

| Mode | `F-CRM-006-02.enabled` | `F-CRM-006-02.readOnly` | Behaviour |
|------|------------------------|--------------------------|-----------|
| Full | true | false | All features available |
| Read-only | true | true | View only, mutations disabled |
| Excluded | false | — | Feature hidden from navigation |

---

## 6. Screen Contract (AUI)

See companion file: `contracts/aui/F-CRM-006-02.aui.yaml`

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

**AC-1: Search contacts where industry=Technology AND tier=Enterprise**
- **Given** a user with appropriate permissions
- **When** search contacts where industry=technology and tier=enterprise
- **Then** builder with two filter rows

**AC-2: Search across all entities for keyword**
- **Given** a user with appropriate permissions
- **When** search across all entities for keyword
- **Then** all entity types selected

---

## 9. Dependencies, Variability & Extension Points

### 9.1 Feature Dependencies

- Requires parent composition `F-CRM-006` to be selected
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