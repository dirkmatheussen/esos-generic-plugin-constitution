---
name: esos-compliance-review
description: Walk an artifact through the ESOS COMPLIANCE specialist's review discipline - privacy (retention, data subject rights, cross-border transfer), financial controls (segregation of duties, currency precision, immutable audit), internationalisation (locales, fallback, externalised strings, ISO formats, stable messageKeys), versioning discipline (breaking-change migration plans, API versioning), scoping and duplication, and applicable regulations cited specifically (vague "GDPR-compliant" is never enough). Use when reviewing or auditing a specification or constitution section for compliance quality. Read-only - produces findings against the compliance.* message-key catalog.
---

# Skill: ESOS compliance review

You apply the COMPLIANCE specialist's review discipline against an
artifact. The authoritative rules live in `rulesets/compliance.md`; this
skill codifies the walk and message-key catalog.

The COMPLIANCE specialist is **trigger-driven**: focus areas apply only
when their trigger condition is met. Skip categories that genuinely don't
apply, but document the skip in the run notes so the audit shows you
considered them.

---

## How to work

1. Read `skills/esos-ruleset-resolution/SKILL.md` to load the primary
   ruleset (`rulesets/compliance.md`) and its includes
   (`domain/glossary.md`, `domain/governance-policies.md`,
   `shared/normative-language.md`).
2. If the target lives in a plugin with a `severity_tier`, read
   `skills/esos-severity-tier/SKILL.md` and resolve the tier first.
3. Read the target. If unreachable, emit
   `compliance.target_unreachable` and stop.
4. Walk each focus area below; apply each only when its trigger is met.
5. Emit findings with `message_key: "compliance.<category>.<stable_key>"`
   per `skills/esos-finding-emission/SKILL.md`.
6. Close with the summary line.

---

## Focus areas to walk (in trigger order)

### 1. Privacy

**Trigger**: personal data is processed, stored, transmitted, or displayed.

- Data categories are identified (name, contact, identifiers, behavioural,
  special-category per applicable law). Missing = ADVISORY.
- Retention period is declared. Missing on a new personal-data store =
  BLOCKING (`compliance.privacy.retention_missing`).
- Data-subject-rights handling is addressed (access, rectification,
  erasure, portability, restriction, objection — as applicable).
  Missing where required = BLOCKING (`compliance.privacy.dsr_missing`).
- Cross-border transfer legal basis is declared when data crosses
  jurisdictions. Missing = BLOCKING
  (`compliance.privacy.cross_border_basis_missing`).
- Applicable regulation articles are cited specifically (e.g. GDPR
  Art. 5(1)(c), Art. 17). Vague "GDPR-compliant" without articles =
  ADVISORY at all tiers; promoted to BLOCKING when a new personal-data
  store is being established and the brief named GDPR.

### 2. Financial controls

**Trigger**: financial transactions, invoicing, or inventory valuation
are in scope.

- Segregation of duties on high-impact operations. Missing =
  BLOCKING (`compliance.financial.sod_missing`).
- Audit-trail immutability for financial actions. Mutable audit =
  BLOCKING (`compliance.financial.audit_not_immutable`).
- Retention period for financial records is declared.
- Currency code AND precision are explicit on every monetary field. Bare
  floats or implicit precision = BLOCKING
  (`compliance.financial.currency_unspecified`).

### 3. Internationalisation

**Trigger**: user-visible text or formatted values are in scope.

- Supported locales declared. Missing = BLOCKING
  (`compliance.i18n.locale_missing`).
- Fallback locale declared (typically `en-US` or `en-GB`).
- User-visible strings are externalised. Hardcoded literals =
  BLOCKING (`compliance.i18n.hardcoded_string`).
- API errors carry a stable `messageKey`. Missing = BLOCKING
  (`compliance.i18n.message_key_missing`).
- Timestamps in API are UTC ISO 8601; currency codes ISO 4217.
- AI-generated text honours the user's locale.

### 4. Versioning and change discipline

- The artifact declares its version ID and last-amended date.
- API versioning strategy is named (URL path / header / content negotiation).
  Missing on a new API = BLOCKING
  (`compliance.versioning.api_version_missing`).
- Breaking-API changes carry a migration / deprecation plan. Breaking
  change without plan = BLOCKING
  (`compliance.versioning.breaking_no_migration`).
- Constitution version is pinned in the catalog metadata.

### 5. Scoping and duplication

- Organizational scope is declared. Specs that quietly extend scope
  beyond the team's mandate = ADVISORY.
- Shared capabilities (auth, audit, billing) are reused, not
  duplicated. Silent duplication of a shared capability = BLOCKING
  (`compliance.scoping.silent_duplication`).

### 6. Domain-specific compliance

Walk any domain-specific focus area the local `rulesets/compliance.md`
names (e.g. clinical-trial reporting, recall traceability, regulated
algorithmic decisions). Apply its keys.

---

## Message-key catalog

| Key | Fires when |
|---|---|
| `compliance.target_unreachable` | Target unreachable |
| `compliance.privacy.retention_missing` | New personal-data store without retention period |
| `compliance.privacy.dsr_missing` | Personal-data processing without data-subject-rights handling |
| `compliance.privacy.cross_border_basis_missing` | Cross-border transfer without legal basis |
| `compliance.privacy.regulation_vague` | "GDPR-compliant" without specific article citations |
| `compliance.financial.sod_missing` | High-impact financial op without segregation of duties |
| `compliance.financial.audit_not_immutable` | Financial audit-trail is mutable |
| `compliance.financial.currency_unspecified` | Monetary field without ISO 4217 code or precision |
| `compliance.i18n.locale_missing` | User-visible UI without declared supported locales |
| `compliance.i18n.hardcoded_string` | User-visible string hardcoded rather than externalised |
| `compliance.i18n.message_key_missing` | API error without a stable `messageKey` |
| `compliance.versioning.api_version_missing` | New API without a versioning strategy |
| `compliance.versioning.breaking_no_migration` | Breaking API change without migration plan |
| `compliance.scoping.silent_duplication` | Shared capability silently duplicated |
| `compliance.include_unresolved` | `@esos-include` directive cannot be resolved |

---

## Boundaries

- Read-only. Findings only.
- Cite regulation articles specifically when `rulesets/compliance.md`
  names applicable regulations. Vague citations are themselves a defect.
- Skip a focus area only when its trigger is genuinely absent; note the
  skip in the run notes so the audit is transparent.
- Never downgrade BLOCKING without explicit written justification in
  `constitution.md` §3.2 or §6.1.
