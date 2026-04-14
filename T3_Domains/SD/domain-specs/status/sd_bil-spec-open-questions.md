# Open Questions - sd.bil

## Q-BIL-001: E-Invoicing Formats
- **Question:** Which structured electronic invoice formats should sd.bil support (ZUGFeRD, Peppol, Factur-X)?
- **Why it matters:** EU e-invoicing mandates are tightening. Germany mandates e-invoicing for B2B from 2025. Peppol is required for EU public procurement. This affects DMS output format and potentially a new structured-data output format alongside PDF.
- **Suggested options:** ZUGFeRD 2.3 (Germany/Austria), Peppol BIS 3.0 (EU-wide), Factur-X (France/Germany), XRechnung (Germany public sector)
- **Owner:** TBD

## Q-BIL-002: Inter-Company Billing
- **Question:** Is inter-company billing required (one legal entity bills another within the same tenant)?
- **Why it matters:** Multi-entity corporate groups require internal invoices between subsidiaries. This affects party model (internal vs external parties), tax treatment, and AR posting logic.
- **Suggested options:** Phase 3 — requires inter-company partner concept in party service; defer until BP suite supports legal entity hierarchy
- **Owner:** TBD

## Q-BIL-003: Self-Billing
- **Question:** Is self-billing required (buyer creates invoice on behalf of supplier)?
- **Why it matters:** Required in automotive and retail supply chains where buyers (not sellers) generate invoices. Reverses the normal billing flow.
- **Suggested options:** Deferred — fundamentally different flow; would require a separate self-billing aggregate or service
- **Owner:** TBD

## Q-BIL-004: Revenue Recognition Integration (IFRS 15 / ASC 606)
- **Question:** Should sd.bil integrate with a revenue recognition engine for multi-element arrangements and deferred revenue?
- **Why it matters:** Under IFRS 15 / ASC 606, revenue for bundled products/services must be allocated across performance obligations. Deferred revenue requires a separate recognition event post-billing.
- **Suggested options:** Phase 2 — requires a `rev-rec` service receiving billing events and computing recognition schedules
- **Owner:** TBD

## Q-BIL-005: Invoice Split Criteria
- **Question:** Which combination of attributes triggers automatic invoice splitting into separate billing documents?
- **Why it matters:** Different payers, payment terms, or billing dates on order lines should generally produce separate invoices. Split logic determines BillingDocument granularity in batch billing runs.
- **Suggested options:** Split by: (a) payer party, (b) payment terms code, (c) billing date, (d) currency, (e) delivery. Configurable per tenant.
- **Owner:** TBD

## Q-BIL-006: Multi-Currency Billing Plans
- **Question:** How are currency conversions handled when billing plan items are invoiced in a different currency than the plan's base currency?
- **Why it matters:** Plan total amount is agreed in contract currency (e.g., EUR), but individual invoices may be required in customer's local currency (e.g., GBP). Exchange rate application and tolerance need to be defined.
- **Suggested options:** Option A: All plan items must be in plan currency (simple). Option B: Per-item currency with exchange rate from ref-data-svc (complex). Recommend Option A for Phase 1.
- **Owner:** TBD

## Q-BIL-007: Feature IDs for sd.bil
- **Question:** Which feature specs and assigned feature IDs cover the capabilities of sd.bil?
- **Why it matters:** Feature IDs are required to complete §11 Feature Dependency Register, UVL catalog entries, and BFF spec references. Without feature IDs, the billing service cannot be included in product configurations.
- **Suggested options:** Author feature specs F-BIL-001 through F-BIL-005 (Billing Document Management, Billing Plan Management, Credit Memo Management, Batch Billing Runs, Billing Block Management)
- **Owner:** TBD

## Q-BIL-008: Developer Portal and API Documentation URL
- **Question:** What is the URL for the API documentation hosting (developer portal, Swagger UI, Redoc)?
- **Why it matters:** Required for §6.4 OpenAPI Specification reference and for integration engineers accessing the live API documentation.
- **Suggested options:** Internal dev portal at `dev.openleap.io/docs/sd/bil`; or hosted Redoc alongside service
- **Owner:** TBD

## Q-BIL-009: Migration Tooling and Cut-Over Strategy
- **Question:** What is the approach for migrating historical billing data from the legacy system (e.g., SAP) to sd.bil?
- **Why it matters:** Historical billing data integrity is required for customer dispute resolution, audit trails, and revenue reporting continuity. SAP tables VBRK/VBRP contain the source data.
- **Suggested options:** (a) Spring Batch migration job; (b) one-time ETL with validation report; (c) read-only shadow access to legacy; (d) no migration (archive legacy, go-live from cut-over date)
- **Owner:** TBD

## Q-BIL-010: Async vs Sync Tax Determination
- **Question:** Should tax determination remain a synchronous REST call to fi.tax, or should it be refactored to async (event-driven with tax cache)?
- **Why it matters:** Synchronous fi.tax dependency adds latency (est. 50–150ms) and creates coupling. If fi.tax is unavailable, billing creation is blocked. Async with cached rates would decouple but introduce eventual consistency for tax amounts.
- **Suggested options:** Option A: Sync REST (current — simpler, tax accurate on create). Option B: Async with tax rate cache refreshed by fi.tax events (faster, more resilient, eventual consistency). Recommend Option A for Phase 1, revisit if fi.tax becomes a bottleneck.
- **Owner:** TBD

## Q-BIL-011: Multiple Billing Plans per Sales Order
- **Question:** Can a single sales order have multiple concurrent billing plans (e.g., one for services, one for hardware)?
- **Why it matters:** Complex project contracts may need separate billing schedules per order component. Current constraint (one plan per order) simplifies lifecycle but may be too restrictive.
- **Suggested options:** Option A: Keep one plan per order (current). Option B: Multiple plans per order, differentiated by plan type or product category. Recommend Option A until a concrete business case arises.
- **Owner:** TBD
