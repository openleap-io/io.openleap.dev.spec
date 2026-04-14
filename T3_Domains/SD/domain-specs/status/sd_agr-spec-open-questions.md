# Open Questions - sd.agr (Sales Agreements)

**Last Updated:** 2026-04-03

---

## Q-AGR-001: Agreement Pricing Conditions Integration

- **Question:** Should agreed prices on AgreementLines feed into COM.PRC as pricing conditions, or remain as local price overrides on the line itself?
- **Why it matters:** If agreed prices are stored locally on AgreementLine, there is no automatic integration with the pricing resolution engine (COM.PRC). This means sd.ord must explicitly use the AgreementLine price rather than running through price resolution. If fed to COM.PRC, pricing becomes centralized but adds a dependency.
- **Suggested options:**
  - A) Local agreed prices only — simpler; agreed price stored on AgreementLine.agreedPrice; sd.ord reads directly
  - B) Feed to COM.PRC as condition records — unified pricing; more complex integration; enables price simulation to include agreement discounts
- **Owner:** TBD

---

## Q-AGR-002: Group Rebates (Buying Groups)

- **Question:** Should rebate agreements support buying groups — where multiple customer party IDs contribute to a single rebate target?
- **Why it matters:** Enterprise purchasing organizations often negotiate rebates on behalf of subsidiaries or franchisee networks. Without group support, each subsidiary requires a separate rebate agreement with independent targets.
- **Suggested options:**
  - A) Single-customer only (current design)
  - B) Add a `BuyingGroup` entity that references multiple customer parties; RebateAgreement references BuyingGroup instead of a single customerPartyId
- **Owner:** TBD

---

## Q-AGR-003: Retroactive Rebate Adjustments

- **Question:** Should the system support retroactive adjustments to rebate accruals (e.g., if a sales order is cancelled after settlement)?
- **Why it matters:** Sales order cancellations after a rebate period ends can create over-settlement. Without retroactive adjustment support, over-paid rebates require manual credit/debit memo corrections.
- **Suggested options:**
  - A) No retroactive adjustments — cancelled orders after settlement are handled manually
  - B) Issue adjustment credit/debit memos via sd.bil in the next settlement period
  - C) Allow reopening a settled period (complex; audit trail implications)
- **Owner:** TBD

---

## Q-AGR-004: Agreement Templates

- **Question:** Should the system support agreement templates for rapid KAM productivity?
- **Why it matters:** KAMs frequently create similar agreements with the same terms for different customers (e.g., standard framework agreement for a customer segment). Templates reduce data entry errors.
- **Suggested options:**
  - A) Phase 2 feature (out of scope for v1.0)
  - B) Store template as an Agreement in DRAFT status with a `isTemplate = true` flag; clone on use
  - C) Separate Template entity (more complex; separate lifecycle)
- **Owner:** TBD

---

## Q-AGR-011: Feature Dependency Register

- **Question:** Which product features (F-AGR-NNN) depend on sd.agr service endpoints?
- **Why it matters:** The §11.2 feature dependency register is currently populated with anticipated feature IDs (F-AGR-001 through F-AGR-005) but no corresponding feature specs have been authored. Without feature specs, the register cannot be validated and BFF aggregation hints remain speculative.
- **Suggested options:**
  - A) Author feature specs for F-AGR-001 through F-AGR-005 before this spec reaches APPROVED status
  - B) Mark §11 as DRAFT pending feature spec authoring; promote spec to APPROVED with the caveat
- **Owner:** TBD

---

## Q-AGR-012: SAP Migration Number Strategy

- **Question:** How should migrated SAP contract and rebate agreement numbers be handled to avoid collision with the new `AGR-YYYY-NNNNN` and `REB-YYYY-NNNNN` number series?
- **Why it matters:** SAP contract numbers (e.g., `40000001`) are numeric strings. The OpenLeap format is `AGR-YYYY-NNNNN`. Without a clear strategy, migrated contracts may collide with new agreements created on day one of go-live.
- **Suggested options:**
  - A) Prefix migrated contracts with `AGR-SAP-` (e.g., `AGR-SAP-40000001`) — easily identifiable but breaks the standard pattern
  - B) Use a separate number range for migrated contracts starting at a high base (e.g., `AGR-2000-NNNNN`) reserved for legacy
  - C) Store the SAP contract number in `customFields.sapContractNumber` and assign a new AGR number — clean separation but requires cross-reference lookup during cutover
- **Owner:** TBD

---

## Q-AGR-013: Inactive Customer Party Behavior

- **Question:** When `bp.bp.party.updated` indicates a customer party is now inactive, should sd.agr automatically take action on that customer's active agreements?
- **Why it matters:** If a customer is marked inactive (e.g., due to bankruptcy or account closure), continuing to allow releases against their agreements could create financial exposure.
- **Suggested options:**
  - A) Flag for review only — alert AGR_MANAGER; no automatic state change (current behavior)
  - B) Auto-terminate all ACTIVE agreements for the inactive customer (aggressive; may have false positives)
  - C) Auto-suspend (custom status) — block releases but don't terminate; allow manual reactivation
- **Owner:** TBD

---

## Q-AGR-014: Minimum Rebate Settlement Threshold

- **Question:** Should rebate settlements below a minimum amount be deferred to the next settlement period rather than creating a credit memo?
- **Why it matters:** Very small credit memos (e.g., €0.50) generate unnecessary billing and accounting overhead. Many rebate programs in distribution and manufacturing have minimum settlement thresholds.
- **Suggested options:**
  - A) No minimum — always settle regardless of amount (current design)
  - B) Configurable minimum per rebate agreement — add `minimumSettlementAmount` field; if calculated amount < minimum, carry over to next period
  - C) Tenant-level configurable default minimum; overridable per rebate agreement
- **Owner:** TBD
