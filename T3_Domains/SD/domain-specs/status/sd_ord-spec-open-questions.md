# Open Questions - sd.ord (Sales Orders)

## Q-ORD-001: Multi-Delivery Split per Line

- **Question:** Should a single SalesDocumentLine support tracking against multiple partial deliveries, or should the line be split when a partial delivery occurs?
- **Why it matters:** Partial delivery is a common scenario (e.g., deliver 60 of 100 units now, 40 later). The current model tracks `confirmedDeliveryDate` and `status` per line but does not model multiple delivery schedule lines per order line. Without this, partial fulfillment tracking is ambiguous.
- **Suggested options:**
  - **Option A:** Add a `deliveryReferences` array field to SalesDocumentLine storing (deliveryId, quantity, date) references. Minimal schema change.
  - **Option B:** Split the original line into sub-lines when a partial delivery occurs (e.g., line 10 becomes 10.1 and 10.2). Preserves existing model but complicates lineNumber uniqueness.
  - **Option C:** Add a separate `ord_delivery_references` table linking (sales_document_id, line_id, delivery_id, quantity). Most flexible; recommended for Phase 2.
- **Owner:** TBD
- **Priority:** Medium — needed before go-live if partial delivery scenarios are in scope

---

## Q-ORD-002: Inter-Company Sales Order Support

- **Question:** Should sd.ord support inter-company sales orders where the seller and buyer are both entities within the same corporate group?
- **Why it matters:** Inter-company sales require bilateral document creation (outbound order at selling entity, inbound purchase order at buying entity), settlement pricing, and profit elimination in consolidation. This significantly impacts the data model (additional IC partner functions, IC pricing conditions, IC settlement events).
- **Suggested options:**
  - **Option A:** Defer to Phase 3; model as a separate document type `IC` in DocumentType enum with specific partner functions.
  - **Option B:** Handle via COM.PRC pricing conditions only; no structural changes to sd.ord.
- **Owner:** TBD
- **Priority:** Low — Phase 3

---

## Q-ORD-003: Feature Spec IDs for SD.ORD Feature Dependency Register

- **Question:** What are the definitive feature IDs (F-SD-ORD-*) for the features that depend on sd-ord-svc? The §11.2 register uses provisional IDs.
- **Why it matters:** Feature dependency register (§11) and BFF aggregation hints (§11.4) require stable feature IDs to be actionable. Without authored feature specs, §11 is a placeholder. Feature IDs must be assigned when SD suite feature specs are authored.
- **Suggested options:**
  - **Option A:** Author the SD suite feature specs first, then backfill §11.2 with confirmed IDs.
  - **Option B:** Confirm provisional IDs (F-SD-ORD-001 through F-SD-ORD-006) in the feature spec authoring sprint.
- **Owner:** SD Product Owner
- **Priority:** High — blocks BFF specification for any product using SD suite

---

## Q-ORD-004: SAP Document Number Migration Format

- **Question:** During migration from SAP, should SAP document numbers (e.g., `0000000123`) be preserved as the `document_number` in OpenLeap, or should they be stored as `externalRef` while OpenLeap generates new document numbers in its own format (e.g., `SO-2026-000457`)?
- **Why it matters:** Determines how customer-facing references change post-migration. If SAP numbers are replaced, customer communications (PO references, delivery notifications) must be updated. If preserved, the OpenLeap numbering scheme must accommodate SAP-format numbers.
- **Suggested options:**
  - **Option A:** Preserve SAP number as `document_number`; configure number range to continue from max SAP number. Minimal customer impact.
  - **Option B:** Store SAP number as `externalRef`; generate new OpenLeap numbers. Clean break; requires customer communication.
  - **Option C:** Store SAP number as a `customFields.sapDocumentNumber` extension field; generate new OpenLeap numbers. Most flexible; preserves both.
- **Owner:** Migration Team / Business Stakeholders
- **Priority:** High — must be decided before migration planning

---

## Q-ORD-005: OpenLeap Starter Version and Service Port Assignment

- **Question:** Which OpenLeap Starter version should sd-ord-svc target, and what port is assigned in the service registry?
- **Why it matters:** The Starter version determines available ADR implementations, Spring Boot version, and available `core-extension` module capabilities (ADR-067). Port assignment is required for Kubernetes service definitions and local development setup.
- **Suggested options:** Assign from platform team; current latest Starter version is TBD.
- **Owner:** Platform Team
- **Priority:** High — required before implementation begins

---

## Q-ORD-006: Repository URI for sd-ord-svc

- **Question:** What is the Git repository URI for the sd-ord-svc implementation?
- **Why it matters:** Required for `metadata.repository` in the spec (§2), CI/CD pipeline configuration, and developer onboarding.
- **Suggested options:** Create under `github.com/io.openleap/sd-ord-svc` or equivalent organization naming convention.
- **Owner:** Platform Team
- **Priority:** Medium — needed before implementation kick-off

---

## Q-ORD-007: Scheduling Agreement Delivery Schedule Lines

- **Question:** Scheduling Agreement (SA) documents require multiple delivery schedule lines per order line (e.g., deliver 100 units on 2026-05-01, 200 units on 2026-06-01, 150 units on 2026-07-01). The current model has only `confirmedDeliveryDate` and `quantity` per line, which is insufficient for SA delivery scheduling.
- **Why it matters:** Without modeling schedule lines, SA documents cannot be properly represented or exported to sd.dlv for scheduled delivery creation. This is a structural gap for the SA document type.
- **Suggested options:**
  - **Option A:** Add a new `ord_schedule_lines` table with columns (id, line_id, schedule_date, quantity, confirmed_quantity, status). Requires schema addition but is the correct normalized approach.
  - **Option B:** Store schedule lines as a JSONB array on `ord_lines.custom_fields`. Simpler but sacrifices queryability and indexing.
  - **Option C:** Defer SA document type to Phase 2; exclude from initial delivery and mark `SA` in DocumentType as `Deprecated: TBD`.
- **Owner:** TBD
- **Priority:** Medium — blocks SA document type support; Phase 2 if SA is not in initial scope
