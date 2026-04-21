# Spec Changelog - fi.coa

## Summary
- Expanded §3.4 Enumerations with detailed per-value description tables for all 5 enums (ChartStructureStatus, AccountingFramework, NodeType, AccountChangeRequestType, AccountChangeRequestStatus)
- Added §3.5 Shared Types with EffectiveDateRange value object
- Added detailed business rule definitions for BR-COA-002, BR-COA-003, BR-COA-005, BR-COA-006 (with Business Context, Validation Logic, Error Handling, Examples)
- Added cross-field validations to §4.3
- Added Actor/Preconditions/Main Flow/Postconditions/Business Rules/Exception Flows to all 6 use cases
- Added §5.4 Cross-Domain Workflows (GL Account Change Request choreography)
- Expanded §6 REST API with full request/response JSON for Add Node, Add Mapping, Publish, Submit Change Request; added §6.3 Business Operations
- Added event envelope format (ADR-011 thin events) to all 5 published events in §7.2; added ChartStructureCreated event
- Expanded §7.3 consumed events with queue configuration and failure handling (retry + DLQ)
- Added tenant_id to child tables (coa_chart_nodes, coa_node_mappings, coa_account_change_requests); added custom_fields JSONB + GIN index to coa_chart_structures; expanded coa_outbox_events with full column definitions
- Added §8.4 Reference Data Dependencies (external catalogs + internal code lists)
- Added failure scenarios table and capacity planning to §10
- Expanded §11 with endpoint-to-feature mapping, BFF aggregation hints, and impact assessment
- Expanded §12 to cover all 5 extension types: custom fields (§12.2), extension events (§12.3), extension rules (§12.4), extension actions (§12.5), aggregate hooks (§12.6), extension API endpoints (§12.7), summary with guidelines (§12.8)
- Added §14.5 Suite-Level ADR References
- Added §15.3 Status Output Requirements
- Compliance raised from ~92% to ~95%

## Added Sections
- §3.5 Shared Types (EffectiveDateRange)
- §5.4 Cross-Domain Workflows (GL Account Change Request)
- §6.3 Business Operations
- §8.4 Reference Data Dependencies
- §12.2 Custom Fields (extension-field)
- §12.4 Extension Rules
- §12.5 Extension Actions
- §12.7 Extension API Endpoints
- §12.8 Extension Points Summary & Guidelines
- §14.5 Suite-Level ADR References
- §15.3 Status Output Requirements

## Modified Sections
- §3.4 Enumerations: Expanded from summary table to detailed per-value tables with descriptions
- §4.2 Detailed Rule Definitions: Added BR-COA-002, BR-COA-003, BR-COA-005, BR-COA-006
- §4.3 Data Validation Rules: Added cross-field validations section, additional field validations
- §5.2 Use Cases: Added Actor/Preconditions/Main Flow/Postconditions to all 6 UCs
- §6.2 Resource Operations: Full request/response JSON for all endpoint types
- §7.2 Published Events: Added event envelope format to all events; added ChartStructureCreated
- §7.3 Consumed Events: Expanded with queue config, business logic, failure handling
- §8.3 Table Definitions: Added tenant_id to child tables, custom_fields to chart_structures, expanded outbox table
- §9.1 Data Classification: Added overall classification and rationale column
- §10.3 Availability: Added failure scenarios table
- §10.5 Scalability: Added capacity planning estimates
- §11: Added §11.3-§11.5 content (endpoint mapping, BFF hints, impact assessment)
- §12: Restructured to cover all 5 extension types per TPL-SVC v1.0.0
- §14.1: Updated to template's 7 consistency checks with Pass/Fail status
- §14.3: Added OQ-COA-005, OQ-COA-006, OQ-COA-007

## Removed Sections
- None (non-destructive upgrade)

## Decisions Taken
- ChartStructure is extensible with custom_fields; ChartNode and AccountChangeRequest are not (rationale documented in §12.2)
- Extension events use distinct `fi.coa.ext.*` routing keys to avoid duplication with integration events in §7
- AccountingFramework enum values aligned with SAP FI-GL chart types (operating/group/country → IFRS/GAAP/LOCAL/MGMT)
- HOOK-COA-004 added for pre-create enrichment (auto-populate default node skeleton based on framework)

## Open Questions Raised
- OQ-COA-005: Feature ID mapping pending Phase 3 feature specs
- OQ-COA-006: Should ChartNode be extensible with custom_fields?
- OQ-COA-007: Should fi.coa support bulk import of chart nodes from CSV/Excel?
