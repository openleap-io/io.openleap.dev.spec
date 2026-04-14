# Spec Changelog - sd.cdm (Credit Decision Management)

**Upgrade Date:** 2026-04-03
**From Version:** 1.1 (~72% TPL-SVC compliance)
**To Version:** 1.2 (~95% TPL-SVC compliance)
**Template:** `domain-service-spec.md` v1.0.0

---

## Summary

- Added full attribute tables for all 5 aggregates/entities: CreditProfile, CreditExposure, CreditCheck, CreditRule, CreditLimitChange
- Expanded §1 Business Context with Strategic Positioning (§1.4) — SAP FD32/FD33 analogy and platform-wide gating role
- Added §1.5 Service Context with responsibilities table, authoritative sources table, and context diagram
- Added detailed business rule definitions (§4.2) for all 6 rules with Business Context, Rule Statement, Applies To, Enforcement, Validation Logic, Error Handling, and Examples
- Added data validation rules (§4.3) — field-level and cross-field constraints
- Added reference data dependencies (§4.4)
- Added business logic placement table (§5.1)
- Expanded use cases (§5.2) with Actor / Preconditions / Main Flow / Postconditions / Business Rules Applied / Alternative Flows / Exception Flows for all 4 existing UCs
- Added two new use cases: UC-CDM-005 (Get Credit Profile with Exposure) and UC-CDM-006 (Recalculate Credit Exposure)
- Added §5.4 Cross-Domain Workflows with three choreography/event-flow diagrams
- Expanded REST API (§6) from bullet point list to full request/response JSON bodies for all 15 endpoints
- Added §6.3 Business Operations pattern documentation
- Added §6.4 OpenAPI Specification reference
- Added §7.1 Architecture Pattern documentation
- Expanded published events (§7.2) from summary rows to full event envelopes with payload JSON and known consumers tables
- Expanded consumed events (§7.3) with handler class names, queue configurations, bindings, and failure handling
- Added §7.4 Event Flow Diagram (Mermaid sequence for idempotent consumer pattern)
- Added §7.5 Integration Points Summary — upstream and downstream dependency tables
- Added §8.2 Conceptual Data Model (Mermaid ER diagram with all 6 tables)
- Added §8.3 full table definitions for all 6 tables with column types, nullability, indexes, and retention policies
- Added §8.4 Reference Data Dependencies
- Added §9.1 Data Classification (CONFIDENTIAL / INTERNAL classification for each data element)
- Added §9.2 Roles & Permissions detail table and data isolation (RLS) description
- Added §9.3 Compliance Requirements (GDPR, SOX, audit trail, data retention)
- Added §10.2 Availability & Reliability (RTO/RPO, failure scenarios table)
- Added §10.3 Scalability (horizontal scaling, read replicas, consumer scaling, data growth estimate)
- Added §10.4 Maintainability (API versioning, metrics, alerting, database migrations)
- Populated §11 Feature Dependencies — all 5 sub-sections with anticipated feature register, endpoint criticality matrix, BFF aggregation hints, and impact assessment
- Populated §12 Extension Points — all 8 sub-sections covering custom fields, extension events (3), extension rules (3), extension actions (4), aggregate hooks (4), extension API endpoints, and summary matrix
- Populated §13 Migration & Evolution — SAP source table mapping, field-level mapping, data quality issues and resolutions, migration strategy, deprecation framework
- Restructured §14 with consistency checks (§14.1), explicit decisions (§14.2), expanded open questions (§14.3), ADR framework (§14.4), platform ADR references (§14.5)
- Expanded §15 Glossary from 4 to 15 terms
- Added §15.3 Status Output Requirements with section-by-section compliance table

---

## Added Sections

