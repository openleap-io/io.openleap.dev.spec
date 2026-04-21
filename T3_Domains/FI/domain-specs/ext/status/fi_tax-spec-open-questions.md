# Open Questions - fi.tax

## Q-TAX-001: Feature Dependency Register
- **Question:** Which product features depend on fi-tax-svc endpoints?
- **Why it matters:** Needed to fully populate SS11 Feature Dependency Register and assess blast radius of API changes.
- **Suggested options:** Derive from product specs when available; currently planned features listed as placeholders.
- **Owner:** TBD

## Q-TAX-002: Interaction with fi.pst (Posting Orchestration)
- **Question:** How does fi.tax interact with fi.pst? The scope section mentions fi.pst but the current flow goes through fi.slc.
- **Why it matters:** Affects the posting workflow for tax settlements and whether fi.pst is an intermediary.
- **Suggested options:** Option A: fi.tax -> fi.slc directly (current design), Option B: fi.tax -> fi.pst -> fi.slc (posting orchestration layer)
- **Owner:** TBD

## Q-TAX-003: Cash-Basis Tax Recognition in Phase 1
- **Question:** Should cash-basis tax recognition be supported in phase 1?
- **Why it matters:** Affects TaxConfig.basisType usage, assessment lifecycle (when to recognize tax), and return preparation logic. Cash-basis VAT schemes exist in the UK (Flat Rate Scheme) and some EU countries for small businesses.
- **Suggested options:** Option A: Accrual only in phase 1 (simpler), Option B: Both accrual and cash from start (more complete)
- **Owner:** TBD

## Q-TAX-004: Tax Rate Changes for In-Flight Documents
- **Question:** How are tax rate changes handled for documents created before a rate change but posted after?
- **Why it matters:** Affects whether assessments lock the rate at creation time or can be recalculated. Example: Germany changed VAT from 19% to 16% temporarily in 2020.
- **Suggested options:** Option A: Lock rate at assessment creation time (simpler, audit-safe), Option B: Allow recalculation until POSTED status (flexible but complex)
- **Owner:** TBD

## Q-TAX-005: Withholding Tax (WHT) Calculation Workflow
- **Question:** What is the detailed withholding tax calculation workflow?
- **Why it matters:** WHT is listed in scope but the assessment workflow is not fully defined. WHT is deducted from vendor payments, not from invoice amounts.
- **Suggested options:** Option A: Integrated with AP bill assessment (single POST /assess call handles both VAT and WHT), Option B: Separate WHT assessment endpoint (POST /assess-wht)
- **Owner:** TBD

## Q-TAX-006: Multi-Tax Jurisdiction Stacking
- **Question:** How are multi-tax scenarios handled (e.g., US state + county + city sales tax)?
- **Why it matters:** Some US jurisdictions have stacked taxes (state 6% + county 1% + city 0.5% = 7.5%). The current model creates one TaxAssessmentLine per tax code, but stacking may require multiple lines per source document line.
- **Suggested options:** Option A: Multiple TaxAssessmentLines per source line (one per tax jurisdiction level), Option B: Compound tax codes that combine multiple rates
- **Owner:** TBD

## Q-TAX-007: Port Assignment for fi-tax-svc
- **Question:** Is port 8460 confirmed for fi-tax-svc in the infrastructure registry?
- **Why it matters:** Port must be unique across all services and registered in infrastructure configuration.
- **Suggested options:** Confirm with infrastructure team; port 8460 assumed based on FI suite port range.
- **Owner:** TBD

## Q-TAX-008: Multiple GL Accounts per Tax Type
- **Question:** Should TaxConfig support multiple GL accounts per tax type within a jurisdiction?
- **Why it matters:** Some jurisdictions require separate GL accounts for standard-rate VAT, reduced-rate VAT, and zero-rated VAT. The current model has one vatPayableAccountId per config.
- **Suggested options:** Option A: One account per config (current; simple), Option B: Account mapping table linking tax_type + rate_category -> GL account (more flexible)
- **Owner:** TBD
