# Open Questions - fi.bnk

## Q-BNK-001: PSD2 / Real-Time Bank Feed Support
- **Question:** Should fi.bnk support real-time bank feeds (PSD2/Open Banking API) in addition to file-based import?
- **Why it matters:** Affects API design (webhook endpoints for bank notifications), event model (streaming vs. batch), and ImportBatch lifecycle (may need a STREAMING format). Also impacts security requirements (PSD2 SCA, consent management).
- **Suggested options:** A) Phase 2 feature with new aggregate (BankFeed) alongside existing ImportBatch, B) Extend ImportBatch with STREAMING format and add webhook ingress endpoint
- **Owner:** TBD

## Q-BNK-002: Reconciliation Feature Ownership
- **Question:** Should reconciliation features (bank balance vs. GL balance comparison) move entirely to fi.rpt or should fi.bnk provide raw balance endpoints?
- **Why it matters:** Determines whether BFF needs to call fi.bnk for balance data or only fi.rpt. Affects API surface and caching strategy.
- **Suggested options:** A) fi.rpt owns all reconciliation views (fi.bnk provides only raw data via events), B) fi.bnk exposes simple balance endpoints for BFF aggregation with fi.gl data
- **Owner:** TBD

## Q-BNK-003: Feature Dependency Completeness
- **Question:** Which product features depend on fi-bnk-svc endpoints? Feature specs for F-FI-BNK-001/002/003 do not yet exist.
- **Why it matters:** Section 11 (Feature Dependencies) cannot be fully populated without feature specs. Affects BFF aggregation hints and impact assessment.
- **Suggested options:** Create feature specs for Bank Statement Import, Cash Position Dashboard, and Payment Instruction Management
- **Owner:** TBD

## Q-BNK-004: Voucher Splitting -- Core vs. Extension
- **Question:** Should the voucher split operation (POSTED -> SPLIT) be a standard platform feature requiring BNK_ADMIN role, or should it be a product-level extension via hooks?
- **Why it matters:** If core, the split logic and child voucher creation must be specified in section 5 (Use Cases) and section 6 (REST API). If extension, it belongs in section 12 only.
- **Suggested options:** A) Core platform feature (split is a common bank accounting operation), B) Product extension via hook-003 pre-transition gate
- **Owner:** TBD

## Q-BNK-005: Port Assignment
- **Question:** What is the exact port assignment for fi-bnk-svc in the FI suite port range?
- **Why it matters:** Needed for service registry, deployment configuration, and local development setup. Port 8210 is assumed based on FI suite port range.
- **Suggested options:** Confirm with platform team; check landscape/service-registry for conventions
- **Owner:** TBD

## Q-BNK-006: Multi-Currency Statement Support
- **Question:** Should fi.bnk support bank statements where individual lines have different currencies (e.g., a multi-currency account)?
- **Why it matters:** Affects balance equation validation (BR-STMT-001) which currently assumes homogeneous currency. Also affects voucher currency handling and GL posting logic.
- **Suggested options:** A) Single-currency per statement (current design; simplest), B) Multi-currency with per-line currency and exchange rate fields
- **Owner:** TBD

## Q-BNK-007: Parser Version Strategy
- **Question:** What is the parser versioning strategy for CAMT.053, MT940, and CSV parsers?
- **Why it matters:** ImportBatch.parserVersion tracks which parser produced the data. Needed for regression testing, re-parsing, and audit trail. Strategy affects how parser upgrades are rolled out.
- **Suggested options:** A) SemVer per parser (e.g., camt053-parser:1.2.0), B) Global parser module version
- **Owner:** TBD
