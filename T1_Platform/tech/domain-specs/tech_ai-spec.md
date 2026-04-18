<!-- Template Meta
     Template-ID:   TPL-SVC
     Version:       1.1.0
     Compliance:    95%+
-->

# AI Service — Domain Specification

> **Conceptual Stack Layer:** Domain / Service
> **Space:** Platform
> **Owner:** Platform Infrastructure Team

> **Meta Information**
> - **Version:** 2026-04-18
> - **Template:** `domain-service-spec.md` v1.1.0
> - **Template Compliance:** 95%+
> - **Status:** DRAFT (new service — no predecessor)
> - **Service ID:** `tech-ai-svc`
> - **Suite:** `tech`
> - **Domain:** `ai`
> - **Base Package:** `io.openleap.tech.ai`
> - **API Path:** `/api/tech/ai/v1`
> - **Port:** 8099
> - **DB Schema:** `tech_ai`
> - **Tier:** T1 — Platform & Technical Foundations

---

## 0. Document Purpose & Scope

This document specifies the **AI Service** (`tech-ai-svc`) as a T1 Platform infrastructure capability: a provider-agnostic LLM + embedding + classification abstraction with tenant-level controls (provider choice, data-residency, redaction).

**Audience:** Backend developers integrating AI capabilities, platform ops, data-protection officers.

**In scope:** LLM completion / chat, embedding generation, text classification, summarization task routing, prompt registry, rate limiting, cost attribution, safety / redaction filters, model-switch variability.
**Out of scope:** Model training, fine-tuning pipelines (→ T4 data / external), domain-specific prompt engineering (those live in the consuming feature specs).

**Related documents:**
- Suite Spec: `T1_Platform/tech/_tech_suite.md`
- OpenAPI: `contracts/http/tech/ai/openapi.yaml`
- Event Schemas: `contracts/events/tech/ai/*.schema.json`

---

## 1. Business Context

### 1.1 Purpose & Responsibility

Single point through which any suite invokes LLM-backed capabilities. Routes requests to a configured provider (Anthropic / OpenAI / Ollama / local / Azure-OpenAI), enforces tenant redaction rules, applies safety filters, meters usage, and emits cost-attribution events. Consumers declare a *task* (e.g. `summarize-ticket`) and parameters; the service resolves the right prompt, model, and provider.

**Owns:**
- AiTask (registered capability, e.g. `summarize-ticket`, `triage-incoming-message`, `draft-reply`, `suggest-kb-article`)
- AiPrompt (versioned prompt template per task + locale + model family)
- AiProviderConfig (per-tenant provider selection with credentials ref and region)
- AiInvocation (every call + input tokens + output tokens + latency + cost + redaction events)
- AiSafetyPolicy (tenant-level content policy: PII redaction, allow/deny topics, length caps)

### 1.2 Business Value

- **One abstraction:** features across suites share a uniform API for AI tasks; swapping providers is a configuration change.
- **Governance:** all AI calls are audited, usage-metered, and policy-filtered at one place.
- **Data control:** tenants with residency requirements pin their own provider (e.g., local Ollama) without feature specs having to change.

---

## 2. Service Identity

| Field | Value |
|-------|-------|
| Service ID | `tech-ai-svc` |
| Suite | `tech` |
| Domain | `ai` |
| Bounded Context | AI Inference |
| Base Package | `io.openleap.tech.ai` |
| API Base Path | `/api/tech/ai/v1` |
| Port | 8099 |
| Database Schema | `tech_ai` |
| Tier | T1 — Platform & Technical Foundations |
| Repository | `openleap-io/io.openleap.tech.ai` |
| Team | `team-platform-infra` |

---

## 3. Domain Model

### 3.1 Aggregates

**AiTask** (AggregateRoot)
- id, slug (e.g. `summarize-ticket`), description, inputSchema (JSON Schema), outputSchema, defaultPromptVersion, defaultModelFamily (`generic-chat`|`embedding`|`classification`), owner (feature owner id).

**AiPrompt** (AggregateRoot)
- id, taskId, locale, version, modelFamily, systemPrompt, userTemplate (merge-field markup), fewShotExamples[], guardrails (token caps, temperature range), status.

**AiProviderConfig** (AggregateRoot)
- id, tenantId (or `PLATFORM_DEFAULT`), provider (`ANTHROPIC`|`OPENAI`|`AZURE_OPENAI`|`OLLAMA`|`LOCAL_HTTP`), endpoint, credentialsRef, region, modelMap (modelFamily → provider model id), active.

**AiInvocation** (AggregateRoot, append-only / event-sourced projection)
- id, tenantId, callerService, taskSlug, promptVersion, providerUsed, inputTokens, outputTokens, latencyMs, costMicros, redactionApplied[], outcome (`OK`|`POLICY_BLOCKED`|`PROVIDER_ERROR`|`TIMEOUT`), correlationId.

**AiSafetyPolicy** (AggregateRoot)
- tenantId, piiClasses[] (`EMAIL`|`PHONE`|`IBAN`|`SSN`|`CUSTOM`), redactionMode (`MASK`|`REMOVE`|`BLOCK`), allowTopics[], denyTopics[], maxOutputTokens.

