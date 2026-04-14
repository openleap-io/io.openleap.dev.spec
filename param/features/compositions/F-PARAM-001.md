# F-PARAM-001 — Reference Data Management

> **Meta Information**
> - **Version:** 2026-04-03
> - **Template:** `feature-composition-spec.md` v1.0.0
> - **Template Compliance:** 100%
> - **Feature ID:** `F-PARAM-001`
> - **Suite:** `param`
> - **Node type:** COMPOSITION
> - **Companion UVL:** `uvl/compositions/F-PARAM-001.uvl`

---

## SS0. Identity

### 0.1 Purpose
This composition node groups all capabilities for managing reference data catalogs and their code items. A platform operator uses these features to browse, create, update, and bulk-manage the controlled vocabularies that all domain services depend on for code validation.

### 0.2 Children
| Child ID | Name | Node type | Group membership |
|---|---|---|---|
| `F-PARAM-001-01` | Browse Catalogs | LEAF | mandatory |
| `F-PARAM-001-02` | Manage Catalog Codes | LEAF | mandatory |
| `F-PARAM-001-03` | Bulk Import/Export | LEAF | optional |

### 0.3 Position in Feature Tree
```
PARAM Suite  [ROOT]
+-- F-PARAM-001  Reference Data Management  [COMPOSITION] <-- you are here
    +-- F-PARAM-001-01  Browse Catalogs  [LEAF]
    +-- F-PARAM-001-02  Manage Catalog Codes  [LEAF]
    +-- F-PARAM-001-03  Bulk Import/Export  [LEAF]
```

---

## SS1. Children & Variability Structure

### 1.1 Group Logic
**Group type:** mixed

**Business rationale:** Browse and Manage are mandatory because any deployment needs the ability to view and edit reference codes. Bulk Import/Export is optional because small deployments may manage codes manually through the UI, while enterprise deployments need CSV/JSON bulk operations for initial data loading and migration.

### 1.2 Sub-Groups

| Sub-group | Children | Group type | Rationale |
|---|---|---|---|
| Core management | `001-01`, `001-02` | mandatory | Minimum viable reference data administration |
| Bulk operations | `001-03` | optional | Only needed for large-scale or migration scenarios |

### 1.3 Rationale for Tree Position
Reference Data Management exists as a distinct composition because ref-svc is the upstream dependency for both i18n-svc and cfg-svc. Managing catalogs and codes is a self-contained administrative workflow that does not overlap with translation or configuration management.

---

## SS2. Constraints

### 2.1 Intra-Node Constraints
| If | Then | Rationale |
|---|---|---|
| `F-PARAM-001-03` is included | `F-PARAM-001-01` must be included | Bulk import results need to be viewable in the browse UI |

### 2.2 Cross-Node Constraints
| If | Then | Rationale |
|---|---|---|
| `F-PARAM-002-02` (Edit Translations) is included | `F-PARAM-001-01` must be included | Editing translations requires browsing catalogs to find the correct namespace |

---

## SS3. Quality & Testing Guidance
All leaf features MUST be independently testable. Integration tests MUST verify that creating a new code in F-PARAM-001-02 makes it immediately available in F-PARAM-001-01 browse results.

---

## SS4. Change Log
| Date | Version | Author | Changes |
|------|---------|--------|---------|
| 2026-04-03 | 1.0.0 | Architecture Team | Initial composition node |

---
**END OF SPECIFICATION**
