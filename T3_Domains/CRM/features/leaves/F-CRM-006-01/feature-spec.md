# F-CRM-006-01 — Global Search Bar

> **Template:** `feature-spec.md` v1.0.0
> **Template Compliance:** 95%+
> **Status:** DRAFT
> **Feature ID:** `F-CRM-006-01`
> **Suite:** `crm`
> **Node type:** LEAF
> **Parent:** `F-CRM-006`
> **Companion UVL:** `F-CRM-006-01.uvl`
> **Companion AUI:** `F-CRM-006-01.aui.yaml`

---

## 0. Feature Identity & Orientation

### 0.1 One-Line Summary

This feature lets a CRM user user searches across all CRM entities from a persistent search bar

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
| `F-CRM-006-01.enabled` | Boolean | deploy | true |
| `F-CRM-006-01.readOnly` | Boolean | runtime | false |

**Tree position:** `CRM_UI` → `F-CRM-006` → `F-CRM-006-01`

---

## 1. User Goal & Scenarios

### 1.1 User Goal

User searches across all CRM entities from a persistent search bar

### 1.2 Scenarios

**Scenario 1: Type 'Acme' in search bar**
- **Outcome:** Typeahead shows matching contacts, accounts, leads, opportunities
- **Notes:** Click result navigates to detail view

**Scenario 2: Search returns no results**
- **Outcome:** Empty state with search tips
- **Notes:** Suggest broadening search terms

---

## 2. User Journey & Screen Layout

### 2.1 Happy Path Sequence

```mermaid
sequenceDiagram
    actor User
    participant BFF as CRM BFF
    participant SVC as crm.search
    User->>BFF: Navigate to Global Search Bar
    BFF->>SVC: POST /search
    SVC-->>BFF: 200 OK (data)
    BFF-->>User: Render Global Search Bar
```

### 2.2 Screen Layout

```
┌─────────────────────────────────────────────────┐
│ [F-CRM-006-01] Global Search Bar                         │
├─────────────────────────────────────────────────┤
│ [search  ] Persistent top-bar search with typeahead su │
│ [list    ] Grouped results: Contacts, Accounts, Leads, │
│ [card    ] Entity icon, name, type badge, key fields p │
├─────────────────────────────────────────────────┤
│ [EXT] Extension zone                            │
└─────────────────────────────────────────────────┘
```

---

## 3. Interaction Requirements

### 3.1 Zones

| Zone | Type | Priority | Description |
|------|------|----------|-------------|
| `searchInput` | `search` | high | Persistent top-bar search with typeahead suggestions |
| `searchResults` | `list` | high | Grouped results: Contacts, Accounts, Leads, Opportunities, etc. |
| `searchResultCard` | `card` | high | Entity icon, name, type badge, key fields preview |
| `extensionZone` | `feature-gated` | low | Extension point for product customization |

### 3.2 Actions

| Action | Trigger | Confirmation | Events |
|--------|---------|-------------|--------|
| Navigate to detail | Row click | None | None |
| Filter | Filter change | None | None |
| Sort | Column header click | None | None |

---

## 4. Edge Cases & Screen States

| State | Condition | Behaviour |
|-------|-----------|-----------|
| Loading | Data fetching in progress | Skeleton loader displayed |
| Empty | No records match criteria | Empty state illustration + CTA |
| Populated | Records available | Normal rendering |
| Error | Service unavailable | Error message with retry button |
| Partial | Some data loaded, some failed | Available data shown, failed sections show inline error |
| Read-only | `F-CRM-006-01.readOnly` = true | All edit controls disabled, view-only mode |

---

## 5. Backend Dependencies & BFF Contract

### 5.1 Service Calls

| Service | Endpoint | Tier | Is Mutation | Failure Mode |
|---------|----------|------|------------|-------------|
| `crm.search` | `POST /search` | core | Yes | Show error toast, allow retry |

### 5.2 Feature Gating

| Mode | `F-CRM-006-01.enabled` | `F-CRM-006-01.readOnly` | Behaviour |
|------|------------------------|--------------------------|-----------|
| Full | true | false | All features available |
| Read-only | true | true | View only, mutations disabled |
| Excluded | false | — | Feature hidden from navigation |

---

## 6. Screen Contract (AUI)

See companion file: `contracts/aui/F-CRM-006-01.aui.yaml`

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

**AC-1: Type 'Acme' in search bar**
- **Given** a user with appropriate permissions
- **When** type 'acme' in search bar
- **Then** typeahead shows matching contacts, accounts, leads, opportunities

**AC-2: Search returns no results**
- **Given** a user with appropriate permissions
- **When** search returns no results
- **Then** empty state with search tips

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