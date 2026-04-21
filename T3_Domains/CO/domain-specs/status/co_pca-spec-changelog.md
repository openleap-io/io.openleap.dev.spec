# Spec Changelog - co.pca

## Summary
- Upgraded co_pca-spec.md from ~75% to ~95% TPL-SVC v1.0.0 compliance
- Added full attribute tables with types, constraints, and read-only flags for all aggregates
- Added detailed business rule definitions (BR-001 through BR-013) with error handling and examples
- Expanded all REST endpoints with full request/response JSON bodies
- Added full event envelope format and payload examples for all published events
- Added consumed event details with queue configuration and failure handling
- Added complete table definitions with column types, indexes, and retention policies
- Populated SS9 (Security), SS10 (Quality), SS11 (Feature Dependencies), SS12 (Extension Points), SS13 (Migration) with substantive content

## Added Sections
- SS3.1: Conceptual overview narrative
- SS3.3: Aggregate Root / Child Entity / Value Object sub-structure with attribute tables
- SS3.4: All five enumerations (SegmentType, ProfitCenterStatus, PricingMethod, BalanceStatus, TransferPriceRuleStatus)
- SS3.5: Shared types (Money, DateRange) with used-by references
- SS4.2: Detailed rule definitions for BR-001, BR-003, BR-004, BR-009, BR-013
- SS4.3: Field-level and cross-field validations
- SS4.4: Reference data dependencies
- SS5.2: Use case detail (preconditions, main flow, postconditions, exception flows) for all UCs
- SS5.2: New use cases UC-001b (UpdateProfitCenter), UC-001c (ChangeProfitCenterStatus), UC-004 (ManageTransferPriceRules)
- SS5.3: Process flow diagram (sequence diagram)
- SS5.4: Cross-domain workflow documentation (Period-End P&L Calculation)
- SS6.1: Authentication and authorization details
- SS6.2: Full request/response bodies for all 10 endpoints
- SS7.2: Event envelope and payload examples for all 3 published events
- SS7.3: Consumed event details with queue configuration and failure handling for all 5 consumed events
- SS7.4: Event flow diagram
- SS7.5: Integration points summary (upstream and downstream)
- SS8.1: Storage technology declaration
- SS8.3: Full table definitions for 4 tables (profit_center, profit_center_balance, transfer_price_rule, pca_outbox_events)
- SS8.4: Reference data dependencies
- SS9.1: Data classification with sensitivity levels
- SS9.2: Roles, permissions, expanded permission matrix, data isolation
- SS9.3: Compliance requirements (SOX, GDPR)
- SS10.1-10.4: Full quality attributes (throughput, concurrency, availability, scalability, maintainability)
- SS11.1-11.5: Feature dependency framework with preliminary feature IDs
- SS12.1-12.8: Full extension points (custom fields, extension events, rules, actions, hooks, API, summary)
- SS13.1: SAP CO-PCA migration mapping
- SS13.2: Deprecation framework
- SS14.1: Consistency checks (7/7 filled)
- SS14.2: Decisions & conflicts
- SS14.3: Renumbered and expanded open questions (Q-PCA-001 through Q-PCA-008)
- SS14.4: ADR framework
- SS14.5: Suite-level ADR references
- SS15.1: Expanded glossary with 8 terms

## Modified Sections
- Preamble: Added Source of Truth Priority block; updated compliance to ~95%
- SS1.5: Added Responsibilities, Authoritative Sources, expanded context diagram
- SS3.3.1 ProfitCenter: Added description, currency, createdAt, updatedAt; added state descriptions, allowed transitions, domain events
- SS3.3.2 TransferPriceRule: Added marketPriceSource, currency, createdAt, updatedAt; added lifecycle, invariants, domain events
- SS5.2 UC-001: Expanded with full preconditions, main flow, postconditions, alternative/exception flows
- SS5.2 UC-002: Expanded with preconditions, postconditions, business rules, alternative/exception flows
- SS9: Restructured from flat table to full 3-section layout
- SS10: Restructured from flat table to full 4-section layout

## Removed Sections
- None (non-destructive upgrade)

## Decisions Taken
- DC-001: Resolved apparent conflict between SS5.4 (cross-domain = YES) and SS7.1 (choreography pattern). The calculation uses sync API calls, not saga orchestration.
- DC-002: ProfitCenterBalance kept as child entity of ProfitCenter despite having its own unique key.
- ProfitCenter and TransferPriceRule marked as extensible (custom_fields JSONB); ProfitCenterBalance marked as not extensible (calculated output).

## Open Questions Raised
- Q-PCA-003: Feature dependency register requires feature spec IDs (not yet authored)
- Q-PCA-004: Balance immutability vs. adjusted status for post-close corrections
- Q-PCA-005: Port assignment
- Q-PCA-006: Repository URI
- Q-PCA-007: Incremental vs. full P&L recalculation
- Q-PCA-008: Currency conversion for consolidated balance summaries
