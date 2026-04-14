# F-OPS-003-01 — Time Entry

> **Conceptual Stack Layer:** Domain-Feature
> **Space:** Business Domain
> **Owner:** Operations Engineering Team
> **Companion files:** `F-OPS-003-01.uvl`, `F-OPS-003-01.aui.yaml`
> **Referenced by:** Suite Feature Catalog §6
> **References:** `domain-specs/ops_tim-spec.md` (backend)

> **Meta Information**
> - **Version:** 2026-04-04
> - **Template:** `feature-spec.md` v1.0.0
> - **Template Compliance:** 100%
> - **Status:** DRAFT
> - **Feature ID:** `F-OPS-003-01`
> - **Suite:** `ops`
> - **Node type:** LEAF
> - **Parent:** `F-OPS-003` — Time & Project Tracking
> - **Companion UVL:** `uvl/leaves/F-OPS-003-01.uvl`
> - **Companion AUI:** `contracts/aui/F-OPS-003-01.aui.yaml`

---

## ═══════════════════════════════════════════════
## PROBLEM SPACE
## ═══════════════════════════════════════════════

## 0. Feature Identity & Orientation

### 0.1 One-Line Summary
This feature lets a **technician or consultant** record time spent on work orders, projects, and tasks — and submit a timesheet for manager approval — so that billing and payroll can be processed accurately.

### 0.2 Non-Goals
- Does not approve time entries — that is a manager action triggered by submission.
- Does not invoice the customer — that is FI suite.
- Does not calculate payroll — that is HR suite.

### 0.3 Entry & Exit Points

**Entry points:**
- Time → "Log Time"
- Direct URL: `/ops/tim/entries/new`
- From work order detail → "Log Time" shortcut

**Exit points:**
- Submit timesheet → entries sent for approval; navigate to time history
- Save draft → entries saved; stay on page

### 0.4 Variability Points

| Variability Point | Model | Values | Default | Binding Time |
|---|---|---|---|---|
| Billable flag visibility | UVL attribute | visible/hidden | visible | deploy |
| Minimum entry duration | UVL attribute | 0.25h, 0.5h, 1h | 0.25h | deploy |
| Timesheet period | UVL attribute | DAILY, WEEKLY | WEEKLY | deploy |

---

## 1. User Goal & Scenarios

### 1.1 User Goal
Quickly and accurately log all time spent today or this week — against the right work order, project, or task — and submit for approval in one action.

### 1.2 Scenarios

| # | Scenario | Precondition | Action | Expected Outcome |
|---|----------|-------------|--------|-----------------|
| S1 | Log time on work order | User authenticated | Select WO, enter hours, save | Time entry created linked to work order |
| S2 | Log project time | User authenticated | Select project + task, enter hours | Time entry linked to project task |
| S3 | Mark as billable | Entry form open | Check "Billable" checkbox | Entry flagged billable; will generate billable item on approval |
| S4 | Submit timesheet | Entries saved for week | Click "Submit Week" | All draft entries submitted for manager approval |
| S5 | View history | — | Open Time History tab | Paginated list of past entries with status |

---

## 2. User Journey & Screen Layout

### 2.1 Sequence Diagram

```mermaid
sequenceDiagram
    actor User
    participant BFF
    participant TIM as ops-tim-svc

    User->>BFF: POST /ops/tim/entries (entry data)
    BFF->>TIM: POST /api/ops/tim/v1/time-entries
    TIM-->>BFF: 201 { entryId, status: DRAFT }
    BFF-->>User: Entry added to week view

    User->>BFF: POST /ops/tim/entries/submit (week)
    BFF->>TIM: POST /api/ops/tim/v1/time-entries/submit
    TIM-->>BFF: 200 { submitted: N }
    BFF-->>User: Confirmation; entries in SUBMITTED state
```

### 2.2 Screen Layout

```
┌─────────────────────────────────────────────────────┐
│ Log Time   [Week: Apr 7–11 ▾]   [+ Add Entry]       │
├────────────┬──────────┬──────┬──────┬───────────────┤
│ Date       │ WO/Proj  │ Hours│ Bill?│ Status        │
├────────────┼──────────┼──────┼──────┼───────────────┤
│ Mon Apr 7  │ WO-001   │ 4.0  │ ✓    │ DRAFT         │
│ Mon Apr 7  │ WO-002   │ 3.5  │ ✓    │ DRAFT         │
│ Tue Apr 8  │ PRJ-023  │ 8.0  │ ✗    │ DRAFT         │
├────────────┴──────────┴──────┴──────┴───────────────┤
│ Week total: 15.5h                                    │
│                        [Save Draft]  [Submit Week]   │
└─────────────────────────────────────────────────────┘
```

---

## 3. Interaction Requirements

### 3.1 Fields Table

| Field | Type | Required | Editable | Validation | i18n Key |
|---|---|---|---|---|---|
| Date | date picker | Yes | Yes | Not future | `F-OPS-003-01.field.date` |
| Work Order / Project | typeahead | Yes | Yes | Must resolve to valid ID | `F-OPS-003-01.field.costObject` |
| Hours | number | Yes | Yes | Min 0.25, max 24 | `F-OPS-003-01.field.hours` |
| Billable | checkbox | No | Yes | — | `F-OPS-003-01.field.billable` |
| Notes | textarea | No | Yes | Max 500 chars | `F-OPS-003-01.field.notes` |

### 3.2 Actions Table

| Action | Trigger | Precondition | Effect |
|---|---|---|---|
| Add Entry | Button | — | Open inline entry form |
| Save Draft | Button | ≥1 entry | POST entries with status DRAFT |
| Submit Week | Button | ≥1 draft entry | POST submit; entries → SUBMITTED |
| Delete entry | Row action | Entry in DRAFT | DELETE entry |

