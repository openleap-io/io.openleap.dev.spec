# Open Questions — param.cfg

**Spec file:** `T1_Platform/param/domain-specs/param_cfg-spec.md`
**Last updated:** 2026-04-03

---

## Q-CFG-001: Tenant Deletion Choreography

- **Question:** Should `param-cfg-svc` subscribe to `iam.tenant.tenant.deleted` events and archive or delete all TENANT-scoped config entries for the deleted tenant?
- **Why it matters:** Without cleanup, orphaned TENANT entries accumulate indefinitely in the database. GDPR Article 17 (right to erasure) may require prompt deletion of personal data associated with a deleted tenant. The `changedByDisplayName` field in `cfg_audit_log` is PII and may require anonymisation.
- **Suggested options:**
  - **Option A (Choreography):** Subscribe to `iam.tenant.tenant.deleted`; soft-delete all TENANT entries for the tenant; produce audit log entries with `changeReason = "Tenant deleted — automated cleanup"`.
  - **Option B (Scheduled Job):** A platform-level data archival job runs nightly and removes orphaned TENANT data.
  - **Option C (No Action):** Orphaned TENANT entries are harmless since the tenant no longer exists and cannot be accessed via RLS; retain for historical analysis.
- **Owner:** TBD
- **Blocking for:** GDPR compliance review; IAM-PARAM integration spec

---

## Q-CFG-002: Encryption at Rest for PARAMETER Values

- **Question:** Should config values with `configType = PARAMETER` that may contain secrets (API keys, connection strings, tokens) be encrypted at rest in PostgreSQL?
- **Why it matters:** Storing plaintext secrets in a config table is a security risk. A database breach or backup compromise could expose platform-wide secrets. Many enterprise deployments require secrets to be stored in a dedicated secrets manager.
- **Suggested options:**
  - **Option A (Secrets Manager):** Explicitly document that `PARAMETER` type MUST NOT store secrets; add a `configType = SECRET` backed by HashiCorp Vault or AWS Secrets Manager with a reference key only in the database.
  - **Option B (Column Encryption):** Encrypt the `value` column using PostgreSQL `pgcrypto` for entries with a new `configType = SECRET`; restrict RBAC.
  - **Option C (Application-Level Encryption):** Encrypt sensitive `value` contents at the application layer before persistence, using a platform key managed by the OpenLeap key management service.
  - **Option D (Status Quo):** Document that `PARAMETER` type is not intended for secrets; enforce this through platform governance and runbook guidance rather than technical controls.
- **Owner:** TBD (Security Architect + Platform Team)
- **Blocking for:** Security classification finalisation in §9.1

---

## Q-CFG-003: Extension Management API Ownership

- **Question:** Should the extension management endpoints (custom field registration, hook registration) be owned by `param-cfg-svc` itself, or by a central `core-extension-svc`?
- **Why it matters:** If each domain service owns its own extension management endpoints, this creates ~40 identical endpoint patterns across the platform (one per service). A central registry reduces surface area but introduces a cross-service dependency at service startup.
- **Suggested options:**
  - **Option A (Per-Service):** `param-cfg-svc` owns `/api/param/cfg/v1/extensions/**`. Simple; no new service dependency.
  - **Option B (Central Registry):** A `core-extension-svc` owns all extension metadata. Domain services call it at startup to register their extension points. Removes endpoint proliferation.
  - **Option C (Hybrid):** Extension point declarations are static (in spec, not runtime-registered). Only active product extensions are registered via a central API.
- **Owner:** TBD (Platform Architecture)
- **Blocking for:** §12.7 Extension API Endpoints finalisation; ADR-067 implementation guide

---

## Q-CFG-004: TENANT Entry Resolution Priority When Overlapping GLOBAL Key

