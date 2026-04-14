# Open Questions - sd.ana

---

## Q-ANA-001: T4 BI Feed Strategy

- **Question:** Should sd.ana feed the T4 BI tier via events (current approach: `sd.ana.kpi.calculated`) or via direct DB replication / CDC?
- **Why it matters:** Events provide loose coupling but may lag and require T4 to build its own ingestion pipeline. CDC/logical replication is lower-latency but creates a direct dependency between T3 and T4 storage infrastructure.
- **Suggested options:**
  - Option A: Keep event-driven feed (current design) — `sd.ana.kpi.calculated` consumed by T4. Loose coupling, standard pattern.
  - Option B: PostgreSQL logical replication / CDC (Debezium) from `ana_kpi_snapshots` directly to T4 data warehouse. Lower latency (~seconds), but tighter coupling.
- **Owner:** TBD

---

## Q-ANA-002: Configurable KPI Definitions per Tenant

- **Question:** Should enterprise tenants be able to define custom KPI formulae (e.g. margin, units sold, weighted sales) in addition to the standard platform KPIs?
- **Why it matters:** Enterprise deployments may require non-standard metrics that cannot be served from the existing `SalesKPI` read model. Without this, product teams must hardcode tenant-specific logic into the core service.
- **Suggested options:**
  - Option A: Phase 2 — implement via Extension Rule Slot `EXT-RULE-ANA-004` (custom ranking metric) and a new `EXT-RULE-ANA-005` (custom KPI formula). Definitions stored in a new `ana_kpi_definitions` configuration table.
  - Option B: Create a separate `sd.tgt` (Sales Targets & KPI Config) domain that owns KPI definitions. sd.ana consumes definitions as configuration events.
- **Owner:** TBD

---

## Q-ANA-003: Sales Target Management (Plan vs. Actual)

- **Question:** Where should sales targets (quotas) and plan-vs-actual tracking live?
- **Why it matters:** Without targets, the analytics domain cannot show "achievement" metrics (e.g. 78% of monthly target). This is a core sales management capability.
- **Suggested options:**
  - Option A: Extend `sd.ana` with a `SalesTarget` read model and a write path for target entry (breaks the "pure read model" pattern).
  - Option B: Create a separate `sd.tgt` domain owning target management. sd.ana consumes target events and calculates achievement KPIs.
  - Option C: Delegate to product-level extension — individual products define their own target tracking via extension rules.
- **Owner:** TBD

---

## Q-ANA-004: Real-Time Streaming Dashboards

- **Question:** Should sd.ana support real-time pushed pipeline updates to UI clients (WebSocket / SSE) rather than polling?
- **Why it matters:** For high-volume sales operations (e.g. trading-floor-style monitoring, live bid tracking), polling-based dashboards with 30-second lag may not be sufficient.
- **Suggested options:**
  - Option A: Keep polling (current) — clients poll `GET /pipeline` every 30s. Simple, no infrastructure changes.
  - Option B: Server-Sent Events (SSE) — `GET /pipeline/stream` pushes pipeline change events when event projections complete. Medium complexity.
  - Option C: WebSocket — bidirectional connection; more complex but supports interactive filtering. Phase 3.
- **Owner:** TBD

---

## Q-ANA-005: Cross-Currency KPI Normalization

- **Question:** When a tenant operates in multiple currencies, should KPI aggregations normalize to a single reporting currency, or should multi-currency KPI rows be maintained separately?
- **Why it matters:** Currently, `ana_kpi_snapshots` stores one row per `(period, org, rep, currency)`. Dashboard widgets that try to show "total pipeline value" across orgs with different currencies will fail or produce misleading totals without normalization.
- **Suggested options:**
  - Option A: Reject cross-currency aggregation (current) — only same-currency totals shown. UI must handle multi-currency display.
  - Option B: Add a tenant-level `reporting_currency_code` configuration. KPI job converts all amounts to reporting currency using daily exchange rates from `ref.currency-svc`. Store both original and converted amounts.
  - Option C: Maintain separate KPI rows per currency and expose a `/kpis/multi-currency` endpoint that returns currency-tagged rows; normalization responsibility pushed to BFF.
- **Owner:** TBD

---

## Q-ANA-006: GDPR Erasure Propagation for Analytics Data

- **Question:** When a customer business partner or user is subject to a GDPR right-to-erasure request, how should the denormalized names and IDs in sd.ana be anonymised?
- **Why it matters:** `ana_pipeline_entries` stores `customer_name` (denormalized display name) and `customer_party_id`. If the customer is erased from the system, this data must also be anonymised. Similarly, `sales_rep_user_id` in KPI snapshots references a user who may request erasure.
- **Suggested options:**
  - Option A: Listen for a cross-domain `gdpr.erasure.requested` event (from IAM or a data governance service). On receipt, null out / anonymise the relevant fields in all `ana_*` tables for the affected entity ID.
  - Option B: Periodic reconciliation job — compare `customer_party_id` values in `ana_pipeline_entries` against an active BP list; anonymise stale entries.
  - Option C: For `customer_name` — accept that historical analytics data retains the name at the time of the transaction (as an audit record). Requires legal sign-off.
- **Owner:** TBD (Legal / Architecture joint decision)

---

## Q-ANA-007: SD Analytics Feature Spec IDs

- **Question:** Feature IDs `F-SD-ANA-001` through `F-SD-ANA-006` in §11.2 are placeholders. When will the SD analytics feature specs be authored?
- **Why it matters:** BFF specs and product `product-config.uvl` files need valid feature IDs to reference. Without authored feature specs, the analytics domain cannot be included in product configurations.
- **Suggested options:**
  - Author 6 feature specs covering: Pipeline View, KPI Dashboard, Revenue Forecast, Personal Performance Widget, Top Rankings, Return Analysis.
- **Owner:** Architecture Team

---

## Q-ANA-008: Extension Endpoint Registration Mechanism

- **Question:** §12.7 references `POST /api/sd/ana/v1/extensions/rule-slots/{slotId}/handlers` for registering extension handlers. This mechanism needs to be standardised across all SD services (and ideally platform-wide).
- **Why it matters:** If each domain invents its own extension registration API, integrators face inconsistent extension patterns. A platform-standard `core-extension` module (per ADR-067) should provide the registration endpoints.
- **Suggested options:**
  - Option A: Define a standard extension registration API in the `core-extension` module (`io.openleap.starter`). All domain services inherit it automatically via the starter.
  - Option B: Define a cross-SD ADR specifying the extension endpoint contract. Each domain implements it locally.
- **Owner:** Architecture Team
