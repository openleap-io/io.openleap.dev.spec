<!-- TEMPLATE COMPLIANCE: ~95%
Template: domain-service-spec.md v1.0.0
Present sections: §0-§15
-->

# iam.principal — Principal Management Service Domain Specification

> **Meta Information**
> - **Version:** 2026-04-03
> - **Template:** `https://github.com/openleap-io/io.openleap.dev.concepts/blob/main/templates/platform/domain/domain-service-spec.md` v1.0.0
> - **Template Compliance:** ~95%
> - **Author(s):** OpenLeap Architecture Team
> - **Status:** DRAFT
> - **Suite:** `iam` (Identity & Access Management)
> - **Domain:** `principal` (Principal Management)
> - **Bounded Context Ref:** `bc:principal-management`
> - **Service ID:** `iam-principal-svc`
> - **basePackage:** `io.openleap.iam.principal`
> - **API Base Path:** `/api/iam/principal/v1`
> - **Port:** `8080`
> - **Repository:** `https://github.com/openleap-io/io.openleap.iam.principal`
> - **Tags:** `iam`, `principal`, `identity`, `authentication`
> - **Team:** `team-iam` / `iam-team@openleap.io` / `#iam-team`

---

## Specification Guidelines Compliance

| Section | Title | Status |
|---------|-------|--------|
| §0 | Document Purpose & Scope | COMPLETE |
| §1 | Business Context | COMPLETE |
| §2 | Service Identity | COMPLETE |
| §3 | Domain Model | COMPLETE |
| §4 | Business Rules | COMPLETE |
| §5 | Use Cases | COMPLETE |
| §6 | REST API | COMPLETE |
| §7 | Events & Integration | COMPLETE |
| §8 | Data Model | COMPLETE |
| §9 | Security & Compliance | COMPLETE |
| §10 | Quality Attributes | COMPLETE |
| §11 | Feature Dependencies | COMPLETE |
| §12 | Extension Points | COMPLETE |
| §13 | Migration & Evolution | COMPLETE |
| §14 | Decisions & Open Questions | COMPLETE |
| §15 | Appendix | COMPLETE |

---

## 0. Document Purpose & Scope

### 0.1 Purpose

This document is the authoritative domain service specification for the `iam.principal` bounded context within the OpenLeap IAM suite. It describes the full lifecycle management of all principal types — human users, service accounts, system accounts, and device identities — including credential management, tenant membership, and GDPR compliance.

This spec is the source of truth for:
- Aggregate definitions and domain model
- Business rules and invariants
- REST API contract
- Event contract
- Data model
- Security posture
- Quality attributes
- Feature dependency mapping

### 0.2 Audience

| Role | Usage |
|------|-------|
| Backend Engineers | Implementation reference for `iam-principal-svc` |
| Frontend / BFF Engineers | API contract and event shapes for UI integration |
| Security / Compliance | Data classification, GDPR controls, audit events |
| Product Owners | Feature scope and business rules |
| Architects | Integration patterns, ADR alignment, extension points |
| QA / Test Engineers | Acceptance criteria, use case flows, business rule tests |

### 0.3 In Scope

- All CRUD and lifecycle operations on Principal and its specializations (HumanPrincipal, ServicePrincipal, SystemPrincipal, DevicePrincipal)
- TenantMembership management (add/remove/revoke)
- ApiKey management (create/rotate/revoke)
- Credential rotation
- Keycloak synchronization (create, update, status changes)
- GDPR data export (Article 20) and erasure (Article 17)
- Outbox-based event publishing for all state transitions

### 0.4 Out of Scope

- Authentication and token validation — delegated to Keycloak
- Authorization policy evaluation — delegated to `iam-authz-svc`
- Role and permission definitions — managed in `iam-role-svc`
- Session management — managed in `iam-session-svc`
- Business domain logic (HR employee sync is a consumer, not owned here)
- Infrastructure security (network policies, TLS termination)

### 0.5 Related Documents

| Document | Location | Relationship |
|----------|----------|--------------|
| IAM Suite Spec | `T1_Platform/iam/iam-suite-spec.md` | Parent suite |
| iam.authz Spec | `T1_Platform/iam/domain-specs/iam_authz-spec.md` | Consumer of principal events |
| iam.audit Spec | `T1_Platform/iam/domain-specs/iam_audit-spec.md` | Audit consumer |
| GDPR Feature Spec | `T1_Platform/iam/features/F-IAM-005-01.md` | GDPR export feature |
| BFF Guideline | `https://github.com/openleap-io/io.openleap.dev.concepts/blob/main/governance/bff-guideline.md` | GOV-BFF-001 |
| ADR Catalog | `io.openleap.dev.guidelines` | ADR-001 through ADR-067 |

---

## 1. Business Context

### 1.1 Domain Purpose

The Principal Management domain provides centralized identity lifecycle management for all principal types within the OpenLeap platform. A *principal* is any authenticated actor — human user, service account, internal system process, or registered device — that can be granted permissions and audited for actions.

This domain is analogous to SAP User Management (SAP UM) / SAP Identity Management (SAP IdM) and serves as the identity foundation on which authorization, audit, and all other IAM capabilities are built.

### 1.2 Business Value

| Value Driver | Description |
|--------------|-------------|
| Single source of truth for identities | All principal types managed from one domain; no duplicate identity silos |
| Security & compliance baseline | Credential rotation, GDPR erasure, audit trail are built-in, not bolted on |
| Multi-tenancy by design | TenantMembership aggregate enables principals to participate in multiple tenants with different roles |
| Platform extensibility | ServicePrincipal and SystemPrincipal enable secure M2M integrations; DevicePrincipal enables IoT scenarios |
| Regulatory readiness | GDPR Article 17 (erasure) and Article 20 (portability) are first-class operations |

### 1.3 Stakeholders

| Stakeholder | Interest |
|-------------|----------|
| Platform Operations | System account provisioning, audit trails |
| Tenant Administrators | User provisioning, role assignment, account deactivation |
| End Users | Profile self-service, MFA management, credential rotation |
| Security Team | Credential rotation policy, GDPR compliance, SOX audit |
| Integration Partners | Service principal provisioning, API key management |
| HR Systems | Employee onboarding/offboarding via profile sync |

### 1.4 Strategic Positioning

`iam-principal-svc` sits at the foundation of Tier 1 (Platform). It is a **critical-path dependency** for every other service: any service that needs to verify who is calling, or look up identity details, depends on this domain's data. Its availability and consistency targets are therefore the most stringent in the platform.

```
┌─────────────────────────────────────────────────────────────────┐
│  T3 Domain Services (HR, COM, CRM, FI, …)                       │
│    ↓ consume principal.created, membership.added events          │
├─────────────────────────────────────────────────────────────────┤
│  T1 IAM Suite                                                    │
│    iam-authz-svc  ←── principal events                          │
│    iam-audit-svc  ←── all principal events                      │
│    iam-session-svc ←── principal status                         │
│    iam-principal-svc  ◄── THIS DOMAIN                           │
│         │                                                        │
│         ↓ sync (REST)                                            │
│    Keycloak (external IdP)                                       │
└─────────────────────────────────────────────────────────────────┘
```

### 1.5 Service Context

`iam-principal-svc` is a Spring Boot 4 microservice following the four-tier layered architecture (ADR-001), CQRS (ADR-002), event-driven integration (ADR-003), and hybrid ingress (ADR-004). It owns the `iam_principal` bounded context and publishes all identity lifecycle events via the transactional outbox pattern (ADR-013).

---

## 2. Service Identity

| Property | Value |
|----------|-------|
| Service ID | `iam-principal-svc` |
| Suite | `iam` |
| Domain | `principal` |
| Bounded Context | `bc:principal-management` |
| Base Package | `io.openleap.iam.principal` |
| API Base Path | `/api/iam/principal/v1` |
| Exchange (topic) | `iam.principal.events` |
| Port | `8080` |
| Repository | `https://github.com/openleap-io/io.openleap.iam.principal` |
| Tech Stack | Java 25, Spring Boot 4, PostgreSQL 16, RabbitMQ |
| Deployment | Kubernetes, single replica min / 3 replica HA |

---

## 3. Domain Model

### 3.1 Aggregate Overview

| Aggregate | Type | Role | Description |
|-----------|------|------|-------------|
| `Principal` | Root (abstract) | Core | Base identity entity — discriminated by `principalType` |
| `HumanPrincipal` | Extension | Core | Human user: extends Principal with profile, MFA, locale |
| `ServicePrincipal` | Extension | Core | Machine/integration identity with OAuth2 client credentials |
| `SystemPrincipal` | Extension | Core | Internal system-to-system identity |
| `DevicePrincipal` | Extension | Core | IoT/edge device identity |
| `TenantMembership` | Entity | Supporting | Membership of a Principal in a Tenant, with assigned roles |
| `ApiKey` | Entity | Supporting | API key credential bound to a Principal |

### 3.2 Aggregate: Principal (Base)

The `Principal` aggregate root uses **single-table inheritance** (`principal_type` discriminator) to store all principal variants in `iam_principal`. Specialization data lives in dedicated extension tables.

#### 3.2.1 Attributes

| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | `UUID` | Yes | Surrogate key — `OlUuid.create()` (ADR-021) |
| `publicId` | `String` | Yes | Human-readable external ID (dual-key, ADR-020) |
| `principalType` | `PrincipalType` | Yes | Discriminator: `HUMAN \| SERVICE \| SYSTEM \| DEVICE` |
| `status` | `PrincipalStatus` | Yes | Lifecycle state (see §3.2.3) |
| `tenantId` | `UUID` | Yes | Owning tenant (RLS enforcement) |
| `username` | `String` | Yes | Unique login identifier (platform-wide) |
| `createdAt` | `Instant` | Yes | Creation timestamp (immutable) |
| `updatedAt` | `Instant` | Yes | Last update timestamp |
| `lastLoginAt` | `Instant` | No | Timestamp of most recent successful authentication |
| `lastRotatedAt` | `Instant` | No | Timestamp of last credential rotation |
| `deletedAt` | `Instant` | No | Soft-delete timestamp (set on GDPR erasure) |
| `version` | `Long` | Yes | Optimistic lock version |
| `customFields` | `JSONB` | No | Extensibility bag (ADR-067) |

> OPEN QUESTION: See Q-PRI-001 in §14.3

#### 3.2.2 Enumerations

**PrincipalType**

| Value | Description |
|-------|-------------|
| `HUMAN` | Human user (employee, contractor, end customer) |
| `SERVICE` | Integration/service account (OAuth2 client credentials) |
| `SYSTEM` | Internal system-to-system account (no interactive login) |
| `DEVICE` | IoT or edge device identity |

**PrincipalStatus**

| Value | Description | Terminal? |
|-------|-------------|-----------|
| `PENDING` | Registered, awaiting activation (e.g., email verification) | No |
| `ACTIVE` | Fully active; allowed to authenticate | No |
| `SUSPENDED` | Temporarily suspended; cannot authenticate | No |
| `DELETED` | Soft-deleted; PII anonymized (GDPR erasure) | Yes |

#### 3.2.3 Lifecycle State Machine

```
           ┌──────────┐
    create │          │
  ─────────►  PENDING │
           │          │
           └────┬─────┘
                │ activate
                ▼
           ┌──────────┐        suspend        ┌───────────┐
           │          │ ─────────────────────► │           │
           │  ACTIVE  │                        │ SUSPENDED │
           │          │ ◄───────────────────── │           │
           └────┬─────┘        reactivate      └─────┬─────┘
                │                                     │
                │ gdpr_erase / delete                 │ gdpr_erase / delete
                ▼                                     ▼
           ┌──────────┐                         ┌──────────┐
           │          │                         │          │
           │ DELETED  │◄────────────────────────│ DELETED  │
           │(terminal)│                         │(terminal)│
           └──────────┘                         └──────────┘
```