- **Question:** If a TENANT entry has the same `key` as a GLOBAL entry, which takes precedence when a consumer resolves `(scope=TENANT, tenantId=X, key=platform.maintenance-mode)` vs `(scope=GLOBAL, key=platform.maintenance-mode)`?
- **Why it matters:** Resolution priority affects feature flag behaviour for tenants with overrides. If TENANT always wins, a tenant could bypass a global maintenance window. If GLOBAL always wins, TENANT overrides lose utility.
- **Suggested options:**
  - **Option A (TENANT overrides GLOBAL):** Standard override semantics. Tenant can always override a GLOBAL value for their context. Exception: well-known keys (BR-CFG-001) cannot be deleted, but a TENANT entry with the same key would shadow them.
  - **Option B (Consumer Responsibility):** The service returns each entry by its exact `(scope, key)` lookup. Resolution priority is defined in BFF guidelines, not in this service.
  - **Option C (GLOBAL wins for well-known):** GLOBAL well-known keys always take precedence; TENANT entries cannot share the same key as a GLOBAL well-known key (enforced by BR-CFG-001 extension).
- **Owner:** TBD (BFF Architecture + Platform Team)
- **Blocking for:** BFF feature-gate resolution documentation; BR-CFG-001 potential extension

---

## Q-CFG-005: Port Assignment Confirmation

- **Question:** Is port `8102` definitively assigned to `param-cfg-svc` in the platform service port registry?
- **Why it matters:** Port conflicts between param suite services (param-ref-svc, param-i18n-svc, param-cfg-svc, param-si-svc) would cause local development failures. The `openapi.yaml` stub specifies port 8102 but this has not been cross-referenced against a central port registry.
- **Suggested options:**
  - Confirm port 8102 in a central port registry (e.g., `landscape/` or `_param_suite.md` §3.1 Service Catalog).
  - Update `_param_suite.md` §3.1 to include the local dev port for each service.
- **Owner:** TBD (team-param)
- **Blocking for:** Local development documentation; `_param_suite.md` §3.1 completion

---

# Open Questions — param.i18n

**Spec file:** `T1_Platform/param/domain-specs/param_i18n-spec.md`
**Last updated:** 2026-04-03

---

## Q-I18N-001: MESSAGE Namespace Validation Against Feature Registry

- **Question:** Should the i18n service validate that a MESSAGE namespace ID (`F-PARAM-002-01`) corresponds to a registered feature leaf in a feature catalog, or is BCP-47 format validation sufficient?
- **Why it matters:** Without feature registry validation, orphaned namespaces accumulate when features are removed. With it, a feature registry service dependency is introduced, creating a coupling that could complicate deployment ordering.
- **Suggested options:**
  - **Option A (Format only, current spec):** Validate `namespaceId` matches `^F-[A-Z]+-[0-9]+-[0-9]+$` pattern. Governance enforced via platform processes.
  - **Option B (Async validation):** Accept the registration; queue an async cross-check against the UVL feature catalog; flag the namespace as `VALIDATION_PENDING` until confirmed.
  - **Option C (Accept all):** No validation — orphan cleanup handled by a scheduled reconciliation job.
- **Owner:** TBD
- **Blocking for:** BR-I18N-007 enforcement level; namespace registration API error contract

---

## Q-I18N-002: Circuit-Breaker Behaviour for param-ref-svc During CATALOG Namespace Registration

- **Question:** When `param-ref-svc` is unavailable during CATALOG namespace registration (BR-I18N-008), should the service (A) return 503 and reject the registration, (B) allow the registration with a `validationPending` flag and validate lazily, or (C) skip validation entirely?
- **Why it matters:** CI/CD pipelines may seed namespaces before `param-ref-svc` is reachable (e.g., cold-start deployment ordering). Blocking on unavailability would break CI/CD pipelines. Allowing with lazy validation could create namespaces for non-existent catalogs.
- **Suggested options:**
  - **Option A (Strict — return 503):** Simplest logic; breaks pipelines if param-ref-svc starts later.
  - **Option B (Lazy validation — preferred):** Register with `status = VALIDATION_PENDING`; background job validates once param-ref-svc is reachable; transitions to `ACTIVE` on success.
  - **Option C (Skip validation entirely):** Rely on governance — namespace teams are responsible for catalog registration discipline.
- **Owner:** TBD
- **Blocking for:** Namespace registration API error contract (§6.2.1); circuit-breaker configuration

---

## Q-I18N-003: Auto-Registration via param-ref-svc Catalog Events

