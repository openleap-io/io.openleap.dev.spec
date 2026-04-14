# Spec Changelog — tech.jc

## Summary

- Written from scratch: the previous file was a 14-line migration stub (migration checklist only; no existing content lost)
- Full TPL-SVC v1.0.0 compliance achieved (~95%) — all 16 sections (§0–§15) present
- Three aggregates specified: `Runner`, `JobType` (with child entity `JobTemplate`), `Job`
- 10 business rules defined (BR-JC-001 through BR-JC-010) with full detail blocks for 8 key rules
- 13 use cases catalogued; detailed Actor/Preconditions/Main Flow/Postconditions for 5 key UCs
- 18 REST endpoints documented with request/response JSON examples for 6 key operations
- 5 published job events + 1 runner event documented with full ADR-011 thin-event envelopes
- 1 consumed event (iam.tenant.deleted) with handler, queue config, and failure handling
- 5 database tables defined with full column/index definitions per ADR-016/ADR-020/ADR-021
- All 5 extension point types covered in §12 (custom fields, events, rules, actions, hooks)
- 7 open questions raised (Q-JC-001 through Q-JC-007)

## Added Sections

- Preamble (template compliance header, migration note superseded by full spec)
- `## Specification Guidelines Compliance` (Non-Negotiables, Source Priority, Style Guide)
- `## 0.` Document Purpose & Scope (0.1–0.4 with full related documents list)
- `## 1.` Business Context (1.1–1.5 including Mermaid context diagram with runners and downstream consumers)
- `## 2.` Service Identity (metadata table + team table)
- `## 3.` Domain Model (3.1–3.5: conceptual overview, class diagram, 3 aggregate definitions with attribute tables/lifecycle state machines/invariants/events, 4 enumerations, 1 shared type)
- `## 4.` Business Rules (4.1 catalog table, 4.2 detailed definitions for BR-JC-001 through BR-JC-008, 4.3 field/cross-field validations, 4.4 reference data dependencies)
- `## 5.` Use Cases & Business Logic (5.1 placement table, 5.2 catalog + detail for 5 UCs, 5.3 process flow diagram reference, 5.4 cross-domain workflow table)
- `## 6.` REST API (6.1 resource/operation matrix, 6.2 request/response JSON for 6 endpoints, 6.3 business operations table, 6.4 OpenAPI reference)
- `## 7.` Integration & Events (7.1 architecture pattern, 7.2 full event envelopes for 6 events, 7.3 consumed event with handler/queue/failure handling, 7.4 flow diagram reference, 7.5 upstream/downstream integration points summary)
- `## 8.` Data Model (8.1 storage technology, 8.2 Mermaid ER diagram, 8.3 DDL for 5 tables with columns/indexes/relationships/retention, 8.4 reference data dependencies)
- `## 9.` Security (9.1 data classification table, 9.2 RBAC matrix + runner authentication + RLS description, 9.3 compliance table)
- `## 10.` Quality Attributes (10.1 performance targets + concurrency, 10.2 availability/RTO/RPO + failure scenarios, 10.3 scalability + capacity planning, 10.4 maintainability + health/metrics/alerting)
- `## 11.` Feature Dependencies (11.1 purpose, 11.2 register of F-TECH-003-xx, 11.3 endpoints per feature, 11.4 BFF hints, 11.5 impact assessment)
- `## 12.` Extension Points (12.1 purpose, 12.2 custom fields for JobType and Job — Runner non-extensible, 12.3 extension events, 12.4 extension rules, 12.5 extension actions, 12.6 aggregate hooks, 12.7 stub, 12.8 summary + guidelines)
- `## 13.` Migration & Evolution (13.1 SAP SM36/SM37 migration table, 13.2 API path deprecation framework)
- `## 14.` Metadata & Open Questions (14.1 consistency checks — all 7 PASS, 14.2 decisions, 14.3 open questions Q-JC-001 through Q-JC-007, 14.4 local ADRs, 14.5 suite ADR references)
- `## 15.` Appendix (15.1 13-term glossary, 15.2 references, 15.3 status output, 15.4 change log)

## Modified Sections

- None — previous file was a stub with no substantive domain content.

## Removed Sections

- None (non-destructive upgrade). The original 14-line migration checklist is superseded by the full spec; no domain content was removed.

## Decisions Taken

- JC does not own cron scheduling — out of scope per §0.3; documented as Q-JC-001
- `fieldValues` stored as JSONB, not relational columns — job type parameter sets are defined at runtime and not known at schema design time
- Runner callbacks use HTTP POST (not events) — runners may not have RabbitMQ connectivity; simpler integration for diverse runner implementations
- Job retention policy: 90 days default — balances operational visibility with storage cost
- `jc_outbox_events` has no `tenant_id` — outbox is a technical relay table; `tenantId` lives in the event payload envelope
- Suite prefix `tech` confirmed per ADR-TECH-004; historical alias `/api/platform/jc/v1` deprecated per ADR-TECH-005 with 6-month sunset
- Runner re-registration of same slug documented as Q-JC-003 (ambiguity)
- Runner and JobTemplate marked as non-extensible with custom fields; JobType and Job are extensible

## Open Questions Raised

- Q-JC-001: Should JC include cron/scheduled job triggering or remain pure dispatch?
- Q-JC-002: Synchronous vs asynchronous runner reachability check at registration time
- Q-JC-003: Re-registration of OFFLINE runner with same slug — reactivate vs new record
- Q-JC-004: §5.3 process flow diagrams — standalone or cross-reference suite spec?
- Q-JC-005: OpenAPI spec population timeline for `contracts/http/tech/jc/openapi.yaml`
- Q-JC-006: Extension management API endpoints — document in spec or reference module docs?
- Q-JC-007: SAP SM36/SM37 migration strategy — reimplementation vs adapter runner
