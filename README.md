# OpenLeap Domain & Service Specifications

This repository is the **authoritative source of truth** for all OpenLeap ERP domain specifications. It contains formal specifications for every platform service, organized by the four-tier architecture. No runnable application code.

**Spec is source of truth:** update specs here first, then derive implementation in service repos.

---

## Structure

Specifications are organized by the four-tier architecture:

| Tier | Path | Contents |
|------|------|----------|
| **T1** Platform | `T1_Platform/` | IAM (principal, authz, tenant, audit), PARAM (cfg, i18n, ref), TECH (dms, jc, nfs, rpt, zugferd, search, email, ai) |
| **T2** Common | `T2_Common/` | two cross-suite suites: **shared** (bp, cap) — master data; **auto** (ntf, wf) — automation fabric (← promoted from crm.ntf, crm.wf) |
| **T3** Core Business | `T3_Domains/` | CO, COM, CRM, FAC, FI, HR, OPS, PPS, PS, SD, SRV, TKS |
| **T4** Data & Analytics | `T4_Data/` | bi (Data Platform) |
| Products | `products/` | Product-specific specifications |

### Per-Domain Structure

Each domain directory follows this layout:

```
{suite}/{suite}_{domain}/
├── domain-specs/       — domain model, aggregates, invariants
├── contracts/          — OpenAPI, AsyncAPI, JSON Schema
├── features/           — feature specs with UVL and screen contracts
└── uvl/                — feature model files
```

---

## Conventions

| Aspect | Pattern | Example |
|--------|---------|---------|
| Exchange | `<suite>.<domain>.events` | `pps.pd.events`, `shared.bp.events`, `auto.ntf.events` |
| Routing key | `<suite>.<domain>.<aggregate>.<event>` | `pps.pd.product.released`, `shared.bp.party.created`, `auto.wf.workflow.triggered` |
| API path | `/api/<suite>/<domain>/v1` | `/api/fi/gl/v1`, `/api/shared/bp/v1`, `/api/auto/ntf/v1` |
| DB schema | `<suite>_<domain>` | `pps_pd`, `shared_bp`, `auto_notification` |
| Feature ID (T3 flat) | `F-{SUITE}-{NNN}[-{NN}]` | `F-PPS-012-01` |
| Feature ID (T1/T2 hierarchical) | `F-{SUITE}-{DOMAIN}-{NNN}[-{NN}]` | `F-SHARED-BP-001-02`, `F-AUTO-NTF-001` |

Domain codes are uppercase in directory names (e.g., `FI`, `CRM`), lowercase in API paths and event exchanges.

---

## Authoring Specs

All specifications must follow the consolidated templates in [dev.concepts](https://github.com/openleap-io/io.openleap.dev.concepts):

- Suite specs: `templates/platform/suite-spec.md`
- Domain/service specs: `templates/platform/domain-service-spec.md`
- Feature specs: `templates/platform/feature-spec.md`

---

## Related Repositories

| Repository | Relationship |
|------------|-------------|
| [dev.hub](https://github.com/openleap-io/io.openleap.dev.hub) | Central documentation, architecture overview, repo landscape |
| [dev.concepts](https://github.com/openleap-io/io.openleap.dev.concepts) | Specification framework — templates, artifact definitions, examples |
| [dev.prompts](https://github.com/openleap-io/io.openleap.dev.prompts) | AI prompts for batch operations on these specs |
| [dev.guidelines](https://github.com/openleap-io/io.openleap.dev.guidelines) | Backend development guidelines (DDD/CQRS, Java 25, Spring Boot 4) |
| [ui.guidelines](https://github.com/openleap-io/io.openleap.ui.guidelines) | Frontend development guidelines (Vue 3, Nuxt 4) |
