# Open Questions - fi.ic

## Q-IC-001: Product Feature Dependencies
- **Question:** Which product features depend on fi.ic service endpoints and events?
- **Why it matters:** Needed for complete SS11 feature dependency register; affects BFF aggregation planning and API change impact assessment.
- **Suggested options:** A) Wait for product feature specs to be authored; B) Derive from product roadmap interviews with product team.
- **Owner:** Product Team

## Q-IC-002: Multilateral Netting as First-Class Aggregate
- **Question:** Should fi.ic support multilateral netting as a dedicated NettingRun aggregate rather than just a PaymentMethod enum value?
- **Why it matters:** Large corporate groups with many entities need to net IC balances across multiple entity pairs in a single run, calculating optimal payment flows. This requires a complex process with its own lifecycle (initiate, propose, approve, execute).
- **Suggested options:** A) Keep netting as a PaymentMethod enum value (simple, settlement-level); B) Add NettingRun aggregate with dedicated use cases and API endpoints; C) Defer to a separate netting domain.
- **Owner:** Architecture Team

## Q-IC-003: FX Rate Source for Buyer-Side Calculation
- **Question:** What is the exact service and endpoint for exchange rates used when calculating buyer-side amounts during IC transaction mirroring?
- **Why it matters:** Implementation needs a reliable FX rate source; affects multi-currency posting accuracy and reconciliation.
- **Suggested options:** A) ref-data-svc GET /api/ref/exchange-rates; B) Dedicated fi.fx service; C) External FX API (e.g., ECB rates); D) User-supplied rate per transaction.
- **Owner:** Architecture Team

## Q-IC-004: Real-Time vs. Batch IC Balance Maintenance
- **Question:** Should IC balances be updated in real-time (incrementally after each transaction posting) or computed in batch (at period-end during reconciliation)?
- **Why it matters:** Real-time provides immediate visibility but adds write overhead to every transaction; batch is simpler but provides stale data between runs.
- **Suggested options:** A) Real-time incremental updates (current assumption in spec); B) Batch computation at period-end only; C) Hybrid -- maintain a running total with periodic recalculation for accuracy.
- **Owner:** Architecture Team

## Q-IC-005: Port Assignment for fi-ic-svc
- **Question:** What port should fi-ic-svc use for local development and service registry?
- **Why it matters:** Needed for service registry, local Docker Compose, and developer setup documentation.
- **Suggested options:** A) 8480 (current placeholder); B) Assign from FI suite port range per DevOps convention.
- **Owner:** DevOps Team

## Q-IC-006: Partial Settlements
- **Question:** Should fi.ic support partial settlements where a large IC transaction is settled in multiple installments?
- **Why it matters:** Real-world IC transactions, especially large ones, may be settled in parts. Current model allows only one settlement per transaction.
- **Suggested options:** A) Single settlement only (current model -- simple); B) Allow multiple partial settlements with a running settled amount and remaining balance; C) Support partial as a v2 feature.
- **Owner:** Product Team

## Q-IC-007: Default Reconciliation Variance Threshold
- **Question:** What should the default reconciliation variance threshold be for out-of-the-box behavior?
- **Why it matters:** Affects whether small FX rounding differences are auto-reconciled or flagged for manual review. Too low creates noise; too high misses real issues.
- **Suggested options:** A) $10 / EUR 10 (configurable per tenant); B) 0.1% of IC balance (percentage-based); C) Tiered: $10 for balances < $100K, $100 for balances >= $100K.
- **Owner:** Product Team
