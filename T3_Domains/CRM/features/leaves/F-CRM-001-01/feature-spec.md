# F-CRM-001-01 — Contact List & Search

> **Template:** `feature-spec.md` v1.0.0
> **Template Compliance:** 95%+
> **Status:** DRAFT
> **Feature ID:** `F-CRM-001-01`
> **Suite:** `crm`
> **Node type:** LEAF
> **Parent:** `F-CRM-001`
> **Companion UVL:** `F-CRM-001-01.uvl`
> **Companion AUI:** `F-CRM-001-01.aui.yaml`

---

## 0. Feature Identity & Orientation

### 0.1 One-Line Summary

This feature lets a CRM user sales Rep browses and searches contacts with filters, sorting, and pagination

### 0.2 Non-Goals

- This feature does NOT implement backend business logic (see domain spec for `crm.contact`).
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
| `F-CRM-001-01.enabled` | Boolean | deploy | true |
| `F-CRM-001-01.readOnly` | Boolean | runtime | false |

**Tree position:** `CRM_UI` → `F-CRM-001` → `F-CRM-001-01`

---

## 1. User Goal & Scenarios

### 1.1 User Goal

Sales Rep browses and searches contacts with filters, sorting, and pagination

### 1.2 Scenarios

**Scenario 1: Search contacts by name/email/company**
- **Outcome:** Results displayed in paginated table
- **Notes:** Click row navigates to Contact Detail

**Scenario 2: Filter contacts by status=Active and owner=me**
- **Outcome:** Table shows only matching contacts
- **Notes:** Clear filters resets to all

**Scenario 3: Empty state when no contacts exist**
- **Outcome:** Helpful illustration + Create Contact CTA
- **Notes:** CTA navigates to Create Contact form

---

## 2. User Journey & Screen Layout

### 2.1 Happy Path Sequence

```mermaid
sequenceDiagram
    actor User
    participant BFF as CRM BFF
    participant SVC as crm.contact
    User->>BFF: Navigate to Contact List & Search
    BFF->>SVC: GET /contacts
    SVC-->>BFF: 200 OK (data)
    BFF-->>User: Render Contact List & Search
```

### 2.2 Screen Layout

```
┌─────────────────────────────────────────────────┐
│ [F-CRM-001-01] Contact List & Search                     │
├─────────────────────────────────────────────────┤
│ [table   ] Paginated table of contacts with name, emai │
│ [form    ] Filter by status, owner, account, tags, dat │
│ [control ] Sort by name, created date, last activity │
├─────────────────────────────────────────────────┤
│ [EXT] Extension zone                            │
└─────────────────────────────────────────────────┘
```

---

## 3. Interaction Requirements

### 3.1 Zones

| Zone | Type | Priority | Description |
|------|------|----------|-------------|
| `contactListView` | `table` | high | Paginated table of contacts with name, email, company, owner, last activity |
| `contactFilterPanel` | `form` | high | Filter by status, owner, account, tags, date range |
| `contactSortOptions` | `control` | high | Sort by name, created date, last activity |
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
| Read-only | `F-CRM-001-01.readOnly` = true | All edit controls disabled, view-only mode |

---

## 5. Backend Dependencies & BFF Contract

### 5.1 Service Calls

| Service | Endpoint | Tier | Is Mutation | Failure Mode |
|---------|----------|------|------------|-------------|
| `crm.contact` | `GET /contacts` | core | No | Show error toast, allow retry |

### 5.2 Feature Gating

| Mode | `F-CRM-001-01.enabled` | `F-CRM-001-01.readOnly` | Behaviour |
|------|------------------------|--------------------------|-----------|
| Full | true | false | All features available |
| Read-only | true | true | View only, mutations disabled |
| Excluded | false | — | Feature hidden from navigation |

---

## 6. Screen Contract (AUI)

See companion file: `contracts/aui/F-CRM-001-01.aui.yaml`

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

**AC-1: Search contacts by name/email/company**
- **Given** a user with appropriate permissions
- **When** search contacts by name/email/company
- **Then** results displayed in paginated table

**AC-2: Filter contacts by status=Active and owner=me**
- **Given** a user with appropriate permissions
- **When** filter contacts by status=active and owner=me
- **Then** table shows only matching contacts

**AC-3: Empty state when no contacts exist**
- **Given** a user with appropriate permissions
- **When** empty state when no contacts exist
- **Then** helpful illustration + create contact cta

---

## 9. Dependencies, Variability & Extension Points

### 9.1 Feature Dependencies

- Requires parent composition `F-CRM-001` to be selected
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