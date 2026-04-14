# F-CRM-002 — Lead Management

> **Template:** `feature-composition-spec.md` v1.0.0
> **Status:** DRAFT
> **Suite:** `crm`
> **Node type:** COMPOSITION
> **Parent:** `CRM_UI`
> **Primary service:** `crm.lead`
> **Companion UVL:** `F-CRM-002.uvl`

---

## SS0 — Identity

**Purpose:** Groups all UI features related to lead management within the CRM Suite.

### Children

| Feature ID | Name | Node Type | Status |
|-----------|------|-----------|--------|
| `F-CRM-002-01` | Lead List & Pipeline View | LEAF | `draft` |
| `F-CRM-002-02` | Lead Detail View | LEAF | `draft` |
| `F-CRM-002-03` | Lead Create/Edit Form | LEAF | `draft` |
| `F-CRM-002-04` | Lead Qualification & Scoring | LEAF | `draft` |
| `F-CRM-002-05` | Lead Conversion Wizard | LEAF | `draft` |
| `F-CRM-002-06` | Lead Import | LEAF | `draft` |

**Position in tree:** `CRM_UI` → `F-CRM-002` → 6 leaf features

---

## SS1 — Children & Variability Structure

**Group type:** `mandatory` — all children are required when F-CRM-002 is selected.

**Business rationale:** Lead Management is a core CRM capability. All leaf features within this composition represent essential screens that users need for complete lead management workflows. Removing any single screen would break the user journey.

### Variability Points

| Leaf | Binding Time | Rationale |
|------|-------------|-----------|
| `F-CRM-002-01` | `compile` | Always included |
| `F-CRM-002-02` | `compile` | Always included |
| `F-CRM-002-03` | `compile` | Always included |
| `F-CRM-002-04` | `compile` | Always included |
| `F-CRM-002-05` | `compile` | Always included |
| `F-CRM-002-06` | `compile` | Always included |

---

## SS2 — Constraints

### Intra-Suite Dependencies

No additional intra-suite constraints beyond those defined at the suite catalog level.

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