- **Question:** Should `param-i18n-svc` consume a `param.ref.catalog.created` event from `param-ref-svc` to auto-register CATALOG namespaces and trigger initial seed pipelines, or is namespace registration a manual/CI-driven step?
- **Why it matters:** Auto-registration would reduce manual setup for new catalogs and ensure every catalog has a corresponding translation namespace. However, it introduces coupling between ref and i18n domains, and requires `param-ref-svc` to publish catalog lifecycle events (which it may not currently do).
- **Suggested options:**
  - **Option A (Manual CI/CD, current approach):** Suite teams register namespaces and seed translations as part of their CI/CD pipeline. No inter-service event coupling.
  - **Option B (Auto-register via event):** Subscribe to `param.ref.catalog.created`; auto-register namespace; notify ownerSuite to seed translations.
- **Owner:** TBD
- **Blocking for:** §7.3 Consumed Events (currently empty); param-ref-svc event catalog

---

## Q-I18N-004: Product-Specific Metadata on Translation Entries

- **Question:** If a product integration needs metadata on translation entries (e.g., source-system origin, machine-translation confidence score, style guide compliance flag), should this be a `custom_fields JSONB` column on `i18n_translation`, or a product-owned extension table?
- **Why it matters:** A JSONB column is simpler to implement but pollutes the shared translation store with product-specific concerns. A product-owned extension table is cleaner architecturally but adds JOIN complexity and deployment coordination.
- **Suggested options:**
  - **Option A (No JSONB column — current DEC-I18N-004):** Product uses a separate extension table with FK to `i18n_translation.id`.
  - **Option B (Add `custom_fields JSONB NOT NULL DEFAULT '{}'`):** Consistent with ADR-067 extensibility pattern used by other aggregates. GIN index required. Slightly violates DEC-I18N-004 rationale.
- **Owner:** TBD
- **Blocking for:** §8.3 final `i18n_translation` table definition; ADR-067 consistency review

---

## Q-I18N-005: Port Assignment Confirmation for param-i18n-svc

- **Question:** Is port `8101` definitively assigned to `param-i18n-svc` in the platform service port registry? The `openapi.yaml` stub specifies `http://localhost:8101` but this has not been confirmed against a central port registry.
- **Why it matters:** Port conflicts between param suite services (param-i18n-svc on 8101, param-cfg-svc on 8102, param-ref-svc and param-si-svc on unknown ports) would break local development environments.
- **Suggested options:**
  - Confirm port 8101 and document in `_param_suite.md` §3 service landscape table.
  - Assign remaining param suite ports and publish in `landscape/` or `_param_suite.md`.
- **Owner:** TBD (team-param)
- **Blocking for:** Local development documentation; `_param_suite.md` §3 completion (see also Q-CFG-005 for related cfg-svc port)

---

# Open Questions — param.ref

**Spec file:** `T1_Platform/param/domain-specs/param_ref-spec.md`
**Last updated:** 2026-04-03

---

## Q-REF-001: ownerSuite Validation Registry

- **Question:** How should `ownerSuite` values be validated when creating a DOMAIN-scoped catalog? Is there a platform-managed registry of valid suite short codes, and if so, where does it live?
- **Why it matters:** Without validation, any string can be supplied as `ownerSuite`, leading to catalogs that claim to belong to non-existent suites. Access control for SUITE_ADMIN users (BR-REF-004) depends on knowing the valid suite short code set.
- **Suggested options:**
  - **Option A (Static config):** Maintain a hardcoded list of valid suite short codes in `param-ref-svc` configuration (e.g., `sd`, `fi`, `hr`, `crm`, `srv`, `pps`, `co`, `com`, `fac`, `ops`, `ps`, `bi`). Update on suite addition.
  - **Option B (Platform suite registry):** A central platform registry service (or a `param-ref-svc` internal catalog named `platform.suites`) holds the canonical list. `ownerSuite` is validated against it at catalog creation time.
  - **Option C (Soft validation):** Accept any `ownerSuite` value; emit a warning if it doesn't match a known pattern; governance enforced by platform architecture review.
- **Owner:** TBD (team-param + Architecture Team)
- **Blocking for:** BR-REF-004 enforcement level; access control implementation for SUITE_ADMIN role claims

---

## Q-REF-002: PLATFORM Catalog Multi-Tenancy Model

