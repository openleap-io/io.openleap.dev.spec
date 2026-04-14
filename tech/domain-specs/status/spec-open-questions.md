# Open Questions — tech.dms

## Q-DMS-001: GDPR Right-to-Erasure vs GoBD COMPLIANCE Object Lock

- **Question:** How should the DMS reconcile a GDPR right-to-erasure request (Art. 17) with a document stored under S3 Object Lock in COMPLIANCE mode for GoBD 10-year retention? COMPLIANCE mode prevents deletion by any user, including platform administrators, until the lock period expires.
- **Why it matters:** An incorrect implementation creates legal liability in either direction — non-compliance with GDPR (failure to erase) or non-compliance with GoBD (premature deletion of tax-relevant documents). The DMS must have a documented reconciliation policy before production use.
- **Suggested options:**
  - **Option A:** GDPR erasure requests for GoBD-locked documents are rejected with a documented legal basis (tax-law obligation overrides erasure right per GDPR Art. 17(3)(b)). Legal team must confirm the specific document types this applies to.
  - **Option B:** GDPR erasure anonymizes all personally-identifiable metadata (name, email, etc.) in PostgreSQL but leaves the S3 binary intact until lock expiry. The binary itself may not contain PII in some cases (e.g., a PDF invoice with only company references).
  - **Option C:** A legal hold override workflow, requiring dual authorization from a Data Protection Officer + Platform Admin, to request early lock removal from the object storage vendor (MinIO enterprise feature).
- **Owner:** Platform Legal + Platform Infra + Data Protection Officer

---

## Q-DMS-002: Search Index Strategy for UC-DMS-012

- **Question:** Should the DMS implement full-text document search (UC-DMS-012) by publishing a search-index event to Elasticsearch, or should Elasticsearch be populated via CDC (Debezium) reading the `tech_dms_document` PostgreSQL table directly?
- **Why it matters:** The choice determines whether the DMS owns the search projection (event-based) or delegates it to a platform-level CDC pipeline. Event-based requires DMS to publish an additional event type and maintain a schema contract with Elasticsearch. CDC is operationally simpler but couples Elasticsearch to the physical table structure.
- **Suggested options:**
  - **Option A (Event-based):** DMS publishes `tech.dms.document.indexed` thin event on document ACTIVE; a search microservice or worker updates Elasticsearch. Looser coupling, additional event contract.
  - **Option B (CDC/Debezium):** A platform-level Debezium pipeline reads `tech_dms_document` and pushes to Elasticsearch. DMS has no search responsibility. Simpler for DMS; Elasticsearch schema tied to table schema.
  - **Option C (Query-time):** No Elasticsearch. DMS exposes PostgreSQL full-text search via `tsvector` index on `title` + `attributes`. Simpler infrastructure; lower scalability ceiling.
- **Owner:** Platform Architecture Team

---

## Q-DMS-003: Per-Tenant Document Type Configuration

- **Question:** Should a future DMS version support per-tenant customization of document types (custom labels, required attribute schemas, icon mappings)? Currently `DocumentType` is a fixed system enum.
- **Why it matters:** Enterprise customers often need custom document classifications (e.g., "Quality Certificate", "Delivery Acceptance Form"). If this is deferred, the enum becomes a long-term migration liability as customers onboard.
- **Suggested options:**
  - **Option A:** Remain enum-based (current). Simpler implementation; evolution via new enum values.
  - **Option B:** Add a `TenantDocumentType` aggregate in a future version that allows per-tenant type registry; current enum becomes the "system type" fallback.
  - **Option C:** Replace enum with a `docTypeCode` free-string referencing a system catalog in the `param` suite. Param becomes the authority for document type definitions.
- **Owner:** Product Management + Platform Architecture

---

## Q-DMS-004: ClamAV Deployment Topology

- **Question:** Should the ClamAV virus scanning worker be deployed as:
  a) A separate microservice (`tech-dms-scanner-svc`) with its own repository and deployment unit, or
  b) A Spring Batch job / container sidecar co-deployed with `tech-dms-svc`?
- **Why it matters:** Affects repository structure, scaling strategy, and operational ownership. A separate microservice adds operational overhead but allows independent scaling and upgrading. A sidecar is simpler but ties scanner updates to DMS releases.
- **Suggested options:**
  - **Option A (Separate microservice):** Independent deployment; scales independently; can be upgraded without DMS redeploy. Additional repo and Helm chart required.
  - **Option B (Sidecar/Job):** Co-deployed with DMS as a Spring Batch step triggered by queue listener. Simpler operations; coupled lifecycle.
  - **Option C (External SaaS):** Delegate scanning to a cloud antivirus API (e.g., VirusTotal API). Removes ClamAV infrastructure but introduces external SaaS dependency and data privacy concerns.
- **Owner:** Platform Infra Team + DevOps

---

## Q-DMS-005: MinIO Server-Side Encryption Mode

- **Question:** What server-side encryption mode should DMS use for MinIO object storage: SSE-S3 (platform-managed keys, single master key per bucket) or SSE-KMS (customer-managed keys per tenant, e.g., via HashiCorp Vault or AWS KMS)?
- **Why it matters:** SSE-KMS with per-tenant keys allows key revocation per tenant (useful for GDPR erasure and tenant offboarding) but adds significant operational complexity (key rotation, Vault integration, key escrow). SSE-S3 is simpler but uses platform-managed keys for all tenants.
- **Suggested options:**
  - **Option A (SSE-S3):** Simpler; platform manages a single encryption master key. Tenant data is encrypted but not with tenant-specific keys. Acceptable for most enterprise customers.
  - **Option B (SSE-KMS per tenant):** Per-tenant encryption keys stored in HashiCorp Vault. Allows cryptographic tenant erasure (key deletion = data unreadable). Required for high-compliance customers.
  - **Option C (Hybrid):** SSE-S3 as default; SSE-KMS as an opt-in for tenants on enterprise plans. Balances simplicity and compliance capability.
- **Owner:** Security & Compliance Team + Platform Infra
