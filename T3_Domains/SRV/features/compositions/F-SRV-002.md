# F-SRV-002 — Appointment & Booking

> **Conceptual Stack Layer:** Platform-Feature (Composition Node)
> **Space:** Platform
> **Owner:** Domain Engineering Team
> **Companion files:** `F-SRV-002.uvl`
> **Referenced by:** Suite Feature Catalog (`_srv_suite.md` §6), Platform-Feature Specs (children)

> **Meta Information**
> - **Version:** 2026-04-02
> - **Author(s):** OpenLeap Architecture Team
> - **Status:** DRAFT
> - **Feature ID:** `F-SRV-002`
> - **Suite:** `srv`
> - **Node type:** COMPOSITION
> - **Parent:** suite root (`SRV_UI`)
> - **Companion UVL:** `F-SRV-002.uvl`

> **What this document is**
> A composition node document describes the **variability structure** over a
> set of child features. It does NOT contain a user journey, screens, or
> service calls — those live in leaf nodes.
>
> This document answers:
> - What capability area does this node group?
> - What is the boolean relationship between its children?
> - Why are the children structured this way?
> **Template:** `feature-composition-spec.md` v1.0.0
> **Template Compliance:** ~100% — all sections present

---

## SS0. Identity

### 0.1 Purpose

This composition node groups all capabilities related to **appointment and booking management** in the SRV suite. It covers the customer-facing booking lifecycle from slot discovery through confirmation, as well as operational concerns like waitlisting and no-show handling. A product that includes any booking-related feature MUST include at least the mandatory children (Slot Discovery and Booking Lifecycle).

### 0.2 Children

| Child ID | Name | Node type | Group membership |
|---|---|---|---|
| `F-SRV-002-01` | Slot Discovery | LEAF | mandatory |
| `F-SRV-002-02` | Booking Lifecycle | LEAF | mandatory |
| `F-SRV-002-03` | Waitlist Management | LEAF | optional |
| `F-SRV-002-04` | No-Show Handling | LEAF | optional |

### 0.3 Position in the Feature Tree

```
SRV_UI                                            [suite root]
├── F-SRV-001  Service Catalog Management         [COMPOSITION]
├── F-SRV-002  Appointment & Booking              [COMPOSITION] ← you are here
│   ├── F-SRV-002-01  Slot Discovery              [LEAF] [mandatory]
│   ├── F-SRV-002-02  Booking Lifecycle           [LEAF] [mandatory]
│   ├── F-SRV-002-03  Waitlist Management         [LEAF] [optional]
│   └── F-SRV-002-04  No-Show Handling            [LEAF] [optional]
├── F-SRV-003  Resource Scheduling                [COMPOSITION]
├── F-SRV-004  Session Execution                  [COMPOSITION]
├── F-SRV-005  Case Management                    [COMPOSITION]
├── F-SRV-006  Entitlements                       [COMPOSITION]
└── F-SRV-007  Billing Intent                     [COMPOSITION]
```

---

## SS1. Children & Variability Structure

### 1.1 Group Logic

**Group type:** mixed (mandatory + optional)

**Business rationale for this structure:**

The two mandatory children represent the irreducible core of appointment-based service delivery. Slot Discovery is the entry point for any booking flow — without it, neither the customer-facing self-service portal nor the back-office scheduler can find available times. Booking Lifecycle manages the subsequent state machine (propose → reserve → confirm → reschedule → cancel). These are inseparable because a booking without slot discovery is blind, and slot discovery without booking is a dead end.

Waitlist Management is optional because not all service contexts require it. High-demand appointment settings (clinics, driving schools with limited instructors) benefit from automated waitlisting; low-demand or B2B scheduling contexts do not need it and would add unnecessary complexity.

No-Show Handling is optional because its value depends on whether the deployment enforces fee policies. Products without no-show fees (e.g., internal resource scheduling for enterprise training) can omit it. When included, it integrates with `srv.bil` to trigger fee intents.

### 1.2 Sub-Groups (if mixed)

| Sub-group | Children | Group type | Rationale |
|---|---|---|---|
| Core Booking | `F-SRV-002-01`, `F-SRV-002-02` | mandatory | Minimum viable booking capability — inseparable |
| Operational Add-ons | `F-SRV-002-03`, `F-SRV-002-04` | optional | Value depends on deployment context |

### 1.3 Rationale for Tree Position

This composition exists at the suite root level because appointment & booking is a **primary capability area** of service delivery, not a sub-concern of another capability. Merging it into Resource Scheduling would conflate availability queries (a data concern) with booking transactions (a customer-facing workflow). Splitting it further (e.g., "Discovery" and "Booking" as separate compositions) would fragment a naturally sequential user journey that most products will include in full.

---

## SS2. Constraints

### 2.1 Intra-Node Cross-Tree Constraints