- **Question:** PLATFORM-scoped catalogs (countries, currencies, languages) are globally applicable and managed by PLATFORM_ADMIN. How are they represented in a multi-tenant deployment? Are they stored once in a system tenant and shared via cross-tenant reads, or are they seeded into every tenant's schema?
- **Why it matters:** If PLATFORM catalogs are stored per-tenant (seeded during provisioning), then each tenant has its own copy and the `tenant_id` on `ref_catalog` is the actual tenant ID. If they are stored in a system schema, the RLS model needs a bypass or a cross-tenant read path that contradicts the standard RLS pattern.
- **Suggested options:**
  - **Option A (Seeded per tenant):** PLATFORM catalogs are seeded into each tenant during provisioning. Simple RLS; each tenant has its own rows. Downside: ISO updates must be applied to all tenants.
  - **Option B (System tenant):** A single `system` tenant owns PLATFORM catalogs. All tenant RLS policies include an OR clause to expose system tenant rows. Efficient; single source of truth for PLATFORM content.
  - **Option C (Separate schema):** PLATFORM catalogs live in a separate `param_ref_global` schema, not RLS-protected. T1 platform-only; tenants access via READ-ONLY view.
- **Owner:** TBD (Architecture Team)
- **Blocking for:** §8.3 `ref_catalog` `tenant_id` semantics; RLS policy definition; §9.2 access control implementation

---

## Q-REF-003: Auto-Namespace Registration via catalog.created Event

- **Question:** Should `param-i18n-svc` consume `param.ref.catalog.created` and automatically register a CATALOG translation namespace for every new catalog, or is namespace registration a manual/CI-driven step?
- **Why it matters:** Auto-registration would reduce manual setup for new catalogs and ensure every catalog has a corresponding translation namespace. However, it introduces coupling between the ref and i18n domains and creates a dependency on event delivery reliability during initial setup. This question also relates to Q-I18N-003 (already raised in param.i18n open questions).
- **Suggested options:**
  - **Option A (Manual CI/CD, current approach):** Suite teams register namespaces and seed translations as part of their CI/CD pipeline. No inter-service event coupling. Explicit and auditable.
  - **Option B (Auto-register via event):** `param-i18n-svc` subscribes to `param.ref.catalog.created`; auto-registers a CATALOG namespace; marks it as `SEEDING_REQUIRED` until content is provided.
  - **Option C (Hybrid):** PLATFORM catalog namespace auto-registered; DOMAIN catalog namespace requires explicit registration by the ownerSuite team.
- **Owner:** TBD (team-param)
- **Blocking for:** §7.2 Known Consumers table for `catalog.created` event; §5.4 cross-domain workflow documentation

---

## Q-REF-004: Tenant Deletion Choreography

- **Question:** Should `param-ref-svc` subscribe to `iam.tenant.tenant.deleted` events and archive or delete all DOMAIN-scoped catalog data for the deleted tenant? What happens to DOMAIN catalogs owned by the deleted tenant?
- **Why it matters:** Without cleanup, orphaned DOMAIN catalog data accumulates. DOMAIN catalogs exist because a suite team for that tenant created them. If the tenant is deleted, those catalogs serve no purpose. GDPR Article 17 may require prompt removal of any PII in `custom_fields` on `ref_catalog_code`.
- **Suggested options:**
  - **Option A (Choreography):** Subscribe to `iam.tenant.tenant.deleted`; deprecate all DOMAIN catalogs for the tenant; soft-delete catalog code `custom_fields` if they contain PII-flagged fields.
  - **Option B (Scheduled cleanup):** A platform-level archival job runs nightly and soft-deletes orphaned tenant data.
  - **Option C (No action):** PLATFORM catalogs survive (correct — they belong to the system). DOMAIN catalogs become inaccessible via RLS after tenant deletion; retain for audit purposes.
- **Owner:** TBD
- **Blocking for:** §7.3 Consumed Events; GDPR compliance review for `custom_fields` on `ref_catalog_code`

---

## Q-REF-005: PLATFORM Catalog tenant_id Assignment

