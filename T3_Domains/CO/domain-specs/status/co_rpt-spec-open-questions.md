# Open Questions - co.rpt

## Q-RPT-001: Ad-hoc User-Defined Report Columns
- **Question:** Should RPT support ad-hoc user-defined report columns via the `custom` report type?
- **Why it matters:** Determines UX complexity for the report builder feature. A full column designer requires schema introspection of read model views. Predefined templates are simpler but less flexible.
- **Suggested options:** A) Full column designer with drag-and-drop, B) Predefined templates only (select from catalog), C) Defer to Phase 2
- **Owner:** TBD

## Q-RPT-002: Boundary Between co.rpt and T4 Analytics
- **Question:** Where do dashboards live — co.rpt or T4 Analytics?
- **Why it matters:** Determines API scope for co.rpt and prevents feature duplication. Affects whether co.rpt needs real-time WebSocket push for dashboard widgets.
- **Suggested options:** A) co.rpt handles structured reports only; T4 handles all dashboards and visualizations, B) co.rpt provides simple KPI dashboard API for embedded management views
- **Owner:** TBD

## Q-RPT-003: Report Data Retention Period
- **Question:** How long should ReportInstances (generated snapshots) be retained?
- **Why it matters:** Storage cost planning; compliance requirements (SOX may require 7 years of supporting documentation). Also affects database sizing and archival strategy.
- **Suggested options:** A) 2 years (then archive to T4), B) 5 years (align with read model retention), C) Configurable per tenant
- **Owner:** TBD

## Q-RPT-004: Feature Dependency Concrete IDs
- **Question:** Which product features depend on this service's endpoints?
- **Why it matters:** Feature dependency register (SS11) needs concrete feature IDs to assess blast radius of API changes. Currently using provisional IDs (F-CO-RPT-001 through F-CO-RPT-005).
- **Suggested options:** Wait for feature spec authoring in CO suite to produce definitive feature IDs
- **Owner:** TBD

## Q-RPT-005: Cross-Controlling-Area Queries
- **Question:** Should read model views support queries spanning multiple controlling areas?
- **Why it matters:** Multi-entity corporate groups may need consolidated views across controlling areas. This affects query API design and database indexing strategy.
- **Suggested options:** A) Single controlling area per query (simple, current design), B) Multi-controlling-area with server-side aggregation (complex, requires currency conversion)
- **Owner:** TBD

## Q-RPT-006: Maximum Report Size for Sync Generation
- **Question:** What is the maximum number of rows before the system forces asynchronous (background) report generation?
- **Why it matters:** Large JSON responses can cause HTTP timeouts and excessive memory usage. The threshold determines when to switch from sync (200 OK) to async (202 Accepted) mode.
- **Suggested options:** A) 1,000 rows, B) 5,000 rows, C) Configurable per definition
- **Owner:** TBD

## Q-RPT-007: Variance Decomposition in ProductCostReportView
- **Question:** Should RPT store decomposed variance components (price variance, quantity variance, efficiency variance) in the ProductCostReportView?
- **Why it matters:** Variance decomposition is critical for SAP-style cost analysis (analogous to transaction KSB1 drill-down). Depends on whether co.pc publishes decomposed variances in its events.
- **Suggested options:** A) Store total variance only (co.pc provides decomposition via API on drill-down), B) Store all decomposition components in the read model (requires co.pc event enhancement)
- **Owner:** TBD
