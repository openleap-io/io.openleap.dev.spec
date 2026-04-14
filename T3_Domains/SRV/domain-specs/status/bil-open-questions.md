# Open Questions - srv.bil

## Q-BIL-001: Port Assignment

- **Question:** What port does `srv-bil-svc` run on?
- **Why it matters:** Required for service registry, local dev setup, and Docker Compose configuration. Missing port blocks implementation onboarding.
- **Suggested options:** Assign next available port in the SRV port range (check existing SRV service port allocations in `https://github.com/openleap-io/io.openleap.dev.hub/blob/main/landscape/impl-status.json`).
- **Owner:** TBD

---

## Q-BIL-002: Repository URI

- **Question:** What is the repository URI for `srv-bil-svc`?
- **Why it matters:** Required for service identity metadata (`metadata.repository`), CI/CD pipeline configuration, and developer portal API docs URL.
- **Suggested options:** `io.openleap/srv-bil-svc` or similar, consistent with other SRV service naming.
- **Owner:** TBD

---

## Q-BIL-003: PostgreSQL Instance Strategy

- **Question:** Should `srv.bil` use a shared PostgreSQL instance (shared SRV schema) or a dedicated database?
- **Why it matters:** Affects schema isolation, migration independence, connection pool sizing, and RLS policy scope.
- **Suggested options:**
  - A) Shared SRV schema (all SRV domains in one DB, separate schemas) — lower ops overhead, less isolation
  - B) Dedicated `bil` database — full isolation, independent migration, higher ops cost
- **Owner:** TBD

---

## Q-BIL-004: Downstream Consumer Routing (SD and/or FI)

- **Question:** Should `srv.bil` publish `ready_for_invoicing` events to both `sd` and `fi`, or pick one canonical consumer per deployment?
- **Why it matters:** Determines event routing topology and whether products must configure which consumer handles billing. Dual consumers may result in duplicate invoice creation if not handled carefully.
- **Suggested options:**
  - A) Both consumers active (configurable per product deployment)
  - B) Single canonical consumer selected at deploy time via routing config
  - C) Default to SD; FI optional for direct-invoice deployments
- **Owner:** TBD

---

## Q-BIL-005: Approval Workflow Before Intent Confirmation

- **Question:** Should there be a configurable approval workflow before DRAFT intents are confirmed (particularly relevant for regulated industries)?
- **Why it matters:** Determines whether the DRAFT → CONFIRMED transition is manual (clerk action), automatic, or approval-gated. Affects UX complexity and compliance posture.
- **Suggested options:**
  - A) Auto-confirm: no manual step; DRAFT advances automatically
  - B) Manual-confirm: billing clerk must explicitly confirm each intent
  - C) Configurable per product (or per service offering type)
- **Owner:** TBD

---

## Q-BIL-006: Fee Policy Ownership

- **Question:** Where do no-show and cancellation fee policies (percentage, cancellation window, tier structure) live — `srv.bil` local configuration or `srv.cat` service offering metadata?
- **Why it matters:** Affects domain boundary clarity and which service is responsible for fee policy CRUD. Also determines coupling between `srv.bil` event handlers and `srv.cat`.
- **Suggested options:**
  - A) `srv.bil` owns fee policies — local config table; `srv.bil` handlers query own DB
  - B) `srv.cat` provides fee policy hints — embedded in offering metadata; event payload carries relevant hints
  - C) Hybrid — `srv.cat` stores policy; `srv.bil` caches locally
- **Owner:** TBD

---

## Q-BIL-007: billing_intent_no Format and Sequence Strategy

- **Question:** What is the format and sequence generation strategy for the `billing_intent_no` business key?
- **Why it matters:** Required for DB schema design (UK constraint) and for generating human-readable reference numbers used in reconciliation and customer communication.
- **Suggested options:**
  - A) `BIL-{YYYY}-{NNNNNN}` — sequential per tenant per year; requires sequence table or DB sequence
  - B) `BIL-{YYYYMM}-{NNNN}` — monthly reset; shorter but less unique long-term
  - C) UUID-based opaque key — no human-readable value; simpler but less usable for clerks
- **Owner:** TBD

---

## Q-BIL-008: LineItem custom_fields Requirement

- **Question:** Should `bil_line_item` support `custom_fields JSONB` for product-specific line-level attributes?
- **Why it matters:** Added speculatively for product use cases (tax category, cost center, GL account on line level). If no product needs this, it adds schema complexity without benefit.
- **Suggested options:**
  - A) Yes — keep as spec'd (current): allows products to tag individual line items with billing codes
  - B) No — remove from spec; simplify schema; move all custom metadata to intent level
