# F-SRV-001 — Service Catalog Management

> **Conceptual Stack Layer:** Platform-Feature (Composition Node)
> **Space:** Platform
> **Owner:** Domain Engineering Team
> **Companion files:** `F-SRV-001.uvl`
> **Referenced by:** Suite Feature Catalog (`_srv_suite.md` §6)

> **Meta Information**
> - **Version:** 2026-04-02
> - **Author(s):** OpenLeap Architecture Team
> - **Status:** DRAFT
> - **Feature ID:** `F-SRV-001`
> - **Suite:** `srv`
> - **Node type:** COMPOSITION
> - **Parent:** suite root (`SRV_UI`)
> - **Companion UVL:** `F-SRV-001.uvl`
> **Template:** `feature-composition-spec.md` v1.0.0
> **Template Compliance:** ~100% — all sections present

---

## SS0. Identity

### 0.1 Purpose

This composition node groups all capabilities related to **service offering management** — the catalog of services that can be booked, scheduled, and delivered. It is the foundational data layer for the entire SRV suite: every downstream domain (booking, session, billing) references offerings defined here.

### 0.2 Children

| Child ID | Name | Node type | Group membership |
|---|---|---|---|
| `F-SRV-001-01` | Offering CRUD | LEAF | mandatory |
| `F-SRV-001-02` | Offering Lifecycle | LEAF | mandatory |
| `F-SRV-001-03` | Variant & Requirement Mgmt | LEAF | optional |

### 0.3 Position in the Feature Tree

```
SRV_UI                                            [suite root]
├── F-SRV-001  Service Catalog Management         [COMPOSITION] ← you are here
│   ├── F-SRV-001-01  Offering CRUD              [LEAF] [mandatory]
│   ├── F-SRV-001-02  Offering Lifecycle          [LEAF] [mandatory]
│   └── F-SRV-001-03  Variant & Requirement Mgmt  [LEAF] [optional]
├── F-SRV-002  Appointment & Booking              [COMPOSITION]
...
```

---

## SS1. Children & Variability Structure

### 1.1 Group Logic

**Group type:** mixed (mandatory + optional)

**Business rationale for this structure:**

Offering CRUD and Offering Lifecycle are inseparable: you cannot manage an offering without being able to create it, and creation without lifecycle management (activate/deactivate/archive) leaves offerings in an uncontrolled state. These two form the minimum viable catalog.

Variant & Requirement Management is optional because simpler deployments may have only one variant per offering with no formal resource requirements. Complex deployments (e.g., driving schools with multiple license categories, clinics with specialist requirements) need it.

### 1.3 Rationale for Tree Position

Service Catalog is the foundational data domain for SRV. It cannot be merged into Appointment & Booking because catalog management is an administrative concern, not a booking concern. Splitting it further (e.g., separate "Offering" and "Variant" compositions) would fragment naturally co-managed data.

---

## SS2. Constraints

### 2.1 Intra-Node Cross-Tree Constraints

| If | Then | Rationale |
|---|---|---|
| `F-SRV-001-03` (Variants) is included | `F-SRV-001-01` (CRUD) must be included | Variants are children of offerings — need the offering editor |

### 2.2 Cross-Node Constraints

| Child | Requires / Excludes | External feature | Rationale |
|---|---|---|---|
| (none) | — | — | Catalog is upstream; no cross-node requires |

### 2.3 Constraint Summary (UVL Reference)

**Companion file:** `F-SRV-001.uvl`
**Constraint count:** 1 intra-node (implicit) + 0 cross-node
**Last validated:** 2026-04-02

---

## SS3. Valid & Invalid Configurations

### 3.1 Valid Configurations

| Configuration name | Selected leaves | Use case |
|---|---|---|
| Full Catalog | `01`, `02`, `03` | Complex service catalog with variants and requirements |
| Core Catalog | `01`, `02` | Simple catalog — one offering = one service |

### 3.2 Invalid Configurations

| Invalid combination | Violated constraint | Why invalid |
|---|---|---|
| `03` without `01` | mandatory group | Cannot manage variants without offering editor |
| Only `01` without `02` | mandatory group | Offerings without lifecycle control cannot be activated |

---

## SS4. Change Log

### 4.1 Open Questions

| ID | Question | Impact | Owner | Needed by |
|---|---|---|---|---|
| Q-001 | Should pricing hints on offerings be managed here or in `sd`/`com`? | Affects data model of F-SRV-001-01 | TBD | Phase 1 |

### 4.3 Version History

| Date | Version | Author | Changes |
|------|---------|--------|---------|
| 2026-04-02 | 1.0 | OpenLeap Architecture Team | Initial composition node |

---

## Review & Approval

**Status:** DRAFT
