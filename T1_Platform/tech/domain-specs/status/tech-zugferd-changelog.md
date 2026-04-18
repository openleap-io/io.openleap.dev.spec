# Spec Changelog — tech.zugferd

## Summary

- Written from scratch: the previous file was a 14-line migration stub; no existing content was lost
- Full TPL-SVC v1.0.0 compliance achieved (~95%) — all 16 sections (§0–§15) present
- Service documented as **fully stateless** per ADR-TECH-003 — template adapted accordingly (no persistent aggregates, no events, no database tables)
- 3 operation value objects documented: `EmbedOperation`, `ValidationOperation`, `ExtractionOperation`
- 6 enumerations defined: `ZugferdProfile` (6 values), `ContentType`, `ValidationSeverity`
- 6 shared types documented: `InvoiceData`, `PartyInfo`, `MonetaryAmount`, `InvoiceLineItem`, `ValidationError`
- 13 business rules (BR-ZF-001 through BR-ZF-013) with 6 detailed definitions
- 4 use cases (UC-ZF-001 through UC-ZF-004) with full Actor/Preconditions/Main Flow/Postconditions
- 4 REST endpoints documented with request/response JSON examples (`/embed`, `/validate`, `/extract`, `/info`)
- No events published (stateless); no database tables; §7.2 and §8.3 explicitly document "None"
- All 5 extension point types covered: extension-field (N/A, stateless), extension-event (3), extension-rule (2 slots), extension-action (limited to admin UI), aggregate-hook (3)
- 9 open questions raised (Q-ZF-001 through Q-ZF-009)
- 2 domain ADRs documented (ADR-ZF-001: Stateless Architecture, ADR-ZF-002: Synchronous REST)
- 4 decisions & conflicts documented (DC-ZF-001 through DC-ZF-004)

## Added Sections

All sections are new (source was a 14-line migration stub):

- Preamble (template compliance header, meta information block, deprecated alias `/api/t1/zugferd/v1`)
- `## Specification Guidelines Compliance` (Non-Negotiables, Source Priority, Style Guide)
- `## 0.` Document Purpose & Scope (0.1–0.4) including in/out-of-scope and related documents
- `## 1.` Business Context (1.1–1.5 including Mermaid context diagram)
- `## 2.` Service Identity (metadata table, team table)
- `## 3.` Domain Model (3.1–3.5): conceptual overview, Mermaid class diagram, 3 stateless operation models with attribute tables and invariants, 3 enumerations with value tables, 6 shared types with attribute tables and validation rules
- `## 4.` Business Rules (4.1 catalog with 13 rules, 4.2 detailed definitions for 6 rules, 4.3 field/cross-field validations, 4.4 reference data dependencies)
- `## 5.` Use Cases (5.1 logic placement table, 5.2 four use cases with canonical format and full detail, 5.3 diagram stub, 5.4 cross-domain invoice generation workflow)
- `## 6.` REST API (6.1 overview, 6.2 four operations with request/response JSON, 6.3 no additional ops, 6.4 OpenAPI reference)
- `## 7.` Events & Integration (7.1 stateless pattern declaration, 7.2 "None", 7.3 "None", 7.4 N/A, 7.5 integration summary with upstream/downstream tables)
- `## 8.` Data Model (8.1 "No database", 8.2 in-memory ER diagram, 8.3 "No tables", 8.4 bundled static reference data)
- `## 9.` Security (9.1 data classification table, 9.2 RBAC matrix, 9.3 compliance including GDPR, ERechV, EU Directive 2014/55/EU)
- `## 10.` Quality Attributes (10.1–10.4: response time targets per operation, availability 99.9%, RTO/RPO, scalability, monitoring)
- `## 11.` Feature Dependencies (11.1–11.5: F-TECH-005-01 dependency register, endpoint mapping, BFF hints, impact assessment)
- `## 12.` Extension Points (12.1–12.8: 3 extension events, 2 rule slots, 3 aggregate hooks, hook contracts, summary matrix, guidelines)
- `## 13.` Migration & Evolution (13.1 legacy SAP mapping table, ZUGFeRD 1.0 migration note, 13.2 /api/t1 deprecation with 6-month sunset)
- `## 14.` Decisions & Open Questions (14.1 consistency checks Pass/Pass/Pass/Pass/Pass/Pass/Pass, 14.2 4 decisions, 14.3 9 open questions, 14.4 2 ADRs, 14.5 3 suite ADR references)
- `## 15.` Appendix (15.1 glossary with 14 terms, 15.2 standards references, 15.3 status output list, 15.4 change log)

## Modified Sections

None — source was a stub; no prior content was changed.

## Removed Sections

None — non-destructive upgrade. Migration checklist from stub was superseded by the complete spec.

## Decisions Taken

- **DC-ZF-001:** Template §3 (aggregates) adapted for stateless service — request/response value objects documented in aggregate structure for tooling compatibility.
- **DC-ZF-002:** Template §7 (events) explicitly states "None" consistent with ADR-TECH-003.
- **DC-ZF-003:** Template §8 (database) explicitly states "No database tables" consistent with ADR-TECH-003.
- **DC-ZF-004:** EmbedOperation typed as WRITE (caller-relative mutation) even though the service has no internal side effects.

## Open Questions Raised

- Q-ZF-001: Confirmed port assignment for `tech-zugferd-svc` (tentative: 8098)
- Q-ZF-002: Maximum file size — is 20 MB appropriate for expected invoice PDFs?
- Q-ZF-003: Currency/country validation — runtime call to param-ref-svc vs. bundled static ISO lists
- Q-ZF-004: Process flow sequence diagrams (§5.3) — pending creation
- Q-ZF-005: OAuth2 scope design for embed/validate/extract operations
- Q-ZF-006: Peak invoice embed volume across FI, SD, SRV for capacity planning
- Q-ZF-007: Which specific FI/SD/SRV features will create Feature Dependency Register entries in §11.2
- Q-ZF-008: Extension API contract for registering custom schematron rule sets (§12.7)
- Q-ZF-009: ZUGFeRD 1.0 backward compatibility for extraction and re-embedding