- **Owner:** TBD

---

## Q-BIL-009: Fee Policy Lookup Mechanism During Event Handling

- **Question:** How should `srv.bil` event handlers look up fee policy data (no-show %, cancellation free window) at event processing time?
- **Why it matters:** Synchronous REST calls to `srv.cat` during event handling introduce coupling and latency. A cache or embedded policy hint is more resilient but requires additional design.
- **Suggested options:**
  - A) Synchronous REST call to `srv-cat-svc` at event processing time — simple but fragile
  - B) Locally cached fee config in `srv.bil` (replicated from `srv.cat` via events) — resilient but complex
  - C) Fee policy hints embedded in triggering event payload (`no_show_marked`, `cancelled`) — decoupled; `srv.apt` enriches before publishing
- **Owner:** TBD

---

## Q-BIL-010: Auto-Confirm and Auto-Mark-Ready Configuration

- **Question:** Should DRAFT intents automatically advance through CONFIRMED to READY_FOR_INVOICING without manual clerk intervention, and how should this be configured?
- **Why it matters:** Auto-confirm eliminates latency to invoicing (important for high-volume deployments). Manual confirm is required for regulated industries or where dispute rates are high.
- **Suggested options:**
  - A) Always manual — maximum control; maximum clerk workload
  - B) Always auto — minimum latency; no oversight
  - C) Configurable per service offering type or product deployment — recommended
- **Owner:** TBD

---

## Q-BIL-011: Message Broker Topology

- **Question:** Does `srv.bil` use RabbitMQ (standard OpenLeap assumption) or Kafka?
- **Why it matters:** Affects queue naming conventions, consumer configuration, message ordering guarantees, and event replay capability.
- **Suggested options:**
  - A) RabbitMQ (assumed) — consistent with rest of OpenLeap platform; topic exchanges and routing keys as spec'd
  - B) Kafka (high-throughput deployments) — better ordering, replay; requires different consumer model and naming
- **Owner:** TBD

---

## Q-BIL-012: Full Feature Consumer Map for §11

- **Question:** Which product-level features beyond F-SRV-007 (Billing Intent) consume `srv.bil` REST endpoints or events?
- **Why it matters:** Required to complete §11 Feature Dependency Register and assess the full impact of API/event changes.
- **Suggested options:** Review product specs that include F-SRV-007, any "Finance Dashboard", "Reconciliation", or "Case Billing" features. Cross-check with BFF specs for SRV and FI suites.
- **Owner:** TBD

---

## Q-BIL-013: GDPR Right-to-Erasure for Append-Only Records

- **Question:** How should GDPR Article 17 (right to erasure / "right to be forgotten") be handled for append-only billing records that reference customer IDs?
- **Why it matters:** Financial records typically cannot be deleted under accounting law. However, GDPR requires erasure of personal data. The `customerPartyId` and potentially `reason` field may need to be pseudonymized.
- **Suggested options:**
  - A) Pseudonymize `customerPartyId` on erasure request: replace with a stable pseudonym; retain all other financial data
  - B) Legal hold exception: assert accounting law override for financial records; document GDPR basis
  - C) Separate PII store: store `customerPartyId` mapping outside the billing domain; replace with opaque reference in billing records
- **Owner:** TBD (Legal + Architecture)

---

## Q-BIL-014: Legacy System Migration Field Mapping

- **Question:** What is the source legacy system for `srv.bil` migration, and what is the detailed field-level mapping?
- **Why it matters:** §13.1 provides a generic SAP FI-BL/SD-BIL framework, but actual migration requires project-specific analysis of the customer's legacy billing system.
- **Suggested options:** Requires legacy system analysis per deployment project. Typical sources: SAP SD-BIL (VBRK/VBRP), custom billing spreadsheets, ERP export files.
- **Owner:** TBD (Implementation Project Team)

---

## Q-BIL-015: Money Value Object Definition

- **Question:** Should a `Money` value object (amount + currencyCode) be formally defined as a shared type in `srv.bil`, replacing the current primitive fields (`unitPriceHint` + `currencyCode`) on `LineItem`?
- **Why it matters:** A typed value object improves domain expressiveness and enables richer validation. However, it adds complexity and the current design intentionally keeps price hints as lightweight primitives (DC-002: authoritative pricing belongs to sd/fi).
- **Suggested options:**
  - A) Keep primitives (current) — minimal complexity; explicit that price hints are non-authoritative
  - B) Add `Money` value object — cleaner domain model; aligns with DDD best practices; implement in §3.5 Shared Types
- **Owner:** TBD
