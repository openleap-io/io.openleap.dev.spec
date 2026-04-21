# Spec Changelog - co.rpt

## Summary
- Upgraded co_rpt-spec.md from ~78% to ~95% TPL-SVC v1.0.0 compliance
- Added full attribute tables with lifecycle states and transitions for both aggregates (ReportDefinition, ReportInstance)
- Added all 6 enumeration tables (ReportType, ScheduleFrequency, OutputFormat, DefinitionStatus, InstanceStatus, GeneratedBy)
- Added detailed business rule definitions (BR-001 through BR-008) with error handling and examples
- Expanded REST API with full request/response JSON bodies for all endpoints
- Added detailed consumed event specifications with handler names, queue config, and failure handling
- Added complete table definitions with columns, indexes, relationships, and retention for 7 tables
- Populated Feature Dependencies (SS11), Extension Points (SS12), and Migration (SS13) sections

## Added Sections
- SS3.3 Aggregate Root subsections (lifecycle states, transitions, state descriptions) for ReportDefinition and ReportInstance
- SS3.4 Enumerations: ScheduleFrequency, OutputFormat, DefinitionStatus, InstanceStatus, GeneratedBy
- SS3.5 Shared Types: Money value object
- SS4.2 Detailed Rule Definitions (BR-001 through BR-008)
- SS4.4 Reference Data Dependencies
- SS5.2 Use Cases: UC-002 (InternalOrderReport), UC-003 (ProductCostReport), UC-004 (ConsolidatedReport), UC-006 (UpdateReportDefinition)
- SS6.2 Full request/response JSON for all CRUD endpoints
- SS7.2 Detailed published event definitions with envelope and payload
- SS7.3 Detailed consumed event definitions with handler, queue config, failure handling
- SS7.4 Event flow diagram (expanded)
- SS7.5 Integration Points Summary (upstream and downstream)
- SS8.1 Storage Technology
- SS8.3 Table definitions: report_definition, report_instance, cc_report_view, order_report_view, product_cost_view, co_rpt_outbox_events, period_close_status
- SS8.4 Reference Data Dependencies (external and internal)
- SS9.3 Compliance Requirements
- SS10.1 Throughput and concurrency targets
- SS10.2 Failure scenarios table
- SS10.4 Maintainability (versioning, monitoring, alerting)
- SS11 Feature Dependencies (full framework with provisional feature IDs)
- SS12 Extension Points (custom fields, events, rules, actions, hooks, API, summary)
- SS13.1 Data Migration from legacy (SAP table mapping)
- SS13.2 Deprecation & Sunset framework
- SS14.1 Consistency Checks (7 checks, 6 Pass, 1 Fail)
- SS14.2 Decisions & Conflicts (DC-001, DC-002)
- SS14.5 Suite-Level ADR References
- SS15.2 References
- SS15.3 Status Output Requirements

## Modified Sections
- Preamble: Added Source of Truth Priority block and complete Style Guide to Guidelines Compliance
- SS1.5: Added Responsibilities and Authoritative Sources tables
- SS3.3.1 ReportDefinition: Added createdAt, updatedAt, createdBy attributes; lifecycle states; invariants; domain events
- SS3.3.2 ReportInstance: Added errorMessage, controllingArea, reportType attributes; lifecycle states; domain events
- SS4.1: Added BR-006 (Unique Definition Name), BR-007 (Valid Data Sources), BR-008 (Period Format)
- SS4.3: Expanded field-level validations and added cross-field validations
- SS5.2 UC-001: Added preconditions, postconditions, business rules, alternative flows, exception flows
- SS5.2 UC-005: Added full detail (preconditions, postconditions, business rules, flows)
- SS5.4: Expanded workflow with participating services and workflow steps
- SS6.2: Expanded from bullet endpoints to full request/response documentation
- SS6.3: Added response bodies for read model queries
- SS7.2: Expanded from summary table to detailed event definitions
- SS7.3: Expanded from summary table to detailed consumed event specifications
- SS9.1: Added Rationale and Protection Measures columns
- SS9.2: Added Roles & Permissions table, Data Isolation description
- SS10.2: Added failure scenarios table
- SS14.3: Renumbered to Q-RPT-NNN pattern, added Q-RPT-004 through Q-RPT-007
- SS14.4: Expanded ADR-RPT-001 with full template format (context, alternatives, consequences)
- SS15.1: Added Aliases column, expanded glossary entries
- SS15.4: Added v2.0 changelog entry
- Meta block: Updated version to 2026-04-04, compliance to ~95%

## Removed Sections
- None (non-destructive upgrade)

## Decisions Taken
- DC-001: Report generation classified as both READ (query data) and WRITE (create instance), resolved by CQRS split
- DC-002: ReportInstance is a proper aggregate despite serving read purposes, because it has lifecycle state transitions
- ReportDefinition marked as extensible (custom fields); ReportInstance marked as non-extensible (immutability constraint)
- All extension hooks set to fail-open mode since reporting is not critical-path for transactions

## Open Questions Raised
- Q-RPT-004: Which product features depend on this service's endpoints?
- Q-RPT-005: Should read model views support cross-controlling-area queries?
- Q-RPT-006: Maximum report size before async generation is forced?
- Q-RPT-007: Should RPT store variance decomposition in ProductCostReportView?
