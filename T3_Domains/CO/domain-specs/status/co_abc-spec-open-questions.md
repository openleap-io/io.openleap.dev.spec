# Open Questions - co.abc

## Q-ABC-001: Automated Activity Volume Feeds
- **Question:** Should activity volume feeds be automated (event-driven from OPS) or manual entry via API?
- **Why it matters:** Determines integration complexity with operational services (MES, SD, PUR). Automated feeds require consumed event handlers and mapping configuration.
- **Suggested options:** A) Automated via ops.activity.completed events, B) Manual entry via API, C) Hybrid (both supported)
- **Owner:** TBD

## Q-ABC-002: Time-Driven ABC (TDABC) Support
- **Question:** Should the system support TDABC as an alternative calculation method alongside traditional ABC?
- **Why it matters:** TDABC uses capacity cost rates and time equations instead of activity drivers. Simpler setup but less granular. Requires different data model (time equations instead of activity drivers).
- **Suggested options:** A) Support both traditional ABC and TDABC in v1, B) Traditional ABC only in v1, TDABC in v2
- **Owner:** TBD

## Q-ABC-003: Shared Activities Across Processes
- **Question:** How to handle activities that logically span multiple business processes (e.g., "IT Support")?
- **Why it matters:** Current data model enforces one activity per process. Shared activities require either many-to-many relationships or cloning, affecting cost calculation and reporting.
- **Suggested options:** A) Allow many-to-many relationship, B) Clone activity per process (independent cost tracking), C) Create a "shared" process for cross-cutting activities
- **Owner:** TBD

## Q-ABC-004: Automatic ABC-to-Product Costing Feed
- **Question:** Should ABC results automatically trigger co.pc costing runs, or remain query-based?
- **Why it matters:** Determines integration pattern between ABC and product costing. Automatic feed means tighter coupling; query-based means co.pc decides when to refresh.
- **Suggested options:** A) co.pc queries ABC on demand (current design), B) ABC pushes rates to co.pc via event, C) Both options available
- **Owner:** TBD

## Q-ABC-005: What-If Simulation Support
- **Question:** Should the system support what-if simulation scenarios beyond the current dryRun flag?
- **Why it matters:** Controllers want to test different driver configurations (e.g., "what if we increase IT cost center share to 40%?") before committing. dryRun only previews the current config.
- **Suggested options:** A) dryRun flag only (current), B) Full simulation workspace with scenario comparison, C) Defer to Phase 2
- **Owner:** TBD

## Q-ABC-006: Feature Spec Dependencies
- **Question:** Which product features depend on this service's endpoints?
- **Why it matters:** Feature specs for the CO suite have not been authored yet. Section 11 is populated with anticipated features but lacks confirmed Feature IDs.
- **Suggested options:** Author CO feature specs to formalize the dependency register
- **Owner:** TBD

## Q-ABC-007: Resource Driver Allocation Mode
- **Question:** Should resource drivers support both percentage-based (sharePct) and absolute quantity (driverQuantity) allocation, and if so, which takes precedence?
- **Why it matters:** Current model has both fields. Need to clarify whether they are alternatives (one or the other) or complementary (sharePct derived from driverQuantity).
- **Suggested options:** A) sharePct is primary, driverQuantity is informational, B) Either/or per driver (enum: pct_based vs. quantity_based), C) driverQuantity primary, sharePct computed
- **Owner:** TBD

## Q-ABC-008: Activity Table Controlling Area Column
- **Question:** Should the activity table include a `controlling_area` column for the unique key, or inherit it from the parent process?
- **Why it matters:** Current unique key is (tenant_id, controlling_area, code) but the activity table does not have a controlling_area column -- it would need a join to the process table. Denormalizing simplifies queries and constraints but adds redundancy.
- **Suggested options:** A) Add controlling_area to activity table (denormalize), B) Unique key enforced via application logic with process join
- **Owner:** TBD
