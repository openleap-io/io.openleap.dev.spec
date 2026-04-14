# Open Questions - srv.res (Resource Scheduling)

## Q-RES-001: Port Assignment

- **Question:** What port should `srv-res-svc` run on?
- **Why it matters:** Required for service registry, local development setup, Docker Compose configuration, and Kubernetes service definitions.
- **Suggested options:**
  - A) Auto-assigned by infrastructure (Kubernetes service mesh)
  - B) Fixed port within a reserved range for the SRV suite
- **Owner:** TBD

---

## Q-RES-002: Repository URI

- **Question:** What is the repository URI for `srv-res-svc`?
- **Why it matters:** Required for `metadata.repository` in the spec, CI/CD pipeline configuration, and OpenAPI docs URL.
- **Suggested options:**
  - A) `github.com/openleap/io.openleap.srv.res`
  - B) Follows suite naming: `github.com/openleap/srv-res-service`
- **Owner:** TBD

---

## Q-RES-003: Skill Catalog Ownership

- **Question:** Where do "skills" (qualifications) live concretely — HR master data, a shared T2 catalog, or remain as free-text in `srv.res`?
- **Why it matters:** Affects validation of `skillTags[]` — if a catalog exists, tags can be validated against it. Also affects eligibility matching accuracy and potential for skill-based routing in `srv.apt`.
- **Suggested options:**
  - A) HR-owned skill catalog (canonical source; `srv.res` subscribes to changes)
  - B) Shared T2 catalog (cross-suite skill vocabulary)
  - C) Free-text (current approach; no validation; fast iteration)
- **Owner:** TBD

---

## Q-RES-004: FAC Event Routing Keys

- **Question:** What are the exact routing key patterns for room/space lifecycle events published by `fac-*-svc`?
- **Why it matters:** Required for `srv.res` queue binding configuration (§7.3). Using a wildcard `fac.#` binding may be too broad and introduce unintended event consumption.
- **Suggested options:**
  - A) `fac.room.created`, `fac.room.updated`, `fac.room.decommissioned`
  - B) `fac.space.created`, `fac.space.updated`, `fac.space.retired`
  - C) Align with FAC domain spec once available
- **Owner:** TBD — depends on FAC domain spec

---

## Q-RES-005: Deactivation Blocking Policy

- **Question:** Should deactivation of a resource (transition to `INACTIVE`) be blocked if the resource has future confirmed appointments?
- **Why it matters:** Affects appointment integrity. If deactivation is allowed without blocking, `srv-apt-svc` may have confirmed bookings referencing an inactive resource. Notification or re-assignment workflow needed.
- **Suggested options:**
  - A) Hard block: deactivation rejected if future confirmed appointments exist
  - B) Soft warning: deactivation allowed with a `requiresForce=true` flag; caller must explicitly acknowledge
  - C) Allow freely: `srv-apt-svc` handles resource status changes via `resource.updated` event
- **Owner:** TBD — requires input from Product Owner (scheduling business rules)

---

## Q-RES-006: Availability Window Constraints

- **Question:** Should there be a maximum duration per availability window and/or a soft cap on the number of windows per resource?
- **Why it matters:** A resource with 10,000+ windows (e.g., one per hour for 5 years) would produce expensive overlap checks and large query results. A cap improves predictability.
- **Suggested options:**
  - A) No hard cap (current approach); monitor via query performance metrics
  - B) Maximum window duration: 24 hours (prevents year-long "always available" windows)
  - C) Soft cap: 1,000 windows per resource per tenant (enforced with a warning, not an error)
- **Owner:** TBD — Architecture Team

---

## Q-RES-007: Fallback When srv-cat-svc Unavailable

- **Question:** If `srv-cat-svc` is unreachable during an availability query with `serviceOfferingId`, should the system fall back to unfiltered results, return an error, or serve cached requirements?
- **Why it matters:** Affects booking UX during catalog service outage. Unfiltered fallback may return ineligible resources; error mode blocks booking entirely.
- **Suggested options:**
  - A) Fall back to unfiltered: return all resources regardless of skill match (degrade gracefully)
  - B) Return 503: booking is blocked until catalog service recovers
  - C) Serve cached requirements: short-lived in-memory cache of last-known requirements
- **Owner:** TBD — Product Owner + SRE

---

