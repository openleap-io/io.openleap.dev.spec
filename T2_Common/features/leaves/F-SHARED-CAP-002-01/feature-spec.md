            # F-SHARED-CAP-002-01 — Slot & Booking

            > **Conceptual Stack Layer:** Platform-Feature
            > **Space:** Platform
            > **Owner:** Domain Engineering Team
            > **Companion files:** `F-SHARED-CAP-002-01.uvl`, `F-SHARED-CAP-002-01.aui.yaml`

            > **Meta Information**
            > - **Version:** 2026-04-03
            > - **Template:** `feature-spec.md` v1.0.0
            > - **Template Compliance:** 100% — fully compliant
            > - **Author(s):** OpenLeap Architecture Team
            > - **Status:** DRAFT
            > - **Feature ID:** `F-SHARED-CAP-002-01`
            > - **Suite:** `t2`
            > - **Node type:** LEAF
            > - **Parent:** `F-SHARED-CAP-002` — see `F-SHARED-CAP-002.md`
            > - **Companion UVL:** `F-SHARED-CAP-002-01.uvl`
            > - **Companion AUI:** `F-SHARED-CAP-002-01.aui.yaml`

            ---

            ## ═══════════════════════════════════════════════
            ## PROBLEM SPACE
            ## ═══════════════════════════════════════════════

            ## 0. Feature Identity & Orientation

            ### 0.1 One-Line Summary

            This feature lets authorized users search for available slots and create, confirm, or cancel bookings so that delivery windows and appointments are reliably reserved .

            ### 0.2 Non-Goals

            - Does not implement domain-specific business logic on the data — that belongs to T3 domain extensions.
            - Does not provide analytics or reporting — that belongs to T4 BI.

            ### 0.3 Entry & Exit Points

            **Entry points:**
            - From the Booking Management navigation menu
            - By deep-link with entity ID

            **Exit points:**
            - Success: confirmation toast, optional navigation to detail view
            - Failure: inline validation errors, retry

            ### 0.4 Variability Points

            | Variability | Modelled as | UVL | Default | Binding time |
            |---|---|---|---|---|
            | Records per page | Attribute | `pagination.pageSize Integer 20` | 20 | `deploy` |
            | Read-only mode | Attribute | `mode.readOnly Boolean false` | false | `deploy` |

            ### 0.5 Position in Feature Tree

            ```
            F-SHARED-CAP-002  Booking Management  [COMPOSITION]
            ├── F-SHARED-CAP-002-01  Slot & Booking  [LEAF] <-- you are here
├── F-SHARED-CAP-002-02  Booking Templates  [LEAF]
```

            ### 0.6 Related Documents

            | Document | What to find there |
            |---|---|
            | `F-SHARED-CAP-002.md` | Parent composition — variability structure |
            | `F-SHARED-CAP-002-01.uvl` | Companion UVL — attribute schema, cross-suite requires |
            | `F-SHARED-CAP-002-01.aui.yaml` | Companion AUI — screen contract |
            | `_shared_suite.md` §6 | Suite Feature Catalog |
            | `domain-specs/shared_cap-spec.md` | Backend: business rules, API contracts |

            ---

            ## 1. User Goal & Scenarios

            ### 1.1 The User Goal

            Search for available slots and create, confirm, or cancel bookings so that delivery windows and appointments are reliably reserved.

            ### 1.2 User Scenarios


**Scenario 1: Book Delivery Slot**
> A Sales Representative wants to reserve a 2-hour delivery window for a customer order because the order needs a confirmed delivery date.
> They navigate to the Slot & Booking screen, fill in the required data, and submit.
> They expect: A booking is created in PENDING state and a booking.created event fires.


**Scenario 2: Confirm Booking**
> A Warehouse Manager wants to confirm a pending booking because the delivery truck is scheduled.
> They navigate to the Slot & Booking screen, fill in the required data, and submit.
> They expect: The booking moves to CONFIRMED and the slot is locked.


