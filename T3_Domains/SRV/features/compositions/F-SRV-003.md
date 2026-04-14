# F-SRV-003 — Resource Scheduling

> **Conceptual Stack Layer:** Platform-Feature (Composition Node)
> **Suite:** `srv` | **Node type:** COMPOSITION | **Parent:** `SRV_UI`
> **Companion UVL:** `F-SRV-003.uvl`
> **Version:** 2026-04-02 | **Status:** DRAFT
> **Template:** `feature-composition-spec.md` v1.0.0
> **Template Compliance:** ~100% — all sections present

---

## SS0. Identity

### 0.1 Purpose
Groups all capabilities related to **scheduling-optimized resource management**: maintaining scheduling projections of people/rooms/assets, defining availability windows, and detecting booking conflicts. Upstream of booking; downstream of HR/FAC masters.

### 0.2 Children
| Child ID | Name | Node type | Group membership |
|---|---|---|---|
| `F-SRV-003-01` | Resource Management | LEAF | mandatory |
| `F-SRV-003-02` | Availability Management | LEAF | mandatory |
| `F-SRV-003-03` | Conflict Detection | LEAF | mandatory |

### 0.3 Position in the Feature Tree
```
SRV_UI
├── F-SRV-001  Service Catalog Management
├── F-SRV-002  Appointment & Booking
├── F-SRV-003  Resource Scheduling              ← you are here
│   ├── F-SRV-003-01  Resource Management       [mandatory]
│   ├── F-SRV-003-02  Availability Management   [mandatory]
│   └── F-SRV-003-03  Conflict Detection        [mandatory]
...
```

---

## SS1. Children & Variability Structure

### 1.1 Group Logic
**Group type:** mandatory

**Business rationale:** All three children are inseparable. Resources without availability are un-schedulable. Availability without conflict detection leads to overbooking. Conflict detection without resources has nothing to check. These three form an atomic scheduling capability.

---

## SS2. Constraints

### 2.2 Cross-Node Constraints
| Child | Requires | External feature | Rationale |
|---|---|---|---|
| `F-SRV-003-01` | — | `hr` (cross-suite, OPEN QUESTION) | Employee references from HR |
| `F-SRV-003-02` | — | `shared.cal` (cross-suite, OPEN QUESTION) | Working-time calendar rules |

---

## SS3. Valid & Invalid Configurations
| Configuration | Selected | Use case |
|---|---|---|
| Full (only option) | `01`, `02`, `03` | All mandatory — always selected together |

---

## SS4. Change Log
| Date | Version | Author | Changes |
|------|---------|--------|---------|
| 2026-04-02 | 1.0 | OpenLeap Architecture Team | Initial |

**Status:** DRAFT
