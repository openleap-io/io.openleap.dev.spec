# F-PPS-002 — Shop Floor Execution

> **Meta Information**
> - **Version:** 2026-04-04
> - **Template:** `feature-composition-spec.md` v1.0.0
> - **Template Compliance:** 100%
> - **Feature ID:** `F-PPS-002`
> - **Suite:** `pps`
> - **Node type:** COMPOSITION
> - **Companion UVL:** `uvl/compositions/F-PPS-002.uvl`

---

## SS0. Identity

### 0.1 Purpose
This composition node groups all capabilities for executing production on the shop floor. Operators and supervisors use these features to manage and confirm work orders, record manufacturing execution data, and perform quality inspections on production lots.

### 0.2 Children
| Child ID | Name | Node type | Group membership |
|---|---|---|---|
| `F-PPS-002-01` | Work Order Management | LEAF | mandatory |
| `F-PPS-002-02` | MES Recording | LEAF | mandatory |
| `F-PPS-002-03` | Quality Inspection | LEAF | optional |

### 0.3 Position in Feature Tree
```
PPS Suite  [ROOT]
+-- F-PPS-002  Shop Floor Execution  [COMPOSITION] <-- you are here
    +-- F-PPS-002-01  Work Order Management  [LEAF]
    +-- F-PPS-002-02  MES Recording  [LEAF]
    +-- F-PPS-002-03  Quality Inspection  [LEAF]
```

---

## SS1. Children & Variability Structure

### 1.1 Group Logic
**Group type:** mixed

**Business rationale:** Work Order Management and MES Recording are mandatory because any shop floor operation requires order confirmation and production data capture. Quality Inspection is optional because some production environments perform quality checks through a dedicated QM system rather than inline within the MES workflow.

### 1.2 Sub-Groups

| Sub-group | Children | Group type | Rationale |
|---|---|---|---|
| Core execution | `002-01`, `002-02` | mandatory | Minimum viable shop floor execution |
| Quality management | `002-03` | optional | Only needed where QM is handled within PPS |

### 1.3 Rationale for Tree Position
Shop Floor Execution depends on Production Planning Core (F-PPS-001) at the cross-node level. Execution features consume production orders created by planning. This composition is intentionally separate from Inventory & Warehouse (F-PPS-003) so that goods movements can be selectively enabled independently of shop floor recording.

---

## SS2. Constraints

### 2.1 Intra-Node Constraints
| If | Then | Rationale |
|---|---|---|
| `F-PPS-002-02` is included | `F-PPS-002-01` must be included | MES recording is always associated with an active work order |
| `F-PPS-002-03` is included | `F-PPS-002-01` must be included | Quality inspection lots are generated from work order operations |

### 2.2 Cross-Node Constraints
| If | Then | Rationale |
|---|---|---|
| `F-PPS-002-01` (Work Order Management) is included | `F-PPS-001-02` must be included | Work orders reference production orders from the planning domain |

---

## SS3. Quality & Testing Guidance
All leaf features MUST be independently testable. Integration tests MUST verify that confirming a work order operation in F-PPS-002-01 makes recording available in F-PPS-002-02. Where F-PPS-002-03 is enabled, completing a work order MUST trigger inspection lot creation.

---

## SS4. Change Log
| Date | Version | Author | Changes |
|------|---------|--------|---------|
| 2026-04-04 | 1.0.0 | Architecture Team | Initial composition node |

---
**END OF SPECIFICATION**
