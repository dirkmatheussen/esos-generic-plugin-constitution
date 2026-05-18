---
name: esos-validate-constitution
description: Use when the user wants to validate, audit, review, or check a candidate domain ESOS plugin for conformance to the Generic Plugin Constitution (v4.0.0). Triggers on phrases like "validate this plugin", "audit the healthcare constitution", "check if this domain plugin is publishable", "review my fintech plugin". Produces a structured findings report with BLOCKING / ADVISORY items per the numbered checks in VALIDATION.md - including v3.2.0's companion-document checks (GEN-022, GEN-023, GEN-122, GEN-123, GEN-124), v3.1.0's marketplace.json check, and v3.0.0's plugin.json / self-containment / CHANGELOG checks. v4.0.0 audits the published §2.2.4 YAML regardless of whether it was generated from the old inline brief field or v4.0.0's templates-directory discovery model.
---

# Skill: Validate a domain ESOS plugin

You audit a candidate domain plugin against:

1. The **Generic Plugin Constitution v4.0.0** (this base, the foundational
   document) — `constitution.md`, `rulesets/`, `domain/`, `shared/`.
2. The **numbered checks** in `VALIDATION.md` (Sections A → D).
3. The **plugin manifest contract** in `plugin.json`.

The long-form audit prompt is in `VALIDATION.md`. This skill orchestrates
the workflow; `VALIDATION.md` defines the contract.

---

## Step 1 — Identify the target

Ask the user (if not already supplied):

- **Path to the candidate plugin** directory (e.g.
  `../healthcare-patient-records/`).
- **Optional**: the derivation brief used to create it (helps with checks
  GEN-106, GEN-108, GEN-109, GEN-121).
- **Optional**: prior published version of the same `logical_id` (for
  GEN-110 monotonicity check).

Verify the path exists and contains at minimum `constitution.md` and
`plugin.json`. If either is missing, stop and ask — both are required for
v3.0.0 conformance.

### Resolve the severity tier

Before walking any check, follow
`skills/esos-severity-tier/SKILL.md` to resolve the candidate's
`severity_tier`:

1. Read `constitution.md` §8 (and cross-check against
   `plugin.json` if it mirrors the tier).
2. If present and valid → use it; record `tier_source: declared`.
3. If absent or malformed → default to `strict`; record
   `tier_source: defaulted`; queue a GEN-120 ADVISORY finding.
4. If a brief is supplied and disagrees with the candidate → use the
   candidate's value; record `tier_source: declared`; queue a GEN-121
   BLOCKING finding.

State the resolved tier to the user before walking checks so they see
what contract the audit is enforcing.

---

## Step 2 — Load the audit context

Read these files before walking checks:

- `VALIDATION.md` — the complete check list (Sections A → D).
- `skills/esos-finding-emission/SKILL.md` — for the YAML emission
  contract used by every finding below.
- `skills/esos-ruleset-resolution/SKILL.md` — for primary-vs-fallback
  and tighten-not-relax discipline; used by GEN-101.
- `skills/esos-severity-tier/SKILL.md` — for the per-tier matrix; used
  by every tier-aware check.
- The candidate's `plugin.json` (parse JSON; validate against
  GEN-017).
- The candidate's `constitution.md` — read fully.
- The candidate's
  `rulesets/{analyst,security,compliance,coding,testing}.md`.
- The candidate's `domain/{glossary,governance-policies,tech-stack}.md`.
- The candidate's
  `shared/{normative-language,security-baseline,acceptance-criteria-format}.md`.
- The candidate's
  `agents/esos-{analyst,security,compliance,coding,testing}.md`.
- The candidate's `skills/` directory — verify all 9 required skills
  exist (3 common + 5 review + `esos-validate-constitution`) and
  `esos-create-constitution` is **absent**.
- `rulesets/<role>.md` from the generic base for each specialist (for
  severity-floor comparison in GEN-101).
- The candidate's `templates/` directory if present (for GEN-023).
- Any candidate specification(s) supplied alongside, including their
  `companion_documents:` blocks (for GEN-122 / GEN-123). Skip silently if
  no specification is supplied — `GEN-122` and `GEN-123` only fire
  against a spec.

---

## Step 3 — Walk every check

Process the checks in order — Section A → Section B → Section C →
Section D. For each check:

1. State the check ID.
2. Examine the relevant files.
3. If the check fires, emit a finding in the YAML format from
   `esos-finding-emission`.
4. If the check passes, do not emit anything (silence = pass).

**Section coverage**:

- **Section A (GEN-001 to GEN-023)** — Foundational conformance.
  **BLOCKING at every tier — never relaxed.** Includes the v3.0.0
  additions: GEN-017 (`plugin.json` well-formed), GEN-018
  (self-containment), GEN-019 (`esos-create-constitution` absent),
  GEN-020 (`CHANGELOG.md` populated). Adds in v3.2.0: GEN-022
  (companion-document kind declarations complete), GEN-023
  (companion-document templates resolve).
- **Section B (GEN-101 to GEN-124)** — Generic baseline. Severities as
  listed; several entries are **tier-aware** and resolve via the
  matrix in `VALIDATION.md`. Section B includes GEN-103a (CODING role
  preserved) and GEN-103b (TESTING role preserved), which are BLOCKING
  at every tier. Adds in v3.2.0: GEN-122 (companion-document presence),
  GEN-123 (recognition signature), GEN-124 (extractor coverage).
