# Spec Changelog - srv.bil

## Summary

- Upgraded `srv_bil-spec.md` from v1.1 to v1.2 to achieve full TPL-SVC v1.0.0 compliance
- Added §4.2 Detailed Rule Definitions for all 6 business rules (BR-001 through BR-006)
- Added §4.3 Data Validation Rules (field-level + cross-field) and §4.4 Reference Data Dependencies
- Added §5.1 Business Logic Placement table; expanded all 9 use cases with Actor/Preconditions/Main Flow/Postconditions/Business Rules/Alternative Flows/Exception Flows
- Added §5.3 Process Flow Diagrams (sequence diagram + state diagram); §5.4 Cross-Domain Workflows documenting the Service Delivery Billing Chain and No-Show Fee choreography patterns
- Expanded §6: full request/response JSON for all 6 REST operations (GET, Search, Confirm, MarkReady, Reverse, Correct); added §6.4 OpenAPI reference
- Expanded §7.2 with event envelopes and payload examples for all 6 published events; §7.3 with queue config and failure handling; §7.4 event flow diagram; §7.5 upstream/downstream tables with fallback strategies
- Added outbox table `bil_outbox_events` to §8.3; added `custom_fields JSONB` + GIN indexes to `bil_billing_intent` and `bil_line_item`; added §8.4 reference data dependencies
- Completed §9 Security (sensitivity table per data element, permission matrix, GDPR/SOX compliance controls)
- Completed §10 Quality (throughput/concurrency, RTO/RPO, failure scenarios, horizontal scaling, monitoring/alerting thresholds)
- Added §11.1 Purpose, §11.3 Endpoints per Feature, §11.4 BFF Aggregation Hints, §11.5 Impact Assessment
- Fully restructured §12 Extension Points to cover all 5 types: custom fields (BillingIntent + LineItem), extension events (4), extension rules (4 slots), extension actions (3 slots), aggregate hooks (4 hooks); added §12.7 Extension API Endpoints and §12.8 Summary
- Added §13.1 Data Migration framework with SAP FI-BL/SD-BIL reference mapping; §13.2 Deprecation & Sunset framework with roadmap items
- Added §14.1 Consistency Checks (7 checks, all Pass or documented partial); §14.5 Suite-Level ADR References (14 ADRs mapped)
- Added §15.2 References and §15.3 Status Output Requirements
- Raised 15 new open questions Q-BIL-001 through Q-BIL-015

## Added Sections

- §3.5 Shared Types (documents intentional absence; references Money decision DC-002)
- §4.2 Detailed Rule Definitions: full rule blocks for BR-001 through BR-006
- §4.3 Data Validation Rules: field-level and cross-field validation tables
- §4.4 Reference Data Dependencies: external catalog dependency table
- §5.1 Business Logic Placement: domain/service/application layer table
- §5.2 Use case detail: Actor/Preconditions/Main Flow/Postconditions/Alt/Exception Flows for UC-001 through UC-009
- §5.3 Process Flow Diagrams: sequence and state diagrams
- §5.4 Cross-Domain Workflows: Service Delivery Billing Chain + No-Show Fee choreography
- §6.2.1 Get Billing Intent: full response JSON with `_links`
- §6.2.2 Search Billing Intents: query parameters and paginated response
- §6.3.1 Confirm Intent: request/response with business rules and events
- §6.3.2 Mark Ready for Invoicing: request/response
- §6.3.3 Reverse Intent: request/response with new REVERSAL intent body
- §6.3.4 Correct Intent: request/response with new CORRECTION intent body
- §6.4 OpenAPI Specification reference block
- §7.1 Architecture Pattern: broker name (RabbitMQ), exchange, rationale
- §7.2 Per-event expansion: Created, Confirmed, ReadyForInvoicing, Reversed, Corrected, Cancelled with envelopes and known consumers
- §7.3 Consumed event expansion: queue config (exchange, binding key, durability) and failure handling (retry 3×, DLQ) for all 3 event streams
- §7.4 Event Flow Diagram: graph of event sources, handlers, and downstream consumers
- §8.3 bil_outbox_events table: full column definitions, indexes, retention policy
- §8.4 Reference Data Dependencies: external catalog lookup table
- §9.1 Data Classification: per-element sensitivity table with protection measures
- §9.2 Access Control: role definitions, permission matrix, RLS description
- §9.3 Compliance Requirements: GDPR, SOX, accounting law applicability + controls
- §10.1 Throughput and Concurrency targets
- §10.2 RTO/RPO targets; failure scenario table
- §10.3 Scalability: horizontal scaling, read replicas, consumer scaling, capacity planning
- §10.4 Maintainability: API versioning policy, monitoring metrics, alerting thresholds
- §11.1 Purpose: explanatory text
- §11.3 Endpoints per Feature: mapping table
- §11.4 BFF Aggregation Hints: SRV BFF and FI BFF hints
- §11.5 Impact Assessment: change-type impact table
- §12.1 Purpose: Open-Closed Principle explanation
- §12.2 Custom Fields: BillingIntent (extensible) + LineItem (extensible) with extension candidates
- §12.3 Extension Events: renumbered from old §12.2; expanded to 4 events with `srv.bil.ext.*` prefix
- §12.4 Extension Rules: 4 rule slots with default behavior and override examples
- §12.5 Extension Actions: 3 action slots
- §12.6 Aggregate Hooks: renumbered from old §12.3; expanded to 4 hooks with contract spec
- §12.7 Extension API Endpoints: register handler + get config endpoints
- §12.8 Extension Points Summary & Guidelines: quick-reference matrix and guidelines
- §13.1 Data Migration: SAP FI-BL/SD-BIL reference; source→target mapping table
- §13.2 Deprecation & Sunset: deprecated features table + communication plan + roadmap
- §14.1 Consistency Checks: all 7 checks with Pass/Partial status
- §14.4 ADR-BIL-002: Non-Authoritative Price Hints (new domain ADR)
- §14.5 Suite-Level ADR References: 14 ADRs mapped to srv.bil
- §15.2 References: full reference table (domain specs, frameworks, SAP docs)
- §15.3 Status Output Requirements: artifact locations + self-assessment table

