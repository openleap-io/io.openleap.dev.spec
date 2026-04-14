<!-- Template Meta
     Template-ID:   TPL-PROD
     Version:       1.0.0
     Last Updated:  2026-04-03
-->

# OpenLeap Demo — Product Specification

> **Conceptual Stack Layer:** Product
> **Space:** Product (Application Engineering)
> **Owner:** OpenLeap Architecture Team

> **Meta Information**
> - **Version:** 2026-04-03
> - **Template:** `product-spec.md` v1.0.0
> - **Template Compliance:** ~80%
> - **Author(s):** OpenLeap Architecture Team
> - **Status:** DRAFT
> - **Product ID:** `demo`
> - **Project:** internal / openleap
> - **Companion UVL:** `demo.product-config.uvl`

---

## 1. Product Purpose

The **OpenLeap Demo** product is a reference configuration that selects ALL available features from all suites with feature specifications. It serves three purposes:

1. **BFF Reference Implementation** — the demo product config drives BFF instantiation for IAM, PS, and SRV suites, providing a complete working example
2. **Regression Surface** — if the demo product config compiles against all suite catalogs, all features are structurally valid
3. **Platform Showcase** — demonstrates the full breadth of OpenLeap capabilities in a single product

## 2. Suite Selection

| Suite | Tier | Compositions | Leaf Features | Status |
|-------|------|-------------|---------------|--------|
| IAM | T1 | 5 | 22 | All selected |
| PS | T3 | 6 | 36 | All selected |
| SRV | T3 | 7 | 21 | All selected |
| **Total** | | **18** | **79** | |

## 3. Feature Selection Policy

The demo product selects **every available feature**, including optional ones. This means:
- All mandatory features from each suite catalog are included
- All optional features (e.g., F-IAM-001-04 Create Device Principal, F-PS-002 Agile & Boards) are also included
- No negative decisions — every feature is selected
- No attribute overrides — all features use Telos defaults

## 4. Target Personas

Since this is a demo/reference product, all platform personas are included:
- IAM Administrator
- Project Manager
- Service Desk Agent
- Finance Controller
- System Auditor

## 5. Deployment

The demo product is intended for local development and showcase environments only. It is NOT deployed to production.

## 6. Open Questions

| ID | Question | Impact | Owner |
|---|---|---|---|
| Q-001 | Should the demo product have its own UI repo (`io.openleap.demo.ui`) or reuse existing UIs? | Determines BFF instantiation scope | Architecture Team |