**Invariants:**
- A `PENDING` principal MUST complete Keycloak sync before transitioning to `ACTIVE` (BR-PRI-006)
- `DELETED` is terminal — no transitions out
- `DELETED` principals have PII fields set to anonymized placeholders (BR-PRI-007)

#### 3.2.4 Domain Events (Principal)

| Event | Trigger | Routing Key |
|-------|---------|-------------|
| `PrincipalCreatedEvent` | Create command succeeds | `iam.principal.principal.created` |
| `PrincipalActivatedEvent` | Status → ACTIVE | `iam.principal.principal.activated` |
| `PrincipalSuspendedEvent` | Status → SUSPENDED | `iam.principal.principal.suspended` |
| `PrincipalDeactivatedEvent` | Status → SUSPENDED from ACTIVE | `iam.principal.principal.deactivated` |
| `PrincipalDeletedEvent` | GDPR erase / hard delete | `iam.principal.principal.deleted` |

### 3.3 Aggregate Extension: HumanPrincipal

Human principals extend `Principal` with profile data stored in `iam_human_profile` (1:1).

#### 3.3.1 Attributes (extension table: `iam_human_profile`)

| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| `principalId` | `UUID` | Yes | FK → `iam_principal.id` |
| `displayName` | `String(255)` | Yes | Full display name (shown in UI) |
| `firstName` | `String(100)` | No | Given name |
| `lastName` | `String(100)` | No | Family name |
| `email` | `String(320)` | Yes | Primary email — unique platform-wide (BR-PRI-001) |
| `emailVerified` | `Boolean` | Yes | Whether email has been confirmed |
| `phone` | `String(30)` | No | Phone in E.164 format |
| `phoneVerified` | `Boolean` | No | Whether phone has been confirmed |
| `locale` | `String(10)` | No | IETF BCP 47 locale tag (e.g., `de-DE`) |
| `timezone` | `String(50)` | No | IANA timezone (e.g., `Europe/Berlin`) |
| `avatarUrl` | `String(2048)` | No | URL to profile avatar image |
| `mfaEnabled` | `Boolean` | Yes | Whether MFA is active for this principal |
| `mfaMethod` | `MfaMethod` | No | `TOTP \| SMS \| EMAIL \| HARDWARE_KEY` |
| `profileData` | `JSONB` | No | Arbitrary profile extension data |
| `gdprConsentAt` | `Instant` | No | Timestamp when GDPR consent was recorded |

**MfaMethod enum:**

| Value | Description |
|-------|-------------|
| `TOTP` | Time-based one-time password (e.g., Google Authenticator) |
| `SMS` | SMS one-time password |
| `EMAIL` | Email one-time password |
| `HARDWARE_KEY` | FIDO2 / WebAuthn hardware security key |

#### 3.3.2 Domain Events (HumanPrincipal)

| Event | Trigger | Routing Key |
|-------|---------|-------------|
| `ProfileUpdatedEvent` | Profile PUT succeeds | `iam.principal.profile.updated` |

### 3.4 Aggregate Extension: ServicePrincipal

Service principals represent machine identities that authenticate via OAuth2 `client_credentials` flow. Credential data stored in `iam_service_credential`.

#### 3.4.1 Attributes (extension table: `iam_service_credential`)

| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| `principalId` | `UUID` | Yes | FK → `iam_principal.id` |
| `clientId` | `String(100)` | Yes | OAuth2 client ID — unique platform-wide (BR-PRI-003) |
| `clientSecretHash` | `String(255)` | Yes | bcrypt hash of client secret |
| `serviceType` | `ServiceType` | Yes | Type of service: `INTEGRATION \| INTERNAL \| PARTNER` |
| `scopes` | `String[]` | No | Allowed OAuth2 scopes |
| `jwksUri` | `String(2048)` | No | JWKS endpoint for JWT-based auth (if using private_key_jwt) |
| `allowedGrantTypes` | `String[]` | Yes | Allowed OAuth2 grant types (e.g., `client_credentials`) |
| `rotationPolicyDays` | `Integer` | No | Days between mandatory credential rotations (BR-PRI-008) |

> OPEN QUESTION: See Q-PRI-008 in §14.3

**ServiceType enum:**

| Value | Description |
|-------|-------------|
| `INTEGRATION` | Third-party integration service |
| `INTERNAL` | Internal platform microservice |
| `PARTNER` | External partner system |

#### 3.4.2 Domain Events (ServicePrincipal)

| Event | Trigger | Routing Key |
|-------|---------|-------------|
| `CredentialsRotatedEvent` | Credential rotation succeeds | `iam.principal.credentials.rotated` |

### 3.5 Aggregate Extension: SystemPrincipal

System principals are used for internal platform-level system-to-system communication (e.g., scheduled jobs, background processes). They have no interactive login.

#### 3.5.1 Attributes (inline in `iam_principal`)

| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| `systemCode` | `String(50)` | Yes | Unique system identifier (e.g., `billing-cron`) |
| `environment` | `String(20)` | Yes | Target environment: `DEV \| TEST \| STAGING \| PROD` |
| `description` | `String(500)` | No | Human-readable description of system function |

### 3.6 Aggregate Extension: DevicePrincipal

Device principals represent IoT devices, edge nodes, or registered client devices.

#### 3.6.1 Attributes (extension table: `iam_device_profile`)

| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| `principalId` | `UUID` | Yes | FK → `iam_principal.id` |
| `deviceId` | `String(100)` | Yes | Vendor device ID or serial number |
| `deviceType` | `DeviceType` | Yes | `MOBILE \| DESKTOP \| IOT \| EDGE` |
| `hardwareFingerprint` | `String(255)` | No | Hardware fingerprint hash |
| `attestationData` | `JSONB` | No | Platform attestation payload (e.g., TPM quote) |
| `osName` | `String(50)` | No | Operating system name |
| `osVersion` | `String(30)` | No | Operating system version |
| `lastSeenAt` | `Instant` | No | Last observed device activity timestamp |

> OPEN QUESTION: See Q-PRI-006 in §14.3

**DeviceType enum:**

| Value | Description |
|-------|-------------|
| `MOBILE` | Mobile device (iOS, Android) |
| `DESKTOP` | Desktop or laptop computer |
| `IOT` | IoT device (sensor, actuator) |
| `EDGE` | Edge compute node |

### 3.7 Aggregate: TenantMembership

`TenantMembership` represents the association between a Principal and a Tenant, including the roles the principal holds within that tenant.

#### 3.7.1 Attributes

| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | `UUID` | Yes | Surrogate key — `OlUuid.create()` |
| `principalId` | `UUID` | Yes | FK → `iam_principal.id` |
| `tenantId` | `UUID` | Yes | Target tenant |
| `status` | `MembershipStatus` | Yes | Membership lifecycle state |
| `roleIds` | `UUID[]` | No | Roles granted in this tenant (FK → `iam_role`) |
| `invitedBy` | `UUID` | No | Principal who created this membership |
| `invitedAt` | `Instant` | No | Invitation timestamp |
| `activatedAt` | `Instant` | No | Activation timestamp |
| `revokedAt` | `Instant` | No | Revocation timestamp |
| `revokedBy` | `UUID` | No | Principal who revoked membership |
| `customFields` | `JSONB` | No | Tenant-specific membership metadata |

> OPEN QUESTION: See Q-PRI-003 in §14.3

**MembershipStatus enum:**

| Value | Description | Terminal? |
|-------|-------------|-----------|
| `PENDING` | Invitation sent, not yet accepted | No |
| `ACTIVE` | Active membership; principal has access | No |
| `REVOKED` | Membership revoked | Yes |

#### 3.7.2 Lifecycle State Machine

```
  invite        accept
  ──────► PENDING ──────► ACTIVE
                             │
                             │ revoke
                             ▼
                          REVOKED (terminal)
```

**Invariants:**
- A principal CANNOT be deleted while any TenantMembership is ACTIVE (BR-PRI-004)
- REVOKED is terminal — use a new invite to restore access

#### 3.7.3 Domain Events (TenantMembership)

| Event | Trigger | Routing Key |
|-------|---------|-------------|
| `MembershipAddedEvent` | Membership ACTIVE | `iam.principal.membership.added` |
| `MembershipRemovedEvent` | Membership REVOKED | `iam.principal.membership.removed` |

### 3.8 Aggregate: ApiKey

`ApiKey` represents an API key credential associated with a Principal. Only the first 8 characters (`keyPrefix`) are stored in plain text for identification; the full key is bcrypt-hashed.

#### 3.8.1 Attributes

| Attribute | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | `UUID` | Yes | Surrogate key — `OlUuid.create()` |
| `principalId` | `UUID` | Yes | Owning principal |
| `keyPrefix` | `String(8)` | Yes | First 8 characters of key (shown in UI for identification) |
| `keyHash` | `String(255)` | Yes | bcrypt hash of full API key |
| `name` | `String(100)` | Yes | Human-readable label for this key |
| `status` | `ApiKeyStatus` | Yes | Key lifecycle state |
| `scopes` | `String[]` | No | Allowed scopes/permissions for this key |
| `expiresAt` | `Instant` | No | Expiry timestamp (null = no expiry) |
| `lastUsedAt` | `Instant` | No | Last successful use timestamp |
| `createdAt` | `Instant` | Yes | Creation timestamp (immutable) |
| `revokedAt` | `Instant` | No | Revocation timestamp |
| `rotatedToId` | `UUID` | No | FK → successor ApiKey (for rotation lineage) |

**ApiKeyStatus enum:**

| Value | Description | Terminal? |
|-------|-------------|-----------|
| `ACTIVE` | Key is valid and usable | No |
| `ROTATED` | Key was rotated; successor exists in `rotatedToId` | Yes |
| `EXPIRED` | Key reached `expiresAt` | Yes |
| `REVOKED` | Key was explicitly revoked | Yes |

#### 3.8.2 Lifecycle State Machine

```
  create
  ──────► ACTIVE ──────► REVOKED  (terminal)
             │
             │ rotate
             ▼
          ROTATED (terminal) + new ACTIVE key created
             │
          (expiresAt reached)
             ▼
          EXPIRED (terminal)
```

**Invariants:**
- Maximum 10 ACTIVE ApiKeys per principal at any time (BR-PRI-005)

#### 3.8.3 Domain Events (ApiKey)

| Event | Trigger | Routing Key |
|-------|---------|-------------|
| `ApiKeyCreatedEvent` | Key created successfully | `iam.principal.apikey.created` |
| `ApiKeyRotatedEvent` | Key rotated; old key ROTATED, new key ACTIVE | `iam.principal.apikey.rotated` |

---

## 4. Business Rules

### 4.1 Rule Catalog

| Rule ID | Aggregate | Rule Summary | Severity |
|---------|-----------|--------------|----------|
| BR-PRI-001 | HumanPrincipal | Email must be unique platform-wide | ERROR |
| BR-PRI-002 | Principal | ACTIVE human principal must have ≥1 ACTIVE TenantMembership | WARNING |
| BR-PRI-003 | ServicePrincipal | `clientId` must be unique platform-wide | ERROR |
| BR-PRI-004 | Principal | Cannot delete principal with ACTIVE TenantMembership | ERROR |
| BR-PRI-005 | ApiKey | Maximum 10 ACTIVE ApiKeys per principal | ERROR |
| BR-PRI-006 | Principal | Keycloak sync must succeed before ACTIVE transition | ERROR |
| BR-PRI-007 | Principal | GDPR erasure anonymizes PII; audit trail ID preserved | ERROR |
| BR-PRI-008 | ServicePrincipal | Credentials must be rotated within `rotationPolicyDays` | WARNING |

