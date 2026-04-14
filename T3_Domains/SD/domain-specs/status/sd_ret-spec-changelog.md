# Spec Changelog - sd.ret

## Summary
- Upgraded `sd_ret-spec.md` from ~72% to ~95% TPL-SVC v1.0.0 compliance
- Added full attribute tables for all four aggregates (ReturnOrder, ReturnLine, Inspection, ReturnReason)
- Added §3.4 Enumerations (7 enums) and §3.5 Shared Types (Money)
- Added §4.2 Detailed Rule Definitions for all 6 business rules with error handling and examples
- Added §4.3 Data Validation Rules (field-level and cross-field) and §4.4 Reference Data Dependencies
- Expanded all REST endpoints with full request/response JSON bodies
- Expanded §7 with event envelopes, payload examples, consumed event queue configs, §7.4 flow diagrams, §7.5 integration points summary
- Added full §8.3 table definitions (5 tables with columns, indexes, relationships, retention)
- Added §9.1 Data Classification, expanded §9.2 Access Control, added §9.3 Compliance Requirements
- Added §10.2 Availability & Reliability, §10.3 Scalability, §10.4 Maintainability
- Fully authored §11 Feature Dependencies, §12 Extension Points (all 5 types), §13 Migration & Evolution
- Restructured §14: added §14.1 Consistency Checks, §14.2 Decisions & Conflicts; moved existing open questions to §14.3; added §14.4 ADRs, §14.5 Suite ADR References
- Added §15.2 References and §15.3 Status Output Requirements

## Added Sections
- §1.4 Strategic Positioning (new)
- §1.5 Service Context — expanded with Responsibilities, Authoritative Sources, context diagram
- §3.3 Attribute tables for ReturnOrder, ReturnLine, Inspection, ReturnReason aggregate roots and child entities
- §3.4 Enumerations: ReturnOrderStatus, ReturnType, RefundMethod, ReturnLineStatus, InspectionCondition, DispositionCode, ReturnReasonCategory
- §3.5 Shared Types: Money
- §4.2 Detailed Rule Definitions (BR-RET-001 through BR-RET-006)
- §4.3 Data Validation Rules
- §4.4 Reference Data Dependencies
- §5.1 Business Logic Placement
- §5.2 Use case detail blocks (Actor, Preconditions, Main Flow, Postconditions) for all use cases
- §5.4 Cross-Domain Workflows (Return-to-Credit choreography)
- §6.2–§6.3 Full request/response bodies for all 14 endpoints
- §6.4 OpenAPI Specification reference
- §7.1 Event-Driven Architecture Pattern (explicit pattern selection)
- §7.2 Event envelopes and payload examples for 4 key events
- §7.3 Consumed events with queue config and failure handling
- §7.4 Event Flow Diagrams
- §7.5 Integration Points Summary (upstream + downstream)
- §8.2 Conceptual ER diagram
- §8.3 Full table definitions for ret_return_orders, ret_return_lines, ret_inspections, ret_return_reasons, ret_outbox_events
- §8.4 Reference Data Dependencies
- §9.1 Data Classification
- §9.3 Compliance Requirements (GDPR, SOX)
- §10.2 Availability & Reliability
- §10.3 Scalability
- §10.4 Maintainability
- §11 Feature Dependencies (Purpose, Register, Endpoints per Feature, BFF Hints, Impact Assessment)
- §12 Extension Points (Custom Fields for 3 aggregates, Extension Events, Rules, Actions, Aggregate Hooks, Extension API, Summary)
- §13 Migration & Evolution (SAP SD RE mapping, deprecation framework)
- §14.1 Consistency Checks
- §14.2 Decisions & Conflicts
- §14.4 ADRs (framework)
- §15.2 References
- §15.3 Status Output Requirements

## Modified Sections
- Preamble: Updated Template Compliance to ~95%; bumped Version to 2026-04-04
- Guidelines Compliance block: Populated with Non-Negotiables, Source Priority, Style Guide
- §1.5 (formerly §1.4): Renamed to Service Context; added Responsibilities, Authoritative Sources table, context diagram
- §2 Service Identity: Added Team table with Name/Email/Slack
- §3.3 ReturnOrder: Retained existing state diagram and state descriptions; added attribute table, Allowed Transitions table, Domain Events list
- §4.1 Business Rules Catalog: Preserved as-is
- §5 Use Cases: Reorganized — UC-RET-001/002/003/004 retained; added UC-RET-002 (Approve), UC-RET-005 (TriggerCredit), UC-RET-006 (List), UC-RET-007 (Return Reasons); renumbered
- §9 Security: Restructured into §9.1/§9.2/§9.3; preserved existing RBAC matrix in §9.2
- §10 Quality: Preserved existing performance numbers in §10.1; added §10.2–§10.4
- §14 Open Questions: Renumbered to §14.3; added Q-RET-005/Q-RET-006/Q-RET-007
- §14 Suite ADR References: Moved to §14.5; preserved ADR-SD-002 reference
- §15.1 Glossary: Preserved; added Thin Event, Outbox, Policy Window, Auto-approve
- §15.4 Change Log: Added 2026-04-04 upgrade entry

## Removed Sections
- None (non-destructive upgrade)

## Decisions Taken
- Inspection modeled as child entity of ReturnOrder (not separate aggregate) — see DC-RET-002
- Credit triggering via event choreography, not synchronous call to sd.bil — see DC-RET-001
- Restocking fees deferred to Phase 2 via extension rule EXT-RULE-RET-003 — see DC-RET-003
- ReturnOrder and ReturnLine marked extensible (custom_fields JSONB); ReturnReason not extensible

## Open Questions Raised
- Q-RET-005: Which product features formally depend on sd.ret endpoints?
- Q-RET-006: RMA number format and generation strategy
- Q-RET-007: Is im.goodsreceipt.posted integration required or optional?
