# F-CRM-016-02 — Ticket Detail & Resolution

> **Template:** `feature-spec.md` v1.0.0
> **Template Compliance:** 95%+
> **Status:** DRAFT
> **Feature ID:** `F-CRM-016-02`
> **Suite:** `crm`
> **Node type:** LEAF
> **Parent:** `F-CRM-016`
> **Companion UVL:** `F-CRM-016-02.uvl`
> **Companion AUI:** `F-CRM-016-02.aui.yaml`

---

## 0. Feature Identity & Orientation

### 0.1 One-Line Summary

This feature lets a CRM user support Agent works on and resolves a ticket

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
| `F-CRM-016-02.enabled` | Boolean | deploy | true |
| `F-CRM-016-02.readOnly` | Boolean | runtime | false |

**Tree position:** `CRM_UI` → `F-CRM-016` → `F-CRM-016-02`

---

## 1. User Goal & Scenarios

### 1.1 User Goal

Support Agent works on and resolves a ticket

### 1.2 Scenarios

**Scenario 1: Work on ticket, add internal comment**
- **Outcome:** Write comment, select internal visibility
- **Notes:** Comment added, not visible to customer

**Scenario 2: Resolve ticket**
- **Outcome:** Fill resolution form, click Resolve
- **Notes:** TicketResolved event, SLA clock stops

---

## 2. User Journey & Screen Layout

### 2.1 Happy Path Sequence

```mermaid
sequenceDiagram
    actor User
    participant BFF as CRM BFF
    participant SVC as crm.sup
    User->>BFF: Navigate to Ticket Detail & Resolution
    BFF->>SVC: GET/PUT /tickets/{id}
    SVC-->>BFF: 200 OK (data)
    BFF-->>User: Render Ticket Detail & Resolution
```

### 2.2 Screen Layout

```
┌─────────────────────────────────────────────────┐
│ [F-CRM-016-02] Ticket Detail & Resolution                │
├─────────────────────────────────────────────────┤
│ [display ] Subject, priority, status, category, SLA co │
│ [timeline] Public and internal comments in chronologic │
│ [form    ] Resolution type, resolution notes, mark as  │
│ [list    ] Suggested knowledge base articles based on  │
│ [toolbar ] Assign, Escalate, Resolve, Reopen, Merge │
├─────────────────────────────────────────────────┤
│ [EXT] Extension zone                            │
└─────────────────────────────────────────────────┘
```

---

## 3. Interaction Requirements

### 3.1 Zones

| Zone | Type | Priority | Description |
|------|------|----------|-------------|
| `ticketHeader` | `display` | high | Subject, priority, status, category, SLA countdown, contact info |
| `ticketConversation` | `timeline` | high | Public and internal comments in chronological order |
| `ticketResolutionForm` | `form` | high | Resolution type, resolution notes, mark as resolved |
| `relatedArticles` | `list` | high | Suggested knowledge base articles based on ticket content |
| `ticketActionBar` | `toolbar` | high | Assign, Escalate, Resolve, Reopen, Merge |
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
| Read-only | `F-CRM-016-02.readOnly` = true | All edit controls disabled, view-only mode |

---

## 5. Backend Dependencies & BFF Contract

### 5.1 Service Calls

| Service | Endpoint | Tier | Is Mutation | Failure Mode |
|---------|----------|------|------------|-------------|
| `crm.sup` | `GET/PUT /tickets/{id}` | core | Yes | Show error toast, allow retry |

### 5.2 Feature Gating

| Mode | `F-CRM-016-02.enabled` | `F-CRM-016-02.readOnly` | Behaviour |
|------|------------------------|--------------------------|-----------|
| Full | true | false | All features available |
| Read-only | true | true | View only, mutations disabled |
| Excluded | false | — | Feature hidden from navigation |

---

## 6. Screen Contract (AUI)

See companion file: `contracts/aui/F-CRM-016-02.aui.yaml`

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

**AC-1: Work on ticket, add internal comment**
- **Given** a user with appropriate permissions
- **When** work on ticket, add internal comment
- **Then** write comment, select internal visibility

**AC-2: Resolve ticket**
- **Given** a user with appropriate permissions
- **When** resolve ticket
- **Then** fill resolution form, click resolve

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