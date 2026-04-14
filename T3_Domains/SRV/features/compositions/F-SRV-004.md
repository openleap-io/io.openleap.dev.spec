# F-SRV-004 — Session Execution

> **Conceptual Stack Layer:** Platform-Feature (Composition Node)
> **Suite:** `srv` | **Node type:** COMPOSITION | **Parent:** `SRV_UI`
> **Companion UVL:** `F-SRV-004.uvl`
> **Version:** 2026-04-02 | **Status:** DRAFT
> **Template:** `feature-composition-spec.md` v1.0.0
> **Template Compliance:** ~100% — all sections present

---

## SS0. Identity

### 0.1 Purpose
Groups all capabilities related to **session execution** — capturing what actually happened during service delivery. Sessions are the bridge between booking and billing.

### 0.2 Children
| Child ID | Name | Node type | Group membership |
|---|---|---|---|
| `F-SRV-004-01` | Session Lifecycle | LEAF | mandatory |
| `F-SRV-004-02` | Outcome Recording | LEAF | mandatory |
| `F-SRV-004-03` | Proof-of-Service | LEAF | optional |

### 0.3 Position in the Feature Tree
```
SRV_UI
├── ...
├── F-SRV-004  Session Execution  [COMPOSITION] ← you are here
│   ├── F-SRV-004-01  Session Lifecycle  [mandatory]
│   ├── F-SRV-004-02  Outcome Recording  [mandatory]
│   └── F-SRV-004-03  Proof-of-Service  [optional]
├── ...
```

---

## SS1. Children & Variability Structure

### 1.1 Group Logic
**Group type:** mixed (mandatory + optional)

**Business rationale:** Session Lifecycle and Outcome Recording are inseparable: a completed session without an outcome is an incomplete business fact. Proof-of-Service is optional because not all service contexts require formal evidence (e.g., internal training).

---

## SS2. Constraints

### 2.2 Cross-Node Constraints
| Child | Requires / Excludes | External feature | Rationale |
|---|---|---|---|
| `F-SRV-004-01` | requires | `F-SRV-002 (Appointment & Booking)` | Sessions created from appointments |
| `F-SRV-004-03` | requires | `t1.dms (OPEN QUESTION)` | Proof artifacts stored in DMS |

---

## SS3. Valid & Invalid Configurations
| Configuration | Selected | Use case |
|---|---|---|
| Full Execution | `01, 02, 03` | Regulated/auditable services |
| Core Execution | `01, 02` | Internal/informal services |

---

## SS4. Change Log
| Date | Version | Author | Changes |
|------|---------|--------|---------|
| 2026-04-02 | 1.0 | OpenLeap Architecture Team | Initial |

**Status:** DRAFT
