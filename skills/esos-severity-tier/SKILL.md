---
name: esos-severity-tier
description: ESOS shared discipline for resolving and applying the severity tier (strict / normal / relaxed) of a plugin under review. Use whenever a specialist subagent, review skill, or validate workflow needs to know which baseline checks BLOCK vs ADVISE for a given plugin. Covers tier resolution (declared / defaulted / brief), tier-aware demotion rules, the never-tier-aware foundational floor, mismatch handling, and the per-check matrix anchored in VALIDATION.md. The contract every ESOS specialist obeys when calibrating finding severity to the plugin's risk posture.
---

# Skill: ESOS severity tier resolution

You are about to walk checks against a plugin. The plugin's **severity tier**
(`strict`, `normal`, or `relaxed`) calibrates which baseline checks BLOCK vs
ADVISE. This skill captures the **shared resolution discipline** — how to
read the tier, what to do when it's missing or conflicts with the brief, and
which checks are never tier-aware regardless.

Read this skill on demand. It is referenced by the five subagent shells, the
five review skills, and the validate workflow.

---

## 1. The three tiers

| Tier | Intent | Typical fit |
|---|---|---|
| `strict` | Maximum enforcement — all baseline defaults BLOCKING; quality heuristics ADVISORY. | Regulated, safety-critical, or high-blast-radius domains. **Default when a brief omits the tier.** |
| `normal` | Balanced — stylistic and concreteness checks demote to ADVISORY; structural checks stay BLOCKING. | Mainstream business domains where rigour is expected but pragmatism is rewarded. |
| `relaxed` | Minimum above the foundational floor — many baseline checks ADVISORY; Section D informational. | Internal tooling, prototypes, low-blast-radius derivations. |

