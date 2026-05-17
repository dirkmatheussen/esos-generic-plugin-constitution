---
name: esos-finding-emission
description: ESOS shared discipline for emitting findings. Use whenever a specialist subagent, a review skill, or the validate workflow produces audit findings against an ESOS artifact (specification, constitution, ruleset, plugin, agent file). Defines the YAML finding shape, the silence-equals-pass rule, the BLOCKING / ADVISORY discipline, message-key naming, the closing summary line, and target-unreachable handling. The contract every ESOS specialist obeys when reporting what they found.
---

# Skill: ESOS finding emission

You produce ESOS findings. This skill captures the **shared emission discipline**
that every specialist subagent, every review skill, and the validate workflow
obeys. The rules below are intentionally written once so they cannot drift
across specialists.

Read this skill on demand. You do not have to remember it; it is part of the
plugin payload, available alongside whichever skill or subagent is running.

---

## 1. The unit of output is the *finding*

A finding describes one specific defect at one specific location. Findings are
**append-only** within a run — emit one per defect.

Standard YAML shape — **always** use exactly these field names:

```yaml
- severity: BLOCKING | ADVISORY
  location:
    file: "<relative path within the plugin or artifact directory>"
    section: "<section header or anchor>"
    line: <integer or "n/a">
  message: "<what is wrong, in one sentence>"
  message_key: "<stable identifier, e.g. analyst.fr_untestable>"
  remediation: "<actionable fix, in one or two sentences>"
```

If the run is a `VALIDATION.md` audit, prepend `check_id: GEN-NNN` to each
finding so it ties back to a numbered check.

Use lowercase severity values exactly as shown. Capitalisation matters — the
audit pipeline matches on the literals.

---

## 2. Silence equals pass

If a check, focus area, or rule does **not** fire, emit nothing for it. Do not
emit a "pass" line. Do not announce that you checked something. The presence of
a finding is the only signal; its absence is the pass.

The reasoning: findings are consumed by tooling that counts BLOCKING and
ADVISORY. Pass-noise inflates output, dilutes signal, and tempts the consumer
to skim. A clean run is a silent run — and that's the desired outcome.

---

## 3. BLOCKING is never silently downgraded

A finding category that the ruleset (or the constitution's severity floor)
declares **BLOCKING** MUST be emitted as `severity: BLOCKING`. You do not have
authority to demote it.

The only legitimate path to a non-BLOCKING outcome for a BLOCKING-default
category is a **written justification** in the candidate's `constitution.md`
§3.2 (specialist waiver) or §6.1 (severity override). When such a
justification exists, you MAY emit the finding as `severity: ADVISORY` and
cite the justification's location in the `remediation` field
(`"Downgraded per constitution.md §6.1 — see written justification"`).
Otherwise, BLOCKING stays BLOCKING.

This rule applies at every severity tier. The `strict` / `normal` / `relaxed`
tier system (see `esos-severity-tier`) demotes a defined set of
**tier-aware** checks per the matrix in `VALIDATION.md`; it does **not**
permit ad-hoc demotion.

---

## 4. Message keys — stable, hierarchical, role-prefixed

Every finding carries a `message_key` that downstream tooling can match
against without parsing the human-readable `message`. Keys are:

- **Stable** — once a key is in use, do not rename it. Add new keys; do not
  rename existing ones. Renaming breaks consumers (dashboards, deduplication,
  triage rules).
- **Hierarchical** — dot-separated, role-prefixed:
  `<role>.<category>.<stable_key>` or `<role>.<stable_key>` for simple cases.
- **Role-prefixed** — the leading segment is the specialist that emits the
  finding:
  - `analyst.*` for ANALYST findings
  - `security.*` for SECURITY findings
  - `compliance.*` for COMPLIANCE findings (categories: `privacy`,
    `financial`, `i18n`, `versioning`, `scoping`)
  - `coding.*` for CODING findings (categories: `shortcut`, `architecture`,
    `pattern`, `scale`, `resilience`, `maintainability`, `operability`,
    `migration`, `api`, `security`, `stack`, `cicd`, `docs`)
  - `testing.*` for TESTING findings (categories: `traceability`, `scenario`,
    `security`, `performance`, `compliance`, `contract`, `quality`,
    `coverage`, `data`, `environment`, `outcome`, `pyramid`, `exploratory`,
    `production`, `accessibility`, `locale`)
  - `constitution.*` for findings about the constitution structure itself
    (used by the validate workflow)
- **Concrete** — describe what failed, not what was checked. Good:
  `coding.scale.n_plus_one`. Less good: `coding.scale.check_failed`.

Each role's review skill (`esos-<role>-review`) carries the authoritative key
catalog for that role. Use those keys; only invent new keys when a genuinely
new category emerges, and document them in the review skill's catalog.

---

## 5. The summary line

After all findings, emit exactly one summary block. Two shapes, depending on
context:

**Specialist subagent / review skill** (no check IDs):

```yaml
summary: { blocking: <int>, advisory: <int> }
```

**Validate workflow** (Section A/B/C/D walk):

```yaml
summary:
  severity_tier: "<strict|normal|relaxed>"
  tier_source: "<declared|defaulted|brief>"
  total_blocking: <int>
  total_advisory: <int>
  total_informational: <int>
  publishable: <true if total_blocking == 0 else false>
  files_audited: [<list of file paths>]
  checks_executed: <int>
```

`publishable` is `true` if and only if `total_blocking == 0`. There is no
other criterion. ADVISORY findings do not block publication; `informational`
findings (Section D at `relaxed` tier) do not count toward BLOCKING or
ADVISORY totals.

---

## 6. Target-unreachable handling

If the artifact under review does not exist, cannot be read, or is empty in
a context where content was expected, emit exactly **one** finding with the
role-prefixed unreachable key, then stop:

| Role | Key |
|---|---|
| analyst | `analyst.target_unreachable` |
| security | `security.target_unreachable` |
| compliance | `compliance.target_unreachable` |
| coding | `coding.target_unreachable` |
| testing | `testing.target_unreachable` |
| validate workflow | `constitution.target_unreachable` |

Do not attempt to walk focus areas against a missing target — the resulting
findings would be noise. The unreachable finding is enough.

---

## 7. What you do NOT emit

- **Recommendations** unrelated to a specific defect. Recommendations belong
  in a role-specific deliverable (e.g. CODING's "Refactoring recommendations
  ordered by impact"), not in the findings list.
- **Pass announcements** ("✓ all checks passed"). Silence is the pass.
- **Speculation** ("this might be problematic if …"). If a defect is
  contingent, name the contingency in `message`, but only emit if the defect
  is real in the artifact as written.
- **Severity not in `{BLOCKING, ADVISORY}`**. The audit pipeline rejects
  unknown severities.
- **Findings without a `message_key`**. Every finding has a stable key.
- **Findings whose `remediation` field is empty** — "what to fix" is the
  reason the finding exists.

---

## 8. Self-check before emitting

- [ ] Every finding has all six fields (`severity`, `location`, `message`,
      `message_key`, `remediation`; plus `check_id` if this is a validate run).
- [ ] No finding has severity outside `{BLOCKING, ADVISORY}`.
- [ ] Every `message_key` follows `<role>.<category>?.<stable_key>` and is
      one of the documented keys (or a new key the corresponding review skill
      now documents).
- [ ] The summary's `blocking` / `total_blocking` count matches the number of
      BLOCKING-severity findings emitted.
- [ ] No "pass" line was emitted.
- [ ] If the target was unreachable, exactly one unreachable-keyed finding
      was emitted and the walk stopped.

If any check fails, fix before delivering.
