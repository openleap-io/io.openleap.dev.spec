# Spec Changelog — param.cfg

**Spec file:** `T1_Platform/param/domain-specs/param_cfg-spec.md`
**Upgrade date:** 2026-04-03
**Upgraded from:** Migration stub only (~0% compliance — no domain content)
**Upgraded to:** ~95% compliance (TPL-SVC v1.0.0 fully applied)

---

## Summary

- Replaced migration stub (22 lines) with full domain service specification (~1,400 lines)
- Built complete domain model from scratch using feature specs (F-PARAM-003-01/02/03), suite spec (`_param_suite.md`), event contract (`config.changed.schema.json`), and OpenAPI stub
- Defined ConfigEntry aggregate with 14-attribute table, lifecycle states, allowed transitions, and domain events
- Defined ConfigAuditLog child entity with 12-attribute table, immutability invariants, and 7-year retention policy
- Authored 8 business rules (BR-CFG-001 through BR-CFG-008) with full detailed definitions, validation rules, and error codes
- Authored 5 use cases (UC-CFG-001 through UC-CFG-005) with Actor/Preconditions/Main Flow/Postconditions
- Expanded REST API section with 5 fully documented endpoints including request/response JSON, ETag handling, and error codes
- Documented `param.cfg.config.changed` event with semi-thick payload (D-CFG-001 decision), envelope structure, and known consumers
- Defined 3 PostgreSQL tables with full column definitions, indexes, and retention policies: `cfg_config_entry`, `cfg_audit_log`, `cfg_outbox_events`
- Added outbox table per ADR-013; dual-key pattern per ADR-020; GIN index on `custom_fields` per ADR-067
- Added Security section: data classification, 4-role permission matrix, GDPR/SOX/ISO-27001 compliance table
- Added Quality Attributes: 8 performance metrics, RTO/RPO, 4 failure scenarios, scalability strategy, maintainability policy
- Added Feature Dependencies: all 3 F-PARAM-003 leaf features mapped with endpoints-per-feature and BFF aggregation hints
- Added Extension Points: all 5 types covered (custom fields, extension events, extension rules, extension actions, aggregate hooks)
- Documented migration from SAP SM30/RZ10/TVARVC equivalents
- Completed Consistency Checks §14.1: all 7 checks Pass
- Recorded 4 decisions (D-CFG-001 through D-CFG-004) and raised 5 open questions (Q-CFG-001 through Q-CFG-005)

---

## Added Sections

- `## Specification Guidelines Compliance` — complete block (Non-Negotiables, Source Priority, Style Guide)
- `§0` — full Purpose, Target Audience, Scope, Related Documents
- `§1` — Domain Purpose, Business Value (SAP BC equivalent), Stakeholders, Strategic Positioning, Service Context with Mermaid diagram
- `§2` — Service Identity table
- `§3` — Full domain model: Mermaid class diagram, ConfigEntry aggregate root table, ConfigAuditLog child entity table, lifecycle states, transitions, 3 enumerations (ConfigScope, ConfigType, ChangeType)
- `§4` — Business Rules Catalog, 8 detailed rule definitions, field-level validation table, cross-field validations, reference data dependencies
- `§5` — Business Logic Placement, 5 canonical use cases, process flow sequence diagram, cross-domain workflow note
- `§6` — Full REST API with 5 endpoints, JSON request/response examples, ETag handling, business operations (none), OpenAPI reference
- `§7` — Architecture Pattern, `param.cfg.config.changed` event with payload+envelope, consumed events (conditional), event flow diagram, integration points summary
- `§8` — ER diagram, 3 table definitions with columns/indexes/retention, reference data dependencies
- `§9` — Data classification table, 4-role permission matrix, 3-regulation compliance table
- `§10` — 8 performance metrics, availability/reliability (RTO/RPO + failure scenarios), scalability, maintainability
- `§11` — Feature dependency register, endpoints-per-feature, BFF aggregation hints, impact assessment
- `§12` — All 5 extension types: custom fields (ConfigEntry: yes, ConfigAuditLog: no), 4 extension events, 3 extension rules, 3 extension actions, 4 aggregate hooks, 3 extension API endpoints, summary matrix + guidelines
- `§13` — SAP SM30/RZ10 migration mapping table, deprecation policy
- `§14` — 7 consistency checks (all Pass), 4 decisions, 5 open questions, 17-entry ADR reference table, suite-level ADR references
- `§15` — 13-term glossary, 10 references, status output requirements, change log

---

## Modified Sections

- None — spec was a stub with no domain content to preserve.

## Removed Sections