Tier is declared once, at the plugin level. A plugin does not mix tiers
across files. (The constitution's §1.4 says so explicitly, and the validate
workflow's GEN-014/GEN-017 enforce it.)

---

## 2. Resolving the tier

The plugin under review declares its tier in `constitution.md` §8, under the
`severity_tier:` key in `constitution_catalog_row:`. Resolve it as follows:

1. **Read `constitution.md` §8.** If `severity_tier` is present and is one of
   `strict`, `normal`, `relaxed`, use it. Record
   `tier_source: declared`.
2. **If `severity_tier` is absent or malformed**, default to `strict` (the
   safest choice for an unknown tier). Record `tier_source: defaulted`. Queue
   a `GEN-120` ADVISORY finding so the gap is visible.
3. **If a derivation brief is supplied and its `SEVERITY_TIER` disagrees with
   the candidate's `constitution.md`**, use the candidate's value (the
   constitution is authoritative; the brief is a record of intent that may
   have drifted). Record `tier_source: declared`. Queue a `GEN-121` BLOCKING
   finding for the mismatch.

State the resolved tier to the user (or to the parent workflow) before
walking any checks. The audit's `summary` MUST record both `severity_tier`
and `tier_source`.

---

## 3. The never-tier-aware floor

These checks are BLOCKING **at every tier**. The tier never relaxes them.

- All of **Section A** of `VALIDATION.md` (GEN-001..020 in v3.0.0) —
  foundational conformance.
- **GEN-101** — Severity floor preserved (no BLOCKING-default item silently
  downgraded in the derived ruleset).
- **GEN-102** — RFC 2119 normative language enforced.
- **GEN-103a** — CODING specialist role framing preserved (senior dev /
  software architect, all 14 focus areas).
- **GEN-103b** — TESTING specialist role framing preserved (senior tester /
  QA architect, all 14 focus areas, Test Anti-Patterns table retained).
- **GEN-110** — Catalog version monotonicity.
- **GEN-111** — No specialist class silently waived (ANALYST / VALIDATION
  never waivable; others require written rationale).
- **GEN-201** — Cross-file consistency on hard claims (e.g. inheritance
  declaration consistent between `constitution.md` and `plugin.json`).
- **GEN-206** — No instruction to bypass validation.

Plus a permanent rule that lives in `esos-finding-emission`: **embedded
secrets / tokens / credentials are ALWAYS BLOCKING** at every tier, in any
file, regardless of context.

You do not demote any of the above on tier grounds. Ever.

---

## 4. The tier matrix (authoritative summary)

The full per-check matrix lives in `VALIDATION.md` under "Severity Tier
(read first)" — that is the source of truth and the validate workflow walks
it directly. The summary below is the same content for quick reference; if
the two ever disagree, `VALIDATION.md` wins.

| Check ID | What it checks | `strict` | `normal` | `relaxed` |
|---|---|---|---|---|
| GEN-103 | GWT mandated for acceptance scenarios | BLOCKING | BLOCKING | ADVISORY |
| GEN-106 | Domain-term minimum in glossary | BLOCKING if <10 | ADVISORY if <5 | ADVISORY if <1 |
| GEN-107 | Tech-stack file is concrete | BLOCKING | ADVISORY | ADVISORY |
| GEN-108 | Governance policies cite specific regulations | BLOCKING | BLOCKING | ADVISORY |
| GEN-109 | Domain risks surface somewhere | BLOCKING | BLOCKING | ADVISORY |
| GEN-114 | Internationalization declared (when UI in scope) | BLOCKING | ADVISORY | ADVISORY |
| GEN-115 | Consistent severity language | BLOCKING | ADVISORY | ADVISORY |
| GEN-116 | CLAUDE.md present and domain-tuned (content checks; presence stays BLOCKING) | BLOCKING | BLOCKING | ADVISORY (presence remains BLOCKING) |
| GEN-117 | Subagent description domain mention (file presence stays BLOCKING under §9) | BLOCKING | BLOCKING | ADVISORY (presence remains BLOCKING) |
| GEN-118 | Validate skill description tuned (file presence stays BLOCKING under §9) | BLOCKING | BLOCKING | ADVISORY (presence remains BLOCKING) |
| GEN-205 | Tech-stack alignment across rulesets | BLOCKING | BLOCKING | ADVISORY |
| GEN-301..306 | Section D heuristics | ADVISORY | ADVISORY | **informational** (not counted in BLOCKING/ADVISORY totals) |

For any check **not** in this matrix, the severity is whatever the check's
body in `VALIDATION.md` declares. (Most checks are not tier-aware.)

---

## 5. Section D at `relaxed`

At `relaxed`, Section D heuristic findings (GEN-301..306) are
**informational only**. They are emitted, but they go into the
`total_informational` counter, not `total_advisory`. They do not block, they
do not advise — they are recorded for the maintainer's optional attention.

This bucket exists only at `relaxed`. At `strict` and `normal`, Section D
findings count as ADVISORY.

---

## 6. Tier MAY be tightened mid-life; loosening is a MAJOR

A plugin maintainer MAY tighten the tier (`normal` → `strict`, `relaxed`
→ `normal`) at any time — this is a MINOR semver bump because it only
strengthens the published contract.

Loosening the tier (`strict` → `normal`, `normal` → `relaxed`) is a MAJOR
semver bump per `constitution.md` §10. It weakens the contract that
downstream workspaces depend on.

The tier in the candidate's current `constitution.md` is the tier that
governs the current audit, regardless of historical values.

---

## 7. Self-check before walking tier-aware checks

- [ ] You read `constitution.md` §8 (or `plugin.json` `constitution.severity_tier`
      if mirrored there) before walking any tier-aware check.
- [ ] You resolved the tier exactly once, at the start of the run, and
      reuse the resolved value throughout.
- [ ] You recorded `tier_source` (`declared` / `defaulted` / `brief`) in the
      summary.
- [ ] You did NOT demote any never-tier-aware check (Section A, GEN-101,
      GEN-102, GEN-103a, GEN-103b, GEN-110, GEN-111, GEN-201, GEN-206)
      regardless of resolved tier.
- [ ] At `relaxed`, Section D findings go into `total_informational`, not
      `total_advisory`.
- [ ] You queued GEN-120 if `tier_source: defaulted`, or GEN-121 if a
      brief/candidate mismatch was detected.

If any check fails, fix before continuing.
