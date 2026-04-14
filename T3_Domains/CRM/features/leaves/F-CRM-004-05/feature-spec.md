# F-CRM-004-05 — Note Editor

> **Template:** `feature-spec.md` v1.0.0
> **Template Compliance:** 95%+
> **Status:** DRAFT
> **Feature ID:** `F-CRM-004-05`
> **Suite:** `crm`
> **Node type:** LEAF
> **Parent:** `F-CRM-004`
> **Companion UVL:** `F-CRM-004-05.uvl`
> **Companion AUI:** `F-CRM-004-05.aui.yaml`

---

## 0. Feature Identity & Orientation

### 0.1 One-Line Summary

This feature lets a CRM user user creates rich-text notes attached to CRM records

### 0.2 Non-Goals

- This feature does NOT implement backend business logic (see domain spec for `crm.act`).
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
| `F-CRM-004-05.enabled` | Boolean | deploy | true |
| `F-CRM-004-05.readOnly` | Boolean | runtime | false |

**Tree position:** `CRM_UI` → `F-CRM-004` → `F-CRM-004-05`

---

## 1. User Goal & Scenarios

### 1.1 User Goal

User creates rich-text notes attached to CRM records

### 1.2 Scenarios

**Scenario 1: Add note to opportunity**
- **Outcome:** Write note, link to opportunity, save
- **Notes:** NoteAdded event → timeline

**Scenario 2: View notes on contact detail**
- **Outcome:** Notes tab shows all notes reverse-chronological
- **Notes:** Click note to expand full text

---

## 2. User Journey & Screen Layout

### 2.1 Happy Path Sequence

```mermaid
sequenceDiagram
    actor User
    participant BFF as CRM BFF
    participant SVC as crm.act
    User->>BFF: Navigate to Note Editor
    BFF->>SVC: POST /notes
    SVC-->>BFF: 200 OK (data)
    BFF-->>User: Render Note Editor
```

### 2.2 Screen Layout

```
┌─────────────────────────────────────────────────┐
│ [F-CRM-004-05] Note Editor                               │
├─────────────────────────────────────────────────┤
│ [richtext] Rich text editor with formatting toolbar: b │
│ [lookup  ] Polymorphic lookup: attach to contact, acco │
│ [list    ] Recent notes for the current record    │
├─────────────────────────────────────────────────┤
│ [EXT] Extension zone                            │
└─────────────────────────────────────────────────┘
```

---

## 3. Interaction Requirements

### 3.1 Zones

| Zone | Type | Priority | Description |
|------|------|----------|-------------|
| `noteEditor` | `richtext` | high | Rich text editor with formatting toolbar: bold, italic, lists, links |
| `noteRelatedTo` | `lookup` | high | Polymorphic lookup: attach to contact, account, lead, opportunity |
| `noteList` | `list` | high | Recent notes for the current record |
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
| Read-only | `F-CRM-004-05.readOnly` = true | All edit controls disabled, view-only mode |

---

## 5. Backend Dependencies & BFF Contract

### 5.1 Service Calls

| Service | Endpoint | Tier | Is Mutation | Failure Mode |
|---------|----------|------|------------|-------------|
| `crm.act` | `POST /notes` | core | Yes | Show error toast, allow retry |

### 5.2 Feature Gating

| Mode | `F-CRM-004-05.enabled` | `F-CRM-004-05.readOnly` | Behaviour |
|------|------------------------|--------------------------|-----------|
| Full | true | false | All features available |
| Read-only | true | true | View only, mutations disabled |
| Excluded | false | — | Feature hidden from navigation |

---

## 6. Screen Contract (AUI)

See companion file: `contracts/aui/F-CRM-004-05.aui.yaml`

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

**AC-1: Add note to opportunity**
- **Given** a user with appropriate permissions
- **When** add note to opportunity
- **Then** write note, link to opportunity, save

**AC-2: View notes on contact detail**
- **Given** a user with appropriate permissions
- **When** view notes on contact detail
- **Then** notes tab shows all notes reverse-chronological

---

## 9. Dependencies, Variability & Extension Points

### 9.1 Feature Dependencies

- Requires parent composition `F-CRM-004` to be selected
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