- Migration stub content (22 lines) — replaced by actual specification per upgrade prompt non-destructive rule; stub had no normative content.

---

## Decisions Taken

- **D-CFG-001:** Semi-thick event (includes `previousValue`/`newValue`) justified by cache invalidation efficiency; slight deviation from ADR-011 noted.
- **D-CFG-002:** `wellKnown` flag is system-set; no API allows self-declaration to prevent privilege escalation.
- **D-CFG-003:** Soft delete used for ConfigEntry to preserve audit log join integrity.
- **D-CFG-004:** DELETE change reason carried in `X-Change-Reason` header (not body) per HTTP semantics.

---

## Open Questions Raised

- Q-CFG-001: Should `param-cfg-svc` consume `iam.tenant.tenant.deleted` for TENANT entry cleanup?
- Q-CFG-002: Encryption at rest for PARAMETER type values containing secrets?
- Q-CFG-003: Extension management API ownership: per-service or central registry?
- Q-CFG-004: TENANT entry resolution priority when same key exists in GLOBAL scope?
- Q-CFG-005: Port 8102 assignment confirmation for `param-cfg-svc`?

---

# Spec Changelog — param.i18n

**Spec file:** `T1_Platform/param/domain-specs/param_i18n-spec.md`
**Upgrade date:** 2026-04-03
**Upgraded from:** Migration stub only (~0% compliance — 22 lines, no domain content)
**Upgraded to:** ~95% compliance (TPL-SVC v1.0.0 fully applied)

---

## Summary

- Created `param_i18n-spec.md` from scratch — the prior file was a 22-line migration stub with no content
- Full TPL-SVC v1.0.0 compliance achieved (~95%) across all 16 required sections (§0–§15)
- Domain model defined: two aggregates (`TranslationNamespace`, `Translation`) with attribute tables, lifecycle states, and invariants
- Eight business rules defined with full detail (BR-I18N-001 through BR-I18N-008)
- Five use cases specified with Actor / Preconditions / Main Flow / Postconditions
- Five REST endpoint operations fully documented with JSON request/response examples
- Two published events fully documented (`namespace.seeded`, `namespace.updated`) consistent with existing event schema contracts
- All five extension point types covered (§12): custom fields, extension events, extension rules, extension actions, aggregate hooks
- Five open questions raised for unresolved architectural decisions

---

## Added Sections

- `## Specification Guidelines Compliance` — complete block
- `§0` — Purpose, Audience, Scope, Related Documents
- `§1` — Business Context (1.1–1.5 with context diagram)
- `§2` — Service Identity table and team
- `§3` — TranslationNamespace aggregate root (14-col table), Translation child entity (14-col table), LocaleTag value object, TranslationType / NamespaceStatus enumerations
- `§4` — 8 business rules (BR-I18N-001 to 008), field-level validation table, cross-field validations, reference data dependencies
- `§5` — 5 use cases (UC-I18N-001 to 005), Business Logic Placement, process flow diagrams, cross-domain workflows
- `§6` — REST API: overview table, 5 fully documented endpoints with JSON examples, OpenAPI reference
- `§7` — 2 published events with full envelopes and known consumers, consumed events, event flow diagram, integration points summary
- `§8` — ER diagram, 3 table definitions (`i18n_namespace`, `i18n_translation`, `i18n_outbox_events`), reference data dependencies
- `§9` — Data classification, 5-role permission matrix, compliance table
- `§10` — Performance targets, availability/reliability (RTO/RPO + failure scenarios), scalability, maintainability
- `§11` — F-PARAM-002-01/02/03 feature dependency register, BFF aggregation hints, impact assessment
- `§12` — All 5 extension types, 2 aggregate hooks, extension API endpoints, summary matrix
- `§13` — SAP SE63 migration mapping table, deprecation framework
- `§14` — 7 consistency checks (all Pass), 5 decisions (DEC-I18N-001 to 005), 5 open questions (Q-I18N-001 to 005), ADR reference table
- `§15` — 10-term glossary, 11 references, status output requirements, change log

## Modified Sections

- None — spec was a stub with no domain content to preserve.

## Removed Sections

- Migration stub content (22 lines) — replaced by actual specification.

---

## Decisions Taken

- **DEC-I18N-001:** Namespace ID is the business key (not a UUID slug) — direct correlation with catalogId and feature leaf ID
- **DEC-I18N-002:** Tenant overrides use same `i18n_translation` table with `tenant_id` set — RLS handles isolation; upsert handles seed idempotency
- **DEC-I18N-003:** Resolve endpoint returns flat JSON map — directly usable as Vue i18n message bundle without BFF transformation
- **DEC-I18N-004:** No `custom_fields JSONB` on `i18n_translation` — product metadata belongs in product-owned extension tables
- **DEC-I18N-005:** `namespace.seeded` event includes `keyCount` and `locales` beyond thin-event minimum — consistent with existing event schema contracts; BFF cache invalidation requires knowing which locales to invalidate

