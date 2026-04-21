# Spec Changelog - fi.clr

## Summary
- Upgraded fi_clr-spec.md from ~30% to ~95% TPL-SVC v1.0.0 compliance
- Added detailed enumeration tables for all 5 enums (ClearingStatus, LinkStatus, OpenItemType, MatchedBy, MatchCriteria)
- Added value objects (Money, MatchScore) and shared types (Money) with validation rules
- Completed all 8 business rule detailed definitions (BR-CLR-001 through BR-CLR-008) with examples
- Added Actor/Preconditions/Main Flow/Postconditions to all 6 use cases
- Expanded REST API with full request/response JSON for all 7 endpoints
- Added event envelope format to all 5 published events and queue config to all 5 consumed events
- Added custom_fields JSONB column to clr_clearing_cases table and full outbox table definition
- Expanded extension points to cover all 5 types: custom fields, extension events, extension rules, extension actions, aggregate hooks
- Added failure scenarios table, capacity planning, monitoring & alerting
- Added SAP FI-GL migration reference (BSEG, BKPF, F.13)
- Added consistency checks (all Pass), suite-level ADR references, 4 new open questions

## Added Sections
- Section 3.3: Value Objects (Money, MatchScore) under ClearingCase aggregate
- Section 3.4: Detailed enumeration tables with descriptions and deprecated flags
- Section 3.5: Shared Types (Money) with used-by references
- Section 4.2: Detailed definitions for BR-CLR-004 through BR-CLR-008
- Section 6.2.2: ClearingCase Retrieve (full response body)
- Section 6.2.3: ClearingCase List (full paginated response)
- Section 6.2.5: ClearingLink Reject (full response body)
- Section 6.2.6: Suggestions Query (expanded response with criteria details)
- Section 6.3: Close operation (full request/response)
- Section 6.3: MatchingRule Management endpoints table
- Section 7: Event envelope format for all published events
- Section 7.3: Queue configuration and failure handling for all consumed events
- Section 8.3: Full outbox table definition with columns and indexes
- Section 8.5: Reference Data Dependencies
- Section 10.2: Failure Scenarios table
- Section 10.3: Capacity Planning table
- Section 12.2: Custom Fields (extension-field) for ClearingCase and MatchingRule
- Section 12.3: Extension Events (4 hooks with routing keys)
- Section 12.4: Extension Rules (3 rule slots)
- Section 12.5: Extension Actions (3 action slots)
- Section 12.6: Expanded aggregate hooks (6 hooks with full contracts)
- Section 12.7: Extension API Endpoints
- Section 12.8: Extension Points Summary & Guidelines
- Section 14.1: Consistency checks (7 checks, all Pass)
- Section 14.5: Suite-Level ADR References
- Section 15.3: Status Output Requirements

## Modified Sections
- Preamble: Added "Keep examples minimal" to style guide
- Section 0.5: Renamed to "Related Documents" per template; added TECHNICAL_STANDARDS.md reference
- Section 1.5: Added Responsibilities list and Authoritative Sources table
- Section 3.3.1: Added Read-Only column to attribute table; added currencyCode and customFields attributes; added State Descriptions table
- Section 3.3.2: Added Collection Constraints and Invariants tables for ClearingLink
- Section 3.3.3: Added version, created_at, updated_at attributes to MatchingRule
- Section 6.1: Structured as table format
- Section 6.4: Added 412 and 503 status codes
- Section 6.5: Added Documentation URL
- Section 7.2: Added full event envelope JSON for each published event
- Section 8.2: Added currency_code and custom_fields to ER diagram
- Section 8.3: Added custom_fields JSONB column and GIN index to clr_clearing_cases
- Section 8.3: Added version, created_at, updated_at to clr_matching_rules
- Section 10.1: Split into Performance, Throughput, and Concurrency sub-tables
- Section 10.4: Added Monitoring & Alerting sub-section
- Section 14.1: Replaced simple yes/no with Pass/Fail per template
- Section 14.2: Added D-CLR-004 (extension vs integration events)
- Section 14.3: Added Q-CLR-005 through Q-CLR-008
- Section 15.1: Added Match Score and Custom Fields to glossary
- Section 15.2: Added ADR-067 reference
- Section 15.4: Added v3.0 changelog entry

## Removed Sections
- None (non-destructive upgrade)

## Decisions Taken
- D-CLR-004: Extension events use separate `fi.clr.ext.*` routing namespace to avoid conflating with integration events
- ClearingCase is extensible (custom fields); MatchingRule is not (configuration entity, not customer-facing)
- Currency code added as explicit attribute (was implicit via voucher reference)
- SAP FI-GL (BSEG/BKPF/F.13) used as migration reference source

## Open Questions Raised
- Q-CLR-005: Feature dependency register requires Phase 3 feature spec IDs
- Q-CLR-006: Multi-currency clearing support decision
- Q-CLR-007: Maximum custom_fields JSONB size limit
- Q-CLR-008: Customer/vendor-based matching criteria for MatchingRule
