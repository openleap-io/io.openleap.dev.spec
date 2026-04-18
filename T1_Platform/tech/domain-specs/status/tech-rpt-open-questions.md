# Open Questions — tech.rpt

---

## Q-RPT-001: Service Port Assignment

- **Question:** Port `8097` is tentatively assigned in the spec. Is this the confirmed port for `tech-rpt-svc`, or is a different port in use / reserved?
- **Why it matters:** Port assignment affects service discovery configuration, firewall rules, and developer local setup documentation.
- **Suggested options:**
  - **A)** Confirm port `8097` (next sequential after JC at `8096`).
  - **B)** Assign a different port from the tech suite port allocation table.
- **Owner:** TBD (Platform Infrastructure Team)

---

## Q-RPT-002: Scheduled / Recurring Render Jobs

- **Question:** Should `tech-rpt-svc` support a built-in schedule for recurring render jobs (e.g., "render the monthly cost center report every 1st of the month"), or should scheduling remain entirely with the calling domain service?
- **Why it matters:** Without built-in scheduling in RPT, each domain service that needs periodic reports must implement its own trigger mechanism. This creates fragmented operational visibility and duplicated scheduling infrastructure across T2–T4 services.
- **Suggested options:**
  - **A)** RPT delegates to `tech-jc-svc`: if JC adds cron scheduling (per Q-JC-001), RPT can submit recurring JC jobs — no RPT-level change needed.
  - **B)** RPT adds a `RenderSchedule` entity (cron expression + templateCode + dataParams) — RPT owns both scheduling and dispatch.
  - **C)** Keep RPT as pure on-demand; domain services own their own trigger schedules.
  - **D)** Introduce a dedicated `tech-sched-svc` for all platform-level cron scheduling.
- **Owner:** TBD

---

## Q-RPT-003: Deprecation Guard on Active Render Jobs

- **Question:** When `POST /templates/{id}:deprecate` is called, what should happen if the template has PENDING or RUNNING render jobs?
- **Why it matters:** Deprecating a template while jobs are in progress would leave those jobs unable to determine the valid template state. The spec currently states "No PENDING or RUNNING render jobs for this template" as a guard but marks it as Q-RPT-003.
- **Suggested options:**
  - **A)** Hard guard: reject the deprecation request with `422 RPT_TEMPLATE_HAS_ACTIVE_JOBS`. Admin must wait until all in-flight jobs complete.
  - **B)** Soft guard: allow deprecation; PENDING/RUNNING jobs that reference this template continue to completion using the snapshotted `templateVersionId`. This is valid because RenderJobs snapshot the version at submission.
  - **C)** Soft deprecation with warning: allow deprecation; return `202 Accepted` with a warning body listing in-flight job IDs.
- **Owner:** TBD

---

## Q-RPT-004: Maximum Template Versions Per Template

- **Question:** The spec tentatively sets a maximum of 100 `TemplateVersion` entries per `ReportTemplate`. Is this limit appropriate, and should it be configurable per tenant?
- **Why it matters:** A hard limit of 100 may be too low for long-lived templates with frequent updates, or unnecessarily high for most deployments. Without a limit, templates with hundreds of stale versions waste DMS storage.
- **Suggested options:**
  - **A)** Keep 100 as a hard platform limit. Enforce with HTTP `422` and error code `RPT_VERSION_LIMIT_EXCEEDED`.
  - **B)** Make the limit configurable per tenant (stored in platform configuration via `param-cfg-svc`).
  - **C)** Remove the limit; rely on DMS storage quotas and admin cleanup.
  - **D)** Implement auto-archival: when the limit is approached, oldest non-active versions are auto-archived to DMS cold storage tier.
- **Owner:** TBD

---

## Q-RPT-005: JRXML Validation Depth at Upload

- **Question:** When a JRXML/JASPER file is uploaded via `POST /templates/{id}/versions`, should RPT validate the file's structural integrity (e.g., parse the XML, check it's a valid Jasper schema), or should it accept any binary upload and defer validation to the runner at render time?
- **Why it matters:** Eager validation catches corrupt files at upload time (better UX) but adds a Jasper library dependency to RPT. Deferred validation means errors only surface at render time — potentially hours later.
- **Suggested options:**
  - **A)** Structural validation at upload: parse JRXML with Jasper XML schema validation; reject malformed files with `422 RPT_TEMPLATE_INVALID_JRXML`.
  - **B)** Checksum-only validation: accept any binary; compute and store SHA-256 checksum; defer semantic validation to the runner.
  - **C)** Async validation: accept upload immediately (return `201`); dispatch a lightweight JC job to validate the JRXML; flag the version with status `VALIDATING` until confirmed.
- **Owner:** TBD

---

## Q-RPT-006: Process Flow Diagrams (§5.3 / §7.4)

- **Question:** Should §5.3 of the RPT spec include standalone Mermaid sequence diagrams for the render pipeline flow (submit → JC dispatch → runner callback → completion → caller notify), or are suite-level flow diagrams in `_tech_suite.md` sufficient to satisfy the TPL-SVC v1.0.0 requirement?
- **Why it matters:** TPL-SVC v1.0.0 requires §5.3 to contain process flow diagrams. Cross-referencing the suite spec keeps domain specs DRY but risks the suite spec becoming a bottleneck. Duplicating diagrams risks drift.
- **Suggested options:**
  - **A)** Add full standalone sequence diagrams in §5.3 and §7.4: render submit + dispatch, running callback, completion callback, failure callback, and timeout detection.
  - **B)** Reference `_tech_suite.md` with explicit cross-links; mark §5.3 as "see suite spec §4.x".
  - **C)** Add abbreviated happy-path diagram in §5.3 with a cross-link to the suite spec for full detail.
