# F-CRM-013-04 — Integration Logs & Monitor

> **Template:** `feature-spec.md` v1.0.0
> **Template Compliance:** 95%+
> **Status:** DRAFT
> **Feature ID:** `F-CRM-013-04`
> **Suite:** `crm`
> **Node type:** LEAF
> **Parent:** `F-CRM-013`
> **Companion UVL:** `F-CRM-013-04.uvl`
> **Companion AUI:** `F-CRM-013-04.aui.yaml`

---

## 0. Feature Identity & Orientation

### 0.1 One-Line Summary

This feature lets a CRM user cRM Admin monitors integration health and troubleshoots sync issues

### 0.2 Non-Goals

- This feature does NOT implement backend business logic (see domain spec for `crm.int`).
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
| `F-CRM-013-04.enabled` | Boolean | deploy | true |
| `F-CRM-013-04.readOnly` | Boolean | runtime | false |

**Tree position:** `CRM_UI` → `F-CRM-013` → `F-CRM-013-04`

---

## 1. User Goal & Scenarios

### 1.1 User Goal

CRM Admin monitors integration health and troubleshoots sync issues

### 1.2 Scenarios

**Scenario 1: View sync logs for last 24 hours**
- **Outcome:** Table sorted by timestamp, errors highlighted
- **Notes:** Click error row for details

**Scenario 2: Retry failed Mailchimp sync**
- **Outcome:** Click retry, sync re-executes
- **Notes:** Success/failure result shown

---

## 2. User Journey & Screen Layout

### 2.1 Happy Path Sequence

```mermaid
sequenceDiagram
    actor User
    participant BFF as CRM BFF
    participant SVC as crm.int
    User->>BFF: Navigate to Integration Logs & Monitor
    BFF->>SVC: GET /sync-logs
    SVC-->>BFF: 200 OK (data)
    BFF-->>User: Render Integration Logs & Monitor
```

### 2.2 Screen Layout

```
┌─────────────────────────────────────────────────┐
│ [F-CRM-013-04] Integration Logs & Monitor                │
├─────────────────────────────────────────────────┤
│ [table   ] Logs: integration, operation, status, recor │
│ [display ] Error message, stack trace, affected record │
│ [button  ] Retry failed sync operations           │
├─────────────────────────────────────────────────┤
│ [EXT] Extension zone                            │
└─────────────────────────────────────────────────┘
```

---

## 3. Interaction Requirements

### 3.1 Zones

| Zone | Type | Priority | Description |
|------|------|----------|-------------|
| `syncLogTable` | `table` | high | Logs: integration, operation, status, records processed, timestamp, duration |
| `errorDetails` | `display` | high | Error message, stack trace, affected records |
| `retryButton` | `button` | high | Retry failed sync operations |
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
| Read-only | `F-CRM-013-04.readOnly` = true | All edit controls disabled, view-only mode |

---

## 5. Backend Dependencies & BFF Contract

### 5.1 Service Calls

| Service | Endpoint | Tier | Is Mutation | Failure Mode |
|---------|----------|------|------------|-------------|
| `crm.int` | `GET /sync-logs` | core | No | Show error toast, allow retry |

### 5.2 Feature Gating

| Mode | `F-CRM-013-04.enabled` | `F-CRM-013-04.readOnly` | Behaviour |
|------|------------------------|--------------------------|-----------|
| Full | true | false | All features available |
| Read-only | true | true | View only, mutations disabled |
| Excluded | false | — | Feature hidden from navigation |

---

## 6. Screen Contract (AUI)

See companion file: `contracts/aui/F-CRM-013-04.aui.yaml`

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

**AC-1: View sync logs for last 24 hours**
- **Given** a user with appropriate permissions
- **When** view sync logs for last 24 hours
- **Then** table sorted by timestamp, errors highlighted

**AC-2: Retry failed Mailchimp sync**
- **Given** a user with appropriate permissions
- **When** retry failed mailchimp sync
- **Then** click retry, sync re-executes

---

## 9. Dependencies, Variability & Extension Points

### 9.1 Feature Dependencies

- Requires parent composition `F-CRM-013` to be selected
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