- **Question:** What `tenant_id` UUID is assigned to PLATFORM-scoped catalog rows in the `ref_catalog` and `ref_catalog_code` tables? Is there a well-known system tenant UUID, and if so, where is it defined?
- **Why it matters:** The RLS policy on `ref_catalog` must be able to expose PLATFORM catalog rows to all tenants. The mechanism for this depends on whether PLATFORM rows use a special system `tenant_id` or are stored in a separate schema. This directly affects the §8.3 table design and §9.2 data isolation description. Related to Q-REF-002.
- **Suggested options:**
  - **Option A (Well-known UUID):** A fixed `tenant_id = '00000000-0000-0000-0000-000000000000'` (nil UUID) or a registered system tenant UUID. RLS policy includes `tenant_id = current_tenant_id OR tenant_id = system_tenant_id`.
  - **Option B (Per-tenant seeding):** PLATFORM catalogs are seeded into every tenant row with the tenant's own `tenant_id`. No RLS bypass needed.
  - **Option C (Null tenant_id):** PLATFORM rows have `tenant_id = NULL`. RLS policy includes `tenant_id IS NULL OR tenant_id = current_tenant_id`.
- **Owner:** TBD (Architecture Team)
- **Blocking for:** §8.3 `ref_catalog` table definition (tenant_id nullable?); §9.2 RLS policy; Q-REF-002 resolution

---

## Q-REF-006: Port Assignment Confirmation for param-ref-svc

- **Question:** Is port `8100` definitively assigned to `param-ref-svc` in the platform service port registry? The `openapi.yaml` stub specifies `http://localhost:8100` but this has not been cross-referenced against a central port registry.
- **Why it matters:** Port conflicts between param suite services (param-ref-svc on 8100, param-i18n-svc on 8101, param-cfg-svc on 8102, param-si-svc on unknown port) would cause local development failures. The param suite has 4 services; all 4 ports should be confirmed and documented centrally.
- **Suggested options:**
  - Confirm port 8100 in a central port registry (`landscape/` or `_param_suite.md` §3 service catalog).
  - Update `_param_suite.md` to include the local dev port for all 4 param suite services.
  - Confirm param-si-svc port assignment (currently unknown — see also Q-CFG-005 and Q-I18N-005).
- **Owner:** TBD (team-param)
- **Blocking for:** Local development documentation; `_param_suite.md` §3 completion

---

# Open Questions — param.si

**Spec file:** `T1_Platform/param/domain-specs/param_si-spec.md`
**Last updated:** 2026-04-03

---

## Q-SI-001: UnitType and UnitStatus as Enumerations vs. param-ref Catalog Codes

- **Question:** Should `UnitType` (BASE, DERIVED, CUSTOM) and `UnitStatus` (ACTIVE, DEPRECATED) be managed as internal Java enumerations, or as reference catalog codes in `param-ref-svc`?
- **Why it matters:** If managed as ref catalog codes, i18n translations can be provided for them, and they participate in the standard code validation lifecycle. However, a `param-ref-svc` runtime dependency at startup would make `param-si-svc` non-independent within the PARAM suite (contradicting the §2.4 bounded context map decision).
- **Suggested options:**
  - **Option A (Internal enumerations — current spec default):** `UnitType` and `UnitStatus` are Java `enum` types. No `param-ref` dependency. Labels resolved via a static MESSAGE namespace in `param-i18n-svc`. Consistent with `bc:units-of-measure` independence.
  - **Option B (param-ref catalog codes):** Registered as `param.unit-type` and `param.unit-status` catalogs in `param-ref-svc`. Breaks suite independence.
  - **Option C (Hybrid):** Internal enum for validation; parallel ref catalog for label resolution only.
- **Owner:** TBD (team-param)
- **Blocking for:** §4.4 Reference Data Dependencies; §3.4 Enumeration governance

---

## Q-SI-002: Hard Delete vs. Soft Delete for Custom Units

- **Question:** When `DELETE /api/param/si/v1/units/{id}` is called on a CUSTOM unit, should the system hard-delete the row or soft-delete (set status = DEPRECATED)?
- **Why it matters:** Soft delete preserves referential integrity for domain records that reference the unit's `unitId`. Hard delete is simpler but risks breaking historical records and audit trails.
- **Suggested options:**
  - **Option A (Soft delete only — current spec default):** `DELETE` sets `status = DEPRECATED`. Historical records remain valid.
  - **Option B (Hard delete with reference check):** Delete only if no active references found across domain services.
  - **Option C (Configurable):** Soft delete default; hard delete via `?force=true` admin parameter.