### 4.2 Detailed Rule Definitions

#### BR-PRI-001 — Email Uniqueness (HumanPrincipal)

- **Description:** The `email` field on HumanPrincipal must be globally unique across the entire platform, not just within a single tenant. A principal can belong to multiple tenants, but always with the same email.
- **Trigger:** On create or profile update when `email` is changed.
- **Validation:** `SELECT COUNT(*) FROM iam_human_profile WHERE email = :email AND principal_id != :principalId` must return 0.
- **Failure response:** `HTTP 409 Conflict` with error code `IAM-PRI-001`.
- **Exception:** Soft-deleted (`DELETED`) principals' emails are anonymized, so the original email becomes available again after GDPR erasure.

#### BR-PRI-002 — Minimum Tenant Membership

- **Description:** An ACTIVE HumanPrincipal should have at least one ACTIVE TenantMembership. This is a warning-level check used by health monitoring and onboarding workflows.
- **Trigger:** Post-activation, checked by membership management operations.
- **Failure response:** Logged as a data quality warning; does not block operations.

#### BR-PRI-003 — Service Principal ClientId Uniqueness

- **Description:** The `clientId` assigned to a ServicePrincipal must be unique across the entire platform.
- **Trigger:** On ServicePrincipal create.
- **Validation:** `SELECT COUNT(*) FROM iam_service_credential WHERE client_id = :clientId` must return 0.
- **Failure response:** `HTTP 409 Conflict` with error code `IAM-PRI-003`.

#### BR-PRI-004 — No Delete with Active Memberships

- **Description:** A Principal with one or more ACTIVE TenantMemberships cannot be deleted (hard or GDPR erase). The caller must first revoke all memberships, or use SUSPEND to temporarily block access.
- **Trigger:** On DELETE or GDPR erase request.
- **Validation:** `SELECT COUNT(*) FROM iam_tenant_membership WHERE principal_id = :id AND status = 'ACTIVE'` must return 0.
- **Failure response:** `HTTP 409 Conflict` with error code `IAM-PRI-004`, listing all blocking tenants.
- **Exception:** Platform administrators with `iam:principal:force-delete` permission bypass this check.

#### BR-PRI-005 — ApiKey Limit

- **Description:** No more than 10 ApiKeys in ACTIVE status may exist for a single principal simultaneously.
- **Trigger:** On ApiKey create.
- **Validation:** Count of ACTIVE keys for the principal must be < 10.
- **Failure response:** `HTTP 422 Unprocessable Entity` with error code `IAM-PRI-005`.

#### BR-PRI-006 — Keycloak Sync Gate

- **Description:** A Principal transition to ACTIVE status is only permitted once Keycloak synchronization has confirmed the identity has been registered in Keycloak. The sync is synchronous (REST call to Keycloak Admin API).
- **Trigger:** On principal activation.
- **Failure response:** If Keycloak sync fails, the principal remains in PENDING status and an error event is logged. `HTTP 502 Bad Gateway` with error code `IAM-PRI-006`.
- **Note:** See Q-PRI-002 for discussion of async sync approach.

> OPEN QUESTION: See Q-PRI-002 in §14.3

#### BR-PRI-007 — GDPR Anonymization

- **Description:** When a GDPR erasure request is executed, all PII fields are replaced with anonymized placeholders while the structural record (ID, audit foreign keys) is retained. Specifically:
  - `email` → `deleted_{uuid}@gdpr.invalid`
  - `firstName`, `lastName`, `displayName` → `[Deleted]`
  - `phone` → `null`
  - `avatarUrl` → `null`
  - `profileData` → `{}`
  - `status` → `DELETED`, `deletedAt` = now
- **Trigger:** On GDPR erase command (`POST /principals/{id}/gdpr/erase`).
- **Constraint:** All active TenantMemberships must be revoked first (BR-PRI-004).

#### BR-PRI-008 — Credential Rotation Policy

- **Description:** ServicePrincipal credentials that have a `rotationPolicyDays` set must be rotated within that period. Expiry is tracked via `lastRotatedAt`. Stale credentials trigger a warning event and may be automatically suspended by the policy enforcer.
- **Trigger:** Background scheduled check; also evaluated on each ServicePrincipal update.
- **Failure response:** Warning event published; principal may be auto-suspended after grace period.

### 4.3 Field-Level Validations

| Field | Rule | Error Code |
|-------|------|-----------|
| `username` | 3–100 chars, alphanumeric + `.`, `_`, `-` only | `IAM-VAL-001` |
| `email` | RFC 5321 format, max 320 chars | `IAM-VAL-002` |
| `phone` | E.164 format (`+<country><number>`, max 30 chars) | `IAM-VAL-003` |
| `locale` | IETF BCP 47, max 10 chars | `IAM-VAL-004` |
| `timezone` | Valid IANA timezone identifier | `IAM-VAL-005` |
| `clientId` | 3–100 chars, alphanumeric + `-` only | `IAM-VAL-006` |
| `scopes` | Each scope max 100 chars; max 50 scopes | `IAM-VAL-007` |
| `deviceId` | Max 100 chars, non-empty | `IAM-VAL-008` |
| `keyName` | 1–100 chars | `IAM-VAL-009` |
| `expiresAt` | Must be in the future at creation time | `IAM-VAL-010` |

### 4.4 Cross-Field Validations

| Rule | Description | Error Code |
|------|-------------|-----------|
| MFA method required if MFA enabled | If `mfaEnabled = true`, `mfaMethod` must be non-null | `IAM-VAL-011` |
| JWKS URI required for `private_key_jwt` | If `allowedGrantTypes` includes `private_key_jwt`, `jwksUri` must be set | `IAM-VAL-012` |
| Rotation policy >= 1 day | If `rotationPolicyDays` is set, value must be ≥ 1 | `IAM-VAL-013` |

---

## 5. Use Cases

### 5.1 Use Case Overview

```
┌──────────────────────────────────────────────────────────────────────────────────┐
│  Actors                                                                            │
│    [TenantAdmin]  [SystemOps]  [HumanUser]  [ServiceClient]  [HRSystem]          │
│                                                                                    │
│  Core Use Cases                                                                    │
│    UC-PRI-001  Create Human Principal                                              │
│    UC-PRI-002  Create Service Principal                                            │
│    UC-PRI-003  Create System Principal                                             │
│    UC-PRI-004  Create Device Principal                                             │
│    UC-PRI-005  Activate Principal                                                  │
│    UC-PRI-006  Suspend Principal                                                   │
│    UC-PRI-007  Update Human Profile                                                │
│    UC-PRI-008  Rotate Credentials                                                  │
│    UC-PRI-009  Create API Key                                                      │
│    UC-PRI-010  Revoke API Key                                                      │
│    UC-PRI-011  GDPR Data Export                                                    │
│    UC-PRI-012  GDPR Data Erasure                                                   │
│    UC-PRI-013  Add Tenant Membership                                               │
│    UC-PRI-014  Revoke Tenant Membership                                            │
└──────────────────────────────────────────────────────────────────────────────────┘
```

### 5.2 Use Case: UC-PRI-001 — Create Human Principal

**Actor:** TenantAdmin, SystemOps  
**Feature:** F-IAM-001-01

**Preconditions:**
1. Caller has permission `iam:principal:human:create`
2. Tenant context is valid
3. `email` is not already in use

**Main Flow:**
1. Caller submits `POST /api/iam/principal/v1/humans` with profile payload
2. System validates input (field-level validations §4.3)
3. System checks email uniqueness (BR-PRI-001)
4. System creates `Principal` record (status=PENDING) via `CreateHumanPrincipalCommand`
5. System creates `HumanProfile` record in `iam_human_profile`
6. System calls Keycloak Admin API to register identity (BR-PRI-006)
7. If Keycloak sync succeeds: status transitions to ACTIVE
8. System publishes `PrincipalCreatedEvent` and `PrincipalActivatedEvent` via outbox (ADR-013)
9. Returns `HTTP 201 Created` with principal resource

**Postconditions:**
- Principal exists in ACTIVE state
- Keycloak has corresponding user record
- `iam-authz-svc` has received `principal.created` event
- `iam-audit-svc` has received audit events

**Alternative Flow A — Keycloak sync fails (BR-PRI-006):**
- Principal remains in PENDING state
- HTTP 502 returned with error code `IAM-PRI-006`
- No domain events published
- Caller may retry activation

**Alternative Flow B — Email already in use (BR-PRI-001):**
- HTTP 409 returned with error code `IAM-PRI-001`
- No records created

### 5.3 Use Case: UC-PRI-002 — Create Service Principal

**Actor:** TenantAdmin, SystemOps  
**Feature:** F-IAM-001-02

**Preconditions:**
1. Caller has permission `iam:principal:service:create`
2. `clientId` is not already in use

**Main Flow:**
1. Caller submits `POST /api/iam/principal/v1/services`
2. System validates `clientId` uniqueness (BR-PRI-003)
3. System generates a client secret (secure random, 32 bytes)
4. System creates Principal (status=ACTIVE) + service credential record
5. System registers in Keycloak as a confidential client
6. System publishes `PrincipalCreatedEvent` via outbox
7. Returns `HTTP 201` with principal resource **and plaintext client secret** (shown once only)

**Postconditions:**
- ServicePrincipal exists in ACTIVE state
- Client secret is bcrypt-hashed in storage; caller has the plaintext only from this response

### 5.4 Use Case: UC-PRI-005 — Activate Principal

**Actor:** TenantAdmin, SystemOps  
**Feature:** F-IAM-001-05

**Preconditions:**
1. Caller has permission `iam:principal:lifecycle:write`
2. Principal exists in PENDING or SUSPENDED state

**Main Flow:**
1. Caller submits `PATCH /api/iam/principal/v1/principals/{id}/status` with `{"status": "ACTIVE"}`
2. System validates target status transition is valid
3. System ensures Keycloak sync is current (BR-PRI-006)
4. Status transitions to ACTIVE
5. System publishes `PrincipalActivatedEvent` via outbox
6. Returns `HTTP 200` with updated principal

### 5.5 Use Case: UC-PRI-006 — Suspend Principal

**Actor:** TenantAdmin, SystemOps  
**Feature:** F-IAM-001-05

**Preconditions:**
1. Caller has permission `iam:principal:lifecycle:write`
2. Principal exists in ACTIVE state

**Main Flow:**
1. Caller submits `PATCH /principals/{id}/status` with `{"status": "SUSPENDED"}`
2. System transitions status to SUSPENDED
3. System updates Keycloak (disable account)
4. System publishes `PrincipalSuspendedEvent` via outbox
5. Existing sessions are invalidated via `iam-session-svc` (async, eventual)
6. Returns `HTTP 200`

### 5.6 Use Case: UC-PRI-007 — Update Human Profile

**Actor:** HumanUser (self), TenantAdmin  
**Feature:** F-IAM-001-06

**Preconditions:**
1. Principal exists and is ACTIVE
2. Caller has `iam:principal:profile:write` or is the principal itself
3. If `email` is being changed: new email must not be in use (BR-PRI-001)

**Main Flow:**
1. Caller submits `PUT /api/iam/principal/v1/humans/{id}/profile`
2. System validates changed fields
3. If email changed: uniqueness check (BR-PRI-001)
4. System updates `iam_human_profile`
5. If email changed: re-sync with Keycloak
6. System publishes `ProfileUpdatedEvent` via outbox
7. Returns `HTTP 200` with updated profile

