# Spec Changelog — srv.ent (Entitlements)

## Summary

- Upgraded from ~92% (self-reported; actual ~58%) to ~97% TPL-SVC v1.0.0 compliance
- Added §1.4 Strategic Positioning: positions `srv.ent` as an entitlement-accounting ledger analogous to SAP MM-IM but for service quotas
- Expanded §3 domain model with §3.1 Conceptual Overview, §3.4 Enumerations (EntitlementStatus, TransactionType tables), §3.5 Shared Types, state transition guard details
- Added §4.2 detailed rule definitions for all 7 business rules (BR-001 through BR-007), including BR-005 Idempotent Consumption and BR-006 Reversal Pairing which were absent from the catalog
- Added §4.3 Data Validation Rules (field-level + cross-field) and §4.4 Reference Data Dependencies
- Expanded §5 with §5.1 Business Logic Placement table, full Actor/Preconditions/Main Flow/Postconditions blocks for all use cases, §5.3 process flow diagrams, §5.4 cross-domain workflow (Purchase → Consumption → Expiry)
- Expanded §6 REST API from a bullet list to full request/response JSON examples for all endpoints; added §6.3 Business Operations and §6.4 OpenAPI Specification reference
- Expanded §7 Events with §7.1 Architecture Pattern (outbox, RabbitMQ), full event envelope and payload JSON for all 6 published events, detailed consumed event handler configuration with queue names and DLQ config, §7.4 Event Flow Diagram, §7.5 Integration Points Summary tables
- Expanded §8.3 Table Definitions with full column definitions for all 3 tables (ent_entitlement, ent_quota_transaction, ent_outbox_events) including outbox table per ADR-013; added `custom_fields JSONB` column and GIN index per ADR-067; added §8.4 Reference Data Dependencies
- Expanded §9 to full 3-section security spec: §9.1 data classification table, §9.2 RBAC matrix + permission matrix + RLS description, §9.3 compliance requirements (GDPR, SOX, financial retention)
- Expanded §10 Quality Attributes with §10.2 Availability & Reliability (RTO/RPO, failure scenarios), §10.3 Scalability (horizontal scaling, partitioning, capacity planning), §10.4 Maintainability (API versioning, health checks, metrics, alerting)
- Expanded §11 Feature Dependencies with §11.1 Purpose, §11.2 full register (added F-SRV-010), §11.3 endpoints-per-feature, §11.4 BFF aggregation hints, §11.5 impact assessment
- Expanded §12 Extension Points from a single hook row to full 5-type coverage: §12.1 Purpose, §12.2 Custom Fields (Entitlement: Yes, QuotaTransaction: No with rationale), §12.3 Extension Events (4 hooks), §12.4 Extension Rules (3 slots), §12.5 Extension Actions (4 actions), §12.6 Aggregate Hooks (4 hooks with full contract), §12.7 Extension API Endpoints, §12.8 Summary & Guidelines
- Expanded §13 Migration & Evolution: §13.1 source system mapping (SAP CRM SD-SC, legacy punch card, subscription DB), §13.2 Deprecation & Sunset framework
- Added §14.1 Consistency Checks (7 checks, all Pass except one Open), §14.2 Decisions (4 decisions), renumbered open questions to Q-ENT-NNN pattern (9 questions), §14.4 domain ADR placeholder, §14.5 full ADR reference table
- Added §15.2 References, §15.3 Status Output Requirements
- Upgraded Guidelines Compliance block to full 3-part template format (Non-Negotiables, Source Priority, Style Guide)

## Added Sections

