# F-FAC-003 — Collections & Risk

> **Meta Information**
> - **Feature ID:** `F-FAC-003`
> - **Suite:** `fac`
> - **Node type:** COMPOSITION
> - **Template:** `feature-composition-spec.md` v1.0.0
> - **Template Compliance:** ~90%

## 0.1 Purpose
Groups all capabilities for **collections management and risk intelligence** — dunning campaigns, cash application, dispute resolution, credit limit management, and risk scoring.

## 0.2 Children

| Child ID | Name | Node type | Group |
|---|---|---|---|
| `F-FAC-003-01` | Dunning & Collection Cases | LEAF | mandatory |
| `F-FAC-003-02` | Credit Limit Management | LEAF | optional |
| `F-FAC-003-03` | Risk Scoring & Coverage | LEAF | optional |

## 0.3 Position in the Feature Tree
```
FAC_UI
├── F-FAC-001  Receivables Management  [COMPOSITION]
├── F-FAC-002  Funding & Settlement    [COMPOSITION]
└── F-FAC-003  Collections & Risk     [COMPOSITION] ← you are here
    ├── F-FAC-003-01  Dunning & Collections   [LEAF] [mandatory]
    ├── F-FAC-003-02  Credit Limit Mgmt       [LEAF] [optional]
    └── F-FAC-003-03  Risk Scoring & Coverage [LEAF] [optional]
```

## 0.4 Domain Coverage
- F-FAC-003-01: `fac.col`
- F-FAC-003-02: `fac.lim`
- F-FAC-003-03: `fac.rsk`
