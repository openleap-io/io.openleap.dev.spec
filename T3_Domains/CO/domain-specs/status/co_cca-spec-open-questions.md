# Open Questions - co.cca

## Q-CCA-001: Statistical Key Figures
- **Question:** Should statistical key figures (headcount, sqm) be managed in CCA or a separate service?
- **Why it matters:** Allocation drivers in co.om need statistical data. If CCA owns it, CCA's data model grows; if a separate service owns it, CCA needs another integration point.
- **Suggested options:** (A) CCA manages statistical key figures as a new aggregate, (B) Separate statistical key figure service, (C) Statistical data managed in ref-data-svc
- **Owner:** TBD

## Q-CCA-002: Multiple Alternative Hierarchies
- **Question:** Should cost center hierarchies support multiple alternative hierarchies (e.g., functional vs. regional vs. legal entity)?
- **Why it matters:** SAP supports alternative hierarchies (standard hierarchy + group hierarchies). Multiple hierarchies increase data model complexity but enable flexible reporting and allocation views.
- **Suggested options:** (A) Single hierarchy only (MVP), (B) Multiple named hierarchies from the start, (C) Single hierarchy initially with extension point for additional hierarchies
- **Owner:** TBD

## Q-CCA-003: Retroactive FI Corrections
- **Question:** How to handle retroactive FI corrections to closed CO periods?
- **Why it matters:** FI may post corrections to prior periods. If CO period is closed, these corrections cannot be posted. This affects reconciliation accuracy.
- **Suggested options:** (A) Reopen period for corrections, (B) Post corrections to current open period with reference to original period, (C) Require manual adjustment by controller
- **Owner:** TBD

## Q-CCA-004: Feature Catalog Dependency
- **Question:** Which product features depend on this service's endpoints? The CO feature catalog (SS6 in suite spec) has not yet been authored.
- **Why it matters:** The feature dependency register in 11.2 is provisional. Accurate feature-to-endpoint mapping is needed for BFF layer generation and product configuration.
- **Suggested options:** (A) Author CO feature catalog first, then update 11.2, (B) Proceed with provisional mapping and refine later
- **Owner:** TBD

## Q-CCA-005: Plan Data Bulk Import
- **Question:** Should PlanData support bulk import via a batch endpoint (e.g., POST /plan-data/batch) or only single-record creation?
- **Why it matters:** Plan data loading typically involves thousands of records per fiscal year. Single-record creation would be extremely slow. A batch endpoint would improve throughput but adds API complexity.
- **Suggested options:** (A) Single-record only (MVP), (B) Batch endpoint with array of plan entries, (C) Async batch import via file upload
- **Owner:** TBD

## Q-CCA-006: Commitment Amount Source
- **Question:** Should the commitment amount in CostCenterBalance be updated from purchase order events (ops/sd suite)?
- **Why it matters:** Commitment tracking requires consuming purchase order creation/change events from external suites. This adds integration complexity and a new consumed event.
- **Suggested options:** (A) No commitment tracking in Phase 1, (B) Consume purchase order events from ops suite, (C) Manual commitment entry by controller
- **Owner:** TBD

## Q-CCA-007: Manual Cost Posting via REST
- **Question:** Should CostPosting support manual creation via REST endpoint in addition to event-driven creation from FI?
- **Why it matters:** Controllers may need to create manual adjustments or reclassification postings directly in CO without going through FI. This requires a new REST endpoint and RBAC controls.
- **Suggested options:** (A) Event-driven only (all postings originate from FI or CO.OM), (B) REST endpoint for manual postings with controller authorization, (C) REST endpoint limited to reversal postings only
- **Owner:** TBD
