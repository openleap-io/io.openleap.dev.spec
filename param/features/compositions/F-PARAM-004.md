# F-PARAM-004 — Unit Management

> **Meta Information**
> - **Version:** 2026-04-03
> - **Template:** `feature-composition-spec.md` v1.0.0
> - **Template Compliance:** 100%
> - **Feature ID:** `F-PARAM-004`
> - **Suite:** `param`
> - **Node type:** COMPOSITION
> - **Companion UVL:** `uvl/compositions/F-PARAM-004.uvl`

---

## SS0. Identity

### 0.1 Purpose
This composition node groups all capabilities for managing SI units of measure, prefixes, and custom unit definitions. Platform administrators and domain engineers use these features to browse the unit catalog, register custom units, and perform interactive unit conversions.

### 0.2 Children
| Child ID | Name | Node type | Group membership |
|---|---|---|---|
| `F-PARAM-004-01` | Browse Units & Prefixes | LEAF | mandatory |
| `F-PARAM-004-02` | Manage Custom Units | LEAF | optional |
| `F-PARAM-004-03` | Unit Conversion Tool | LEAF | optional |

### 0.3 Position in Feature Tree
```
PARAM Suite  [ROOT]
+-- F-PARAM-004  Unit Management  [COMPOSITION] <-- you are here
    +-- F-PARAM-004-01  Browse Units & Prefixes  [LEAF]
    +-- F-PARAM-004-02  Manage Custom Units  [LEAF]
    +-- F-PARAM-004-03  Unit Conversion Tool  [LEAF]
```

---

## SS1. Children & Variability Structure

### 1.1 Group Logic
**Group type:** mixed

**Business rationale:** If the Unit Management composition is included, Browse is mandatory because operators need to see the unit catalog. Custom Units is optional — only deployments with non-SI units (e.g., imperial, industry-specific) need it. Conversion Tool is optional — it is a convenience feature for domain engineers and support staff.

### 1.2 Sub-Groups

| Sub-group | Children | Group type | Rationale |
|---|---|---|---|
| Core browse | `004-01` | mandatory | Minimum viable unit catalog visibility |
| Administration | `004-02` | optional | Only needed when custom units are required |
| Utility | `004-03` | optional | Convenience tool, not required for operations |

### 1.3 Rationale for Tree Position
Unit Management is the only optional top-level composition in the param suite because not all products handle physical quantities. Products focused purely on financial or HR domains may not need unit conversion capabilities.

---

## SS2. Constraints

### 2.1 Intra-Node Constraints
| If | Then | Rationale |
|---|---|---|
| `F-PARAM-004-02` is included | `F-PARAM-004-01` must be included | Custom unit creation results must be visible in browse |

### 2.2 Cross-Node Constraints
None. Unit management has no dependencies on ref, i18n, or cfg features.

---

## SS3. Quality & Testing Guidance
All leaf features MUST be independently testable. Integration tests MUST verify dimensional analysis enforcement: creating a custom unit with specific dimensions and then attempting an incompatible conversion MUST fail.

---

## SS4. Change Log
| Date | Version | Author | Changes |
|------|---------|--------|---------|
| 2026-04-03 | 1.0.0 | Architecture Team | Initial composition node |

---
**END OF SPECIFICATION**
