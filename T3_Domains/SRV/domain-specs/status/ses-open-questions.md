# Open Questions - srv.ses

## Q-SES-001: Port Assignment

- **Question:** What is the assigned port number for `srv-ses-svc` in the local development environment and service registry?
- **Why it matters:** Required for service registry entries, local development `docker-compose.yml`, and cross-service REST calls during development.
- **Suggested options:**
  - Option A: Follow the SRV suite port allocation block (check with infra team for the next free port in the SRV range)
  - Option B: Use dynamic port assignment with service discovery
- **Owner:** TBD (Infrastructure / Platform team)

---

## Q-SES-002: Repository URI

- **Question:** What is the GitHub/GitLab repository URI for `srv-ses-svc`?
- **Why it matters:** Required for the §2 Service Identity table, OpenAPI documentation URL (§6.4), and developer onboarding.
- **Suggested options:**
  - Option A: `https://github.com/openleap/io.openleap.srv.ses`
  - Option B: Follow the repository naming convention already established in other SRV services
- **Owner:** TBD (Platform team)

---

## Q-SES-003: outcomeCode Vocabulary

- **Question:** What is the platform's `outcomeCode` vocabulary for session completion?
- **Why it matters:**
  - BR-008 enforces that `outcomeCode` must reference a known vocabulary value.
  - Billing rules in `srv.bil` may evaluate `outcomeCode` to determine billing intent type.
  - Reporting and analytics depend on a consistent set of outcome codes.
- **Suggested options:**
  - Option A: Minimal fixed vocabulary — `DELIVERED`, `PARTIAL_DELIVERY`, `CANCELLED_DURING`, `NO_SHOW_FEE`
  - Option B: Domain-configurable reference data managed in `param.ref` or `srv.cat` — allows per-industry customization
  - Option C: Start with Option A as platform baseline; allow product extension via `customFields.extendedOutcomeCode`
- **Owner:** TBD (Product + Architecture)

---

## Q-SES-004: Maximum Proof Artifacts per Session

- **Question:** Is there a maximum number of proof artifacts that can be attached to a single session?
- **Why it matters:**
  - Affects storage planning (each artifact is a DMS reference but also a `ses_proof_artifact` row).
  - Affects UI design (infinite scroll vs. fixed limit).
  - Affects API contract (collection constraint in §3.3.1 ProofArtifact).
- **Suggested options:**
  - Option A: No hard limit at the platform level (enforce only via product-level config)
  - Option B: Cap at 20 artifacts per session as a sensible default
  - Option C: Cap at 50 artifacts per session for edge cases (multi-stage checklist photos)
- **Owner:** TBD (Product + Architecture)

---

## Q-SES-005: outcomeCode as Shared Type vs. Plain String

- **Question:** Should `outcomeCode` be modeled as a Shared Type (`OutcomeCode`) in §3.5, or remain a validated plain string with reference data lookup?
- **Why it matters:**
  - If other aggregates in the SRV suite (e.g., billing intent in `srv.bil`) reference outcome codes, a shared type would enforce consistency.
  - A shared type would need to be defined in §3.5 and potentially in a shared library.
- **Suggested options:**
  - Option A: Keep as plain `String` with vocabulary validation via `param.ref` reference data (simpler, more flexible)
  - Option B: Introduce `OutcomeCode` as a value object in §3.5 with compile-time or runtime validation (more type-safe)
- **Owner:** TBD (Architecture)

---

## Q-SES-006: PostgreSQL Version Confirmation

- **Question:** Which PostgreSQL version is confirmed for the `srv-ses-svc` deployment environment?
- **Why it matters:**
  - `INTERVAL` type semantics differ slightly between versions.
  - JSONB GIN index performance improvements are version-dependent.
  - The upgrade prompt references PostgreSQL 16+; §8.1 documents "PostgreSQL 16+".
- **Suggested options:**
  - Option A: PostgreSQL 16 (current stable; recommended)
  - Option B: PostgreSQL 15 (previous LTS; still supported)
- **Owner:** TBD (Infrastructure team)

---

## Q-SES-007: Data Retention Period

- **Question:** What is the minimum required retention period for session records in `ses_session`?
- **Why it matters:**
  - Legal and compliance requirement. Session records are billing evidence and may be required for tax audits.
  - Regulated industries (healthcare, legal) may have extended retention requirements (7–10 years).
  - Affects GDPR anonymization timing: customer PII in `customer_party_id` cannot be pseudonymized before the retention period ends.
- **Suggested options:**
  - Option A: 7 years (standard commercial/accounting minimum in EU)
  - Option B: 10 years (conservative; suitable for healthcare/regulated industries)
  - Option C: Configurable per tenant/product with a platform minimum of 5 years
- **Owner:** TBD (Legal / Compliance)

---

## Q-SES-008: Synchronous Validation of serviceOfferingId

- **Question:** Should session creation synchronously validate that `serviceOfferingId` exists in `srv.cat`?
- **Why it matters:**
  - If yes: `srv.cat` becomes a synchronous dependency on the session creation critical path. A `srv.cat` outage would block all session creation from appointment events.
  - If no: Invalid `serviceOfferingId` references are accepted; reporting/billing may reference non-existent offerings.
  - This is an architectural trade-off between strict consistency and availability.
- **Suggested options:**
  - Option A: Strict validation at create time — reject session if offering not found (consistent, but fragile)
  - Option B: Accept without validation at create time; validate lazily at complete time (resilient, but deferred error detection)
  - Option C: Validate asynchronously — accept session, publish a validation event, and flag invalid sessions later
- **Owner:** TBD (Architecture)

---

## Q-SES-009: Additional Product Features Depending on srv.ses

- **Question:** Are there additional product features (beyond F-SRV-004-01/02/03, F-SRV-006, F-SRV-007) that depend on `srv.ses` REST endpoints or events?
- **Why it matters:**
  - The Feature Dependency Register in §11.2 must be complete to support correct BFF design and feature gating.
  - Missing feature dependencies may result in incomplete API coverage in product BFF specs.
- **Suggested options:**
  - Review product specs for the SRV suite when available
  - Consider: scheduling dashboard features that display live session status (would consume `session.started`)
  - Consider: customer portal session history feature (needs `GET /sessions` with customer scoping)
- **Owner:** TBD (Product team)
