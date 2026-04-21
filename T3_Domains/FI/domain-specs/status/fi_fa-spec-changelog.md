# Spec Changelog - fi.fa

## Summary
- Upgraded fi.fa (Fixed Assets) domain service spec from ~60% to ~95% TPL-SVC v1.0.0 compliance
- Added missing Specification Guidelines Compliance block (preamble)
- Added Section 2 (Service Identity) with full metadata table
- Added Section 1.5 (Service Context) with responsibilities, authoritative sources, and context diagram
- Expanded Section 3 with full attribute tables (Type/Format/Description/Constraints/Required/Read-Only), added Section 3.4 (Enumerations) and Section 3.5 (Shared Types)
- Added Section 4.2 (Detailed Rule Definitions), Section 4.3 (Data Validation Rules), Section 4.4 (Reference Data Dependencies)
- Restructured Sections 5-8 to match template numbering: Use Cases (5), REST API (6), Events & Integration (7), Data Model (8)
- Added Sections 11 (Feature Dependencies), 12 (Extension Points), expanded Sections 13 (Migration), 14 (Decisions & Open Questions), 15 (Appendix)

## Added Sections
- Specification Guidelines Compliance block (Non-Negotiables, Source of Truth Priority, Style Guide)
- Section 1.5 Service Context (Responsibilities, Authoritative Sources, context diagram)
- Section 2 Service Identity (full identity table, team table)
- Section 3.4 Enumerations (AssetStatus, BookType, DepreciationMethod, RunStatus, TransactionType, TransactionStatus)
- Section 3.5 Shared Types (Money value object)
- Section 4.2 Detailed Rule Definitions (BR-AST-001, BR-AST-003, BR-RUN-001, BR-RUN-002)
- Section 4.3 Data Validation Rules (field-level + cross-field)
- Section 4.4 Reference Data Dependencies
- Section 5.1 Business Logic Placement
- Section 5.2 Use Cases in canonical format (UC-001 through UC-008 with full Actor/Preconditions/Main Flow/Postconditions)
- Section 5.4 Cross-Domain Workflows (Asset Acquisition from AP Invoice)
- Section 6 REST API with full request/response JSON examples
- Section 6.4 OpenAPI Specification reference
- Section 7 Events & Integration restructured with full event envelope examples, known consumers, consumed event handlers with queue config and failure handling
- Section 7.5 Integration Points Summary (upstream + downstream tables)
- Section 8.2 Conceptual Data Model (Mermaid ER diagram)
- Section 8.3 Table Definitions expanded to full format (columns, indexes, relationships, data retention)
- Section 8.4 Reference Data Dependencies
- Section 9.1 Data Classification (sensitivity levels table)
- Section 9.3 Compliance Requirements expanded (SOX, audit trail, segregation of duties)
- Section 10.2 Availability & Reliability (RTO/RPO, failure scenarios)
- Section 10.3 Scalability (scaling strategy, capacity planning)
- Section 10.4 Maintainability (versioning, monitoring, alerting)
- Section 11 Feature Dependencies (register, endpoints, BFF hints, impact assessment)
- Section 12 Extension Points (custom fields, extension events, extension rules, extension actions, aggregate hooks, extension API, summary matrix)
- Section 13.1 Data Migration expanded with SAP FI-AA table mapping
- Section 13.2 Deprecation & Sunset framework
- Section 14.1 Consistency Checks (7 checks, 6 Pass, 1 Fail)
- Section 14.2 Decisions & Conflicts
- Section 14.3 Open Questions (Q-FA-001 through Q-FA-007)
- Section 14.4 ADRs (ADR-FA-001 Multi-Book Accounting, ADR-FA-002 Synchronous GL Posting)
- Section 14.5 Suite-Level ADR References
- Section 15.2 References
- Section 15.3 Status Output Requirements
- Section 15.4 Change Log
- Document Review & Approval block

## Modified Sections
- Preamble: Added full meta block (Bounded Context Ref, basePackage, API Base Path, Port, Repository, Tags, Team)
- Section 3.3: All aggregate definitions now include full attribute tables with Type/Format/Description/Constraints/Required/Read-Only columns; added State Descriptions and Allowed Transitions tables; added Domain Events Emitted lists
- Section 3.3.1 Asset: Added Value Object (Money) under aggregate
- Section 3.3.2 DepreciationRun: Converted DepreciationLine from standalone aggregate to child entity
- Section 3.3.4 AssetClass: Added BR-CLS-001, BR-CLS-002 invariants
- Section 4.1: Added error codes column and new rules (BR-CLS-001, BR-CLS-002, BR-TX-001, BR-TX-002)
- Section 9.2: Restructured RBAC to include Roles & Permissions table and Data Isolation description
- Section numbering: Renumbered all sections to match TPL-SVC v1.0.0 (old section 2 -> 3, old 3 -> 5, old 5 -> 7, old 6 -> 7, old 7 -> 6, old 8 -> 8, old 9 -> 9, old 10 -> 10, old 11 -> 13, old 12 -> 14, old 13 -> 15)

## Removed Sections
- None (non-destructive upgrade)

## Decisions Taken
- DC-FA-001: Single POST /transactions endpoint with transactionType discriminator (instead of separate endpoints per type)
- DC-FA-002: Use action pattern POST /depreciation-runs/{id}:post for business operations
- DepreciationLine modeled as child entity of DepreciationRun aggregate (not standalone aggregate)
- Asset and AssetTransaction marked as extensible (custom_fields); AssetClass and DepreciationRun marked as not extensible
- Port 8440 assigned as placeholder (Q-FA-003 raised)

## Open Questions Raised
- Q-FA-001: Which product features depend on fi.fa's endpoints?
- Q-FA-002: Should fi.fa auto-create assets from AP bill events?
- Q-FA-003: Confirm port assignment for fi-fa-svc
- Q-FA-004: Should depreciation runs support partial posting?
- Q-FA-005: How should intercompany asset transfers be handled?
- Q-FA-006: Should custom_fields be added to fa_asset_transactions table?
- Q-FA-007: What is the depreciation run scheduling mechanism?