### 3.3 Validation Messages

| Field | Condition | Message |
|---|---|---|
| Hours | < 0.25 | `F-OPS-003-01.validation.hours.min` |
| Hours | > 24 | `F-OPS-003-01.validation.hours.max` |
| Date | Future date | `F-OPS-003-01.validation.date.future` |

---

## 4. Edge Cases & Screen States

### 4.1 Component States

| State | When | Behaviour |
|---|---|---|
| **Empty** | No entries this week | "No time entries for this week. Tap + to add." |
| **Submitted** | All entries submitted | Read-only view; submit button disabled |
| **Rejected** | Manager rejected | Entries highlighted; re-submit allowed after editing |

### 4.2 Specific Edge Cases

| Case | Behaviour | Affected users |
|---|---|---|
| Duplicate entry (same date + WO + hours) | Warning shown; allow proceed | User |
| Submitted entry edited | Not allowed; must ask manager to reopen | User |

### 4.3 Attribute-Driven Behaviour Changes

| Attribute | Non-default value | Observable change |
|---|---|---|
| `billableFlagVisibility` | hidden | Billable checkbox not shown; entries non-billable by default |
| `timesheetPeriod` | DAILY | Submit button submits only today's entries |

### 4.4 Connectivity
Offline: entries saved locally; submitted when back online.

---

## ═══════════════════════════════════════════════
## SOLUTION SPACE
## ═══════════════════════════════════════════════

## 5. Backend Dependencies & BFF Contract

### 5.1 Service Calls

| # | Service | Endpoint | Tier | isMutation | Failure Mode |
|---|---------|----------|------|------------|-------------|
| 1 | ops-tim-svc | `GET /api/ops/tim/v1/time-entries?week=2026-W15` | T3 | No | Show cached or error |
| 2 | ops-tim-svc | `POST /api/ops/tim/v1/time-entries` | T3 | Yes | Queue offline |
| 3 | ops-tim-svc | `POST /api/ops/tim/v1/time-entries/submit` | T3 | Yes | Queue offline |

### 5.2 BFF View-Model Shape

```jsonc
{
  "week": "2026-W15",
  "entries": [
    {
      "entryId": "te-uuid",
      "date": "2026-04-07",
      "costObjectId": "wo-uuid",
      "costObjectType": "WORK_ORDER",
      "hours": 4.0,
      "billable": true,
      "status": "DRAFT"
    }
  ],
  "totalHours": 15.5
}
```

### 5.3 Feature-Gating Rules

| Mode | Behaviour |
|---|---|
| Full | All interactions available |
| Excluded | Menu item hidden; URL returns 404 |

### 5.4 Failure Modes

| Failure | User Experience |
|---------|----------------|
| ops-tim-svc down | Cached entries shown; offline banner; new entries queued |

### 5.5 Caching Hints
BFF SHOULD cache weekly entry list per user for 2 minutes. Mobile app caches for offline use.

### 5.6 i18n Keys

| Key | Default (en) |
|-----|-------------|
| `F-OPS-003-01.title` | `Log Time` |
| `F-OPS-003-01.action.addEntry` | `Add Entry` |
| `F-OPS-003-01.action.saveDraft` | `Save Draft` |
| `F-OPS-003-01.action.submitWeek` | `Submit Week` |
| `F-OPS-003-01.field.hours` | `Hours` |
| `F-OPS-003-01.field.billable` | `Billable` |

---

## 6. AUI Screen Contract

See companion file `contracts/aui/F-OPS-003-01.aui.yaml`.

---

## ═══════════════════════════════════════════════
## BRIDGE ARTIFACTS
## ═══════════════════════════════════════════════

## 7. Permissions & Accessibility

### 7.1 Permission Matrix

| Action | TECHNICIAN | CONSULTANT | FIELD_ENGINEER | OPERATIONS_MANAGER |
|---|---|---|---|---|
| Create/edit own entries | ✓ | ✓ | ✓ | ✓ |
| Submit own timesheet | ✓ | ✓ | ✓ | ✓ |
| View others' entries | ✗ | ✗ | ✗ | ✓ |

### 7.2 Accessibility
- Week navigation MUST be keyboard accessible.
- Billable checkbox MUST have `aria-label` describing its effect.

---

## 8. Acceptance Criteria

| AC | Scenario | Given | When | Then |
|----|----------|-------|------|------|
| AC-01 | S1 | User opens Log Time | Creates entry for WO-001, 4h | Entry saved with status DRAFT |
| AC-02 | S3 | Entry form open | User checks Billable | Entry flagged billable; on approval ops.tim.time-entry.submitted event published with billable=true |
| AC-03 | S4 | Draft entries for week | User clicks Submit Week | All draft entries change to SUBMITTED; approval workflow triggered |
| AC-04 | Error | ops-tim-svc unavailable | User creates entry | Entry queued locally; submitted when back online |

---

## 9. Variability & Extension

### 9.1 Feature Dependencies
Requires IAM authentication. Cost objects resolved via ops-ord-svc (work orders) and ops-prj-svc (projects).

### 9.2 Attributes
See §0.4 variability points. Binding time: `deploy`.

### 9.3 Extension Points
| Extension Zone | Interface | Default Behaviour |
|---|---|---|
| `ext.timeEntryCustomFields` | Additional entry fields | Hidden |

### 9.4 Companion UVL
See `uvl/leaves/F-OPS-003-01.uvl`.

---

**END OF SPECIFICATION**
