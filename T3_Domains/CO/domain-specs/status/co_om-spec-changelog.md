# Spec Changelog - co.om

## Summary
- Upgraded co_om-spec.md from ~83% to ~95% TPL-SVC v1.0.0 compliance
- Added Source of Truth Priority block to Specification Guidelines Compliance
- Expanded all 4 aggregates with full attribute tables, state descriptions, allowed transitions, child entities, and value objects
- Added 8 complete enumeration definitions (from 1 partial)
- Added detailed definitions for all 12 business rules (BR-001 through BR-012)
- Expanded REST API with full request/response JSON examples for all endpoints
- Added event envelope/payload structures for all 6 published events
- Added full table definitions with columns, indexes, and retention for 8 tables including outbox
- Populated §11 Feature Dependencies, §12 Extension Points (all 5 types), and §13 Migration with SAP CO-OM mapping
- Added consistency checks, decisions & conflicts, suite-level ADR references

## Added Sections
- §3.3 Child Entity sections: AllocationReceiver, AllocationResult, SettlementReceiver (with attribute tables)
- §3.3 Value Object: Money
- §3.4 Enumerations: DriverType, RuleStatus, RunStatus, ActivityCategory, ActivityTypeStatus, SettlementType, SettlementRuleStatus, ReceiverType
- §3.5 Shared Types: Money (with Used By references)
- §4.2 Detailed rule definitions for BR-001 through BR-005, BR-007 through BR-012 (BR-006 existed)
- §5.2 Use cases UC-002 (UpdateAllocationRule), UC-007 (GetAllocationRunResults READ), UC-008 (ListAllocationRules READ), UC-009 (ListActivityTypes READ)
- §5.2 Preconditions/Postconditions/Business Rules Applied for all use cases
- §6.2 Full CRUD endpoints with request/response JSON for allocation rules, activity types, settlement rules
- §7.2 Event envelopes and payload examples for all published events
- §7.3 Queue configuration and failure handling for consumed events
- §7.4 Event Flow Diagram
- §7.5 Integration Points Summary (upstream dependencies, downstream consumers)
- §8.1 Storage Technology
- §8.3 Full table definitions (8 tables: om_allocation_rule, om_allocation_receiver, om_allocation_run, om_allocation_result, om_activity_type, om_settlement_rule, om_settlement_receiver, om_outbox_events)
- §10.2 Failure Scenarios table
- §10.3 Capacity Planning
- §10.4 Maintainability
- §11 Feature Dependencies (full framework with indicative features)
- §12 Extension Points (custom fields, extension events, extension rules, extension actions, aggregate hooks, extension API, summary)
- §13.1 SAP CO-OM migration mapping and strategy
- §13.2 Deprecation & Sunset framework
- §14.1 Consistency Checks (7 checks)
- §14.2 Decisions & Conflicts (structured)
- §14.5 Suite-Level ADR References
- §15.2 References
- §15.3 Status Output Requirements

## Modified Sections
- Preamble: Added Source of Truth Priority block to Guidelines Compliance
- Meta: Updated version to 2026-04-04, compliance to ~95%
- §3.3.1 AllocationRule: Added Aggregate Root heading, State Descriptions table, Allowed Transitions table, additional events
- §3.3.2 AllocationRun: Added Aggregate Root heading, State Descriptions, Allowed Transitions, dryRun attribute
- §3.3.3 ActivityType: Added Aggregate Root heading, State Descriptions, Allowed Transitions, controllingArea attribute, BR-011/BR-012
- §3.3.4 SettlementRule: Added Aggregate Root heading, State Descriptions, Allowed Transitions
- §4.1 Business Rules Catalog: Added BR-011, BR-012
- §4.3 Data Validation Rules: Expanded with more fields and cross-field validations
- §4.4 Reference Data Dependencies: Restructured with External Catalogs Required and Internal Code Lists tables
- §5.2 All existing use cases: Added full Preconditions/Postconditions/Business Rules Applied/Alternative Flows/Exception Flows
- §6.2-6.3: Expanded from bullet endpoints to full request/response JSON format
- §7.2-7.3: Expanded from summary tables to full event detail with envelopes
- §8.2 Renamed from Entity Definitions to full table definitions
- §9.1: Added Overall Classification and expanded sensitivity table
- §9.2: Added Roles & Permissions table, Data Isolation
- §9.3: Added regulation checklist, compliance controls
- §10.1: Added Throughput, Concurrency
- §14: Restructured to match template (14.1-14.5)
- §15.4 Change Log: Added upgrade entry

## Removed Sections
- None (non-destructive upgrade)

## Decisions Taken
- DC-001: OM as orchestrator (preserved existing ADR-OM-001)
- DC-002: Hybrid sync/async for co.cca integration (sync for reads, async for postings)
- AllocationRule, ActivityType, SettlementRule marked extensible (custom_fields); AllocationRun and AllocationResult not extensible (system-generated records)
- 10 year data retention for SOX compliance on all financial records

## Open Questions Raised
- Q-OM-004: Which product features depend on this service's endpoints?
- Q-OM-005: Should the service support multi-currency allocations within a single run?
- Q-OM-006: What is the exact port assignment for co-om-svc?
- Q-OM-007: Should AllocationRule support effective-dated versioning?
