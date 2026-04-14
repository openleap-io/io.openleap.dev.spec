# Open Questions - srv.apt

## Q-APT-001: Slots as First-Class Entities vs Derived

- **Question:** Does `srv.apt` own slots as first-class persistent entities, or are slots purely derived (computed on request) from `srv.res` availability + `shared.cal` calendar rules?
- **Why it matters:** A first-class slot entity requires a `apt_slot` table, slot lifecycle management, and slot-level conflict locking. A derived/virtual approach keeps the data model leaner but increases read-time complexity and dependency on `srv.res` + `shared.cal` for every slot discovery call.
- **Suggested options:**
  - A) First-class entities — stored slots with explicit availability flags; supports pre-publishing available slots for portal display
  - B) Derived/virtual — computed on each request; simpler data model; current spec assumes this approach
- **Owner:** TBD

---

## Q-APT-002: Minimum Policy Set for MVP

- **Question:** What is the minimum set of booking policies (cancellation cutoffs, no-show fees, waitlist auto-promotion) required for the MVP release?
- **Why it matters:** The policy engine (BR-005) is documented as configurable and SHOULD-level. Implementing a full policy engine is significantly more complex than a hard-coded cutoff. This decision affects implementation scope and timeline.
- **Suggested options:**
  - A) No policies for MVP (simplest) — cancel is always allowed; no-show fee is manual
  - B) Cancellation cutoff only — single configurable cutoff window per tenant
  - C) Full policy engine — per-offering cancellation, no-show, and rescheduling rules
- **Owner:** TBD

---

## Q-APT-003: ABAC Constraint for Customer Self-Service

- **Question:** How should customer self-service access be enforced? A customer logged into a portal should only be able to read and modify their own appointments, not any appointment in the tenant.
- **Why it matters:** Without this constraint, a customer with `SRV_APT_VIEWER` scope could read all appointments in the tenant (e.g., via GET /appointments?page=0). This is a PII/privacy risk.
- **Suggested options:**
  - A) ABAC via `customerPartyId` claim in JWT — system filters all queries to `customerPartyId = jwtClaim.partyId`
  - B) Separate role `SRV_APT_SELF` with server-side enforcement
  - C) BFF-enforced scoping — portal BFF injects `customerPartyId` filter before forwarding requests
- **Owner:** TBD

---

## Q-APT-004: Performance Targets

- **Question:** What are the exact performance targets for `srv.apt` operations in production? Specifically: slot discovery p95 latency, peak booking throughput (req/sec), and maximum concurrent users.
- **Why it matters:** These targets determine architecture decisions: whether slot discovery needs a dedicated cache, whether read replicas are needed at launch, and how many service instances to deploy.
- **Suggested options:**
  - Slot discovery: < 500ms p95 (multi-service query; caching recommended)
  - Booking confirmation: < 200ms p95 (already specified)
  - Peak write throughput: 50 req/sec (initial estimate)
  - Peak read throughput: 200 req/sec
  - Concurrent users: To be determined from product team projections
- **Owner:** TBD

---

## Q-APT-005: Port Assignment

- **Question:** What port should `srv-apt-svc` listen on in local development and staging environments?
- **Why it matters:** Needed for docker-compose configuration, service registry setup, and local developer documentation.
- **Suggested options:** Assign from the SRV service port block (presumably 8200–8299 or similar platform convention).
- **Owner:** team-srv

---

## Q-APT-006: Repository URI

- **Question:** What is the GitHub/GitLab repository URI for `srv-apt-svc`?
- **Why it matters:** Required for §2 Service Identity, CI/CD pipeline configuration, and developer onboarding documentation.
- **Suggested options:** Pattern: `github.com/openleap/io.openleap.srv.apt`
- **Owner:** team-srv

---

## Q-APT-007: Minimum Slot Duration Enforcement

- **Question:** Should `srv.apt` enforce that the appointment duration meets the service offering's minimum slot length? Currently BR-003 only checks `end > start`.
- **Why it matters:** Without a minimum duration check, a customer could book a 1-minute appointment for a service that requires 60 minutes. The offering catalog (`srv.cat`) presumably defines `minDuration` per offering.
- **Suggested options:**
  - A) Enforce: `end - start >= offering.minDuration` — add to BR-003
  - B) Leave to `srv.cat` or the UI — `srv.apt` trusts the duration passed in
- **Owner:** TBD

---

## Q-APT-008: OpenAPI Documentation URL

- **Question:** What is the public/internal URL for the hosted OpenAPI documentation for `srv-apt-svc`?
- **Why it matters:** Referenced in §6.4. Needed for developer portal and integration partner onboarding.
- **Suggested options:** `https://api.openleap.io/docs/srv/apt` (suggested pattern)
- **Owner:** team-srv

