# F-IAM-002 — Authorization

> **Meta Information**
> - **Version:** 2026-04-03
> - **Template:** `feature-composition-spec.md` v1.0.0
> - **Template Compliance:** 100%
> - **Feature ID:** `F-IAM-002`
> - **Suite:** `iam`
> - **Node type:** COMPOSITION
> - **Companion UVL:** `uvl/compositions/F-IAM-002.uvl`

---

## SS0. Identity

### 0.1 Purpose
This composition node groups all authorization capabilities.

### 0.2 Children
| Child ID | Name | Node type | Group membership |
|---|---|---|---|
| `F-IAM-002-01` | RBAC Management | LEAF | mandatory |
| `F-IAM-002-02` | Permission Check | LEAF | mandatory |
| `F-IAM-002-03` | ABAC Policy Evaluation | LEAF | optional |
| `F-IAM-002-04` | Delegation & Impersonation | LEAF | optional |

### 0.3 Position in Feature Tree
```
IAM Suite  [ROOT]
+-- F-IAM-002  Authorization  [COMPOSITION] <-- you are here
    +-- F-IAM-002-01  RBAC Management  [LEAF]
    +-- F-IAM-002-02  Permission Check  [LEAF]
    +-- F-IAM-002-03  ABAC Policy Evaluation  [LEAF]
    +-- F-IAM-002-04  Delegation & Impersonation  [LEAF]
```

---

## SS1. Children & Variability Structure

### 1.1 Group Logic
**Group type:** mixed

**Business rationale:** RBAC and Permission Check form the minimum viable authorization. ABAC and Delegation are optional advanced features.

---

## SS2. Constraints
No cross-suite dependencies. IAM is foundational.

---

## SS3. Quality & Testing Guidance
All leaf features MUST be independently testable. Integration tests MUST cover mandatory children together.

---

## SS4. Change Log
| Date | Version | Author | Changes |
|------|---------|--------|---------|
| 2026-04-03 | 1.0 | Architecture Team | Initial composition node |

---
**END OF SPECIFICATION**
