# F-TKS-320 — KB Search & Suggest

> **Template:** `feature-composition-spec.md` v1.0.0
> **Status:** DRAFT
> **Suite:** `tks`
> **Node type:** COMPOSITION
> **Parent:** `TKS_UI`
> **Primary service:** `tks.kb`
> **Bounded context:** `bc:knowledge × bc:channels`
> **Companion UVL:** `F-TKS-320.uvl`
> **Cross-suite deps:** tech.search, tech.ai

---

## SS0 — Identity

**Purpose:** KB Search & Suggest — grouping feature within the TKS suite.

**Position in tree:** `TKS_UI` → `F-TKS-320`

---

## SS1 — Children & Variability Structure

**Group type:** `optional`

**Business rationale:** KB Search & Suggest is a optional capability within the TKS suite. Features can be selected or combined per product composition.

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
| End Customer / Requester | Indirect beneficiary |

---

## SS4 — Status & Ownership

| Field | Value |
|---|---|
| Status | DRAFT |
| Owner | team-tks |
| Last reviewed | 2026-04-18 |