## Modified Sections

- Preamble / Meta Information: version bumped to 2026-04-03 / 1.2.0; compliance score updated to ~98%
- Preamble / Guidelines Compliance: expanded to full Non-Negotiables + Source Priority + Style Guide
- §2 Service Identity: added Repository and Tags rows; version bumped to 1.2.0
- §3.3.1 BillingIntent Aggregate Root: added `billingIntentNo` field to attribute table; added Allowed Transitions table; added `billingintent.cancelled` to Domain Events Emitted
- §3.3.2 LineItem Child Entity: added Collection Constraints and Invariants subsections
- §8.2 Conceptual Data Model: updated ER diagram to include `custom_fields`, `billing_intent_no`, and `bil_outbox_events` table
- §8.3 bil_billing_intent: added `billing_intent_no` UK column, `custom_fields JSONB` column, GIN index; restructured column table to include PK/UK/FK flags; added retention and RLS notes
- §8.3 bil_line_item: added `custom_fields JSONB` column, `created_at` column, GIN index; restructured column table
- §15.4 Change Log: added v1.2 entry

## Removed Sections

- None (non-destructive upgrade)

## Decisions Taken

- BillingIntent marked as extensible (custom_fields JSONB): rationale = customer-facing financial records with deployment-specific cost center / GL account needs
- LineItem marked as extensible (custom_fields JSONB): rationale = product-specific billing codes (tax category, revenue category) applied at line level
- No Money value object defined: keep `unitPriceHint` + `currencyCode` as primitives; final Money handling belongs to sd/fi (DC-002)
- Added `billingIntentNo` business key to aggregate and table definition for financial reconciliation
- §12 extension events use `srv.bil.ext.*` prefix to distinguish from integration events (`srv.bil.billingintent.*` in §7)
- CORRECTION intent goes directly to READY_FOR_INVOICING (not DRAFT); original marked REVERSED — consistent with §3 state transitions and append-only principle
- Choreography (not saga/orchestration) confirmed for all cross-domain workflows — no coordinator needed for the billing chain

## Open Questions Raised

- Q-BIL-001: Port assignment for srv-bil-svc
- Q-BIL-002: Repository URI for srv-bil-svc
- Q-BIL-003: Shared vs. dedicated PostgreSQL instance
- Q-BIL-004: Single or dual downstream consumer (SD and/or FI)
- Q-BIL-005: Approval workflow before intent confirmation
- Q-BIL-006: Fee policy ownership (srv.bil vs srv.cat)
- Q-BIL-007: billing_intent_no format and sequence strategy
- Q-BIL-008: LineItem custom_fields — confirm product requirement
- Q-BIL-009: Fee policy lookup mechanism during event handling
- Q-BIL-010: Auto-confirm and auto-mark-ready configuration
- Q-BIL-011: Message broker topology (RabbitMQ vs Kafka)
- Q-BIL-012: Full feature consumer map for §11
- Q-BIL-013: GDPR right-to-erasure handling for append-only records
- Q-BIL-014: Legacy system migration field mapping
- Q-BIL-015: Money value object decision
