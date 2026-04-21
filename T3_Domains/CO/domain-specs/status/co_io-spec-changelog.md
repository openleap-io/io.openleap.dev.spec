# Spec Changelog - co.io

## Summary
- Upgraded co.io Internal Orders spec from ~82% to ~95% TPL-SVC v1.0.0 compliance
- Added Source of Truth Priority to Guidelines Compliance block
- Expanded §3 with full child entity definitions (OrderCostPosting, OrderCommitment), value objects (Money, DateRange), and 7 enumeration tables
- Added detailed rule definitions for all 11 business rules (BR-001 through BR-011) in §4.2
- Expanded all use cases (UC-001 through UC-009) with Preconditions, Main Flow, Postconditions, Business Rules, Alternative/Exception Flows
- Added full REST API request/response bodies for all endpoints in §6, including budgets, postings, commitments, and all business operations
- Added event envelope format and detailed payload examples for all 3 published events; expanded consumed events with handler names, queue config, and failure handling
- Added complete table definitions with columns, indexes, relationships, retention policies in §8; added outbox table
- Fully populated §11 Feature Dependencies, §12 Extension Points (all 5 types), and §13 Migration & Evolution

## Added Sections
- §3.3 Child Entity sections: OrderCostPosting, OrderCommitment (full attribute tables)
- §3.3 Value Objects: Money, DateRange
- §3.4 Enumerations: OrderType, BudgetType, BudgetControlMode, ApprovalStatus, PostingOrigin, CommitmentStatus
- §3.5 Shared Types: Money (with Used By references)
- §4.2 Detailed definitions for BR-001, BR-002, BR-004 through BR-011
- §5.2 UC-007 (CreateBudget), UC-008 (CancelOrder), UC-009 (CloseOrder)
- §6.2.3-6.2.8 Full request/response for Update, List, Budget Create/List, Postings List, Commitments List
- §6.3 Submit Budget, Reject Budget operations
- §7.2 Event envelopes for all published events
- §7.3 Handler classes, queue configuration, failure handling for all consumed events
- §7.3 pur.commitment.invoiced consumed event
- §7.4 Event Flow Diagrams (Mermaid sequence)
- §7.5 Full upstream/downstream integration table with criticality and fallback
- §8.1 Storage Technology (PostgreSQL)
- §8.3 Full table definitions for internal_order, order_budget, order_cost_posting, order_commitment, io_outbox_events
- §8.4 External catalogs and internal code lists tables
- §9.1 Overall classification, expanded sensitivity levels
- §9.2 Expanded permission matrix, data isolation description
- §9.3 Compliance controls (retention, audit, erasure, segregation of duties)
- §10.1 Throughput and concurrency targets
- §10.2 Failure scenarios table
- §10.3 Capacity planning details
- §10.4 Maintainability (versioning, monitoring, alerting)
- §11 Feature Dependencies (full structure with provisional register)
- §12 Extension Points (custom fields, extension events, rules, actions, hooks, API, summary)
- §13 Migration strategy with SAP CO-OM-OPA table mapping, rollback plan, deprecation framework
- §14.1 Consistency checks (all 7 checks with Pass/Fail)
- §14.2 Decisions & Conflicts table
- §14.4 ADR-IO-002: Posting Immutability
- §14.5 Suite-Level ADR References
- §15.2 References
- §15.3 Status Output Requirements

## Modified Sections
- Preamble: Updated version to 2026-04-04, compliance to ~95%, added Source of Truth Priority
- §3.3.1 InternalOrder: Added Allowed Transitions table, createdAt/updatedAt/statisticalFlag attributes
- §3.3.2 OrderBudget: Added approvedAt, justification, version attributes
- §4.1 Catalog: Added BR-010 (Currency Consistency) and BR-011 (Posting Immutability)
- §4.3 Expanded field-level validations and cross-field validations
- §5.2 All existing use cases expanded with full detail
- §5.4 Added Participating Services tables, Workflow Steps, Business Implications; added FI Cost Posting workflow
- §6 All endpoints expanded with full JSON examples, headers, rules, events, errors
- §7.2 Existing events expanded with full envelope/payload
- §7.3 Existing consumed events expanded with handler/queue/failure details
- §8.2 ER diagram expanded with custom_fields, outbox table, timestamps
- §14.3 Renumbered to Q-IO-xxx pattern, added Q-IO-003 through Q-IO-007

## Removed Sections
- None (non-destructive upgrade)

## Decisions Taken
- DC-002: Posting immutability model (corrections via reversals, aligning with SOX)
- DC-003: Hard-stop handling for FI-originated postings (record blocked posting for reconciliation)
- InternalOrder and OrderBudget marked as extensible (custom_fields); OrderCostPosting and OrderCommitment not extensible
- Statistical order flag added provisionally (Q-IO-001 remains open)

## Open Questions Raised
- Q-IO-003: Which product features depend on this service? (CO feature specs not yet authored)
- Q-IO-004: Should orderNumber generation be configurable per tenant?
- Q-IO-005: How to handle hard_stop rejection for FI-originated postings?
- Q-IO-006: Multi-currency budget support in future?
- Q-IO-007: Should the service support order hierarchies?