### 5.7 Use Case: UC-PRI-008 — Rotate Credentials

**Actor:** ServiceClient (self), TenantAdmin  
**Feature:** F-IAM-001-07

**Preconditions:**
1. Principal is ServicePrincipal in ACTIVE state
2. Caller has `iam:principal:credentials:rotate`

**Main Flow:**
1. Caller submits `POST /api/iam/principal/v1/principals/{id}/credentials/rotate`
2. System generates new client secret
3. System updates `keyHash` in `iam_service_credential`; sets `lastRotatedAt`
4. System updates Keycloak client secret
5. System publishes `CredentialsRotatedEvent` via outbox
6. Returns `HTTP 200` with new plaintext secret (shown once only)

### 5.8 Use Case: UC-PRI-009 — Create API Key

**Actor:** HumanUser, TenantAdmin  
**Feature:** F-IAM-001-07

**Preconditions:**
1. Principal is ACTIVE
2. Principal has < 10 ACTIVE ApiKeys (BR-PRI-005)

**Main Flow:**
1. Caller submits `POST /api/iam/principal/v1/principals/{id}/apikeys` with `{name, scopes, expiresAt}`
2. System validates limit (BR-PRI-005) and field validations (§4.3)
3. System generates random API key (32-byte secure random, URL-safe base64)
4. System stores `keyPrefix` (first 8 chars) and `bcrypt(fullKey)` in `iam_api_key`
5. System publishes `ApiKeyCreatedEvent` via outbox
6. Returns `HTTP 201` with the full plaintext key (shown once only) and key metadata

### 5.9 Use Case: UC-PRI-011 — GDPR Data Export

**Actor:** HumanUser (data subject), TenantAdmin, DPO  
**Feature:** F-IAM-005-01

**Preconditions:**
1. Principal is HUMAN type
2. Caller has `iam:principal:gdpr:export` or is the data subject

**Main Flow:**
1. Caller submits `POST /api/iam/principal/v1/principals/{id}/gdpr/export`
2. System collects all personal data: profile, memberships, API keys (metadata only), audit events
3. System assembles structured JSON export (ISO 8601 timestamps, portable format)
4. Returns `HTTP 200` with export payload

**Postconditions:**
- Export event logged in audit trail

### 5.10 Use Case: UC-PRI-012 — GDPR Data Erasure

**Actor:** HumanUser (data subject), TenantAdmin, DPO  
**Feature:** F-IAM-005-02

**Preconditions:**
1. Principal is HUMAN type
2. No ACTIVE TenantMemberships (BR-PRI-004)
3. Caller has `iam:principal:gdpr:erase`

**Main Flow:**
1. Caller submits `POST /api/iam/principal/v1/principals/{id}/gdpr/erase` with reason
2. System checks for ACTIVE TenantMemberships (BR-PRI-004 guard)
3. System anonymizes all PII fields per BR-PRI-007 mapping
4. System deletes account in Keycloak
5. System transitions status to DELETED, sets `deletedAt`
6. System publishes `PrincipalDeletedEvent` via outbox
7. Returns `HTTP 200` confirming erasure

**Postconditions:**
- All PII is anonymized; structural record (IDs, timestamps) retained for audit
- Email is now available for reuse by a new principal

### 5.11 Use Case: UC-PRI-013 — Add Tenant Membership

**Actor:** TenantAdmin  
**Feature:** F-IAM-001-05

**Main Flow:**
1. Caller submits `POST /api/iam/principal/v1/principals/{id}/memberships`
2. System creates TenantMembership (status=PENDING)
3. If auto-approve: transition to ACTIVE immediately
4. System publishes `MembershipAddedEvent`
5. Returns `HTTP 201` with membership resource

### 5.12 Cross-Domain Workflows

#### Workflow: Employee Onboarding

```
HR-SVC (T3)         IAM-PRINCIPAL-SVC (T1)        KEYCLOAK
    │                        │                         │
    │  POST /humans          │                         │
    │───────────────────────►│                         │
    │                        │  Admin API sync         │
    │                        │────────────────────────►│
    │                        │  OK                     │
    │                        │◄────────────────────────│
    │                        │                         │
    │  201 Created           │  principal.created ──► iam-authz-svc
    │◄───────────────────────│  principal.activated ──► notification-svc
```

#### Workflow: GDPR Erasure with Cascade

```
DPO Portal          IAM-PRINCIPAL-SVC         KEYCLOAK     IAM-AUDIT-SVC
    │                       │                     │              │
    │  POST /gdpr/erase      │                     │              │
    │──────────────────────►│                     │              │
    │                       │  check memberships  │              │
    │                       │  anonymize PII      │              │
    │                       │  DELETE user        │              │
    │                       │────────────────────►│              │
    │                       │  200 OK             │              │
    │                       │◄────────────────────│              │
    │  200 OK               │  principal.deleted  │              │
    │◄──────────────────────│────────────────────────────────────►
```

---

## 6. REST API

### 6.1 Endpoint Catalog

| Method | Path | Feature | Auth | Description |
|--------|------|---------|------|-------------|
| `POST` | `/api/iam/principal/v1/humans` | F-IAM-001-01 | `iam:principal:human:create` | Create human principal |
| `POST` | `/api/iam/principal/v1/services` | F-IAM-001-02 | `iam:principal:service:create` | Create service principal |
| `POST` | `/api/iam/principal/v1/systems` | F-IAM-001-03 | `iam:principal:system:create` | Create system principal |
| `POST` | `/api/iam/principal/v1/devices` | F-IAM-001-04 | `iam:principal:device:create` | Create device principal |
| `GET` | `/api/iam/principal/v1/principals/{id}` | F-IAM-001-05 | `iam:principal:read` | Get principal by ID |
| `GET` | `/api/iam/principal/v1/principals` | F-IAM-001-05 | `iam:principal:read` | Search/list principals |
| `PUT` | `/api/iam/principal/v1/humans/{id}/profile` | F-IAM-001-06 | `iam:principal:profile:write` | Update human profile |
| `PATCH` | `/api/iam/principal/v1/principals/{id}/status` | F-IAM-001-05 | `iam:principal:lifecycle:write` | Change status |
| `POST` | `/api/iam/principal/v1/principals/{id}/credentials/rotate` | F-IAM-001-07 | `iam:principal:credentials:rotate` | Rotate credentials |
| `POST` | `/api/iam/principal/v1/principals/{id}/gdpr/export` | F-IAM-005-01 | `iam:principal:gdpr:export` | GDPR data export |
| `POST` | `/api/iam/principal/v1/principals/{id}/gdpr/erase` | F-IAM-005-02 | `iam:principal:gdpr:erase` | GDPR data erasure |
| `DELETE` | `/api/iam/principal/v1/principals/{id}` | F-IAM-001-05 | `iam:principal:admin:delete` | Hard delete (admin only) |
| `GET` | `/api/iam/principal/v1/principals/{id}/memberships` | F-IAM-001-05 | `iam:principal:read` | Get tenant memberships |
| `POST` | `/api/iam/principal/v1/principals/{id}/memberships` | F-IAM-001-05 | `iam:principal:membership:write` | Add tenant membership |
| `POST` | `/api/iam/principal/v1/principals/{id}/apikeys` | F-IAM-001-07 | `iam:principal:apikey:write` | Create API key |
| `DELETE` | `/api/iam/principal/v1/principals/{id}/apikeys/{keyId}` | F-IAM-001-07 | `iam:principal:apikey:write` | Revoke API key |

**Full OpenAPI contract:** `contracts/http/iam/principal/openapi.yaml`

### 6.2 Request/Response: Create Human Principal

**Request:** `POST /api/iam/principal/v1/humans`

```json
{
  "username": "jane.doe",
  "tenantId": "550e8400-e29b-41d4-a716-446655440000",
  "profile": {
    "displayName": "Jane Doe",
    "firstName": "Jane",
    "lastName": "Doe",
    "email": "jane.doe@example.com",
    "phone": "+49891234567",
    "locale": "de-DE",
    "timezone": "Europe/Berlin"
  },
  "mfaEnabled": false
}
```

**Response 201 Created:**

```json
{
  "id": "7f3e8900-c12b-41d4-b891-556677880000",
  "publicId": "usr_7f3e8900",
  "principalType": "HUMAN",
  "status": "ACTIVE",
  "username": "jane.doe",
  "tenantId": "550e8400-e29b-41d4-a716-446655440000",
  "profile": {
    "displayName": "Jane Doe",
    "firstName": "Jane",
    "lastName": "Doe",
    "email": "jane.doe@example.com",
    "emailVerified": false,
    "phone": "+49891234567",
    "phoneVerified": false,
    "locale": "de-DE",
    "timezone": "Europe/Berlin",
    "mfaEnabled": false,
    "mfaMethod": null
  },
  "createdAt": "2026-04-03T10:00:00Z",
  "updatedAt": "2026-04-03T10:00:00Z",
  "lastLoginAt": null,
  "_links": {
    "self": { "href": "/api/iam/principal/v1/principals/7f3e8900-c12b-41d4-b891-556677880000" },
    "memberships": { "href": "/api/iam/principal/v1/principals/7f3e8900-c12b-41d4-b891-556677880000/memberships" },
    "apikeys": { "href": "/api/iam/principal/v1/principals/7f3e8900-c12b-41d4-b891-556677880000/apikeys" }
  }
}
```

**Response 409 Conflict (BR-PRI-001):**

```json
{
  "status": 409,
  "errorCode": "IAM-PRI-001",
  "message": "Email address 'jane.doe@example.com' is already registered to another principal.",
  "timestamp": "2026-04-03T10:00:00Z"
}
```

### 6.3 Request/Response: Get Principal

**Request:** `GET /api/iam/principal/v1/principals/{id}`

**Response 200 OK:**

```json
{
  "id": "7f3e8900-c12b-41d4-b891-556677880000",
  "publicId": "usr_7f3e8900",
  "principalType": "HUMAN",
  "status": "ACTIVE",
  "username": "jane.doe",
  "tenantId": "550e8400-e29b-41d4-a716-446655440000",
  "createdAt": "2026-04-03T10:00:00Z",
  "updatedAt": "2026-04-03T10:00:00Z",
  "lastLoginAt": "2026-04-03T09:00:00Z",
  "profile": {
    "displayName": "Jane Doe",
    "email": "jane.doe@example.com",
    "emailVerified": true,
    "locale": "de-DE",
    "timezone": "Europe/Berlin",
    "mfaEnabled": true,
    "mfaMethod": "TOTP"
  }
}
```

**Response 404 Not Found:**

```json
{
  "status": 404,
  "errorCode": "IAM-PRI-404",
  "message": "Principal '7f3e8900-c12b-41d4-b891-556677880000' not found.",
  "timestamp": "2026-04-03T10:00:00Z"
}
```

### 6.4 Request/Response: Update Human Profile

**Request:** `PUT /api/iam/principal/v1/humans/{id}/profile`

```json
{
  "displayName": "Jane M. Doe",
  "phone": "+49891234568",
  "locale": "en-US",
  "timezone": "America/New_York"
}
```

**Response 200 OK:** Returns full principal resource as in §6.3.

### 6.5 Request/Response: Change Status

**Request:** `PATCH /api/iam/principal/v1/principals/{id}/status`

```json
{
  "status": "SUSPENDED",
  "reason": "Security investigation — ticket INC-00123"
}
```

**Response 200 OK:**

