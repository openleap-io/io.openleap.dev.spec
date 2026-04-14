# Open Questions - sd.dlv

## Q-DLV-001: Wave Picking Support
- **Question:** Should sd.dlv support wave picking — grouping pick tasks across multiple deliveries for a single warehouse run?
- **Why it matters:** Required for high-volume warehouse operations; impacts PickTask aggregate design and WM integration
- **Suggested options:** (A) Phase 2 feature, (B) Delegate entirely to WM
- **Owner:** TBD

## Q-DLV-002: Cross-Docking
- **Question:** Should sd.dlv support cross-docking (goods received and immediately staged for outbound without entering storage)?
- **Why it matters:** Required for transit hub operations; changes goods movement flow significantly
- **Suggested options:** (A) Phase 3 feature, (B) Separate bounded context
- **Owner:** TBD

## Q-DLV-003: Catch-Weight Handling
- **Question:** Should sd.dlv support catch-weight products where actual weight differs from ordered quantity?
- **Why it matters:** Required for industries with weight-variable products (meat, fish, cheese)
- **Suggested options:** (A) Deferred to v2, (B) Add toleranceWeight fields to DeliveryLine
- **Owner:** TBD

## Q-DLV-004: Measurement Value Object
- **Question:** Should weight+uomCode and volume+uomCode be extracted as a Measurement value object?
- **Why it matters:** Currently duplicated as flat fields across Delivery and HandlingUnit; a VO would improve consistency
- **Suggested options:** (A) Keep flat fields for simplicity, (B) Extract Measurement VO in v2 refactor
- **Owner:** Architecture Team

## Q-DLV-005: Product Feature Spec IDs
- **Question:** Which product feature specs in the SD suite reference sd.dlv endpoints?
- **Why it matters:** Feature IDs in §11.2 (F-SD-DLV-001 through F-SD-DLV-009) are provisional; need validation against actual feature catalog
- **Suggested options:** Review SD product feature catalog; align with _sd_suite.md feature registry
- **Owner:** Product Owner

## Q-DLV-006: Historical SAP Migration Strategy
- **Question:** What is the data migration strategy for historical SAP delivery documents (LIKP/LIPS)?
- **Why it matters:** Determines §13.1 scope; full history vs. cut-over date has major impact on migration effort
- **Suggested options:** (A) Full history migration, (B) Cut-over date only (historical in SAP read-only), (C) No migration
- **Owner:** Migration Team

## Q-DLV-007: Pick Task Ownership
- **Question:** Should sd.dlv create PickTask stubs internally, or should WM create pick tasks entirely from the pickReleased event?
- **Why it matters:** Impacts coupling between sd.dlv and WM; affects audit trail ownership
- **Suggested options:** (A) sd.dlv creates pick stubs (current design), WM enriches with location data; (B) WM owns full pick task lifecycle, sd.dlv only tracks aggregate quantities
- **Owner:** Architecture Team

## Q-DLV-008: Additional Domain-Level ADRs
- **Question:** Are there additional domain-level architectural decisions for sd.dlv that should be documented as ADRs?
- **Why it matters:** ADR-DLV-001 and ADR-DLV-002 cover the main decisions; further ADRs may be needed as the service matures
- **Suggested options:** Review with Architecture Team in next design review
- **Owner:** Architecture Team
