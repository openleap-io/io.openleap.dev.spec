# Spec Changelog - srv.apt

## Summary

- Upgraded `srv_apt-spec.md` from v1.1 to v1.2 to achieve full TPL-SVC v1.0.0 compliance
- Added §4.2 Detailed Rule Definitions for all 7 business rules (BR-001 through BR-007)
- Expanded all 10 use cases with Actor/Preconditions/Main Flow/Postconditions/Business Rules/Alternative Flows/Exception Flows
- Added §5.4 Cross-Domain Workflows documenting the Booking-to-Session choreography pattern
- Expanded §6.2–6.3 REST API with full request/response JSON examples for all endpoints
- Added §7.2 event envelopes and payload examples for all 6 published events; §7.3 expanded with queue config and failure handling; §7.4 event flow diagram added
- Fully restructured §12 Extension Points to cover all 5 types: custom fields, extension events, rules, actions, aggregate hooks
- Completed §9 Security (sensitivity table, permission matrix, GDPR compliance controls), §10 Quality (Scalability, Maintainability), §11 Feature Dependencies (full §11.1–11.5), §13 Migration & Evolution, §14.1 all 7 consistency checks, §15.2–15.3 References and Status Requirements
- Raised 15 numbered open questions (Q-APT-001 through Q-APT-015)

## Added Sections

- §3.3.2 WaitlistEntry: lifecycle states, invariants, domain events emitted
- §3.5 Shared Types: `TimeConstraints` value object for `constraintsJson`
- §4.2 Detailed Rule Definitions: full rule blocks for BR-001 through BR-007
- §4.4 Reference Data Dependencies: external catalog table
- §5.2 Use case detail: Preconditions/Postconditions/Main Flow/Alt Flows/Exception Flows on UC-001 through UC-010
- §5.4 Cross-Domain Workflows: Booking-to-Session choreography workflow
- §6.2.2 Create Appointment: full request/response JSON
- §6.2.3 Retrieve Appointment: full response JSON
- §6.2.4 Search Appointments: query parameters and response
- §6.3 Business Operations: full request/response for Reserve, Confirm, Reschedule, Cancel, MarkNoShow, AddToWaitlist
- §6.4 OpenAPI Specification: reference block
- §7.1 Architecture Pattern: broker name, rationale, choreography/orchestration distinction
- §7.2 Per-event expansion: Offered, Reserved, Booked, Rescheduled, Cancelled, NoShowMarked with envelopes and known consumers
- §7.3 Consumed events: queue config and failure handling for all 4 consumed event streams
- §7.4 Event Flow Diagrams: booking confirmation sequence diagram
- §7.5 Integration Points Summary: Fallback column added; upstream/downstream tables completed
- §8.1 Storage Technology: ADR-016/021 references added
- §8.3 Table Definitions: Business Description, Relationships, Data Retention for all tables; `custom_fields JSONB` on `apt_appointment` and `apt_waitlist_entry`; `apt_outbox_events` table
- §8.4 Reference Data Dependencies
- §9.1 Data Classification: sensitivity levels table
- §9.2 Access Control: expanded permission matrix, data isolation description
- §9.3 Compliance Requirements: GDPR controls (erasure, portability, audit trail)
- §10.1 Throughput and Concurrency subsections
- §10.2 Availability: RTO/RPO targets, failure scenarios table
- §10.3 Scalability: horizontal scaling, read replicas, capacity planning
- §10.4 Maintainability: API versioning strategy, monitoring, alerting
- §11.1 Purpose: standard purpose text
- §11.3 Endpoints Used per Feature: F-SRV-002 and F-SRV-004 endpoint tables
- §11.4 BFF Aggregation Hints: table for F-SRV-002
- §11.5 Impact Assessment: table framework
- §12.1 Purpose: Open-Closed Principle explanation
- §12.2 Custom Fields (extension-field): Appointment and WaitlistEntry custom field declarations
- §12.4 Extension Rules: 4 rule slots
- §12.5 Extension Actions: 4 custom actions
- §12.6 Aggregate Hooks: 5 hooks with contracts
- §12.7 Extension API Endpoints: register/config endpoints
- §12.8 Extension Points Summary & Guidelines: quick-reference matrix + 7 guidelines
- §13.1 Data Migration: legacy mapping table, SAP reference, migration strategy, rollback plan
- §13.2 Deprecation & Sunset: framework and communication plan
- §14.1 Consistency Checks: all 7 checks completed
- §14.2 Decisions: DC-002, DC-003 added
- §15.2 References: business, technical, and schema references
- §15.3 Status Output Requirements: required output files and formats