```json
{
  "id": "7f3e8900-c12b-41d4-b891-556677880000",
  "status": "SUSPENDED",
  "updatedAt": "2026-04-03T11:00:00Z"
}
```

**Response 409 Conflict (invalid transition):**

```json
{
  "status": 409,
  "errorCode": "IAM-PRI-TRANSITION",
  "message": "Invalid status transition: DELETED → ACTIVE",
  "timestamp": "2026-04-03T11:00:00Z"
}
```

### 6.6 Request/Response: Rotate Credentials (Service Principal)

**Request:** `POST /api/iam/principal/v1/principals/{id}/credentials/rotate`

```json
{
  "reason": "Scheduled 90-day rotation"
}
```

**Response 200 OK:**

```json
{
  "principalId": "9a1b2c3d-e4f5-6789-abcd-ef0123456789",
  "clientId": "my-integration-svc",
  "clientSecret": "nwKb3Xq8mP...plaintext-shown-once",
  "rotatedAt": "2026-04-03T12:00:00Z",
  "previousRotatedAt": "2026-01-03T12:00:00Z"
}
```

### 6.7 Request/Response: GDPR Data Erasure

**Request:** `POST /api/iam/principal/v1/principals/{id}/gdpr/erase`

```json
{
  "reason": "User request under GDPR Article 17",
  "requestReference": "GDPR-2026-0042",
  "confirmedBy": "dpo@example.com"
}
```

**Response 200 OK:**

```json
{
  "principalId": "7f3e8900-c12b-41d4-b891-556677880000",
  "status": "DELETED",
  "erasedAt": "2026-04-03T12:00:00Z",
  "anonymizedFields": ["email", "firstName", "lastName", "displayName", "phone", "avatarUrl", "profileData"],
  "auditReference": "GDPR-2026-0042"
}
```

**Response 409 Conflict (active memberships — BR-PRI-004):**

```json
{
  "status": 409,
  "errorCode": "IAM-PRI-004",
  "message": "Cannot erase principal with active tenant memberships. Revoke memberships first.",
  "activeMemberships": [
    { "tenantId": "550e8400-...", "tenantName": "Acme Corp" }
  ]
}
```

### 6.8 Common Error Response Format

All error responses follow the OpenLeap standard error envelope:

```json
{
  "status": 422,
  "errorCode": "IAM-VAL-001",
  "message": "Validation failed",
  "timestamp": "2026-04-03T10:00:00Z",
  "traceId": "4bf92f3577b34da6a3ce929d0e0e4736",
  "violations": [
    {
      "field": "username",
      "message": "Username must be 3-100 characters and contain only alphanumeric, '.', '_', '-' characters.",
      "rejectedValue": "j!"
    }
  ]
}
```

---

## 7. Events & Integration

### 7.1 Event Architecture

All events follow ADR-011 (thin events): the payload contains only the event trigger data plus the aggregate ID. Consumers fetch full state via the API if needed. Events are published via the transactional outbox (ADR-013) on exchange `iam.principal.events`.

### 7.2 Published Events

| Routing Key | Description | Consumers |
|-------------|-------------|-----------|
| `iam.principal.principal.created` | New principal registered | `iam-audit-svc`, `iam-authz-svc` |
| `iam.principal.principal.activated` | Principal status → ACTIVE | `iam-audit-svc`, `notification-svc` |
| `iam.principal.principal.deactivated` | Principal status → SUSPENDED from ACTIVE | `iam-audit-svc`, `iam-session-svc` |
| `iam.principal.principal.suspended` | Principal status → SUSPENDED | `iam-audit-svc`, `iam-session-svc` |
| `iam.principal.principal.deleted` | GDPR erasure or hard delete | `iam-audit-svc`, `iam-authz-svc` |
| `iam.principal.profile.updated` | Human profile fields changed | `iam-audit-svc`, `hr-svc` |
| `iam.principal.apikey.created` | New API key generated | `iam-audit-svc` |
| `iam.principal.apikey.rotated` | API key rotated | `iam-audit-svc` |
| `iam.principal.credentials.rotated` | Service principal credentials rotated | `iam-audit-svc` |
| `iam.principal.membership.added` | Principal added to a tenant | `iam-audit-svc`, `iam-authz-svc` |
| `iam.principal.membership.removed` | Principal removed from a tenant | `iam-audit-svc`, `iam-authz-svc` |

### 7.3 Event Format: `iam.principal.principal.created`

```json
{
  "eventId": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
  "eventType": "iam.principal.principal.created",
  "schemaVersion": "1.0",
  "timestamp": "2026-04-03T10:00:00.000Z",
  "traceId": "4bf92f3577b34da6a3ce929d0e0e4736",
  "source": "iam-principal-svc",
  "tenantId": "550e8400-e29b-41d4-a716-446655440000",
  "payload": {
    "principalId": "7f3e8900-c12b-41d4-b891-556677880000",
    "publicId": "usr_7f3e8900",
    "principalType": "HUMAN",
    "username": "jane.doe",
    "status": "ACTIVE"
  }
}
```

### 7.4 Event Format: `iam.principal.profile.updated`

```json
{
  "eventId": "a1b2c3d4-e5f6-7890-abcd-ef0123456789",
  "eventType": "iam.principal.profile.updated",
  "schemaVersion": "1.0",
  "timestamp": "2026-04-03T11:00:00.000Z",
  "traceId": "5ce03g4688c45eb7b4df030e1f1f5847",
  "source": "iam-principal-svc",
  "tenantId": "550e8400-e29b-41d4-a716-446655440000",
  "payload": {
    "principalId": "7f3e8900-c12b-41d4-b891-556677880000",
    "changedFields": ["displayName", "phone", "locale"]
  }
}
```

**Note:** Changed field values are NOT included in the event (thin event pattern, ADR-011). Consumers that need updated values call `GET /principals/{id}`.

### 7.5 Event Format: `iam.principal.membership.added`

```json
{
  "eventId": "b2c3d4e5-f6a7-8901-bcde-f01234567890",
  "eventType": "iam.principal.membership.added",
  "schemaVersion": "1.0",
  "timestamp": "2026-04-03T12:00:00.000Z",
  "traceId": "6df14h5799d56fc8c5eg141f2g2g6958",
  "source": "iam-principal-svc",
  "tenantId": "550e8400-e29b-41d4-a716-446655440000",
  "payload": {
    "membershipId": "c3d4e5f6-a7b8-9012-cdef-012345678901",
    "principalId": "7f3e8900-c12b-41d4-b891-556677880000",
    "targetTenantId": "660f9511-f30c-52e5-b827-557766551111",
    "roleIds": ["role-tenant-user"]
  }
}
```

### 7.6 Event Format: `iam.principal.principal.deleted` (GDPR)

```json
{
  "eventId": "d4e5f6a7-b8c9-0123-def0-123456789012",
  "eventType": "iam.principal.principal.deleted",
  "schemaVersion": "1.0",
  "timestamp": "2026-04-03T13:00:00.000Z",
  "traceId": "7eg25i6800e67gd9d6fh252g3h3h7069",
  "source": "iam-principal-svc",
  "tenantId": "550e8400-e29b-41d4-a716-446655440000",
  "payload": {
    "principalId": "7f3e8900-c12b-41d4-b891-556677880000",
    "principalType": "HUMAN",
    "deletionType": "GDPR_ERASURE",
    "auditReference": "GDPR-2026-0042"
  }
}
```

### 7.7 Event Format: `iam.principal.credentials.rotated`

```json
{
  "eventId": "e5f6a7b8-c9d0-1234-ef01-234567890123",
  "eventType": "iam.principal.credentials.rotated",
  "schemaVersion": "1.0",
  "timestamp": "2026-04-03T14:00:00.000Z",
  "traceId": "8fh36j7911f78he0e7gi363h4i4i8170",
  "source": "iam-principal-svc",
  "tenantId": "550e8400-e29b-41d4-a716-446655440000",
  "payload": {
    "principalId": "9a1b2c3d-e4f5-6789-abcd-ef0123456789",
    "principalType": "SERVICE",
    "rotatedAt": "2026-04-03T14:00:00.000Z",
    "rotationReason": "Scheduled 90-day rotation"
  }
}
```

### 7.8 Consumed Events

`iam-principal-svc` currently consumes **no external events**. Keycloak synchronization is synchronous (REST).

> OPEN QUESTION: See Q-PRI-002 in §14.3

### 7.9 Downstream Consumers Summary

| Consumer Service | Events Consumed | Purpose |
|-----------------|-----------------|---------|
| `iam-audit-svc` | All 11 events | Audit trail for all principal lifecycle changes |
| `iam-authz-svc` | `principal.created`, `principal.deleted`, `membership.added`, `membership.removed` | Sync authorization policy cache |
| `iam-session-svc` | `principal.suspended`, `principal.deactivated` | Invalidate active sessions |
| `hr-svc` | `profile.updated` | Sync employee profile changes from IAM to HR |
| `notification-svc` | `principal.activated` | Send welcome email to new human principals |

---

## 8. Data Model

### 8.1 Entity-Relationship Overview

```
┌───────────────────────────────────────────────────────────────────────────────┐
│                                                                                 │
│  iam_principal (1) ─────────────────── (0..1) iam_human_profile               │
│       │                                                                         │
│       │ (1) ────────────────────────── (0..1) iam_service_credential           │
│       │                                                                         │
│       │ (1) ────────────────────────── (0..1) iam_device_profile               │
│       │                                                                         │
│       │ (1) ────────────────────────── (0..N) iam_tenant_membership            │
│       │                                                                         │
│       │ (1) ────────────────────────── (0..N) iam_api_key                      │
│       │                                                                         │
│       └── iam_principal_outbox_events  (1..N)                                  │
│                                                                                 │
└───────────────────────────────────────────────────────────────────────────────┘
```

### 8.2 Table: `iam_principal`

Primary table for all principal types using single-table inheritance.

```sql
CREATE TABLE iam_principal (
    id                  UUID            NOT NULL DEFAULT ol_uuid_generate(),
    public_id           VARCHAR(50)     NOT NULL,
    principal_type      VARCHAR(10)     NOT NULL CHECK (principal_type IN ('HUMAN','SERVICE','SYSTEM','DEVICE')),
    status              VARCHAR(10)     NOT NULL CHECK (status IN ('PENDING','ACTIVE','SUSPENDED','DELETED')),
    tenant_id           UUID            NOT NULL,
    username            VARCHAR(100)    NOT NULL,
    -- SystemPrincipal fields (inline)
    system_code         VARCHAR(50),
    environment         VARCHAR(10),
    system_description  VARCHAR(500),
    -- timestamps
    created_at          TIMESTAMPTZ     NOT NULL DEFAULT NOW(),
    updated_at          TIMESTAMPTZ     NOT NULL DEFAULT NOW(),
    last_login_at       TIMESTAMPTZ,
    last_rotated_at     TIMESTAMPTZ,
    deleted_at          TIMESTAMPTZ,
    -- optimistic locking
    version             BIGINT          NOT NULL DEFAULT 0,
    -- extensibility
    custom_fields       JSONB,

    CONSTRAINT pk_iam_principal PRIMARY KEY (id),
    CONSTRAINT uq_iam_principal_public_id UNIQUE (public_id),
    CONSTRAINT uq_iam_principal_username UNIQUE (username)
);

CREATE INDEX idx_iam_principal_tenant_id ON iam_principal (tenant_id);
CREATE INDEX idx_iam_principal_status    ON iam_principal (status);
CREATE INDEX idx_iam_principal_type      ON iam_principal (principal_type);

-- Row-Level Security
ALTER TABLE iam_principal ENABLE ROW LEVEL SECURITY;
CREATE POLICY rls_tenant ON iam_principal
    USING (tenant_id = current_setting('app.tenant_id')::UUID);
```

