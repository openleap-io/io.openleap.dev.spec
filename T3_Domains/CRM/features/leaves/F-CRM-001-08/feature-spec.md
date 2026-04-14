# F-CRM-001-08 — Duplicate Detection & Merge

> **Template:** `feature-spec.md` v1.0.0
> **Template Compliance:** 95%+
> **Status:** DRAFT
> **Feature ID:** `F-CRM-001-08`
> **Suite:** `crm`
> **Node type:** LEAF
> **Parent:** `F-CRM-001`
> **Companion UVL:** `F-CRM-001-08.uvl`
> **Companion AUI:** `F-CRM-001-08.aui.yaml`

---

## 0. Feature Identity & Orientation

### 0.1 One-Line Summary

This feature lets a CRM user cRM Admin detects and merges duplicate contact records

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
| `F-CRM-001-08.enabled` | Boolean | deploy | true |
| `F-CRM-001-08.readOnly` | Boolean | runtime | false |

**Tree position:** `CRM_UI` → `F-CRM-001` → `F-CRM-001-08`

---

## 1. User Goal & Scenarios

### 1.1 User Goal

CRM Admin detects and merges duplicate contact records

### 1.2 Scenarios

**Scenario 1: Detect duplicates for a contact**
- **Outcome:** System shows matches with >80% similarity
- **Notes:** User reviews and decides to merge or dismiss

**Scenario 2: Merge two contacts**
- **Outcome:** Wizard guides through field resolution
- **Notes:** Survivor record updated, merged record archived, all references transferred

**Scenario 3: No duplicates found**
- **Outcome:** Empty state: No potential duplicates detected
- **Notes:** User can manually search

---

## 2. User Journey & Screen Layout

### 2.1 Happy Path Sequence

```mermaid
sequenceDiagram
    actor User
    participant BFF as CRM BFF
    participant SVC as crm.contact
    User->>BFF: Navigate to Duplicate Detection & Merge
    BFF->>SVC: GET /contacts/{id}/duplicates, POST /contacts/merge
    SVC-->>BFF: 200 OK (data)
    BFF-->>User: Render Duplicate Detection & Merge
```

### 2.2 Screen Layout

```
┌─────────────────────────────────────────────────┐
│ [F-CRM-001-08] Duplicate Detection & Merge               │
├─────────────────────────────────────────────────┤
│ [table   ] Potential duplicates with similarity score, │
│ [wizard  ] Step 1: Select survivor. Step 2: Resolve fi │
│ [display ] Side-by-side comparison of records to merge │
│ [display ] Summary of merged record, transferred relat │
├─────────────────────────────────────────────────┤
│ [EXT] Extension zone                            │
└─────────────────────────────────────────────────┘
```

---

## 3. Interaction Requirements

### 3.1 Zones

| Zone | Type | Priority | Description |
|------|------|----------|-------------|
| `duplicateList` | `table` | high | Potential duplicates with similarity score, match fields highlighted |
| `mergeWizard` | `wizard` | high | Step 1: Select survivor. Step 2: Resolve field conflicts. Step 3: Review & confirm |
| `mergePreview` | `display` | high | Side-by-side comparison of records to merge with field resolution controls |
| `mergeResultSummary` | `display` | high | Summary of merged record, transferred relationships, archived records |
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
| Read-only | `F-CRM-001-08.readOnly` = true | All edit controls disabled, view-only mode |

---

## 5. Backend Dependencies & BFF Contract

### 5.1 Service Calls

| Service | Endpoint | Tier | Is Mutation | Failure Mode |
|---------|----------|------|------------|-------------|
| `crm.contact` | `GET /contacts/{id}/duplicates, POST /contacts/merge` | core | Yes | Show error toast, allow retry |

### 5.2 Feature Gating

| Mode | `F-CRM-001-08.enabled` | `F-CRM-001-08.readOnly` | Behaviour |
|------|------------------------|--------------------------|-----------|
| Full | true | false | All features available |
| Read-only | true | true | View only, mutations disabled |
| Excluded | false | — | Feature hidden from navigation |

---

## 6. Screen Contract (AUI)

See companion file: `contracts/aui/F-CRM-001-08.aui.yaml`

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

**AC-1: Detect duplicates for a contact**
- **Given** a user with appropriate permissions
- **When** detect duplicates for a contact
- **Then** system shows matches with >80% similarity

**AC-2: Merge two contacts**
- **Given** a user with appropriate permissions
- **When** merge two contacts
- **Then** wizard guides through field resolution

**AC-3: No duplicates found**
- **Given** a user with appropriate permissions
- **When** no duplicates found
- **Then** empty state: no potential duplicates detected

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