- **Owner:** TBD (team-param + Architecture Team)
- **Blocking for:** UC-SI-005 implementation; §8.3 `si_unit` data retention policy

---

## Q-SI-003: Custom Prefix API Support

- **Question:** Should `param-si-svc` expose a `POST /api/param/si/v1/prefixes` endpoint for Platform Administrators to register custom (non-SI) prefix definitions?
- **Why it matters:** Some industry/regional conventions use non-BIPM decimal prefixes (e.g., "lakh" = 10⁵). Allowing custom prefixes would help such deployments, but adds complexity to symbol resolution (BR-SI-008, BR-SI-009).
- **Suggested options:**
  - **Option A (Immutable seed only — current spec):** Prefixes are read-only SI standard data only.
  - **Option B (Custom prefixes for PLATFORM_ADMIN):** Add CRUD for prefixes with `prefixType = CUSTOM / STANDARD`.
  - **Option C (Future roadmap):** Defer to v2.0; document in §13.2.
- **Owner:** TBD (team-param)
- **Blocking for:** §6 REST API (prefix endpoints); §3.3.2 Prefix aggregate mutability

---

## Q-SI-004: Extension API Endpoint Ownership

- **Question:** Should extension management endpoints (`POST /extensions/units/handlers`) be served by `param-si-svc` directly or by a central `core-extension-svc`? (Same question as Q-CFG-003 — answer MUST be consistent across all domain services.)
- **Why it matters:** Per-service extension endpoints create ~40 identical endpoint patterns platform-wide. A central registry reduces surface area but introduces a startup dependency.
- **Suggested options:**
  - **Option A (Per-service — current spec default):** `param-si-svc` serves `/api/param/si/v1/extensions/**`.
  - **Option B (Central registry):** A `core-extension-svc` (T1_Platform) owns all extension metadata.
  - **Option C (Static declarations only):** Extension points are spec-only; no runtime registration API.
- **Owner:** TBD (Platform Architecture)
- **Blocking for:** §12.7 Extension API Endpoints; ADR-067 alignment

---

## Q-SI-005: SI Unit Seeding Strategy

- **Question:** Should the 7 SI base units, ~30 SI derived units, and 28 SI prefixes be seeded via Liquibase migrations or via an application-layer startup seeder?
- **Why it matters:** Liquibase is versioned and auditable; if a BIPM redefinition occurs, the update must be applied carefully without breaking existing tenant data. A startup seeder is flexible but risks overwriting manual corrections.
- **Suggested options:**
  - **Option A (Liquibase seed migrations — preferred):** Versioned changesets. BIPM updates require a new migration. Consistent with OpenLeap starter convention.
  - **Option B (Application startup seeder):** `ApplicationReadyEvent` handler seeds via upsert on every startup.
  - **Option C (External seed script):** Admin CLI or CI/CD pipeline step; decoupled from service lifecycle.
- **Owner:** TBD (team-param)
- **Blocking for:** §13.1 Data Migration; implementation guide

---

## Q-SI-006: Compound Dimensional Expression Support in Conversion API

- **Question:** Should `POST /api/param/si/v1/conversions` support compound dimensional expressions (e.g., convert "5 kg·m/s²" to "N")?
- **Why it matters:** Engineering domain services may need compound quantity conversions. Supporting compound expressions significantly increases implementation complexity and may introduce expression injection risks.
- **Suggested options:**
  - **Option A (Scalar only — current spec):** One `fromUnit` → one `toUnit`. Compound quantities decomposed by the calling service.
  - **Option B (Compound in v2):** Add `POST /api/param/si/v1/conversions/compound` in a future version; document as roadmap item in §13.2.
  - **Option C (Dimensional lookup):** `GET /api/param/si/v1/units?dimensionalSignature=...` to resolve named units by dimension.
- **Owner:** TBD (team-param + domain service consumers)
- **Blocking for:** §6.3 Business Operations; future feature roadmap
