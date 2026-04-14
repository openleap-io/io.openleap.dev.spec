# Open Questions - sd.ret

## Q-RET-001: Exchange Processing

- **Question:** Should exchange processing (return + new replacement order in one atomic flow) be supported?
- **Why it matters:** Customers frequently want to exchange a wrong or damaged item for the correct one. Without this, two separate transactions are required (return + new order), creating a gap in customer experience and revenue tracking.
- **Suggested options:**
  - A) `ReturnType.EXCHANGE` triggers an automatic new sales order in sd.ord, linked to the return
  - B) Separate `ExchangeOrder` aggregate within sd.ret that orchestrates the two-leg process
  - C) Leave as Phase 2; customers manually create new order after return approval
- **Owner:** TBD

---

## Q-RET-002: Warranty Claims — Domain Boundary

- **Question:** Should warranty claims be modeled as `ReturnType.WARRANTY` within sd.ret, or as a separate domain (e.g., `sd.war`)?
- **Why it matters:** Warranty claims have distinct rules (coverage periods, supplier chargebacks, warranty registration validation) that may exceed sd.ret's scope. Getting this boundary wrong creates debt that is expensive to unwind.
- **Suggested options:**
  - A) `ReturnType.WARRANTY` within sd.ret — simpler, leverages existing inspection/disposition flow
  - B) Separate `sd.war` domain — cleaner boundary, supports complex warranty rules and supplier integration
- **Owner:** TBD

---

## Q-RET-003: Restocking Fee Calculation

- **Question:** Should the platform support restocking fee deduction from refund amounts, and if so, how?
- **Why it matters:** Many businesses charge a restocking fee (e.g., 15% of refund amount) for certain return types (CHANGED_MIND). Without this, the platform cannot correctly compute net refund amounts for these cases.
- **Suggested options:**
  - A) Tenant-level configuration (percentage fee) applied automatically on credit calculation
  - B) Extension rule EXT-RULE-RET-003 — product teams implement their own fee logic
  - C) Defer to Phase 2; current `refundAmount = acceptedQty × unitPrice` remains the standard
- **Owner:** TBD

---

## Q-RET-004: Cross-Border Return Logistics

- **Question:** How should cross-border returns (different country of origin/destination) be handled, including customs documentation and tax implications?
- **Why it matters:** International returns require customs declarations, potentially import duties on re-entry, and may have different policy windows by jurisdiction.
- **Suggested options:**
  - A) Phase 3 dedicated extension or separate compliance domain
  - B) Custom fields on ReturnOrder (e.g., `customsDeclarationNumber`) as interim measure
- **Owner:** TBD

---

## Q-RET-005: Feature Spec Authoring

- **Question:** Which product features formally depend on sd.ret's endpoints, and when will their feature specs be authored?
- **Why it matters:** Feature specs (F-SD-RET-001 through F-SD-RET-004) are required to complete §11 Feature Dependency Register accurately and enable BFF contract generation.
- **Suggested options:**
  - A) Author feature specs as part of SD suite Phase 1 delivery
  - B) Use placeholder feature IDs in §11 until feature specs are created
- **Owner:** TBD

---

## Q-RET-006: RMA Number Format and Generation

- **Question:** What is the required format and generation strategy for `rmaNumber`?
- **Why it matters:** The RMA number is communicated to the customer and printed on return shipping labels. Format must be human-readable, unique, and compatible with carrier systems.
- **Suggested options:**
  - A) Tenant-configurable prefix + year + sequential number (e.g., `RMA-2026-00042`)
  - B) UUID-based (unique but not human-friendly)
  - C) Fixed format with check digit for validation on receipt
- **Owner:** TBD

---

## Q-RET-007: im.goodsreceipt.posted Integration

- **Question:** Is the `im.goodsreceipt.posted` event integration required for marking returns as RECEIVED, or is `sd.dlv.delivery.received` sufficient?
- **Why it matters:** If both events can trigger the RECEIVED transition, there is a risk of duplicate processing. If only `sd.dlv.delivery.received` is used, a delayed sd.dlv event could leave returns stuck in RETURN_SHIPPED.
- **Suggested options:**
  - A) `sd.dlv.delivery.received` is the authoritative trigger; `im.goodsreceipt.posted` is a secondary fallback (idempotent)
  - B) Only `sd.dlv.delivery.received` — remove im.goodsreceipt.posted consumer
  - C) Both required (dual confirmation before RECEIVED transition)
- **Owner:** TBD
