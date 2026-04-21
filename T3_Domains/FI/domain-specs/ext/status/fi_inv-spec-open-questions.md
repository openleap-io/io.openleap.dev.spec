# Open Questions - fi.inv

## Q-INV-001: Feature Dependency Register Completeness
- **Question:** Which product features depend on fi.inv service endpoints? Feature specs for the FI-INV area have not yet been created.
- **Why it matters:** The feature dependency register (SS11) is populated based on endpoint mapping, but without feature specs, the feature IDs and exact scope are estimates. Incorrect mapping could lead to missing BFF routes or incorrect feature gating.
- **Suggested options:**
  - Option A: Create FI-INV feature specs (F-FI-INV-001 through F-FI-INV-006) to formalize the mapping
  - Option B: Defer until a product spec selects fi.inv features
- **Owner:** Product Team

## Q-INV-002: Multi-Currency Parallel Valuation
- **Question:** Should fi.inv support multi-currency parallel valuation (e.g., local currency + group reporting currency simultaneously)?
- **Why it matters:** Multinational deployments need inventory valued in both local entity currency and group reporting currency. This significantly impacts the aggregate model (dual-value fields), posting logic (two journal entries per movement), and query API (currency parameter).
- **Suggested options:**
  - Option A: Single currency (MVP) — inventory valued in entity's local currency only
  - Option B: Dual currency valuation — parallel valuation with configurable group currency
  - Option C: Deferred — handle via fi.cnsl (consolidation) currency translation at period end
- **Owner:** Architecture Team

## Q-INV-003: FIFO/LIFO Cost Layer Storage
- **Question:** How should FIFO/LIFO cost layers be physically stored and managed?
- **Why it matters:** Cost layers are fundamental to FIFO/LIFO valuation methods. The storage strategy impacts query performance (layer consumption lookups), data model complexity, and whether cost layers deserve their own microservice.
- **Suggested options:**
  - Option A: Separate `inv_cost_layers` table with layer_id, item_id, location, remaining_qty, cost_per_unit, created_at
  - Option B: JSONB array on inv_positions (simpler but less queryable)
  - Option C: Dedicated cost layer microservice (if complexity warrants separation)
- **Owner:** Architecture Team

## Q-INV-004: Variance Posting as Separate Aggregate
- **Question:** Should variance postings (PPV, MUV) be modeled as a separate aggregate, a movement subtype, or handled entirely by fi.ap?
- **Why it matters:** Currently PPV is described in the flow but not formally modeled as an aggregate. Separate aggregate gives cleaner domain model but adds complexity. Having fi.ap handle PPV simplifies fi.inv but blurs domain boundaries.
- **Suggested options:**
  - Option A: Separate `Variance` aggregate in fi.inv with its own lifecycle
  - Option B: Variance as a movement subtype (type=PPV) within InventoryMovement
  - Option C: fi.ap handles PPV directly (fi.inv only tracks standard cost movements)
- **Owner:** Domain Team

## Q-INV-005: Service Port Number
- **Question:** What is the confirmed port number for fi-inv-svc?
- **Why it matters:** Port allocation must be coordinated across all FI suite services to avoid conflicts.
- **Suggested options:**
  - Option A: 8460 (currently assigned in spec meta block)
  - Option B: Allocate from FI suite port range (confirm with DevOps)
- **Owner:** DevOps

## Q-INV-006: Batch Revaluation API Design
- **Question:** Should batch revaluation (revaluing all items in a category due to standard cost update) be a synchronous or asynchronous operation?
- **Why it matters:** Annual standard cost rolls may affect thousands of items. Synchronous processing would timeout; async requires job management, status polling, and partial failure handling.
- **Suggested options:**
  - Option A: Synchronous per-item revaluation (client iterates)
  - Option B: Async batch job returning 202 Accepted with job status endpoint
  - Option C: Bulk endpoint that processes items serially within a single request (with streaming response)
- **Owner:** Architecture Team

## Q-INV-007: Intercompany Transfer Handling
- **Question:** How should fi.inv handle intercompany (inter-entity) inventory transfers?
- **Why it matters:** Intercompany transfers involve two legal entities with potentially different cost policies and currencies. They require intercompany accounting entries (fi.ic integration) and may have transfer pricing implications.
- **Suggested options:**
  - Option A: Two separate movements — GI in source entity + GR in destination entity, linked by a transfer reference
  - Option B: Single XFER movement spanning entities, with fi.inv handling the intercompany posting internally
  - Option C: Delegate to fi.ic (Intercompany service) for orchestration; fi.inv only handles intra-entity movements
- **Owner:** Domain Team

## Q-INV-008: Extension Event Synchronicity
- **Question:** Should extension events (SS12.3 ext.pre-movement-post, etc.) be synchronous (blocking the posting until handler responds) or asynchronous (fire-and-forget)?
- **Why it matters:** Synchronous hooks allow products to gate/reject operations but add latency and failure risk. Asynchronous hooks are simpler but cannot block operations.
- **Suggested options:**
  - Option A: Fire-and-forget for all extension events (proposed in spec)
  - Option B: Synchronous with timeout for pre-* hooks, fire-and-forget for post-* hooks
  - Option C: Configurable per hook registration (product chooses sync/async)
- **Owner:** Architecture Team
