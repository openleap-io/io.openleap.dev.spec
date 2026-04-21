# Spec Changelog - co.pa

## Summary
- Upgraded co_pa-spec.md to full TPL-SVC v1.0.0 compliance (~95%)
- Expanded Guidelines Compliance block with full Non-Negotiables, Source of Truth Priority, and Style Guide
- Added ContributionMarginScheme aggregate with MarginLevel child entity (§3.3.3)
- Expanded all business rules with detailed definitions including error handling and examples (§4.2)
- Added field-level and cross-field data validation rules (§4.3) and reference data dependencies (§4.4)
- Expanded all use cases with Actor, Preconditions, Main Flow, Postconditions, Business Rules Applied (§5.2)
- Added UC-002 (CreateContributionMarginScheme), UC-005 (ViewProfitabilitySummary), UC-006 (TopDownDistribution)
- Expanded REST API with full request/response JSON bodies, headers, error responses for all endpoints (§6)
- Expanded all published/consumed events with envelope format, payload examples, queue config, failure handling (§7)
- Added full table definitions with columns, indexes, relationships, retention for all tables (§8.3)
- Added co_pa_standard_cost_cache and co_pa_outbox_events tables (§8.3)
- Populated §9 with data classification rationale, roles/permissions matrix, compliance requirements
- Expanded §10 with throughput, concurrency, failure scenarios, scalability, monitoring, alerting
- Populated §11 Feature Dependencies with estimated feature mapping and BFF aggregation hints
- Populated §12 Extension Points with all 5 extension types (custom fields, events, rules, actions, hooks)
- Populated §13 Migration with SAP CO-PA source mapping and deprecation framework
- Added ADR-PA-002 (Batch Operation), ADR-PA-003 (Standard Cost Cache) to §14.2
- Expanded §14.3 with 7 new open questions (Q-PA-004 through Q-PA-010)
- Ran consistency checks in §14.1 — all passing
- Expanded glossary with 5 new terms

## Added Sections
- §1.5 Responsibilities and Authoritative Sources
- §3.3.3 ContributionMarginScheme aggregate (with MarginLevel child entity)
- §3.4 Enumerations (SegmentStatus, RunStatus, SalesChannel, SchemeStatus)
- §3.5 Shared Types (Money, DimensionSet value objects)
- §4.2 Detailed Rule Definitions (BR-001 through BR-008)
- §4.3 Data Validation Rules (field-level and cross-field)
- §4.4 Reference Data Dependencies
- §5.2 UC-002, UC-005, UC-006 (new use cases)
- §5.4 Cross-Domain Workflows (Period-End Profitability Calculation)
- §6.2.1-6.2.6 Full resource operations with request/response bodies
- §6.3.3 Top-Down Distribution endpoint
- §7.2 Detailed event definitions with envelopes (4 events)
- §7.3 Detailed consumed events with queue config and failure handling
- §7.4 Event Flow Diagram
- §7.5 Integration Points Summary
- §8.1 Storage Technology section
- §8.3 Full table definitions (7 tables)
- §8.4 Reference Data Dependencies
- §9.3 Compliance Requirements
- §10.2-§10.4 Availability, Scalability, Maintainability
- §11.1-§11.5 Feature Dependencies (full framework)
- §12.1-§12.8 Extension Points (full framework)
- §13.1-§13.2 Migration & Evolution (full framework)
- §14.1 Consistency Checks
- §14.4-§14.5 ADR framework and suite-level references

## Modified Sections
- Preamble: Updated version to 2026-04-04, compliance to ~95%
- Guidelines Compliance: Added Source of Truth Priority block
- §3.1 Conceptual Overview: Added contribution margin scheme concept
- §3.2 Core Concepts: Added ContributionMarginScheme and MarginLevel to class diagram
- §3.3.1 ProfitSegment: Added version, businessKey, createdAt, updatedAt; added state descriptions and transitions
- §3.3.2 ProfitabilityRun: Promoted to full aggregate root with attribute table, states, transitions
- §4.1 Business Rules Catalog: Added BR-003, BR-007, BR-008
- §5.2 Use Cases: Expanded UC-001 and UC-003 with full flow details
- §5.4 Cross-Domain Workflows: Changed from NO to YES with full workflow documentation
- §6 REST API: Expanded from bullet endpoints to full request/response format
- §7 Events: Expanded from summary tables to detailed event definitions
- §8 Data Model: Expanded from ER diagram only to full table definitions
- §9 Security: Expanded with classification rationale and compliance requirements
- §10 Quality Attributes: Expanded with throughput, failure scenarios, scalability
- §14 Decisions: Restructured with consistency checks, decisions, open questions
- §15 Appendix: Expanded glossary, added references

## Removed Sections
- None (non-destructive upgrade)

## Decisions Taken
- ADR-PA-002: Profitability run is a batch operation (matches SAP CO-PA period-end paradigm)
- ADR-PA-003: Local standard cost cache to reduce coupling with co.pc
- ProfitSegment and ContributionMarginScheme marked as extensible (custom_fields); ProfitabilityRun not extensible
- Cross-domain workflow changed from NO to YES (period-end profitability involves SD, co.pc, co.om, co.cca)

## Open Questions Raised
- Q-PA-004: Multiple distribution keys for top-down distribution
- Q-PA-005: Incremental vs. full recalculation for profitability runs
- Q-PA-006: Exact queue naming convention for consumed events
- Q-PA-007: Plan data storage alongside actuals
- Q-PA-008: Multi-currency conversion timing
- Q-PA-009: Conditional formulas in contribution margin schemes
- Q-PA-010: Product feature specs not yet created for CO-PA