- §1.4 Strategic Positioning
- §1.5 Authoritative Sources table and context diagram (added to existing §1.5)
- §3.1 Conceptual Overview
- §3.4 Enumerations (EntitlementStatus, TransactionType)
- §3.5 Shared Types
- §3.3 Value Objects note (with OPEN QUESTION)
- §4.2 Detailed Rule Definitions (BR-001 through BR-007)
- §4.3 Data Validation Rules
- §4.4 Reference Data Dependencies
- §5.1 Business Logic Placement
- §5.2 Use case detail blocks (Actor/Preconditions/Main Flow/Postconditions/Business Rules/Alternative Flows/Exception Flows) for UC-001 through UC-008
- §5.3 Process Flow Diagrams (reservation flow, session completion flow)
- §5.4 Cross-Domain Workflows
- §6.2 Resource Operations (full request/response JSON for all endpoints)
- §6.3 Business Operations (reserve, consume with full JSON)
- §6.4 OpenAPI Specification reference block
- §7.1 Architecture Pattern
- §7.2 Full event envelope and payload JSON for all 6 published events
- §7.3 Full consumed event detail (handler class, queue name, DLQ, retry policy) for all 5 consumed events
- §7.4 Event Flow Diagrams
- §7.5 Integration Points Summary (upstream + downstream tables)
- §8.3 Full column definitions for ent_entitlement, ent_quota_transaction, ent_outbox_events
- §8.4 Reference Data Dependencies
- §9.2 Access Control (RBAC matrix, permission matrix, RLS description)
- §9.3 Compliance Requirements
- §10.2 Availability & Reliability
- §10.3 Scalability
- §10.4 Maintainability
- §11.1 Purpose
- §11.3 Endpoints per Feature
- §11.4 BFF Aggregation Hints
- §11.5 Impact Assessment
- §12.1 Purpose
- §12.2 Custom Fields (Entitlement: Yes; QuotaTransaction: No)
- §12.3 Extension Events (4 events)
- §12.4 Extension Rules (3 slots)
- §12.5 Extension Actions (4 actions)
- §12.7 Extension API Endpoints
- §12.8 Extension Points Summary & Guidelines
- §13.1 Data Migration (source system mapping)
- §13.2 Deprecation & Sunset framework
- §14.1 Consistency Checks
- §14.2 Decisions & Conflicts (4 decisions)
- §14.4 Domain-Level ADRs placeholder
- §14.5 Suite-Level ADR References (16 ADRs)
- §15.2 References
- §15.3 Status Output Requirements

## Modified Sections

- **Preamble / Guidelines Compliance:** Expanded from minimal 3-bullet stub to full 3-part block (Non-Negotiables, Source Priority, Style Guide)
- **§0.4 Related Documents:** Restructured from inline list to formal table with document type column
- **§1.5 Service Context:** Added Authoritative Sources table and context diagram
- **§2 Service Identity:** Added Repository and Tags rows, formal Team table
- **§3.2 Core Concepts:** Preserved Mermaid class diagram unchanged
- **§3.3.1 Entitlement — Aggregate Root:** Added Read-Only column to attribute table; expanded state table with business meaning; added full allowed transitions table
- **§4.1 Business Rules Catalog:** Added BR-005, BR-006, BR-007 (were implied but not catalogued)
- **§11.2 Feature Dependency Register:** Added F-SRV-010 (Customer Balance View)
- **§12.3 (renamed):** Was labeled "Aggregate Hooks" but incorrectly placed as §12.3; corrected to §12.6 (Aggregate Hooks) per template; existing `hook-001` preserved and expanded
- **§14.3 Open Questions:** Renumbered to Q-ENT-NNN format; added Q-ENT-004 through Q-ENT-009
- **§15.1 Glossary:** Added Balance, Punch Card, Treatment Series, Eligibility, Consumption, Reversal, Reservation terms
- **§15.4 Change Log:** Added v1.2 entry

## Removed Sections

- None (non-destructive upgrade)

## Decisions Taken

- **DEC-ENT-001:** Cached quota counters (`consumedQuota`, `reservedQuota`) retained rather than computing from transaction log on every read — performance requirement for sub-50ms balance checks
- **DEC-ENT-002:** QuotaTransaction confirmed as append-only / immutable — BR-007 formalised
- **DEC-ENT-003:** Entitlement creation is event-driven (not REST POST) — separation of commercial and operational concerns
- **DEC-ENT-004:** Reservation is optional at booking (deployment-dependent) — Q-ENT-002 remains open
- **Extensibility:** `Entitlement` aggregate marked as extensible; `QuotaTransaction` explicitly excluded (immutability constraint)
- **Outbox table:** Added `ent_outbox_events` to §8.3 per ADR-013 (was missing from original data model)
- **`custom_fields` JSONB:** Added to `ent_entitlement` table and ER diagram per ADR-067

## Open Questions Raised

- Q-ENT-001: Pricing/valuation references in `srv.ent` (carried over, renumbered)
- Q-ENT-002: Reservation mandatory vs optional (carried over, renumbered)
- Q-ENT-003: Exact event keys from `sd`/`com` (carried over, renumbered)
- Q-ENT-004: Service port number (new)
- Q-ENT-005: Repository URI (new)
- Q-ENT-006: `QuotaAllocation` value object (new — domain model design question)
- Q-ENT-007: Legal retention period for records (new — compliance question)
- Q-ENT-008: Additional planned features (new — feature dependency completeness)
- Q-ENT-009: Extension API versioning (new — API design question)
