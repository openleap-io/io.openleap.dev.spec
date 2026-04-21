            # F-SHARED-CAP-001-01 — Resource Calendars

            > **Conceptual Stack Layer:** Platform-Feature
            > **Space:** Platform
            > **Owner:** Domain Engineering Team
            > **Companion files:** `F-SHARED-CAP-001-01.uvl`, `F-SHARED-CAP-001-01.aui.yaml`

            > **Meta Information**
            > - **Version:** 2026-04-03
            > - **Template:** `feature-spec.md` v1.0.0
            > - **Template Compliance:** 100% — fully compliant
            > - **Author(s):** OpenLeap Architecture Team
            > - **Status:** DRAFT
            > - **Feature ID:** `F-SHARED-CAP-001-01`
            > - **Suite:** `t2`
            > - **Node type:** LEAF
            > - **Parent:** `F-SHARED-CAP-001` — see `F-SHARED-CAP-001.md`
            > - **Companion UVL:** `F-SHARED-CAP-001-01.uvl`
            > - **Companion AUI:** `F-SHARED-CAP-001-01.aui.yaml`

            ---

            ## ═══════════════════════════════════════════════
            ## PROBLEM SPACE
            ## ═══════════════════════════════════════════════

            ## 0. Feature Identity & Orientation

            ### 0.1 One-Line Summary

            This feature lets authorized users create and configure resource or organizational calendars with timezone and holiday inheritance so that availability can be computed .

            ### 0.2 Non-Goals

            - Does not implement domain-specific business logic on the data — that belongs to T3 domain extensions.
            - Does not provide analytics or reporting — that belongs to T4 BI.

            ### 0.3 Entry & Exit Points

            **Entry points:**
            - From the Calendar Management navigation menu
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
            F-SHARED-CAP-001  Calendar Management  [COMPOSITION]
            ├── F-SHARED-CAP-001-01  Resource Calendars  [LEAF] <-- you are here
├── F-SHARED-CAP-001-02  Working Patterns  [LEAF]
├── F-SHARED-CAP-001-03  Holiday Management  [LEAF]
├── F-SHARED-CAP-001-04  Availability Query  [LEAF]
```

            ### 0.6 Related Documents

            | Document | What to find there |
            |---|---|
            | `F-SHARED-CAP-001.md` | Parent composition — variability structure |
            | `F-SHARED-CAP-001-01.uvl` | Companion UVL — attribute schema, cross-suite requires |
            | `F-SHARED-CAP-001-01.aui.yaml` | Companion AUI — screen contract |
            | `_shared_suite.md` §6 | Suite Feature Catalog |
            | `domain-specs/shared_cap-spec.md` | Backend: business rules, API contracts |

            ---

            ## 1. User Goal & Scenarios

            ### 1.1 The User Goal

            Create and configure resource or organizational calendars with timezone and holiday inheritance so that availability can be computed.

            ### 1.2 User Scenarios


**Scenario 1: Create Machine Calendar**
> A Production Planner wants to create a calendar for a CNC machine because a new machine was installed on the shopfloor.
> They navigate to the Resource Calendars screen, fill in the required data, and submit.
> They expect: The calendar is created with MACHINE kind and inheritsHolidays=true.


**Scenario 2: Create Org Calendar**
> A Plant Manager wants to create a plant-level calendar because the plant needs its own shutdown schedule.
> They navigate to the Resource Calendars screen, fill in the required data, and submit.
> They expect: An ORGANIZATION calendar is created and linked to the plant party.



            ---

            ## 2. User Journey & Screen Layout

            ### 2.1 Happy-Path Flow

            ```mermaid
            sequenceDiagram
                actor User as Authorized User
                participant UI as Resource Calendars UI
                participant BFF as BFF
                participant SVC as shared-cap-svc

                User->>UI: Navigate to Resource Calendars
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

            **Screen: Resource Calendars — Default State**

            ```
            ┌─────────────────────────────────────────────────┐
            │  [Calendar Management] > Resource Calendars              │  ← Breadcrumb
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

            Fields are defined authoritatively in `domain-specs/shared_cap-spec.md` §3 (Domain Model) and §6 (REST API). This feature renders the fields relevant to resource calendars.

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

            See companion file: `contracts/aui/F-SHARED-CAP-001-01.aui.yaml`

            ---

            ## ═══════════════════════════════════════════════
            ## BRIDGE ARTIFACTS
            ## ═══════════════════════════════════════════════

            ## 7. i18n, Permissions & Accessibility

            ### 7.1 Permission Matrix

            | Role | Read | Create | Update | Delete |
            |---|---|---|---|---|
            | CAP_VIEWER | Y | N | N | N |
| CAP_USER | Y | Y | Y | N |
| CAP_ADMIN | Y | Y | Y | Y |


            ### 7.2 Accessibility

            - All form fields MUST have associated labels (WCAG 2.1 AA).
            - Keyboard navigation MUST be supported for all actions.
            - Error messages MUST be linked to their fields via aria-describedby.

            ---

            ## 8. Acceptance Criteria


            **AC-1: Create Machine Calendar**
            - **Given** a Production Planner is on the Resource Calendars screen
            - **When** they create a calendar for a CNC machine
            - **Then** The calendar is created with MACHINE kind and inheritsHolidays=true.

            **AC-2: Create Org Calendar**
            - **Given** a Plant Manager is on the Resource Calendars screen
            - **When** they create a plant-level calendar
            - **Then** An ORGANIZATION calendar is created and linked to the plant party.


            ---

            ## 9. Dependencies, Variability & Extension Points

            ### 9.1 Feature Dependencies

            | This Feature | Requires | Reason |
            |---|---|---|
            | `F-SHARED-CAP-001-01` | T1 Reference Data | Validation of codes |

            ### 9.2 Variability Points

            See §0.4 above.

            ### 9.3 Extension Points

            | Extension Point ID | Type | Description | Default Behavior |
            |---|---|---|---|
            | `ext.f_t2_002_01.customFields` | zone | Additional fields from T3 domains | Zone hidden |

            ---

            ## 10. Change Log & Review

            ### 10.1 Change Log

            | Date | Version | Author | Changes |
            |---|---|---|---|
            | 2026-04-03 | 1.0.0 | OpenLeap Architecture Team | Initial leaf feature spec |

            ### 10.2 Review & Approval

            **Status:** DRAFT