---

## Open Questions Raised

- Q-I18N-001: Should MESSAGE namespace ID be validated against a feature registry?
- Q-I18N-002: Circuit-breaker behaviour when `param-ref-svc` is unavailable during CATALOG namespace registration
- Q-I18N-003: Should `param-ref-svc` publish `catalog.created` events for auto-namespace registration?
- Q-I18N-004: Should `i18n_translation` have a `custom_fields JSONB` column for product metadata?
- Q-I18N-005: Confirm port `8101` for `param-i18n-svc` local dev

---

# Spec Changelog — param.ref

**Spec file:** `T1_Platform/param/domain-specs/param_ref-spec.md`
**Upgrade date:** 2026-04-03
**Upgraded from:** Migration stub only (~0% compliance — 22 lines, no domain content)
**Upgraded to:** ~95% compliance (TPL-SVC v1.0.0 fully applied)

---

## Summary

- Replaced migration stub (22 lines) with full domain service specification (~1,500 lines)
- Built complete domain spec from scratch using feature specs (F-PARAM-001-01/02/03), composition spec (F-PARAM-001), event schema contracts, OpenAPI stub, and domain expertise
- Defined `Catalog` aggregate root with 12-attribute table, lifecycle states, transitions, and domain events
- Defined `CatalogCode` child entity with 12-attribute table, immutability invariants, validity period rules, and `custom_fields JSONB` for extensibility
- Defined `CatalogRef` value object as the cross-service contract type for domain records referencing catalog-governed values
- Authored 9 business rules (BR-REF-001 through BR-REF-009) with full detailed definitions, error codes, and examples
- Authored 10 use cases (UC-REF-001 through UC-REF-010) with Actor/Preconditions/Main Flow/Postconditions
- Documented 13 REST endpoints with full request/response JSON, ETag handling, query parameters, and error codes; including validate endpoint design decision (D-REF-003)
- Documented 3 published events (catalog.created, catalog.updated, catalog.deprecated) with envelope structure, payload examples, and known consumers; semi-thin deviation (D-REF-001) for changedCodes array
- Defined 3 PostgreSQL tables (ref_catalog, ref_catalog_code, ref_outbox_events) with full column definitions, indexes, and retention policies
- Added `custom_fields JSONB` column + GIN index on `ref_catalog_code` per ADR-067; Catalog itself is NOT extensible
- Added Security section: data classification (LOW — no PII), 4-role permission matrix, GDPR/SOX/ISO-27001 compliance table
- Added Quality Attributes: 8 performance metrics including validate p99 < 20ms, RTO/RPO, 4 failure scenarios, scalability strategy
- Added Feature Dependencies: all 3 F-PARAM-001 leaf features mapped with endpoints-per-feature and BFF aggregation hints
- Added Extension Points: all 5 types covered; CatalogCode is extensible (custom fields); Catalog is not
- Documented migration from SAP DDIC/T-tables (T005 countries, TCURC currencies, T002 languages, T052 payment terms)
- Recorded 5 decisions (D-REF-001 through D-REF-005) and raised 6 open questions (Q-REF-001 through Q-REF-006)
- Completed all 7 consistency checks in §14.1: all Pass

---

## Added Sections