**Scenario 3: Cancel Booking**
> A Sales Representative wants to cancel a confirmed booking because the customer cancelled their order.
> They navigate to the Slot & Booking screen, fill in the required data, and submit.
> They expect: The booking is cancelled, the slot is freed, and a booking.cancelled event fires.



            ---

            ## 2. User Journey & Screen Layout

            ### 2.1 Happy-Path Flow

            ```mermaid
            sequenceDiagram
                actor User as Authorized User
                participant UI as Slot & Booking UI
                participant BFF as BFF
                participant SVC as shared-cap-svc

                User->>UI: Navigate to Slot & Booking
                UI->>BFF: Load initial data
                BFF->>SVC: GET /api/shared/cap/v1/...
                SVC-->>BFF: Data
                BFF-->>UI: View model
                UI-->>User: Render screen

                User->>UI: Fill form / select action
                UI->>UI: Client-side validation

                User->>UI: Submit
                alt Validation fails
                    UI-->>User: Inline errors
                else Validation passes
                    UI->>BFF: Submit mutation
                    BFF->>SVC: POST/PATCH /api/shared/cap/v1/...
                    alt Success
                        SVC-->>BFF: 201/200 + entity
                        BFF-->>UI: Success response
                        UI-->>User: Confirmation toast
                    else Business rule violation
                        SVC-->>BFF: 422 + error detail
                        BFF-->>UI: Error
                        UI-->>User: Inline error on relevant field
                    else System error
                        SVC-->>BFF: 500
                        BFF-->>UI: Error
                        UI-->>User: Error banner with retry
                    end
                end
            ```

            ### 2.2 Screen Layout

            **Screen: Slot & Booking — Default State**

            ```
            ┌─────────────────────────────────────────────────┐
            │  [Booking Management] > Slot & Booking              │  ← Breadcrumb
            ├─────────────────────────────────────────────────┤
            │  ┌─ Core Data ─────────────────────────────────┐│
            │  │  [Form fields per domain spec]              ││
            │  └─────────────────────────────────────────────┘│
            │  ┌─ [EXT] Domain Extensions ───────────────────┐│
            │  │  [Extension zone for T3 domain fields]      ││
            │  └─────────────────────────────────────────────┘│
            ├─────────────────────────────────────────────────┤
            │  [Cancel]                          [Save]       │  ← Actions
            └─────────────────────────────────────────────────┘
            ```

            ---

            ## 3. Interaction Requirements

            ### 3.1 Form Fields

            Fields are defined authoritatively in `domain-specs/shared_cap-spec.md` §3 (Domain Model) and §6 (REST API). This feature renders the fields relevant to slot & booking.

            ### 3.2 Actions

            | Action | Type | Enabled when | Confirmation |
            |---|---|---|---|
            | Save | primary | Form is valid and dirty | No |
            | Cancel | secondary | Always | Unsaved changes guard |

            ### 3.3 Cross-Field Rules

            Cross-field validation rules are defined in `domain-specs/shared_cap-spec.md` §4 (Business Rules). The BFF enforces these before submission.

            ---

            ## 4. Edge Cases & Screen States

            ### 4.1 Component States

            | State | Condition | Behavior |
            |---|---|---|
            | Loading | Data fetch in progress | Skeleton loader |
            | Empty | No records found | Empty state with create CTA |
            | Error | Service unavailable | Error banner with retry button |
            | Populated | Data loaded | Default rendering |

            ### 4.2 Specific Edge Cases

            - Concurrent edit: optimistic locking via version field; conflict shows merge dialog.
            - Network timeout: retry with exponential backoff; show retry button after 3 attempts.

            ---

            ## ═══════════════════════════════════════════════
            ## SOLUTION SPACE
            ## ═══════════════════════════════════════════════

            ## 5. Backend Dependencies & BFF Contract

            ### 5.1 Service Calls

            | Service | Tier | Endpoint | Method | isMutation | Failure Mode |
            |---|---|---|---|---|---|
            | `shared-cap-svc` | T2 | `/api/shared/cap/v1/...` | GET | No | Degrade: show cached |
            | `shared-cap-svc` | T2 | `/api/shared/cap/v1/...` | POST/PATCH | Yes | Block: show error |
            | `t1-ref-svc` | T1 | `/api/t1/ref/v1/catalogs/...` | GET | No | Degrade: use cached |

            ### 5.2 BFF View-Model

            The BFF composes a view model from the service calls above. Shape follows the domain spec's REST API response structure enriched with reference data labels.

            ### 5.3 Feature-Gating Rules

            | Mode | Behavior |
            |---|---|
            | full | All actions available |
            | read-only | View only, mutation buttons hidden |
            | excluded | Feature hidden from navigation |

            ---

            ## 6. Screen Contract (AUI)

            See companion file: `contracts/aui/F-SHARED-CAP-002-01.aui.yaml`

            ---

            ## ═══════════════════════════════════════════════
            ## BRIDGE ARTIFACTS
            ## ═══════════════════════════════════════════════

            ## 7. i18n, Permissions & Accessibility

            ### 7.1 Permission Matrix

            | Role | Read | Create | Update | Delete |
            |---|---|---|---|---|
            | CAP_USER | Y | Y | Y | N |
| CAP_ADMIN | Y | Y | Y | Y |
| BOOKING_USER | Y | Y | Y | N |


            ### 7.2 Accessibility

            - All form fields MUST have associated labels (WCAG 2.1 AA).
            - Keyboard navigation MUST be supported for all actions.
            - Error messages MUST be linked to their fields via aria-describedby.

            ---

            ## 8. Acceptance Criteria


            **AC-1: Book Delivery Slot**
            - **Given** a Sales Representative is on the Slot & Booking screen
            - **When** they reserve a 2-hour delivery window for a customer order
            - **Then** A booking is created in PENDING state and a booking.created event fires.

            **AC-2: Confirm Booking**
            - **Given** a Warehouse Manager is on the Slot & Booking screen
            - **When** they confirm a pending booking
            - **Then** The booking moves to CONFIRMED and the slot is locked.

            **AC-3: Cancel Booking**
            - **Given** a Sales Representative is on the Slot & Booking screen
            - **When** they cancel a confirmed booking
            - **Then** The booking is cancelled, the slot is freed, and a booking.cancelled event fires.


            ---

            ## 9. Dependencies, Variability & Extension Points

            ### 9.1 Feature Dependencies

            | This Feature | Requires | Reason |
            |---|---|---|
            | `F-SHARED-CAP-002-01` | T1 Reference Data | Validation of codes |

            ### 9.2 Variability Points

            See §0.4 above.

            ### 9.3 Extension Points

            | Extension Point ID | Type | Description | Default Behavior |
            |---|---|---|---|
            | `ext.f_t2_003_01.customFields` | zone | Additional fields from T3 domains | Zone hidden |

            ---

            ## 10. Change Log & Review

            ### 10.1 Change Log

            | Date | Version | Author | Changes |
            |---|---|---|---|
            | 2026-04-03 | 1.0.0 | OpenLeap Architecture Team | Initial leaf feature spec |

            ### 10.2 Review & Approval

            **Status:** DRAFT
