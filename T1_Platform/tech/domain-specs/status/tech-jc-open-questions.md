# Open Questions — tech.jc

## Q-JC-001: Cron / Scheduled Job Triggering

- **Question:** Should `tech-jc-svc` include a built-in cron scheduling capability (i.e., define a schedule that automatically submits a job on a cron expression), or should this remain out of scope and be delegated to callers (e.g., Spring `@Scheduled` in each domain service)?
- **Why it matters:** Without cron support in JC, each domain service that needs regularly scheduled jobs must implement its own trigger mechanism. This creates monitoring fragmentation, inconsistent retry behaviour, and duplicated scheduling infrastructure across T2–T4 services. Conversely, adding scheduling to JC expands its scope and complexity beyond pure dispatch.
- **Suggested options:**
  - **A)** Add a `jc_job_schedule` entity (cron expression + jobTypeSlug + fieldValues) to JC — JC owns both scheduling and dispatch.
  - **B)** Keep JC as pure dispatch; domain services own their own trigger schedules via Spring `@Scheduled` or Quartz.
  - **C)** Introduce a dedicated scheduling service (`tech-sched-svc`) that translates cron expressions into JC job submissions.
  - **D)** Defer to Phase 3 roadmap item (see `_tech_suite.md §10`).
- **Owner:** TBD

---

## Q-JC-002: Runner Reachability Check at Registration

- **Question:** Should JC perform a synchronous health-check HTTP call to the runner's `callbackUrl` or `healthCheckUrl` during `POST /runners`, and fail registration with `422` if the endpoint is unreachable?
- **Why it matters:** Eager validation prevents ghost registrations that would silently receive job dispatches and immediately fail. However, synchronous dependency on the runner at registration complicates rolling deployments where the runner may not be fully ready when it self-registers.
- **Suggested options:**
  - **A)** Synchronous health check at registration (fail fast with `JC_RUNNER_CALLBACK_UNREACHABLE`).
  - **B)** Asynchronous health check: accept registration optimistically, mark runner `DEGRADED` if health check fails within 30 seconds of registration.
  - **C)** No automatic check at registration — rely on heartbeat timeout to detect unresponsive runners.
- **Owner:** TBD

---

## Q-JC-003: Re-registration of OFFLINE Runner with Same Slug

- **Question:** If a runner with the same `(tenantId, slug)` previously existed in `OFFLINE` state and attempts to re-register, should JC reactivate the existing record (updating `callbackUrl`, `supportedJobTypeSlugs`, and resetting status to `ACTIVE`) or create a new record?
- **Why it matters:** Reactivating the existing record preserves the association between the runner's identity and historical jobs (for audit/history queries). Creating a new record is simpler but produces orphan OFFLINE records and breaks historical linkage.
- **Suggested options:**
  - **A)** Reactivate existing OFFLINE record: `UPDATE jc_runner SET status='ACTIVE', callback_url=..., updated_at=now()` where `(tenant_id, slug)` matches an OFFLINE row.
  - **B)** Create new record; OFFLINE record is left as-is (orphaned).
  - **C)** Return `409 Conflict` with a message instructing the operator to explicitly delete the OFFLINE record before re-registering.
- **Owner:** TBD

---

## Q-JC-004: Process Flow Diagrams (§5.3)

- **Question:** Should §5.3 of the JC spec include standalone Mermaid sequence diagrams for each key workflow (dispatch, status callback, timeout detection, retry), or are the suite-level diagrams in `_tech_suite.md §4.2 Flow 3` sufficient to satisfy the template requirement?
- **Why it matters:** TPL-SVC v1.0.0 requires §5.3 to contain process flow diagrams. Cross-referencing the suite spec keeps the domain spec DRY but risks the suite spec becoming a bottleneck. Duplicating diagrams here risks drift.
- **Suggested options:**
  - **A)** Add full standalone sequence diagrams in §5.3 for: job submission + dispatch, runner callback (running/completed/failed), timeout detection, and job retry.
  - **B)** Reference `_tech_suite.md §4.2 Flow 3` with explicit cross-links; mark §5.3 as "see suite spec".
  - **C)** Add abbreviated versions in §5.3 (happy path only) with a cross-link to the suite spec for full detail.
- **Owner:** TBD

---

## Q-JC-005: OpenAPI Specification Population

- **Question:** The companion file `T1_Platform/tech/contracts/http/tech/jc/openapi.yaml` is currently a 1-line stub. When and by whom will it be populated with the full OpenAPI 3.1 contract derived from this spec?
- **Why it matters:** The OpenAPI spec is the machine-readable contract consumed by BFF code generation, API gateway configuration, and integration test scaffolding. Without it, BFF development for F-TECH-003-xx cannot proceed with contract-first tooling.
- **Suggested options:**
  - **A)** Architecture team populates the OpenAPI spec in parallel with this spec upgrade, derived from §6.
  - **B)** The OpenAPI spec is generated as an output artifact during the domain service implementation phase (`io.openleap.tech.jc` Spring Boot service).
  - **C)** An AI-assisted generation run derives the OpenAPI 3.1 YAML from §6 of this spec.
- **Owner:** Platform Infrastructure Team

---

## Q-JC-006: Extension Management API Endpoints (§12.7)

- **Question:** Should the custom field registration, extension rule registration, and aggregate hook management endpoints provided by the `core-extension` module (`io.openleap.starter`) be explicitly documented in §12.7 of this spec, or is a cross-reference to `io.openleap.dev.guidelines` (ADR-067 / ADR-011) sufficient?
- **Why it matters:** The spec self-containedness principle (§ Spec Guidelines Non-Negotiables) states that the spec should not rely on implicit external context. If extension API endpoints are not documented here, BFF authors and integration engineers must chase down the module documentation separately.
- **Suggested options:**
  - **A)** Document the full extension API endpoint set in §12.7 (custom field CRUD, rule registration, hook registration) with request/response examples.
  - **B)** Add a short summary table in §12.7 with endpoint patterns and link to `io.openleap.dev.guidelines ADR-067` for full detail.
  - **C)** Leave §12.7 as a stub — accept the compliance gap — until the `core-extension` module's API surface is stable.
- **Owner:** TBD

---

## Q-JC-007: SAP Background Processing Migration Strategy

- **Question:** When customers migrate from SAP ERP to OpenLeap, what is the recommended strategy for migrating existing SAP background jobs (SM36/SM37, tables `TBTCO`/`TBTCS`) to JC job types and runners?
- **Why it matters:** SAP background jobs often represent critical business automation (month-end closing, batch postings, payroll runs). A clear migration strategy is needed before go-live planning. The mapping complexity varies: simple ABAP reports may map cleanly to a new runner; complex transaction chains may require saga redesign.
- **Suggested options:**
  - **A)** Full reimplementation: each SAP background job is analysed and reimplemented as a native JC runner (recommended for greenfield deployments).
  - **B)** SAP BC-XOM adapter runner: a bridge runner that wraps SAP RFC/ABAP job invocations, allowing legacy jobs to be submitted via JC without reimplementation (recommended for phased migrations).
  - **C)** Hybrid: migrate simple, stateless ABAP batch reports as native runners; use the BC-XOM adapter for complex cross-module jobs during the transition period.
- **Owner:** TBD (Migration Architecture Team)
