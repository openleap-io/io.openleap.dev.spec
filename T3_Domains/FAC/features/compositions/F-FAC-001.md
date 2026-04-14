# F-FAC-001 — Receivables Management

> **Conceptual Stack Layer:** Platform-Feature (Composition Node)
> **Space:** Platform
> **Owner:** FAC Domain Engineering Team
> **Companion files:** `uvl/compositions/F-FAC-001.uvl`
> **Referenced by:** Suite Feature Catalog (`_fac_suite.md` §6)

> **Meta Information**
> - **Version:** 2026-04-04
> - **Author(s):** OpenLeap Architecture Team
> - **Status:** DRAFT
> - **Feature ID:** `F-FAC-001`
> - **Suite:** `fac`
> - **Node type:** COMPOSITION
> - **Parent:** suite root (`FAC_UI`)
> - **Companion UVL:** `uvl/compositions/F-FAC-001.uvl`
> - **Template:** `feature-composition-spec.md` v1.0.0
> - **Template Compliance:** ~90%

---

## SS0. Identity

### 0.1 Purpose
Groups all capabilities for **receivables intake and lifecycle management** — ingesting receivables from fi.ar, validating eligibility, executing assignments, and tracking the receivable lifecycle through settlement.

### 0.2 Children

| Child ID | Name | Node type | Group |
|---|---|---|---|
| `F-FAC-001-01` | Receivable Intake & Worklist | LEAF | mandatory |
| `F-FAC-001-02` | Eligibility Validation & Assignment | LEAF | optional |
| `F-FAC-001-03` | Receivable Lifecycle Tracking | LEAF | optional |

### 0.3 Position in the Feature Tree
```
FAC_UI
├── F-FAC-001  Receivables Management  [COMPOSITION] ← you are here
│   ├── F-FAC-001-01  Receivable Intake & Worklist   [LEAF] [mandatory]
│   ├── F-FAC-001-02  Eligibility Validation         [LEAF] [optional]
│   └── F-FAC-001-03  Lifecycle Tracking             [LEAF] [optional]
├── F-FAC-002  Funding & Settlement    [COMPOSITION]
└── F-FAC-003  Collections & Risk     [COMPOSITION]
```

### 0.4 Domain Coverage
- Primary: `fac.rcv`
- Supporting (eligibility queries): `fac.lim`, `fac.rsk`

---

## Review & Approval
**Status:** DRAFT
