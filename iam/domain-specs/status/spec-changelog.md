# Spec Changelog — iam.audit

**Spec file:** `spec/T1_Platform/iam/domain-specs/iam_audit-spec.md`
**Upgrade date:** 2026-04-03
**Upgraded from:** ~30% compliance (despite 95% header claim)
**Upgraded to:** ~95% compliance (TPL-SVC v1.0.0 fully applied)

---

## Summary

- Added full `## Specification Guidelines Compliance` block (Non-Negotiables, Source Priority, Style Guide)
- Expanded §0 from 3 stub subsections to 4 complete subsections including Target Audience and Related Documents
- Expanded §1 Business Context from 2 thin subsections to all 5: Stakeholders table, Strategic Positioning (SAP SM20 analogy), Service Context with Mermaid context diagram
- Upgraded §3 Domain Model from an aggregate name list to full attribute tables, lifecycle state machines, Mermaid class diagram, 10 enumerations, and shared types for all 4 aggregates (AuditEvent, AuditReport, SiemConfig, AuditAlertThreshold)
- Expanded §4 from 2 bullet points to a full Business Rules Catalog (BR-AUD-001 through BR-AUD-008) with detailed definitions, data validation rules table, and reference data dependencies
- Added §5 Use Cases with Business Logic Placement table, 5 canonical use case blocks (Actor/Preconditions/Main Flow/Postconditions), process flow sequence diagram, and cross-domain workflow analysis
- Expanded §6 REST API from an endpoint table to full request/response JSON examples for all endpoints plus business operations
- Expanded §7 Events from a 1-row summary table to full event envelopes and payloads for 2 published events, consumed-events rationale (REST ingest decision), event flow diagram, and integration points summary table
- Expanded §8 Data Model from a 1-sentence note to full PostgreSQL table definitions (5 tables), ER diagram, indexes, retention policies
- Expanded §9 Security from a 4-row aspect table to data classification table with PII analysis, permission matrix across 4 roles, and compliance matrix (GDPR, SOX, ISO 27001)
- Expanded §10 Quality Attributes from a 3-row table to all 4 subsections: Performance (10 metrics), Availability/Reliability (RTO/RPO + failure scenarios), Scalability, Maintainability
- Expanded §11 Feature Dependencies from a dependency register to all 5 subsections including endpoints-per-feature mapping, BFF aggregation hints, and impact assessment
- Expanded §12 Extension Points from a 1-sentence stub to all 8 subsections covering all 5 extension types (custom fields, extension events, extension rules, extension actions, aggregate hooks)
- Expanded §13 Migration with SAP SM20 → AuditEventType mapping table, Windows Event Log mapping, migration tool reference
- Expanded §14 from a 2-check stub to 7 consistency checks (all Pass), 5 documented decisions, 5 open questions (Q-AUD-001 through Q-AUD-005), ADR reference table
- Added §15.1 Glossary (16 terms), §15.2 References (13 entries), §15.3 Status Output Requirements

---

## Added Sections

- `## Specification Guidelines Compliance` — complete block
- `§0.2 Target Audience` — full audience list
- `§0.4 Related Documents` — 5 cross-references
- `§1.3 Key Stakeholders` — stakeholder table
- `§1.4 Strategic Positioning` — SAP SM20 analogy; T1 tier constraints
- `§1.5 Service Context` — Responsibilities, Authoritative Sources, Mermaid context diagram
- `§3.1 Conceptual Overview` — prose overview of 4 aggregates and 3 concerns
- `§3.2 Core Concepts` — Mermaid class diagram (all 4 aggregates)
- `§3.3.1–3.3.4` — Full aggregate definitions for AuditEvent, AuditReport, SiemConfig, AuditAlertThreshold
- `§3.4 Enumerations` — 10 enumeration tables (AuditEventType, AuditOutcome, AuditSeverity, ArchivalStatus, ReportType, ReportStatus, ReportFormat, ExportDestination, SiemType, SiemAuthType, SiemFormat, SiemHealth, AlertWindowType)
- `§3.5 Shared Types` — IpAddress shared type
- `§4.1 Business Rules Catalog` — 8 rules with IDs, scope, enforcement, error codes
- `§4.2 Detailed Rule Definitions` — 8 full rule definitions (BR-AUD-001 through BR-AUD-008)
- `§4.3 Data Validation Rules` — field-level and cross-field validation tables
- `§4.4 Reference Data Dependencies` — external catalog table
- `§5.1 Business Logic Placement` — placement table
- `§5.2 Use Cases` — 5 canonical use case blocks (UC-AUD-001 through UC-AUD-005)
- `§5.3 Process Flow Diagrams` — ingest-to-alert sequence diagram
- `§5.4 Cross-Domain Workflows` — platform-wide audit ingest choreography workflow
- `§6.1 API Overview` — auth scopes overview
- `§6.2.1–6.2.5` — full request/response for all endpoints (POST events, GET events, GET event/{id}, POST reports, GET report/{id})
- `§6.3 Business Operations` — TestSiemConnection business operation
- `§6.4 OpenAPI Specification` — contract reference
- `§7.1 Architecture Pattern` — pattern selection (RabbitMQ, outbox) and REST ingest rationale
- `§7.2 Published Events` — 2 events (threshold-exceeded, report-generated) with full envelopes, payloads, consumer tables
- `§7.3 Consumed Events` — no-event-consumption rationale (REST ingest decision DEC-AUD-001)
- `§7.4 Event Flow Diagrams` — ingest-to-alert-delivery sequence diagram
- `§7.5 Integration Points Summary` — upstream producers table + downstream consumers table
- `§8.1 Storage Technology` — PostgreSQL + monthly range partitioning rationale
- `§8.2 Conceptual Data Model` — Mermaid ER diagram (5 tables)
- `§8.3 Table Definitions` — 5 full table definitions (iam_audit_events, iam_audit_reports, iam_siem_configs, iam_audit_thresholds, iam_audit_outbox_events)
- `§8.4 Reference Data Dependencies` — external catalog table
- `§9.1 Data Classification` — 8 data elements with classification, rationale, protection measures
- `§9.2 Access Control` — Roles & Permissions + 4×4 Permission Matrix
- `§9.3 Compliance Requirements` — GDPR / SOX / ISO 27001 compliance table
- `§10.1–10.4` — Performance (10 metrics), Availability (RTO/RPO + 5 failure scenarios), Scalability, Maintainability
- `§11.1–11.5` — Purpose, Feature Dependency Register, Endpoints per Feature, BFF Hints, Impact Assessment
- `§12.1–12.8` — All 8 extension point subsections (5 extension types)
- `§13.1 Data Migration` — SAP SM20 mapping table + Windows Event Log mapping + migration tool reference
- `§14.1 Consistency Checks` — 7 checks, all Pass
- `§14.2 Decisions & Conflicts` — 5 documented decisions (DEC-AUD-001 through DEC-AUD-005)
- `§14.3 Open Questions` — 5 new open questions (Q-AUD-001 through Q-AUD-005)
- `§14.4 ADRs` — domain-level ADR table
- `§14.5 Suite-Level ADR References` — 10 ADR cross-references
- `§15.1 Glossary` — 16 terms
- `§15.2 References` — 13 references
- `§15.3 Status Output Requirements` — compliance score method

