# F-FAC-002 — Funding & Settlement

> **Meta Information**
> - **Feature ID:** `F-FAC-002`
> - **Suite:** `fac`
> - **Node type:** COMPOSITION
> - **Template:** `feature-composition-spec.md` v1.0.0
> - **Template Compliance:** ~90%

## 0.1 Purpose
Groups all capabilities for **funding operations** — advance calculation, disbursement tracking, interest accrual, and settlement recording.

## 0.2 Children

| Child ID | Name | Node type | Group |
|---|---|---|---|
| `F-FAC-002-01` | Funding Creation & Advance Calculation | LEAF | mandatory |
| `F-FAC-002-02` | Disbursement Tracking | LEAF | optional |
| `F-FAC-002-03` | Settlement & Reserve Release | LEAF | optional |

## 0.3 Position in the Feature Tree
```
FAC_UI
├── F-FAC-001  Receivables Management  [COMPOSITION]
├── F-FAC-002  Funding & Settlement    [COMPOSITION] ← you are here
│   ├── F-FAC-002-01  Funding Creation         [LEAF] [mandatory]
│   ├── F-FAC-002-02  Disbursement Tracking    [LEAF] [optional]
│   └── F-FAC-002-03  Settlement & Reserve     [LEAF] [optional]
└── F-FAC-003  Collections & Risk     [COMPOSITION]
```

## 0.4 Domain Coverage
- Primary: `fac.fnd`
