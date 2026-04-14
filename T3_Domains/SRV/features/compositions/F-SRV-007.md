# F-SRV-007 — Billing Intent

> **Conceptual Stack Layer:** Platform-Feature (Composition Node)
> **Suite:** `srv` | **Node type:** COMPOSITION | **Parent:** `SRV_UI`
> **Companion UVL:** `F-SRV-007.uvl`
> **Version:** 2026-04-02 | **Status:** DRAFT
> **Template:** `feature-composition-spec.md` v1.0.0
> **Template Compliance:** ~100% — all sections present

---

## SS0. Identity

### 0.1 Purpose
Groups capabilities for **billing intent management** — auditable billable positions derived from service delivery facts. Bridges execution reality to downstream invoicing.

### 0.2 Children
| Child ID | Name | Node type | Group membership |
|---|---|---|---|
| `F-SRV-007-01` | Intent Derivation | LEAF | mandatory |
| `F-SRV-007-02` | Correction & Reversal | LEAF | mandatory |
| `F-SRV-007-03` | Reconciliation View | LEAF | optional |

### 0.3 Position in the Feature Tree
```
SRV_UI
├── ...
├── F-SRV-007  Billing Intent  [COMPOSITION] ← you are here
│   ├── F-SRV-007-01  Intent Derivation  [mandatory]
│   ├── F-SRV-007-02  Correction & Reversal  [mandatory]
│   └── F-SRV-007-03  Reconciliation View  [optional]
├── ...
```

---

## SS1. Children & Variability Structure

### 1.1 Group Logic
**Group type:** mixed (mandatory + optional)

**Business rationale:** Intent Derivation and Correction & Reversal are inseparable: billing intents without corrections violate audit requirements. Reconciliation View is optional because some deployments handle reconciliation in downstream FI systems.

---

## SS2. Constraints

### 2.2 Cross-Node Constraints
| Child | Requires / Excludes | External feature | Rationale |
|---|---|---|---|
| `F-SRV-007-01` | requires | `F-SRV-004 (Session Execution)` | Intents derived from sessions |
| `F-SRV-007-01` | requires | `F-SRV-002 (Appointment & Booking)` | Fee intents from no-show/cancel |

---

## SS3. Valid & Invalid Configurations
| Configuration | Selected | Use case |
|---|---|---|
| Full Billing | `01, 02, 03` | Self-contained billing reconciliation |
| Core Billing | `01, 02` | Reconciliation done in downstream FI |

---

## SS4. Change Log
| Date | Version | Author | Changes |
|------|---------|--------|---------|
| 2026-04-02 | 1.0 | OpenLeap Architecture Team | Initial |

**Status:** DRAFT
