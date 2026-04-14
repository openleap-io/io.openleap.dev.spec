# OPS Suite — Open Questions

## [OQ-OPS-001] Service Order vs Work Order boundary
- **Status:** Open
- **Question:** Is ops.svc a thin wrapper over ops.ord, or does it have independent aggregate roots?
- **Impact:** Feature model and event schema design

## [OQ-OPS-002] Time entry billing integration with FI
- **Status:** Open
- **Question:** Does ops.tim directly post to FI via events, or does COM/billing mediate?
- **Impact:** `billable_flag` processing flow and FI event routing

## [OQ-OPS-003] Document storage backend for ops.doc
- **Status:** Open
- **Question:** Does ops.doc use a dedicated document store (S3/MinIO) or the platform DMS (T1)?
- **Impact:** Storage configuration and feature gating for F-OPS-001-03