- §1.4 Strategic Positioning (SAP SD-BF-CM analogy, platform gating role)
- §1.5 Service Context (responsibilities table, authoritative sources table, context diagram)
- §3.3.1 CreditProfile — full aggregate property table, attribute table, state descriptions, allowed transitions table, domain events emitted
- §3.3.2 CreditExposure — aggregate property table, attribute table, business purpose, invariants
- §3.3.3 CreditCheck — aggregate property table, attribute table, business purpose, invariants
- §3.3.4 CreditRule — aggregate property table, attribute table, lifecycle description, domain events note
- §3.3.5 CreditLimitChange — aggregate property table, attribute table, business purpose, invariants
- §3.4 Enumerations (RiskCategory, ReviewFrequency, ProfileStatus, CheckTriggerType, CheckResult, RuleType, RuleAction — all 7 enums with value tables)
- §3.5 Shared Types (Money value object with PostgreSQL storage note)
- §4.2 Detailed Rule Definitions (BR-CDM-001 through BR-CDM-006 — all 6 rules with full template)
- §4.3 Data Validation Rules (field-level and cross-field)
- §4.4 Reference Data Dependencies
- §5.1 Business Logic Placement table
- §5.4 Cross-Domain Workflows (3 Mermaid diagrams: credit check choreography, AR exposure update, auto-block propagation)
- UC-CDM-005: Get Credit Profile with Exposure (READ)
- UC-CDM-006: Recalculate Credit Exposure (WRITE — event-triggered)
- §6.2 Full request/response JSON bodies for all 15 endpoints
- §6.3 Business Operations (custom action pattern documentation)
- §6.4 OpenAPI Specification reference
- §7.1 Architecture Pattern (RabbitMQ, outbox, at-least-once delivery)
- §7.2 Full event envelopes for all 5 published events with payload JSON and known consumers
- §7.3 Handler class names, queue configs, bindings, and failure handling for all 6 consumed events
- §7.4 Event Flow Diagram (idempotent consumer pattern)
- §7.5 Integration Points Summary (upstream and downstream tables)
- §8.2 Conceptual Data Model (Mermaid ER diagram)
- §8.3 Full Table Definitions (6 tables: cdm_credit_profiles, cdm_credit_exposures, cdm_credit_checks, cdm_credit_rules, cdm_limit_changes, cdm_outbox_events)
- §8.4 Reference Data Dependencies
- §9.1 Data Classification (CONFIDENTIAL / INTERNAL per data element)
- §9.2 Roles & Permissions Detail table; Data Isolation (RLS) description
- §9.3 Compliance Requirements (GDPR, SOX, audit trail, retention)
- §10.2 Availability & Reliability (RTO/RPO, failure scenarios)
- §10.3 Scalability (horizontal scaling, read replicas, data growth)
- §10.4 Maintainability (versioning, metrics, alerting, migrations)
- §11.1–§11.5 Feature Dependencies (all sub-sections)
- §12.1–§12.8 Extension Points (all sub-sections)
- §13.1 Data Migration (SAP source tables, field mapping, data quality issues, migration strategy)
- §13.2 Deprecation & Sunset (versioned deprecation table, communication plan)
- §14.1 Consistency Checks (all 7 checks — Pass)
- §14.2 Decisions & Conflicts (DEC-CDM-001 through DEC-CDM-004)
- §14.4 ADRs (domain-level candidates)
- §14.5 Platform ADR References
- §15.3 Status Output Requirements

---

## Modified Sections

- **Preamble:** Version updated 1.1 → 1.2; compliance updated ~72% → ~95%; date updated to 2026-04-03
- **Specification Guidelines Compliance:** Populated with Non-Negotiables, Source of Truth Priority, Style Guide (was empty)
- **§0.2 Target Audience:** Added "Development Teams implementing the sd-cdm-svc microservice"
- **§0.3 Scope — Out of Scope:** Added Q-CDM-001/002 cross-references
- **§2 Service Identity:** Added Base Package, Port, Repository rows; added Team sub-table
- **§3.1 Conceptual Overview:** Expanded description
- **§3.2 Core Concepts (Mermaid diagram):** Added `customFields: JSONB?` to CreditProfile and CreditRule; updated enum type names to match §3.4
- **§4.1 Business Rules Catalog:** Enforcement column updated to distinguish Domain Object / Domain Service / Application layers
- **§5.2 Use Cases UC-CDM-001 through UC-CDM-004:** Added full detail below each canonical table (Actor, Preconditions, Main Flow, Postconditions, Business Rules Applied, Alternative Flows, Exception Flows)
- **§5.3 Process Flow Diagrams:** Sequence diagram enhanced (db and outbox participants added)
- **§6.1 API Overview:** Added authentication, content-type, versioning, pagination, idempotency, ETag notes
- **§9.2 Access Control:** Added Roles & Permissions Detail table; added Data Isolation description
- **§10.1 Performance Requirements:** Reformatted as table; added Throughput and Concurrency rows
- **§14.3 Open Questions:** Preserved Q-CDM-001 through Q-CDM-004; added Q-CDM-005 through Q-CDM-007
- **§15.2 Change Log:** Added v1.2 entry
- **§15.1 Glossary:** Expanded from 4 to 15 terms

---

## Removed Sections

None (non-destructive upgrade).

---

## Decisions Taken

- **DEC-CDM-001:** Credit check is synchronous REST (not event-driven) — order confirmation requires an immediate pass/fail decision; async would require polling, increasing complexity with no benefit. This is formalized as ADR-SD-003 at the suite level.
- **DEC-CDM-002:** Exposure stored as snapshot (not calculated on-the-fly) — on-the-fly would require synchronous calls to sd.ord, sd.dlv, and fi.ar during the credit check, violating the < 300ms SLA. Snapshot trades accuracy for performance; staleness bounded by event latency (typically < 5s).
- **DEC-CDM-003:** One CreditProfile per (tenant, customerPartyId) — mirrors SAP's one credit master per (customer, credit control area). Group-level deferred to Q-CDM-004.
- **DEC-CDM-004:** CreditCheck is immutable after creation (except override fields) — audit and SOX compliance requirements. Override is a structured mutation, not deletion of the original decision.

---

## Open Questions Raised

- **Q-CDM-005:** Which product feature IDs from the SD suite catalog depend on sd.cdm endpoints? §11.2 register populated with anticipated IDs; validate after feature specs authored.
- **Q-CDM-006:** Should CDM support a direct "adjust-exposure" endpoint for immediate credit memo relief, or rely solely on `fi.ar.exposure.changed` events?
- **Q-CDM-007:** How should CDM handle currency mismatch between requestedAmount and creditLimit in multi-currency tenants? Currently: reject with CDM_CURRENCY_MISMATCH (v1 limitation). Resolution depends on Q-CDM-003 (multi-currency limits).
