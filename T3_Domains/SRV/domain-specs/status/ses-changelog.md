# Spec Changelog - srv.ses

## Summary

- Upgraded `srv_ses-spec.md` from v1.1 to v1.2 with full TPL-SVC v1.0.0 compliance
- Expanded business rules from 4 to 10 (added BR-005 through BR-010 with full definitions in §4.2)
- Added complete field-level and cross-field validation rules (§4.3)
- Added reference data dependencies table (§4.4 and §8.4)
- Expanded all 8 use cases with Actor/Preconditions/Main Flow/Postconditions detail
- Expanded REST API from bullet points to full request/response JSON bodies for all 8 endpoints
- Expanded event definitions with full event envelopes and known consumers for all 5 published events
- Expanded consumed events with queue names, handler classes, and DLQ/retry configuration
- Added full PostgreSQL column definitions for all 3 tables (ses_session, ses_proof_artifact, ses_outbox_events)
- Restructured §12 Extension Points to cover all 5 extension types (custom fields, events, rules, actions, hooks)
- Added GDPR compliance controls in §9.3
- Added scalability (§10.3) and maintainability (§10.4) sections with metrics and alerting thresholds

## Added Sections

- §4.2 Detailed Rule Definitions (BR-001 through BR-010)
- §4.3 Data Validation Rules (field-level + cross-field)
- §4.4 Reference Data Dependencies
- §5.1 Business Logic Placement
- §5.3 Process Flow Diagrams (sequence diagram)
- §5.4 Cross-Domain Workflows (3 workflows: appointment-to-session, session-to-billing, session-to-entitlement)
- §6.3 Business Operations summary table
- §6.4 OpenAPI Specification reference
- §7.1 Architecture Pattern
- §7.4 Event Flow Diagrams
- §7.5 Integration Points Summary (upstream + downstream tables)
- §8.4 Reference Data Dependencies
- §9.3 Compliance Requirements (GDPR controls)
- §10.3 Scalability (horizontal scaling, read replicas, capacity planning)
- §10.4 Maintainability (API versioning, monitoring, alerting)
- §11.1 Feature Dependencies Purpose
- §11.3 Endpoints per Feature
- §11.4 BFF Aggregation Hints
- §11.5 Impact Assessment
- §12.1 Extension Points Purpose
- §12.2 Custom Fields (Session: extensible, ProofArtifact: not extensible)
- §12.4 Extension Rules (3 rule slots)
- §12.5 Extension Actions (3 action slots)
- §12.7 Extension API Endpoints
- §12.8 Extension Points Summary & Guidelines
- §13.1 Data Migration (legacy mapping, SAP CS reference)
- §13.2 Deprecation & Sunset framework
- §14.1 Consistency Checks (7 checks, all Pass or Partial)
- §14.4 Domain-Level ADRs (ADR-SES-001 through ADR-SES-003)
- §14.5 Suite-Level ADR References
- §15.2 References
- §15.3 Status Output Requirements

## Modified Sections

- **Preamble**: Added full Source of Truth Priority and Style Guide sub-sections to Guidelines Compliance block
- **§2 Service Identity**: Added Team table, Repository row, Tags row
- **§3.3.1 Session**: Added Allowed Transitions table; expanded initial/terminal states property block; expanded domain events list
- **§3.4 Enumerations**: Added `Description` preamble lines to SessionStatus and ProofType
- **§4.1 Business Rules Catalog**: Added BR-005 through BR-010 rows
- **§5.2 Use Cases**: Added full detail blocks to all use cases; added UC-005 (RecordNoShow), UC-008 (AttachProofArtifact); renumbered GetSession and SearchSessions
- **§6.2 Resource Operations**: Expanded from 6 bullet points to 8 full endpoint sections with request/response JSON
- **§7.2 Published Events**: Expanded from summary table to individual event sections with envelopes and consumer tables
- **§7.3 Consumed Events**: Expanded from summary table to individual event sections with queue config and failure handling
- **§8.1 Storage Technology**: Confirmed PostgreSQL; added schema name and multitenancy note
- **§8.2 Conceptual Data Model**: Added `ses_outbox_events` to ER diagram; added `custom_fields` and `version` columns
- **§8.3 Table Definitions**: Added full column definitions, purpose descriptions, and data retention policies for all 3 tables
- **§9.1 Data Classification**: Added data element sensitivity table
- **§9.2 Access Control**: Added permission matrix table alongside existing role list
- **§11.2 Feature Dependency Register**: Expanded with F-SRV-004-01/02/03 sub-features
- **§12.2 → §12.3**: Renumbered Extension Events (was §12.2); corrected subsection numbering throughout §12
- **§12.3 → §12.6**: Renumbered Aggregate Hooks (was §12.3); expanded with hook contract details
- **§14.2 Decisions & Conflicts**: Added DC-002 (event choreography) and DC-003 (INTERVAL type)
- **§14.3 Open Questions**: Renumbered to Q-SES-NNN pattern; added Q-SES-001 through Q-SES-009
- **§15.1 Glossary**: Added Outbox, Billing Intent, Entitlement, No-Show, Proof Certificate entries
- **§15.4 Change Log**: Added v1.2 entry

## Removed Sections

None — this was a non-destructive upgrade. All existing content was preserved.

## Decisions Taken

- ADR-SES-001: Session creation via event choreography (not direct REST) documented
- ADR-SES-002: Proof artifacts stored by reference only (no binary in ses service)
- ADR-SES-003: Sessions are append-only after COMPLETED
- DC-003: `delivered_duration` stored as PostgreSQL INTERVAL (not integer seconds)
- Session aggregate marked as extensible; ProofArtifact marked as not extensible
- outcomeCode vocabulary deferred to OPEN QUESTION Q-SES-003 (no fabrication of unknown vocabulary)

## Open Questions Raised

- Q-SES-001: Port assignment for srv-ses-svc
- Q-SES-002: Repository URI for srv-ses-svc
- Q-SES-003: outcomeCode platform vocabulary definition
- Q-SES-004: Maximum proof artifacts per session
- Q-SES-005: outcomeCode as shared type vs plain string
- Q-SES-006: PostgreSQL version confirmation with infra
- Q-SES-007: Data retention period (legal/compliance decision)
- Q-SES-008: Whether serviceOfferingId is validated synchronously on session create
- Q-SES-009: Additional product features depending on srv.ses endpoints
