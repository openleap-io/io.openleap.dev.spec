# F-CRM-016-01 — Ticket List & Queue

> **Template:** `feature-spec.md` v1.0.0
> **Template Compliance:** 95%+
> **Status:** DRAFT
> **Feature ID:** `F-CRM-016-01`
> **Suite:** `crm`
> **Node type:** LEAF
> **Parent:** `F-CRM-016`
> **Companion UVL:** `F-CRM-016-01.uvl`
> **Companion AUI:** `F-CRM-016-01.aui.yaml`

---

## 0. Feature Identity & Orientation

### 0.1 One-Line Summary

This feature lets a CRM user support Agent views and manages ticket queue

### 0.2 Non-Goals

- This feature does NOT implement backend business logic (see domain spec for `crm.sup`).
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
| `F-CRM-016-01.enabled` | Boolean | deploy | true |
| `F-CRM-016-01.readOnly` | Boolean | runtime | false |

**Tree position:** `CRM_UI` → `F-CRM-016` → `F-CRM-016-01`

---

## 1. User Goal & Scenarios

### 1.1 User Goal

Support Agent views and manages ticket queue

### 1.2 Scenarios

**Scenario 1: View my ticket queue**
- **Outcome:** Table filtered to assigned tickets, sorted by SLA urgency
- **Notes:** Click row opens ticket detail

**Scenario 2: View unassigned tickets**
- **Outcome:** Unassigned queue with pick-up button
- **Notes:** Click pick-up to self-assign

---

## 2. User Journey & Screen Layout

### 2.1 Happy Path Sequence

```mermaid
sequenceDiagram
    actor User
    participant BFF as CRM BFF
    participant SVC as crm.sup
    User->>BFF: Navigate to Ticket List & Queue
    BFF->>SVC: GET /tickets
    SVC-->>BFF: 200 OK (data)
    BFF-->>User: Render Ticket List & Queue
```

### 2.2 Screen Layout

```
┌─────────────────────────────────────────────────┐
│ [F-CRM-016-01] Ticket List & Queue                       │
├─────────────────────────────────────────────────┤
│ [table   ] Subject, priority, status, category, SLA st │
│ [form    ] Filter by priority, status, category, assig │
│ [tabs    ] Tabs: My Queue, Unassigned, All Tickets │
│ [badge   ] SLA badges: on track (green), at risk (yell │
├─────────────────────────────────────────────────┤
│ [EXT] Extension zone                            │
└─────────────────────────────────────────────────┘
```

---

## 3. Interaction Requirements

### 3.1 Zones

| Zone | Type | Priority | Description |
|------|------|----------|-------------|
| `ticketTable` | `table` | high | Subject, priority, status, category, SLA status, assignee, created date |
| `ticketFilter` | `form` | high | Filter by priority, status, category, assignee, SLA status |
| `queueSelector` | `tabs` | high | Tabs: My Queue, Unassigned, All Tickets |
| `slaIndicator` | `badge` | high | SLA badges: on track (green), at risk (yellow), breached (red) |
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
| Read-only | `F-CRM-016-01.readOnly` = true | All edit controls disabled, view-only mode |

---

## 5. Backend Dependencies & BFF Contract

### 5.1 Service Calls

| Service | Endpoint | Tier | Is Mutation | Failure Mode |
|---------|----------|------|------------|-------------|
| `crm.sup` | `GET /tickets` | core | No | Show error toast, allow retry |

### 5.2 Feature Gating

| Mode | `F-CRM-016-01.enabled` | `F-CRM-016-01.readOnly` | Behaviour |
|------|------------------------|--------------------------|-----------|
| Full | true | false | All features available |
| Read-only | true | true | View only, mutations disabled |
| Excluded | false | — | Feature hidden from navigation |

---

## 6. Screen Contract (AUI)

See companion file: `contracts/aui/F-CRM-016-01.aui.yaml`

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

**AC-1: View my ticket queue**
- **Given** a user with appropriate permissions
- **When** view my ticket queue
- **Then** table filtered to assigned tickets, sorted by sla urgency

**AC-2: View unassigned tickets**
- **Given** a user with appropriate permissions
- **When** view unassigned tickets
- **Then** unassigned queue with pick-up button

---

## 9. Dependencies, Variability & Extension Points

### 9.1 Feature Dependencies

- Requires parent composition `F-CRM-016` to be selected
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