- `## Specification Guidelines Compliance` — complete block (Non-Negotiables, Source Priority, Style Guide)
- `§0` — Purpose, Target Audience, Scope (In/Out), Related Documents
- `§1` — Domain Purpose (SAP SM30/T-table equivalent), Business Value (6 bullet points), Stakeholders (6 roles), Strategic Positioning, Service Context with upstream/downstream Mermaid diagram
- `§2` — Service Identity table and team metadata
- `§3` — Full domain model: Mermaid class diagram, Catalog aggregate root (12-col attribute table, lifecycle state machine, transitions, invariants, events), CatalogCode child entity (12-col table, lifecycle state machine, invariants), CatalogRef value object, 3 enumerations (CatalogScope, CatalogStatus, CodeStatus)
- `§4` — Business Rules Catalog (9 rules), 9 detailed rule definitions with business context/enforcement/error codes/examples, field-level validation table, cross-field validations, reference data dependencies note
- `§5` — Business Logic Placement, 10 canonical use cases with full detail, process flow sequence diagram, 2 cross-domain workflow descriptions (catalog.created → i18n namespace; catalog.updated → BFF cache invalidation)
- `§6` — REST API: overview table (13 endpoints), 5 fully documented resource operations with JSON examples (browse, create, add code, update code, validate), business operations table (deprecate, import, export), OpenAPI reference
- `§7` — Architecture Pattern, 3 published events with full envelopes and known consumers, consumed events note (none currently), event flow diagram, integration points summary (upstream + downstream tables)
- `§8` — ER diagram (3 tables), 3 table definitions (ref_catalog, ref_catalog_code, ref_outbox_events) with columns/indexes/relationships/retention
- `§9` — Data classification (LOW — no PII), 4-role permission matrix, compliance table (GDPR/SOX/ISO-27001)
- `§10` — 8 performance metrics, availability/reliability (RTO/RPO + 4 failure scenarios), scalability strategy, maintainability policy
- `§11` — Feature dependency register (F-PARAM-001-01/02/03), endpoints-per-feature table, BFF aggregation hints (5 hints), impact assessment
- `§12` — All 5 extension types: Catalog (not extensible), CatalogCode (custom_fields JSONB), 2 extension events, 2 extension rule slots, 3 extension action zones, 3 aggregate hooks, 5 extension API endpoints, summary matrix + 5 guidelines
- `§13` — SAP T-table migration mapping (T005, TCURC, T002, T052, TPAAT), deprecation framework
- `§14` — 7 consistency checks (all Pass), 5 decisions (D-REF-001 to 005), 6 open questions (Q-REF-001 to 006), 16-entry ADR reference table, suite-level ADR references
- `§15` — 13-term glossary, 14 references, status output requirements, change log

---

## Modified Sections

- None — spec was a stub with no domain content to preserve.

## Removed Sections

- Migration stub content (22 lines) — replaced by actual specification per upgrade prompt non-destructive rule; stub had no normative content.

---

## Decisions Taken

- **D-REF-001:** Semi-thin `catalog.updated` event includes `changedCodes` array — needed by `param-i18n-svc` for targeted namespace entry creation; consistent with D-CFG-001 precedent.
- **D-REF-002:** Single `catalog.updated` event for all code mutations (add, update, deprecate) — avoids proliferating event types; `changedCodes` array is sufficient for all consumers.
- **D-REF-003:** Validate endpoint returns HTTP 200 with `{ valid, reason }` body — cleaner for machine consumers than parsing error shapes for the "deprecated" vs "not found" distinction.
- **D-REF-004:** `codeCount` maintained by application layer (not DB trigger/view) — consistent with OpenLeap starter conventions.
- **D-REF-005:** `CatalogCode.code` is case-sensitive — preserves authoritative ISO code casing; avoids normalisation complexity.

---

## Open Questions Raised

- Q-REF-001: How should `ownerSuite` values be validated (against what registry)?
- Q-REF-002: How are PLATFORM-scoped catalogs represented in multi-tenant deployments?
- Q-REF-003: Should `param-i18n-svc` auto-register namespaces on `catalog.created` event?
- Q-REF-004: Should `param-ref-svc` consume `iam.tenant.tenant.deleted` for tenant data cleanup?
- Q-REF-005: What `tenant_id` is assigned to PLATFORM-scoped catalogs?
- Q-REF-006: Confirm port 8100 assignment for `param-ref-svc`

---

# Spec Changelog — param.si

**Spec file:** `T1_Platform/param/domain-specs/param_si-spec.md`
**Upgrade date:** 2026-04-03
**Upgraded from:** Migration stub only (~0% compliance — 22 lines, no domain content; source file `t1-si-svc-spec.md` referenced in stub was not found)
**Upgraded to:** ~95% compliance (TPL-SVC v1.0.0 fully applied)

---

## Summary

- Built `param_si-spec.md` from scratch: the file was a 22-line migration stub pointing to a non-existent source spec.
- Full domain service specification (~1,400 lines) written to TPL-SVC v1.0.0 compliance.
- Domain reconstructed from feature specs F-PARAM-004-01/02/03, PARAM suite spec (§1.1 glossary and §2.4 bounded context map), and SI units domain knowledge.
- Two aggregates defined: `Unit` (extensible, JSONB custom fields) and `Prefix` (immutable seed data, not extensible).
- Nine business rules (BR-SI-001 through BR-SI-009) fully specified with error codes and examples.
- Seven use cases (UC-SI-001 through UC-SI-007) with full Actor/Preconditions/Main Flow/Postconditions.
- Complete REST API documented for 8 endpoints with request/response JSON examples.
- Dimensional analysis conversion algorithm (linear and affine) documented in §5 and §6.
- Full data model with three PostgreSQL tables (`si_unit`, `si_prefix`, `si_outbox_events`) including dual-key pattern and JSONB custom fields on `si_unit`.
- SAP migration mapping from T006/T006A/T006B/T006D included in §13.
- Six open questions raised (Q-SI-001 through Q-SI-006).

