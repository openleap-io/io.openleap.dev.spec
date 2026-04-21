# Spec Changelog - fi.cnsl

## Summary
- Upgraded from ~60% to ~92% TPL-SVC v1.0.0 compliance
- Added full Specification Guidelines Compliance block
- Added SS2 Service Identity section
- Restructured and renumbered all sections from legacy numbering (2-13) to template numbering (0-15)
- Expanded SS3 Domain Model with full attribute tables (Type/Format/Description/Constraints/Required/Read-Only), lifecycle states, allowed transitions, invariants, and domain events for all aggregates
- Added complete SS3.4 Enumerations with all 7 enum types and value descriptions
- Added SS3.5 Shared Types (Money, ExchangeRateSet)
- Expanded SS4 Business Rules with detailed rule definitions (BR-GRP-001, BR-RUN-002, BR-RUN-003, BR-ELIM-001) including error codes, examples, and enforcement details
- Added SS4.3 Data Validation Rules and SS4.4 Reference Data Dependencies
- Restructured SS5 Use Cases to canonical format (8 use cases with full Actor/Preconditions/Main Flow/Postconditions)
- Added SS5.1 Business Logic Placement and SS5.4 Cross-Domain Workflows
- Expanded SS6 REST API with full request/response JSON examples for all endpoints
- Added SS6.3 Business Operations (GenerateReports) and SS6.4 OpenAPI Specification reference
- Expanded SS7 Events with full envelope format, payload examples, and known consumers for all 4 published events
- Added SS7.3 Consumed Events (fi.gl.period.closed) with queue config and failure handling
- Added SS7.5 Integration Points Summary with upstream and downstream tables
- Expanded SS8 Data Model with full table definitions (7 tables + outbox), column details, indexes, relationships, and retention policies
- Added SS8.2 Conceptual Data Model (ER diagram)
- Expanded SS9 Security with data classification table, compliance requirements (SOX, IFRS, GDPR), and compliance controls
- Expanded SS10 Quality Attributes with throughput, concurrency, availability, scalability, and maintainability sub-sections
- Added SS11 Feature Dependencies (structural framework with OPEN QUESTIONs)
- Added SS12 Extension Points (custom fields, extension events, rules, actions, hooks, API endpoints, summary)
- Expanded SS13 Migration with SAP FI-LC/EC-CS legacy mapping table
- Added SS14 Decisions & Open Questions with consistency checks, decisions, 8 open questions, 2 ADRs, suite ADR references
- Expanded SS15 Appendix with comprehensive glossary, references, and change log

## Added Sections
- Specification Guidelines Compliance block (preamble)
- SS1.5 Service Context (responsibilities, authoritative sources, context diagram)
- SS2 Service Identity (full identity table and team)
- SS3.3 Aggregate attribute tables (full format with Type/Format/Constraints/Required/Read-Only)
- SS3.3 Lifecycle state descriptions and allowed transitions tables
- SS3.3 Child entity and value object detail sections
- SS3.4 Enumerations (7 enum definitions)
- SS3.5 Shared Types (Money, ExchangeRateSet)
- SS4.2 Detailed Rule Definitions
- SS4.3 Data Validation Rules
- SS4.4 Reference Data Dependencies
- SS5.1 Business Logic Placement
- SS5.2 Canonical use case format (8 use cases)
- SS5.4 Cross-Domain Workflows
- SS6.2 Full request/response JSON for all endpoints
- SS6.3 Business Operations
- SS6.4 OpenAPI Specification reference
- SS7.1 Architecture Pattern (hybrid, rationale)
- SS7.2 Full event envelope format for all events
- SS7.3 Consumed Events (fi.gl.period.closed)
- SS7.4 Event Flow Diagrams
- SS7.5 Integration Points Summary
- SS8.1 Storage Technology
- SS8.2 Conceptual Data Model (ER diagram)
- SS8.3 Full table definitions (7 tables + outbox with columns, indexes, relationships, retention)
- SS8.4 Reference Data Dependencies
- SS9.1 Data Classification
- SS9.3 Compliance Requirements
- SS10.2 Availability & Reliability
- SS10.3 Scalability
- SS10.4 Maintainability
- SS11 Feature Dependencies (full section)
- SS12 Extension Points (full section with all 5 types)
- SS13.2 Deprecation & Sunset
- SS14.1 Consistency Checks
- SS14.2 Decisions & Conflicts
- SS14.3 Open Questions (8 questions)
- SS14.4 ADRs (2 domain ADRs)
- SS14.5 Suite-Level ADR References
- SS15.3 Status Output Requirements
- SS15.4 Change Log

## Modified Sections
- Preamble: Added full meta block with bounded context, service ID, base package, API path, port, repository, tags, team
- SS0.4: Added CONCEPTUAL_STACK.md, EVENT_STANDARDS.md, TECHNICAL_STANDARDS.md references
- SS1.4: Added SAP FI-LC/EC-CS equivalence note
- SS3.3: Restructured aggregate definitions from simplified tables to full TPL-SVC format
- SS5: Restructured from narrative use cases to canonical format with UC table + detail
- SS6: Restructured from bullet-point endpoints to full REST API format with request/response JSON
- SS7: Merged old sections 5 (Integration) and 6 (Events) into unified SS7
- SS8: Expanded existing SQL DDL to full table definition format
- SS9: Expanded from minimal RBAC to full security section
- SS10: Expanded from basic performance numbers to full quality attributes
- SS13: Expanded migration with SAP EC-CS mapping table

## Removed Sections
- None (non-destructive upgrade)

## Decisions Taken
- DC-002: API base path normalized to `/api/fi/cnsl/v1` (matching domain short code convention)
- DC-003: Confirmed Temporal for workflow orchestration (documented as ADR-CNSL-002)
- ConsolidationGroup gets DRAFT/ACTIVE/INACTIVE/ARCHIVED lifecycle (was only isActive boolean)
- ConsolidationRun gets FAILED state (was not in original spec)
- Added groupCode business key to ConsolidationGroup (was missing, needed for dual-key pattern)
- Added custom_fields JSONB to extensible aggregates (ConsolidationGroup, ConsolidationRun)

## Open Questions Raised
- Q-CNSL-001: Feature dependency register (no feature specs yet)
- Q-CNSL-002: Extension action candidates for product UI
- Q-CNSL-003: Incremental consolidation support
- Q-CNSL-004: Goodwill impairment testing scope
- Q-CNSL-005: Port assignment for fi-cnsl-svc
- Q-CNSL-006: Multi-level sub-consolidation support
- Q-CNSL-007: Exchange rate sourcing strategy
- Q-CNSL-008: Equity method separate aggregate decision
