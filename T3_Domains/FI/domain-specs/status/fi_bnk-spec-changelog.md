# Spec Changelog - fi.bnk

## Summary
- Upgraded fi_bnk-spec.md from ~60% to ~95% TPL-SVC v1.0.0 compliance
- Added missing section 2 (Service Identity) with full metadata table
- Added section 1.5 (Service Context) with responsibilities, authoritative sources, and context diagram
- Expanded section 3 (Domain Model) with full template-compliant attribute tables, lifecycle states, state descriptions, allowed transitions, and domain events for all 5 aggregates
- Added section 3.4 (Enumerations) with 9 enum definitions and section 3.5 (Shared Types) with Money VO
- Added section 4.2 (Detailed Rule Definitions) for key business rules, section 4.3 (Data Validation Rules), and section 4.4 (Reference Data Dependencies)
- Restructured section 5 (Use Cases) to canonical format with 9 use cases (6 WRITE, 3 READ)
- Expanded section 6 (REST API) with full request/response JSON examples for all endpoints
- Expanded section 7 (Events) with detailed event specifications including envelopes, payloads, and consumer tables
- Added section 8 table definitions in template format with columns, indexes, relationships, and retention
- Added section 11 (Feature Dependencies) and section 12 (Extension Points) -- both entirely new
- Added Specification Guidelines Compliance block
- Added consistency checks in section 14.1

## Added Sections
- Specification Guidelines Compliance block (preamble)
- Section 1.5 Service Context
- Section 2 Service Identity (full table)
- Section 3.4 Enumerations (9 enums)
- Section 3.5 Shared Types (Money VO)
- Section 4.2 Detailed Rule Definitions (6 rules)
- Section 4.3 Data Validation Rules
- Section 4.4 Reference Data Dependencies
- Section 5.1 Business Logic Placement
- Section 6.4 OpenAPI Specification
- Section 6.5 Error Responses (consolidated)
- Section 7.1 architecture pattern detail
- Section 7.4 Event Flow Diagrams
- Section 7.5 Integration Points Summary (upstream + downstream)
- Section 8.1 Storage Technology
- Section 8.2 Conceptual Data Model (ER diagram with column details)
- Section 10.3 Scalability
- Section 10.4 Maintainability
- Section 11 Feature Dependencies (entire section)
- Section 12 Extension Points (entire section with 5 extension types)
- Section 14.1 Consistency Checks
- Section 14.5 Suite-Level ADR References
- Section 15.3 Status Output Requirements

## Modified Sections
- Preamble: Added Bounded Context Ref, basePackage, Port, Repository, Tags, Team, OpenLeap Starter Version
- Section 0: Minor wording alignment, preserved all content
- Section 1: Added section 1.5 Service Context with full detail
- Section 3: Renumbered from original section 2; added full attribute tables with Type/Format/Description/Constraints/Required/Read-Only; added lifecycle tables, state descriptions, allowed transitions, invariant references, and domain events emitted for all aggregates
- Section 4: Added detailed rule definitions, data validation tables, reference data dependencies
- Section 5: Renumbered from original section 3; restructured to canonical use case format with 9 use cases
- Section 6: Renumbered from original section 5; expanded all endpoints with request/response JSON
- Section 7: Renumbered from original section 6; expanded all events with envelopes and payloads
- Section 8: Renumbered from original section 7; converted SQL DDL to template table format; added custom_fields columns for extensible aggregates
- Section 9: Renumbered from original section 8; added Overall Classification and expanded compliance controls
- Section 10: Renumbered from original section 9; added scalability and maintainability
- Section 13: Renumbered from original section 10; added SAP FI-BL migration mapping, rollback plan
- Section 14: Renumbered from original section 11; renumbered ADRs to ADR-BNK-NNN pattern; added consistency checks and suite-level references
- Section 15: Renumbered from original section 12; expanded glossary and references

## Removed Sections
- None (non-destructive upgrade)

## Decisions Taken
- DC-001: Renumbered all sections to match TPL-SVC v1.0.0 template structure
- DC-002: Added Service Identity as section 2; moved Domain Model to section 3
- DC-003: Added tenant_id to bnk_statement_lines DDL for RLS consistency (was missing)
- Marked Voucher, PaymentInstruction, and BankAccount as extensible (custom_fields); ImportBatch, BankStatement, and BankStatementLine as non-extensible
- Used fail-closed for payment-related hooks (sanctions screening) and fail-open for voucher creation hooks

## Open Questions Raised
- Q-BNK-001: PSD2/real-time bank feed support (preserved from original Q-001)
- Q-BNK-002: Reconciliation feature ownership (preserved from original Q-002)
- Q-BNK-003: Feature dependency completeness (new)
- Q-BNK-004: Voucher splitting as core vs. extension (new)
- Q-BNK-005: Port assignment confirmation (new)
- Q-BNK-006: Multi-currency statement support (new)
- Q-BNK-007: Parser version strategy (new)