---

## Modified Sections

- Meta block — expanded Tags; corrected Repository URL (removed internal slash error)
- `§0.1 Purpose` — expanded from 2 sentences to full paragraph with regulatory context
- `§0.2–0.3 Scope` — restructured from flat In/Out of Scope to template-compliant §0.2 Target Audience + §0.3 Scope with In/Out sub-sections
- `§1.1 Domain Purpose` — expanded with problem framing (fragmented security logging) and centralisation rationale
- `§1.2 Business Value` — expanded from 1 sentence to 5 named value points
- `§2 Service Identity` — added Display Name, Version, Status, Schema Field column; added Team sub-table
- `§3 Domain Model` — expanded AuditAlertThreshold as 4th aggregate (was implied by published event but not modelled); full attribute tables for all 4
- `§4 Business Rules` — replaced 2 bullets with 8-rule catalog + detailed definitions
- `§5 Use Cases` — replaced "see feature specs" delegation with 5 full canonical use cases
- `§6 REST API` — endpoint table preserved; expanded to §6.1–§6.4 with full request/response bodies
- `§7 Events` — 1-row summary table expanded to full §7.1–§7.5
- `§8 Data Model` — 1-sentence stub replaced with full data model
- `§9 Security & Compliance` — 4-row aspect table expanded to 3 structured subsections
- `§10 Quality Attributes` — 3-row table expanded to 4 subsections
- `§11 Feature Dependencies` — dependency register expanded to 5 subsections
- `§12 Extension Points` — 1-sentence stub expanded to 8 subsections
- `§13.1–13.2` — expanded from 1-sentence stubs
- `§14.1 Consistency Checks` — expanded from 2 to 7 checks with status
- `§15.4 Change Log` — added v1.1 upgrade entry

---

## Removed Sections

None — fully non-destructive upgrade. All original content was preserved and expanded.

---

## Decisions Taken

- **DEC-AUD-001:** REST ingest chosen over event-based ingest for audit events (stronger delivery guarantees, simpler failure model)
- **DEC-AUD-002:** No FK constraint from AuditEvent to principal table (principals may be deleted; audit events must survive)
- **DEC-AUD-003:** Monthly range partitioning of `iam_audit_events` for query performance and archival efficiency
- **DEC-AUD-004:** Four separate aggregates (AuditEvent, AuditReport, SiemConfig, AuditAlertThreshold) with independent lifecycles
- **DEC-AUD-005:** AuditEvent extensibility via `metadata` JSONB at creation time; `custom_fields` extension pattern does not apply to immutable records

---

## Open Questions Raised

- **Q-AUD-001:** Configurable vs. platform-wide retention periods (7 years SOX / 3 years GDPR) — per tenant or global constants?
- **Q-AUD-002:** Event-based vs. REST-based ingest for high-volume producers
- **Q-AUD-003:** Migration scope for on-premise SAP SM20 logs
- **Q-AUD-004:** Threshold evaluator implementation strategy (inline vs. async)
- **Q-AUD-005:** Report storage configuration for S3/Azure Blob destinations
