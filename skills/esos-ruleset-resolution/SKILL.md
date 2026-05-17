---
name: esos-ruleset-resolution
description: ESOS shared discipline for locating and applying the right specialist ruleset. Use whenever a subagent, review skill, or workflow needs to find and load the authoritative rules for a specialist (analyst / security / compliance / coding / testing) - in this plugin, in a derived plugin under audit, or across an inheritance chain. Covers primary-vs-fallback resolution, the tighten-not-relax rule, `@esos-include` directives, the severity-floor preservation contract, and self-containment enforcement. The contract every ESOS specialist obeys when deciding which rules to apply.
---

# Skill: ESOS ruleset resolution

You are about to read or apply an ESOS specialist's rules. This skill captures
the **shared resolution discipline** — how to find the right ruleset, how it
relates to the base, how includes resolve, and how to spot a derivation that
illegally weakens an inherited rule.

Read this skill on demand. The rules below apply identically to all five
specialists.

---

## 1. The ruleset is the specialist's source of truth

For each of the five specialists, the **authoritative ruleset file** lives at:

| Specialist | Ruleset path |
|---|---|
| ANALYST | `rulesets/analyst.md` |
| SECURITY | `rulesets/security.md` |
| COMPLIANCE | `rulesets/compliance.md` |
| CODING | `rulesets/coding.md` |
| TESTING | `rulesets/testing.md` |

The path is **relative to the plugin root**. In this base, `rulesets/` is at
the top level of `generic-plugin-constitution/`. In any derived plugin, it's
at the top level of `<domain-slug>/`. Both are valid; the meaning of
"primary ruleset" depends on context (see §3).

The ruleset is what the specialist enforces. You do not invent rules outside
it. If you find yourself reasoning "this *should* be a BLOCKING finding even
though the ruleset doesn't say so" — stop. Either the ruleset is the truth
and your reasoning is wrong, or the ruleset has a gap and the right move is
a `BASE-FEEDBACK` advisory, not a free-form finding.

---

## 2. Includes resolve relative to the ruleset file

A ruleset transitively pulls in cross-cutting content via:

```
@esos-include ../domain/glossary.md
@esos-include ../shared/normative-language.md
```

(equivalent form: `<!-- esos-include: ../path/to/file.md -->`).

Resolution rules:

- Paths are **relative to the file that contains the directive**. From
  `rulesets/analyst.md`, `../domain/glossary.md` points at `domain/glossary.md`
  in the plugin root. From `agents/esos-analyst.md` the same relative path
  also points at the same file — both files sit one level under the plugin
  root.
- An include MUST resolve inside the same plugin. Following an include path
  outside the plugin root indicates either a derivation defect or a
  misplaced ruleset. Either way, you do not follow it. The validate
  workflow's GEN-015 / GEN-018 will flag the defect.
- An unresolved include is BLOCKING at validation time. At runtime, you
  treat an unresolved include as a missing context fragment — keep walking
  the focus areas the directly-loaded ruleset names, but emit an ADVISORY
  finding `<role>.include_unresolved` so the gap is recorded.

---

## 3. Primary-vs-fallback: which ruleset wins

Three operating contexts, each with a slightly different resolution:

### A. Auditing or reviewing inside this base (`generic-plugin-constitution/`)

The primary ruleset IS this base's `rulesets/<role>.md`. There is no
fallback. The base is the foundational floor — there is no document above it.

### B. Auditing or reviewing inside a derived plugin (`<domain-slug>/`)

The primary ruleset IS the derived plugin's local `rulesets/<role>.md` (the
domain-specific version). The base is referenced indirectly via the
inheritance contract: the derived ruleset MAY tighten the base; it MUST NOT
relax it. You do not consult the base ruleset at review time — the derived
ruleset has already incorporated everything required from the base (its
authoring was constrained by `PROMPT.md` and is verified by
`VALIDATION.md`).

If the derived plugin is malformed and is missing its local ruleset, the
correct response is GEN-014 BLOCKING (required file missing), not "fall
back to the base" — derived plugins are self-contained by contract.

### C. Cross-plugin reasoning (rare)

You generally do not reason across plugins. ESOS plugins are independent
units; each carries its own complete ruleset. The only legitimate
cross-plugin reasoning is the validate workflow's GEN-101 severity-floor
check, which compares the candidate's `rulesets/<role>.md` to the
corresponding generic-plugin-constitution `rulesets/<role>.md` to confirm
that no BLOCKING-default finding has been silently downgraded.

---

## 4. The tighten-not-relax rule

A derived ruleset MAY:

- Add new focus areas, new BLOCKING items, new message keys, stricter
  thresholds.
- Promote any ADVISORY in the base to BLOCKING.
- Add domain-specific examples and concrete checks.

A derived ruleset MUST NOT:

- Remove a focus area listed in the base.
- Downgrade any BLOCKING-default finding to ADVISORY.
- Remove the `<!-- esos:keep -->` markers (or their content).
- Reframe the CODING specialist's identity (senior dev / software architect)
  or the TESTING specialist's identity (senior tester / QA architect). Both
  framings are protected by `VALIDATION.md` GEN-103a / GEN-103b.
- Drop the Test Anti-Patterns table from `rulesets/testing.md` §12.

The validate workflow's GEN-101 (severity floor preserved) catches relaxation
defects. If you are running a specialist review and notice the candidate's
ruleset has silently removed something the base had as BLOCKING, emit a
`<role>.severity_relaxed` ADVISORY (the validate workflow will upgrade it to
BLOCKING via GEN-101).

---

## 5. Self-containment is the boundary

A plugin (this base or any derivation) is self-contained
(`plugin.json.self_containment.policy: "strict"`). Concretely, when you
resolve a ruleset or an include, the resolved path is **always inside the
plugin root**. You never reach across plugin boundaries at runtime.

If you find a ruleset or skill that instructs you to load a file from
another plugin (e.g. `~/.claude/plugins/some-other-plugin/...`), that is a
self-containment violation. Do not follow it. Emit `constitution.self_containment_violation`
and stop walking the offending file.

---

## 6. Reading the ruleset efficiently

Rulesets are large (the coding ruleset is ~500 lines, the testing ruleset
~530). Two reading strategies:

- **Walking a single focus area** (most common): Read the ruleset section
  for that focus area, plus any include the section explicitly references.
  Don't load the entire ruleset just to walk one section.
- **Auditing the ruleset itself** (during validate): Read the whole file.
  GEN-101 (severity floor) and GEN-103a/GEN-103b (role framing) require it.

The review skills (`esos-<role>-review`) name the focus-area order; follow
that order so the report reads consistently across runs.

---

## 7. Self-check before applying a ruleset

- [ ] You identified which plugin's ruleset is primary (this base /
      candidate-under-audit).
- [ ] Every `@esos-include` directive in the loaded ruleset resolves inside
      the same plugin root.
- [ ] You have NOT consulted a ruleset from outside the current plugin
      (except via GEN-101's explicit base-vs-candidate comparison during
      the validate workflow).
- [ ] You are NOT inventing rules outside the ruleset; if the ruleset has
      a gap, you note it as `BASE-FEEDBACK` rather than free-forming a
      finding.

If any check fails, stop and resolve before proceeding.
