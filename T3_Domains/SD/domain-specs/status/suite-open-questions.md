# SD Suite — Open Questions

## [OQ-SD-001] Returns (sd.ret) vs Billing (sd.bil) credit note ownership
- **Status:** Open
- **Question:** When a return is processed, does sd.ret or sd.bil own the credit note creation event?
- **Impact:** Event routing for `sd.ret.return.approved` vs `sd.bil.credit-note.created`

## [OQ-SD-002] CDM (Customer Data Management) vs CRM suite boundary
- **Status:** Open
- **Question:** Is sd.cdm responsible for customer master data, or does the CRM suite own it?
- **Impact:** Feature model constraints and event producer ownership

## [OQ-SD-003] Delivery split threshold for sd.dlv
- **Status:** Open
- **Question:** What is the maximum line count before a delivery must be split?
- **Impact:** Business rule definition in F-SD-002-02 (Shipment Execution)
