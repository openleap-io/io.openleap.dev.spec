# Open Questions - fi.cnsl

## Q-CNSL-001: Feature Dependency Register
- **Question:** Which product features depend on fi.cnsl service endpoints and events?
- **Why it matters:** SS11 Feature Dependency Register cannot be populated without feature specs. This affects BFF aggregation hints and impact assessment for API changes.
- **Suggested options:** Wait for FI consolidation feature specs to be created; populate SS11 once feature specs exist.
- **Owner:** Product Team

## Q-CNSL-002: Extension Action Candidates
- **Question:** What custom extension actions should be available on completed consolidation runs in the product UI?
- **Why it matters:** SS12.5 Extension Actions needs product UI context to define meaningful action extension points.
- **Suggested options:** A) Export to legacy consolidation system, B) Generate custom report format, C) Submit for approval workflow, D) All of the above.
- **Owner:** Product Team

## Q-CNSL-003: Incremental Consolidation
- **Question:** Should fi.cnsl support incremental consolidation (re-run only changed entities instead of full re-consolidation)?
- **Why it matters:** Performance optimization for large groups with 50+ entities where only a few entities have changes. Full re-run may take 30+ minutes.
- **Suggested options:** A) Always full re-run (simpler, audit-friendly), B) Incremental with delta detection (complex, faster for large groups).
- **Owner:** Architecture Team

## Q-CNSL-004: Goodwill Impairment Testing
- **Question:** How should goodwill impairment testing be handled within or alongside fi.cnsl?
- **Why it matters:** Goodwill from acquisitions (recorded via IC_INVESTMENT and GOODWILL elimination types) requires annual impairment testing under IAS 36 / ASC 350. This is a significant accounting process.
- **Suggested options:** A) Separate goodwill management service, B) Sub-process within fi.cnsl, C) Defer to manual process initially.
- **Owner:** Domain Team

## Q-CNSL-005: Port Assignment
- **Question:** What is the correct port assignment for fi-cnsl-svc?
- **Why it matters:** Service identity metadata requires an assigned port. Currently set to 8490 as placeholder.
- **Suggested options:** Check port registry for next available port in FI suite range.
- **Owner:** Platform Team

## Q-CNSL-006: Multi-Level Sub-Consolidation
- **Question:** Should fi.cnsl support multi-level sub-consolidation (e.g., consolidate regional sub-groups first, then roll up to global)?
- **Why it matters:** Large multinational groups (SAP EC-CS supports this) consolidate regionally before global rollup. This affects the ConsolidationGroup data model (group-of-groups) and workflow complexity.
- **Suggested options:** A) Flat consolidation only (all entities at same level), B) Support sub-group consolidation with cascading runs.
- **Owner:** Architecture Team

## Q-CNSL-007: Exchange Rate Sourcing
- **Question:** How should exchange rates be sourced for currency translation -- exclusively from ref-data-svc, or with manual override capability?
- **Why it matters:** Currency translation accuracy depends on correct exchange rates. Some organizations need to use specific rates (e.g., central bank rates) that may differ from market rates in ref-data-svc.
- **Suggested options:** A) ref-data-svc only, B) Manual entry only, C) ref-data-svc with manual override (manual takes priority).
- **Owner:** Domain Team

## Q-CNSL-008: Equity Method as Separate Aggregate
- **Question:** Should equity method investments (IAS 28 associates and joint ventures) be modeled as a separate aggregate rather than being part of the ConsolidationRun?
- **Why it matters:** Equity method accounting has a different lifecycle (single-line investment, share of profit/loss) than full consolidation. Mixing both in ConsolidationRun may create unnecessary complexity.
- **Suggested options:** A) Keep within ConsolidationRun (simpler, one consolidation process), B) Separate EquityMethodInvestment aggregate with own lifecycle.
- **Owner:** Architecture Team
