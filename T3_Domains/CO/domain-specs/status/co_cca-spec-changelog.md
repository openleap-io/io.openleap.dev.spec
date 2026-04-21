# Spec Changelog - co.cca

## Summary
- Added Source of Truth Priority to Specification Guidelines Compliance block
- Expanded PlanData and CostCenterBalance to full aggregate definitions with attribute tables (3.3.4, 3.3.5)
- Added shared types Money and FiscalPeriod (3.5)
- Added PlanType and PostingOrigin enumerations (3.4)
- Added detailed rule definitions for BR-001 through BR-010 (previously only BR-006 and BR-009 had details)
- Added new business rules BR-011 through BR-014 for PlanData and CostCenterBalance
- Added UC-007 ImportPlanData use case with full flow details
- Expanded all use cases with missing preconditions, postconditions, alternative/exception flows
- Added event envelope format to all published events in 7
- Added 7.5 Integration Points Summary (upstream dependencies, downstream consumers)
- Added 8.1 Storage Technology section and 8.3 full column-level table definitions for all 6 tables
- Added co_cca_outbox_events table per ADR-013
- Populated 11 Feature Dependencies with provisional feature mapping and BFF hints
- Populated 12 Extension Points with all 5 types (custom fields, events, rules, actions, hooks)
- Added custom_fields JSONB column to CostCenter and CostElement tables
- Added 13.1 SAP CO-CCA migration mapping (CSKS, CSKB, COEP, COSP/COSS, PLKO)
- Added 14.1 Consistency Checks (all passing)
- Added 14.5 Suite-Level ADR References
- Added new open questions Q-CCA-004 through Q-CCA-007

## Added Sections
- 3.3.4 PlanData aggregate definition
- 3.3.5 CostCenterBalance aggregate definition
- 3.4 PlanType enumeration
- 3.4 PostingOrigin enumeration
- 3.5 Money shared type
- 3.5 FiscalPeriod shared type
- 5.2 UC-007 ImportPlanData
- 6.2.6 Cost Elements - Create
- 6.2.7 Cost Elements - List
- 6.2.8 Plan Data - Import
- 6.3 Deactivate Cost Center operation
- 7.2 PlanData.imported event
- 7.5 Integration Points Summary
- 8.1 Storage Technology
- 8.3 Full table definitions (6 tables with columns, indexes, relationships, retention)
- 11.1-11.5 Feature Dependencies (full section)
- 12.1-12.8 Extension Points (full section)
- 14.1 Consistency Checks
- 14.2 Decisions & Conflicts
- 14.5 Suite-Level ADR References

## Modified Sections
- Preamble: Added Source of Truth Priority block, updated compliance to ~95%
- 3.3.1 CostCenter: Added customFields, createdAt, updatedAt attributes
- 3.3.2 CostElement: Added lifecycle states, state descriptions, allowed transitions, customFields, createdAt, updatedAt
- 3.3.3 CostPosting: Added reversalOfId, isReversal attributes
- 4.1: Added BR-011 through BR-014
- 4.2: Added detailed definitions for BR-001 through BR-005, BR-007, BR-008, BR-010
- 4.3: Added field validations for responsiblePersonId, companyCode, controllingArea, elementCode, planAmount
- 4.4: Added GL Accounts and Business Partners as reference data dependencies
- 5.2: Added full flow details to UC-002, UC-004, UC-005; added alternative/exception flows to UC-001, UC-003, UC-006
- 6.2.2: Added full JSON response body
- 6.2.3: Added business rules and events published
- 6.3: Changed URL pattern to colon-separated (`:activate`, `:deactivate`, `:close`)
- 7.2: Added event envelope format to all events, added "When Published" field, added Handler column to consumers
- 7.3: Added handler class names and prefetch count to queue configs
- 8.2: Added custom_fields and reversal columns to ER diagram
- 8.4: Added plan_type to internal code lists
- 9.3: Added GoBS regulation and Compliance Controls sub-section
- 10.3: Added plan data capacity, event volume
- 10.4: Added DLQ depth and balance drift alerts
- 13.1: Replaced generic legacy mapping with SAP CO-CCA specific tables
- 13.2: Added deprecation communication plan framework
- 14.3: Added Q-CCA-004 through Q-CCA-007
- 15.1: Added Plan Data, Materialized Balance, Reconciliation to glossary
- 15.2: Added SAP CO-CCA and HGB 257 references

## Removed Sections
- None (non-destructive upgrade)

## Decisions Taken
- D-CCA-001: Adopted colon-separated URL pattern for business operations (`:activate`, `:deactivate`, `:close`) per template convention
- CostPosting marked as non-extensible (no custom fields) due to high volume and immutability
- CostCenter and CostElement marked as extensible (custom fields supported)

## Open Questions Raised
- Q-CCA-004: Feature catalog dependency (SS6 not authored)
- Q-CCA-005: PlanData bulk import API design
- Q-CCA-006: Commitment amount update from purchase order events
- Q-CCA-007: Manual cost posting creation via REST
