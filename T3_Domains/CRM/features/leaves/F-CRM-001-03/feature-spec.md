# F-CRM-001-03 — Contact Create/Edit Form

> **Template:** `feature-spec.md` v1.0.0
> **Template Compliance:** 95%+
> **Status:** DRAFT
> **Feature ID:** `F-CRM-001-03`
> **Suite:** `crm`
> **Node type:** LEAF
> **Parent:** `F-CRM-001`
> **Companion UVL:** `F-CRM-001-03.uvl`
> **Companion AUI:** `F-CRM-001-03.aui.yaml`

---

## 0. Feature Identity & Orientation

### 0.1 One-Line Summary

This feature lets a CRM user sales Rep creates or edits a contact record with validation

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
| `F-CRM-001-03.enabled` | Boolean | deploy | true |
| `F-CRM-001-03.readOnly` | Boolean | runtime | false |

**Tree position:** `CRM_UI` → `F-CRM-001` → `F-CRM-001-03`

---

## 1. User Goal & Scenarios

### 1.1 User Goal

Sales Rep creates or edits a contact record with validation

### 1.2 Scenarios

**Scenario 1: Create new contact with valid data**
- **Outcome:** Contact saved, redirected to detail view, toast confirmation
- **Notes:** ContactCreated event published

**Scenario 2: Create contact with duplicate email**
- **Outcome:** Duplicate warning shown with link to existing contact
- **Notes:** User can proceed or cancel

**Scenario 3: Edit contact and change account**
- **Outcome:** Account lookup typeahead, select new account
- **Notes:** Relationship updated, previous relationship preserved

---

## 2. User Journey & Screen Layout

### 2.1 Happy Path Sequence

```mermaid
sequenceDiagram
    actor User
    participant BFF as CRM BFF
    participant SVC as crm.contact
    User->>BFF: Navigate to Contact Create/Edit Form
    BFF->>SVC: POST/PUT /contacts
    SVC-->>BFF: 200 OK (data)
    BFF-->>User: Render Contact Create/Edit Form
```

### 2.2 Screen Layout

```
┌─────────────────────────────────────────────────┐
│ [F-CRM-001-03] Contact Create/Edit Form                  │
├─────────────────────────────────────────────────┤
│ [form    ] First name, last name, email, phone, mobile │
│ [lookup  ] Typeahead search for existing accounts or c │
│ [alert   ] Warning panel when potential duplicates det │
├─────────────────────────────────────────────────┤
│ [EXT] Extension zone                            │
└─────────────────────────────────────────────────┘
```

---

## 3. Interaction Requirements

### 3.1 Zones

| Zone | Type | Priority | Description |
|------|------|----------|-------------|
| `contactForm` | `form` | high | First name, last name, email, phone, mobile, title, department, account (lookup), owner (lookup), lead source, lifecycle stage, address, tags, custom fields |
| `accountLookup` | `lookup` | high | Typeahead search for existing accounts or create new |
| `duplicateWarning` | `alert` | high | Warning panel when potential duplicates detected on save |
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
| Read-only | `F-CRM-001-03.readOnly` = true | All edit controls disabled, view-only mode |

---

## 5. Backend Dependencies & BFF Contract

### 5.1 Service Calls

| Service | Endpoint | Tier | Is Mutation | Failure Mode |
|---------|----------|------|------------|-------------|
| `crm.contact` | `POST/PUT /contacts` | core | Yes | Show error toast, allow retry |

### 5.2 Feature Gating

| Mode | `F-CRM-001-03.enabled` | `F-CRM-001-03.readOnly` | Behaviour |
|------|------------------------|--------------------------|-----------|
| Full | true | false | All features available |
| Read-only | true | true | View only, mutations disabled |
| Excluded | false | — | Feature hidden from navigation |

---

## 6. Screen Contract (AUI)

See companion file: `contracts/aui/F-CRM-001-03.aui.yaml`

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

**AC-1: Create new contact with valid data**
- **Given** a user with appropriate permissions
- **When** create new contact with valid data
- **Then** contact saved, redirected to detail view, toast confirmation

**AC-2: Create contact with duplicate email**
- **Given** a user with appropriate permissions
- **When** create contact with duplicate email
- **Then** duplicate warning shown with link to existing contact

**AC-3: Edit contact and change account**
- **Given** a user with appropriate permissions
- **When** edit contact and change account
- **Then** account lookup typeahead, select new account

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