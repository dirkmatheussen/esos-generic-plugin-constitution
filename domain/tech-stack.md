# Technology Stack (Generic Base)

This document lists **stack-agnostic principles**. The derived constitution replaces
the placeholder choices with concrete technologies, frameworks, and versions.

A specification under this constitution MUST align with the derived stack file. Stack
deviations MUST be declared in the specification's `assumptions` section with rationale.

---

## 1. Principles

- Use well-supported, actively maintained libraries.
- Prefer managed services over self-hosted where operationally reasonable.
- Keep secrets out of source control — always resolve from a managed secret store.
- Minimize the number of distinct runtimes the team has to operate.
- Pin versions in production deployment artifacts (digest, not floating tag).
- Prefer interoperable open standards (OpenAPI, OpenTelemetry, CloudEvents, OAuth 2.0,
  OIDC) over proprietary equivalents.

---

## 2. Stack Selection

<!-- esos:domain-customization -->

The derived constitution fills in the actual technologies. Replace placeholders with
concrete choices. If a layer is not in scope, write `Not in scope` and the reason.

| Layer                          | Choice                                       | Notes                                                |
| ------------------------------ | -------------------------------------------- | ---------------------------------------------------- |
| **API style**                  | `{{ API_STYLE }}` (e.g. REST + OpenAPI 3.1)  | Machine-readable contract required.                  |
| **Backend language(s)**        | `{{ BACKEND_LANG }}`                         | Pin to currently-supported LTS / stable version.     |
| **Backend framework(s)**       | `{{ BACKEND_FRAMEWORK }}`                    | e.g. Spring Boot 3.x, ASP.NET 8, FastAPI, NestJS.    |
| **Frontend language**          | `{{ FRONTEND_LANG }}` (typically TypeScript) |                                                      |
| **Frontend framework**         | `{{ FRONTEND_FRAMEWORK }}`                   | React, Vue, Angular, Svelte — choose one per app.    |
| **Relational database**        | `{{ RELATIONAL_DB }}`                        | e.g. PostgreSQL 16 (managed).                        |
| **Document / wide-column DB**  | `{{ DOCUMENT_DB }}` or `Not in scope`        |                                                      |
| **Cache**                      | `{{ CACHE }}` (e.g. Redis, managed)          |                                                      |
| **Messaging**                  | `{{ MESSAGE_BROKER }}`                       | e.g. Kafka, RabbitMQ, Service Bus, SNS/SQS.          |
| **Event envelope**             | `{{ EVENT_ENVELOPE }}` (CloudEvents default) |                                                      |
| **Identity provider**          | `{{ IDP }}`                                  | e.g. Azure AD / Entra, Okta, Keycloak, Auth0.        |
| **Service-to-service identity**| `{{ S2S_IDENTITY }}`                         | Managed identity / workload identity preferred.      |
| **Secret store**               | `{{ SECRET_STORE }}`                         | e.g. Azure Key Vault, AWS Secrets Manager, Vault.    |
| **IaC**                        | `{{ IAC_TOOL }}`                             | Terraform / Bicep / Pulumi / CDK.                    |
| **CI/CD**                      | `{{ CICD }}`                                 | The team's standard pipeline platform.               |
| **Observability**              | `{{ OBS_STACK }}`                            | OpenTelemetry → backend (App Insights, Datadog, …).  |
| **Migration tool**             | `{{ MIGRATION_TOOL }}`                       | Liquibase / Flyway / Alembic / Prisma migrate.       |
| **Container runtime**          | `{{ CONTAINER_RUNTIME }}` or `Not in scope`  | If containerised.                                    |

<!-- /esos:domain-customization -->

---

## 3. Identity & Access

- External-facing APIs use **OAuth 2.0 / OIDC** by default unless the derived stack
  declares otherwise with rationale.
- Service-to-service calls use **platform-provided identity** (managed identity,
  workload identity) over long-lived shared secrets.
- Custom identity / password stores are not permitted without an explicit, signed-off
  exception in `assumptions`.

---

## 4. Security Scanning

The CI pipeline MUST run, at minimum:

- **Secret detection** in pre-commit and CI (gitleaks, trufflehog, or platform
  equivalent).
- **Dependency / SCA scan** in CI with an agreed critical-CVE policy.
- **SAST** in CI for production code.
- **Container image scan** for containerised deployments, before promotion to
  production.
- **DAST** for externally exposed APIs (scheduled or per-release; need not be per-PR).

The derived constitution names the actual tooling.

---

## 5. Data Residency

When applicable (e.g. EU personal data, regulated data):

- Data residency MUST be declared in the specification's `security_compliance`
  section.
- Replicate only the data that must be replicated, only to jurisdictions that are
  allowed.
- Cross-border transfers require a documented legal basis (GDPR SCC, adequacy, …).

The derived constitution lists the jurisdictions in scope and the residency rules.

---

## 6. Deviation Process

If a change requires technology outside the declared stack:

1. Note the deviation and its reason in the specification's `assumptions` section.
2. Confirm the new dependency fits the team's operational model (scanning, patching,
   on-call, license).
3. Record the decision so future specifications can reference it.

There is no formal exception board for routine deviations; the team owns its stack.
For deviations that introduce a new runtime, a new persistence engine, or a new
identity provider, sign-off from the engineering lead AND security lead is required.

---

## 7. Versioning of the Stack File

This file is itself versioned in the constitution repository. Material changes to the
stack (adding or removing a runtime, changing the IDP, changing the migration tool)
trigger a MINOR or MAJOR version bump of the constitution per `constitution.md` §10.