---

## Q-APT-009: Message Broker Type

- **Question:** Is RabbitMQ the confirmed message broker for the SRV suite, or is Kafka or another broker used?
- **Why it matters:** Affects queue naming conventions, consumer group configuration, dead-letter queue setup, and outbox publisher implementation.
- **Suggested options:**
  - A) RabbitMQ — assumed based on typical OpenLeap platform pattern
  - B) Apache Kafka — topic/partition-based; changes consumer group semantics
- **Owner:** team-srv

---

## Q-APT-010: Offering Deactivation Impact on Existing Appointments

- **Question:** When `srv.cat` deactivates a service offering, what should happen to existing PROPOSED/RESERVED appointments referencing that offering?
- **Why it matters:** Determines the business logic in `CatalogEventHandler` (§7.3). Auto-cancellation is aggressive but prevents invalid bookings from progressing; flagging for review is safer but requires manual intervention.
- **Suggested options:**
  - A) Auto-cancel PROPOSED and RESERVED appointments — immediate but disruptive
  - B) Flag for review — set a `requiresReview` flag; back office decides
  - C) Allow to complete — no action; CONFIRMED appointments proceed; only block new bookings
- **Owner:** TBD

---

## Q-APT-011: Entitlement Integration Pattern

- **Question:** How should `srv.apt` integrate with `srv.ent` for eligibility checks during booking? Should it call `srv.ent` synchronously during the booking flow, or maintain a local entitlement state updated via events?
- **Why it matters:** Synchronous calls to `srv.ent` during booking confirmation create a critical runtime dependency (if `srv.ent` is down, booking fails). Event-sourced local state is more resilient but complex to implement.
- **Suggested options:**
  - A) Synchronous API call to `srv.ent` at confirm time — simpler; creates hard dependency
  - B) Local entitlement replica updated by `srv.ent.*` events — higher availability; eventual consistency
  - C) Skip eligibility check at booking time; validate at session execution in `srv.ses`
- **Owner:** TBD

---

## Q-APT-012: Appointment Completed Event

- **Question:** Should `srv.apt` emit a `srv.apt.appointment.completed` integration event when an appointment transitions to COMPLETED (triggered by `srv.ses.session.completed`)?
- **Why it matters:** Some consumers (e.g., `srv.cas`, reporting) may want to react to appointment completion specifically, without having to subscribe to `srv.ses` events. A separate event keeps the `srv.apt` event stream self-contained.
- **Suggested options:**
  - A) Yes — emit `srv.apt.appointment.completed` when COMPLETED transition occurs
  - B) No — consumers that need this fact should subscribe to `srv.ses.session.completed` directly
- **Owner:** TBD

---

## Q-APT-013: HIPAA Applicability

- **Question:** Is HIPAA applicable to deployments of `srv.apt` where appointments relate to healthcare services (e.g., physiotherapy, dental, medical consultations)?
- **Why it matters:** HIPAA requires additional technical safeguards (encryption at rest and in transit, audit logging, Business Associate Agreements). This would affect the data classification of `srv.apt` data and require specific compliance controls.
- **Suggested options:**
  - A) HIPAA applicable — designate `srv.apt` as a potential covered entity component; implement BAA requirements
  - B) HIPAA not applicable for current target markets — revisit when healthcare customers onboard
- **Owner:** TBD (Legal/Compliance)

---

## Q-APT-014: Projected Booking Volumes

- **Question:** What are the projected booking volumes for initial target customers (appointments per day, month, tenant)?
- **Why it matters:** Required for §10.3 Scalability capacity planning — determines when read replicas, caching, or horizontal scaling are needed at launch vs. deferred.
- **Suggested options:** Gather from product team based on target customer profiles (driving schools: 50–200 bookings/day per tenant; clinics: 100–500/day; enterprise: 1000+/day)
- **Owner:** Product Team

---

## Q-APT-015: Complete Feature Dependency List

- **Question:** Which product feature IDs beyond F-SRV-002 (Appointment & Booking), F-SRV-004 (Session Execution), and F-SRV-007 (Billing Intent) depend on `srv.apt` endpoints or events?
- **Why it matters:** Required for a complete §11.2 Feature Dependency Register. Missing feature dependencies mean the domain team cannot accurately assess the blast radius of API changes.
- **Suggested options:** Complete after SRV feature catalog (`features/` directory) is fully populated. Check all F-SRV-* feature specs for `srv.apt` endpoint references.
- **Owner:** TBD
