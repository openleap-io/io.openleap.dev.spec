# Open Questions — iam.audit

**Spec file:** `T1_Platform/iam/domain-specs/iam_audit-spec.md`
**Generated:** 2026-04-03

---

## Q-AUD-001: Configurable vs. Platform-Wide Retention Periods

- **Question:** Should the retention periods (7 years for SOX financial-action events, 3 years for other events) be configurable per tenant, or enforced as platform-wide constants? Some enterprise customers may have jurisdiction-specific requirements (e.g., 10 years for certain financial regulators in Germany or the US).
- **Why it matters:** Per-tenant configuration requires a tenant retention config table, validation that the value never falls below the platform minimum, and a more complex retention job. Global constants are simpler but may not satisfy all customer regulatory environments.
- **Suggested options:**
  - (A) Platform-wide constants (7 years SOX, 3 years other) — simpler implementation; no configuration surface
  - (B) Tenant-configurable with platform minimum floor — flexible; cannot be set below 3 years; requires config table and validation
  - (C) Per-event-type retention configured at the suite/platform level — fine-grained but operationally complex
- **Owner:** TBD (Legal + Platform Architecture)

---

## Q-AUD-002: Event-Based vs. REST-Based Ingest for High-Volume Producers

- **Question:** The current design uses synchronous REST ingest (UC-AUD-001, `POST /api/iam/audit/v1/events`). For high-volume producers (T3 domain services generating millions of events per day), is the REST approach sufficient, or should a broker-based ingest path be added?
- **Why it matters:** REST ingest provides strong delivery guarantees (synchronous acknowledgement) but may become a throughput bottleneck at extreme scale. Broker-based ingest decouples producers and scales more naturally, but introduces eventual consistency (events may lag by seconds) and risks silent gaps on DLQ overflow.
- **Suggested options:**
  - (A) REST only (current design) — synchronous, strong delivery guarantees, simpler operational model
  - (B) Add optional broker-based ingest exchange `iam.audit.events.in` — higher throughput, eventual consistency
  - (C) Hybrid: REST for critical events (AUTH_*, CONFIG_CHANGED), broker for high-volume informational events (DATA_ACCESSED, AUTHZ_GRANTED)
- **Owner:** TBD (Platform Architecture)

---

## Q-AUD-003: Migration Scope for On-Premise SAP SM20 Logs

- **Question:** Is migration of existing SAP SM20 Security Audit Log entries into `iam-audit-svc` required for customers migrating from SAP to OpenLeap? What is the maximum expected historical data volume?
- **Why it matters:** Determines the migration tool scope, the `iam_audit_events` table initial sizing, import performance requirements (batch ingest throughput), and whether partitions need to be pre-created for historical dates.
- **Suggested options:**
  - (A) No migration — greenfield only; historical SAP logs remain accessible in SAP or as archived exports
  - (B) Optional migration tool supporting last N years of SM20 data (N configurable, default 3)
  - (C) Full historical migration supported (may require bulk-import path bypassing the REST ingest API)
- **Owner:** TBD (Product Management + Customer Success)

---

## Q-AUD-004: Threshold Evaluator Implementation Strategy

- **Question:** Should the `AuditAlertThreshold` evaluator execute inline within the event ingest transaction (synchronous) or as an asynchronous background job that polls `iam_audit_events`?
- **Why it matters:** Inline evaluation adds 5–20ms latency to every ingest call and creates a failure dependency (threshold evaluation error could cause ingest to fail). Async evaluation minimises ingest latency but introduces a detection delay of 10–60 seconds before a threshold breach is signalled.
- **Suggested options:**
  - (A) Inline evaluation in the ingest application service — strong consistency; simplest implementation; higher ingest latency
  - (B) Async background evaluator polling `iam_audit_events` every N seconds — minimal ingest latency; detection lag; separate deployment unit
  - (C) Separate threshold evaluation microservice consuming from the outbox — cleanest separation; adds infrastructure complexity
- **Owner:** TBD (Platform Architecture + IAM Team)

---

## Q-AUD-005: Report Storage Configuration for S3/Azure Blob Destinations

- **Question:** When `destination = S3` or `destination = AZURE_BLOB` is specified in a report generation request, where does the target bucket/container configuration reside? Is it provided per-request in the request body, or stored as tenant-level configuration?
- **Why it matters:** Per-request credentials would require the API to accept and temporarily handle cloud provider credentials in the request body (security risk). Tenant-level configuration requires a separate config endpoint and table, but is more secure and reusable.
- **Suggested options:**
  - (A) Per-request: caller provides bucket URL + IAM role ARN or SAS token in the request body — flexible but requires careful secrets handling
  - (B) Tenant-level configuration: bucket/container URL and access policy stored in `iam-tenant-svc` or a dedicated export-config table; report request only specifies `destination = S3` — more secure and reusable
  - (C) Platform-managed storage only (`DOWNLOAD` destination) — simplest; S3/Azure destinations deferred to future version
- **Owner:** TBD (Platform Architecture + IAM Team)
