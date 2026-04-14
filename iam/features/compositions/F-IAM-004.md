# F-IAM-004 — Security Audit

> **Meta Information**
> - **Version:** 2026-04-03
> - **Template:** `feature-composition-spec.md` v1.0.0
> - **Template Compliance:** 100%
> - **Feature ID:** `F-IAM-004`
> - **Suite:** `iam`
> - **Node type:** COMPOSITION
> - **Companion UVL:** `uvl/compositions/F-IAM-004.uvl`

---

## SS0. Identity

### 0.1 Purpose
This composition node groups all security audit capabilities.

### 0.2 Children
| Child ID | Name | Node type | Group membership |
|---|---|---|---|
| `F-IAM-004-01` | Security Event Logging | LEAF | mandatory |
| `F-IAM-004-02` | Audit Query & Reporting | LEAF | mandatory |
| `F-IAM-004-03` | Audit Export & Archival | LEAF | optional |
| `F-IAM-004-04` | SIEM Integration | LEAF | optional |

### 0.3 Position in Feature Tree
```
IAM Suite  [ROOT]
+-- F-IAM-004  Security Audit  [COMPOSITION] <-- you are here
    +-- F-IAM-004-01  Security Event Logging  [LEAF]
    +-- F-IAM-004-02  Audit Query & Reporting  [LEAF]
    +-- F-IAM-004-03  Audit Export & Archival  [LEAF]
    +-- F-IAM-004-04  SIEM Integration  [LEAF]
```

---

## SS1. Children & Variability Structure

### 1.1 Group Logic
**Group type:** mixed

**Business rationale:** Event logging and query/reporting are non-negotiable for compliance. Export/archival and SIEM integration are optional for advanced deployments.

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
