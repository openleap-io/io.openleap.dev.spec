# F-IAM-005 — GDPR Compliance

> **Meta Information**
> - **Version:** 2026-04-03
> - **Template:** `feature-composition-spec.md` v1.0.0
> - **Template Compliance:** 100%
> - **Feature ID:** `F-IAM-005`
> - **Suite:** `iam`
> - **Node type:** COMPOSITION
> - **Companion UVL:** `uvl/compositions/F-IAM-005.uvl`

---

## SS0. Identity

### 0.1 Purpose
This composition node groups all gdpr compliance capabilities.

### 0.2 Children
| Child ID | Name | Node type | Group membership |
|---|---|---|---|
| `F-IAM-005-01` | Data Export | LEAF | mandatory |
| `F-IAM-005-02` | Data Erasure | LEAF | mandatory |

### 0.3 Position in Feature Tree
```
IAM Suite  [ROOT]
+-- F-IAM-005  GDPR Compliance  [COMPOSITION] <-- you are here
    +-- F-IAM-005-01  Data Export  [LEAF]
    +-- F-IAM-005-02  Data Erasure  [LEAF]
```

---

## SS1. Children & Variability Structure

### 1.1 Group Logic
**Group type:** mandatory

**Business rationale:** GDPR Article 15 (access) and Article 17 (erasure) are both legally required for any deployment processing EU personal data.

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