- **Section C (GEN-201 to GEN-206)** — Cross-file consistency. Mostly
  BLOCKING; GEN-205 is tier-aware.
- **Section D (GEN-301 to GEN-306)** — Heuristic quality. ADVISORY at
  `strict` / `normal`; **informational** at `relaxed` (counted in
  `total_informational`, not `total_advisory`).

Apply the tier matrix from `VALIDATION.md` when resolving each
tier-aware check's severity. Do **not** invent checks not in
`VALIDATION.md`. Do **not** skip checks listed there. Do **not** relax
any Section A check or any "Never tier-aware" check regardless of the
resolved tier.

---

## Step 4 — Spawn specialist subagents (optional, for deep checks)

For checks that require interpreting the candidate's specialist
ruleset against the base ruleset, you MAY spawn the matching subagent
(which itself composes the review skill + common skills):

| Check pattern | Subagent | Review skill it invokes |
|---|---|---|
| Severity-floor in `rulesets/analyst.md` | `esos-analyst` | `esos-analyst-review` |
| Severity-floor in `rulesets/security.md` | `esos-security` | `esos-security-review` |
| Severity-floor in `rulesets/compliance.md` | `esos-compliance` | `esos-compliance-review` |
| 14 focus areas / role framing in `rulesets/coding.md` (GEN-103a) | `esos-coding` | `esos-coding-review` |
| 14 focus areas / role framing in `rulesets/testing.md` (GEN-103b) | `esos-testing` | `esos-testing-review` |

Use them sparingly — most checks in `VALIDATION.md` are textual (file
existence, token presence, header structure) and do not require a
subagent. Spawn one when the check is genuinely about the specialist's
domain expertise (e.g. "is this a real N+1 risk?", "is this audit
pattern compliant with the base?").

---

## Step 5 — Emit the report

After all checks, emit the report per
`skills/esos-finding-emission/SKILL.md`:

```yaml
findings:
  - check_id: GEN-NNN
    severity: BLOCKING | ADVISORY
    location:
      file: "<relative path within candidate dir>"
      section: "<section header or anchor>"
      line: <integer or "n/a">
    message: "<what is wrong, in one sentence>"
    message_key: "<stable identifier, e.g. constitution.missing_mandatory_section>"
    remediation: "<actionable fix, in one or two sentences>"
  - ...

summary:
  severity_tier: "<strict|normal|relaxed>"
  tier_source: "<declared|defaulted|brief>"
  total_blocking: <int>
  total_advisory: <int>
  total_informational: <int>
  publishable: <true if blocking == 0 else false>
  files_audited: [<list of file paths>]
  checks_executed: <int>
```

The candidate is **publishable only if `total_blocking == 0`**. The
tier under which publishability was assessed is recorded in
`severity_tier` — re-auditing at a stricter tier may produce additional
BLOCKING findings.

---

## Step 6 — Self-check before delivering

- [ ] Every check in Sections A, B, C, D was run against every relevant
      file.
- [ ] No finding has severity outside `{BLOCKING, ADVISORY}`.
- [ ] Every finding has a stable `message_key`.
- [ ] The summary's `publishable` field is `true` only if
      `total_blocking == 0`.
- [ ] You did not invent checks not listed in `VALIDATION.md`.
- [ ] You did not skip any check listed in `VALIDATION.md`.
- [ ] If you treated a BLOCKING-default check as ADVISORY because of a
      written justification in the candidate, you cited the
      justification's location in the `remediation` field.
- [ ] `severity_tier` was resolved once at the start and applied
      consistently. The summary records both `severity_tier` and
      `tier_source`.
- [ ] No Section A check or any "Never tier-aware" check was demoted
      on tier grounds.
- [ ] At `relaxed`, Section D findings appear in `total_informational`,
      not `total_advisory`.
- [ ] The plugin manifest checks (GEN-017), self-containment
      (GEN-018), no-create-skill (GEN-019), and CHANGELOG presence
      (GEN-020) were all walked.
- [ ] GEN-022 / GEN-023 (companion-document declarations and template
      refs) were walked even if no candidate spec is in scope.
- [ ] GEN-122 / GEN-123 were skipped silently when no candidate spec
      was supplied (they cannot fire against a constitution-only
      audit).
- [ ] GEN-124 (extractor coverage) was evaluated for every
      `allowed_formats` extension declared in §2.2.4.

If any of the above fails, redo the audit — don't deliver a partial
report.

---

## Step 7 — Suggest remediation

If there are BLOCKING findings, group them by file and offer to fix
the straightforward ones (token replacements, missing sections,
inheritance declarations, manifest fields). For BLOCKING findings that
require domain judgment (e.g. "Specialist waiver lacks rationale",
"Regulation cited without specific articles"), summarize what the user
needs to decide — don't invent the answer.

If there are no BLOCKING findings, congratulate the user briefly and
remind them of the next step (publish to ESOS catalog as a new
plugin version, or create a Pull Request to the plugin repo).

---

## Boundaries

- This skill produces a findings report. It does NOT modify the
  candidate by default — apply fixes only if the user explicitly
  asks.
- This skill does NOT publish or unpublish from the ESOS catalog
  (that's an ESOS admin action).
- This skill does NOT replace ESOS runtime validation — it audits the
  plugin itself, not specifications validated against it.
