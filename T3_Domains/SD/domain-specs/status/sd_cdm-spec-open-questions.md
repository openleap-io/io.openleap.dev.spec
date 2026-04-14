# Open Questions - sd.cdm (Credit Decision Management)

**Last Updated:** 2026-04-03

---

## Q-CDM-001: External Credit Bureau Integration

- **Question:** Should sd.cdm integrate with external credit bureaus (Schufa, Dun & Bradstreet) to enrich the credit assessment with external credit scores?
- **Why it matters:** Internal scoring alone is limited to payment history within the OpenLeap system. New customers have no history. External scores (e.g., Schufa Bonitätsscore, D&B PAYDEX) would allow more accurate risk classification at onboarding and reduce bad debt for new accounts. The `creditScoreExternal` field is already modelled in CreditProfile to receive this data.
- **Suggested options:**
  - A) Phase 3 via an adapter service — define an `ext.cdm.post-credit-limit-change` extension hook or a scheduled bureau query job; CDM does not call bureaus directly
  - B) Direct API integration — add `GET /credit-profiles/{id}:fetch-external-score` endpoint that calls the bureau and updates `creditScoreExternal` (adds external dependency to CDM)
  - C) Bureau-as-a-Service — dedicate a separate `cdm-bureau-adapter-svc` microservice that handles rate limits, caching, and consent management; publishes `cdm.external.score.updated` event to CDM
- **Owner:** TBD
- **Priority:** Phase 2/3 — does not block v1.0 launch

---

## Q-CDM-002: Credit Insurance Integration

- **Question:** Should sd.cdm support credit insurance — where an insurer sets an approved credit line per customer that may override or supplement the internal credit limit?
- **Why it matters:** Many B2B businesses insure trade receivables (e.g., Euler Hermes, Atradius). The insurer sets a credit limit per customer. If the insured limit differs from the internal limit, the effective credit check limit is the lower of the two. Without modeling this, companies using credit insurance cannot automate their credit management with OpenLeap.
- **Suggested options:**
  - A) Deferred — current design uses a single `creditLimit`; credit insurance tracked manually via `customFields.creditInsurancePolicyNumber`
  - B) Add `creditInsuranceLimit: Money?` field to CreditProfile; effective limit = `MIN(creditLimit, creditInsuranceLimit)` — requires change to BR-CDM-003 logic
  - C) Model `CreditInsurancePolicy` as a separate aggregate with its own lifecycle; CreditProfile references it; check engine queries both limits
- **Owner:** TBD
- **Priority:** Phase 2 — required for manufacturing and distribution verticals with large customer concentrations

---

## Q-CDM-003: Multi-Currency Credit Limits

- **Question:** Should credit limits be expressible in multiple currencies, and should credit checks convert the `requestedAmount` to the limit currency for comparison?
- **Why it matters:** International businesses often have customers that place orders in different currencies (e.g., a German customer placing EUR and USD orders). The current design requires the requestedAmount currency to match the credit limit currency (CDM_CURRENCY_MISMATCH error). This blocks multi-currency trading partners.
- **Suggested options:**
  - A) Single currency only (current v1 design) — simpler; currency conversion is handled upstream before submitting the check
  - B) Accept cross-currency checks — add currency conversion at check time using exchange rates from `ref-data-svc`; convert `requestedAmount` to `creditLimit.currencyCode` before comparison; snapshot the rate used in `CreditCheck` for audit
  - C) Multi-currency limits — store an array of `{currency, limit}` pairs on CreditProfile; use the matching currency limit for each check; fallback to base currency if no matching limit
- **Owner:** TBD
- **Priority:** Phase 2 — required before any international customer deployment

---

## Q-CDM-004: Group-Level Credit Management (Parent/Subsidiary)

