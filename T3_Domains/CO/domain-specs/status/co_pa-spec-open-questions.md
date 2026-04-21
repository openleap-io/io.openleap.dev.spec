# Open Questions - co.pa

## Q-PA-001: Custom Dimensions Beyond Standard Six
- **Question:** Should co.pa support custom analysis dimensions beyond the six standard ones (customer, product, productGroup, salesOrg, region, salesChannel)?
- **Why it matters:** SAP CO-PA operating concerns support up to 50 characteristics. If customers need more dimensions, the data model must support a dynamic dimension schema (e.g., JSONB dimension map) rather than fixed columns.
- **Suggested options:** (A) Fixed six dimensions + custom_fields for extras, (B) Dynamic dimension map (JSONB) replacing fixed columns, (C) Configurable dimension registry with up to 20 dimensions
- **Owner:** TBD

## Q-PA-002: Costing-Based vs. Account-Based PA
- **Question:** Should Phase 2 support account-based PA (actual GL postings mapped to profit segments) alongside costing-based PA?
- **Why it matters:** Account-based PA enables reconciliation with FI general ledger. Many enterprises require both views. Supporting both doubles the data volume and algorithm complexity.
- **Suggested options:** (A) Support both with a toggle per controlling area, (B) Support both simultaneously with separate value field sets, (C) Account-based only as a separate domain service
- **Owner:** TBD

## Q-PA-003: Transfer Pricing for Inter-Company Segments
- **Question:** How should transfer pricing be handled for inter-company profit segments?
- **Why it matters:** Multi-entity enterprises need to eliminate inter-company margins for consolidated profitability. This requires integration with co.pca (profit center accounting) and potentially fi.ic (inter-company).
- **Suggested options:** (A) Dedicated transfer pricing rules in co.pa, (B) Delegate to co.pca for inter-company adjustments, (C) Out of scope — handle in T4 Analytics
- **Owner:** TBD

## Q-PA-004: Multiple Distribution Keys for Top-Down Distribution
- **Question:** Should UC-006 (Top-Down Distribution) support applying different distribution keys to different value fields in a single operation?
- **Why it matters:** Controllers often need to distribute marketing costs by revenue but overhead by headcount. Currently UC-006 supports one key per execution.
- **Suggested options:** (A) One key per execution (current), (B) Multi-key with a distribution plan object, (C) Batch execution of multiple single-key distributions
- **Owner:** TBD

## Q-PA-005: Incremental vs. Full Recalculation
- **Question:** Should the profitability run support incremental recalculation (only segments with changed data) vs. full recalculation of all segments?
- **Why it matters:** Full recalculation is simpler and guarantees consistency but may not scale beyond 100K segments. Incremental is faster but requires change tracking.
- **Suggested options:** (A) Full recalculation only (Phase 1), (B) Full + incremental with a mode flag, (C) Always incremental with periodic full reconciliation
- **Owner:** TBD

## Q-PA-006: Queue Naming Convention
- **Question:** What is the exact queue naming pattern for consumed events? The spec assumes `co.pa.in.{source-suite}.{source-domain}.{topic}`.
- **Why it matters:** Queue names must align with the platform's messaging infrastructure configuration. Inconsistent naming causes routing failures.
- **Suggested options:** (A) `co.pa.in.sd.bil.invoice-posted`, (B) `co-pa-svc.sd.bil.invoice.posted`, (C) Follow suite-level convention from _co_suite.md
- **Owner:** TBD

## Q-PA-007: Plan Data Storage
- **Question:** Should co.pa store budgeted/plan profitability data alongside actuals for variance-to-plan analysis?
- **Why it matters:** Controllers need to compare actual margins against plan. If plan data is stored in co.pa, it enables native variance reporting. If stored in a planning suite, co.pa must integrate with it.
- **Suggested options:** (A) Store plan data in co.pa with a data_type discriminator (actual/plan), (B) Receive plan data from a planning suite via events, (C) Out of scope — variance analysis in T4 Analytics
- **Owner:** TBD

## Q-PA-008: Multi-Currency Conversion
- **Question:** How should currency conversion be handled when profit segments span multiple currencies?
- **Why it matters:** Revenue may be in USD, COGS in EUR. The profitability run needs a consistent currency for margin calculation. The timing of conversion (posting time vs. run time) affects accuracy.
- **Suggested options:** (A) Convert at posting time using spot rate, (B) Convert at run time using period average rate, (C) Store in original currency + group currency, calculate margins in both
- **Owner:** TBD

## Q-PA-009: Conditional Formulas in Contribution Margin Schemes
- **Question:** Should the contribution margin scheme support conditional formulas (e.g., different overhead allocation rates for different product groups)?
- **Why it matters:** Some enterprises have different margin structures for manufacturing vs. services. Conditional formulas add complexity but enable more accurate analysis.
- **Suggested options:** (A) Flat formulas only (current), (B) Conditional formulas with IF/THEN syntax, (C) Multiple schemes per controlling area with segment-level assignment
- **Owner:** TBD

## Q-PA-010: Feature Specs for CO-PA
- **Question:** Which product features depend on co.pa endpoints? No feature specs exist yet for CO-PA.
- **Why it matters:** §11 Feature Dependencies requires mapping features to endpoints. Without feature specs, the mapping is estimated and may be inaccurate.
- **Suggested options:** (A) Create feature specs for CO-PA as next step, (B) Derive features from use cases, (C) Wait for product spec to drive feature identification
- **Owner:** TBD
