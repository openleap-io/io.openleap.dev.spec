# Open Questions — srv.cas

## Q-CAS-001: Port Assignment for srv-cas-svc

- **Question:** What port is assigned to `srv-cas-svc` in the OpenLeap service registry and local development setup?
- **Why it matters:** Required for service registry registration, Docker Compose configuration, and developer onboarding documentation. Without this, the service cannot be started locally alongside other SRV services.
- **Suggested options:**
  - A) Follow the SRV suite port range convention (check `_srv_suite.md` or SRV Docker Compose)
  - B) Assign the next available port in the SRV range
- **Owner:** TBD

---

## Q-CAS-002: Repository URI for srv-cas-svc

- **Question:** What is the Git repository URI for the `srv-cas-svc` implementation?
- **Why it matters:** Required for the `metadata.repository` field in the service identity block and for developer navigation from spec to code.
- **Suggested options:**
  - A) `github.com/openleap/io.openleap.srv.cas`
  - B) Sub-module within an SRV monorepo
  - C) OPEN — repository not yet created
- **Owner:** TBD

---

## Q-CAS-003: Maximum Number of Session Links per Case

- **Question:** Is there a maximum number of session links allowed per case? A 10-session physiotherapy series has a natural upper bound, but a multi-year consulting mandate might accumulate hundreds of sessions.
- **Why it matters:** Affects UI pagination strategy, storage estimates, potential table partitioning, and whether a BR-004 should be added with a configurable maximum.
- **Suggested options:**
  - A) No hard limit — paginate the UI; no enforcement in the platform
  - B) Soft limit (e.g., 500) — enforced as a warning, not an error
  - C) Hard limit enforced as a business rule (configurable per tenant/product)
- **Owner:** TBD

---

## Q-CAS-004: Entitlement Validation Depth at Case Creation

- **Question:** When `entitlementId` is provided at case creation, should `srv.cas` validate the entitlement against `srv.ent`? And if the entitlement is not found, should creation fail (hard) or proceed with a warning (soft)?
- **Why it matters:** Affects reliability of the create operation. If `srv.ent` is temporarily unavailable, a hard validation would block all case creation with entitlements. The entitlement link is informational at creation time — entitlement consumption is driven by session linking, not case creation.
- **Suggested options:**
  - A) Validate and fail on 404 — case cannot be created without a valid entitlement
  - B) Validate and warn (non-blocking) — creation proceeds; warning in response
  - C) No validation at case creation — validate only at session linking time
  - D) Validate asynchronously — create proceeds; event published for entitlement check
- **Owner:** TBD

---

## Q-CAS-005: OpenAPI Documentation URL

- **Question:** What is the URL for the Swagger UI / OpenAPI documentation for `srv-cas-svc`?
- **Why it matters:** Required for §6.4 OpenAPI Specification reference and for integration engineers.
- **Suggested options:**
  - A) `/api/srv/cas/swagger-ui.html` (Spring Boot default)
  - B) Central developer portal URL (e.g., `developer.openleap.io/srv/cas`)
- **Owner:** TBD

---

## Q-CAS-006: Auto-Activation of Case on First Appointment Booking

- **Question:** When `srv.apt.appointment.booked` is consumed with a `caseId`, should `srv.cas` automatically transition the case from OPEN to ACTIVE?
- **Why it matters:** Affects case lifecycle automation and the meaning of the OPEN state. If the first appointment booking auto-activates the case, the OPEN→ACTIVE transition is implicit. Otherwise, an explicit `ActivateCase` command is required.
- **Suggested options:**
  - A) Yes — first appointment booking automatically activates the case (OPEN → ACTIVE)
  - B) No — explicit `ActivateCase` REST call is always required
  - C) Configurable per tenant/product — platform default is "no auto-activation"
- **Owner:** TBD

---

## Q-CAS-007: Regulated Industry Compliance Requirements

- **Question:** In which regulated industry contexts does `srv.cas` operate? Medical (HIPAA/IS-H), educational, or general professional services?
- **Why it matters:** Determines whether additional compliance controls (HIPAA Business Associate Agreement, data residency requirements, mandatory retention periods) apply. Medical contexts require stricter data handling than general professional services.
- **Suggested options:**
  - A) Medical (IS-H, HIPAA, specific data retention and audit requirements)
  - B) Educational (FERPA or local equivalents)
  - C) General professional services only (GDPR sufficient)
  - D) All of the above — compliance is product-specific; platform provides GDPR baseline; industry extensions add vertical compliance
- **Owner:** TBD

---

## Q-CAS-008: GDPR Right-to-Erasure Cascade Strategy

- **Question:** When a customer (BusinessPartner) is deleted in `shared.bp` due to a GDPR right-to-erasure request, how does `srv.cas` handle the cascade deletion of all case records for that customer?
- **Why it matters:** Regulatory requirement. All personal data must be removed within the legally required timeframe. The strategy affects whether `srv.cas` subscribes to a deletion event or exposes a deletion endpoint.
- **Suggested options:**
  - A) Subscribe to `bp.businessPartner.deleted` event and cascade hard-delete all `cas_case` records for `customerPartyId`
  - B) Expose a `DELETE /cases?customerPartyId={id}` endpoint called by the `shared.bp` data erasure service
  - C) Anonymize rather than delete — replace `customerPartyId` with a sentinel UUID; retain case structure for reporting
- **Owner:** TBD

---

## Q-CAS-009: Exact Feature IDs in SRV Suite Feature Catalog

- **Question:** What are the authoritative feature IDs (F-SRV-005-xx) for case management features in the SRV suite feature catalog?
- **Why it matters:** The feature IDs in §11 (Feature Dependencies) are currently provisional. They must match the IDs in the SRV feature specs and `_srv_suite.md` to enable correct BFF configuration and feature gating.
- **Suggested options:** Check `spec/T3_Domains/SRV/_srv_suite.md` feature section and any existing feature specs under `spec/T3_Domains/SRV/features/`.
- **Owner:** TBD

---

## Q-CAS-010: Case Types — srv.cat vs. Custom Fields

*(Migrated from Q-001 in previous spec version)*

- **Question:** Do we need configurable "case types" per industry in a service catalog (`srv.cat`) or via the custom fields extension model?
- **Why it matters:** Affects the extensibility architecture. A `CaseType` reference data entity in `srv.cat` (if that service exists) would provide structured, validated case type classifications with display metadata. The custom field approach (`caseTypeCode` as a string extension field) is simpler but less structured.
- **Suggested options:**
  - A) Types in `srv.cat` — structured reference data with validation and display config
  - B) Custom field `caseTypeCode` — free-form or constrained to product-defined enum values
  - C) Dedicated `CaseType` aggregate within `srv.cas` — owned by this service, not shared
- **Owner:** TBD

---

## Q-CAS-011: Plan Templates Ownership

*(Migrated from Q-002 in previous spec version)*

- **Question:** Should `srv.cas` own "plan templates" (treatment/lesson plan templates that define expected session sequences) or only plan instances (actual case records)?
- **Why it matters:** Affects the scope boundary between `srv.cas` and `ps` (project/planning suite). If plan templates belong in `srv.cas`, the domain model needs a `CasePlanTemplate` aggregate. If templates belong in `ps` or a future `srv.cat`, then `srv.cas` only instantiates plans from templates owned elsewhere.
- **Suggested options:**
  - A) Templates + instances in `srv.cas` — self-contained case management
  - B) Instances only in `srv.cas`; templates in `ps` (project planning) or a shared catalog
  - C) No templates — cases are always free-form (no template-based guidance)
- **Owner:** TBD
