---
name: esos-security-review
description: Walk an artifact through the ESOS SECURITY specialist's review discipline - data classification, authentication, authorization (server-side, multi-tenant isolation), secrets handling, encryption in-transit and at-rest, audit and non-repudiation, input safety (SSRF / injection / validation), sensitive-data-in-logs, and threat surface. Use when reviewing or auditing a specification, constitution section, or candidate artifact for security quality, whether or not the esos-security subagent is invoked. Read-only - produces findings against the security.* message-key catalog. Embedded secrets are ALWAYS BLOCKING regardless of context.
---

# Skill: ESOS security review

You apply the SECURITY specialist's review discipline against an artifact.
The authoritative rules live in `rulesets/security.md`; this skill codifies
the walk and message-key catalog.

---

## How to work

1. Read `skills/esos-ruleset-resolution/SKILL.md` to load the primary
   ruleset (`rulesets/security.md`) and its includes (`domain/glossary.md`,
   `domain/tech-stack.md`, `shared/security-baseline.md`).
2. If the target lives in a plugin with a `severity_tier`, read
   `skills/esos-severity-tier/SKILL.md` and resolve the tier first.
3. Read the target. If unreachable, emit `security.target_unreachable`
   and stop.
4. Walk every focus area below in order. Silence equals pass.
5. Emit findings with `message_key: "security.<stable_key>"` per
   `skills/esos-finding-emission/SKILL.md`.
6. Close with the summary line.

---

## Focus areas to walk (in order)

### 1. Data sensitivity and classification

Every entity processed by the artifact carries a sensitivity class (Public /
Internal / Confidential / Restricted, or the domain's local taxonomy).
Missing sensitivity metadata on a sensitive entity = BLOCKING
(`security.sensitivity_missing`).

### 2. Authentication

- The artifact identifies the authentication mechanism. Roll-your-own auth
  without a written, justified exception = BLOCKING
  (`security.auth_custom_unjustified`).
- Session tokens / API keys / OAuth flows align with `domain/tech-stack.md`.

### 3. Authorization

- Authorization decisions MUST run server-side. Client-side-only gating =
  BLOCKING (`security.authz_missing` or `security.tenant_isolation_ui_only`).
- Multi-tenant isolation: tenant scoping MUST be enforced at the data
  access layer, not at the UI. UI-only scoping = BLOCKING
  (`security.tenant_isolation_ui_only`).
- Sensitive operations MUST carry an explicit authorization rule.

### 4. Secrets, tokens, credentials

**Embedded secrets are ALWAYS BLOCKING**, at every tier, in any file,
regardless of context. This rule is never tier-aware. Use
`security.secret_embedded`.

Heuristics to scan for: `BEGIN PRIVATE KEY`, `AKIA...`, `xoxb-...`,
`Bearer eyJ...`, JDBC strings with passwords, `password=`, `apiKey=`,
`client_secret=`, hex strings of length ≥ 32 in obvious credential
positions. "Sample", "placeholder", "redacted-looking" — all BLOCKING.

### 5. Encryption

- TLS in-transit for any network call. Plain `http://` outside an
  explicitly allowlisted intranet exception = BLOCKING
  (`security.tls_missing`).
- Encryption at-rest for sensitive data (per the sensitivity class).
  Encryption-disabled-by-default = BLOCKING
  (`security.encryption_disabled`).

### 6. Audit and non-repudiation

- Privileged operations (admin actions, financial transactions, data
  exports, configuration changes) MUST emit an audit log entry with actor,
  timestamp, operation, target, outcome. Missing = BLOCKING
  (`security.audit_missing_privileged`).
- The audit log MUST be immutable / append-only.

### 7. Input safety

- Server-Side Request Forgery: when the artifact fetches a URL, it MUST
  use controlled egress and deny private-IP ranges and cloud metadata
  endpoints. Missing protection = BLOCKING
  (`security.ssrf_unprotected`).
- Injection: SQL / NoSQL / shell injection paths MUST be closed via
  parameterized queries or equivalent. String concatenation into SQL =
  BLOCKING (`security.sql_concatenation`).
- Validation at trust boundaries: every external input is validated at
  the boundary, not in the deep handler.

### 8. Sensitive data in logs

PII / PHI / credentials / tokens MUST NOT appear in logs. Patterns that
write sensitive fields to logs = BLOCKING (`security.log_sensitive`).

### 9. Threat surface (lightweight threat model)

For new integrations or new attack surfaces, the artifact carries a
lightweight threat model: trust boundaries, data flows, key assets,
top-3 threats with mitigations. Missing on a non-trivial new surface =
BLOCKING (`security.threat_model_missing`).

### 10. Domain-specific focus

Walk any domain-specific focus area the local `rulesets/security.md`
names. Apply its keys.

---

## Message-key catalog

| Key | Fires when |
|---|---|
| `security.target_unreachable` | Target unreachable |
| `security.sensitivity_missing` | Sensitive entity lacks classification metadata |
| `security.auth_custom_unjustified` | Custom auth scheme without written justification |
| `security.authz_missing` | Sensitive operation lacks an authorization rule |
| `security.tenant_isolation_ui_only` | Multi-tenant scoping enforced only at UI layer |
| `security.secret_embedded` | Any embedded secret, token, key, or credential — ALWAYS BLOCKING |
| `security.tls_missing` | Plain HTTP without allowlisted exception |
| `security.encryption_disabled` | At-rest encryption disabled for sensitive data |
| `security.audit_missing_privileged` | Privileged operation lacks audit-log emission |
| `security.ssrf_unprotected` | Outbound fetch without SSRF protection |
| `security.sql_concatenation` | SQL built via string concatenation |
| `security.log_sensitive` | Sensitive field written to logs |
| `security.threat_model_missing` | New integration without a threat model |
| `security.include_unresolved` | `@esos-include` directive cannot be resolved |

---

## Boundaries

- Read-only. Findings only.
- Never invent rules outside `rulesets/security.md` and its includes.
- Embedded secrets / credentials are ALWAYS BLOCKING. Never demote.
- Never downgrade other BLOCKING-default findings without an explicit
  written justification in `constitution.md` §3.2 or §6.1.
