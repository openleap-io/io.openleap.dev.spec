# Open Questions — srv.ent (Entitlements)

## Q-ENT-001: Pricing / Valuation References

- **Question:** Should entitlement pricing or valuation ever be referenced in `srv.ent` (e.g., a `unitPrice` or `contractValue` field), or should this information strictly remain in `sd`/`fi`?
- **Why it matters:** If pricing hints are needed for billing intent generation in `srv.bil`, a reference field may need to be added to the data model, affecting the scope boundary with `sd`/`fi`.
- **Suggested options:**
  - A) No pricing data in `srv.ent`; `srv.bil` retrieves pricing from `sd`/`com` independently
  - B) Store a read-only pricing hint (e.g., `unitPrice`) for billing intent generation convenience
- **Owner:** TBD (Product Owner + Finance domain lead)

---

## Q-ENT-002: Reservation Mandatory vs Optional

- **Question:** Is quota reservation at appointment booking time mandatory to prevent oversubscription, or optional per deployment configuration?
- **Why it matters:** Affects the booking flow: if reservation is optional, `srv.apt` does not need to call `srv.ent` before confirming a booking, simplifying the critical path. If mandatory, the reservation call is on the critical path and must meet the < 200ms SLA.
- **Suggested options:**
  - A) Mandatory — all bookings must reserve quota before confirmation
  - B) Optional per product/tenant configuration — reservation can be bypassed if oversubscription risk is acceptable
  - C) Conditional — mandatory for prepaid entitlements, optional for unlimited entitlements
- **Owner:** TBD (Product Owner)

---

## Q-ENT-003: Event Keys from `sd`/`com` for Entitlement Purchase

- **Question:** What are the exact routing keys published by the `sd` (Sales) and `com` (Commerce) suites when a customer purchases a service entitlement package?
- **Why it matters:** `srv.ent` must subscribe to these events to create Entitlement aggregates. Without the exact keys, the consumed event handler (`EntitlementPurchaseEventHandler`) cannot be configured. This is a blocking dependency for integration testing.
- **Suggested options:** Coordinate with `sd`/`com` domain owners to agree on canonical routing key patterns (e.g., `sd.order.entitlement.purchased`, `com.checkout.subscription.activated`)
- **Owner:** TBD (sd/com domain lead + srv team)

---

## Q-ENT-004: Service Port Number

- **Question:** What port does `srv-ent-svc` run on in the local development and staging environments?
- **Why it matters:** Required for deployment configuration, service discovery registration, and developer documentation.
- **Suggested options:** Follow suite-wide port allocation convention (OPEN QUESTION at suite level)
- **Owner:** team-srv

---

## Q-ENT-005: Repository URI

- **Question:** What is the Git repository URI for `srv-ent-svc`?
- **Why it matters:** Required for ADR references, CI/CD pipeline configuration, and the spec metadata block.
- **Suggested options:** `https://github.com/openleap/srv-ent-svc` (or internal GitLab equivalent)
- **Owner:** team-srv

---

## Q-ENT-006: `QuotaAllocation` Value Object

- **Question:** Should a `QuotaAllocation` value object be introduced to encapsulate the three quota counters (`totalQuota`, `consumedQuota`, `reservedQuota`) as an immutable snapshot, rather than having them as flat scalar attributes on the `Entitlement` aggregate root?
- **Why it matters:** A value object would make state transitions more explicit and allow the balance computation (`availableQuota = total - consumed - reserved`) to be encapsulated in a single object. However, it adds complexity to the aggregate and the data model mapping.
- **Suggested options:**
  - A) Keep flat scalar counters on `Entitlement` (current approach; simpler)
  - B) Introduce `QuotaAllocation` value object (cleaner DDD, but more mapping overhead)
- **Owner:** TBD (Architecture Team)

---

## Q-ENT-007: Legal Retention Period for Entitlement and Transaction Records

- **Question:** What is the legally required retention period for `ent_entitlement` and `ent_quota_transaction` records?
- **Why it matters:** Determines the data retention policy, archive strategy, and GDPR right-to-erasure approach (can we anonymize customer references while retaining accounting records?).
- **Suggested options:**
  - A) 7 years — standard accounting record retention in most EU jurisdictions
  - B) 10 years — conservative; applies in some regulated industries (healthcare, financial services)
  - C) Jurisdiction-dependent — configure per tenant deployment
- **Owner:** Legal / Compliance team

---

## Q-ENT-008: Additional Planned Features Depending on `srv.ent`

- **Question:** Are there additional product features planned that depend on `srv.ent` endpoints, beyond the currently identified F-SRV-006 (Entitlements), F-SRV-002 (Appointment & Booking), and F-SRV-010 (Customer Balance View)?
- **Why it matters:** Completeness of the feature dependency register in §11; affects BFF design and impact assessment.
- **Suggested options:** Coordinate with Product Owner to review the full feature catalog for `srv` suite
- **Owner:** TBD (Product Owner)

---

## Q-ENT-009: Extension Management API Versioning

- **Question:** Should extension management endpoints (register hook, register custom field, list rule slots) be versioned as part of the main `/api/srv/ent/v1` namespace, or have a separate versioning prefix (e.g., `/api/srv/ent/extension/v1`)?
- **Why it matters:** Extension management endpoints have a different lifecycle than the core business API — they evolve with the extensibility framework (ADR-067) rather than the business domain.
- **Suggested options:**
  - A) Same v1 namespace: `/api/srv/ent/v1/extension/...` (simpler; consistent base path)
  - B) Separate versioning: `/api/srv/ent/extension/v1/...` (allows independent versioning of extensibility API)
- **Owner:** Architecture Team
