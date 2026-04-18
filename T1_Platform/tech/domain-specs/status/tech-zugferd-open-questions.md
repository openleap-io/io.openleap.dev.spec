# Open Questions — tech.zugferd

## Q-ZF-001: Confirmed Port Assignment

- **Question:** What is the confirmed port assignment for `tech-zugferd-svc`? The spec uses `8098` based on the existing sequence (DMS: 8095, JC: 8096, RPT/NFS: 8097). Is `8098` correct, or is a different port reserved?
- **Why it matters:** Pod configuration, Kubernetes service definitions, and DNS entries depend on the assigned port. Incorrect port causes service discovery failures.
- **Suggested options:**
  - **Option A:** `8098` — next in sequence after the confirmed JC port (`8096`) and tentative RPT/NFS port (`8097`).
  - **Option B:** A separately assigned port from the platform port registry.
- **Owner:** TBD (`team-tech`)

---

## Q-ZF-002: Maximum File Size for Invoice PDFs

- **Question:** Is the 20 MB maximum file size (BR-ZF-004, `maxFileSizeBytes: 20971520`) appropriate for the expected range of invoice PDFs? A standard one-page invoice is typically < 500 KB; complex multi-attachment PDFs may be 5–10 MB; large batch-prepared PDFs could reach 50+ MB.
- **Why it matters:** Too low → callers cannot embed large invoices; too high → OOM risk in the stateless service and degraded latency. The value is returned in `GET /info` and enforced at request entry.
- **Suggested options:**
  - **Option A:** 5 MB (standard invoices; large PDFs split upstream)
  - **Option B:** 20 MB (current default; covers most real-world cases)
  - **Option C:** 50 MB (maximum; requires more generous memory allocation per pod)
- **Owner:** TBD (Finance stakeholders + `team-tech`)

---

## Q-ZF-003: Currency and Country Validation — Runtime vs. Bundled Static Lists

- **Question:** Should the service validate `currencyCode` (ISO 4217) and `countryCode` (ISO 3166-1) by calling `param-ref-svc` at runtime, or by using a bundled static JSON list on the classpath?
- **Why it matters:** This is a stateless service; runtime calls to `param-ref-svc` add latency and create a runtime dependency on a separate service. Bundled static lists are faster but may be stale if ISO updates are not included in each release.
- **Suggested options:**
  - **Option A:** Bundled static JSON files on classpath, updated on each service release. Zero latency; zero external dependency; minor risk of stale codes.
  - **Option B:** Call `param-ref-svc` at runtime with a short-lived in-memory cache (5-minute TTL). Always current; adds ~10–50 ms latency on cache miss.
  - **Option C:** Bundled lists with an optional runtime refresh endpoint (`POST /api/tech/zugferd/v1/admin/refresh-reference-data`).
- **Owner:** TBD (`team-tech`)

---

## Q-ZF-004: Process Flow Sequence Diagrams (§5.3)

- **Question:** The §5.3 Process Flow Diagrams section is stubbed. Should Mermaid sequence diagrams be added for the three main flows: embed (invoiceData path), validate, and extract?
- **Why it matters:** Some code generation tooling and architecture review processes require sequence diagrams to verify correct service interaction and error handling paths.
- **Suggested options:**
  - **Option A:** Add Mermaid sequence diagrams for all four operations in §5.3.
  - **Option B:** Leave §5.3 as a stub and reference §5.4 (cross-domain workflow) as the primary diagram artifact.
- **Owner:** TBD (Architecture Team)

---

## Q-ZF-005: OAuth2 Scope / Authorization for Embed, Validate, Extract

- **Question:** What OAuth2 scope or role is required to call the embedding, validation, and extraction endpoints? The tech suite specification marks ZUGFeRD authorization as an open question (suite spec §7 security note: "ZUGFeRD: open question (Q-001 in spec)"). This is that question.
- **Why it matters:** BFF feature gating, API gateway authorization policy, and the permission matrix in §9.2 depend on this decision. Incorrect scoping could expose financial invoice data to unauthorized consumers.
- **Suggested options:**
  - **Option A:** Single scope `tech.zugferd:use` for embed/validate/extract; `tech.zugferd:read` for info. Simple; low maintenance overhead.
  - **Option B:** Separate scopes: `tech.zugferd:embed`, `tech.zugferd:validate`, `tech.zugferd:extract`. Granular; useful if some callers should only validate, not embed.
  - **Option C:** Role-based (RBAC): Domain service M2M tokens get embed/validate/extract; human roles get validate/extract only; PLATFORM_ADMIN gets all. Matches existing platform IAM pattern.
