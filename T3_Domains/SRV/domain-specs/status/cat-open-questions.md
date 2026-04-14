# Open Questions - srv.cat (Service Catalog)

## Q-CAT-001: Effective-Dated Offerings

- **Question:** Do we need versioned/effective-dated offerings for long-running entitlements and historical billing?
- **Why it matters:** Affects data model (needs `valid_from`/`valid_to` or a `cat_offering_version` table) and API design; code generators need to know if temporal versioning is required. Without it, billing reconciliation for historical periods may be inaccurate if an offering's attributes change over time.
- **Suggested options:**
  - A) No versioning (simpler MVP) — acceptable if offerings rarely change substantively
  - B) Effective-dated rows with `valid_from`/`valid_to` columns on `cat_service_offering`
  - C) Separate `cat_offering_version` table (full audit history)
- **Owner:** TBD

## Q-CAT-002: Documentation Template References

- **Question:** Where do documentation templates live (`t1.rpt`/`t1.dms`) and how does `srv.cat` reference them?
- **Why it matters:** Affects the Requirement model — if proof-of-service documents are attached to requirements, a `documentation_template_ref` field or a new RequirementType value may be needed.
- **Suggested options:**
  - A) Template reference stored in `Requirement.refId` with `type=CUSTOM` and `description` documenting the template
  - B) Separate configuration entity linking offering to document templates
- **Owner:** TBD

## Q-CAT-003: Skill Catalog Ownership

- **Question:** Where do skills live concretely — HR module or a shared T1/T2 skill catalog?
- **Why it matters:** Determines whether `srv.cat` must consume `hr.skill.*` events for requirement reference validation and whether the `cat_requirement.ref_id` for SKILL type is HR-owned or platform-owned.
- **Suggested options:**
  - A) HR-owned skill catalog with event sync to `srv.cat` (`hr.skill.skill.created/retired`)
  - B) Shared T2 catalog with direct REST lookup by `srv.cat` at requirement creation
- **Owner:** TBD

## Q-CAT-004: Database Technology Confirmation

- **Question:** Confirm target database technology (PostgreSQL assumed per ADR-016).
- **Why it matters:** DDL generation and ORM configuration depends on confirmed RDBMS. INTERVAL type for durations is PostgreSQL-specific.
- **Suggested options:** PostgreSQL (assumed)
- **Owner:** TBD

## Q-CAT-005: Port Assignment

- **Question:** Confirm port assignment for `srv-cat-svc`.
- **Why it matters:** Deployment configuration and local dev environment port registry.
- **Suggested options:** To be assigned by platform/infra team.
- **Owner:** TBD

## Q-CAT-006: Repository URI

- **Question:** Confirm Git repository name/URI for `srv-cat-svc`.
- **Why it matters:** Code generation, CI/CD pipeline configuration, cross-reference in landscaping.
- **Suggested options:** `io.openleap.srv.cat` (assumed naming convention)
- **Owner:** TBD

## Q-CAT-007: Requirement Uniqueness Enforcement

- **Question:** Should the combination of `(type, refId)` within an offering's requirement list be enforced as a hard constraint (409 Conflict) or a soft warning?
- **Why it matters:** Hard constraint prevents duplicate requirements but may block bulk import tools. Soft warning is more user-friendly but may allow inconsistent data.
- **Suggested options:**
  - A) Hard constraint — return 409 Conflict; cleaner data model
  - B) Warning — return 201 with `warnings: [...]` array; more flexible for bulk operations
- **Owner:** TBD

## Q-CAT-008: Message Broker Confirmation

- **Question:** Confirm message broker technology (RabbitMQ assumed per SRV suite pattern).
- **Why it matters:** Event infrastructure configuration, queue naming conventions, and consumer setup all depend on confirmed broker.
- **Suggested options:** RabbitMQ (assumed per `_srv_suite.md`)
- **Owner:** TBD

## Q-CAT-009: Retirement Block on Active Bookings

- **Question:** Should retirement be blocked or warned when active future bookings reference the offering?
- **Why it matters:** Determines the retirement guard condition in UC-003. Blocking requires a cross-service check against `srv.apt` (coupling); warning keeps `srv.cat` independent; no-check relies on consumers to handle via event.
- **Suggested options:**
  - A) Block retirement if active future bookings exist (requires sync call to `srv.apt`)
  - B) Warn only (return 200 with `warnings` array)
  - C) No check — consumers handle booking invalidation via the `srv.cat.service.retired` event
- **Owner:** TBD
