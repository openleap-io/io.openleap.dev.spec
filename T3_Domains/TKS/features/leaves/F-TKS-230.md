# F-TKS-230 — Webhook Channel

> **Template:** `feature-spec.md` v1.0.0 (leaf)
> **Status:** DRAFT
> **Suite:** `tks`
> **Node type:** LEAF (Screen / Use-Case)
> **Parent composition:** `F-TKS`
> **Primary service:** `tks.ch`
> **Bounded context:** `bc:channels`
> **Companion UVL:** `F-TKS-230.uvl`
> **Companion AUI:** `contracts/aui/F-TKS-230.aui.yaml`

---

## SS0 — Identity & Purpose

**Purpose:** HMAC-signed inbound HTTP endpoint for integrations (monitoring, alerting, custom apps) to push ticket events programmatically.

**User goal:** Achieve the workflow step this screen supports with minimum friction, one-click access to most-used actions, and clear error signals when validation fails.

---

## SS1 — Primary Journey

1. User opens screen (navigation or deep link).
2. System loads required view-model via BFF.
3. User completes interaction; optimistic UI for low-risk operations.
4. Server confirms; UI updates.
5. On error: inline message per field + top-of-form alert with guidance.

---

## SS2 — Interaction Requirements

| Aspect | Detail |
|---|---|
| Primary action | Context-appropriate (Submit / Save / Send / Create / Confirm) |
| Secondary actions | Cancel, Save as draft (where applicable) |
| Keyboard support | Full tab order; Ctrl/⌘+Enter submits |
| Accessibility | WCAG 2.2 AA (labels, ARIA live-regions, contrast ≥ 4.5:1) |

---

## SS3 — View-Model Shape

Canonical view-model composed by BFF; see `contracts/aui/F-TKS-230.aui.yaml` for the AUI (abstract UI) contract and `../domain-specs/*.md` §6 for the underlying REST calls.

---

## SS4 — Backend Dependencies

| Service | Tier | Purpose | Is Mutation |
|---------|------|---------|-------------|
| `tks.ch-svc` | T3 | Primary domain operations | Yes / No per operation |
| `shared-bp-svc` | T2 | Party lookup (reporter / commenter) | No |
| `iam-principal-svc` | T1 | Actor identity | No |

Full call list in BFF contract (`bff-contract.md`, deferred to implementation phase).

---

## SS5 — Variability

No sub-selectable variability at leaf level beyond tenant configuration (language, branding — handled by Elara product composition, not here).

---

## SS6 — Status & Ownership

| Field | Value |
|---|---|
| Status | DRAFT |
| Owner | team-tks |
| Last reviewed | 2026-04-18 |