---

## 4. Business Rules

### 4.1 Rule Catalog

| ID | Rule | Type |
|----|------|------|
| BR-AI-001 | Every invocation MUST carry `tenantId`, `callerService`, `taskSlug`; request rejected otherwise | CONSTRAINT |
| BR-AI-002 | Tenant's `AiSafetyPolicy` applies to input AND output; blocked output produces `POLICY_BLOCKED` and no downstream event | CONSTRAINT |
| BR-AI-003 | If no `AiProviderConfig` for a tenant, fall back to `PLATFORM_DEFAULT` unless tenant has `residencyStrict=true` in which case call fails | POLICY |
| BR-AI-004 | Model family must be satisfiable by the tenant's `modelMap`; missing mapping produces 422 | CONSTRAINT |
| BR-AI-005 | All invocations logged to `AiInvocation` for audit + cost reporting; retention 24 months minimum | POLICY |
| BR-AI-006 | Prompt templates are versioned and immutable; edits create new version with status `DRAFT` | POLICY |
| BR-AI-007 | Rate limit: 60 rpm per tenant default, overridable; 429 on breach | POLICY |
| BR-AI-008 | Input payload MUST NOT exceed tenant-configured token ceiling (default 100 k) | CONSTRAINT |
| BR-AI-009 | Providers configured with external endpoints MUST present valid TLS cert; TLS skip is forbidden | CONSTRAINT |

---

## 5. Use Cases

| Endpoint | Use Case Type | Primary Aggregate |
|----------|--------------|------------------|
| `POST /tasks/{slug}/invoke` | WRITE (inference) | AiInvocation |
| `POST /embeddings` | WRITE | AiInvocation |
| `POST /classify` | WRITE | AiInvocation |
| `GET /tasks` | READ | AiTask |
| `POST /tasks` | WRITE (admin) | AiTask |
| `GET /tasks/{slug}/prompts` | READ | AiPrompt |
| `POST /tasks/{slug}/prompts` | WRITE | AiPrompt |
| `GET /provider-configs` | READ | AiProviderConfig |
| `PUT /provider-configs/{tenantId}` | WRITE | AiProviderConfig |
| `GET /safety-policies` | READ | AiSafetyPolicy |
| `PUT /safety-policies/{tenantId}` | WRITE | AiSafetyPolicy |
| `GET /invocations` | READ (audit + billing) | AiInvocation |
| `GET /invocations/{id}` | READ | AiInvocation |

---

## 6. REST API

**Base Path:** `/api/tech/ai/v1`
**Authentication:** OAuth2 / JWT. Scopes `tech.ai:invoke`, `tech.ai:read`, `tech.ai:admin`.
Invocation endpoints accept the task input per `AiTask.inputSchema` and return output per `outputSchema` plus metadata (invocationId, tokensUsed, costMicros, providerUsed, redactionApplied).

See `contracts/http/tech/ai/openapi.yaml` for full payloads.

---

## 7. Events & Integration

### 7.1 Published Events

| Event | Routing Key |
|-------|------------|
| AiInvocationCompleted | `tech.ai.invocation.completed` |
| AiInvocationBlocked | `tech.ai.invocation.blocked` |
| AiInvocationFailed | `tech.ai.invocation.failed` |
| AiCostAccrued | `tech.ai.cost.accrued` |
| AiPolicyUpdated | `tech.ai.policy.updated` |
| AiProviderSwitched | `tech.ai.provider.switched` |

### 7.2 Consumed Events

- `iam.tenant.tenant.created` → seed `PLATFORM_DEFAULT` provider config.
- `iam.tenant.tenant.deleted` → erase invocation logs after retention window.

---

## 8. Data Model

**Storage:** PostgreSQL 16, schema `tech_ai`.

Key tables: `ai_tasks`, `ai_prompts` (versioned), `ai_provider_configs` (per tenant), `ai_invocations` (append-only, partitioned by month), `ai_safety_policies`.

Vector storage (for embeddings) optional; v1 delegates embedding persistence to consumers (e.g., `tech.search` for semantic search). Revisit in ADR-AI-003.

---

## 9. Security & Compliance

### 9.1 Access Control

| Role | Access Level |
|------|-------------|
| `platform-admin` | Manage tasks, platform-default provider, global safety baseline |
| `tenant-admin` | Override tenant provider + safety policy; read tenant invocations |
| `tenant-user` / `service-account` | Invoke tasks granted to caller |
| `billing-ops` | Read `AiInvocation` cost fields |

### 9.2 Data Classification

