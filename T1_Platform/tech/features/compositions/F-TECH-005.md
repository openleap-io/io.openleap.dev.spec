# F-TECH-005 — E-Invoicing

> **Meta Information**
> - **Version:** 2026-04-03
> - **Template:** `feature-composition-spec.md` v1.0.0
> - **Template Compliance:** 100%
> - **Feature ID:** `F-TECH-005`
> - **Suite:** `tech`
> - **Node type:** COMPOSITION
> - **Companion UVL:** `uvl/compositions/F-TECH-005.uvl`

---

## SS0. Identity

### 0.1 Purpose
Minimal admin composition for the stateless ZUGFeRD service. Provides service health, version info, default profile, and configuration visibility.

### 0.2 Children
| Child ID | Name | Node type | Group membership |
|---|---|---|---|
| `F-TECH-005-01` | Service Health & Info | LEAF | mandatory |

### 0.3 Position in Feature Tree
```
TECH Suite  [ROOT]
+-- F-TECH-005  E-Invoicing  [COMPOSITION] <-- you are here
    +-- F-TECH-005-01  Service Health & Info  [LEAF]
```

---

## SS1. Children & Variability Structure

### 1.1 Group Logic
**Group type:** mixed
**Business rationale:** Core browse and management features are mandatory; audit/history features are optional for non-compliance deployments.

---

## SS2. Constraints
All TECH features require IAM authentication (cross-suite).

---

## SS3. Quality & Testing Guidance
All leaf features MUST be independently testable.

---

## SS4. Change Log
| Date | Version | Author | Changes |
|------|---------|--------|---------|
| 2026-04-03 | 1.0.0 | Architecture Team | Initial |

---
**END OF SPECIFICATION**