### 8.3 Table: `iam_human_profile`

```sql
CREATE TABLE iam_human_profile (
    principal_id        UUID            NOT NULL,
    display_name        VARCHAR(255)    NOT NULL,
    first_name          VARCHAR(100),
    last_name           VARCHAR(100),
    email               VARCHAR(320)    NOT NULL,
    email_verified      BOOLEAN         NOT NULL DEFAULT FALSE,
    phone               VARCHAR(30),
    phone_verified      BOOLEAN         NOT NULL DEFAULT FALSE,
    locale              VARCHAR(10),
    timezone            VARCHAR(50),
    avatar_url          VARCHAR(2048),
    mfa_enabled         BOOLEAN         NOT NULL DEFAULT FALSE,
    mfa_method          VARCHAR(20)     CHECK (mfa_method IN ('TOTP','SMS','EMAIL','HARDWARE_KEY')),
    profile_data        JSONB,
    gdpr_consent_at     TIMESTAMPTZ,

    CONSTRAINT pk_iam_human_profile PRIMARY KEY (principal_id),
    CONSTRAINT fk_human_profile_principal FOREIGN KEY (principal_id)
        REFERENCES iam_principal(id) ON DELETE CASCADE,
    CONSTRAINT uq_human_profile_email UNIQUE (email)
);

CREATE INDEX idx_human_profile_email ON iam_human_profile (email);
```

### 8.4 Table: `iam_service_credential`

```sql
CREATE TABLE iam_service_credential (
    principal_id            UUID            NOT NULL,
    client_id               VARCHAR(100)    NOT NULL,
    client_secret_hash      VARCHAR(255)    NOT NULL,
    service_type            VARCHAR(15)     NOT NULL CHECK (service_type IN ('INTEGRATION','INTERNAL','PARTNER')),
    scopes                  TEXT[],
    jwks_uri                VARCHAR(2048),
    allowed_grant_types     TEXT[]          NOT NULL,
    rotation_policy_days    INTEGER         CHECK (rotation_policy_days >= 1),

    CONSTRAINT pk_iam_service_credential PRIMARY KEY (principal_id),
    CONSTRAINT fk_service_cred_principal FOREIGN KEY (principal_id)
        REFERENCES iam_principal(id) ON DELETE CASCADE,
    CONSTRAINT uq_service_cred_client_id UNIQUE (client_id)
);
```

### 8.5 Table: `iam_device_profile`

```sql
CREATE TABLE iam_device_profile (
    principal_id            UUID            NOT NULL,
    device_id               VARCHAR(100)    NOT NULL,
    device_type             VARCHAR(10)     NOT NULL CHECK (device_type IN ('MOBILE','DESKTOP','IOT','EDGE')),
    hardware_fingerprint    VARCHAR(255),
    attestation_data        JSONB,
    os_name                 VARCHAR(50),
    os_version              VARCHAR(30),
    last_seen_at            TIMESTAMPTZ,

    CONSTRAINT pk_iam_device_profile PRIMARY KEY (principal_id),
    CONSTRAINT fk_device_profile_principal FOREIGN KEY (principal_id)
        REFERENCES iam_principal(id) ON DELETE CASCADE
);
```

### 8.6 Table: `iam_tenant_membership`

```sql
CREATE TABLE iam_tenant_membership (
    id              UUID            NOT NULL DEFAULT ol_uuid_generate(),
    principal_id    UUID            NOT NULL,
    tenant_id       UUID            NOT NULL,
    status          VARCHAR(10)     NOT NULL CHECK (status IN ('PENDING','ACTIVE','REVOKED')),
    role_ids        UUID[],
    invited_by      UUID,
    invited_at      TIMESTAMPTZ,
    activated_at    TIMESTAMPTZ,
    revoked_at      TIMESTAMPTZ,
    revoked_by      UUID,
    custom_fields   JSONB,

    CONSTRAINT pk_iam_tenant_membership PRIMARY KEY (id),
    CONSTRAINT fk_membership_principal FOREIGN KEY (principal_id)
        REFERENCES iam_principal(id) ON DELETE CASCADE,
    CONSTRAINT uq_membership_principal_tenant UNIQUE (principal_id, tenant_id)
);

CREATE INDEX idx_membership_principal_id ON iam_tenant_membership (principal_id);
CREATE INDEX idx_membership_tenant_id    ON iam_tenant_membership (tenant_id);
CREATE INDEX idx_membership_status       ON iam_tenant_membership (status);

ALTER TABLE iam_tenant_membership ENABLE ROW LEVEL SECURITY;
CREATE POLICY rls_tenant_membership ON iam_tenant_membership
    USING (tenant_id = current_setting('app.tenant_id')::UUID);
```

### 8.7 Table: `iam_api_key`

```sql
CREATE TABLE iam_api_key (
    id              UUID            NOT NULL DEFAULT ol_uuid_generate(),
    principal_id    UUID            NOT NULL,
    key_prefix      VARCHAR(8)      NOT NULL,
    key_hash        VARCHAR(255)    NOT NULL,
    name            VARCHAR(100)    NOT NULL,
    status          VARCHAR(10)     NOT NULL CHECK (status IN ('ACTIVE','ROTATED','EXPIRED','REVOKED')),
    scopes          TEXT[],
    expires_at      TIMESTAMPTZ,
    last_used_at    TIMESTAMPTZ,
    created_at      TIMESTAMPTZ     NOT NULL DEFAULT NOW(),
    revoked_at      TIMESTAMPTZ,
    rotated_to_id   UUID,

    CONSTRAINT pk_iam_api_key PRIMARY KEY (id),
    CONSTRAINT fk_api_key_principal FOREIGN KEY (principal_id)
        REFERENCES iam_principal(id) ON DELETE CASCADE,
    CONSTRAINT fk_api_key_rotated_to FOREIGN KEY (rotated_to_id)
        REFERENCES iam_api_key(id)
);

CREATE INDEX idx_api_key_principal_id ON iam_api_key (principal_id);
CREATE INDEX idx_api_key_status       ON iam_api_key (status);
CREATE INDEX idx_api_key_prefix       ON iam_api_key (key_prefix);
```

### 8.8 Table: `iam_principal_outbox_events`

```sql
CREATE TABLE iam_principal_outbox_events (
    id              UUID            NOT NULL DEFAULT ol_uuid_generate(),
    aggregate_type  VARCHAR(50)     NOT NULL,
    aggregate_id    UUID            NOT NULL,
    event_type      VARCHAR(100)    NOT NULL,
    routing_key     VARCHAR(200)    NOT NULL,
    payload         JSONB           NOT NULL,
    status          VARCHAR(10)     NOT NULL DEFAULT 'PENDING'
                        CHECK (status IN ('PENDING','PUBLISHED','FAILED')),
    created_at      TIMESTAMPTZ     NOT NULL DEFAULT NOW(),
    published_at    TIMESTAMPTZ,
    retry_count     INTEGER         NOT NULL DEFAULT 0,
    error_message   TEXT,

    CONSTRAINT pk_outbox PRIMARY KEY (id)
);

CREATE INDEX idx_outbox_status_created ON iam_principal_outbox_events (status, created_at)
    WHERE status = 'PENDING';
```

---

## 9. Security & Compliance

### 9.1 Data Classification

| Data Element | Classification | Storage | Retention |
|--------------|---------------|---------|-----------|
| `email` | RESTRICTED (PII) | Encrypted at rest | Until GDPR erasure |
| `firstName`, `lastName` | RESTRICTED (PII) | Encrypted at rest | Until GDPR erasure |
| `phone` | RESTRICTED (PII) | Encrypted at rest | Until GDPR erasure |
| `avatarUrl` | INTERNAL | Plaintext | Until GDPR erasure |
| `clientSecretHash` | RESTRICTED (credential) | bcrypt hash only | Until rotation |
| `keyHash` | RESTRICTED (credential) | bcrypt hash only | Until revocation |
| `hardwareFingerprint` | CONFIDENTIAL | Hashed | Until GDPR erasure |
| `attestationData` | CONFIDENTIAL | JSONB encrypted | Until GDPR erasure |
| `profileData` | INTERNAL | JSONB | Until GDPR erasure |
| Audit trail records | INTERNAL | Plaintext | 7 years (SOX) |

> OPEN QUESTION: See Q-PRI-007 in §14.3

### 9.2 Permission Matrix

| Operation | Required Permission | Notes |
|-----------|--------------------|-|
| Create human principal | `iam:principal:human:create` | Tenant admin or platform ops |
| Create service principal | `iam:principal:service:create` | Platform ops only |
| Create system principal | `iam:principal:system:create` | Platform ops only |
| Create device principal | `iam:principal:device:create` | Tenant admin |
| Read any principal | `iam:principal:read` | Scoped by tenant via RLS |
| Update own profile | `iam:principal:profile:self:write` | Self-service; no admin perm needed |
| Update any profile | `iam:principal:profile:write` | Tenant admin |
| Change lifecycle status | `iam:principal:lifecycle:write` | Tenant admin or platform ops |
| Rotate credentials | `iam:principal:credentials:rotate` | Owner or tenant admin |
| GDPR export | `iam:principal:gdpr:export` | Self or DPO |
| GDPR erase | `iam:principal:gdpr:erase` | DPO only |
| Hard delete | `iam:principal:admin:delete` | Platform ops only; bypasses BR-PRI-004 |
| Manage memberships | `iam:principal:membership:write` | Tenant admin |
| Manage API keys | `iam:principal:apikey:write` | Owner or tenant admin |

### 9.3 Authentication & Authorization Controls

| Control | Implementation |
|---------|---------------|
| Authentication | JWT Bearer token (Keycloak-issued); validated on every request |
| Authorization | RBAC evaluated by `iam-authz-svc`; ABAC for self-service paths |
| Tenant isolation | PostgreSQL RLS on `tenant_id`; set via `SET LOCAL app.tenant_id` per request |
| Rate limiting | 1,000 req/min per client IP; 100 req/min for write operations |
| Input sanitization | All string inputs sanitized; SQL injection prevented by parameterized queries |
| Audit logging | Every write operation produces an audit event via outbox |

### 9.4 GDPR Compliance Controls

| Requirement | Article | Implementation |
|-------------|---------|----------------|
| Right to erasure | Art. 17 | `POST /gdpr/erase` — anonymizes PII, retains structural ID |
| Right to portability | Art. 20 | `POST /gdpr/export` — structured JSON export of all personal data |
| Data minimization | Art. 5(1)(c) | Optional fields are truly optional; no shadow profiling |
| Consent tracking | Art. 7 | `gdprConsentAt` field on HumanProfile |
| Retention limits | Art. 5(1)(e) | Automated cleanup of DELETED principals after retention period |
| Breach notification | Art. 33 | Security events trigger `iam-audit-svc` breach reporting workflow |

### 9.5 SOX Compliance Controls

| Control | Description |
|---------|-------------|
| Audit trail immutability | `iam_principal_outbox_events` records are append-only |
| System principal controls | SystemPrincipal used in financial processing requires `SOX_CRITICAL` flag and enhanced audit |
| Segregation of duties | Create and activate operations are separately permissioned |
| 7-year audit retention | Audit events retained 7 years per SOX requirements |

---

## 10. Quality Attributes

### 10.1 Performance

