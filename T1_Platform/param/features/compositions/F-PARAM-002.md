# F-PARAM-002 — Translation Management

> **Meta Information**
> - **Version:** 2026-04-03
> - **Template:** `feature-composition-spec.md` v1.0.0
> - **Template Compliance:** 100%
> - **Feature ID:** `F-PARAM-002`
> - **Suite:** `param`
> - **Node type:** COMPOSITION
> - **Companion UVL:** `uvl/compositions/F-PARAM-002.uvl`

---

## SS0. Identity

### 0.1 Purpose
This composition node groups all capabilities for managing translations and localized labels. Suite administrators use these features to browse, edit, and bulk-seed translations that the BFF layer resolves for user-visible text.

### 0.2 Children
| Child ID | Name | Node type | Group membership |
|---|---|---|---|
| `F-PARAM-002-01` | Browse Translations | LEAF | mandatory |
| `F-PARAM-002-02` | Edit Translations | LEAF | mandatory |
| `F-PARAM-002-03` | Seed Translation Bundles | LEAF | optional |

### 0.3 Position in Feature Tree
```
PARAM Suite  [ROOT]
+-- F-PARAM-002  Translation Management  [COMPOSITION] <-- you are here
    +-- F-PARAM-002-01  Browse Translations  [LEAF]
    +-- F-PARAM-002-02  Edit Translations  [LEAF]
    +-- F-PARAM-002-03  Seed Translation Bundles  [LEAF]
```

---

## SS1. Children & Variability Structure

### 1.1 Group Logic
**Group type:** mixed

**Business rationale:** Browse and Edit are mandatory — every deployment needs the ability to view and update translations for localization. Seed Translation Bundles is optional — it supports CI/CD bulk seeding workflows that are only needed by suite teams during initial deployment or major releases.

### 1.2 Sub-Groups

| Sub-group | Children | Group type | Rationale |
|---|---|---|---|
| Core management | `002-01`, `002-02` | mandatory | Minimum viable translation administration |
| Bulk seeding | `002-03` | optional | CI/CD pipeline integration for initial data loading |

### 1.3 Rationale for Tree Position
Translation Management is distinct from Reference Data Management because translations operate on a different aggregate (namespace + key + locale) and have their own access control model based on namespace ownership.

---

## SS2. Constraints

### 2.1 Intra-Node Constraints
| If | Then | Rationale |
|---|---|---|
| `F-PARAM-002-03` is included | `F-PARAM-002-01` must be included | Seed results need to be verifiable in browse UI |

### 2.2 Cross-Node Constraints
| If | Then | Rationale |
|---|---|---|
| Any PARAM-002 feature | `F-PARAM-001-01` must be included | Translations reference catalog namespaces — browsing catalogs is needed for namespace lookup |

---

## SS3. Quality & Testing Guidance
All leaf features MUST be independently testable. Integration tests MUST verify that editing a translation in F-PARAM-002-02 triggers a `param.i18n.namespace.updated` event and is visible in F-PARAM-002-01.

---

## SS4. Change Log
| Date | Version | Author | Changes |
|------|---------|--------|---------|
| 2026-04-03 | 1.0.0 | Architecture Team | Initial composition node |

---
**END OF SPECIFICATION**
