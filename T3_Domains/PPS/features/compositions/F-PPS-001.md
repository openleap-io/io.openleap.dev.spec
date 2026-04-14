# F-PPS-001 — Production Planning Core

> **Meta Information**
> - **Version:** 2026-04-04
> - **Template:** `feature-composition-spec.md` v1.0.0
> - **Template Compliance:** 100%
> - **Feature ID:** `F-PPS-001`
> - **Suite:** `pps`
> - **Node type:** COMPOSITION
> - **Companion UVL:** `uvl/compositions/F-PPS-001.uvl`

---

## SS0. Identity

### 0.1 Purpose
This composition node groups all capabilities for managing production planning at the suite level. A production planner uses these features to execute MRP runs, browse and track production orders, and analyse work center capacity to identify and resolve bottlenecks before they impact the shop floor.

### 0.2 Children
| Child ID | Name | Node type | Group membership |
|---|---|---|---|
| `F-PPS-001-01` | MRP Run | LEAF | mandatory |
| `F-PPS-001-02` | Production Order Browse | LEAF | mandatory |
| `F-PPS-001-03` | Capacity Planning | LEAF | optional |

### 0.3 Position in Feature Tree
```
PPS Suite  [ROOT]
+-- F-PPS-001  Production Planning Core  [COMPOSITION] <-- you are here
    +-- F-PPS-001-01  MRP Run  [LEAF]
    +-- F-PPS-001-02  Production Order Browse  [LEAF]
    +-- F-PPS-001-03  Capacity Planning  [LEAF]
```

---

## SS1. Children & Variability Structure

### 1.1 Group Logic
**Group type:** mixed

**Business rationale:** MRP Run and Production Order Browse are mandatory because any PPS deployment needs the ability to trigger planning runs and view resulting production orders. Capacity Planning is optional because smaller operations may not have formal work-centre capacity constraints and can rely on manual scheduling.

### 1.2 Sub-Groups

| Sub-group | Children | Group type | Rationale |
|---|---|---|---|
| Core planning | `001-01`, `001-02` | mandatory | Minimum viable production planning |
| Advanced planning | `001-03` | optional | Only needed for constrained-capacity environments |

### 1.3 Rationale for Tree Position
Production Planning Core is the root composition for all planning-side features. It sits above Shop Floor Execution (F-PPS-002) because planning must exist before execution can begin. It is independent of Inventory & Warehouse (F-PPS-003) at the feature-model level, though runtime constraints apply.

---

## SS2. Constraints

### 2.1 Intra-Node Constraints
| If | Then | Rationale |
|---|---|---|
| `F-PPS-001-03` is included | `F-PPS-001-02` must be included | Capacity planning results reference production orders that must be browsable |

### 2.2 Cross-Node Constraints
| If | Then | Rationale |
|---|---|---|
| `F-PPS-002-01` (Work Order Management) is included | `F-PPS-001-02` must be included | Work orders are derived from production orders; browse access is a prerequisite |

---

## SS3. Quality & Testing Guidance
All leaf features MUST be independently testable. Integration tests MUST verify that completing an MRP run (F-PPS-001-01) produces production orders that immediately appear in the browse list (F-PPS-001-02). Capacity Planning (F-PPS-001-03) MUST reflect load changes within one planning cycle after order updates.

---

## SS4. Change Log
| Date | Version | Author | Changes |
|------|---------|--------|---------|
| 2026-04-04 | 1.0.0 | Architecture Team | Initial composition node |

---
**END OF SPECIFICATION**
