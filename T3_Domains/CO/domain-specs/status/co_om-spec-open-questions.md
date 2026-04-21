# Open Questions - co.om

## Q-OM-001: Reciprocal Allocation in Phase 1
- **Question:** Should reciprocal allocation (iterative matrix solving for mutual service exchanges) be supported in Phase 1?
- **Why it matters:** Reciprocal allocation requires a fundamentally different algorithm (iterative solver or matrix inversion) compared to step-down. It affects the AllocationMethod enum, the execution engine, and performance characteristics.
- **Suggested options:** A) Include in Phase 1 with iterative solver, B) Defer to Phase 2, C) Never support (use activity-based costing as alternative)
- **Owner:** TBD

## Q-OM-002: Allocation Dry-Run Preview Report
- **Question:** Should allocation dry-run generate a preview report (PDF/Excel) without posting, or only return JSON results?
- **Why it matters:** Controllers want to review allocation outcomes before committing. A rich preview improves UX and reduces reversal frequency, but adds report generation complexity.
- **Suggested options:** A) Full preview with PDF/Excel export, B) JSON preview only (UI renders), C) Deferred to Phase 2
- **Owner:** TBD

## Q-OM-003: Partial Settlements by Cost Element
- **Question:** How to handle partial settlements that settle only specific cost elements from an internal order?
- **Why it matters:** Some customers need to settle material costs to products and overhead costs to cost centers in separate steps. Current model settles all costs per the rule.
- **Suggested options:** A) Add costElementFilter to SettlementRule, B) Create separate partial settlement endpoint, C) Defer to Phase 2
- **Owner:** TBD

## Q-OM-004: Feature Dependency Register
- **Question:** Which product features depend on this service's endpoints?
- **Why it matters:** The Feature Dependency Register (§11) cannot be completed without CO feature specs. This affects BFF planning and API change impact assessment.
- **Suggested options:** A) Author CO feature specs first, B) Mark as TBD until product spec authored
- **Owner:** TBD

## Q-OM-005: Multi-Currency Allocations
- **Question:** Should the service support multi-currency allocations within a single run (cost centers in different currencies)?
- **Why it matters:** Multi-national organizations may have cost centers reporting in different currencies. This affects allocation calculation logic, exchange rate handling, and integration with ref-data-svc.
- **Suggested options:** A) Single currency per run (require upfront conversion), B) Multi-currency with automatic conversion at allocation time
- **Owner:** TBD

## Q-OM-006: Port Assignment
- **Question:** What is the exact port assignment for co-om-svc?
- **Why it matters:** Needed for service registry, deployment configuration, and local development setup.
- **Suggested options:** Consult platform team for port range allocation within the CO suite range
- **Owner:** TBD

## Q-OM-007: Effective-Dated Rule Versioning
- **Question:** Should AllocationRule support effective-dated versioning where multiple versions can be active for different periods simultaneously?
- **Why it matters:** Currently a rule can be revised but only one version is active at a time. Period-based versioning would allow a new rule version to take effect for future periods while the old version applies to the current period.
- **Suggested options:** A) Single active version (current design), B) Period-based version overlap allowed
- **Owner:** TBD
