<!-- TEMPLATE COMPLIANCE: ~5%
Template: suite-spec.md v1.0.0 (or domain-service-spec.md — unclear; file acts as both suite and domain placeholder)
Present sections: none formally — only naming conventions for API path, exchange, routing keys
Missing sections: §0-§11 (all suite sections) or §0-§15 (all domain sections) — this is a stub placeholder
Naming issues: file should be _hr_suite.md (if suite spec) or hr_core-spec.md (if domain spec); uppercase "HR_core.md" is non-standard
Duplicates: none
Priority: HIGH — placeholder with no substantive content; needs full spec authoring
-->
# Human Resources (HR) — Core Specification (Placeholder)

> **Meta Information**
> - **Version:** 2025-12-05
> - **Template:** `suite-spec.md` v1.0.0
> - **Template Compliance:** ~5% — §0-§7 all missing (stub placeholder)
> - **Author(s):** OpenLeap Architecture Team
> - **Status:** STUB

This file is a placeholder for HR Core specs. Conventions:
- API base path: `/api/hr/hr/v1` (or `/api/hr/core/v1`)
- Exchange: `hr.hr.events`
- Routing keys: `hr.hr.<aggregate>.<event>` e.g., `hr.hr.employee.hired`, `hr.hr.employee.terminated`.

To be completed.
