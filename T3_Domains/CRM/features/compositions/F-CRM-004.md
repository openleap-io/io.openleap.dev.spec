# F-CRM-004 — Activities & Timeline

> **Template:** `feature-composition-spec.md` v1.0.0
> **Status:** DRAFT
> **Suite:** `crm`
> **Node type:** COMPOSITION
> **Parent:** `CRM_UI`
> **Primary service:** `crm.act`
> **Companion UVL:** `F-CRM-004.uvl`

---

## SS0 — Identity

**Purpose:** Groups all UI features related to activities & timeline within the CRM Suite.

### Children

| Feature ID | Name | Node Type | Status |
|-----------|------|-----------|--------|
| `F-CRM-004-01` | Activity Timeline View | LEAF | `draft` |
| `F-CRM-004-02` | Task Management | LEAF | `draft` |
| `F-CRM-004-03` | Meeting Scheduler | LEAF | `draft` |
| `F-CRM-004-04` | Call Logging | LEAF | `draft` |
| `F-CRM-004-05` | Note Editor | LEAF | `draft` |

**Position in tree:** `CRM_UI` → `F-CRM-004` → 5 leaf features

---

## SS1 — Children & Variability Structure

**Group type:** `mandatory` — all children are required when F-CRM-004 is selected.

**Business rationale:** Activities & Timeline is a core CRM capability. All leaf features within this composition represent essential screens that users need for complete activities & timeline workflows. Removing any single screen would break the user journey.

### Variability Points

| Leaf | Binding Time | Rationale |
|------|-------------|-----------|
| `F-CRM-004-01` | `compile` | Always included |
| `F-CRM-004-02` | `compile` | Always included |
| `F-CRM-004-03` | `compile` | Always included |
| `F-CRM-004-04` | `compile` | Always included |
| `F-CRM-004-05` | `compile` | Always included |

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