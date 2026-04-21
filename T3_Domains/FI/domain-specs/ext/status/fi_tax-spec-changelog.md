# Spec Changelog - fi.tax

## Summary
- Upgraded from ~65% to ~95% TPL-SVC v1.0.0 compliance
- Renumbered all sections from original numbering to standard SS0-SS15 structure
- Added SS2 Service Identity section with full metadata
- Added SS1.5 Service Context with responsibilities, authoritative sources, and context diagram
- Expanded SS3 Domain Model with full attribute tables (Type/Format/Description/Constraints/Required/Read-Only), lifecycle state tables, and enumeration tables (SS3.4)
- Added SS4.2 Detailed Rule Definitions, SS4.3 Data Validation Rules, SS4.4 Reference Data Dependencies
- Restructured SS5 Use Cases to canonical format with Actor/Preconditions/Main Flow/Postconditions
- Expanded SS6 REST API with full request/response JSON examples, error responses, and SS6.4 OpenAPI reference
- Expanded SS7 Events with full envelope format, payload examples, consumed event detail with queue config and failure handling, and SS7.5 Integration Points Summary
- Restructured SS8 Data Model from SQL DDL to template-format table definitions with columns, indexes, relationships, retention; added ER diagram, outbox table, version/updated_at columns
- Expanded SS9 Security with data classification, compliance controls, SOX requirements
- Expanded SS10 Quality Attributes with availability, scalability, and maintainability sections
- Added SS11 Feature Dependencies (structural framework with planned entries)
- Added SS12 Extension Points (all 5 types: custom fields, events, rules, actions, hooks)
- Expanded SS13 Migration with SAP source mapping table and deprecation framework
- Restructured SS14 Decisions with consistency checks, open questions table, suite ADR references
- Expanded SS15 Appendix with full glossary, references, status output requirements, change log

## Added Sections
- SS1.5 Service Context
- SS2 Service Identity (full table)
- SS3.4 Enumerations (7 enum tables: TaxType, SourceType, AssessmentStatus, ReturnStatus, PartyType, BasisType, ReturnFrequency)
- SS3.5 Shared Types (Money)
- SS4.2 Detailed Rule Definitions (6 full rule definitions)
- SS4.3 Data Validation Rules (field-level and cross-field)
- SS4.4 Reference Data Dependencies
- SS5.1 Business Logic Placement table
- SS5.4 Cross-Domain Workflows (2 workflows)
- SS6.4 OpenAPI Specification reference
- SS7.1 Architecture Pattern (with rationale)
- SS7.3 Consumed Events (3 events with full detail)
- SS7.5 Integration Points Summary (upstream + downstream tables)
- SS8.2 Conceptual Data Model (ER diagram)
- SS8.3 fi_tax_outbox_events table
- SS8.4 Reference Data Dependencies (external + internal catalogs)
- SS9.1 Data Classification
- SS9.3 Compliance Requirements (expanded with controls)
- SS10.2 Availability & Reliability
- SS10.3 Scalability
- SS10.4 Maintainability
- SS11 Feature Dependencies (complete section)
- SS12 Extension Points (complete section with all 5 types)
- SS13.2 Deprecation & Sunset
- SS14.1 Consistency Checks (7 checks, all evaluated)
- SS14.3 Open Questions (8 questions)
- SS14.5 Suite-Level ADR References
- SS15.3 Status Output Requirements
- SS15.4 Change Log

## Modified Sections
- Preamble: Added Port, Repository, Tags, Team, Bounded Context Ref, OpenLeap Starter Version; updated Guidelines Compliance to full template format
- SS3 Domain Model: All aggregate attribute tables expanded to full template format (added Type, Format, Constraints, Required, Read-Only columns); added lifecycle state/transition tables
- SS4 Business Rules: Added error codes to catalog; expanded with full detailed definitions
- SS5 Use Cases: Restructured to canonical format tables; added UC-007 (CreateTaxCode), UC-008 (ListTaxAssessments), UC-009 (ListTaxReturns)
- SS6 REST API: Expanded all endpoints to full request/response JSON format
- SS7 Events: Restructured to template format with envelope, payload structure, and known consumers table
- SS8 Data Model: Converted SQL DDL to template-format tables; added version and updated_at columns to all tables; added custom_fields JSONB to extensible aggregates
- SS9 Security: Restructured access control; added data classification and compliance controls
- SS14 Decisions: Added consistency checks, renumbered open questions to Q-TAX-NNN pattern

## Removed Sections
- None (non-destructive upgrade)

## Decisions Taken
- DC-001: Synchronous tax assessment API confirmed (was already decided as ADR-001)
- DC-002: TaxRule modeled as child entity of TaxCode (not separate aggregate)
- DC-003: Section numbering aligned to TPL-SVC v1.0.0 standard
- TaxAssessment and TaxReturn marked as extensible (custom_fields); TaxCode and TaxConfig not extensible
- Extension hook-003 (pre-file TaxReturn) is fail-closed; all others fail-open

## Open Questions Raised
- Q-TAX-001: Feature dependency register (awaits feature specs)
- Q-TAX-002: fi.tax vs fi.pst interaction
- Q-TAX-003: Cash-basis tax recognition in phase 1
- Q-TAX-004: Tax rate changes for in-flight documents
- Q-TAX-005: Withholding tax workflow detail
- Q-TAX-006: Multi-tax jurisdiction stacking
- Q-TAX-007: Port assignment confirmation
- Q-TAX-008: Multiple GL accounts per tax type