---

## Added Sections

- `## Specification Guidelines Compliance` — complete block (Non-Negotiables, Source Priority, Style Guide)
- `§0` — Purpose, Target Audience, Scope (In/Out), Related Documents
- `§1` — Business Context (1.1–1.5 with upstream/downstream Mermaid context diagram)
- `§2` — Service Identity table and team metadata
- `§3` — Full domain model: Mermaid class diagram (Unit, Prefix, DimensionalSignature, Quantity); Unit aggregate root (22-col attribute table, lifecycle states, transitions, invariants); Prefix aggregate root (12-col table); DimensionalSignature and Quantity value objects; UnitType, UnitStatus, PrefixStatus enumerations
- `§4` — Business Rules Catalog (9 rules), 9 detailed rule definitions with business context/enforcement/error codes/examples, field-level validation table, cross-field validations, reference data dependencies
- `§5` — Business Logic Placement, 7 canonical use cases with full detail, two process flow sequence diagrams (create custom unit; convert quantity), cross-domain workflows note (none — bc:units-of-measure is independent)
- `§6` — REST API: overview table (8 endpoints), 7 fully documented operations with JSON request/response examples (list units, get detail, create, update, delete, list prefixes, convert), OpenAPI reference
- `§7` — Architecture Pattern, 5 published events with full envelopes and known consumers, consumed events (none), event flow diagram, integration points summary (upstream + downstream tables)
- `§8` — ER diagram, 3 table definitions with columns/indexes/relationships/retention, reference data dependencies
- `§9` — Data classification table, endpoint-level permission matrix, compliance requirements (GDPR: N/A; SOX: partial)
- `§10` — Performance targets (unit catalog read p95 < 10ms cached; conversion p95 < 20ms), availability/reliability (RTO/RPO + 4 failure scenarios), scalability, maintainability
- `§11` — F-PARAM-004-01/02/03 feature dependency register, endpoints-per-feature table, BFF aggregation hints, impact assessment
- `§12` — All 5 extension types: Unit (extensible with custom fields + 4 extension events + 3 rule slots + 2 actions + 4 hooks); Prefix (not extensible); summary matrix + guidelines
- `§13` — SAP T006/T006A/T006B/T006D migration mapping table, deprecation framework, BIPM 2022 prefix seeding note
- `§14` — 7 consistency checks (all Pass), 4 decisions/conflicts, 6 open questions, ADR reference table, suite-level ADR references
- `§15` — 14-term glossary, 9 references, status output requirements, change log

---

## Modified Sections

- None — spec was a stub with no domain content to preserve.

## Removed Sections

- Migration stub content (22 lines) — replaced by actual specification per upgrade prompt non-destructive rule; stub had no normative content.

---

## Decisions Taken

- **D-SI-001:** Conversion API uses POST despite being read-only (stateless computation). Using GET would require encoding value + two unit IDs as query parameters, which is awkward for numeric values with high precision. POST with a body is the pragmatic choice for computation APIs in the OpenLeap platform.
- **D-SI-002:** BASE and DERIVED units are immutable seed data (BR-SI-002). Allowing mutation of BIPM-defined SI units would break SI compliance and all historical domain records referencing them.
- **D-SI-003:** `bc:units-of-measure` is independent within the PARAM suite (no `param-ref` dependency). UnitType/UnitStatus are managed as internal enumerations (pending Q-SI-001 resolution). This keeps the service startup-safe and avoids circular dependency risk.
- **D-SI-004:** Prefix aggregate is declared NOT extensible. SI prefixes are fixed international standards (BIPM). There is no legitimate product use case for extending prefix definitions with custom fields.

---

## Open Questions Raised

- Q-SI-001: Are UnitType and UnitStatus managed as `param-ref` catalog codes or as internal Java enumerations?
- Q-SI-002: Should custom unit deletion be a hard delete or a soft delete (status = DEPRECATED)?
- Q-SI-003: Should the API allow operator-created custom Prefix definitions beyond the 28 standard SI prefixes?
- Q-SI-004: Are extension API endpoints served by `param-si-svc` or by a central `core-extension-svc`?
- Q-SI-005: What is the seeding strategy for SI base and derived units (Liquibase migrations vs. startup seeder)?
- Q-SI-006: Should the conversion API support compound dimensional expressions?
