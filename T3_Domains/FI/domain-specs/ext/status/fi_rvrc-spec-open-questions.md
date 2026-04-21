# Open Questions - fi.rvrc

## Q-RVRC-001: Feature Dependency Register
- **Question:** Which platform features depend on this service's endpoints?
- **Why it matters:** The feature dependency register (§11) cannot be completed without feature specs. This affects BFF aggregation hints, impact assessment, and change management planning.
- **Suggested options:** A) Create feature specs for revenue contract management, recognition run, and reconciliation; B) Defer until product spec is created and features are defined
- **Owner:** TBD

## Q-RVRC-002: Variable Consideration Default Method
- **Question:** Should variable consideration use the expected value or most likely amount method as default?
- **Why it matters:** ASC 606 allows two methods for estimating variable consideration. The default affects allocation calculation for contracts with variable pricing (rebates, bonuses, penalties).
- **Suggested options:** A) Expected value — weighted average of possible outcomes (better for large populations); B) Most likely amount — single most likely estimate (better for binary outcomes); C) Configurable per contract (most flexible)
- **Owner:** TBD

## Q-RVRC-003: Port Assignment
- **Question:** What is the exact port assignment for fi-rvrc-svc?
- **Why it matters:** Service registry, deployment configuration, and local development setup require a defined port. Currently using placeholder 8443.
- **Suggested options:** A) 8443 (current placeholder); B) Assign from FI suite port range per infrastructure team conventions
- **Owner:** TBD

## Q-RVRC-004: Usage-Based POB Integration
- **Question:** Should usage-based POBs integrate with a metering/usage tracking service?
- **Why it matters:** The USAGE recognition method requires external progress data (e.g., GB consumed, API calls made). Without a metering integration, usage progress must be manually updated.
- **Suggested options:** A) Consume usage events from a dedicated metering service; B) Accept manual usage updates via REST API; C) Support both patterns
- **Owner:** TBD

## Q-RVRC-005: Multi-Currency Contract Handling
- **Question:** How should multi-currency contracts be handled when POBs are in different currencies?
- **Why it matters:** SSP allocation requires comparing POB prices in a common currency. Different approaches have different implications for rounding, exchange rate risk, and compliance.
- **Suggested options:** A) Force all POBs to contract currency at creation time; B) Convert at allocation-time exchange rate (requires fx-rate service integration); C) Reject multi-currency POBs (simplest)
- **Owner:** TBD

## Q-RVRC-006: Percentage-of-Completion Cost-Based Measurement
- **Question:** Should fi.rvrc support the percentage-of-completion (POC) input method with cost-based measurement?
- **Why it matters:** Some industries (construction, engineering) measure progress by costs incurred vs. estimated total costs. This is a common revenue recognition pattern not currently modeled explicitly.
- **Suggested options:** A) Support via MILESTONE method with cost-based milestones (reuse existing model); B) Add explicit POC method with cost tracking fields; C) Defer to product extension via custom fields and hooks
- **Owner:** TBD

## Q-RVRC-007: Deferred Revenue Reconciliation Process
- **Question:** What is the reconciliation process between fi.rvrc deferred revenue subledger and fi.gl control accounts?
- **Why it matters:** Period-end close requires deferred revenue reconciliation. The process determines whether fi.rvrc provides automated reconciliation reports or relies on fi.rpt.
- **Suggested options:** A) Automated reconciliation report endpoint in fi.rvrc; B) Manual reconciliation via fi.rpt using data from both fi.rvrc and fi.gl; C) Both (automated report + manual override)
- **Owner:** TBD

## Q-RVRC-008: Separate RVR_OPERATOR Role
- **Question:** Should there be a separate RVR_OPERATOR role for running recognition without full admin access?
- **Why it matters:** SOX compliance requires segregation of duties. Currently, contract creation and recognition run both require RVR_ADMIN. A separate operator role would allow the recognition run to be executed by a different person.
- **Suggested options:** A) Add RVR_OPERATOR role with permissions: read + run recognition only; B) Keep two-role model (VIEWER, ADMIN) and rely on product-level RBAC for SOX segregation; C) Delegate to product-level RBAC customization
- **Owner:** TBD
