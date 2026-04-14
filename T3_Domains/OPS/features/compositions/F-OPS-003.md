# F-OPS-003 — Time & Project Tracking

> **Meta Information**
> - **Version:** 2026-04-04
> - **Template:** `feature-composition-spec.md` v1.0.0
> - **Template Compliance:** 100%
> - **Feature ID:** `F-OPS-003`
> - **Suite:** `ops`
> - **Node type:** COMPOSITION
> - **Companion UVL:** `uvl/compositions/F-OPS-003.uvl`

---

## SS0. Identity

### 0.1 Purpose
This composition node groups all capabilities for capturing time spent on work and projects, monitoring project health on a status board, and managing individual tasks within projects. A technician, consultant, or project manager uses these features to record effort, track progress, and ensure projects stay on schedule.

### 0.2 Children
| Child ID | Name | Node type | Group membership |
|---|---|---|---|
| `F-OPS-003-01` | Time Entry | LEAF | mandatory |
| `F-OPS-003-02` | Project Status Board | LEAF | mandatory |
| `F-OPS-003-03` | Task Management | LEAF | optional |

### 0.3 Position in Feature Tree
```
OPS Suite  [ROOT]
+-- F-OPS-003  Time & Project Tracking  [COMPOSITION] <-- you are here
    +-- F-OPS-003-01  Time Entry  [LEAF]
    +-- F-OPS-003-02  Project Status Board  [LEAF]
    +-- F-OPS-003-03  Task Management  [LEAF]
```

---

## SS1. Children & Variability Structure

### 1.1 Group Logic
**Group type:** mixed

**Business rationale:** Time Entry is mandatory because billing and payroll both depend on accurate time capture. Project Status Board is mandatory because projects without a health view cannot be managed. Task Management is optional because some organizations manage tasks in an external tool (e.g., Jira) and only use OPS for time logging and project-level reporting.

### 1.2 Sub-Groups

| Sub-group | Children | Group type | Rationale |
|---|---|---|---|
| Core tracking | `003-01`, `003-02` | mandatory | Minimum viable time and project oversight |
| Task planning | `003-03` | optional | Needed when OPS is the primary task management tool |

### 1.3 Rationale for Tree Position
Time & Project Tracking is complementary to Work Order Management (F-OPS-001). Time entries can reference both work orders and project tasks, making this composition an extension of the core execution layer rather than a dependency on it.

---

## SS2. Constraints

### 2.1 Intra-Node Constraints
| If | Then | Rationale |
|---|---|---|
| `F-OPS-003-03` is included | `F-OPS-003-02` must be included | Task management requires the project board to display task status within project context |

### 2.2 Cross-Node Constraints
None. Time & Project Tracking can operate independently of F-OPS-001 and F-OPS-002, although time entries are typically linked to work orders.

---

## SS3. Quality & Testing Guidance
Integration tests MUST verify that a time entry submitted via F-OPS-003-01 appears in the project's logged hours on the F-OPS-003-02 board. Task status changes in F-OPS-003-03 MUST be reflected immediately on the F-OPS-003-02 board without a page refresh.

---

## SS4. Change Log
| Date | Version | Author | Changes |
|------|---------|--------|---------|
| 2026-04-04 | 1.0.0 | Architecture Team | Initial composition node |

---
**END OF SPECIFICATION**
