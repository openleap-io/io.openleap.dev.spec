# F-PPS-003 — Inventory & Warehouse

> **Meta Information**
> - **Version:** 2026-04-04
> - **Template:** `feature-composition-spec.md` v1.0.0
> - **Template Compliance:** 100%
> - **Feature ID:** `F-PPS-003`
> - **Suite:** `pps`
> - **Node type:** COMPOSITION
> - **Companion UVL:** `uvl/compositions/F-PPS-003.uvl`

---

## SS0. Identity

### 0.1 Purpose
This composition node groups all capabilities for managing inventory and warehouse operations within the PPS suite. Warehouse managers and operators use these features to view current stock levels, post goods movements (receipts, issues, transfers), and conduct periodic physical inventory counts.

### 0.2 Children
| Child ID | Name | Node type | Group membership |
|---|---|---|---|
| `F-PPS-003-01` | Stock Overview | LEAF | mandatory |
| `F-PPS-003-02` | Goods Movement | LEAF | mandatory |
| `F-PPS-003-03` | Physical Inventory | LEAF | optional |

### 0.3 Position in Feature Tree
```
PPS Suite  [ROOT]
+-- F-PPS-003  Inventory & Warehouse  [COMPOSITION] <-- you are here
    +-- F-PPS-003-01  Stock Overview  [LEAF]
    +-- F-PPS-003-02  Goods Movement  [LEAF]
    +-- F-PPS-003-03  Physical Inventory  [LEAF]
```

---

## SS1. Children & Variability Structure

### 1.1 Group Logic
**Group type:** mixed

**Business rationale:** Stock Overview and Goods Movement are mandatory because any manufacturing operation requires visibility into available stock and the ability to post production-related goods movements. Physical Inventory is optional because the count cycle may be managed through a dedicated WMS or periodic offline process.

### 1.2 Sub-Groups

| Sub-group | Children | Group type | Rationale |
|---|---|---|---|
| Core inventory | `003-01`, `003-02` | mandatory | Minimum viable inventory management |
| Periodic counting | `003-03` | optional | Only needed where physical counts are managed within PPS |

### 1.3 Rationale for Tree Position
Inventory & Warehouse is a peer composition to Production Planning Core and Shop Floor Execution. It can be deployed independently to support inventory visibility without activating full planning or execution features. See OQ-PPS-002 for open questions about WMS integration boundaries.

---

## SS2. Constraints

### 2.1 Intra-Node Constraints
| If | Then | Rationale |
|---|---|---|
| `F-PPS-003-03` is included | `F-PPS-003-01` must be included | Physical inventory count differences must be viewable in the stock overview |

### 2.2 Cross-Node Constraints
| If | Then | Rationale |
|---|---|---|
| `F-PPS-002-01` (Work Order Management) is included | `F-PPS-003-02` should be included | Goods issue to production is typically performed alongside work order confirmation |

---

## SS3. Quality & Testing Guidance
All leaf features MUST be independently testable. Integration tests MUST verify that posting a goods movement in F-PPS-003-02 is immediately reflected in the stock overview in F-PPS-003-01. Where F-PPS-003-03 is enabled, posting count differences MUST update stock values visible in F-PPS-003-01.

---

## SS4. Change Log
| Date | Version | Author | Changes |
|------|---------|--------|---------|
| 2026-04-04 | 1.0.0 | Architecture Team | Initial composition node |

---
**END OF SPECIFICATION**
