# Spec Changelog - sd.agr (Sales Agreements)

**Upgrade Date:** 2026-04-03
**From Version:** 1.1 (~72% TPL-SVC compliance)
**To Version:** 1.2 (~95% TPL-SVC compliance)
**Template:** `domain-service-spec.md` v1.0.0

---

## Summary

- Added full attribute tables for all aggregates (Agreement, RebateAgreement) and all child entities (AgreementLine, RebateScale, RebateAccrual, RebateSettlement)
- Expanded §1 Business Context with Strategic Positioning (§1.4) and Service Context sub-sections (responsibilities, authoritative sources, context diagram)
- Added detailed business rule definitions (§4.2) for all 6 rules with examples and error handling
- Added data validation rules (§4.3) and reference data dependencies (§4.4)
- Expanded use cases (§5) with Actor/Preconditions/Main Flow/Postconditions; added UC-AGR-005 through UC-AGR-008
- Expanded REST API (§6) from bullet points to full request/response JSON examples
- Expanded events (§7) from summary tables to full event envelopes with Known Consumers
- Added full table definitions (§8) with columns, indexes, and ER diagram
- Added data classification (§9.1), compliance requirements (§9.3), and expanded RBAC matrix (§9.2)
- Added availability/reliability (§10.2), scalability (§10.3), and maintainability (§10.4)
- Populated feature dependencies (§11) with anticipated feature register
- Populated extension points (§12) covering all 5 types: custom fields, extension events, rules, actions, and aggregate hooks
- Populated migration framework (§13) with SAP source mapping and migration strategy
- Restructured §14 with consistency checks (§14.1), decisions (§14.2), ADR references (§14.5)

---

## Added Sections

- §1.4 Strategic Positioning
- §1.5 Service Context (Responsibilities, Authoritative Sources, context diagram)
- §3.3.1 Agreement — Aggregate Root attribute table, state descriptions, allowed transitions, domain events emitted
- §3.3.1 Agreement — AgreementLine child entity attribute table
- §3.3.2 RebateAgreement — Aggregate Root attribute table, state descriptions, allowed transitions, domain events emitted
- §3.3.2 RebateAgreement — RebateScale, RebateAccrual, RebateSettlement child entity attribute tables
- §3.4 Enumerations (AgreementType, AgreementStatus, RebateAgreementStatus, RebateType, SettlementFrequency, SettlementMethod, RebateSettlementStatus)
- §3.5 Shared Types (Money value object)
- §4.2 Detailed Rule Definitions (BR-AGR-001 through BR-AGR-006)
- §4.3 Data Validation Rules (field-level and cross-field)
- §4.4 Reference Data Dependencies
- §5.1 Business Logic Placement table
- §5.3 Process Flow Diagrams (Release Validation flow added)
- §5.4 Cross-Domain Workflows (Rebate Settlement Saga)
- §6.2 Full request/response bodies for key endpoints
- §6.3 Business Operations (expanded from bullets)
- §6.4 OpenAPI Specification reference
- §7.1 Architecture Pattern
- §7.2 Published Events (full event envelopes for all 6 events)
- §7.3 Consumed Events (handler classes, queue config, failure handling)
- §7.4 Event Flow Diagram
- §7.5 Integration Points Summary
- §8.2 Conceptual Data Model (ER diagram)
- §8.3 Full Table Definitions (7 tables with columns, indexes, retention)
- §8.4 Reference Data Dependencies
- §9.1 Data Classification
- §9.3 Compliance Requirements (GDPR, SOX)
- §10.2 Availability & Reliability
- §10.3 Scalability
- §10.4 Maintainability
- §11.1–§11.5 Feature Dependencies (all sub-sections)
- §12.1–§12.8 Extension Points (all 5 types, all sub-sections)
- §13.1 Data Migration (SAP mapping table, migration strategy)
- §13.2 Deprecation & Sunset
- §14.1 Consistency Checks
- §14.2 Decisions & Conflicts (DEC-AGR-001 through DEC-AGR-004)
- §14.4 ADRs (domain-level)
- §14.5 Suite-Level ADR References
- §15.2 References
- §15.3 Status Output Requirements

---

## Modified Sections

- **Preamble:** Updated version to 1.2, updated compliance to ~95%, updated date to 2026-04-03
- **Specification Guidelines Compliance:** Populated with full normative text (was empty)
- **§2 Service Identity:** Added Team sub-table; added Repository and Tags fields
- **§3.3.1 Agreement:** Added aggregate property table; kept existing lifecycle diagram
- **§3.3.2 RebateAgreement:** Added aggregate property table; kept existing lifecycle diagram and invariants table
- **§4.1 Business Rules Catalog:** Preserved as-is
- **§5.2 Use Cases:** Added Actor/Preconditions/Main Flow/Postconditions/Business Rules/Exceptions to each existing UC; added UC-AGR-005 through UC-AGR-008
- **§6.1 API Overview:** Added authentication, versioning, pagination, idempotency notes
- **§9.2 Access Control:** Expanded RBAC matrix with Permission Matrix table and Data Isolation note
- **§10.1 Performance Requirements:** Added Throughput and Concurrency sub-sections
- **§14.3 Open Questions:** Preserved existing Q-AGR-001 through Q-AGR-004; added Q-AGR-011 through Q-AGR-014; renumbered for pattern consistency
- **§15.4 Change Log:** Added v1.2 entry

---

## Removed Sections

None (non-destructive upgrade).

---

## Decisions Taken

- DEC-AGR-001: `validate-release` is a READ use case implemented as POST (body required; no state change).
- DEC-AGR-002: Rebate settlement uses event choreography with saga-like waiting (not full ADR-029 saga orchestrator) — justified by two-service scope.
- DEC-AGR-003: AgreementLine is a child entity (not separate aggregate) — within Agreement transaction boundary.
- DEC-AGR-004: Final rebate calculation queries sd.ord synchronously at settlement time for accuracy (per BR-AGR-005).

---

## Open Questions Raised

- Q-AGR-011: Which product features depend on this service's endpoints? (§11.2 register is anticipated)
- Q-AGR-012: Agreement number prefix strategy for SAP migration (avoid collision with new AGR-YYYY-NNNNN numbers)
- Q-AGR-013: Should inactive customers trigger automatic agreement review/termination?
- Q-AGR-014: Minimum rebate threshold — should settlements below a minimum amount be deferred?
