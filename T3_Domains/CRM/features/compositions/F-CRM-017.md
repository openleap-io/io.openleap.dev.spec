# F-CRM-017 — Configure-Price-Quote

> **Template:** `feature-composition-spec.md` v1.0.0
> **Status:** DRAFT
> **Suite:** `crm`
> **Node type:** COMPOSITION
> **Parent:** `CRM_UI`
> **Primary service:** `crm.cpq`
> **Companion UVL:** `F-CRM-017.uvl`

---

## SS0 — Identity

**Purpose:** Groups all UI features related to configure-price-quote within the CRM Suite.

### Children

| Feature ID | Name | Node Type | Status |
|-----------|------|-----------|--------|
| `F-CRM-017-01` | Quote List | LEAF | `draft` |
| `F-CRM-017-02` | Quote Builder | LEAF | `draft` |
| `F-CRM-017-03` | Quote Approval Workflow | LEAF | `draft` |
| `F-CRM-017-04` | Quote PDF Generation | LEAF | `draft` |
| `F-CRM-017-05` | Quote Acceptance Portal | LEAF | `draft` |

**Position in tree:** `CRM_UI` → `F-CRM-017` → 5 leaf features

---

## SS1 — Children & Variability Structure

**Group type:** `optional` — any subset of children may be selected.

**Business rationale:** Configure-Price-Quote is an extension capability. Tenants may enable a subset of features based on their needs and licensing tier. Each leaf feature provides standalone value.

### Variability Points

| Leaf | Binding Time | Rationale |
|------|-------------|-----------|
| `F-CRM-017-01` | `deploy` | Tenant-configurable |
| `F-CRM-017-02` | `deploy` | Tenant-configurable |
| `F-CRM-017-03` | `deploy` | Tenant-configurable |
| `F-CRM-017-04` | `deploy` | Tenant-configurable |
| `F-CRM-017-05` | `deploy` | Tenant-configurable |

---

## SS2 — Constraints

### Intra-Suite Dependencies

- `F-CRM-017` `requires` `F-CRM-005` (Product & Pricing)
- `F-CRM-017` `requires` `F-CRM-003` (Opportunity Management)
- `F-CRM-017` `requires` `F-CRM-011` (Workflow Automation) — for approval workflows

### Cross-Suite Dependencies

- All CRM features require `F-IAM-001` (Authentication) — users must be authenticated.

---

## SS3 — Quality & Testing Guidance

- **Integration tests:** Each leaf feature MUST have at least 3 integration tests covering happy path, validation error, and permission denial.
- **E2E tests:** At least 1 end-to-end test per mainstream scenario defined in each leaf's §1.
- **Accessibility:** All screens MUST meet WCAG 2.1 AA compliance.
- **Performance:** List/search screens MUST render within 2 seconds on standard hardware.

---

## SS4 — Change Log

| Date | Version | Author | Changes |
|------|---------|--------|---------|
| 2026-04-03 | 1.0.0 | OpenLeap Architecture Team | Initial composition spec |

### Open Questions

| # | Question | Status |
|---|----------|--------|
| 1 | Should any children be alternative (mutually exclusive) rather than mandatory/optional? | Open |