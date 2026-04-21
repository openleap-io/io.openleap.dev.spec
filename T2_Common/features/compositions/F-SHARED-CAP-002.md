        # F-SHARED-CAP-002 --- Booking Management

        > **Conceptual Stack Layer:** Platform-Feature (Composition Node)
        > **Space:** Platform
        > **Owner:** Domain Engineering Team
        > **Companion files:** F-SHARED-CAP-002.uvl

        > **Meta Information**
        > - **Version:** 2026-04-03
        > - **Template:** `feature-composition-spec.md` v1.0.0
        > - **Template Compliance:** 100% — fully compliant
        > - **Author(s):** OpenLeap Architecture Team
        > - **Status:** DRAFT
        > - **Feature ID:** `F-SHARED-CAP-002`
        > - **Suite:** `t2`
        > - **Node type:** COMPOSITION
        > - **Parent:** suite root (T2_UI)
        > - **Companion UVL:** `F-SHARED-CAP-002.uvl`

        ---

        ## SS0. Identity

        ### 0.1 Purpose

        This composition node groups all capabilities related to Booking Management. This is an optional composition that products may include based on their operational requirements.

        ### 0.2 Children

        | Child ID | Name | Node type | Group membership |
        |---|---|---|---|
        | `F-SHARED-CAP-002-01` | Slot & Booking | LEAF | mandatory |
| `F-SHARED-CAP-002-02` | Booking Templates | LEAF | optional |

        ### 0.3 Position in the Feature Tree

        ```
        T2_UI  (suite root)
        +-- F-SHARED-CAP-002  Booking Management  <-- you are here
            +-- F-SHARED-CAP-002-01  Slot & Booking  [LEAF] [mandatory]
    +-- F-SHARED-CAP-002-02  Booking Templates  [LEAF] [optional]
```

        ---

        ## SS1. Children & Variability Structure

        ### 1.1 Group Logic

        **Group type:** mixed

        **Business rationale for this structure:**

        The mandatory children represent core primitives that are inseparable — including any part of Booking Management without the others would leave the capability incomplete. Optional children extend the core with additional functionality that not every product requires.

        ### 1.3 Rationale for Tree Position

        This composition node exists because booking management is a cohesive capability area with its own bounded context within T2. Promoting its children to the suite root would lose the grouping semantics and make product configuration harder. Decomposing further would create artificial separation between tightly related features.

        ---

        ## SS2. Constraints

        ### 2.1 Intra-Node Cross-Tree Constraints

        | If | Then | Rationale |
        |---|---|---|
        | — | — | No intra-node constraints |

        ### 2.2 Cross-Node Constraints

        | This Node | Requires | Rationale |\n|---|---|---|\n| `F-SHARED-CAP-002` | `F-SHARED-CAP-001` | Bookings require calendars for slot computation |

        ### 2.3 Edge Cases

        - If the optional child `F-SHARED-CAP-002-02` is excluded, related data entry fields are hidden in the mandatory children's screens.

        ---

        ## SS3. Quality & Testing Guidance

        - Integration tests MUST verify that selecting this composition includes all mandatory children.
        - Optional children must be testable in isolation (when the rest of the composition is present).
        - UVL solver validation MUST pass for all valid product configurations that include this node.

        ---

        ## SS4. Change Log

        ### 4.1 Open Questions

        | ID | Question | Impact | Owner | Needed by |
        |---|---|---|---|---|
        | — | No open questions | — | — | — |

        ### 4.2 Decision Log

        **D-001: Feature tree structure for Booking Management**
        - **Decided:** 2026-04-03 by Architecture Team
        - **Decision:** Group children as shown above with mixed grouping
        - **Rationale:** Matches the bounded context model and domain spec structure

        ### 4.3 Change Log

        | Date | Version | Author | Changes |
        |---|---|---|---|
        | 2026-04-03 | 1.0.0 | OpenLeap Architecture Team | Initial composition spec |
