# F-TKS-100 — Ticket Lifecycle

> **Template:** `feature-composition-spec.md` v1.0.0
> **Status:** DRAFT
> **Suite:** `tks`
> **Node type:** COMPOSITION
> **Parent:** `TKS_UI`
> **Primary service:** `tks.tkt`
> **Bounded context:** `bc:tickets`
> **Companion UVL:** `F-TKS-100.uvl`
> **Cross-suite deps:** iam.principal, iam.audit

---

## SS0 — Identity

**Purpose:** Ticket Lifecycle — grouping feature within the TKS suite.

### Children

| Feature ID | Name | Node Type | Status |
|-----------|------|-----------|--------|
| `F-TKS-100-01` | Create Ticket (agent form) | LEAF | `draft` |
| `F-TKS-100-02` | Ticket Detail & Timeline | LEAF | `draft` |

**Position in tree:** `TKS_UI` → `F-TKS-100` → 2 leaf feature(s)

---

## SS1 — Children & Variability Structure

**Group type:** `mandatory`

**Business rationale:** Ticket Lifecycle is a mandatory capability within the TKS suite. All children are required when this feature is selected.

### Variability Points

- None beyond child selection; child features carry their own attributes.

---

## SS2 — Cross-Feature Dependencies

See `_tks_suite.md` SS6.3 for the cross-suite dependency catalogue. Intra-TKS dependencies are encoded in `tks.catalog.uvl` constraints.

---

## SS3 — User Personas

| Persona | Interest |
|---------|----------|
| Support Agent | Handles tickets daily |
| Support Manager | Configures + monitors |
| End Customer / Requester | Indirect beneficiary |

---

## SS4 — Status & Ownership

| Field | Value |
|---|---|
| Status | DRAFT |
| Owner | team-tks |
| Last reviewed | 2026-04-18 |
