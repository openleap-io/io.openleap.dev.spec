# F-FAC-001-01 — Receivable Intake & Worklist

> **Meta Information**
> - **Feature ID:** `F-FAC-001-01`
> - **Suite:** `fac`
> - **Node type:** LEAF
> - **Parent:** `F-FAC-001` (Receivables Management)
> - **Template:** `feature-spec.md` v1.0.0
> - **Template Compliance:** ~90%
> - **AUI Screen Contract:** `contracts/aui/F-FAC-001-01.aui.yaml`

---

## 0. Identity

**Name:** Receivable Intake & Worklist  
**Description:** Allows factoring operations staff to view all incoming receivables from fi.ar, filter by status/debtor/date, and take assignment or rejection actions.

**Primary Domain:** `fac.rcv`  
**API:** `GET /api/fac/rcv/v1/receivables`

---

## 1. Screens

| Screen ID | Name | Type | Description |
|-----------|------|------|-------------|
| SCR-FAC-001-01-A | Receivable List | List | Paginated, filterable worklist of all receivables |
| SCR-FAC-001-01-B | Receivable Detail | Detail | Full receivable view with eligibility result and audit trail |

---

## 2. UI Capabilities

- Filter by: status, clientPartyId, debtorPartyId, dueDateRange, amountRange
- Sort by: dueDate, amount, status, createdAt
- Bulk selection for batch assignment or rejection
- Inline eligibility check status badges (✓ Limit OK / ✗ Coverage failed / ?)
- Link to Assignment record when in ASSIGNED+ status
- Link to CollectionCase when in FUNDED/PARTLY_PAID status

---

## 3. Permissions

- **View:** `FAC_RCV_VIEWER`
- **Assign/Reject:** `FAC_RCV_EDITOR`

---

## 4. Dependencies

- Requires IAM authentication
- Calls `fac.rcv` API for data
- Eligibility badges pull from `GET /receivables/{id}/eligibility`
