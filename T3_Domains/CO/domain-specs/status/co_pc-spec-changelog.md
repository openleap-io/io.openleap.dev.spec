# Spec Changelog - co.pc

## Summary
- Upgraded co_pc-spec.md from ~80% to ~95% TPL-SVC v1.0.0 compliance
- Added full attribute tables for all three aggregates (ProductCost, CostVariance, CostingRun)
- Expanded all 7 business rules with detailed definitions (BR-001 through BR-007) and added 3 new rules (BR-008, BR-009, BR-010)
- Added complete REST API request/response JSON examples for all endpoints
- Expanded all 4 published events with envelope format, payload examples, and consumer tables
- Expanded all 3 consumed events with handler names, queue config, and failure handling
- Added full table definitions with columns, indexes, relationships, and retention for all 5 tables
- Populated §11 Feature Dependencies, §12 Extension Points, and §13 Migration with substantive content
- Added Guidelines Compliance block (Source of Truth Priority section was missing)
- Added consistency checks in §14.1 with pass/fail status

## Added Sections
- §1.5 Service Context: Responsibilities and Authoritative Sources tables
- §3.3.1 ProductCost: State Descriptions, Allowed Transitions tables
- §3.3.3 CostingRun: Full aggregate definition with lifecycle states
- §3.3 Child Entity: CostComponent with full attribute table and constraints
- §3.3 Value Objects: Money and DateRange
- §3.4 Enumerations: CostingType, ProductCostStatus, CostingRunStatus, CostingScope, VarianceCategory (expanded descriptions)
- §3.5 Shared Types: Money with Used By references
- §4.2 Detailed Rule Definitions: All 7 original rules + 3 new rules
- §5 Use Cases: UC-001a (ActivateProductCost), UC-005 (QueryProductCost), UC-006 (QueryVarianceSummary)
- §5.2 Use Case details: Preconditions, Main Flow, Postconditions, Business Rules Applied, Alt/Exception flows for all use cases
- §5.4 Cross-Domain Workflows: Actual Cost Capture workflow
- §6.2 Full request/response JSON for Create, Retrieve, List, Get Components
- §6.3 Full request/response JSON for Activate, Execute Costing Run, Calculate Variances, Variance Summary, Costing Run Status
- §7.2 Published Events: Full envelope and payload for all 5 events (added ProductCost.Created, ProductCost.Superseded, CostingRun.Started)
- §7.3 Consumed Events: Handler, queue config, failure handling for all 3 events
- §7.4 Event Flow Diagram (activation flow)
- §7.5 Integration Points Summary: Upstream and downstream tables
- §8.1 Storage Technology
- §8.3 Table Definitions: product_cost, cost_component, cost_variance, costing_run, pc_outbox_events
- §8.4 Reference Data Dependencies: External and internal catalogs
- §9.1 Data Classification: Sensitivity levels table
- §9.2 Access Control: Roles & Permissions table, Data Isolation
- §9.3 Compliance Requirements: SOX controls, audit trail, change control
- §10.1 Throughput and Concurrency
- §10.2 Failure Scenarios table
- §10.3 Capacity Planning
- §10.4 Maintainability: Versioning, monitoring, alerting
- §11 Feature Dependencies: Full section with register, endpoints, BFF hints, impact assessment
- §12 Extension Points: Full section with custom fields, extension events, rules, actions, hooks, API, summary
- §13.1 Data Migration: SAP CO-PC source mapping, strategy, rollback
- §13.2 Deprecation & Sunset framework
- §14.1 Consistency Checks
- §14.2 Decisions & Conflicts: DC-001, DC-002
- §14.3 Open Questions: Renumbered to Q-PC-xxx pattern, added Q-PC-004 through Q-PC-008
- §14.4 ADRs: ADR-PC-002 (CostingRun as Separate Aggregate)
- §14.5 Suite-Level ADR References
- §15.2 References
- §15.3 Status Output Requirements

## Modified Sections
- Preamble: Updated version to 2026-04-04, compliance to ~95%, added Source of Truth Priority block
- §3.3.1 ProductCost: Added plantCode, description, createdAt, updatedAt attributes; expanded business purpose
- §3.3.2 CostVariance: Added controllingArea, costingRunId, createdAt, updatedAt; expanded descriptions
- §4.1 Business Rules Catalog: Added BR-008, BR-009, BR-010
- §4.4 Reference Data Dependencies: Added cost elements, cost centers, activity types, controlling areas
- §5.2 Use Cases: Added Actor/Flow/Postconditions for UC-002, UC-003, UC-004
- §5.4 Cross-Domain Workflows: Expanded with participating services, workflow steps, business implications
- §6.1 API Overview: Added execute scope
- §6.4 OpenAPI Specification: Added version and docs URL
- §7.1 Architecture Pattern: Added rationale text
- §7.2 Published Events: Expanded from summary table to full event definitions
- §10.1 Performance: Preserved existing numbers, added throughput and concurrency
- §15.1 Glossary: Added 7 new terms
- §15.4 Change Log: Added v2.0 entry

## Removed Sections
- None (non-destructive upgrade)

## Decisions Taken
- DC-001: CostingRun modeled as separate aggregate (not property of ProductCost) due to independent lifecycle
- DC-002: CostVariance modeled as separate aggregate due to independent query patterns
- ProductCost is extensible via custom fields; CostingRun is not (internal batch record)
- CostVariance custom field extensibility left as open question (Q-PC-005)
- Used SAP CO-PC (KEKO/KEPH tables, CK11N/CK40N transactions) as reference for migration mapping

## Open Questions Raised
- Q-PC-004: Which product features depend on this service's endpoints?
- Q-PC-005: Should CostVariance support custom fields?
- Q-PC-006: WIP valuation scope boundary (co.pc vs fi.acc vs separate domain)
- Q-PC-007: Should costing runs support auto-activation of calculated costs?
- Q-PC-008: Material ledger / actual costing run support for periodic revaluation
