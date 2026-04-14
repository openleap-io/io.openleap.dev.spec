# F-IAM-001 — Principal Management

> **Meta Information**
> - **Version:** 2026-04-03
> - **Template:** `feature-composition-spec.md` v1.0.0
> - **Template Compliance:** 100%
> - **Feature ID:** `F-IAM-001`
> - **Suite:** `iam`
> - **Node type:** COMPOSITION
> - **Companion UVL:** `uvl/compositions/F-IAM-001.uvl`

---

## SS0. Identity

### 0.1 Purpose
This composition node groups all principal management capabilities.

### 0.2 Children
| Child ID | Name | Node type | Group membership |
|---|---|---|---|
| `F-IAM-001-01` | Create Human Principal | LEAF | mandatory |
| `F-IAM-001-02` | Create Service Principal | LEAF | mandatory |
| `F-IAM-001-03` | Create System Principal | LEAF | mandatory |
| `F-IAM-001-04` | Create Device Principal | LEAF | optional |
| `F-IAM-001-05` | Principal Lifecycle Mgmt | LEAF | mandatory |
| `F-IAM-001-06` | Profile Management | LEAF | mandatory |
| `F-IAM-001-07` | Credential Management | LEAF | mandatory |

### 0.3 Position in Feature Tree
```
IAM Suite  [ROOT]
+-- F-IAM-001  Principal Management  [COMPOSITION] <-- you are here
    +-- F-IAM-001-01  Create Human Principal  [LEAF]
    +-- F-IAM-001-02  Create Service Principal  [LEAF]
    +-- F-IAM-001-03  Create System Principal  [LEAF]
    +-- F-IAM-001-04  Create Device Principal  [LEAF]
    +-- F-IAM-001-05  Principal Lifecycle Mgmt  [LEAF]
    +-- F-IAM-001-06  Profile Management  [LEAF]
    +-- F-IAM-001-07  Credential Management  [LEAF]
```

---

## SS1. Children & Variability Structure

### 1.1 Group Logic
**Group type:** mixed

**Business rationale:** Core principal types (Human, Service, System) and lifecycle operations are mandatory. Device principal is optional (not all deployments involve IoT).

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
