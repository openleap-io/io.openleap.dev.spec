# F-TKS-200 — Multi-Channel Intake

> **Template:** `feature-composition-spec.md` v1.0.0
> **Status:** DRAFT
> **Suite:** `tks`
> **Node type:** COMPOSITION
> **Parent:** `TKS_UI`
> **Primary service:** `tks.ch`
> **Bounded context:** `bc:channels`
> **Companion UVL:** `F-TKS-200.uvl`
> **Cross-suite deps:** tech.email, iam.authz

---

## SS0 — Identity

**Purpose:** Multi-Channel Intake — grouping feature within the TKS suite.

### Children

| Feature ID | Name | Node Type | Status |
|-----------|------|-----------|--------|
| `F-TKS-210` | Email Channel Adapter | LEAF | `draft` |
| `F-TKS-220` | Web Form / Portal | LEAF | `draft` |
| `F-TKS-230` | Webhook Channel | LEAF | `draft` |
| `F-TKS-240` | Chat Channel (placeholder) | LEAF | `draft` |

**Position in tree:** `TKS_UI` → `F-TKS-200` → 4 leaf feature(s)

---

## SS1 — Children & Variability Structure

**Group type:** `optional`

**Business rationale:** Multi-Channel Intake is a optional capability within the TKS suite. Features can be selected or combined per product composition.

### Variability Points

- None beyond child selection; child features carry their own attributes.

---

## SS2 — Cross-Feature Dependencies

See `_tks_suite.md` SS6.3 for the cross-suite dependency catalogue. Intra-TKS dependencies are encoded in `tks.catalog.uvl` constraints.

---

## SS3 — User Personas

| Persona | Interest |
|---------|----------|
| Support Agent | Accesses as needed |
| Support Manager | Configures + monitors |
| End Customer / Requester | Self-serves via portal |

---

## SS4 — Status & Ownership

| Field | Value |
|---|---|
| Status | DRAFT |
| Owner | team-tks |
| Last reviewed | 2026-04-18 |
