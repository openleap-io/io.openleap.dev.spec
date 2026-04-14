# Spec Changelog - sd.ana

## Summary

- Upgraded `sd_ana-spec.md` from ~70% to ~95% TPL-SVC v1.0.0 compliance
- Added complete attribute tables for all 6 read models (SalesPipelineEntry, SalesKPI, SalesForecast, TopEntity, ReturnAnalytics, EventCheckpoint)
- Expanded §4 with detailed rule definitions for all 5 business rules, field-level validations, and reference data dependencies
- Expanded §5 use cases with Actor, Preconditions, Main Flow, Postconditions, Business Rules, and Exception Flows
- Expanded §6 REST API with full request/response JSON for all 9 endpoint groups
- Expanded §7 with event envelopes, consumed event handler table with queue config and failure handling, integration points summary
- Replaced §8 stub with complete column definitions, indexes, and retention policies for all 6 tables
- Added §9.3 Compliance section (GDPR controls for customer name and sales rep ID fields)
- Added §10.2–10.4 (Availability/Reliability, Scalability, Maintainability)
- Populated §11 Feature Dependencies with placeholder feature IDs and BFF aggregation hints
- Populated §12 Extension Points covering all 5 types (custom fields, extension events, rules, actions, hooks)
- Populated §13 Migration & Evolution with SAP SD-IS source mapping and roadmap items
- Restructured §14 with consistency checks, decisions register (3 decisions), and 8 open questions
- Added §15.3 Status Output Requirements

## Added Sections

- `§1.4` Strategic Positioning
- `§1.5` full Authoritative Sources table and context diagram
- `§2` Team sub-table
- `§3.3` Read Model Definitions — attribute tables for SalesPipelineEntry, SalesKPI, SalesForecast, TopEntity, ReturnAnalytics, EventCheckpoint
- `§3.4` Enumerations — DocumentType, PeriodType, ForecastMethod, EntityType, ReturnGroupDimension
- `§3.5` Shared Types — Money
- `§4.2` Detailed rule definitions (BR-ANA-001 through BR-ANA-005)
- `§4.3` Data validation rules (field-level and cross-field)
- `§4.4` Reference data dependencies
- `§5.1` Business Logic Placement table
- `§5.3` Process Flow Diagram (sequence diagram: event processing → KPI → forecast)
- `§5.4` Cross-Domain Workflows
- `§6.2.1–6.2.9` Full REST endpoint request/response with JSON examples
- `§6.3` Business Operations table
- `§6.4` OpenAPI Specification reference
- `§7.2` Event envelopes and Known Consumers for both published events
- `§7.3` Consumed events handler table with queue config and DLQ failure handling
- `§7.5` Integration Points Summary (upstream dependencies + downstream consumers)
- `§8.2` Conceptual Data Model ER diagram
- `§8.3` Full table definitions for all 6 tables (columns, indexes, retention)
- `§8.4` Reference Data Dependencies
- `§9.1` Data Classification table (sensitivity levels per data element)
- `§9.3` Compliance Requirements (GDPR, SOX, audit trail)
- `§10.2` Availability & Reliability (RTO/RPO, failure scenarios)
- `§10.3` Scalability (horizontal scaling, read replicas, capacity)
- `§10.4` Maintainability (API versioning, health checks, metrics, alerting)
- `§11.1–11.5` Full Feature Dependencies section
- `§12.1–12.8` Full Extension Points section (all 5 extension types)
- `§13.1–13.2` Migration & Evolution (SAP source mapping, deprecation framework, roadmap)
- `§14.1` Consistency Checks (7 checks, all assessed)
- `§14.2` Decisions & Conflicts register (3 decisions)
- `§14.4` ADR references table
- `§14.5` Suite-Level ADR References
- `§15.3` Status Output Requirements

## Modified Sections

- **Preamble / Guidelines Compliance block** — Populated with full Non-Negotiables, Source Priority, and Style Guide from template
- **§3.1 Conceptual Overview** — Expanded with Event Projector pattern explanation
- **§3.2 Core Concepts** — Added ReturnAnalytics and EventCheckpoint to Mermaid class diagram
- **§5.2 Use Cases** — Added UC-ANA-006 (Trigger Forecast Recalculation); expanded all existing UCs with full detail
- **§7.1 Architecture Pattern** — Added broker and rationale
- **§9.2 Access Control** — Added Roles & Permissions table and Data Isolation description
- **§14.3 Open Questions** — Preserved existing Q-ANA-001 through Q-ANA-004; added Q-ANA-005 through Q-ANA-008; reformatted to `Q-ANA-NNN` pattern
- **§15.4 Change Log** — Added v1.2 entry

## Removed Sections

- None (non-destructive upgrade — all existing content preserved)

## Decisions Taken

- **D-ANA-001:** No outbox table — conscious deviation from ADR-013 for the two minimal analytics events (low duplicate risk, jobs are idempotent). Future events may require reconsideration.
- **D-ANA-002:** Single broad queue `sd.ana.in.sd.all` with routing key `sd.#` — simplifies topology; can be split by domain if throughput requires it.
- **D-ANA-003:** `repUserId` sourced from JWT only (not query param) — security constraint preventing cross-rep data access by parameter manipulation.
- **Domain model framing:** Read models documented as "Read Model Definitions" (§3.3) rather than "Aggregate Definitions" to accurately reflect the CQRS read-side nature of sd.ana. Template aggregate root structure adapted accordingly.

## Open Questions Raised

- Q-ANA-005: Cross-currency KPI normalization strategy
- Q-ANA-006: GDPR erasure propagation for analytics data (customer names, sales rep IDs)
- Q-ANA-007: SD analytics feature spec IDs (placeholder IDs used in §11.2)
- Q-ANA-008: Extension endpoint registration standardisation across SD services
