# F-CRM-018-02 — Project Detail & Timeline

> **Template:** `feature-spec.md` v1.0.0
> **Template Compliance:** 95%+
> **Status:** DRAFT
> **Feature ID:** `F-CRM-018-02`
> **Suite:** `crm`
> **Node type:** LEAF
> **Parent:** `F-CRM-018`
> **Companion UVL:** `F-CRM-018-02.uvl`
> **Companion AUI:** `F-CRM-018-02.aui.yaml`

---

## 0. Feature Identity & Orientation

### 0.1 One-Line Summary

This feature lets a CRM user account Manager views project progress with milestones

### 0.2 Non-Goals

- This feature does NOT implement backend business logic (see domain spec for `crm.prj`).
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
| `F-CRM-018-02.enabled` | Boolean | deploy | true |
| `F-CRM-018-02.readOnly` | Boolean | runtime | false |

**Tree position:** `CRM_UI` → `F-CRM-018` → `F-CRM-018-02`

---

## 1. User Goal & Scenarios

### 1.1 User Goal

Account Manager views project progress with milestones

### 1.2 Scenarios

**Scenario 1: View project with milestones**
- **Outcome:** Timeline shows all milestones with dates and status
- **Notes:** Click milestone to mark complete

**Scenario 2: View completed project**
- **Outcome:** All milestones green, status=Completed
- **Notes:** Read-only mode

---

## 2. User Journey & Screen Layout

### 2.1 Happy Path Sequence

```mermaid
sequenceDiagram
    actor User
    participant BFF as CRM BFF
    participant SVC as crm.prj
    User->>BFF: Navigate to Project Detail & Timeline
    BFF->>SVC: GET /projects/{id}
    SVC-->>BFF: 200 OK (data)
    BFF-->>User: Render Project Detail & Timeline
```

### 2.2 Screen Layout

```
┌─────────────────────────────────────────────────┐
│ [F-CRM-018-02] Project Detail & Timeline                 │
├─────────────────────────────────────────────────┤
│ [display ] Name, account, status, overall progress bar │
│ [timeline] Visual timeline of milestones with status b │
│ [timeline] Recent project activities and updates  │
│ [toolbar ] Edit, Add Milestone, Add Task, Complete │
├─────────────────────────────────────────────────┤
│ [EXT] Extension zone                            │
└─────────────────────────────────────────────────┘
```

---

## 3. Interaction Requirements

### 3.1 Zones

| Zone | Type | Priority | Description |
|------|------|----------|-------------|
| `projectHeader` | `display` | high | Name, account, status, overall progress bar, due date |
| `milestoneTimeline` | `timeline` | high | Visual timeline of milestones with status badges |
| `projectActivityFeed` | `timeline` | high | Recent project activities and updates |
| `projectActionBar` | `toolbar` | high | Edit, Add Milestone, Add Task, Complete |
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
| Read-only | `F-CRM-018-02.readOnly` = true | All edit controls disabled, view-only mode |

---

## 5. Backend Dependencies & BFF Contract

### 5.1 Service Calls

| Service | Endpoint | Tier | Is Mutation | Failure Mode |
|---------|----------|------|------------|-------------|
| `crm.prj` | `GET /projects/{id}` | core | No | Show error toast, allow retry |

### 5.2 Feature Gating

| Mode | `F-CRM-018-02.enabled` | `F-CRM-018-02.readOnly` | Behaviour |
|------|------------------------|--------------------------|-----------|
| Full | true | false | All features available |
| Read-only | true | true | View only, mutations disabled |
| Excluded | false | — | Feature hidden from navigation |

---

## 6. Screen Contract (AUI)

See companion file: `contracts/aui/F-CRM-018-02.aui.yaml`

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

**AC-1: View project with milestones**
- **Given** a user with appropriate permissions
- **When** view project with milestones
- **Then** timeline shows all milestones with dates and status

**AC-2: View completed project**
- **Given** a user with appropriate permissions
- **When** view completed project
- **Then** all milestones green, status=completed

---

## 9. Dependencies, Variability & Extension Points

### 9.1 Feature Dependencies

- Requires parent composition `F-CRM-018` to be selected
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