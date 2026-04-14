# Spec Changelog — tech.dms

## Summary

- Written from scratch: the previous file was a 14-line migration stub; no existing content was lost
- Full TPL-SVC v1.0.0 compliance achieved (~95%) — all 16 sections (§0–§15) present
- Three aggregates specified: `Document`, `DocumentVersion`, `RetentionPolicy`
- 10 business rules defined (BR-DMS-001 through BR-DMS-010) with full detail blocks
- 13 use cases catalogued; detailed Actor/Preconditions/Flow/Postconditions for 7 key UCs
- 19 REST endpoints documented with request/response JSON examples for 5 key operations
- 7 published events and 1 consumed event documented with full ADR-011 thin-event envelopes
- 5 database tables defined with full column/index DDL per ADR-016/ADR-020/ADR-021
- All 5 extension point types covered in §12 (custom fields, events, rules, actions, hooks)
- 5 open questions raised (Q-DMS-001 through Q-DMS-005)

## Added Sections

- Preamble (template compliance header, meta information block)
- `## Specification Guidelines Compliance` (Non-Negotiables, Source Priority, Style Guide)
- `## 0.` Document Purpose & Scope (0.1–0.4)
- `## 1.` Business Context (1.1–1.5 including Mermaid context diagram)
- `## 2.` Service Identity (metadata table + team table)
- `## 3.` Domain Model (3.1–3.5: conceptual overview, class diagram, 3 aggregate definitions with attribute tables/lifecycle/invariants/events, 5 enumerations, 2 shared types)
- `## 4.` Business Rules (4.1 catalog, 4.2 detailed definitions, 4.3 field/cross-field validations, 4.4 reference data dependencies)
- `## 5.` Use Cases & Business Logic (5.1–5.4: placement table, 13 UCs with detail for 7, virus-scan sequence diagram stub, cross-domain workflow table)
- `## 6.` REST API (6.1 overview table, 6.2 request/response JSON for 5 endpoints, 6.3 business operations table, 6.4 OpenAPI reference)
- `## 7.` Integration & Events (7.1–7.5: architecture pattern, 7 published event full envelopes, consumed event with GDPR purge handler, Mermaid sequence diagram, integration points summary)
- `## 8.` Data Model (8.1 storage technology, 8.2 Mermaid ER diagram, 8.3 SQL DDL for 5 tables, 8.4 reference data dependencies)
- `## 9.` Security (9.1 data classification, 9.2 RBAC matrix, 9.3 compliance table)
- `## 10.` Quality Attributes (10.1–10.4: performance targets, availability/RTO/RPO, scalability, maintainability)
- `## 11.` Feature Dependencies (11.1–11.5 covering F-TECH-001-01/02/03)
- `## 12.` Extension Points (12.1–12.8: Document extensible, DocumentVersion/RetentionPolicy non-extensible, extension events/rules/actions/hooks)
- `## 13.` Migration & Evolution (13.1 SAP ArchiveLink/GOS migration table, 13.2 deprecation framework)
- `## 14.` Metadata & Open Questions (14.1 consistency checks all PASS, 14.2 decisions, 14.3 open questions, 14.4 local ADRs, 14.5 suite ADR refs)
- `## 15.` Appendix (15.1 17-term glossary, 15.2 references, 15.3 status output, 15.4 change log)

## Modified Sections

- None — previous file was a stub with no substantive content.

## Removed Sections

- None (non-destructive upgrade). The original migration checklist content is superseded by the full spec but no domain content was removed.

## Decisions Taken

- `RetentionPolicy` is a system-level aggregate (no `tenant_id`) — retention is a compliance requirement, not a per-tenant configuration
- DRAFT → ACTIVE transition is gated on virus scan CLEAN result (scan-gated activation)
- Presigned URL TTL: 15 minutes upload, 60 minutes download (asymmetric)
- `latest_version_id` is NOT updated for versions with `INFECTED` scan status
- ClamAV worker uses a push model: subscribes to `tech.dms.document.version.created` and calls `PATCH /versions/{id}/scan-status`
- German GoBD references use COMPLIANCE-mode S3 Object Lock; GDPR erasure conflict documented as Q-DMS-001
- Suite prefix `tech` confirmed per ADR-TECH-004

## Open Questions Raised

- Q-DMS-001: GDPR right-to-erasure vs GoBD COMPLIANCE-locked objects
- Q-DMS-002: Search index strategy (DMS event → Elasticsearch vs CDC/Debezium)
- Q-DMS-003: Per-tenant document type configuration in future version
- Q-DMS-004: ClamAV deployment topology (separate microservice vs sidecar/job)
- Q-DMS-005: MinIO SSE mode (SSE-S3 vs SSE-KMS per tenant)