- **Question:** Should the system support consolidated credit management across a corporate group, where a parent company has a group-level credit limit shared among subsidiaries?
- **Why it matters:** Enterprise customers often have parent/subsidiary relationships (e.g., Volkswagen AG and its brands). Individual subsidiary credit limits may each be small, but the group relationship means the parent guarantees payment. Without group support, credit controllers must manually coordinate limits across subsidiaries, losing visibility into total group exposure.
- **Suggested options:**
  - A) Deferred — each subsidiary has its own independent CreditProfile (current design); group relationships tracked via `customFields`
  - B) Add `CreditGroup` aggregate — group has its own `groupCreditLimit`; CreditProfile references a group; exposure check evaluates: `individualLimit` AND `groupLimit − groupTotalExposure`; the more restrictive applies
  - C) Use BP hierarchy — rely on BP suite's party hierarchy to resolve group membership; CDM queries BP to find parent and aggregate exposure across subsidiaries in real-time (performance concern for large groups)
- **Owner:** TBD
- **Priority:** Phase 3 — complex model change; requires BP party hierarchy spec first

---

## Q-CDM-005: Feature Dependency Register Validation

- **Question:** Which specific feature IDs from the SD suite feature catalog depend on sd.cdm service endpoints? The §11.2 register uses anticipated feature codes (F-CDM-001 through F-CDM-006) that have not yet been validated against authored feature specs.
- **Why it matters:** The feature dependency register (§11.2) drives BFF spec authoring and product configuration (UVL feature selection). If anticipated feature IDs (F-CDM-001 through F-CDM-006) do not match the actual feature catalog, BFF specs and product config files will reference non-existent features.
- **Suggested options:**
  - A) Author feature specs for F-CDM-001 through F-CDM-006 before this spec reaches APPROVED status; update §11.2 with validated IDs
  - B) Mark §11 as DRAFT pending feature spec authoring; promote sd_cdm-spec to APPROVED with explicit caveat that §11.2 is unvalidated
- **Owner:** TBD
- **Priority:** Required before APPROVED status; does not block DRAFT→REVIEW promotion

---

## Q-CDM-006: Credit Memo Exposure Adjustment

- **Question:** Should CDM provide a dedicated REST endpoint to immediately reduce open receivables exposure when a credit memo is issued, rather than waiting for the `fi.ar.exposure.changed` event?
- **Why it matters:** When a large credit memo is issued (e.g., returning €50,000 of goods), the customer's available credit should increase immediately. The `fi.ar.exposure.changed` event from FI.AR may arrive seconds to minutes later, depending on event processing lag. During that window, the customer's orders may be incorrectly blocked. For high-value, time-sensitive transactions this lag is operationally significant.
- **Suggested options:**
  - A) Rely solely on `fi.ar.exposure.changed` event (current design) — simpler; consistent with event-driven principles; lag is acceptable in most scenarios
  - B) Add `POST /api/sd/cdm/v1/credit-profiles/{id}:adjust-exposure` endpoint — allows FI team or credit controller to immediately credit/debit an exposure component; requires careful authorization (CDM_CONTROLLER minimum) and audit trail
  - C) Subscribe to additional FI events (e.g., `fi.ar.creditmemo.posted`) with higher event priority — reduces lag without adding REST endpoint
- **Owner:** TBD
- **Priority:** Phase 2 — operational improvement; not blocking for v1.0

---

## Q-CDM-007: Currency Mismatch Handling

- **Question:** How should CDM handle a credit check where `requestedAmount.currencyCode` differs from `creditProfile.creditLimit.currencyCode`?
- **Why it matters:** The current v1 design rejects such checks with `CDM_CURRENCY_MISMATCH` (422 error). This means sd.ord must convert order amounts to the credit limit currency before submitting checks. This puts the conversion responsibility on the order service, which may not have up-to-date exchange rates. Additionally, if Q-CDM-003 (multi-currency limits) is resolved with option B (conversion at check time), this question becomes moot.
- **Suggested options:**
  - A) Keep current behavior — caller (sd.ord) must convert to limit currency before submitting; CDM rejects mismatches (simplest; clear responsibility)
  - B) CDM converts at check time — use exchange rate from `ref-data-svc` at check time; snapshot the applied rate in `CreditCheck` for audit; adds ref-data dependency to the critical check path
  - C) Reject with helpful error — return the expected currency in the error response so the caller knows what to convert to (current behavior + better error message)
- **Owner:** TBD
- **Priority:** Must resolve before multi-currency deployment; linked to Q-CDM-003
