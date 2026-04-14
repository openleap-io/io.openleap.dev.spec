# Spec Changelog — tech.rpt

## Summary

- Written from scratch: the previous file was a 14-line migration stub (migration checklist only; no existing domain content lost)
- Full TPL-SVC v1.0.0 compliance achieved (~95%) — all 16 sections (§0–§15) present
- Two aggregates specified: `ReportTemplate` (with child entity `TemplateVersion`) and `RenderJob`
- 8 business rules defined (BR-RPT-001 through BR-RPT-008) with full detail blocks for 6 key rules
- 11 use cases catalogued; detailed Actor/Preconditions/Main Flow/Postconditions for 5 key UCs
- 11 REST endpoints documented with request/response JSON examples for 6 key operations
- 5 published events (template.created, template.activated, render.requested, render.completed, render.failed) documented with full ADR-011 thin-event envelopes; confirmed against companion `contracts/events/tech/rpt/*.schema.json` files
- 1 consumed event (`iam.tenant.deleted`) with handler class, queue config, and failure handling
- 4 database tables defined with full column/index/relationship/retention definitions per ADR-016/ADR-020/ADR-021
- All 5 extension point types covered in §12 (custom fields on 2 aggregates, 4 extension events, 3 rule slots, 2 action zones, 2 aggregate hooks)
- 11 open questions raised (Q-RPT-001 through Q-RPT-011)
- SAP ADS / Smart Forms / SAPscript migration mapping documented in §13.1

## Added Sections

- Preamble (template compliance header; migration stub superseded by full spec)
- `## Specification Guidelines Compliance` (Non-Negotiables, Source Priority, Style Guide)
- `## 0.` Document Purpose & Scope (0.1–0.4 with full related documents list including companion event schemas)
- `## 1.` Business Context (1.1–1.5 including Mermaid context diagram showing DMS, JC, IAM, and T2–T4 caller topology)
- `## 2.` Service Identity (metadata table + team table)
- `## 3.` Domain Model (3.1–3.5: conceptual overview, class diagram, 2 aggregate definitions with attribute tables/lifecycle state machines/invariants/events, 1 child entity TemplateVersion, 5 enumerations, 1 shared value object DataSourceConfig)
- `## 4.` Business Rules (4.1 catalog table, 4.2 detailed definitions for BR-RPT-001 through BR-RPT-008, 4.3 field/cross-field validations, 4.4 reference data dependencies)
- `## 5.` Use Cases & Business Logic (5.1 placement table, 5.2 catalog of 11 UCs + detail for 5 key UCs, 5.3 stub reference, 5.4 cross-domain workflows table with render pipeline step-by-step breakdown)
- `## 6.` REST API (6.1 resource/operation matrix, 6.2 full request/response JSON for 6 endpoints, 6.3 business operations table, 6.4 OpenAPI reference)
- `## 7.` Integration & Events (7.1 architecture pattern, 7.2 full event envelopes for 5 published events with known consumers, 7.3 consumed event with handler/queue/failure handling, 7.4 stub, 7.5 upstream/downstream integration points summary)
- `## 8.` Data Model (8.1 storage technology, 8.2 Mermaid ER diagram, 8.3 DDL for 4 tables with columns/indexes/relationships/retention, 8.4 reference data dependencies)
- `## 9.` Security (9.1 data classification including credential classification, 9.2 RBAC permission matrix + RLS + credential encryption, 9.3 GDPR compliance table)
- `## 10.` Quality Attributes (10.1 performance targets + concurrency, 10.2 availability/RTO/RPO + failure scenarios table, 10.3 scalability + capacity planning, 10.4 maintainability + health/metrics/alerting)
- `## 11.` Feature Dependencies (11.1 purpose, 11.2 register of F-TECH-002 and children, 11.3 endpoints per feature, 11.4 BFF aggregation hints, 11.5 impact assessment)
- `## 12.` Extension Points (12.1 purpose, 12.2 custom fields — ReportTemplate Yes, RenderJob Yes, TemplateVersion No; 12.3 4 extension events; 12.4 3 rule slots; 12.5 2 action zones; 12.6 2 aggregate hooks; 12.7 stub; 12.8 summary matrix + guidelines)
- `## 13.` Migration & Evolution (13.1 SAP ADS/Smart Forms/SAPscript migration table, 13.2 `/api/t1/rpt/v1` deprecation sunset table + communication plan)
- `## 14.` Metadata & Open Questions (14.1 consistency checks — all 7 PASS, 14.2 decisions & conflicts, 14.3 Q-RPT-001 through Q-RPT-011, 14.4 domain ADRs, 14.5 suite-level ADR references)
- `## 15.` Appendix (15.1 13-term glossary, 15.2 references table, 15.3 status output requirements, 15.4 change log)

## Modified Sections

- None — previous file was a stub with no substantive domain content.

## Removed Sections

- None (non-destructive upgrade). The original 14-line migration checklist is superseded by the full spec; no domain content was removed.

## Decisions Taken

- JRXML/JASPER files stored in DMS (not in RPT DB) — consistent with platform binary storage policy
- Render dispatch via JC (`pdf-render` job type) rather than embedded Jasper engine — aligns with JC's mandate; runner scalable independently of RPT
- `dataSourceConfig` credentials encrypted at service layer (not dedicated KMS) in v1.0.0 — see Q-RPT-010
- `callerCallbackUrl` notifications are fire-and-forget (not retried) — callers who need reliability should poll `GET /renders/{id}` or consume events
- `versionNumber` auto-incremented by service (not caller-provided) — prevents gaps and conflicts
- No `template.deprecated` event in v1.0.0 — no downstream consumers identified; see Q-RPT-011
- TemplateVersion marked as non-extensible (custom fields: No) — binary content integrity contract; version metadata is immutable
- 90-day retention for RenderJobs — balances operational visibility with storage cost

## Open Questions Raised

- Q-RPT-001: Service port assignment (tentative 8097 in spec)
- Q-RPT-002: Scheduled / recurring render jobs — should RPT or a dedicated scheduler own this?
- Q-RPT-003: Deprecation guard — what happens to PENDING/RUNNING render jobs when a template is deprecated?
- Q-RPT-004: Maximum template versions per template (tentative: 100)
- Q-RPT-005: JRXML validation depth at upload — parse/validate or accept any binary?
- Q-RPT-006: Process flow diagrams (§5.3 / §7.4) — standalone or cross-reference suite spec?
- Q-RPT-007: OpenAPI specification population timeline for `contracts/http/tech/rpt/openapi.yaml`
- Q-RPT-008: PENDING RenderJob timeout monitoring — who detects and transitions stuck PENDING jobs?
- Q-RPT-009: Extension management API endpoints — document in spec or reference core-extension module docs?
- Q-RPT-010: Credential encryption strategy — service-layer AES-256 vs. platform KMS
- Q-RPT-011: template.deprecated event — needed in future versions?