| Metric | Target | Measurement |
|--------|--------|-------------|
| GET (read) p95 latency | < 50ms | Prometheus histogram |
| GET (cached authz check) p95 latency | < 10ms | Prometheus histogram |
| POST (create) p95 latency | < 200ms | Prometheus histogram (includes Keycloak sync) |
| PATCH (status change) p95 latency | < 100ms | Prometheus histogram |
| GDPR export p95 latency | < 2,000ms | Prometheus histogram |
| Throughput | 1,000+ req/sec per instance | k6 load test |

**Performance constraints:**
- Keycloak Admin API calls are the primary latency driver for write operations; circuit breaker pattern applied (ADR-004)
- Read operations on `iam_principal` are served from read replicas (ADR-017)
- Caching: principal status cached in Redis with 60s TTL for `iam-authz-svc` hot path

### 10.2 Availability

| Metric | Target | Notes |
|--------|--------|-------|
| Uptime SLA | 99.95% | ~4.4 hours downtime/year |
| RTO (Recovery Time Objective) | 5 minutes | IAM is critical path for all authentication |
| RPO (Recovery Point Objective) | 1 minute | PostgreSQL streaming replication with synchronous commit |
| Deployment strategy | Rolling update, zero downtime | Kubernetes Deployment with readiness probes |

> OPEN QUESTION: See Q-PRI-005 in §14.3

### 10.3 Scalability

| Dimension | Approach |
|-----------|----------|
| Horizontal scaling | Stateless service; scale by adding pod replicas |
| Database read scaling | PostgreSQL read replicas via `iam-principal-svc` read model (ADR-017) |
| Connection pooling | HikariCP, pool size 20 per instance |
| Peak load | Designed for 10,000 concurrent active users with 3 replicas |

### 10.4 Security Quality

| Attribute | Target |
|-----------|--------|
| OWASP Top 10 coverage | All applicable items addressed in implementation |
| Dependency vulnerability scan | No HIGH or CRITICAL CVEs in production dependencies |
| Penetration test | Annual external pen test; findings remediated within 30 days |
| Secret rotation | Keycloak admin credentials rotated every 90 days |

### 10.5 Maintainability

| Attribute | Target |
|-----------|--------|
| Test coverage (unit) | ≥ 80% line coverage |
| Test coverage (integration) | All REST endpoints and business rules covered |
| Code complexity | Cyclomatic complexity ≤ 10 per method |
| API backward compatibility | Additive-only changes within a major version; breaking changes require new major version |
| OpenAPI spec drift | CI check: OpenAPI spec must match running service (contract test) |

---

## 11. Feature Dependencies

### 11.1 Purpose

This section maps domain service capabilities to the leaf features that expose them. Each feature has a dedicated feature spec (`T1_Platform/iam/features/F-IAM-XXX-YY.md`) that provides actor stories, acceptance criteria, and AUI screen contracts.

### 11.2 Feature Dependency Register

| Feature ID | Feature Name | Dependency Type | Endpoints Used | Status |
|------------|-------------|-----------------|----------------|--------|
| F-IAM-001-01 | Create Human Principal | `sync_api` | `POST /humans` | DEFINED |
| F-IAM-001-02 | Create Service Principal | `sync_api` | `POST /services` | DEFINED |
| F-IAM-001-03 | Create System Principal | `sync_api` | `POST /systems` | DEFINED |
| F-IAM-001-04 | Create Device Principal | `sync_api` | `POST /devices` | DEFINED |
| F-IAM-001-05 | Principal Lifecycle Management | `sync_api` | `GET /principals/{id}`, `GET /principals`, `PATCH /status`, `DELETE /principals/{id}`, `GET /memberships`, `POST /memberships` | DEFINED |
| F-IAM-001-06 | Profile Management | `sync_api` | `PUT /humans/{id}/profile` | DEFINED |
| F-IAM-001-07 | Credential & API Key Management | `sync_api` | `POST /credentials/rotate`, `POST /apikeys`, `DELETE /apikeys/{id}` | DEFINED |
| F-IAM-005-01 | GDPR Data Export | `sync_api` | `POST /gdpr/export` | DEFINED |
| F-IAM-005-02 | GDPR Data Erasure | `sync_api` | `POST /gdpr/erase` | DEFINED |

### 11.3 Endpoints per Feature

| Endpoint | Feature | Notes |
|----------|---------|-------|
| `POST /humans` | F-IAM-001-01 | Core create flow |
| `POST /services` | F-IAM-001-02 | Includes secret generation |
| `POST /systems` | F-IAM-001-03 | Admin-only |
| `POST /devices` | F-IAM-001-04 | Device attestation optional |
| `GET /principals/{id}` | F-IAM-001-05 | Read model |
| `GET /principals` | F-IAM-001-05 | Paginated search |
| `PATCH /principals/{id}/status` | F-IAM-001-05 | Lifecycle transitions |
| `DELETE /principals/{id}` | F-IAM-001-05 | Admin hard delete |
| `GET /principals/{id}/memberships` | F-IAM-001-05 | Membership list |
| `POST /principals/{id}/memberships` | F-IAM-001-05 | Add membership |
| `PUT /humans/{id}/profile` | F-IAM-001-06 | Full profile replace |
| `POST /credentials/rotate` | F-IAM-001-07 | Service principal rotation |
| `POST /apikeys` | F-IAM-001-07 | API key creation |
| `DELETE /apikeys/{id}` | F-IAM-001-07 | API key revocation |
| `POST /gdpr/export` | F-IAM-005-01 | Data portability |
| `POST /gdpr/erase` | F-IAM-005-02 | Right to erasure |

### 11.4 BFF Hints

Products consuming IAM principal features through a BFF layer should be aware of:

| Concern | Guidance |
|---------|---------|
| Profile display | BFF should enrich principal responses with tenant-specific display labels |
| Self-service paths | `profile:self:write` permission bypasses admin checks — BFF must enforce `sub` claim matching principal ID |
| GDPR export | Export response can be large; BFF should stream the response rather than buffering |
| API key display | Full key is returned only once on creation; BFF must display and warn user accordingly |
| MFA method selection | BFF screen contract for MFA setup is in `F-IAM-001-05` AUI spec |

> OPEN QUESTION: See Q-PRI-004 in §14.3

### 11.5 Impact Assessment

| Change | Affected Features | Migration Required? |
|--------|------------------|---------------------|
| Add new principal type | F-IAM-001-01 through 04, F-IAM-001-05 | New feature spec + new endpoint |
| Add custom field to Principal base | All features | Additive — no migration, use `customFields` JSONB |
| Change email uniqueness scope (platform → tenant) | F-IAM-001-01, F-IAM-001-06 | Data migration + BR-PRI-001 update |
| Remove MFA method | F-IAM-001-05, F-IAM-001-06 | Deprecation cycle required |
| Change API key limit (BR-PRI-005) | F-IAM-001-07 | Config change only |

---

## 12. Extension Points

### 12.1 Extension Architecture

`iam-principal-svc` follows the Open-Closed Principle: it is open for extension via defined extension points and closed for modification of core lifecycle logic. Extensions are governed by ADR-067 (Extensibility).

### 12.2 Custom Fields (§12.2)

The `customFields` (`JSONB`) column on `iam_principal` and `custom_fields` on `iam_tenant_membership` allow product teams and tenant operators to store domain-specific metadata without schema changes.

| Aggregate | Field | Max Size | Schema Governance |
|-----------|-------|----------|-------------------|
| `Principal` | `customFields` | 64 KB | No schema enforced; validated via product-level Zod in BFF |
| `TenantMembership` | `customFields` | 16 KB | No schema enforced; tenant-specific metadata |

**Usage guidelines:**
- Do not store sensitive PII in `customFields` without explicit GDPR lifecycle management
- Product-specific mandatory fields should be managed in the product BFF layer
- Cross-service query on `customFields` requires GIN index; raise a spec ticket if needed

> OPEN QUESTION: See Q-PRI-001 in §14.3

### 12.3 Extension Events (§12.3)

Products may emit extension events by listening to the core events and enriching them. `iam-principal-svc` does not currently support pluggable event hooks directly; extension is via consumer-side processing.

| Core Event | Suggested Extension Pattern |
|------------|----------------------------|
| `principal.created` | HR system emits `hr.employee.onboarding.initiated` |
| `principal.activated` | Notification service emits personalized welcome workflow |
| `membership.added` | Tenant service emits `tenant.user.added` for tenant-level workflows |

### 12.4 Extension Rules (§12.4)

Products may register custom validation rules via the `customFields` mechanism and enforce them in the BFF layer (Zod schemas). Core business rules (BR-PRI-001 through BR-PRI-008) are not overridable.

| Extension Point | Mechanism | Owner |
|-----------------|-----------|-------|
| Required custom profile fields | BFF Zod validation on `POST /humans` | Product team |
| Tenant-specific membership rules | BFF validation before `POST /memberships` | Tenant admin |
| Device attestation validation | `attestationData` JSONB schema in DevicePrincipal | Platform ops |

### 12.5 Extension Actions (§12.5)

The following hooks allow extension logic to be injected into core workflows without modifying the service:

| Hook | Trigger Point | Interface |
|------|--------------|-----------|
| `PostPrincipalCreatedHook` | After `PrincipalCreatedEvent` published | Async event consumer |
| `PostMembershipAddedHook` | After `MembershipAddedEvent` published | Async event consumer |
| `PreGdprEraseHook` | Before GDPR erasure executes | Synchronous — currently no mechanism; planned via saga (see Q-PRI-002) |

### 12.6 Aggregate Hooks (§12.6)

Spring application events are fired within the transaction boundary for in-process extension:

| Spring Event | Phase | Usage |
|-------------|-------|-------|
| `PrincipalPreCreateEvent` | Before persistence | Input enrichment, additional validation |
| `PrincipalPostActivateEvent` | After persistence, before outbox write | Notification triggers |
| `TenantMembershipPreRevokeEvent` | Before revocation | Pre-revocation checks from other domains |

### 12.7 Extension API Endpoints (§12.7)

Products that need to store and retrieve custom principal metadata can use the `customFields` field via the standard profile PUT endpoint. There are no dedicated extension API endpoints in v1; if required, raise a spec ticket.

### 12.8 Extension Summary

| Extension Type | Available in v1 | Mechanism |
|----------------|----------------|-----------|
| Custom fields on Principal | Yes | `customFields` JSONB |
| Custom fields on TenantMembership | Yes | `customFields` JSONB |
| Extension events | Yes (consumer-side) | Event-driven consumers |
| Custom validation rules | Yes (BFF layer) | Zod schema in BFF |
| In-process hooks | Yes (Spring events) | Spring application events |
| Custom API endpoints | No | Raise spec ticket |

---

## 13. Migration & Evolution

### 13.1 Migration Framework

#### 13.1.1 Greenfield Deployment

For new OpenLeap deployments, `iam-principal-svc` starts from an empty schema. Flyway manages schema migrations in `db/migration/V*.sql`. Migration scripts follow:
- **V1__create_iam_principal.sql** — Base `iam_principal` table
- **V2__create_iam_human_profile.sql** — Human profile extension
- **V3__create_iam_service_credential.sql** — Service credential extension
- **V4__create_iam_device_profile.sql** — Device profile extension
- **V5__create_iam_tenant_membership.sql** — Tenant membership
- **V6__create_iam_api_key.sql** — API key
- **V7__create_iam_outbox.sql** — Outbox events table

#### 13.1.2 Migration from Legacy Identity Systems

For organizations migrating from an existing identity provider (e.g., LDAP, Active Directory, legacy IAM):

