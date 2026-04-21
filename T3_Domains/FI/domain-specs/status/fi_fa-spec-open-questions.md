# Open Questions - fi.fa

## Q-FA-001: Feature Dependency Register
- **Question:** Which product features depend on fi.fa's endpoints and events?
- **Why it matters:** Section 11 (Feature Dependencies) cannot be fully populated until feature specs exist. This affects BFF design and API change impact analysis.
- **Suggested options:** A) Wait for FI feature specs to be created, then populate section 11. B) Create placeholder feature specs for core fi.fa functionality.
- **Owner:** Product Team

## Q-FA-002: Automatic Asset Creation from AP Bill Events
- **Question:** Should fi.fa automatically create assets when it receives fi.ap.bill.posted events with a capital expenditure flag?
- **Why it matters:** Determines the fi.ap.bill.posted event handler behavior (UC-001 alternative flow). Automatic creation reduces manual work but may require additional validation logic.
- **Suggested options:** A) Auto-create if capex flag set and all required fields present. B) Always require manual asset creation with optional bill linking. C) Configurable per tenant.
- **Owner:** Architecture Team

## Q-FA-003: Port Assignment for fi-fa-svc
- **Question:** What is the exact port assignment for fi-fa-svc?
- **Why it matters:** Port 8440 is currently a placeholder. Needs confirmation from the platform team's port allocation table to avoid conflicts.
- **Suggested options:** Confirm with platform team port allocation table.
- **Owner:** Platform Team

## Q-FA-004: Partial Depreciation Run Posting
- **Question:** Should depreciation runs support partial posting (a subset of assets)?
- **Why it matters:** Current design assumes all-or-nothing per run. Partial posting would complicate BR-RUN-001 (period uniqueness) and the depreciation line reconciliation.
- **Suggested options:** A) All assets in one run (current design, simpler). B) Allow asset selection per run (more flexible but more complex uniqueness checks).
- **Owner:** Domain Team

## Q-FA-005: Intercompany Asset Transfers
- **Question:** How should inter-entity (intercompany) asset transfers be handled for GL posting?
- **Why it matters:** The spec supports fromEntity/toEntity fields on AssetTransaction but does not detail the intercompany posting logic. This affects whether fi.fa handles IC postings directly or delegates to fi.ic.
- **Suggested options:** A) fi.fa handles IC posting via fi.slc (simpler, self-contained). B) fi.fa delegates IC posting to fi.ic domain (better separation of concerns, avoids IC complexity in fi.fa).
- **Owner:** Architecture Team

## Q-FA-006: Custom Fields on AssetTransaction Table
- **Question:** Should the custom_fields JSONB column be added to the fa_asset_transactions table?
- **Why it matters:** Section 12.2 marks AssetTransaction as extensible, but the current section 8.3 table definition does not include the custom_fields column. Need to confirm alignment.
- **Suggested options:** A) Add custom_fields JSONB now for consistency with section 12. B) Defer until a product team requests it.
- **Owner:** Domain Team

## Q-FA-007: Depreciation Run Scheduling Mechanism
- **Question:** What mechanism triggers the monthly depreciation run?
- **Why it matters:** UC-002 mentions the run is "scheduled monthly" but does not specify the scheduling mechanism. This affects infrastructure requirements and operational procedures.
- **Suggested options:** A) External cron job triggers POST /depreciation-runs. B) Built-in Spring scheduler in fi.fa service. C) External workflow engine (e.g., Temporal) triggers the run.
- **Owner:** Platform Team
