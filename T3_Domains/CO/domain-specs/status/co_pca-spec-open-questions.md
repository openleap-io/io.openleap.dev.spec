# Open Questions - co.pca

## Q-PCA-001: Statistical Profit Centers
- **Question:** Should the system support statistical profit centers (cost allocation only, no revenue)?
- **Why it matters:** Affects the data model. Statistical PCs are common in SAP CO-PCA for overhead distribution but do not carry revenue. This changes P&L calculation logic and the SegmentType enum or requires a separate boolean flag.
- **Suggested options:** A) Add `statistical: boolean` to ProfitCenter; B) Add `statistical` value to SegmentType enum; C) Model as separate aggregate
- **Owner:** TBD

## Q-PCA-002: IFRS 8 Segment Mapping
- **Question:** Should the IFRS 8 segment mapping from profit centers to external reporting segments be automated or manual?
- **Why it matters:** Determines the integration contract with fi-acc-svc. Automated mapping requires a configuration table; manual mapping pushes responsibility to finance users in fi.acc.
- **Suggested options:** A) Automated mapping via configuration table in co-pca-svc; B) Manual mapping managed in fi-acc-svc; C) Hybrid with suggested mapping and manual override
- **Owner:** TBD

## Q-PCA-003: Feature Dependency Register
- **Question:** Which product features depend on this service's endpoints?
- **Why it matters:** The Feature Dependency Register (SS11) cannot be completed without feature spec IDs. This affects impact assessment and BFF aggregation hints.
- **Suggested options:** Wait for CO feature specs to be authored, then update SS11.
- **Owner:** TBD

## Q-PCA-004: Balance Record Immutability
- **Question:** Should balance records be immutable once status is `final`, or should an `adjusted` status be allowed for post-close corrections?
- **Why it matters:** Financial integrity and audit trail. Post-close adjustments are common in practice (audit findings, late postings, error corrections). Immutability is simpler but may not meet real-world requirements.
- **Suggested options:** A) Immutable final -- corrections require a new calculation period; B) Allow `adjusted` status with full audit trail (current design); C) Create correction records rather than modifying originals
- **Owner:** TBD

## Q-PCA-005: Port Assignment
- **Question:** What port should co-pca-svc use?
- **Why it matters:** Deployment configuration and service registry.
- **Suggested options:** Coordinate with platform team for port allocation within the CO suite port range.
- **Owner:** TBD

## Q-PCA-006: Repository URI
- **Question:** What is the repository URI for co-pca-svc?
- **Why it matters:** Metadata completeness and traceability from spec to code.
- **Suggested options:** Depends on GitHub organization setup (e.g., `github.com/openleap/co-pca-svc`).
- **Owner:** TBD

## Q-PCA-007: Incremental P&L Calculation
- **Question:** Should the P&L calculation support incremental updates (delta processing) or always recalculate from scratch?
- **Why it matters:** Performance at scale. Full recalculation is simpler and guarantees consistency, but may be slow for tenants with hundreds of profit centers and high transaction volumes. Incremental updates are faster but more complex and risk drift.
- **Suggested options:** A) Always full recalculation (current design, simpler); B) Incremental delta processing (faster, requires change tracking); C) Full recalculation with caching of intermediate results
- **Owner:** TBD

## Q-PCA-008: Consolidated Balance Currency Conversion
- **Question:** How should the consolidated balance summary handle profit centers with different local currencies?
- **Why it matters:** The balance summary endpoint aggregates across profit centers. If PCs have different currencies, a conversion is needed for meaningful totals.
- **Suggested options:** A) Use controlling area's default currency for consolidation; B) Accept target currency as query parameter and convert at period-end rates; C) Return per-currency subtotals without conversion
- **Owner:** TBD
