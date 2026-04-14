# F-CRM-017-02 — Quote Builder

> **Template:** `feature-spec.md` v1.0.0
> **Template Compliance:** 95%+
> **Status:** DRAFT
> **Feature ID:** `F-CRM-017-02`
> **Suite:** `crm`
> **Node type:** LEAF
> **Parent:** `F-CRM-017`
> **Companion UVL:** `F-CRM-017-02.uvl`
> **Companion AUI:** `F-CRM-017-02.aui.yaml`

---

## 0. Feature Identity & Orientation

### 0.1 One-Line Summary

This feature lets a CRM user sales Rep builds a quote with line items, pricing, and discounts

### 0.2 Non-Goals

- This feature does NOT implement backend business logic (see domain spec for `crm.cpq`).
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
| `F-CRM-017-02.enabled` | Boolean | deploy | true |
| `F-CRM-017-02.readOnly` | Boolean | runtime | false |

**Tree position:** `CRM_UI` → `F-CRM-017` → `F-CRM-017-02`

---

## 1. User Goal & Scenarios

### 1.1 User Goal

Sales Rep builds a quote with line items, pricing, and discounts

### 1.2 Scenarios

**Scenario 1: Build quote from opportunity**
- **Outcome:** Link to opportunity, add products from catalog
- **Notes:** Prices pulled from default price book

**Scenario 2: Add custom discount to line item**
- **Outcome:** Enter discount percentage or amount
- **Notes:** Grand total recalculates, approval may be required

---

## 2. User Journey & Screen Layout

### 2.1 Happy Path Sequence

```mermaid
sequenceDiagram
    actor User
    participant BFF as CRM BFF
    participant SVC as crm.cpq
    User->>BFF: Navigate to Quote Builder
    BFF->>SVC: POST/PUT /quotes
    SVC-->>BFF: 200 OK (data)
    BFF-->>User: Render Quote Builder
```

### 2.2 Screen Layout

```
┌─────────────────────────────────────────────────┐
│ [F-CRM-017-02] Quote Builder                             │
├─────────────────────────────────────────────────┤
│ [form    ] Opportunity (lookup), account (auto), valid │
│ [table   ] Editable table: product search, quantity, u │
│ [display ] Subtotal, total discount, tax, grand total │
│ [display ] Auto-applied discount rules with manual ove │
├─────────────────────────────────────────────────┤
│ [EXT] Extension zone                            │
└─────────────────────────────────────────────────┘
```

---

## 3. Interaction Requirements

### 3.1 Zones

| Zone | Type | Priority | Description |
|------|------|----------|-------------|
| `quoteHeader` | `form` | high | Opportunity (lookup), account (auto), valid until, notes, terms |
| `lineItemEditor` | `table` | high | Editable table: product search, quantity, unit price (from price book), discount, line total |
| `pricingSummary` | `display` | high | Subtotal, total discount, tax, grand total |
| `discountApplicator` | `display` | high | Auto-applied discount rules with manual override option |
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
| Read-only | `F-CRM-017-02.readOnly` = true | All edit controls disabled, view-only mode |

---

## 5. Backend Dependencies & BFF Contract

### 5.1 Service Calls

| Service | Endpoint | Tier | Is Mutation | Failure Mode |
|---------|----------|------|------------|-------------|
| `crm.cpq` | `POST/PUT /quotes` | core | Yes | Show error toast, allow retry |

### 5.2 Feature Gating

| Mode | `F-CRM-017-02.enabled` | `F-CRM-017-02.readOnly` | Behaviour |
|------|------------------------|--------------------------|-----------|
| Full | true | false | All features available |
| Read-only | true | true | View only, mutations disabled |
| Excluded | false | — | Feature hidden from navigation |

---

## 6. Screen Contract (AUI)

See companion file: `contracts/aui/F-CRM-017-02.aui.yaml`

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

**AC-1: Build quote from opportunity**
- **Given** a user with appropriate permissions
- **When** build quote from opportunity
- **Then** link to opportunity, add products from catalog

**AC-2: Add custom discount to line item**
- **Given** a user with appropriate permissions
- **When** add custom discount to line item
- **Then** enter discount percentage or amount

---

## 9. Dependencies, Variability & Extension Points

### 9.1 Feature Dependencies

- Requires parent composition `F-CRM-017` to be selected
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