## Modified Sections

- Preamble: Added "Style Guide" subsection and "When sources conflict" intro to Guidelines Compliance block
- §2 Service Identity: Added Repository and Tags rows to identity table
- §3.3.1 Appointment: Added `customFields` attribute row; updated `tenant_id` description
- §3.3.1 Reservation: Added `tenantId` attribute; added Collection Constraints and Invariants
- §3.3.2 WaitlistEntry: Added `updatedAt`, `version` attributes; corrected `constraintsJson` type reference
- §4.3 Data Validation Rules: Expanded with more fields, added cross-field validations
- §5.2 UC-010 AddToWaitlist: Added missing use case
- §7.2 Published Events: Renamed from summary table to full event-per-section format
- §8.2 Conceptual Data Model: Updated ER diagram to include `custom_fields`, `updated_at`, `apt_outbox_events`
- §8.3 apt_reservation: Added `tenant_id`, `created_at`, `updated_at` columns; added indexes
- §8.3 apt_waitlist_entry: Added `custom_fields`, `version`, `updated_at` columns; added indexes
- §12.2/12.3 renumbered: Former §12.2 Extension Events → §12.3; former §12.3 Aggregate Hooks → §12.6
- §14.3 Open Questions: Renumbered from Q-001–Q-004 to Q-APT-001–Q-APT-004; added Q-APT-005 through Q-APT-015
- §15.4 Change Log: Added v1.2 entry

## Removed Sections

- None (non-destructive upgrade per prompt Core Principles §1)

## Decisions Taken

- `apt_appointment` and `apt_waitlist_entry` are marked extensible (customer-facing aggregates per §12.2 criteria)
- `custom_fields` excluded from thin event payloads (ADR-011 compliance; DC-002)
- Appointment COMPLETED transition triggered by `srv.ses.session.completed` event, not direct API call (DC-003)
- Extension events use distinct `srv.apt.ext.*` routing key namespace to avoid collision with integration events in §7
- `constraintsJson` documented as `TimeConstraints` shared type in §3.5 rather than leaving as opaque JSON
- All timestamps changed from TIMESTAMP to TIMESTAMPTZ in table definitions for timezone correctness

## Open Questions Raised

- Q-APT-001: Slots as first-class entities vs derived from srv.res + shared.cal
- Q-APT-002: Minimum policy set for MVP (cutoffs, waitlist)
- Q-APT-003: ABAC constraint for customer self-service
- Q-APT-004: Exact performance targets (slot discovery p95, throughput, concurrent users)
- Q-APT-005: Port assignment for srv-apt-svc
- Q-APT-006: Repository URI for srv-apt-svc
- Q-APT-007: Minimum slot duration enforcement from service offering
- Q-APT-008: OpenAPI documentation URL
- Q-APT-009: Message broker type confirmation (RabbitMQ assumed)
- Q-APT-010: Auto-cancel vs flag behavior when offering is deactivated
- Q-APT-011: Sync vs async integration pattern with srv.ent for eligibility
- Q-APT-012: Whether srv.apt should emit appointment.completed event
- Q-APT-013: HIPAA applicability for healthcare deployments
- Q-APT-014: Projected booking volumes for capacity planning
- Q-APT-015: Complete feature dependency list beyond F-SRV-002/004/007