## Q-RES-008: Window Deletion Blocking Policy

- **Question:** Should deletion of an availability window be blocked if confirmed appointments exist within the window's time range?
- **Why it matters:** Deleting a window that covers existing bookings could cause data integrity issues in `srv-apt-svc` (appointments now reference a resource with no availability window in that period).
- **Suggested options:**
  - A) Hard block: deletion rejected if appointments are confirmed in the window's range
  - B) Soft block: deletion allowed for `SRV_RES_ADMIN` role with explicit acknowledgement
- **Owner:** TBD — requires alignment with `srv-apt-svc` team

---

## Q-RES-009: Assignment Confirmation Event Ownership

- **Question:** Who is responsible for publishing `srv.res.assignment.confirmed` — `srv-res-svc` (on receiving an assignment command) or `srv-apt-svc` (which confirms the booking)?
- **Why it matters:** Event ownership determines which service's outbox table contains the event, and which team is responsible for reliability. If `srv-apt-svc` publishes this, the routing key should arguably be `srv.apt.*` not `srv.res.*`.
- **Suggested options:**
  - A) `srv-res-svc` publishes: `srv-apt-svc` calls a `POST /resources/{id}/assignments` endpoint; `srv-res-svc` publishes `srv.res.assignment.confirmed`
  - B) `srv-apt-svc` publishes: on booking confirmation, `srv-apt-svc` emits `srv.apt.appointment.confirmed` which implicitly carries resource assignment context
  - C) Dual event: `srv-apt-svc` publishes its booking event; `srv-res-svc` reacts and publishes its own assignment confirmation
- **Owner:** TBD — Architecture Decision needed; involves `srv-apt-svc` team

---

## Q-RES-010: Archived Resource Retention Period

- **Question:** What is the retention period for ARCHIVED scheduling resources before they may be physically purged?
- **Why it matters:** Affects storage planning and GDPR compliance. Resources that are physically deleted cannot be referenced in historical session/appointment records.
- **Suggested options:**
  - A) 7 years (conservative; aligns with common legal retention requirements)
  - B) 3 years (moderate; aligns with typical employment audit requirements)
  - C) Configurable per tenant (flexible; higher complexity)
  - D) Never purged (append-only; simplest; highest storage cost)
- **Owner:** TBD — Legal / Compliance Team

---

## Q-RES-011: Skill Tags Sensitivity Classification

- **Question:** Should `skillTags[]` be classified as "Sensitive" rather than "Internal" when they reveal medical specializations (e.g., `"oncology-nurse"`, `"psychiatric-therapist"`)?
- **Why it matters:** Affects GDPR data classification, access control, and potentially the BFF field-level security policy for custom fields. If Sensitive, additional controls may be required (e.g., role-based field masking).
- **Suggested options:**
  - A) Classify as Sensitive — add BFF field-level masking; require `SRV_RES_ADMIN` to view unmasked tags
  - B) Treat as Internal with access control — `SRV_RES_VIEWER` can see tags; no masking required
  - C) Dynamic classification: products declare sensitivity per deployment (via custom field policy)
- **Owner:** TBD — Privacy Officer / Data Classification Team

---

## Q-RES-012: SRV Feature Catalog IDs

- **Question:** What are the final feature IDs in the SRV suite feature catalog for resource scheduling capabilities?
- **Why it matters:** Required for §11.2 Feature Dependency Register. Placeholder IDs (F-SRV-002, F-SRV-003, F-SRV-005) used in this spec must be verified against the finalized SRV feature catalog.
- **Suggested options:**
  - Align with SRV feature catalog once `_srv_suite.md` §SS6 feature list is finalized.
- **Owner:** TBD — SRV Product Owner

---

## Q-RES-013: Legacy Skill Code Mapping

- **Question:** How should legacy skill/qualification codes from existing scheduling systems be mapped to the new `skillTags[]` vocabulary?
- **Why it matters:** Migration accuracy depends on a clean mapping. Unmapped codes either block migration or result in resources with empty/incorrect skill tags, breaking eligibility matching.
- **Suggested options:**
  - A) Define a formal mapping table (source code → new tag) as part of the migration project
  - B) Direct string conversion (lowercased, normalized legacy codes become skill tags)
  - C) Manual review: migration team reviews each unique legacy code and assigns new tags
- **Owner:** TBD — Migration Project Lead
