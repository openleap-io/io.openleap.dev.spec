# Spec Changelog - fi.inv

## Summary
- Upgraded fi.inv spec from ~60% to ~92% TPL-SVC v1.0.0 compliance
- Added Specification Guidelines Compliance block (preamble)
- Added SS1.5 Service Context with responsibilities and authoritative sources
- Added SS2 Service Identity (previously missing)
- Renumbered all sections from legacy numbering (SS2-SS13) to template numbering (SS0-SS15)
- Expanded SS3 Domain Model with full attribute tables (Type/Format/Description/Constraints/Required/Read-Only), state descriptions, allowed transitions, enumerations (SS3.4), shared types (SS3.5)
- Added SS4.2 Detailed Rule Definitions for all 10 business rules with full context/enforcement/error handling
- Added SS4.3 Data Validation Rules (field-level and cross-field) and SS4.4 Reference Data Dependencies
- Added SS5 Use Cases in canonical format (7 use cases with Actor/Preconditions/Main Flow/Postconditions)
- Expanded SS6 REST API with full request/response JSON, headers, error responses, business rules checked, events published
- Expanded SS7 Events with full envelope format, payload examples, known consumers, consumed event detail with queue config and failure handling
- Expanded SS8 Data Model with ER diagram (SS8.2), full table definitions with column/index/relationship/retention detail, outbox table (ADR-013)
- Added custom_fields JSONB column and GIN index on extensible aggregates (inv_movements, inv_revaluations)
- Expanded SS9 Security with data classification table, expanded permission matrix, compliance requirements
- Expanded SS10 Quality with throughput, concurrency, failure scenarios, scalability, maintainability, alerting
- Added SS11 Feature Dependencies (register, endpoint mapping, BFF hints, impact assessment)
- Added SS12 Extension Points (custom fields, extension events, extension rules, extension actions, aggregate hooks, extension API, summary)
- Expanded SS13 Migration with SAP movement type mapping, migration steps, deprecation framework
- Added SS14 full governance section (consistency checks, decisions, open questions, ADRs, suite-level ADR references)
- Added SS15 Appendix with expanded glossary, references, change log

## Added Sections
- Specification Guidelines Compliance block
- SS1.5 Service Context
- SS2 Service Identity
- SS3.4 Enumerations (4 enumerations with full value tables)
- SS3.5 Shared Types (Money)
- SS4.2 Detailed Rule Definitions (7 detailed rules)
- SS4.3 Data Validation Rules
- SS4.4 Reference Data Dependencies
- SS5.1 Business Logic Placement
- SS5.4 Cross-Domain Workflows
- SS6.3 Business Operations (ReverseMovement, Revaluations, Positions, Reconciliation, Cost Policies with full JSON)
- SS6.4 OpenAPI Specification reference
- SS6.5 Error Responses Summary
- SS7.1 Architecture Pattern (structured)
- SS7.3 Consumed Events (4 events with queue config, failure handling)
- SS7.4 Event Flow Diagrams
- SS7.5 Integration Points Summary
- SS8.2 Conceptual Data Model (ER diagram)
- SS8.3 Table Definitions (reformatted with column/index/relationship/retention tables)
- SS8.3 inv_outbox_events table (ADR-013)
- SS8.4 Reference Data Dependencies
- SS9.1 Data Classification
- SS9.3 Compliance Requirements (expanded)
- SS10.2 Availability & Reliability (failure scenarios)
- SS10.3 Scalability
- SS10.4 Maintainability
- SS11 Feature Dependencies (complete section)
- SS12 Extension Points (complete section)
- SS13.2 Deprecation & Sunset
- SS14.1 Consistency Checks
- SS14.2 Decisions & Conflicts
- SS14.3 Open Questions (8 questions)
- SS14.4 ADRs (3 domain ADRs)
- SS14.5 Suite-Level ADR References
- SS15.2 References
- SS15.4 Change Log

## Modified Sections
- Preamble: Added full meta fields (bounded context ref, basePackage, API base path, starter version, port, repository, tags, team)
- SS3 Domain Model: Reformatted attribute tables to template format with Type/Format/Description/Constraints/Required/Read-Only columns
- SS3.3 Aggregate Definitions: Added aggregate ID/name property tables, child entity structure, value objects
- SS5 Use Cases: Converted to canonical format tables with all fields (id, type, trigger, aggregate, domainOperation, inputs, outputs, events, rest, idempotency, errors)
- SS6 REST API: Expanded all endpoints with full request/response JSON examples
- SS7 Events: Restructured with envelope format per ADR-011 thin events pattern
- SS8 Data Model: Added version, updated_at, custom_fields columns to tables; reformatted to template table structure
- SS9 Security: Expanded RBAC matrix with full permission table
- SS10 Quality: Added structured tables for response time, throughput, concurrency

## Removed Sections
- None (non-destructive upgrade)

## Decisions Taken
- DEC-INV-001: Event-driven choreography (preserved existing decision)
- DEC-INV-002: GL posting delegation to fi.slc (preserved existing pattern)
- DEC-INV-003: Inventory position as CQRS read model (preserved existing design)
- DEC-INV-004: Quantity sign convention (always positive, direction by type)
- InventoryMovement and Revaluation marked as extensible aggregates; CostPolicy not extensible
- Port 8460 assigned (flagged as open question Q-INV-005 for confirmation)

## Open Questions Raised
- Q-INV-001: Which product features depend on fi.inv endpoints?
- Q-INV-002: Multi-currency parallel valuation support?
- Q-INV-003: FIFO/LIFO cost layer physical storage strategy?
- Q-INV-004: Should variance posting be a separate aggregate?
- Q-INV-005: Port number confirmation (8460)
- Q-INV-006: Batch revaluation async vs sync?
- Q-INV-007: Intercompany transfer handling?
- Q-INV-008: Extension event synchronicity?
