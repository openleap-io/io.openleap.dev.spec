# Spec Changelog - sd.ord (Sales Orders)

## Summary

- Upgraded `sd_ord-spec.md` from ~88% to ~95% TPL-SVC v1.0.0 compliance
- Added full entity attribute tables for all 4 child entities (PartnerFunction, OrderBlock, DocumentNote, SalesDocumentLine)
- Added 4 new enumerations (LineStatus, BlockType, NoteType, PartnerFunctionCode) and expanded DocumentType/DocumentStatus descriptions
- Formalised Shared Types section with Money and PriceSnapshot type definitions
- Added detailed BR definitions for BR-ORD-003 through BR-ORD-011 (9 rules total now fully specified)
- Expanded §6.2 from bullet-point list to 9 full request/response endpoint specifications
- Expanded §7.2 from summary table to per-event payload + envelope + known consumers (8 events)
- Expanded §7.3 from summary table to per-consumer handler class, queue config, and failure handling (5 handlers)
- Added §7.4 event flow diagram
- Added full column definitions for all 5 data tables plus outbox table in §8.3
- Replaced 4 OPEN QUESTION stubs with full content (§10.3, §10.4, §11, §12)
- Added SAP SD migration mapping in §13.1; added deprecation framework in §13.2
- Filled in all 7 consistency checks in §14.1

## Added Sections

- §3.4 — LineStatus, BlockType, NoteType, PartnerFunctionCode enumerations (new)
- §3.5 — Shared Types: Money and PriceSnapshot (replaced stub)
- §4.2 — BR-ORD-003 through BR-ORD-011 detailed definitions (8 new rule blocks)
- §6.2.1 through §6.2.9 — Full endpoint request/response specifications (expanded from bullet list)
- §7.2 — Per-event detailed blocks with payload, envelope, and known consumers
- §7.3 — Per-consumer handler: CreditCheckResultHandler, CustomerAddressUpdateHandler, DeliveryCompletedHandler, InvoiceCreatedHandler, AgreementActivatedHandler
- §7.4 — Event flow diagram (new subsection)
- §7.5 — Integration Points Summary (was §7.4; content preserved, renumbered)
- §8.3 — Full column tables for: ord_sales_documents, ord_lines, ord_partners, ord_blocks, ord_notes, ord_outbox_events
- §10.3 — Scalability: horizontal scaling, read replicas, consumer scaling, capacity planning (replaced stub)
- §10.4 — Maintainability: API versioning, backward compatibility, monitoring, alerting (replaced stub)
- §11.1 — Feature Dependencies Purpose (replaced stub)
- §11.2 — Provisional Feature Dependency Register (F-SD-ORD-001 through F-SD-ORD-006)
- §11.3 — Endpoints per Feature table
- §11.4 — BFF Aggregation Hints
- §11.5 — Impact Assessment
- §12.1 — Extension Points Purpose
- §12.2 — Custom Fields for SalesDocument and SalesDocumentLine
- §12.3 — 7 Extension Event slots
- §12.4 — 4 Extension Rule slots
- §12.5 — 3 Extension Action slots
- §12.6 — 6 Aggregate Hooks with contracts
- §12.7 — Extension API Endpoints
- §12.8 — Extension Points Summary & Guidelines
- §13.1 — SAP SD data migration mapping (replaced future extensions list, moved to §13.2 roadmap)
- §13.2 — Deprecation & Sunset framework (replaced stub)
- §14.1 — All 7 consistency checks filled in with Pass/Partial status

## Modified Sections

- **Preamble** — Updated version to 2026-04-04, compliance to ~95%
- **§2 Service Identity** — Updated version to 1.2.0
- **§3.3.1 SalesDocument Aggregate Root** — Added `incotermsLocation`, `notes`, `createdAt`, `updatedAt` to attribute table; expanded all constraint descriptions
- **§3.3 SalesDocumentLine** — Added missing attributes (storageLocationCode, version, createdAt, updatedAt)
- **§3.4 DocumentType/DocumentStatus** — Expanded Value descriptions with business context
- **§4.1 Business Rules Catalog** — Added BR-ORD-011 (Active Block Prevents Confirmation)
- **§8.2 ER Diagram** — Added ORD_NOTES table and ORD_OUTBOX_EVENTS to diagram
- **§9.2 Access Control** — Added Permission Matrix table
- **§9.3 Compliance** — Added compliance controls table (retention, audit trail, erasure, portability)
- **§10.1 Performance** — Added Throughput event processing rate and Concurrency section
- **§10.2 Availability** — Added Failure Scenarios table
- **§14.2 Decisions & Conflicts** — Added DC-ORD-004 and DC-ORD-005
- **§14.3 Open Questions** — Added Q-ORD-003 through Q-ORD-007
- **§15.1 Glossary** — Added EDA, SP, SH, BT, PY, RLS, DLQ, UCUM terms
- **§15.2 References** — Added Dev Guidelines ADR reference list
- **§15.3** — Added status output requirements pointing to this file

## Removed Sections

- None (non-destructive upgrade; all existing content preserved)

## Decisions Taken

- **SalesDocumentLine extensibility:** Marked as extensible (custom_fields JSONB) — lines are customer-facing with high variance across deployments (DC-ORD-005).
- **Shared Types:** Money defined in both §3.3 (Value Object under aggregate) and §3.5 (Shared Types cross-reference); PriceSnapshot added as new shared type.
- **§7 renumbering:** Added §7.4 Event Flow Diagram; former §7.4 Integration Points Summary became §7.5. Content preserved.
- **BR-ORD-011:** Added new business rule "Active Block Prevents Confirmation" — was implicit in the design but not formally specified.
- **PartnerFunction.uk:** Added UNIQUE constraint on (sales_document_id, function_code) — each function code may appear only once per document.
- **Custom fields on PartnerFunction, OrderBlock, DocumentNote:** Decided NOT extensible — these are operational/internal entities; custom metadata should be stored on the parent SalesDocument.

## Open Questions Raised

- Q-ORD-003: Which product features depend on sd-ord-svc? (§11 feature IDs provisional)
- Q-ORD-004: SAP document number migration format?
- Q-ORD-005: OpenLeap Starter version and port assignment
- Q-ORD-006: Repository URI for sd-ord-svc
- Q-ORD-007: Scheduling Agreement delivery schedule lines — separate table or JSONB?