- **Owner:** TBD

---

## Q-RPT-007: OpenAPI Specification Population

- **Question:** The companion file `T1_Platform/tech/contracts/http/tech/rpt/openapi.yaml` is currently a 1-line stub. When and by whom will it be populated with the full OpenAPI 3.1 contract derived from §6 of this spec?
- **Why it matters:** The OpenAPI spec is the machine-readable contract consumed by BFF code generation, API gateway configuration, and integration test scaffolding. Without it, BFF development for F-TECH-002-xx cannot proceed with contract-first tooling.
- **Suggested options:**
  - **A)** Architecture team populates the OpenAPI spec in parallel with this spec upgrade, derived from §6.
  - **B)** The OpenAPI spec is generated as an output artifact during the domain service implementation phase (`io.openleap.tech.rpt` Spring Boot service).
  - **C)** An AI-assisted generation run derives the OpenAPI 3.1 YAML from §6 of this spec.
- **Owner:** Platform Infrastructure Team

---

## Q-RPT-008: PENDING RenderJob Timeout Monitoring

- **Question:** If a `RenderJob` remains in `PENDING` state for an extended period (e.g., JC accepted the job but the runner never picked it up, or the JC callback was never received by RPT), who detects and transitions the stale job to `FAILED`?
- **Why it matters:** Without a timeout mechanism, stuck PENDING jobs accumulate indefinitely, causing incorrect operational metrics and leaving callers waiting for completion notifications that never arrive.
- **Suggested options:**
  - **A)** RPT owns a scheduled background task (e.g., `@Scheduled`) that polls `rpt_render_jobs WHERE status = 'PENDING' AND created_at < now() - interval 'N hours'` and transitions them to FAILED with error code `RPT_JOB_TIMEOUT`.
  - **B)** JC owns the timeout: JC marks its job as FAILED after `timeoutSeconds`; RPT receives the JC failure callback and transitions accordingly. (Preferred if JC timeout is reliably configured for the `pdf-render` job type.)
  - **C)** Both: JC primary timeout + RPT backstop at 2× JC timeout for belt-and-suspenders reliability.
- **Owner:** TBD

---

## Q-RPT-009: Extension Management API Endpoints

- **Question:** Should `tech-rpt-svc` document its own extension management REST endpoints in §12.7 (e.g., `POST /api/tech/rpt/v1/extensions/custom-field-definitions`), or should these be fully delegated to and documented in the `core-extension` module documentation in `io.openleap.dev.guidelines`?
- **Why it matters:** TPL-SVC v1.0.0 §12.7 expects "extension management endpoints" to be specified. Without documenting them here, BFF developers cannot determine the correct API path without consulting the implementation guidelines.
- **Suggested options:**
  - **A)** Document RPT-specific extension management endpoints in §12.7 (e.g., list/create custom field definitions scoped to RPT aggregates).
  - **B)** Cross-reference `core-extension` module documentation; mark §12.7 as "see `io.openleap.dev.guidelines` ADR-067".
  - **C)** Document the standard `core-extension` endpoint pattern here with RPT-specific examples.
- **Owner:** TBD

---

## Q-RPT-010: Credential Encryption Strategy

- **Question:** The spec states that `dataSourceConfig.password` and `dataSourceConfig.authHeader` are "encrypted at rest using the platform's secrets management facility". In v1.0.0, is this a dedicated KMS (e.g., HashiCorp Vault), a service-layer AES-256 encryption with a key stored in environment config, or a different approach?
- **Why it matters:** The encryption strategy affects the security review outcome, key rotation procedure, and compliance evidence for GDPR Art. 32 (security of processing). The implementation choice must be explicit before the service goes to review.
- **Suggested options:**
  - **A)** Service-layer AES-256 with key in environment config (`ENCRYPTION_KEY` env var). Simple for v1.0.0; key rotation requires re-encryption of all rows.
  - **B)** HashiCorp Vault Transit encryption: service calls Vault to encrypt/decrypt. Stronger; key never leaves Vault; rotation is transparent.
  - **C)** PostgreSQL `pgcrypto` extension: database-level symmetric encryption. Tight coupling between DB and application layer.
  - **D)** Defer to platform-wide secrets management decision (cross-cutting concern not yet decided).
- **Owner:** Platform Security Team

---

## Q-RPT-011: template.deprecated Event

- **Question:** The spec does not publish a `tech.rpt.template.deprecated` event when a template is deprecated. Should such an event be added to notify downstream consumers (e.g., domain services that have cached the template's `code` for render submissions)?
- **Why it matters:** Without a deprecation event, callers discover the deprecated state only when their next `POST /renders` returns a `422 RPT_TEMPLATE_NOT_ACTIVE` error. An event would allow proactive notification and graceful migration to a replacement template.
- **Suggested options:**
  - **A)** Add `tech.rpt.template.deprecated` event; update §7.2 and `contracts/events/tech/rpt/` with the schema.
  - **B)** No event needed; callers should use `GET /templates` polling or subscribe to the general audit stream.
  - **C)** Add the event but defer schema definition to the next spec revision.
- **Owner:** TBD
