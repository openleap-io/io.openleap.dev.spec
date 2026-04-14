# Spec Changelog — srv.cas

## Summary

- Upgraded `srv_cas-spec.md` from ~92% to ~95% TPL-SVC v1.0.0 compliance
- Added all missing structural sections (§1.4, §3.1, §3.5, §4.2–4.4, §5.1/5.3/5.4, §6.2–6.4, §7.1/7.4/7.5, §8.3 columns/8.4, §9.1/9.3, §10.1–10.4, §11.1–11.5, §12.1–12.5/12.7–12.8, §13.1–13.2, §14.1/14.2/14.4/14.5, §15.2–15.3)
- Expanded all 5 published events with full payload + envelope JSON examples and known consumers
- Expanded all 2 consumed events with handler class, queue config, and failure handling (ADR-014)
- Added full request/response JSON bodies for all 6 REST endpoints
- Added complete column definitions for all 3 database tables (cas_case, cas_session_link, cas_outbox_events) including `custom_fields JSONB` and GIN index
- Added §12.2 Custom Fields: Case aggregate marked extensible; SessionLink marked non-extensible
- Added all 5 extension point types (fields, events, rules, actions, hooks)
- Raised 11 new open questions (Q-CAS-001 through Q-CAS-011)
- Added Guidelines Compliance block with all 3 required sub-sections

## Added Sections

- `§1.4 Strategic Positioning`
- `§1.5 Service Context diagram` (Mermaid context graph)
- `§3.1 Conceptual Overview`
- `§3.3 State Descriptions table` and `Allowed Transitions table` for Case
- `§3.3 SessionLink Collection Constraints and Invariants`
- `§3.5 Shared Types`
- `§4.2 Detailed Rule Definitions` (BR-001, BR-002, BR-003)
- `§4.3 Data Validation Rules` (field-level and cross-field)
- `§4.4 Reference Data Dependencies`
- `§5.1 Business Logic Placement`
- `§5.3 Process Flow Diagrams` (CreateCase and SessionLink auto-link sequences)
- `§5.4 Cross-Domain Workflows` (auto-link choreography)
- `§6.2 Resource Operations` (full request/response for all 4 resource endpoints)
- `§6.3 Business Operations` (CloseCase, LinkSession with full request/response)
- `§6.4 OpenAPI Specification reference`
- `§7.1 Architecture Pattern`
- `§7.2 Published Events` — detailed blocks for all 5 events
- `§7.3 Consumed Events` — queue config and failure handling for both events
- `§7.4 Event Flow Diagrams`
- `§7.5 Integration Points Summary`
- `§8.3 Column definitions` for all 3 tables
- `§8.4 Reference Data Dependencies`
- `§9.1 Data Classification table`
- `§9.3 Compliance Requirements` (GDPR controls)
- `§10.1 Throughput and Concurrency`
- `§10.2 Availability & Reliability` (RTO/RPO, failure scenarios)
- `§10.3 Scalability`
- `§10.4 Maintainability`
- `§11.1 Purpose`
- `§11.2 Feature Dependency Register` (expanded)
- `§11.3 Endpoints per Feature`
- `§11.4 BFF Aggregation Hints`
- `§11.5 Impact Assessment`
- `§12.1 Purpose`
- `§12.2 Custom Fields` (Case: extensible; SessionLink: not extensible)
- `§12.3 Extension Events` (3 hooks)
- `§12.4 Extension Rules` (3 slots)
- `§12.5 Extension Actions` (3 slots)
- `§12.7 Extension API Endpoints`
- `§12.8 Extension Points Summary & Guidelines`
- `§13.1 Data Migration` (legacy mapping framework)
- `§13.2 Deprecation & Sunset`
- `§14.1 Consistency Checks` (7 checks, all Pass or Partial)
- `§14.2 Decisions & Conflicts` (5 decisions documented)
- `§14.4 ADRs` (candidate ADRs)
- `§14.5 Suite-Level ADR References` (16 ADRs)
- `§15.2 References`
- `§15.3 Status Output Requirements`
- `Guidelines Compliance block` — full 3-section version

## Modified Sections

- `Preamble / Guidelines Compliance` — expanded from minimal to full 3-section block
- `§2 Service Identity` — added Team sub-table; added Repository and Tags rows
- `§3.2 Core Concepts` — preserved (Mermaid class diagram unchanged)
- `§3.3 Aggregate Definitions` — preserved existing attribute tables; added State Descriptions and Allowed Transitions
- `§3.4 Enumerations` — preserved; descriptions expanded
- `§4.1 Business Rules Catalog` — added BR-003 (No Duplicate Session Link)
- `§5.2 Use Cases` — preserved canonical tables; added Actor/Preconditions/Main Flow/Postconditions for all 6 use cases
- `§6.1 API Overview` — preserved; endpoint table added
- `§8.2 Conceptual Data Model` — ER diagram updated to include `cas_outbox_events`
- `§9.2 Access Control` — preserved roles; expanded to include Permission Matrix and Data Isolation
- `§10.1 Performance Requirements` — preserved latency numbers; added throughput and concurrency
- `§14.3 Open Questions` — preserved Q-001 and Q-002; renumbered to Q-CAS-010 and Q-CAS-011; added Q-CAS-001 through Q-CAS-009

## Removed Sections

- None (non-destructive upgrade)

## Decisions Taken

- DEC-001: Case aggregate is extensible (custom_fields JSONB) — customer-facing, high cross-industry variance
- DEC-002: SessionLink is NOT extensible — thin join entity
- DEC-003: `statusChanged` event published on ALL transitions; `closed` event additionally on close
- DEC-004: Auto-link from `session.planned` is non-blocking for missing/closed cases
- DEC-005: Pre-close aggregate hook is non-blocking by default

## Open Questions Raised

- Q-CAS-001: Port assignment for srv-cas-svc
- Q-CAS-002: Repository URI for srv-cas-svc
- Q-CAS-003: Maximum session links per case
- Q-CAS-004: Entitlement validation depth at case creation
- Q-CAS-005: OpenAPI docs URL
- Q-CAS-006: Auto-activation on first appointment booking
- Q-CAS-007: Regulated industry compliance requirements
- Q-CAS-008: GDPR right-to-erasure cascade strategy
- Q-CAS-009: Exact feature IDs in SRV suite catalog
- Q-CAS-010: Case types — srv.cat vs. custom field (migrated from Q-001)
- Q-CAS-011: Plan templates ownership — srv.cas vs. ps (migrated from Q-002)
