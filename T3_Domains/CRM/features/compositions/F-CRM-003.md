# F-CRM-003 — Opportunity Management

> **Template:** `feature-composition-spec.md` v1.0.0
> **Status:** DRAFT
> **Suite:** `crm`
> **Node type:** COMPOSITION
> **Parent:** `CRM_UI`
> **Primary service:** `crm.opp`
> **Companion UVL:** `F-CRM-003.uvl`

---

## SS0 — Identity

**Purpose:** Groups all UI features related to opportunity management within the CRM Suite.

### Children

| Feature ID | Name | Node Type | Status |
|-----------|------|-----------|--------|
| `F-CRM-003-01` | Opportunity List & Kanban | LEAF | `draft` |
| `F-CRM-003-02` | Opportunity Detail View | LEAF | `draft` |
| `F-CRM-003-03` | Opportunity Create/Edit Form | LEAF | `draft` |
| `F-CRM-003-04` | Opportunity Line Items | LEAF | `draft` |
| `F-CRM-003-05` | Stage Progression & Guidance | LEAF | `draft` |
| `F-CRM-003-06` | Win/Loss Analysis Dashboard | LEAF | `draft` |

**Position in tree:** `CRM_UI` → `F-CRM-003` → 6 leaf features

---

## SS1 — Children & Variability Structure

**Group type:** `mandatory` — all children are required when F-CRM-003 is selected.

**Business rationale:** Opportunity Management is a core CRM capability. All leaf features within this composition represent essential screens that users need for complete opportunity management workflows. Removing any single screen would break the user journey.

### Variability Points

| Leaf | Binding Time | Rationale |
|------|-------------|-----------|
| `F-CRM-003-01` | `compile` | Always included |
| `F-CRM-003-02` | `compile` | Always included |
| `F-CRM-003-03` | `compile` | Always included |
| `F-CRM-003-04` | `compile` | Always included |
| `F-CRM-003-05` | `compile` | Always included |
| `F-CRM-003-06` | `compile` | Always included |

---

## SS2 — Constraints

### Intra-Suite Dependencies

- `F-CRM-003-04` (Line Items) `requires` `F-CRM-005` (Product & Pricing)

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