| If | Then | Rationale |
|---|---|---|
| `F-SRV-002-03` (Waitlist) is included | `F-SRV-002-01` (Slot Discovery) must be included | Waitlist entries are created when slot discovery returns no availability — discovery is the entry point |
| `F-SRV-002-04` (No-Show) is included | `F-SRV-002-02` (Booking Lifecycle) must be included | No-show is a terminal state of a booking — requires the booking lifecycle |

**Note:** Both intra-node constraints above are already satisfied by the mandatory group on 01 and 02. They are documented here for completeness and to make the business reasoning explicit.

### 2.2 Cross-Node Constraints

| Child | Requires / Excludes | External feature | Rationale |
|---|---|---|---|
| `F-SRV-002-01` | requires | `F-SRV-003` (Resource Scheduling) | Slot discovery queries resource availability from `srv.res` |
| `F-SRV-002-02` | requires | `F-SRV-001` (Service Catalog) | Booking references active service offerings from `srv.cat` |
| `F-SRV-002-04` | requires | `F-SRV-007` (Billing Intent) | No-show triggers fee intent derivation in `srv.bil` |

### 2.3 Constraint Summary (UVL Reference)

**Companion file:** `F-SRV-002.uvl`
**Constraint count:** 2 intra-node (implicit via mandatory) + 3 cross-node
**Last validated:** 2026-04-02

---

## SS3. Valid & Invalid Configurations

### 3.1 Valid Configurations

| Configuration name | Selected leaves | Use case |
|---|---|---|
| Full Booking | `01`, `02`, `03`, `04` | High-demand service with fees: driving school, clinic, salon |
| Core Booking | `01`, `02` | Minimum booking capability: internal scheduling, low-demand B2B |
| Booking with Waitlist | `01`, `02`, `03` | High-demand without fee enforcement: popular training courses |
| Booking with No-Show | `01`, `02`, `04` | Fee-enforcing context without waitlist: private practice |

### 3.2 Invalid Configurations

| Invalid combination | Violated constraint | Why invalid |
|---|---|---|
| `03` without `01` and `02` | mandatory group | Waitlist without core booking is meaningless |
| `04` without `01` and `02` | mandatory group | No-show handling without booking lifecycle has nothing to act on |
| Only `01` without `02` | mandatory group | Slot discovery without booking is a dead-end for the user |
| Only `02` without `01` | mandatory group | Blind booking without slot discovery violates user experience |

### 3.3 Edge Cases

| Configuration | Status | Notes |
|---|---|---|
| Core Booking only (`01`, `02`) without `F-SRV-003` | Invalid | Slot discovery depends on resource availability — cross-node constraint |
| Full Booking without `F-SRV-007` | Valid but incomplete | No-show feature works but fee intents won't be derived — add `F-SRV-007` |

---

## SS4. Change Log

### 4.1 Open Questions

| ID | Question | Impact | Owner | Needed by |
|---|---|---|---|---|
| Q-001 | Should recurring appointments (series bookings) be a separate leaf or an attribute on Booking Lifecycle? | If separate leaf: new spec needed. If attribute: variability point on F-SRV-002-02 | TBD | Phase 2 |
| Q-002 | Should group bookings (multiple customers, one slot) be modelled here or in a separate composition? | Affects data model and UI journey for F-SRV-002-02 | TBD | Phase 2 |

### 4.2 Decision Log

#### D-001: Waitlist and No-Show as Optional Leaves

**Decided:** 2026-04-02 by Architecture Team
**Decision:** Waitlist Management and No-Show Handling are optional leaves, not mandatory.
**Rationale:** Not all deployment contexts require these capabilities. Making them optional avoids unnecessary complexity for simpler products.
**Rejected alternatives:** (a) Making all 4 leaves mandatory — over-constrains minimal products. (b) Separate composition nodes — fragments a cohesive booking capability.

#### D-002: No-Show as Booking Feature, Not Session Feature

**Decided:** 2026-04-02 by Architecture Team
**Decision:** No-show handling is a child of Appointment & Booking (F-SRV-002), not Session Execution (F-SRV-004).
**Rationale:** No-show is determined at booking time (customer didn't show for the appointment). The session is never started. The booking aggregate owns the `NO_SHOW` state transition.
**Rejected alternatives:** Session-level no-show tracking — rejected because sessions are created only for attended appointments; a no-show means no session exists.

### 4.3 Version History

| Date | Version | Author | Changes |
|------|---------|--------|---------|
| 2026-04-02 | 1.0 | OpenLeap Architecture Team | Initial composition node |

---

## Review & Approval

**Status:** DRAFT

**Reviewers:**
- Suite UX Lead: {Name} — {Date}
- Suite Architect: {Name} — {Date}

**Approval:**
- Suite Architect: {Name} — {Date} — [ ] Approved
