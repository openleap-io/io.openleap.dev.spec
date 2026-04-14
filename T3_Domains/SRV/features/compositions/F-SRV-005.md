# F-SRV-005 — Case Management

> **Conceptual Stack Layer:** Platform-Feature (Composition Node)
> **Suite:** `srv` | **Node type:** COMPOSITION | **Parent:** `SRV_UI`
> **Companion UVL:** `F-SRV-005.uvl`
> **Version:** 2026-04-02 | **Status:** DRAFT
> **Template:** `feature-composition-spec.md` v1.0.0
> **Template Compliance:** ~100% — all sections present

---

## SS0. Identity

### 0.1 Purpose
Groups capabilities for **case management** — grouping related sessions under a case umbrella for cross-session reasoning, treatment tracking, and aggregated billing.

### 0.2 Children
| Child ID | Name | Node type | Group membership |
|---|---|---|---|
| `F-SRV-005-01` | Case Lifecycle | LEAF | mandatory |
| `F-SRV-005-02` | Session Linking | LEAF | mandatory |

### 0.3 Position in the Feature Tree
```
SRV_UI
├── ...
├── F-SRV-005  Case Management  [COMPOSITION] ← you are here
│   ├── F-SRV-005-01  Case Lifecycle  [mandatory]
│   └── F-SRV-005-02  Session Linking  [mandatory]
├── ...
```

---

## SS1. Children & Variability Structure

### 1.1 Group Logic
**Group type:** mandatory

**Business rationale:** A case without session linking is an empty container. Session linking without case lifecycle has nothing to link to. Both are always required together.

---

## SS2. Constraints

### 2.2 Cross-Node Constraints
| Child | Requires / Excludes | External feature | Rationale |
|---|---|---|---|
| `F-SRV-005-01` | requires | `F-SRV-004 (Session Execution)` | Cases aggregate sessions |

---

## SS3. Valid & Invalid Configurations
| Configuration | Selected | Use case |
|---|---|---|
| Full (only option) | `01, 02` | Always selected together |

---

## SS4. Change Log
| Date | Version | Author | Changes |
|------|---------|--------|---------|
| 2026-04-02 | 1.0 | OpenLeap Architecture Team | Initial |

**Status:** DRAFT
