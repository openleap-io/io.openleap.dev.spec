# F-SRV-006 — Entitlements

> **Conceptual Stack Layer:** Platform-Feature (Composition Node)
> **Suite:** `srv` | **Node type:** COMPOSITION | **Parent:** `SRV_UI`
> **Companion UVL:** `F-SRV-006.uvl`
> **Version:** 2026-04-02 | **Status:** DRAFT
> **Template:** `feature-composition-spec.md` v1.0.0
> **Template Compliance:** ~100% — all sections present

---

## SS0. Identity

### 0.1 Purpose
Groups capabilities for **entitlement management** — customer rights to consume services (quotas, subscriptions, treatment series). Answers: 'Is the customer entitled, and what remains?'

### 0.2 Children
| Child ID | Name | Node type | Group membership |
|---|---|---|---|
| `F-SRV-006-01` | Entitlement Management | LEAF | mandatory |
| `F-SRV-006-02` | Eligibility Check | LEAF | mandatory |
| `F-SRV-006-03` | Quota Consumption | LEAF | mandatory |

### 0.3 Position in the Feature Tree
```
SRV_UI
├── ...
├── F-SRV-006  Entitlements  [COMPOSITION] ← you are here
│   ├── F-SRV-006-01  Entitlement Management  [mandatory]
│   ├── F-SRV-006-02  Eligibility Check  [mandatory]
│   └── F-SRV-006-03  Quota Consumption  [mandatory]
├── ...
```

---

## SS1. Children & Variability Structure

### 1.1 Group Logic
**Group type:** mandatory

**Business rationale:** All three are inseparable: entitlements without eligibility checks are unenforceable, eligibility without consumption tracking is incomplete, and consumption without entitlement management has nothing to consume against.

---

## SS2. Constraints

### 2.2 Cross-Node Constraints
| Child | Requires / Excludes | External feature | Rationale |
|---|---|---|---|
| `F-SRV-006-01` | requires | `F-SRV-001 (Service Catalog)` | Entitlements reference offerings |
| `F-SRV-006-03` | requires | `F-SRV-004 (Session Execution)` | Consumption triggered by sessions |

---

## SS3. Valid & Invalid Configurations
| Configuration | Selected | Use case |
|---|---|---|
| Full (only option) | `01, 02, 03` | Always selected together |

---

## SS4. Change Log
| Date | Version | Author | Changes |
|------|---------|--------|---------|
| 2026-04-02 | 1.0 | OpenLeap Architecture Team | Initial |

**Status:** DRAFT
