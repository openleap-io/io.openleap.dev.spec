# F-OPS-001 — Work Order Management

> **Meta Information**
> - **Version:** 2026-04-04
> - **Template:** `feature-composition-spec.md` v1.0.0
> - **Template Compliance:** 100%
> - **Feature ID:** `F-OPS-001`
> - **Suite:** `ops`
> - **Node type:** COMPOSITION
> - **Companion UVL:** `uvl/compositions/F-OPS-001.uvl`

---

## SS0. Identity

### 0.1 Purpose
This composition node groups all capabilities for managing work orders in the field — from creation through execution to closure and reporting. A dispatcher, operations manager, or field technician uses these features to initiate, perform, and document operational tasks.

### 0.2 Children
| Child ID | Name | Node type | Group membership |
|---|---|---|---|
| `F-OPS-001-01` | Create Work Order | LEAF | mandatory |
| `F-OPS-001-02` | Work Order Execution | LEAF | mandatory |
| `F-OPS-001-03` | Work Order Close & Report | LEAF | optional |

### 0.3 Position in Feature Tree
```
OPS Suite  [ROOT]
+-- F-OPS-001  Work Order Management  [COMPOSITION] <-- you are here
    +-- F-OPS-001-01  Create Work Order  [LEAF]
    +-- F-OPS-001-02  Work Order Execution  [LEAF]
    +-- F-OPS-001-03  Work Order Close & Report  [LEAF]
```

---

## SS1. Children & Variability Structure

### 1.1 Group Logic
**Group type:** mixed

**Business rationale:** Create and Execute are mandatory because any deployment of operational services must be able to originate and carry out work. Close & Report (which generates a digital service document and optionally captures customer signature) is optional because lightweight organizations may use paper-based job completion or do not require digital PDF generation.

### 1.2 Sub-Groups

| Sub-group | Children | Group type | Rationale |
|---|---|---|---|
| Core work order lifecycle | `001-01`, `001-02` | mandatory | Minimum viable field service operation |
| Completion & documentation | `001-03` | optional | Required only when digital job completion reports are needed |

### 1.3 Rationale for Tree Position
Work Order Management is the central composition of the OPS suite. All resource scheduling (F-OPS-002) and time tracking (F-OPS-003) capabilities ultimately support work orders. Keeping create, execute, and close in one composition reflects the natural lifecycle of a single work order from inception to completion.

---

## SS2. Constraints

### 2.1 Intra-Node Constraints
| If | Then | Rationale |
|---|---|---|
| `F-OPS-001-03` is included | `F-OPS-001-02` must be included | Cannot close a work order without first executing it |

### 2.2 Cross-Node Constraints
| If | Then | Rationale |
|---|---|---|
| `F-OPS-002-02` (Schedule Assignment) is included | `F-OPS-001-01` must be included | Scheduling requires work orders to assign resources to |

---

## SS3. Quality & Testing Guidance
All leaf features MUST be independently testable. Integration tests MUST verify that a work order created via F-OPS-001-01 appears in the execution view of F-OPS-001-02, and that closing via F-OPS-001-03 transitions the work order to status `CLOSED`.

---

## SS4. Change Log
| Date | Version | Author | Changes |
|------|---------|--------|---------|
| 2026-04-04 | 1.0.0 | Architecture Team | Initial composition node |

---
**END OF SPECIFICATION**
