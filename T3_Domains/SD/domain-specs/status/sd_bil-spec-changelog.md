# Spec Changelog - sd.bil

## Summary
- Added full attribute tables with Type/Format/Description/Constraints/Required/Read-Only for all 5 entities (BillingDocument, BillingLine, BillingBlock, BillingPlan, BillingPlanItem)
- Added §3.4 Enumerations for BillingType, BillingStatus, BillingPlanType, BillingPlanStatus, BillingPlanItemStatus, BillingBlockType
- Added §3.5 Shared Types (Money value object with database mapping)
- Added §4.2 Detailed Rule Definitions for all 7 business rules (BR-BIL-001 to BR-BIL-007)
- Added §4.3 Data Validation Rules (field-level and cross-field validations)
- Added §4.4 Reference Data Dependencies table
- Added §5.1 Business Logic Placement table; expanded use cases with Actor/Preconditions/Main Flow/Postconditions; added UC-BIL-005 through UC-BIL-009; added §5.4 Cross-Domain Workflows (Order-to-Cash, Rebate Settlement)
- Expanded §6 with full request/response JSON examples for all endpoints; added §6.4 OpenAPI reference
- Expanded §7.2 with event envelope format and known consumers; expanded §7.3 with handler names, queue config, failure handling; added §7.5 Integration Points Summary
- Replaced §8 bullet table list with full ER diagram (§8.2) and column-level table definitions (§8.3) for all 6 tables; added §8.4 Reference Data Dependencies
- Expanded §9.1 with data classification table; restructured §9.2 with role descriptions and permission matrix; expanded §9.3 compliance controls
- Added §10.2 Availability & Reliability (RTO/RPO, failure scenarios), §10.3 Scalability, §10.4 Maintainability
- Added full §11 Feature Dependencies (5 anticipated features, endpoint mapping, BFF hints, impact assessment)
- Added full §12 Extension Points covering all 5 types: custom fields (§12.2), extension events (§12.3), extension rules (§12.4), extension actions (§12.5), aggregate hooks (§12.6), extension API endpoints (§12.7), summary matrix (§12.8)
- Added §13.1 Data Migration with SAP VBRK/VBRP/FPLA/FPLAN mapping table; added §13.2 Deprecation & Sunset framework
- Restructured §14: added §14.1 Consistency Checks, §14.2 Decisions & Conflicts (5 decisions), renumbered open questions to §14.3 (11 total), added §14.4 ADRs placeholder, moved Suite ADRs to §14.5
- Completed Guidelines Compliance block with Source of Truth Priority and Style Guide
- Added §15.2 References, §15.3 Status Output Requirements; updated §15.4 Change Log
- Updated template compliance from ~75% to ~95%

## Added Sections
- §3.4 Enumerations (6 enums)
- §3.5 Shared Types (Money)
- §4.2 Detailed Rule Definitions
- §4.3 Data Validation Rules
- §4.4 Reference Data Dependencies
- §5.1 Business Logic Placement
- §5.4 Cross-Domain Workflows
- §6.4 OpenAPI Specification reference
- §7.4 Event Flow Diagrams (reference to §5.3)
- §7.5 Integration Points Summary
- §8.2 ER Diagram (Mermaid)
- §8.4 Reference Data Dependencies
- §10.2 Availability & Reliability
- §10.3 Scalability
- §10.4 Maintainability
- §11.1–11.5 Feature Dependencies (full content)
- §12.1–12.8 Extension Points (full content)
- §13.1 Data Migration (SAP mapping)
- §13.2 Deprecation & Sunset
- §14.1 Consistency Checks
- §14.2 Decisions & Conflicts
- §14.4 ADRs
- §15.2 References
- §15.3 Status Output Requirements

## Modified Sections
- Preamble: Updated version to 2026-04-03; updated compliance to ~95%
- Guidelines Compliance: Added Source of Truth Priority and Style Guide sub-sections
- §1.5: Added Authoritative Sources table
- §3.3.1–3.3.5: Added full attribute tables, lifecycle tables, invariants for all entities
- §4.1: Minor corrections (BR-BIL-003 description clarified)
- §5.2: Added Actor/Preconditions/Main Flow/Postconditions/Alternative Flows/Exception Flows to all use cases; added 5 new use cases
- §5.3: Renamed from "§5.2 Billing Flow"; added BillingPlan flow diagram
- §6.1–6.3: Replaced bullet endpoints with full HTTP request/response format
- §7.2: Replaced summary table rows with full event blocks (routing key, purpose, payload, envelope, consumers)
- §7.3: Added Handler, queue config, failure handling details
- §8.1: Expanded with ADR references
- §8.3: Replaced bullet table list with full column definitions
- §9.1–9.3: Expanded with tables and compliance controls
- §10.1: Preserved existing metrics; added throughput and concurrency
- §14.3 (formerly §14.1): Added 7 new open questions (Q-BIL-005 to Q-BIL-011)
- §14.5 (formerly §14.2): Preserved suite ADR reference
- §15.4: Added changelog entry for v1.2

## Removed Sections
- None (non-destructive upgrade)

## Decisions Taken
- DEC-BIL-001: Tax determination via synchronous REST (not async EDA) to ensure legal validity on create
- DEC-BIL-002: Price discrepancy creates block (not hard rejection) to support batch billing runs
- DEC-BIL-003: BillingLine has no custom fields (price integrity risk)
- DEC-BIL-004: One billing plan per sales order (simplification decision)
- DEC-BIL-005: PDF generation is best-effort; DMS unavailability MUST NOT block billing release

## Open Questions Raised
- Q-BIL-005: Invoice split criteria
- Q-BIL-006: Multi-currency billing plans
- Q-BIL-007: Feature IDs for sd.bil
- Q-BIL-008: Developer portal URL
- Q-BIL-009: Migration tooling and cut-over strategy
- Q-BIL-010: Async vs sync tax determination
- Q-BIL-011: Multiple billing plans per sales order