- **Owner:** TBD (`team-tech`, IAM team)

---

## Q-ZF-006: Peak Invoice Embed Volume for Capacity Planning

- **Question:** What are the expected peak embed request volumes per second across FI, SD, and SRV domains? The §10.3 capacity planning uses a placeholder of 50 req/sec at peak, derived from an assumed daily volume of ~5 000 invoices. Actual volumes are unknown.
- **Why it matters:** Autoscaling thresholds, pod resource requests/limits, and horizontal pod autoscaler (HPA) configuration depend on accurate volume estimates. Over-provisioning wastes cost; under-provisioning causes latency spikes at invoice dispatch peak.
- **Suggested options:**
  - **Option A:** Conduct volume estimation workshops with FI, SD, SRV domain teams before production deployment.
  - **Option B:** Start with 2 instances, monitor actual load via `embed_duration_ms` and error rate metrics, and adjust HPA targets accordingly.
- **Owner:** TBD (FI / SD / SRV domain teams + `team-tech`)

---

## Q-ZF-007: FI, SD, SRV Feature Dependency Entries for §11.2

- **Question:** Which specific features in the FI (finance), SD (sales distribution), and SRV (service) suites will call `POST /api/tech/zugferd/v1/embed` as part of their invoice dispatch workflow? These features should be added to the §11.2 Feature Dependency Register.
- **Why it matters:** The §11 blast radius analysis is incomplete without these entries. Changes to the embed API (error codes, request fields, response structure) may break these features; they must be tracked to enable impact assessment.
- **Suggested options:**
  - **Option A:** Survey FI, SD, SRV domain leads to identify all invoice dispatch features and add them to §11.2.
  - **Option B:** Add a placeholder row per suite (`F-FI-xxx`, `F-SD-xxx`, `F-SRV-xxx`) and update as those features are specced.
- **Owner:** TBD (FI / SD / SRV domain leads)

---

## Q-ZF-008: Extension API Contract for Custom Schematron Rule Registration (§12.7)

- **Question:** What is the full API contract for the extension endpoints that allow product teams to register custom schematron rule sets (§12.7)? The section is currently stubbed.
- **Why it matters:** Without a defined contract, product teams cannot implement custom EN 16931 extensions. The registration API must define: rule set format (`.sch` file vs. compiled `.xsl`), tenancy scope, activation/deactivation mechanism, and conflict resolution with standard rules.
- **Suggested options:**
  - **Option A:** Accept compiled `.xsl` files via multipart form upload; associate with `tenantId` in an in-memory registry (lost on restart → requires persistence, contradicting ADR-TECH-003).
  - **Option B:** Accept `.sch` rule set via JSON body, compile at registration, store in `param-cfg-svc` as a config blob per tenant. Extension rules loaded on service startup from config.
  - **Option C:** Defer extension API entirely; document as ROADMAP item for v2. Products use the `invoiceXml` path to bypass standard field enforcement if needed.
- **Owner:** TBD (Architecture Team)

---

## Q-ZF-009: ZUGFeRD 1.0 Backward Compatibility for Extraction and Re-Embedding

- **Question:** Should the extract endpoint support ZUGFeRD 1.0 hybrid PDFs (which use the attachment filename `ZUGFeRD-invoice.xml` and a different XML namespace)? Currently, extraction from ZUGFeRD 1.0 PDFs would fail with `ZF_UNKNOWN_PROFILE_NAMESPACE` because the namespace is not in the `ZugferdProfile` enumeration.
- **Why it matters:** Customers migrating from legacy systems may have historical ZUGFeRD 1.0 PDFs in DMS that need to be re-processed or re-validated. Failing with an opaque error is a poor developer experience and blocks migration workflows.
- **Suggested options:**
  - **Option A:** Return raw XML with `detectedProfile: "LEGACY_10"` and add `LEGACY_10` to the `ZugferdProfile` enum (non-breaking addition). No re-embedding support for LEGACY_10; caller must convert externally.
  - **Option B:** Silently detect ZUGFeRD 1.0 namespace, auto-translate to ZUGFeRD 2.3 equivalent, and return with `detectedProfile: "BASIC"`. Transparent to caller; complex to implement correctly.
  - **Option C:** Reject with a clear error `ZF_LEGACY_FORMAT_UNSUPPORTED` and a message advising upgrade. Simple; caller knows they need to convert upstream.
- **Owner:** TBD (Architecture Team, Finance stakeholders)
