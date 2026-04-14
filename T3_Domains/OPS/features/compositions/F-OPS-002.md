# F-OPS-002 — Resource & Scheduling

> **Meta Information**
> - **Version:** 2026-04-04
> - **Template:** `feature-composition-spec.md` v1.0.0
> - **Template Compliance:** 100%
> - **Feature ID:** `F-OPS-002`
> - **Suite:** `ops`
> - **Node type:** COMPOSITION
> - **Companion UVL:** `uvl/compositions/F-OPS-002.uvl`

---

## SS0. Identity

### 0.1 Purpose
This composition node groups all capabilities for managing resource availability, assigning resources to work orders on a schedule board, and reporting on utilization. A resource planner or dispatcher uses these features to ensure the right people and assets are in the right place at the right time.

### 0.2 Children
| Child ID | Name | Node type | Group membership |
|---|---|---|---|
| `F-OPS-002-01` | Resource Availability | LEAF | mandatory |
| `F-OPS-002-02` | Schedule Assignment | LEAF | mandatory |
| `F-OPS-002-03` | Utilization Report | LEAF | optional |

### 0.3 Position in Feature Tree
```
OPS Suite  [ROOT]
+-- F-OPS-002  Resource & Scheduling  [COMPOSITION] <-- you are here
    +-- F-OPS-002-01  Resource Availability  [LEAF]
    +-- F-OPS-002-02  Schedule Assignment  [LEAF]
    +-- F-OPS-002-03  Utilization Report  [LEAF]
```

---

## SS1. Children & Variability Structure

### 1.1 Group Logic
**Group type:** mixed

**Business rationale:** Resource Availability and Schedule Assignment are mandatory because planning without visibility into who is free is impossible. Utilization Report is optional because small teams may track utilization informally or via external BI tools, while larger operations teams need the built-in view.

### 1.2 Sub-Groups

| Sub-group | Children | Group type | Rationale |
|---|---|---|---|
| Core planning | `002-01`, `002-02` | mandatory | Minimum viable resource scheduling |
| Analytics | `002-03` | optional | Needed for capacity management and performance review |

### 1.3 Rationale for Tree Position
Resource & Scheduling is a supporting composition for Work Order Management (F-OPS-001). It has a cross-node dependency: F-OPS-002-02 requires F-OPS-001-01 because you can only assign resources to existing work orders.

---

## SS2. Constraints

### 2.1 Intra-Node Constraints
| If | Then | Rationale |
|---|---|---|
| `F-OPS-002-02` is included | `F-OPS-002-01` must be included | Schedule assignment requires visibility into resource availability |
| `F-OPS-002-03` is included | `F-OPS-002-01` must be included | Utilization report is derived from availability and assignment data |

### 2.2 Cross-Node Constraints
| If | Then | Rationale |
|---|---|---|
| `F-OPS-002-02` is included | `F-OPS-001-01` must be included | Cannot schedule resources to work orders that don't exist |

---

## SS3. Quality & Testing Guidance
Integration tests MUST verify that blocking a resource in F-OPS-002-01 prevents their selection in F-OPS-002-02 for the same time window. Utilization figures in F-OPS-002-03 MUST reconcile with raw assignment data from F-OPS-002-02.

---

## SS4. Change Log
| Date | Version | Author | Changes |
|------|---------|--------|---------|
| 2026-04-04 | 1.0.0 | Architecture Team | Initial composition node |

---
**END OF SPECIFICATION**
