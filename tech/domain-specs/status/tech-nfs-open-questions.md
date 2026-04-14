# Open Questions — tech.nfs (News Feed Service)

## Q-NFS-001: Queue Provisioning Failure Recovery

- **Question:** What is the recovery mechanism when RabbitMQ queue provisioning fails and a Subscription enters `FAILED_PROVISION` state? Is there an automatic retry? Does the platform alert the admin? Is there a self-service "retry provisioning" API endpoint?
- **Why it matters:** Without a defined recovery path, subscriptions stuck in `FAILED_PROVISION` require manual database intervention with no defined SLA. Feature F-TECH-004-01 (Browse Subscriptions) will surface `FAILED_PROVISION` subscriptions to platform admins who will have no action to take.
- **Suggested options:**
  - A: Automatic retry with exponential backoff up to 5 attempts; alert on final failure
  - B: Manual retry via `POST /subscriptions/{id}:reprovision` endpoint
  - C: Both A and B (automatic retry with manual override)
- **Owner:** TBD
- **Status:** Open

---

## Q-NFS-002: Enrichment API Failure Behavior

- **Question:** If the enrichment API call fails (timeout, 5xx, 401), should NFS:
  (A) retry enrichment up to N times then fail the delivery,
  (B) deliver the un-enriched thin event, or
  (C) let the subscription configuration choose the behavior via a policy field?
- **Why it matters:** This is a fundamental reliability trade-off. Option A gives subscribers richer data but risks delivery delays or failures that upstream the producer's outage. Option B ensures delivery but may send incomplete data. Option C increases configuration complexity.
- **Suggested options:**
  - A: Retry enrichment 3× then fail delivery (enrichment failures cascade to delivery failures)
  - B: Deliver un-enriched thin event on enrichment failure (partial data, always delivered)
  - C: New subscription field `enrichmentFailurePolicy: FAIL | DELIVER_THIN` (subscriber chooses)
- **Owner:** TBD
- **Status:** Open

---

## Q-NFS-003: Subscription Lifecycle Audit Events

- **Question:** Should NFS publish subscription lifecycle events (subscription.created, subscription.suspended, subscription.deprovisioned) to a platform exchange for audit consumers? The Tech Suite Spec (§5.4) states NFS publishes no domain events, but the audit trail (`iam-audit-svc`) typically consumes domain events from all platform services.
- **Why it matters:** Without published events, `iam-audit-svc` cannot receive lifecycle notifications through its standard consumption mechanism. The current spec reserves `nfs_outbox_events` for this purpose but does not yet commit to publishing.
- **Suggested options:**
  - A: No platform events; the outbox table is an internal implementation detail; `iam-audit-svc` accesses audit via the NFS REST API
  - B: Publish minimal lifecycle events to a `tech.nfs.events` exchange; update the Tech Suite Spec §5.4 and §5.3 event catalog accordingly
- **Owner:** Architecture Team
- **Status:** Open

---

## Q-NFS-004: Port Assignment Confirmation

- **Question:** Is port `8097` confirmed for `tech-nfs-svc`? This was inferred from the port sequence of existing tech services (JC on 8096, DMS on earlier ports).
- **Why it matters:** Port conflicts prevent local multi-service development; service discovery configurations reference port numbers.
- **Suggested options:** Confirm `8097` or assign from the platform port registry.
- **Owner:** Platform Infrastructure Team
- **Status:** Open

---

## Q-NFS-005: Repository URI Creation

- **Question:** Has the repository `https://github.com/openleap-io/io.openleap.tech.nfs` been created?
- **Why it matters:** Feature specs, CI/CD pipelines, and the landscape `implementation-status.json` reference this URI. Developer teams cannot set up the service without it.
- **Owner:** Platform Infrastructure Team
- **Status:** Open

---

## Q-NFS-006: Tenant Class Classification for Production Encryption Rule

- **Question:** How does NFS determine whether a tenant is a "production" tenant for enforcement of BR-NFS-008 (no `NONE` encryption)? Is there:
  (A) A tenant attribute `tenantClass: PRODUCTION|TEST|DEV` from `iam-tenant-svc`?
  (B) An environment variable or deployment config flag?
  (C) A platform-wide config entry in `param-cfg-svc`?
- **Why it matters:** Without a reliable tenant classification mechanism, BR-NFS-008 cannot be programmatically enforced. The alternative is relying on platform administrators to never use `NONE` in production — which is not a technical control.
- **Suggested options:** Option A (tenant attribute from IAM) is preferred for consistency with other cross-tenant policy enforcement patterns.
- **Owner:** IAM + NFS teams
- **Status:** Open

---

## Q-NFS-007: Source Exchange Validation

- **Question:** Should NFS validate that `sourceExchange` references a known, existing RabbitMQ exchange at subscription creation time? If so, where is the authoritative exchange catalog maintained?
- **Why it matters:** A subscription registering against a non-existent exchange will silently never receive events. From the subscriber's perspective, this looks identical to "no events have occurred yet" — making debugging difficult.
- **Suggested options:**
  - A: Validate against RabbitMQ Management API at creation time (real-time check; adds RabbitMQ as a runtime dependency for the create operation)
  - B: Maintain an exchange allowlist in NFS config or `param-cfg-svc`; validate against the allowlist
  - C: Accept any exchange; add a validation warning in the create response; document as operator responsibility
- **Owner:** TBD
- **Status:** Open

---

## Q-NFS-008: Key Rotation Dual-Key Transition Window

- **Question:** When a public key is rotated via `POST /subscriptions/{id}:rotateKey`, in-flight events encrypted with the old key may not yet have been consumed by the subscriber. Should NFS:
  (A) Maintain a configurable dual-key window (e.g., 24h) where both old and new keys are valid for verification purposes?
  (B) Switch immediately to the new key; treat the transition as the subscriber's operational responsibility?
  (C) Include key ID metadata in the delivery envelope so subscribers can identify which key was used for decryption?
- **Why it matters:** Subscribers that batch-consume or have slow queue processing may receive a mix of old-key and new-key messages after a rotation, causing decryption failures if they switch to the new key too quickly or too slowly.
- **Suggested options:** Option C (key ID in envelope) combined with Option B (immediate switch) provides the best balance of security (no dual-key storage) and subscriber operational flexibility.
- **Owner:** TBD
- **Status:** Open

---

## Q-NFS-009: Extension API Endpoints

- **Question:** What are the specific REST endpoints for managing extension rules, hooks, and custom field definitions for NFS subscriptions? §12.7 of the spec is currently a stub pending finalization of the `core-extension` module API in `io.openleap.dev.guidelines` (ADR-067).
- **Why it matters:** Product developers building extensions for NFS subscriptions (custom validation rules, aggregate hooks, custom fields) cannot implement or test these without a defined API.
- **Suggested options:** Follow the `core-extension` module API pattern once ADR-067 is fully documented in `io.openleap.dev.guidelines`. Assign an NFS team member to monitor ADR-067 progress and update §12.7 when the module API is stable.
- **Owner:** Platform Architecture Team (core-extension module)
- **Status:** Open — blocked on ADR-067 finalization
