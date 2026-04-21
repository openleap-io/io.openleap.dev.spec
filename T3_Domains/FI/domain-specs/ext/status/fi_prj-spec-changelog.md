# Spec Changelog - fi.prj

## Summary
- Upgraded fi.prj spec from ~60% to ~95% TPL-SVC v1.0.0 compliance
- Renumbered all sections to match template structure (§0-§15)
- Added Specification Guidelines Compliance block
- Added §2 Service Identity, §1.5 Service Context, §11 Feature Dependencies, §12 Extension Points
- Expanded §3 Domain Model with full attribute tables (Type/Format/Description/Constraints/Required/Read-Only), §3.4 Enumerations, §3.5 Shared Types
- Expanded §4 Business Rules with detailed definitions (§4.2), field-level validations (§4.3), reference data dependencies (§4.4)
- Restructured §5 Use Cases to canonical format with Actor/Preconditions/Main Flow/Postconditions; added §5.1 Business Logic Placement, §5.4 Cross-Domain Workflows
- Expanded §6 REST API with full request/response JSON bodies and error responses
- Expanded §7 Events with full event envelopes, consumed event details (handler, queue config, failure handling), §7.5 Integration Points Summary
- Expanded §8 Data Model with ER diagram, full table definitions (columns, indexes, relationships, retention), outbox table
- Expanded §9 Security with data classification (§9.1) and compliance requirements (§9.3)
- Expanded §10 Quality Attributes with availability (§10.2), scalability (§10.3), maintainability (§10.4)
- Added §13 Migration with SAP PS source mapping
- Added §14 Consistency Checks, Decisions & Conflicts, structured Open Questions, ADRs

## Added Sections
- Specification Guidelines Compliance block (preamble)
- §1.5 Service Context (responsibilities, authoritative sources, context diagram)
- §2 Service Identity (full metadata table, team table)
- §3.4 Enumerations (12 enum definitions with value tables)
- §3.5 Shared Types (Money shared type)
- §4.2 Detailed Rule Definitions (7 fully expanded rules)
- §4.3 Data Validation Rules (field-level and cross-field)
- §4.4 Reference Data Dependencies
- §5.1 Business Logic Placement table
- §5.4 Cross-Domain Workflows (Cost Capture, Capitalization)
- §6.3 Business Operations (CaptureCost, RunWIP, PostWIPRun, CapitalizeProject, CreateBillingEvent)
- §6.4 OpenAPI Specification reference
- §7.1 Architecture Pattern (with rationale)
- §7.3 Consumed Events (4 events with full handler/queue/failure details)
- §7.4 Event Flow Diagrams
- §7.5 Integration Points Summary (upstream and downstream)
- §8.2 Conceptual Data Model (Mermaid ER diagram)
- §8.4 Reference Data Dependencies
- Table: prj_outbox_events (ADR-013)
- §9.1 Data Classification
- §9.3 Compliance Requirements (SOX, IFRS 15, GDPR)
- §10.2 Availability & Reliability (RTO/RPO, failure scenarios)
- §10.3 Scalability (scaling strategy, capacity planning)
- §10.4 Maintainability (versioning, monitoring, alerting)
- §11 Feature Dependencies (full section with register, endpoints, BFF hints)
- §12 Extension Points (full section: custom fields, events, rules, actions, hooks, API, summary)
- §13.1 SAP PS migration mapping
- §13.2 Deprecation & Sunset framework
- §14.1 Consistency Checks (7 checks, all Pass)
- §14.2 Decisions & Conflicts
- §14.3 Open Questions (Q-PRJ-001 through Q-PRJ-006)
- §14.4 ADRs (ADR-PRJ-001, ADR-PRJ-002)
- §14.5 Suite-Level ADR References
- §15.2 References
- §15.3 Status Output Requirements
- §15.4 Change Log

## Modified Sections
- Preamble: Updated meta block with full template fields (bounded context, service ID, base package, port, repository, tags, team)
- §0: Added Related Documents entries for technical standards
- §1.4: Preserved existing strategic positioning diagram
- §3 (was §2): Renumbered; added full attribute tables with Format/Required/Read-Only columns to all aggregates
- §3.3.3 BillingEvent: Promoted to full aggregate definition with lifecycle states and transitions
- §4.1: Added Error Code column to catalog table
- §5 (was §3): Renumbered; restructured use cases to canonical table format; added UC-PRJ-007 (ListProjects) and UC-PRJ-008 (GetProjectProfitability)
- §6 (was §7): Renumbered; expanded all endpoints with full request/response JSON
- §7 (was §5+§6): Merged integration and events sections; expanded published events with envelopes
- §8 (was §8): Restructured table definitions from SQL DDL to template format; added version/updated_at/custom_fields columns
- §9 (was §9): Expanded access control section
- §10 (was §10): Added throughput and concurrency metrics
- §13 (was §11): Expanded migration with SAP PS table mapping
- §14 (was §12): Restructured; ADR renamed from ADR-001 to ADR-PRJ-001
- §15 (was §13): Expanded glossary with aliases column

## Removed Sections
- None (non-destructive upgrade)

## Decisions Taken
- DC-PRJ-001: Renumbered sections to match TPL-SVC v1.0.0 template (§2 Domain Model -> §3 Domain Model)
- DC-PRJ-002: Split combined Integration Architecture + Event Catalog into unified §7 Events & Integration
- Promoted BillingEvent from child entity concept to standalone aggregate (it has its own lifecycle and identity)
- Set provisional port 8480 for fi-prj-svc (flagged as open question)
- Marked Project and BillingEvent as extensible aggregates; WIPRun and Task as non-extensible

## Open Questions Raised
- Q-PRJ-001: Feature dependency register needs concrete feature spec IDs
- Q-PRJ-002: Multi-currency project support
- Q-PRJ-003: Port assignment for fi-prj-svc
- Q-PRJ-004: Overhead allocation automation
- Q-PRJ-005: Cost reversal handling
- Q-PRJ-006: WIP run project-level filtering
