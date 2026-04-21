# Open Questions - co.pc

## Q-PC-001: Multi-Level BOM Cost Rollup
- **Question:** Should co.pc support multi-level BOM cost rollup (semi-finished -> finished products)?
- **Why it matters:** Multi-level BOM explosion is required for accurate costing of finished goods that contain sub-assemblies. Without it, semi-finished product costs must be pre-calculated and treated as material inputs. This affects the data model (recursive cost aggregation) and the costing run algorithm.
- **Suggested options:**
  - A) Flat rollup only — semi-finished costs are pre-calculated inputs
  - B) Recursive rollup with caching — co.pc explodes multi-level BOMs and caches intermediate results
  - C) Deferred to PD service — PD provides a flattened BOM; co.pc only processes flat structures
- **Owner:** TBD

## Q-PC-002: Joint/By-Product Costing
- **Question:** How should co.pc handle joint product and by-product costing?
- **Why it matters:** Manufacturing processes that produce multiple outputs (co-products, by-products) require cost allocation methods. This affects the variance analysis algorithm since standard costs must be split across outputs.
- **Suggested options:**
  - A) Proportional by weight or volume
  - B) Net realizable value (NRV) method — allocate based on sales value
  - C) Out of scope for Phase 1 — defer to future release
- **Owner:** TBD

## Q-PC-003: BOM Integration Pattern
- **Question:** What integration pattern should co.pc use to retrieve BOMs from PD?
- **Why it matters:** Costing runs process thousands of products. Synchronous API calls per product may be too slow. Caching introduces staleness risk. A bulk fetch endpoint may be needed.
- **Suggested options:**
  - A) Synchronous REST API per product — simple but potentially slow for large runs
  - B) Cached BOM snapshot — co.pc maintains a local copy updated via events
  - C) Bulk BOM fetch endpoint — PD provides batch API for costing run scenarios
- **Owner:** TBD

## Q-PC-004: Feature Dependency Register
- **Question:** Which product features depend on this service's endpoints?
- **Why it matters:** §11 Feature Dependencies requires concrete feature IDs to enable impact assessment for API changes. Without this, the BFF aggregation hints and impact assessment tables are incomplete.
- **Suggested options:**
  - A) Wait for CO suite feature specs to be authored
  - B) Define preliminary feature IDs based on use cases (F-CO-PC-001 through F-CO-PC-006)
- **Owner:** TBD

## Q-PC-005: CostVariance Custom Fields
- **Question:** Should the CostVariance aggregate support custom fields (JSONB column)?
- **Why it matters:** Determines the extension model for variance data. Products may need to annotate variances with custom categorization or action tracking.
- **Suggested options:**
  - A) Yes — add custom_fields JSONB column to cost_variance table
  - B) No — use an external annotation/notes service instead; keep variance records lean
- **Owner:** TBD

## Q-PC-006: WIP Valuation Scope
- **Question:** Should co.pc calculate WIP (Work-in-Progress) values, or should this be delegated to fi.acc or a separate domain?
- **Why it matters:** WIP is listed as in-scope in §0.3, but the calculation method and ownership are undefined. WIP valuation sits at the boundary between production costing and financial accounting.
- **Suggested options:**
  - A) co.pc calculates WIP values based on production order progress and standard costs
  - B) co.pc provides cost data; fi.acc calculates WIP for accounting purposes
  - C) Create a separate co.wip domain to handle WIP-specific logic
- **Owner:** TBD

## Q-PC-007: Costing Run Auto-Activation
- **Question:** Should costing runs support automatic activation of calculated product costs?
- **Why it matters:** Currently, costing runs create ProductCost records in draft status, requiring manual activation. Controllers may want an option to auto-activate costs from a run, reducing manual effort during period-end processing.
- **Suggested options:**
  - A) Always require manual activation (current design)
  - B) Configurable per run — add an `autoActivate` flag to the costing run parameters
  - C) Auto-activate with mandatory approval workflow integration
- **Owner:** TBD

## Q-PC-008: Material Ledger / Actual Costing Run
- **Question:** Should co.pc support periodic actual cost revaluation (equivalent to SAP Material Ledger)?
- **Why it matters:** Material Ledger functionality adjusts standard cost to actual cost at period end, producing a "periodic unit price" used for COGS and inventory valuation. This is a significant functional scope extension beyond basic variance analysis.
- **Suggested options:**
  - A) Out of scope — actual costing revaluation is not planned
  - B) Phase 2 — add material ledger functionality after core product costing is stable
  - C) Separate domain — create co.ml for material ledger concerns
- **Owner:** TBD
