# Glossary (Generic Base)

This glossary defines **cross-domain terminology** used across all ESOS specifications
and agent prompts. The derived domain constitution adds domain-specific terms in a
`## Domain Terms — {{ DOMAIN_NAME }}` section appended at the end.

Every specification under this constitution SHOULD use terms from this glossary
exactly. Inconsistent or undefined terminology is flagged as **ADVISORY** by the
ANALYST specialist.

---

## General Terms

| Term                  | Abbreviation | Definition                                                                                              |
| --------------------- | ------------ | ------------------------------------------------------------------------------------------------------- |
| Specification         | Spec         | A structured description of a feature or change, written for review before implementation.              |
| Constitution          | —            | The governance document that tells ESOS how to validate specifications under a given workspace.        |
| Workspace             | —            | The unit ESOS attaches a constitution to; typically corresponds to a product or product area.          |
| Functional Requirement| FR           | A numbered, normative, independently testable requirement (FR-001, FR-002, …) in a specification.      |
| Success Criterion     | SC           | A numbered, measurable, technology-agnostic outcome (SC-001, SC-002, …) in a specification.            |
| Acceptance Scenario   | AC           | A Given/When/Then scenario describing expected observable behaviour, owned by a user story.            |
| User Story            | —            | A short statement of user-visible value, scoped to a single actor and outcome.                         |
| Use Case              | UC           | A higher-level description of a recurring user-system interaction; specifications cite UC-xxx.         |
| Definition of Ready   | DOR          | The conditions a specification meets before implementation begins.                                     |
| Definition of Done    | DOD          | The conditions a feature meets before it is considered complete.                                       |
| Stakeholder           | —            | Anyone with an interest in the outcome of the change (business, ops, compliance, security, legal).     |

---

## Architecture & Integration Terms

| Term                          | Abbreviation | Definition                                                                                       |
| ----------------------------- | ------------ | ------------------------------------------------------------------------------------------------ |
| Application Programming Interface | API      | A service interface exposed for programmatic access.                                              |
| Domain Event                  | —            | An event scoped to a single product domain, used for internal choreography.                      |
| Integration Event             | —            | An event shared across domain boundaries or with external systems.                               |
| Canonical Data Model          | CDM          | An agreed schema for shared business objects.                                                    |
| Service-to-Service            | S2S          | Communication between backend services, typically authenticated via platform identity.           |
| Idempotency Key               | —            | A client-supplied identifier used to safely retry a mutating operation without double-effect.    |
| Optimistic Concurrency        | —            | A concurrency-control pattern using a version field to detect conflicting writes.                |
| Event Envelope                | —            | The common metadata wrapper for events (CloudEvents is a typical default).                       |
| Backwards Compatibility       | BC           | The property that a new version of an interface still works for existing clients.                |

---

## Security & Data Terms

| Term                                | Abbreviation | Definition                                                                                       |
| ----------------------------------- | ------------ | ------------------------------------------------------------------------------------------------ |
| Personally Identifiable Information | PII          | Data that can identify an individual, directly or in combination with other data.                |
| Protected Health Information        | PHI          | Health-related personal data subject to specialized regulation (e.g. HIPAA).                     |
| Data Subject                        | —            | The individual whose personal data is being processed (GDPR term).                               |
| Data Classification                 | —            | A label indicating data sensitivity (e.g. Public / Internal / Confidential / Restricted).        |
| Audit Log                           | —            | An immutable record of security-relevant events.                                                 |
| Segregation of Duties               | SoD          | Control requiring that a single principal cannot both initiate and approve a sensitive operation.|
| Multi-Tenancy                       | —            | A deployment where multiple distinct tenant organizations share infrastructure with isolation.   |
| Server-Side Request Forgery         | SSRF         | An attack where a server is induced to make outbound requests on the attacker's behalf.          |
| Trust Boundary                      | —            | A perimeter at which data crossing requires authentication, authorization, or validation.        |
| Threat Model                        | —            | A structured description of assets, threats, controls, and residual risk for a feature.          |

---

## Delivery & Operations Terms

| Term                                  | Abbreviation | Definition                                                                                       |
| ------------------------------------- | ------------ | ------------------------------------------------------------------------------------------------ |
| Pull Request                          | PR           | A proposed change submitted for review before merging.                                           |
| Continuous Integration                | CI           | Automated build and test on every commit / PR.                                                   |
| Continuous Delivery                   | CD           | Automated deployment to staging or production after CI passes.                                   |
| Static Application Security Testing   | SAST         | Automated scanning of source code for security issues.                                           |
| Dynamic Application Security Testing  | DAST         | Automated scanning of a running application for security issues.                                 |
| Software Composition Analysis         | SCA          | Automated scanning of dependencies for known vulnerabilities.                                    |
| Service Level Objective               | SLO          | A target for a service's reliability or performance metric.                                      |
| Service Level Agreement               | SLA          | A contractual commitment to one or more SLOs.                                                    |
| Recovery Time Objective               | RTO          | The maximum acceptable time to restore service after disruption.                                 |
| Recovery Point Objective              | RPO          | The maximum acceptable amount of data loss measured in time.                                     |
| Dead-Letter Queue                     | DLQ          | A holding queue for messages that cannot be processed after retries.                             |

---

## ESOS-Specific Terms

| Term                       | Abbreviation | Definition                                                                                                |
| -------------------------- | ------------ | --------------------------------------------------------------------------------------------------------- |
| Specialist Class           | —            | One of the five ESOS agent classes: BUSINESS / SECURITY / COMPLIANCE / IMPLEMENTATION / TEST verification.|
| Pipeline                   | —            | The ordered sequence of specialist steps a validation run executes (`constitution.md` §3).                 |
| Confidence Threshold       | —            | The minimum specialist confidence below which corrective paths trigger (`constitution.md` §4).             |
| Constitution Waiver        | —            | An explicit declaration that a specialist class is skipped, with recorded rationale (`constitution.md` §3.2 + §7). |
| Specification Binding      | —            | The snapshot of specifications captured at run start; the run validates against this binding.             |
| Trace Event                | —            | An immutable audit-trail entry (`constitution.md` §7).                                                     |
| Channel Parity             | —            | The property that MCP-initiated and UI-initiated runs produce identical outcomes for the same inputs.     |

---

## Domain Terms — `{{ DOMAIN_NAME }}`

<!-- esos:domain-customization -->

The derived constitution adds at least 10 domain-specific terms here, drawn from the
derivation brief's `KEY_DOMAIN_TERMS`. Each entry follows the same table format:

| Term | Abbreviation | Definition |
| ---- | ------------ | ---------- |
| `{{ DOMAIN_TERM }}` | `{{ ABBR }}` | `{{ DEFINITION }}` |

<!-- /esos:domain-customization -->

---

Add project-specific terms as they emerge; keep definitions concise. Flag specifications
that use undefined or ambiguous terms as **NEEDS CLARIFICATION**.
