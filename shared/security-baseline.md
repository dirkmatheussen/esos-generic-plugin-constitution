# Security Baseline

A security checklist included by the SECURITY and CODING specialists, applied to every
specification under any constitution derived from this base. Each item applies when its
trigger condition is met — not every item applies to every specification.

---

## Identity & Access

- [ ] User authentication uses a recognized mechanism (OAuth 2.0 / OIDC, or the
      platform standard declared in `domain/tech-stack.md`).
- [ ] Service-to-service calls use platform-provided identity (managed identity,
      workload identity); long-lived shared secrets are not used where platform
      identity is available.
- [ ] Authorization rules are declared for every sensitive operation: who may do what.
- [ ] Multi-tenant / multi-user isolation is enforced server-side; UI-only filtering
      is not used as a security control.
- [ ] Privileged operations use just-in-time access where it is available.
- [ ] Default-deny: a request without a known role is denied, not granted a default
      role.
- [ ] Token lifetimes are bounded; refresh-token rotation is in place where applicable.

## Data Protection

- [ ] Every data entity in scope is classified (Public / Internal / Confidential /
      Restricted, or the equivalent scheme declared in
      `domain/governance-policies.md`).
- [ ] Data in transit over untrusted networks uses TLS (1.2+; prefer 1.3).
- [ ] Data at rest in stores holding sensitive data uses the platform's standard
      encryption.
- [ ] Secrets, tokens, signing keys, and connection strings live in a managed secret
      store — never plaintext in code, config, repos, or specifications.
- [ ] Personal data fields carry sensitivity / retention metadata at the data
      definition (column comment, schema annotation, or the equivalent for the chosen
      persistence engine).
- [ ] Backup data inherits the same classification and protection as the primary
      store.

## Audit & Non-Repudiation

- [ ] Security-relevant events are audited:
  - Authentication failures and successes (configurable verbosity).
  - Authorization denials.
  - Privilege use (e.g. admin actions, break-glass).
  - Data corrections that materially change state.
  - High-impact approvals (financial, configuration changes, data exports).
- [ ] Each audit record captures: who, what, when (UTC ISO 8601), on which entity,
      from which channel.
- [ ] Audit records are immutable from the application's perspective; corrections
      generate new audit records, never overwrite.
- [ ] Audit trail retention matches the applicable audit regime (declared in
      `domain/governance-policies.md`).

## Input Safety

- [ ] External URL inputs are protected against SSRF: allowlist + controlled egress +
      private-IP-range denial + cloud-metadata-endpoint denial.
- [ ] Database queries use parameterized statements; no string concatenation, no
      dynamic-SQL composition from user input.
- [ ] External / user input is validated at trust boundaries (schema validation, size
      limits, content-type checks).
- [ ] File uploads declare type / size limits and content-type validation; uploaded
      files are scanned before processing where applicable.
- [ ] Sensitive values (PII, credentials, tokens, pricing, health data) do not appear
      in logs, trace attributes, or metrics. Logger configuration enforces filtering;
      developer discipline alone is insufficient.

## API Surface Hygiene

- [ ] Error responses do not leak internal exception messages, stack traces, or
      configuration values.
- [ ] Pagination is declared for list operations that can grow.
- [ ] Rate limiting is declared for unauthenticated and tenant-scoped endpoints.
- [ ] CORS policy is explicit; no wildcard for credentialed endpoints.
- [ ] Security headers (HSTS, CSP, X-Content-Type-Options, X-Frame-Options where
      applicable) are configured per platform standard.

## DevSecOps Gates

- [ ] CI runs the team's standard SAST gate.
- [ ] CI runs dependency / SCA scanning with an agreed critical-CVE policy.
- [ ] CI runs secret detection (pre-commit and CI).
- [ ] Container or artifact builds are scanned before deployment.
- [ ] No specification proposes bypassing or disabling the team's security gates
      without a clear, signed-off reason in `assumptions`.
- [ ] Production deployment artifacts are pinned by digest, not floating tag.

---

## Common Threats and Default Controls

| Threat                       | Typical Control                                                                  |
| ---------------------------- | -------------------------------------------------------------------------------- |
| Unauthorized data access     | Server-side authorization enforcement; token scope validation.                   |
| PII exposure                 | Data classification; filtered logging; response-field allow-listing.             |
| Financial fraud              | Segregation of duties; dual authorization for high-value transactions.           |
| Supply chain attack          | Dependency scanning; pinned versions; trusted registries; SBOM generation.       |
| Credential leakage           | Pre-commit secret detection; managed secret store; rotation policy.              |
| SSRF                         | URL allowlist; egress firewall; private-IP-range denial; metadata-endpoint deny. |
| Privilege escalation         | Tightly-scoped role definitions; just-in-time privileged access; review of role assignments. |
| Injection (SQL / NoSQL / OS) | Parameterized queries; input validation at trust boundaries; principle of least query privilege. |
| Cross-site scripting         | Output encoding by default in the UI framework; CSP enforcement.                 |
| Cross-site request forgery   | Same-site cookies + anti-CSRF tokens for state-changing requests.                |
| Insecure deserialization     | Deny untrusted-source deserialization; allow-list types.                         |
| Prompt injection (LLM)       | Output filtering; tool-use scoping; input provenance tracking.                   |

---

## Domain-Specific Additions

<!-- esos:domain-customization -->

The derived constitution adds domain-specific checklist items here. Examples:

- Automotive: VIN handling avoids enumeration disclosure; OBD telemetry is signed and
  rate-limited; recall data is access-controlled per role.
- Healthcare: PHI access logged with patient identifier hashed in audit; break-glass
  use is reviewed weekly.
- Fintech: PAN never stored in clear; all card-data flows annotated with PCI scope.
- AI/ML: model weights stored in a controlled artifact registry; inference inputs
  inspected for prompt-injection patterns.

<!-- /esos:domain-customization -->
