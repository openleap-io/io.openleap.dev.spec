# Open Questions - fi.clr

## Q-CLR-001: Canonical Event Names for Vouchers
- **Question:** What is the canonical event name set for vouchers in FI v2.1?
- **Why it matters:** Affects consumed event handler routing keys in fi.clr. Current assumption is `fi.bnk.voucher.created` but this needs alignment with fi.bnk spec.
- **Suggested options:** Align with fi.bnk spec authors; confirm routing key pattern follows suite convention.
- **Owner:** Architecture Team

## Q-CLR-002: Manual Override Security Model
- **Question:** How are manual overrides secured (role + approval workflow)?
- **Why it matters:** Clearing is a financial control. Manual link creation and rejection bypass automatic matching and could be exploited. SOX compliance may require dual-approval for certain thresholds.
- **Suggested options:** 1) Role only (FI_CLR_OPERATOR), 2) Role + second-approver for amounts above threshold, 3) Configurable per tenant
- **Owner:** Finance Compliance

## Q-CLR-003: Port Assignment
- **Question:** Port assignment for fi-clr-svc.
- **Why it matters:** Deployment planning and service registry configuration.
- **Suggested options:** Follow platform port registry convention.
- **Owner:** Architecture Team

## Q-CLR-004: Partial Case Auto-Escalation
- **Question:** Should PARTIAL cases auto-escalate after N days without closure?
- **Why it matters:** Without escalation, partial cases may accumulate indefinitely, creating reconciliation discrepancies at period-end and inflating the PARTIAL backlog.
- **Suggested options:** 1) No auto-escalation (operator responsibility), 2) Configurable TTL + notification event, 3) Automatic closure as PARTIAL after TTL
- **Owner:** Product Owner

## Q-CLR-005: Feature Dependency Register
- **Question:** Which product features depend on this service's endpoints?
- **Why it matters:** Section 11 Feature Dependencies requires feature spec IDs to properly track blast radius of API changes. Currently all IDs are TBD.
- **Suggested options:** Populate after Phase 3 feature spec authoring for the FI suite.
- **Owner:** Product Owner

## Q-CLR-006: Multi-Currency Clearing
- **Question:** Should ClearingCase support multi-currency clearing (voucher in one currency, open item in another)?
- **Why it matters:** Cross-currency payments are common in international trade. Without support, foreign currency payments cannot be automatically matched.
- **Suggested options:** 1) Single currency only for v1 (simpler, defer complexity), 2) Add FX tolerance field to MatchingRule with exchange rate lookup, 3) Separate cross-currency clearing as a future domain extension
- **Owner:** Architecture Team

## Q-CLR-007: Custom Fields Size Limit
- **Question:** What is the maximum custom_fields JSONB size limit per ClearingCase?
- **Why it matters:** Large JSONB values impact query performance and storage costs. GIN index performance degrades with very large documents.
- **Suggested options:** 1) 10KB default (platform standard), 2) Configurable per tenant via extension API
- **Owner:** Architecture Team

## Q-CLR-008: Customer/Vendor-Based Matching
- **Question:** Should MatchingRule support customer/vendor-based matching criteria?
- **Why it matters:** Recurring payments from the same customer often have consistent patterns. Adding customer/vendor matching would significantly improve auto-match accuracy.
- **Suggested options:** 1) Add customerMatch boolean field to MatchingRule (requires open item to carry business partner reference), 2) Defer to ML-based scoring in future extension, 3) Add as MatchCriteria enum value with rule configuration
- **Owner:** Product Owner
