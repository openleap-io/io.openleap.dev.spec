# F-IAM-003 — Tenant Management

> **Meta Information**
> - **Version:** 2026-04-03
> - **Template:** `feature-composition-spec.md` v1.0.0
> - **Template Compliance:** 100%
> - **Feature ID:** `F-IAM-003`
> - **Suite:** `iam`
> - **Node type:** COMPOSITION
> - **Companion UVL:** `uvl/compositions/F-IAM-003.uvl`

---

## SS0. Identity

### 0.1 Purpose
This composition node groups all tenant management capabilities.

### 0.2 Children
| Child ID | Name | Node type | Group membership |
|---|---|---|---|
| `F-IAM-003-01` | Tenant Lifecycle | LEAF | mandatory |
| `F-IAM-003-02` | Organization Hierarchy | LEAF | optional |
| `F-IAM-003-03` | Tenant Configuration | LEAF | mandatory |
| `F-IAM-003-04` | Tenant Quotas | LEAF | optional |

### 0.3 Position in Feature Tree
```
IAM Suite  [ROOT]
+-- F-IAM-003  Tenant Management  [COMPOSITION] <-- you are here
    +-- F-IAM-003-01  Tenant Lifecycle  [LEAF]
    +-- F-IAM-003-02  Organization Hierarchy  [LEAF]
    +-- F-IAM-003-03  Tenant Configuration  [LEAF]
    +-- F-IAM-003-04  Tenant Quotas  [LEAF]
```

---

## SS1. Children & Variability Structure

### 1.1 Group Logic
**Group type:** mixed

**Business rationale:** Tenant lifecycle and configuration are required for any multi-tenant deployment. Organization hierarchy and quotas are optional for simpler deployments.

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
