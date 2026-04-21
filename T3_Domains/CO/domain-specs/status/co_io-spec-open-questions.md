# Open Questions - co.io

## Q-IO-001: Statistical Order Support
- **Question:** Should the service fully support statistical orders (no budget enforcement, informational cost tracking only)?
- **Why it matters:** Affects data model (statisticalFlag added provisionally), budget check logic (skip for statistical), and order type semantics. SAP CO supports statistical orders as a standard feature.
- **Suggested options:** A) Full support with statisticalFlag on InternalOrder (current provisional approach), B) Separate order type "statistical" with different validation rules, C) Defer to Phase 2
- **Owner:** TBD

## Q-IO-002: PS Module Integration for Project Orders
- **Question:** Should co.io integrate with the PS (Project System) module for project-linked internal orders?
- **Why it matters:** Project orders in SAP are linked to WBS elements. If PS integration is needed, co.io must consume PS events or share aggregate references. Affects scope and cross-domain workflow design.
- **Suggested options:** A) Consume PS events for project order sync, B) Shared aggregate pattern, C) Defer to Phase 2
- **Owner:** TBD

## Q-IO-003: Feature Dependency Register
- **Question:** Which product features depend on this service's endpoints?
- **Why it matters:** Section 11 (Feature Dependencies) is populated provisionally. Accurate feature specs are needed for BFF design, impact assessment, and API change management.
- **Suggested options:** Author CO suite feature specs (F-CO-IO-001 through F-CO-IO-003)
- **Owner:** TBD

## Q-IO-004: Order Number Generation Pattern
- **Question:** Should orderNumber generation be configurable per tenant (different numbering conventions)?
- **Why it matters:** Different organizations have different internal order numbering standards. A fixed pattern limits flexibility; a configurable pattern adds complexity.
- **Suggested options:** A) Fixed pattern "IO-{year}-{seq}" (current), B) Configurable pattern template per tenant (e.g., "{prefix}-{year}-{seq}"), C) Allow manual entry with uniqueness check
- **Owner:** TBD

## Q-IO-005: Hard Stop Rejection for FI-Originated Postings
- **Question:** How should co.io handle hard_stop budget rejection for postings originating from FI events?
- **Why it matters:** FI posting is already committed in a separate transaction when co.io receives the event. Rejecting the co.io posting creates an inconsistency between FI (cost recorded) and CO (cost not recorded). This is a fundamental tension between event-driven architecture and synchronous budget checks.
- **Suggested options:** A) Record blocked posting + alert responsible person (current approach in DC-003), B) Accept posting and mark order as over-budget with alert, C) Publish compensating FI reversal event
- **Owner:** TBD

## Q-IO-006: Multi-Currency Budget Support
- **Question:** Should the service support budgets in a different currency than the order's operational currency?
- **Why it matters:** Multinational organizations may budget in group currency (e.g., EUR) but post costs in local currency (e.g., USD). Requires currency conversion logic, exchange rate references, and more complex balance calculations.
- **Suggested options:** A) Single currency per order (current design, simpler), B) Multi-currency with conversion at posting time using configurable exchange rates
- **Owner:** TBD

## Q-IO-007: Order Hierarchies (Parent/Child Orders)
- **Question:** Should the service support order hierarchies (parent orders with child sub-orders)?
- **Why it matters:** SAP supports order groups for complex budgeting. Large programs (e.g., factory renovation) may need a parent order with sub-orders for individual work packages, with cascading budget controls.
- **Suggested options:** A) Flat orders only (current, simpler), B) Order hierarchy with cascading budgets and rollup reporting
- **Owner:** TBD