1. **Export phase:** Extract users from source system in standard CSV/LDIF format
2. **Transform phase:** Map source attributes to `iam_principal` + `iam_human_profile` schema
3. **Load phase:** Use bulk import API (`POST /api/iam/principal/v1/humans/bulk`) — available in migration tooling only, not in production API
4. **Keycloak sync phase:** Run bulk Keycloak sync for all imported principals
5. **Validation phase:** Verify email uniqueness, membership assignments, credential reset notifications sent
6. **Cutover phase:** Disable source system; switch DNS/token issuer to Keycloak

**Migration constraints:**
- Email uniqueness (BR-PRI-001) is enforced during migration; duplicate emails require manual resolution
- Credential reset: Imported principals receive activation emails to set new credentials
- Legacy session invalidation: Source system sessions expire naturally (TTL) or are forcibly invalidated

#### 13.1.3 Data Migration Scripts

Migration scripts are maintained in `tools/migration/` in the service repository, not in this spec repo.

### 13.2 Deprecation Framework

#### 13.2.1 API Versioning Policy

`iam-principal-svc` follows semantic versioning for its REST API:
- **PATCH** (e.g., 1.0.0 → 1.0.1): Bug fixes only; no API changes
- **MINOR** (e.g., 1.0.0 → 1.1.0): Additive changes (new fields, new optional parameters, new endpoints)
- **MAJOR** (e.g., 1.0.0 → 2.0.0): Breaking changes; old version supported for minimum 6 months

#### 13.2.2 Deprecation Process

1. Mark field/endpoint as `deprecated` in OpenAPI spec with `x-deprecated-since` and `x-sunset-date`
2. Add `Deprecation` and `Sunset` HTTP response headers to deprecated endpoints
3. Notify consumers via `#iam-team` and changelog
4. Maintain deprecated endpoint for ≥ 6 months after sunset announcement
5. Remove after sunset date with MAJOR version bump

#### 13.2.3 Currently Deprecated Elements

No deprecated elements at time of writing (v1.0.0).

---

## 14. Decisions & Open Questions

### 14.1 Consistency Checks

| # | Check | Status | Notes |
|---|-------|--------|-------|
| C1 | Every REST WRITE endpoint maps to exactly one use case in §5 | Pass | All 10 write endpoints mapped |
| C2 | Every event in §7.2 table appears in aggregate domain events §3.x.4 | Pass | All 11 events traced to aggregate |
| C3 | Every business rule in §4.1 catalog has a detailed definition in §4.2 | Pass | BR-PRI-001 through 008 fully detailed |
| C4 | Every feature in §11.2 has a corresponding endpoint in §6.1 | Pass | All 9 features map to endpoints |
| C5 | Every table in §8 has a corresponding aggregate in §3 | Pass | 7 tables match 7 aggregates/extensions |
| C6 | Security permissions in §9.2 align with auth requirements in §6.1 | Pass | All endpoints have permission listed |
| C7 | Quality targets in §10.1 include Keycloak dependency latency impact | Pass | §10.1 notes Keycloak as latency driver |

### 14.2 Decisions & Resolved Conflicts

| Decision | Resolution | Date | Rationale |
|----------|-----------|------|-----------|
| Single-table vs. joined-table inheritance | Single-table inheritance with extension tables | 2026-04-03 | Simplifies queries for common operations; extension tables provide type-specific isolation |
| Platform-wide email uniqueness | Platform-wide (not tenant-scoped) | 2026-04-03 | A user with multiple tenant memberships uses the same identity; email is the identity anchor |
| Keycloak sync strategy | Synchronous on create/activate | 2026-04-03 | Simplest initial approach; eliminates partial state window. Revisit if Keycloak becomes bottleneck. |
| API key storage | bcrypt hash + prefix | 2026-04-03 | bcrypt for security; prefix for UI identification without exposing the key |
| GDPR erasure approach | Anonymize-in-place (not hard delete) | 2026-04-03 | Preserves referential integrity for audit trail; complies with Art. 17 while maintaining audit obligations |

### 14.3 Open Questions

| ID | Question | Category | Raised | Owner |
|----|----------|----------|--------|-------|
| Q-PRI-001 | Which aggregates should support custom fields for product extensibility? Currently only `Principal` and `TenantMembership` have `customFields`. Should `HumanProfile`, `ServiceCredential`, and `DeviceProfile` also get `customFields` JSONB? | Domain model | 2026-04-03 | Architecture Team |
| Q-PRI-002 | Should Keycloak sync be synchronous (blocking create flow, current approach) or async (event-based, higher resilience)? Async would require a saga pattern and a PENDING→ACTIVE transition guard. | Architecture | 2026-04-03 | team-iam |
| Q-PRI-003 | Should `TenantMembership.customFields` be schema-governed per tenant (via a membership schema registry) or remain free-form JSONB? Schema governance improves discoverability at the cost of complexity. | Domain model | 2026-04-03 | Architecture Team |
| Q-PRI-004 | Which feature IDs from other suites (e.g., `hr`, `bp`, `com`) consume principal endpoints directly (not via events)? This is needed to complete the cross-suite impact analysis in §11.5. | Feature dependencies | 2026-04-03 | Product Owners |
| Q-PRI-005 | RTO of 5 minutes and RPO of 1 minute for IAM — have these been formally agreed with the SRE team and documented in the SLA register? | Quality / SRE | 2026-04-03 | SRE Team |
| Q-PRI-006 | Should `DevicePrincipal` support hardware attestation (e.g., TPM quote, Apple Secure Enclave)? The `attestationData` JSONB field is reserved but validation logic is unspecified. | Domain model | 2026-04-03 | Security Team |
| Q-PRI-007 | What is the retention period for GDPR-anonymized principal records (`status=DELETED`)? The structural record is retained for audit; but what is the maximum time before the anonymized record itself is purged? | GDPR compliance | 2026-04-03 | DPO / Legal |
| Q-PRI-008 | Should `ServicePrincipal` support mTLS client certificate authentication in addition to the OAuth2 `client_credentials` flow? mTLS provides stronger identity assurance for high-trust integrations. | Architecture | 2026-04-03 | Security Team |

### 14.4 ADR References

| ADR | Title | Relevance |
|-----|-------|-----------|
| ADR-001 | Four-Tier Layered Architecture | Service lives in T1 Platform tier |
| ADR-002 | CQRS | Separate read/write models for principal queries vs. commands |
| ADR-003 | Event-Driven Integration | All state changes publish events via outbox |
| ADR-004 | Hybrid Ingress | REST for sync operations; events for async integration |
| ADR-006 | Commands as Java Records | `CreateHumanPrincipalCommand`, `ActivatePrincipalCommand`, etc. |
| ADR-007 | Separate Command Handlers | `CreateHumanPrincipalCommandHandler` etc. |
| ADR-008 | Central Command Gateway | Gateway routes commands to handlers |
| ADR-011 | Thin Events | Event payloads contain IDs only; consumers fetch full state |
| ADR-013 | Outbox Pattern | Transactional outbox ensures event delivery reliability |
| ADR-014 | At-Least-Once Delivery | Consumers must be idempotent on principal events |
| ADR-016 | PostgreSQL | Primary data store for all tables |
| ADR-017 | Separate Read/Write Models | Read replica for GET /principals queries |
| ADR-020 | Dual-Key Pattern | Both `id` (UUID) and `publicId` (display key) on all aggregates |
| ADR-021 | OlUuid.create() | Surrogate key generation |
| ADR-067 | Extensibility | `customFields` JSONB extension pattern |

---

## 15. Appendix

### 15.1 Glossary

| Term | Definition |
|------|-----------|
| **Principal** | Any authenticated actor in the system: human, service, system, or device |
| **HumanPrincipal** | A principal representing a human user (employee, contractor, end customer) |
| **ServicePrincipal** | A principal representing a machine/service identity using OAuth2 client credentials |
| **SystemPrincipal** | A principal for internal platform-level system-to-system communication |
| **DevicePrincipal** | A principal representing a registered IoT or edge device |
| **TenantMembership** | The association between a Principal and a Tenant, including granted roles |
| **ApiKey** | A long-lived credential (token) associated with a Principal for API access |
| **Keycloak** | The external identity provider used by OpenLeap for authentication token issuance |
| **GDPR erasure** | Anonymization of all PII fields while retaining structural record for audit (Art. 17) |
| **Thin event** | An event containing only the aggregate ID and minimal trigger data (per ADR-011) |
| **Outbox** | A transactional inbox for events ensuring at-least-once delivery (per ADR-013) |
| **RLS** | Row-Level Security — PostgreSQL mechanism to enforce tenant isolation at DB level |
| **DPO** | Data Protection Officer — the responsible party for GDPR compliance |
| **SOX** | Sarbanes-Oxley Act — US financial reporting compliance requiring audit trails |
| **MFA** | Multi-Factor Authentication — additional verification beyond username/password |
| **FIDO2 / WebAuthn** | Web Authentication API standard for hardware security keys |
| **bcrypt** | Password hashing algorithm used for API keys and client secrets |
| **E.164** | International Telecommunication Union standard for phone number formatting |
| **IETF BCP 47** | Internet standard for language/locale tags (e.g., `de-DE`, `en-US`) |
| **IANA timezone** | Internet Assigned Numbers Authority timezone database identifiers |

### 15.2 References

| Reference | Location | Description |
|-----------|----------|-------------|
| IAM Suite Spec | `T1_Platform/iam/iam-suite-spec.md` | Parent suite architecture |
| Domain Service Template | `https://github.com/openleap-io/io.openleap.dev.concepts/blob/main/templates/platform/domain/domain-service-spec.md` v1.0.0 | Template used for this spec |
| Template Governance | `https://github.com/openleap-io/io.openleap.dev.concepts/blob/main/governance/template-governance.md` (GOV-TPL-001) | Template versioning policy |
| BFF Guideline | `https://github.com/openleap-io/io.openleap.dev.concepts/blob/main/governance/bff-guideline.md` (GOV-BFF-001) | BFF pattern for product UIs |
| Conceptual Stack | `https://github.com/openleap-io/io.openleap.dev.concepts/blob/main/conceptual-stack.md` | SPLE platform model |
| Artifact Catalog | `https://github.com/openleap-io/io.openleap.dev.concepts/blob/main/artifact-catalog.md` | Normative artifact definitions |
| ADR Catalog | `io.openleap.dev.guidelines` | All 63 ADR references |
| OpenAPI Contract | `contracts/http/iam/principal/openapi.yaml` | Machine-readable API contract |
| Event Schemas | `contracts/events/iam/principal/` | JSON Schema for all events |
| Keycloak Admin API | `https://www.keycloak.org/docs-api/latest/rest-api/` | External IdP integration reference |
| GDPR Article 17 | `https://gdpr-info.eu/art-17-gdpr/` | Right to erasure |
| GDPR Article 20 | `https://gdpr-info.eu/art-20-gdpr/` | Right to data portability |

### 15.3 Change Log

| Date | Version | Author | Changes |
|------|---------|--------|---------|
| 2026-04-03 | 1.1.0 | Architecture Team | Full upgrade to TPL-SVC v1.0.0 compliance (~95%). Added: full aggregate attribute tables (§3), complete BR catalog with detailed definitions (§4), 14 use cases with canonical format (§5), request/response bodies for 7 endpoints (§6), full event format for 5 events (§7), ER diagram + DDL for all 7 tables (§8), data classification + permission matrix (§9), full quality attributes including RTO/RPO (§10), feature-endpoint mapping + BFF hints + impact assessment (§11), all 6 extension types (§12), migration framework + deprecation policy (§13), 7 consistency checks + 5 decisions + 8 open questions + 15 ADR references (§14), glossary + full references (§15). |
| 2026-04-03 | 1.0.0 | Architecture Team | Initial thin specification |

---

**END OF SPECIFICATION**
