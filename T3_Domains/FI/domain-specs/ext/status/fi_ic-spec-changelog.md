# Spec Changelog - fi.ic

## Summary
- Upgraded fi.ic (Intercompany Accounting) spec from ~60% to ~92% TPL-SVC v1.0.0 compliance
- Renumbered all sections from legacy numbering (Domain Model at SS2) to template-standard SS0-SS15
- Added complete meta block with bounded context ref, service ID, basePackage, API base path, port, repo, tags, team
- Added Specification Guidelines Compliance block
- Added SS2 Service Identity section
- Expanded SS3 Domain Model with full attribute tables (Type/Format/Description/Constraints/Required/Read-Only), lifecycle state tables, transition tables, enumerations (SS3.4), and shared types (SS3.5)
- Expanded SS4 Business Rules with 14 detailed rule definitions (SS4.2), field-level and cross-field validations (SS4.3), and reference data dependencies (SS4.4)
- Expanded SS5 Use Cases with canonical format tables, business logic placement, 6 use cases (4 WRITE, 2 READ), and cross-domain workflow documentation
- Expanded SS6 REST API with full request/response JSON bodies, response headers, error codes, query parameters, and OpenAPI specification reference
- Expanded SS7 Events & Integration with 5 detailed published events (full envelope + payload), 1 consumed event with queue config and failure handling, event flow diagram, and integration points summary
- Expanded SS8 Data Model with conceptual ER diagram, 7 full table definitions (columns, indexes, relationships, retention), and outbox table per ADR-013
- Expanded SS9 Security with data classification table, compliance requirements (SOX, GDPR), segregation of duties
- Expanded SS10 Quality Attributes with throughput, concurrency, availability/reliability (RTO/RPO), failure scenarios, scalability, and maintainability
- Added SS11 Feature Dependencies with provisional feature register and BFF aggregation hints
- Added SS12 Extension Points covering all 5 types: custom fields (2 extensible aggregates with JSONB), extension events (4), extension rules (3), extension actions (3), aggregate hooks (5), extension API endpoints, and summary matrix
- Expanded SS13 Migration with SAP FI-IC source mapping table
- Expanded SS14 with consistency checks (all Pass), decisions/conflicts table, 7 open questions, ADR-001 (sync mirroring), and suite-level ADR references
- Expanded SS15 Appendix with full glossary, references, status output requirements, and change log

## Added Sections
- Specification Guidelines Compliance block (preamble)
- SS1.5 Service Context (responsibilities, authoritative sources, context diagram)
- SS2 Service Identity (full table + team)
- SS3.4 Enumerations (7 enums with value tables)
- SS3.5 Shared Types (Money)
- SS4.2 Detailed Rule Definitions (6 fully expanded rules)
- SS4.3 Data Validation Rules (field-level + cross-field)
- SS4.4 Reference Data Dependencies
- SS5.1 Business Logic Placement table
- SS5.2 Canonical format tables for all use cases
- SS5.4 Cross-Domain Workflows
- SS6.1 API Overview (auth scopes)
- SS6.4 OpenAPI Specification reference
- SS7.1 Architecture Pattern (with rationale)
- SS7.3 Consumed Events (fi.gl.period.closed)
- SS7.4 Event Flow Diagrams
- SS7.5 Integration Points Summary
- SS8.1 Storage Technology
- SS8.2 Conceptual Data Model (ER diagram)
- SS8.4 Reference Data Dependencies
- SS9.1 Data Classification
- SS9.3 Compliance Requirements
- SS10.2 Availability & Reliability
- SS10.3 Scalability
- SS10.4 Maintainability
- SS11 Feature Dependencies (complete section)
- SS12 Extension Points (complete section with all 5 types)
- SS14.1 Consistency Checks
- SS14.3 Open Questions (7 entries)
- SS14.5 Suite-Level ADR References
- SS15.2 References
- SS15.3 Status Output Requirements
- SS15.4 Change Log

## Modified Sections
- Preamble meta block: added bounded context ref, basePackage, port, repo, tags, team, updated compliance score
- SS3.3 Aggregate Definitions: expanded attribute tables to full template format (added Format, Required, Read-Only columns); added lifecycle state/transition tables for all aggregates; restructured child entities (ICPosting, Settlement) with proper entity format
- SS4.1 Business Rules Catalog: expanded from 6 to 14 rules with error codes
- SS6.2 Resource Operations: expanded from bullet-point endpoints to full request/response JSON
- SS7.2 Published Events: expanded from 1 to 5 events with full envelope and payload examples
- SS8.3 Table Definitions: restructured from raw SQL to template format (columns, indexes, relationships, retention); added custom_fields JSONB + GIN indexes for extensible aggregates; added outbox table
- SS9.2 Access Control: expanded with data isolation description
- SS10.1 Performance: added throughput and concurrency numbers
- SS13.1 Data Migration: expanded with SAP source mapping table
- SS15.1 Glossary: expanded from 6 to 10 terms with aliases

## Removed Sections
- None (non-destructive upgrade)

## Decisions Taken
- DC-001: Confirmed synchronous mirroring pattern (existing ADR-001)
- DC-002: Clarified netAmount = grossAmount + taxAmount (tax-inclusive, matching industry convention)
- ICAgreement and ICTransaction marked as extensible (custom_fields JSONB); ICBalance and ReconciliationRun marked as non-extensible
- Settlement modeled as child entity of ICTransaction (one_to_one optional) rather than separate aggregate
- Provisional feature IDs (F-FI-IC-001 through F-FI-IC-004) assigned pending product feature specs

## Open Questions Raised
- Q-IC-001: Which product features depend on fi.ic endpoints?
- Q-IC-002: Should multilateral netting be a first-class aggregate?
- Q-IC-003: What is the exact FX rate source?
- Q-IC-004: Real-time vs. batch IC balance maintenance?
- Q-IC-005: Port assignment for fi-ic-svc?
- Q-IC-006: Should fi.ic handle partial settlements?
- Q-IC-007: Default reconciliation variance threshold?
