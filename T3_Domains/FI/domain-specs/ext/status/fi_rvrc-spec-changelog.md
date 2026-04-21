# Spec Changelog - fi.rvrc

## Summary
- Upgraded from ~65% to ~95% TPL-SVC v1.0.0 compliance
- Renumbered all sections from non-standard numbering to §0-§15 template layout
- Added §2 Service Identity table with full metadata
- Expanded §3 Domain Model with template-format attribute tables (Type/Format/Description/Constraints/Required/Read-Only), State Descriptions, Allowed Transitions, Domain Events Emitted, formal Child Entity and Value Object sections
- Added §3.4 Enumerations (11 enums) and §3.5 Shared Types (Money)
- Added §4.2 detailed rule definitions for key business rules with error handling and examples
- Added §4.3 Data Validation Rules and §4.4 Reference Data Dependencies
- Restructured §5 Use Cases to canonical format with UC tables, added §5.1 Business Logic Placement, §5.4 Cross-Domain Workflows
- Expanded §6 REST API with full request/response JSON examples, error responses, and §6.4 OpenAPI reference
- Expanded §7 Events with detailed event envelopes, consumer tables, consumed event handler details with queue config and failure handling, §7.5 Integration Points Summary
- Expanded §8 Data Model with §8.2 ER diagram, template-format column/index tables, outbox table, §8.4 Reference Data Dependencies; added version/updated_at/custom_fields columns
- Expanded §9 Security with §9.1 Data Classification, detailed compliance controls
- Expanded §10 Quality Attributes with §10.2 Availability, §10.3 Scalability, §10.4 Maintainability
- Added §11 Feature Dependencies (structural framework with open questions)
- Added §12 Extension Points (all 5 types: custom fields, events, rules, actions, hooks)
- Expanded §13 Migration with SAP RAR mapping table and deprecation framework
- Added §14.1 Consistency Checks, expanded §14.2 Decisions, added §14.3 Open Questions (8 items), §14.5 Suite ADR References
- Added §15.3 Status Output Requirements, §15.4 Change Log
- Added full Guidelines Compliance block (Non-Negotiables, Source of Truth Priority, Style Guide)
- Added §1.5 Service Context with responsibilities, authoritative sources, and context diagram

## Added Sections
- §1.5 Service Context
- §2 Service Identity (full table)
- §3.3.x formal Child Entity sections (PerformanceObligation, AllocationSnapshot, ScheduleLine)
- §3.3.x Value Objects (Money, ContractTerms)
- §3.4 Enumerations (ContractStatus, PobStatus, ScheduleStatus, LineStatus, RecognitionMethod, RecognitionBasis, MeasureType, RecognitionFrequency, ModificationType, AdjustmentPolicy, ModificationStatus)
- §3.5 Shared Types (Money)
- §4.2 Detailed Rule Definitions (BR-CTR-001 through BR-MOD-001)
- §4.3 Data Validation Rules
- §4.4 Reference Data Dependencies
- §5.1 Business Logic Placement table
- §5.4 Cross-Domain Workflows (Revenue Recognition Posting, Contract Auto-Creation)
- §6.2 Full request/response JSON for all endpoints
- §6.3 Business Operations (allocate, generate, run, modify)
- §6.4 OpenAPI Specification reference
- §7.2 Detailed event envelopes for all published events
- §7.3 Detailed consumed event handlers with queue config
- §7.5 Integration Points Summary
- §8.2 Conceptual Data Model (ER diagram)
- §8.3 Template-format table definitions (all 7 tables including outbox)
- §8.4 Reference Data Dependencies
- §9.1 Data Classification
- §9.3 Compliance Controls (expanded)
- §10.2 Availability & Reliability
- §10.3 Scalability
- §10.4 Maintainability
- §11 Feature Dependencies (complete section)
- §12 Extension Points (complete section, all 8 sub-sections)
- §13.2 Deprecation & Sunset
- §14.1 Consistency Checks
- §14.3 Open Questions (Q-RVRC-001 through Q-RVRC-008)
- §14.4 ADR-002 (Choreography over Orchestration)
- §14.5 Suite-Level ADR References
- §15.2 References
- §15.3 Status Output Requirements
- §15.4 Change Log

## Modified Sections
- Preamble: Added Bounded Context Ref, Port, Repository, Tags, Team; updated compliance to ~95%
- Guidelines Compliance: Replaced minimal block with full template-standard block
- §3.3 Aggregate Definitions: Expanded attribute tables to template format with Type/Format columns, added State Descriptions and Allowed Transitions tables, added Domain Events Emitted lists
- §5 Use Cases: Added canonical format tables (id, type, trigger, etc.) to all existing use cases; added UC-007 ListContracts and UC-008 GetScheduleLines
- §6 REST API: Expanded from bullet format to full request/response JSON with headers, business rules, events, error responses
- §7.1 Architecture Pattern: Added message broker, follows suite pattern fields
- §8.1 Storage Technology: Added ADR references
- §8.3 Table Definitions: Converted from SQL DDL to template-format column/index tables; added version, updated_at, custom_fields columns
- §9.2 Access Control: Expanded permission matrix
- §14.2 Decisions: Added DC-001 through DC-003

## Removed Sections
- None (non-destructive upgrade)

## Decisions Taken
- DC-001: Standardized API base path to `/api/fi/rvrc/v1` (consistent with domain short code)
- DC-002: Renumbered sections to TPL-SVC v1.0.0 standard
- Contract aggregate marked as extensible (custom_fields); RecognitionSchedule and ContractModification marked as non-extensible
- ADR-002: Confirmed choreography pattern over orchestration for GL posting flow

## Open Questions Raised
- Q-RVRC-001: Feature dependency register incomplete (feature specs not yet created)
- Q-RVRC-002: Variable consideration default method (expected value vs. most likely amount)
- Q-RVRC-003: Port assignment for fi-rvrc-svc
- Q-RVRC-004: Usage-based POB integration with metering service
- Q-RVRC-005: Multi-currency contract handling
- Q-RVRC-006: Percentage-of-completion (POC) cost-based measurement support
- Q-RVRC-007: Reconciliation process for deferred revenue subledger
- Q-RVRC-008: Separate RVR_OPERATOR role for SOX segregation of duties
