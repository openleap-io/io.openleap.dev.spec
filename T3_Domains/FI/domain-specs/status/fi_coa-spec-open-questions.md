# Open Questions - fi.coa

## OQ-COA-001: GL Account Ref Validation Strategy
- **Question:** Should `fi.coa` validate `glAccountRef` synchronously against `fi.gl` at write time, or asynchronously via events?
- **Why it matters:** Determines latency and coupling profile. Sync validation adds a REST dependency on fi-gl-svc; async validation accepts eventual consistency but allows mapping of accounts that do not yet exist.
- **Suggested options:**
  1. Synchronous REST lookup (tight coupling, immediate validation)
  2. Async validation on `fi.gl.account.created` events (eventual consistency, loose coupling — current approach)
  3. Hybrid: async by default with optional sync lookup for critical publishing workflows
- **Owner:** Architecture Team

## OQ-COA-002: Port Assignment
- **Question:** What port should be assigned to fi-coa-svc?
- **Why it matters:** Deployment configuration and service registry.
- **Suggested options:** Follow platform port registry convention.
- **Owner:** Architecture Team

## OQ-COA-003: AccountChangeRequest Resolution Pattern
- **Question:** Should AccountChangeRequest resolution (ACCEPTED/REJECTED) be driven by `fi.gl` publishing an event or by a REST callback?
- **Why it matters:** Determines integration pattern between fi.coa and fi.gl for the change request workflow.
- **Suggested options:**
  1. Event-driven: `fi.gl.account.created` implies ACCEPTED (current approach)
  2. REST callback from fi.gl to fi.coa with explicit accept/reject
  3. Dedicated `fi.gl.accountChangeRequest.resolved` event
- **Owner:** FI Domain Team

## OQ-COA-004: NodeMapping Weight Validation Strictness
- **Question:** Should NodeMapping weight validation (sum to 1.0) be MUST or SHOULD?
- **Why it matters:** Hard validation (MUST) prevents incomplete reporting allocations but increases friction for Finance Admins setting up complex structures. Soft validation (SHOULD, current) issues a warning but allows flexibility.
- **Suggested options:**
  1. Hard validation (MUST) — reject if weights do not sum to 1.0
  2. Warning only (SHOULD) — current approach, emit COA-WARN-005
  3. Configurable per tenant (hard or soft via extension rule)
- **Owner:** Finance Product Owner

## OQ-COA-005: Feature Dependency Mapping
- **Question:** Which feature IDs (F-FI-*) depend on fi-coa-svc endpoints?
- **Why it matters:** Feature dependency register (§11) is incomplete until Phase 3 feature specs are authored. Needed for blast radius assessment of API changes.
- **Suggested options:** Populate during FI feature spec authoring in Phase 3.
- **Owner:** FI Feature Team

## OQ-COA-006: ChartNode Custom Fields Extensibility
- **Question:** Should ChartNode be extensible with custom_fields (JSONB column)?
- **Why it matters:** Some deployments may need extra metadata on individual nodes (e.g., regulatory classification tag, department ownership code). Currently marked as non-extensible.
- **Suggested options:**
  1. Keep non-extensible (current) — simplicity, no product-level need identified
  2. Add custom_fields to coa_chart_nodes — flexibility for future needs
- **Owner:** Architecture Team

## OQ-COA-007: Bulk Import of Chart Nodes
- **Question:** Should fi.coa support bulk import of chart nodes from CSV/Excel?
- **Why it matters:** Initial setup of large chart structures (e.g., SKR04 with 200+ accounts) is labor-intensive via individual REST calls. Migration from legacy systems also benefits from bulk import.
- **Suggested options:**
  1. REST-only (current) — simple, standard
  2. Add bulk import endpoint (`POST /chartStructures/{id}/nodes/import`)
  3. Offline tool that generates REST calls from CSV/Excel
- **Owner:** Finance Product Owner
