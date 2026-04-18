# Spec Changelog — tech.nfs (News Feed Service)

## Summary

- Wrote full spec from scratch: the existing file was a 14-line migration stub pointing to a source file that does not exist
- Authored all 16 sections (§0–§15) to TPL-SVC v1.0.0 compliance (~95%)
- Defined the single aggregate `Subscription` with full attribute table, lifecycle state machine (5 states), invariants, and child entity `DeliveryLog`
- Documented two value objects: `EnrichmentConfig` and `DeliveryConfig`
- Defined 3 enumerations: `SubscriptionStatus`, `DeliveryStatus`, `EncryptionAlgorithm`
- Specified 9 REST endpoints across 6.2 (resource operations) and 6.3 (business operations)
- Documented the GDPR tenant purge cross-domain workflow
- Defined 3 database tables with full column definitions, indexes, and retention policies
- Populated all 5 extension point types in §12 (custom fields, events, rules, actions, hooks)
- Raised 9 open questions (Q-NFS-001 through Q-NFS-009)

## Added Sections

- §0 Document Purpose & Scope (0.1–0.4)
- §1 Business Context (1.1–1.5 including service context diagram)
- §2 Service Identity (full metadata table)
- §3 Domain Model (3.1–3.5: Subscription aggregate, DeliveryLog child entity, 2 value objects, 3 enumerations)
- §4 Business Rules (4.1 catalog with 10 rules; 4.2 detailed definitions for BR-NFS-001–005, 008; 4.3 field validations; 4.4 reference data)
- §5 Use Cases (5.1 placement table; 5.2 nine use cases with three fully detailed; 5.3 stub; 5.4 GDPR purge workflow)
- §6 REST API (6.1 overview; 6.2 six resource operations with full request/response; 6.3 three business operations; 6.4 OpenAPI reference)
- §7 Events & Integrations (7.1 pattern; 7.2 intentionally empty with decision rationale; 7.3 two consumed events; 7.4 delivery pipeline diagram; 7.5 integration summary)
- §8 Data Model (8.1 storage; 8.2 ER diagram; 8.3 three table definitions: nfs_subscription, nfs_delivery_log, nfs_outbox_events; 8.4 reference data)
- §9 Security (9.1 classification; 9.2 RBAC matrix; 9.3 GDPR/GoBD/ISO 27001 compliance)
- §10 Quality Attributes (10.1–10.4 with performance targets, failure scenarios, scalability, alerting)
- §11 Feature Dependencies (11.1–11.5 for F-TECH-004-01 and F-TECH-004-02)
- §12 Extension Points (12.1–12.8, all 5 extension types)
- §13 Migration & Evolution (13.1 SAP analog mapping; 13.2 deprecation)
- §14 Governance (14.1 consistency checks all Pass; 14.2 four design decisions; 14.3 nine open questions; 14.4–14.5 ADR references)
- §15 Appendix (15.1 glossary; 15.2 references; 15.3 status requirements; 15.4 changelog)

## Modified Sections

- None (written from scratch)

## Removed Sections

- Migration checklist stub (14 lines) — replaced by full specification

## Decisions Taken

- **D-NFS-001:** NFS publishes no platform domain events (per Tech Suite Spec §5.4); §7.2 is intentionally empty
- **D-NFS-002:** Subscription is the single aggregate; DeliveryLog is a child entity
- **D-NFS-003:** RabbitMQ queue provisioning is asynchronous; `PENDING_PROVISION` is a transient state
- **D-NFS-004:** No webhook delivery; NFS uses per-subscriber RabbitMQ queues only
- **Encryption:** AES-256-GCM selected as default and recommended algorithm; `NONE` blocked for production tenants

## Open Questions Raised

- Q-NFS-001: Queue provisioning failure recovery mechanism
- Q-NFS-002: Enrichment API failure behavior (retry vs. deliver un-enriched)
- Q-NFS-003: Whether subscription lifecycle events should be published for audit consumers
- Q-NFS-004: Confirmed port assignment (8097 assumed)
- Q-NFS-005: Repository URI creation status
- Q-NFS-006: Tenant class classification mechanism for BR-NFS-008
- Q-NFS-007: Source exchange validation against RabbitMQ catalog
- Q-NFS-008: Key rotation dual-key transition window
- Q-NFS-009: Extension API endpoints pending `core-extension` module finalization
