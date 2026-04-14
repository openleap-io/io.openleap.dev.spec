# Spec Changelog - srv.res (Resource Scheduling)

## Summary

- Upgraded `srv_res-spec.md` from v1.1 (~92% compliance) to v1.2 (~97% TPL-SVC v1.0.0 compliance)
- Expanded Guidelines Compliance block with all three mandatory sub-sections (Non-Negotiables, Source of Truth Priority, Style Guide)
- Added full attribute tables with state descriptions and allowed transitions for `SchedulingResource`; added collection constraints and invariants for `AvailabilityWindow`
- Added Value Object (`TimeRange`) and Shared Type (`AvailableSlot`) to §3
- Added four detailed business rule definitions (§4.2), data validation table (§4.3), reference data dependencies (§4.4)
- Expanded all seven use cases with Actor/Preconditions/Main Flow/Postconditions/Exception Flows; added §5.1 Business Logic Placement, §5.3 Process Flow Diagrams, §5.4 Cross-Domain Workflows
- Replaced §6.2 stub ("See UC-001") with full request/response JSON examples for all 7 endpoints; added §6.3 Business Operations and §6.4 OpenAPI reference
- Expanded §7.2 with event envelope and payload JSON for all 4 published events; expanded §7.3 with queue config and DLQ/retry details; added §7.4 Event Flow Diagram; completed §7.5 with fallback and endpoints-used columns
- Added full column definitions, relationships, and retention policy to §8.3 table definitions; added outbox table; added `custom_fields JSONB` column to `res_scheduling_resource`; added §8.4 Reference Data Dependencies
- Expanded §9.2 with full permission matrix and RLS data isolation description; added §9.3 Compliance Requirements (GDPR controls)
- Added throughput/concurrency metrics to §10.1; RTO/RPO and failure scenarios to §10.2; added §10.3 Scalability and §10.4 Maintainability
- Restructured §11 with all five sub-sections (Purpose, Register, Endpoints per Feature, BFF Hints, Impact Assessment)
- Restructured §12 with all eight sub-sections covering all five extension point types (custom-field, extension-event, extension-rule, extension-action, aggregate-hook); added §12.7 Extension API Endpoints and §12.8 Summary & Guidelines
- Added migration mapping table and deprecation framework to §13
- Added §14.1 Consistency Checks (7 checks, all Pass or Partial); added §14.5 Suite-Level ADR References (16 ADRs)
- Added §15.2 References, §15.3 Status Output Requirements; added 13 new open questions (Q-RES-001 through Q-RES-013)

## Added Sections

- §3.3.1 State Descriptions table
- §3.3.1 Allowed Transitions table
- §3.3.1 AvailabilityWindow: Collection Constraints, Invariants
- §3.3 Value Objects: TimeRange
- §3.5 Shared Types: AvailableSlot
- §4.2 Detailed Rule Definitions (BR-001 through BR-005)
- §4.3 Data Validation Rules
- §4.4 Reference Data Dependencies
- §5.1 Business Logic Placement
- §5.3 Process Flow Diagrams
- §5.4 Cross-Domain Workflows
- §6.2 Full request/response bodies for all 7 endpoints
- §6.3 Business Operations (Delete Availability Window)
- §6.4 OpenAPI Specification reference
- §7.2 Event envelope and payload JSON per event
- §7.3 Queue config and failure handling per consumed event
- §7.4 Event Flow Diagram
- §8.3 Full column definitions (all tables), relationships, retention
- §8.3 `res_outbox_events` table
- §8.4 Reference Data Dependencies
- §9.3 Compliance Requirements (GDPR)
- §10.3 Scalability
- §10.4 Maintainability
- §11.1 Purpose
- §11.3 Endpoints per Feature
- §11.4 BFF Aggregation Hints
- §11.5 Impact Assessment
- §12.1 Purpose
- §12.2 Custom Fields (SchedulingResource: Yes; AvailabilityWindow: No)
- §12.3 Extension Events (ext-001, ext-002, ext-003)
- §12.4 Extension Rules (rule.res.001–003)
- §12.5 Extension Actions (action.res.001–003)
- §12.6 Aggregate Hooks (4 hooks)
- §12.7 Extension API Endpoints
- §12.8 Extension Points Summary & Guidelines
- §13.1 Data Migration framework
- §13.2 Deprecation & Sunset framework
- §14.1 Consistency Checks (7 checks)
- §14.5 Suite-Level ADR References (16 ADRs)
- §15.2 References
- §15.3 Status Output Requirements
- Q-RES-001 through Q-RES-013 (13 new open questions)

## Modified Sections

- **Preamble/Meta:** Compliance score updated from ~92% to ~97%; version updated to 2026-04-03
- **Guidelines Compliance block:** Expanded from Non-Negotiables only to all three sub-sections
- **§2 Service Identity:** Added Team sub-table
- **§3.3.1 SchedulingResource:** Added State Descriptions table, Allowed Transitions table; expanded Invariants (BR-004, BR-005 added)
- **§3.4 Enumerations:** Added Description line per enumeration
- **§7.5 Integration Points Summary:** Added Fallback and Endpoints/Keys Used columns to both tables
- **§8.2 Conceptual Data Model:** Added `custom_fields` column and outbox table to ER diagram
- **§9.1 Data Classification:** Expanded from single sentence to full sensitivity table
- **§9.2 Access Control:** Added Permission Matrix table and Data Isolation description
- **§10.1 Performance:** Added throughput and concurrency targets
- **§10.2 Availability & Reliability:** Added RTO/RPO targets and failure scenarios table
- **§11 Feature Dependencies:** Restructured from flat table to §11.1–11.5 sub-sections
- **§12 Extension Points:** Restructured from 2 misstructured sub-sections to 8 correct sub-sections
- **§14.2 Decisions & Conflicts:** Added DC-002, DC-003, DC-004; added Source Priority Note
- **§14.4 ADRs:** Added References sub-section to ADR-SRV-001
- **§15.4 Change Log:** Added v1.2 entry

## Removed Sections

None — non-destructive upgrade. All existing content preserved.

## Decisions Taken

- DC-002: Skill tags remain free-text JSONB array (not validated against catalog) pending T2 skill catalog definition
- DC-003: Overlap check uses half-open `[from, to)` intervals (standard scheduling convention)
- DC-004: `custom_fields` GIN index added by default on `SchedulingResource` table
- `AvailabilityWindow` marked as NOT extensible (technical construct, not customer-facing)
- `SchedulingResource` marked as extensible (customer-facing, known variance across deployments)

## Open Questions Raised

- Q-RES-001: Port assignment for `srv-res-svc`
- Q-RES-002: Repository URI
- Q-RES-003: Skill catalog ownership (HR vs. T2 shared)
- Q-RES-004: Exact FAC event routing keys
- Q-RES-005: Deactivation blocking policy when future appointments exist
- Q-RES-006: Maximum window duration / window count soft cap
- Q-RES-007: Fallback behavior when `srv-cat-svc` is unreachable
- Q-RES-008: Deletion blocking when confirmed appointments exist in window
- Q-RES-009: Who publishes `srv.res.assignment.confirmed` (res vs. apt)
- Q-RES-010: ARCHIVED resource retention period
- Q-RES-011: `skillTags` sensitivity classification (Internal vs. Sensitive)
- Q-RES-012: Final feature IDs in SRV feature catalog
- Q-RES-013: Legacy skill code mapping strategy for migration
