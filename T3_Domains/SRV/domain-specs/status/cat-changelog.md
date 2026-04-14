# Spec Changelog - srv.cat (Service Catalog)

## Summary

- Upgraded `srv_cat-spec.md` to full TPL-SVC v1.0.0 compliance (100%)
- Added §3.5 Shared Types for `ServiceDuration` value type
- Added detailed rule definitions for BR-002, BR-004, BR-005; cross-field validations and reference data dependencies (§4.3–§4.4)
- Added §5.3 Process Flow Diagrams and §5.4 Cross-Domain Workflows
- Expanded §6.2.2–6.2.4 with full JSON bodies; added §6.4 OpenAPI reference
- Expanded §7.2 event payload/envelope JSON for all 4 events; added §7.3 consumed events framework; added §7.4 Event Flow Diagrams; renamed §7.5 Integration Points
- Added outbox table and `custom_fields JSONB` column in §8.3; added §8.4 Reference Data Dependencies
- Expanded §9.2 permission matrix; §9.3 full compliance controls
- Added throughput/concurrency to §10.1; added §10.4 Maintainability
- Added §11.3–§11.5 (Feature Dependencies detail)
- Full §12 restructure: custom fields, extension events, rules, actions, hooks, API endpoints, summary
- Full migration and deprecation frameworks in §13
- All 7 consistency checks in §14.1; added Q-CAT-007–Q-CAT-009

## Added Sections

- §3.5 Shared Types (`ServiceDuration`)
- §4.4 Reference Data Dependencies
- §5.3 Process Flow Diagrams
- §5.4 Cross-Domain Workflows
- §6.4 OpenAPI Specification reference
- §7.4 Event Flow Diagrams
- §8.4 Reference Data Dependencies
- §10.4 Maintainability
- §11.3 Endpoints per Feature
- §11.4 BFF Aggregation Hints
- §11.5 Impact Assessment
- §12.2 Custom Fields (extension-field)
- §12.4 Extension Rules (3 slots)
- §12.5 Extension Actions (4 actions)
- §12.7 Extension API Endpoints
- §12.8 Extension Points Summary & Guidelines

## Modified Sections

- **Preamble**: Compliance → 100%; version → 2026-04-03; service version → 1.2.0
- **§1.5**: Added ASCII context diagram
- **§3.3.1**: Added `customFields` attribute; enhanced attribute descriptions
- **§3.3 Child Entities**: Added invariants cross-references
- **§4.1**: Added BR-005 (Active Transition Guard)
- **§4.2**: Added full definitions for BR-002, BR-004, BR-005; enhanced BR-001, BR-003 with examples
- **§4.3**: Added variant/requirement field validations and cross-field rules
- **§5.2**: Added Main Flow/Alt/Exception blocks for UC-003, UC-004
- **§6.2.1–6.2.4**: Added JSON request/response bodies for all endpoints
- **§7.2**: Added payload and envelope JSON for all 4 events
- **§7.3**: Added consumed events framework (pending Q-CAT-003)
- **§8.2**: Added `cat_outbox_events` to ER diagram
- **§8.3 cat_service_offering**: Added `custom_fields JSONB` column + GIN index
- **§8.3**: Added `cat_outbox_events` table definition
- **§9.2**: Added Permission Matrix
- **§9.3**: Added GDPR/SOX compliance controls
- **§10.1**: Added throughput and concurrency
- **§10.2**: Added RTO/RPO
- **§10.3**: Added capacity planning table
- **§12.1**: Added purpose and 5 extension types table
- **§12.3** (renamed from §12.2): Extension Events; added 2 new extension events
- **§12.6** (renamed from §12.3): Aggregate Hooks; added 3 new hooks + Hook Contract table
- **§13.1**: Full migration framework
- **§13.2**: Deprecation framework
- **§14.1**: Added 3 missing consistency checks (now all 7 present)
- **§14.2**: Added DC-002, DC-003
- **§14.3**: Renumbered to Q-CAT-NNN; added Q-CAT-007–Q-CAT-009
- **§15.1**: Added glossary entries
- **§15.2**: Added ADR reference list

## Removed Sections

None — non-destructive upgrade. All existing content preserved.

## Decisions Taken

- **ServiceOffering is extensible** (custom_fields JSONB): customer-facing, known deployment variance
- **OfferingVariant and Requirement are NOT extensible**: narrow sub-entities; CUSTOM type covers product customization
- **Extension events use `srv.cat.ext.*`** to distinguish from integration events (`srv.cat.service.*`)
- **BR-005 added** to make activation precondition explicit and referenceable
- **§12 restructured** to match template 12.1–12.8 order

## Open Questions Raised

- Q-CAT-007: Should requirement `(type, refId)` uniqueness be hard constraint or warning?
- Q-CAT-008: Confirm message broker technology (RabbitMQ assumed)
- Q-CAT-009: Should retirement be blocked if active future bookings exist?
