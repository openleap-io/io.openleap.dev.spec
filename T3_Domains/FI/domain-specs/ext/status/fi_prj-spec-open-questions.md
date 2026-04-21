# Open Questions - fi.prj

## Q-PRJ-001: Feature Dependency Register
- **Question:** Which product features formally depend on fi-prj-svc endpoints?
- **Why it matters:** The Feature Dependency Register (§11.2) needs concrete feature IDs from feature specs. Without this, BFF specs and impact assessments cannot be completed.
- **Suggested options:** Create feature specs F-FI-PRJ-001 (Project Cost Tracking), F-FI-PRJ-002 (WIP & Revenue Recognition), F-FI-PRJ-003 (Project Billing), F-FI-PRJ-004 (Project Capitalization)
- **Owner:** Product Team

## Q-PRJ-002: Multi-Currency Project Support
- **Question:** Should fi.prj support multi-currency projects (costs in different currencies than project currency)?
- **Why it matters:** Affects POC calculation (currency conversion timing), WIP aggregation, data model (need exchange rate storage), and reporting. International projects commonly incur costs in multiple currencies.
- **Suggested options:** A) Single currency only — convert at capture, B) Multi-currency with conversion at capture time (simpler), C) Multi-currency with conversion at WIP run time (more accurate, more complex)
- **Owner:** Architecture Team

## Q-PRJ-003: Port Assignment
- **Question:** What is the exact port assignment for fi-prj-svc?
- **Why it matters:** Needed for service registry, local development setup, and infrastructure configuration. Port 8480 is provisional.
- **Suggested options:** Confirm 8480 or assign based on FI suite port range convention
- **Owner:** DevOps Team

## Q-PRJ-004: Overhead Allocation Automation
- **Question:** Should overhead allocation to projects be an automated batch process or manual entry only?
- **Why it matters:** Overhead allocation is a standard project accounting process (SAP PS uses periodic overhead calculation via COGI/KGI2). Automation would require a scheduled job and allocation rules engine.
- **Suggested options:** A) Manual entry only (source = MANUAL, costType = OVERHEAD), B) Automated allocation via scheduled job with configurable rates, C) Both manual and automated
- **Owner:** Business Analysts

## Q-PRJ-005: Cost Reversal Handling
- **Question:** How should fi.prj handle cost reversals (e.g., timesheet correction, AP bill reversal)?
- **Why it matters:** Upstream systems may reverse previously posted costs. fi.prj needs a mechanism to reduce WIP without violating the amount > 0 constraint (BR-COST-001). This affects the CostLine data model and WIP calculation.
- **Suggested options:** A) Allow negative CostLine amounts (simplest, but breaks BR-COST-001), B) Create separate CostReversal entity with negative amounts, C) Link reversal CostLine to original via sourceDocId with special source type (e.g., REVERSAL)
- **Owner:** Architecture Team

## Q-PRJ-006: WIP Run Project Filtering
- **Question:** Should WIP run support project-level filtering (calculate WIP for a subset of projects)?
- **Why it matters:** Large organizations may have thousands of projects. Running WIP for all projects at once may be too slow or operationally impractical. Some organizations may need to run WIP by business unit or project group.
- **Suggested options:** A) Always calculate for all active projects (current design), B) Optional project filter (by projectType, entityId, or explicit project list), C) Introduce project group concept for batch processing
- **Owner:** Business Analysts
