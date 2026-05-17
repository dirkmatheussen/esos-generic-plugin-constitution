# Governance Policies (Generic Base)

This document captures **cross-domain governance principles** that apply to every
specification under any constitution derived from this base. The derived constitution
adds domain-specific policies (regulations, audit regimes, retention periods,
classification mappings) by extending each section.

---

## 1. Ways of Working

- Specifications are the contract between business intent and engineering execution.
  They are produced before implementation begins and are revised as understanding
  improves.
- Cross-functional review (product, engineering, security, compliance, operations) is
  the norm; specialist agents surface findings as input to that review, not as a
  replacement.
- Findings are addressed by revising the specification and re-running validation, not
  by overriding agents.
- Teams own their products end-to-end; constitutions define the boundary of "good
  enough" rather than the path to it.

---

## 2. Specification Responsibilities

- Non-trivial changes MUST have a specification before implementation begins. The
  specification lives in version control alongside the code, or in a designated
  document store with a stable link from the work item.
- Specifications describe **WHAT** and **WHY**; implementation detail belongs in
  technical design documents.
- Each specification cites the use case(s) it implements and lists deltas from the
  current state.
- Each specification carries a version identifier and a last-amended date.

---

## 3. Change Management

- Production changes follow the team's release process. The constitution does not
  prescribe the release tool — but it does require a documented rollback plan for
  changes affecting customer-visible systems.
- Emergency changes are permitted; the post-change record (incident-style writeup) is
  produced within the team's standard window.
- Breaking changes to public interfaces require a migration / deprecation plan with a
  named sunset date.

---

## 4. Security-by-Design

- Security is a normal-design concern, not a separate stage. The SECURITY specialist
  reviews every specification; findings flow back to the spec author.
- A lightweight threat model is required for features that introduce a new data flow,
  trust boundary, or external integration.
- Data sensitivity is identified close to the data — not buried in prose.

---

## 5. Data Classification

A four-tier classification is the default. The derived constitution MAY adopt a
different scheme (Govt classifications, organization-specific categories) — but every
entity in scope MUST carry a label.

| Classification | Guidance                                                           |
| -------------- | ------------------------------------------------------------------ |
| **Public**     | No harm if disclosed externally.                                   |
| **Internal**   | Intended for internal use; limited harm if disclosed.              |
| **Confidential** | Material harm if disclosed; access limited to authorized roles. |
| **Restricted** | Severe harm or legal consequences if disclosed; need-to-know only. |

Label entities at the level matching actual risk — not the worst-case imaginable.

---

## 6. Privacy

**When applicable** (personal data is processed, stored, transmitted, or displayed):

- Identify the personal data categories explicitly.
- Declare a retention period.
- Address data subject rights (access, rectification, erasure, portability, objection)
  — or note as out of scope with a reason.
- Declare the legal basis for cross-border transfers.

Default retention guidance (the derived constitution refines per regulation):

| Record Type             | Default Retention | Notes                                                |
| ----------------------- | ----------------- | ---------------------------------------------------- |
| Financial records       | 7 years           | Adjust per applicable audit regime.                  |
| Operational records     | 3 years           | Server logs, application logs.                       |
| Personal data           | Per regulation    | GDPR / CCPA / domain-specific minimum-necessary.     |
| Audit logs              | Per regulation    | Often 6-7 years for finance, 6 years for HIPAA.      |

<!-- esos:domain-customization -->

The derived constitution lists the actual retention periods enforced for this domain
and the regulation that drives each.

<!-- /esos:domain-customization -->

---

## 7. Financial Controls

**When applicable** (financial transactions are in scope):

- Identify the financial process affected and which control applies.
- Apply segregation of duties on high-impact operations: no single principal both
  initiates and approves.
- Keep audit trails immutable; retain for the period required by the applicable audit
  regime.
- Currency-bearing values carry currency code (ISO 4217) and explicit precision.

---

## 8. Architecture Principles

These are sensible defaults — derived constitutions tighten or replace as the domain
warrants.

| Principle                                        | Implication                                                                                |
| ------------------------------------------------ | ------------------------------------------------------------------------------------------ |
| **API-first**                                    | Business capabilities are exposed as APIs before the UI is built.                          |
| **Domain-owned data**                            | Products own their data; avoid sharing databases across product boundaries.                |
| **Event-driven where decoupling matters**        | Use events when synchronous coupling would otherwise create operational coupling.          |
| **Zero-trust-ish**                               | Authenticate service-to-service calls; do not assume network position grants trust.        |
| **Observability-by-design**                      | Logs, metrics, traces are part of the feature — not added later.                           |
| **Idempotency for retry-prone operations**       | Mutating operations whose retries are plausible declare idempotency semantics.             |
| **Migration-safe data changes**                  | Schema changes apply expand-and-contract or equivalent for zero-downtime deployments.      |

---

## 9. Quality Gates

### 9.1 Definition of Ready

A specification is ready for development when:

- [ ] Stakeholders have reviewed and agreed.
- [ ] Every mandatory section is present and non-empty.
- [ ] All ESOS specialist findings flagged BLOCKING are resolved.
- [ ] Dependencies and assumptions are captured.
- [ ] Open questions are resolved or explicitly parked with an owner and a date.

### 9.2 Definition of Done

A feature is done when:

- [ ] Acceptance scenarios pass in CI.
- [ ] Coverage of new code meets the team's standard.
- [ ] SAST, dependency scan, and secret detection pass with no critical findings.
- [ ] Deployed per the team's release process with a documented rollback path.
- [ ] Monitoring / alerting is in place for new user journeys.
- [ ] Audit records exist for security-relevant operations introduced.

The derived constitution MAY add gates (regulatory sign-off, accessibility audit,
safety case approval); it MUST NOT silently lower these.

---

## 10. Audit & Traceability

Audit records MUST capture:

- **Who** — authenticated identity or service principal.
- **What** — the action performed and the entity affected.
- **When** — UTC timestamp, ISO 8601.
- **From which channel** — API, UI, batch, event consumer, MCP, etc.
- **Outcome** — success / failure, with a stable error / event identifier.

Audit records MUST be immutable from the application's perspective. Corrections
themselves generate new audit records.

---

## 11. Domain-Specific Regulations and Audit Regimes

<!-- esos:domain-customization -->

The derived constitution MUST list every regulation, standard, or audit regime that
applies to this domain. For each one, describe:

- **Name and citation** (e.g. GDPR Art. 5(1)(c); ISO 26262:2018 §7; HIPAA Security Rule
  45 CFR §164.308).
- **Scope** — which kinds of changes it applies to.
- **Controls implemented** — which constitution rules implement each requirement.
- **Evidence expectations** — what artifacts must exist as evidence (audit logs,
  signed approvals, test reports, model cards).

Vague references ("we comply with GDPR") are insufficient. Every cited regulation MUST
map to specific controls and evidence.

<!-- /esos:domain-customization -->

---

## 12. Constitution Maintenance

- The constitution itself follows semantic versioning (`constitution.md` §10).
- Amendments require sign-off appropriate to the change scope (MAJOR > MINOR > PATCH).
- Workspaces re-attach to a new constitution version explicitly; they are not
  auto-migrated.
- The constitution maintainer keeps a changelog (the Git log of the constitution
  repository is sufficient).