- **Restricted**: inputs may include PII, PHI, financial data. Redaction MUST be applied per policy before the provider call.
- **Confidential**: provider credentials (stored in `iam.tenant` secret ref, never directly in this service's DB).

### 9.3 Compliance

- **GDPR** Art. 28 third-party processing: tenants choose their own provider; data-processing agreements tracked per `AiProviderConfig`.
- **EU AI Act (risk classification)**: task registry includes `riskClass` field (`MINIMAL`|`LIMITED`|`HIGH`); HIGH-risk tasks MUST have explicit tenant opt-in.
- **DORA**: external providers treated as critical ICT third parties — see §9.4.

### 9.4 DORA ICT Risk References

| Risk ID | Title | Treatment |
|---|---|---|
| `RISK-TECH-AI-001` | Provider outage — single-provider tenants lose feature availability | Mitigate: circuit breaker + tenant-visible degraded mode + multi-provider fallback option |
| `RISK-TECH-AI-002` | Data leakage via third-party LLM provider | Mitigate: redaction policy + regional provider pinning + annual TPR assessment per provider |
| `RISK-TECH-AI-003` | Prompt injection enabling unauthorized actions | Mitigate: output validation against outputSchema; tool-use disabled unless task explicitly declares tools |

### 9.5 SBOM Requirements

CycloneDX SBOM per build; provider SDK dependencies flagged; retention 5 years (per GOV-DORA-005 §5).

---

## 10. Quality Attributes

### 10.1 Performance Targets

| Operation | p95 | p99 |
|-----------|-----|-----|
| `POST /tasks/{slug}/invoke` (short task, < 500 output tokens) | < 2 s | < 8 s |
| `POST /embeddings` (single batch ≤ 100) | < 500 ms | < 2 s |
| Admin CRUD | < 100 ms | < 300 ms |

### 10.2 Availability

99.5 % for invocation (provider-dependent); 99.9 % for registry + policy APIs.

### 10.3 Scalability

- Stateless API; async worker pool per provider with circuit breakers.
- Cost accounting batched (outbox).
- Prompt registry cached with 60 s TTL.

### 10.5 SLI/SLO Reference

Companion `sli-slo-spec-tech-ai.md` tracks availability, provider-error-rate, redaction-success-rate.

### 10.6 Resilience Requirements

| Aspect | Value |
|---|---|
| RTO | < 15 min |
| RPO | < 5 min (invocation log) |
| Fallback | multi-provider configured tenants auto-failover; single-provider tenants surface degraded mode |

---

## 11. Feature Dependencies

| Feature ID | Feature Name |
|-----------|-------------|
| `F-TECH-AI-001` | Task Registry & Prompt Management |
| `F-TECH-AI-002` | Provider Configuration (tenant-admin) |
| `F-TECH-AI-003` | Safety Policy Management |
| `F-TECH-AI-004` | Cost & Usage Dashboard |

Consumed by features across suites: `F-TKS-500` AI Ticket Summary, `F-TKS-510` AI Auto-Triage, `F-TKS-520` AI Writing Assistant, `F-TKS-320` KB Suggest, and any future suite AI features.

---

## 12. Extension Points

- New provider adapters via `POST /provider-configs/extensions/providers`.
- New task types + output schemas via `POST /tasks`.
- Pre-invoke / post-invoke extension events `tech.ai.ext.pre-invoke`, `tech.ai.ext.post-invoke` for tenant-custom filters.
- Redaction-function registry `POST /safety-policies/extensions/redactors`.

---

## 13. Migration & Evolution

v1 has no predecessor. Roadmap:

- v1.0 — LLM chat + embedding + classification; Anthropic, OpenAI, Ollama providers.
- v1.1 — vector storage integration with `tech.search`; semantic-search-as-a-service.
- v1.2 — fine-tuning orchestration (pluggable).
- v1.3 — on-prem provider templates (Azure-OpenAI, AWS-Bedrock via private VPC).

---

## 14. Decisions & Open Questions

| # | Question | Severity | Status |
|---|----------|----------|--------|
| Q-AI-001 | Default platform provider for non-pinned tenants at GA? | High | Open — recommend Anthropic |
| Q-AI-002 | Do we store embeddings here or delegate to `tech.search`? | Medium | Open — ADR-AI-003 placeholder |
| Q-AI-003 | Is prompt authoring a platform-admin concern or tenant-admin? | Medium | Open |
| Q-AI-004 | Cost billing model: platform absorbs vs. tenant-pass-through? | High | Open — owner: platform-product |

### ADRs

- **ADR-AI-001** *(Proposed)*: Provider-agnostic abstraction; no direct SDK usage from consuming services.
- **ADR-AI-002** *(Proposed)*: All AI calls logged as `AiInvocation` for audit + billing; immutable once recorded.
- **ADR-AI-003** *(Proposed)*: Embedding storage delegated to `tech.search` (vector index) in v1.1; v1 returns vectors to caller only.

---

## 15. Appendix

### 15.1 Glossary

See `T1_Platform/tech/_tech_suite.md` SS1. Additions: *task*, *prompt*, *invocation*, *safety policy*, *redaction*, *provider*, *model family*.

### 15.2 Change Log

| Date | Version | Author | Changes |
|------|---------|--------|---------|
| 2026-04-18 | 1.0.0 | OpenLeap Architecture Team | Initial platform AI abstraction spec (new T1 service) |

### 15.3 Companion Files

- OpenAPI: `contracts/http/tech/ai/openapi.yaml`
- Event Schemas: `contracts/events/tech/ai/*.schema.json`
