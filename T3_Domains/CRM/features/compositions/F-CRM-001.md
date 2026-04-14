# F-CRM-001 — Contact & Account Management

> **Template:** `feature-composition-spec.md` v1.0.0
> **Status:** DRAFT
> **Suite:** `crm`
> **Node type:** COMPOSITION
> **Parent:** `CRM_UI`
> **Primary service:** `crm.contact`
> **Companion UVL:** `F-CRM-001.uvl`

---

## SS0 — Identity

**Purpose:** Groups all UI features related to contact & account management within the CRM Suite.

### Children

| Feature ID | Name | Node Type | Status |
|-----------|------|-----------|--------|
| `F-CRM-001-01` | Contact List & Search | LEAF | `draft` |
| `F-CRM-001-02` | Contact Detail View | LEAF | `draft` |
| `F-CRM-001-03` | Contact Create/Edit Form | LEAF | `draft` |
| `F-CRM-001-04` | Account List & Search | LEAF | `draft` |
| `F-CRM-001-05` | Account Detail View | LEAF | `draft` |
| `F-CRM-001-06` | Account Create/Edit Form | LEAF | `draft` |
| `F-CRM-001-07` | Relationship Management | LEAF | `draft` |
| `F-CRM-001-08` | Duplicate Detection & Merge | LEAF | `draft` |

**Position in tree:** `CRM_UI` → `F-CRM-001` → 8 leaf features

---

## SS1 — Children & Variability Structure

**Group type:** `mandatory` — all children are required when F-CRM-001 is selected.

**Business rationale:** Contact & Account Management is a core CRM capability. All leaf features within this composition represent essential screens that users need for complete contact & account management workflows. Removing any single screen would break the user journey.

### Variability Points

| Leaf | Binding Time | Rationale |
|------|-------------|-----------|
| `F-CRM-001-01` | `compile` | Always included |
| `F-CRM-001-02` | `compile` | Always included |
| `F-CRM-001-03` | `compile` | Always included |
| `F-CRM-001-04` | `compile` | Always included |
| `F-CRM-001-05` | `compile` | Always included |
| `F-CRM-001-06` | `compile` | Always included |
| `F-CRM-001-07` | `compile` | Always included |
| `F-CRM-001-08` | `compile` | Always included |

---

## SS2 — Constraints

### Intra-Suite Dependencies

No additional intra-suite constraints beyond those defined at the suite catalog level.

### Cross-Suite Dependencies

- All CRM features require `F-IAM-001` (Authentication) — users must be authenticated.
- `F-CRM-001` requires `F-BP-001` (Business Partner Registry) — for master data sync.

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