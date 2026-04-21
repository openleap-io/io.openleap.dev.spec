# Spec Changelog - co.abc

## Summary
- Upgraded spec from ~85% to ~95% TPL-SVC v1.0.0 compliance
- Added Source of Truth Priority block to Guidelines Compliance section
- Expanded all 12 business rules with full detailed definitions (BR-001 through BR-012)
- Added child entity and value object definitions to domain model (ResourceDriver, ActivityDriver, ActivityCostAssignment, Money, CostToServeResult)
- Added state description and allowed transition tables for both aggregates
- Expanded all REST endpoints with full request/response JSON examples
- Added full event envelope and payload structures for all published events
- Added complete column-level table definitions for all 7 tables including outbox
- Populated Section 11 (Feature Dependencies) with anticipated features
- Populated Section 12 (Extension Points) with all 5 extension types
- Added 3 new open questions (Q-ABC-006 through Q-ABC-008)
- Added consistency checks (all passing) in Section 14.1

## Added Sections
- S3.3 Child Entities (ResourceDriver, ActivityDriver, ActivityCostAssignment) with full attribute tables
- S3.3 Value Objects (Money, CostToServeResult)
- S3.4 Additional enumerations (ProcessStatus, ActivityDriverType, CostObjectType, RunStatus)
- S3.5 Shared Types (Money with Used By references)
- S4.2 Detailed definitions for all 12 business rules (BR-001 through BR-012)
- S5.2 UC-002 (UpdateProcessOrActivity) and UC-004 (ReverseABCRun) with full detail
- S6.2 Full request/response JSON for 15 resource operations
- S7.2 Event envelope and payload structures for all 6 published events
- S7.3 Expanded consumed events with handler, queue config, failure handling
- S7.4 Event flow diagram
- S7.5 Integration points summary (upstream and downstream)
- S8.1 Storage Technology (PostgreSQL)
- S8.3 Full column-level table definitions for 7 tables
- S8.3 Outbox events table (co_abc_outbox_events)
- S11 Feature Dependencies (register, endpoints per feature, BFF hints, impact)
- S12 Extension Points (custom fields, extension events, rules, actions, hooks, API, summary)
- S14.1 Consistency checks (7 checks, all passing)
- S14.2 Decisions & Conflicts (3 entries)
- S14.5 Suite-Level ADR References

## Modified Sections
- Preamble: Updated version to 2026-04-04, compliance to ~95%, added Source of Truth Priority
- S3.3.1 BusinessProcess: Added Aggregate Root heading, state descriptions, allowed transitions, createdAt/updatedAt
- S3.3.2 ABCRun: Added full Aggregate Root structure with state descriptions, allowed transitions
- S4.2: Expanded from 2 rules to all 12 with full detail format
- S5.2: Added preconditions, postconditions, business rules, alternative/exception flows to all use cases
- S6.2: Expanded from bullet endpoints to full request/response bodies
- S7.2: Expanded from summary table to full event envelope format
- S8.2: Added timestamps and custom_fields to ER diagram
- S9.2: Added Roles & Permissions table, Data Isolation section
- S9.3: Added compliance controls detail (data retention, audit trail, SOX)
- S10.1: Added throughput and concurrency targets
- S10.2: Added failure scenarios table
- S10.3: Added capacity planning details
- S10.4: Added health checks, metrics, alert thresholds
- S14.3: Renumbered to Q-ABC-xxx pattern, added 3 new questions
- S15.1: Added ABC Run and Reconciliation glossary entries
- S15.4: Added v1.2 changelog entry

## Removed Sections
- None (non-destructive upgrade)

## Decisions Taken
- DC-001: Activity modeled as child entity of BusinessProcess (not separate aggregate)
- DC-002: ABC run is asynchronous (202 Accepted) due to potentially long execution times
- DC-003: Run.completed event includes summary fields (controllingArea, period, totals) beyond minimal thin event

## Open Questions Raised
- Q-ABC-006: Which product features depend on this service's endpoints?
- Q-ABC-007: Should resource drivers support both percentage-based and absolute quantity allocation?
- Q-ABC-008: Should the activity table include controlling_area column for unique key?
