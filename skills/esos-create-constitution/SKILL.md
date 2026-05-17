---
name: esos-create-constitution
description: Use when the user wants to create, derive, or generate a domain-specific ESOS plugin from the generic plugin constitution. Triggers on phrases like "create a constitution for healthcare", "derive an automotive plugin", "new ESOS plugin for fintech", "generate a domain constitution", "make me a healthcare ESOS plugin". Walks the user through a derivation brief and produces a complete, fully self-contained derived plugin directory mirroring the generic base with all customization markers replaced. Base-only — derived plugins do NOT ship this skill (they don't derive further plugins).
---

# Skill: Create a domain-specific ESOS plugin

You are deriving a new domain plugin from the Generic Plugin Constitution
(v3.2.0). The Generic Plugin Constitution lives in this repository
(`generic-plugin-constitution/`) and is the floor every derivation starts
from — there is no document above it.

The long-form production rules are in `PROMPT.md` — read it before producing
output. This skill orchestrates the workflow; `PROMPT.md` defines the
contract.

**Output**: a fully self-contained derived plugin directory. The whole
directory IS an installable Claude Code plugin source — `plugin.json` at
its root, every ruleset and skill and subagent shell inlined.

---

## Step 1 — Collect the derivation brief

Ask the user for the following. **Do not invent values.** If any required
field is missing, stop and ask before continuing.

**Required**:

- `TARGET_DIRECTORY` — absolute or workspace-relative path where the derived
  plugin directory will be created (e.g. `../`, `~/work/plugins/`,
  `/Users/me/Documents/plugins/`). The skill creates
  `<TARGET_DIRECTORY>/<DOMAIN_SLUG>/` underneath it. **Stop and ask the
  user explicitly if this is not supplied** — do not assume "sibling of
  `generic-plugin-constitution/`" or any other default. Confirm the
  resolved final path (`<TARGET_DIRECTORY>/<DOMAIN_SLUG>/`) with the user
  before producing any files. If the final path already exists and is
  non-empty, stop and ask whether to abort, choose a different location,
  or overwrite — never overwrite silently.
- `DOMAIN_NAME` — human-readable name (e.g. "Healthcare Patient Records").
- `DOMAIN_SLUG` — kebab-case directory name (e.g.
  `healthcare-patient-records`). The plugin's `name` becomes
  `esos-<DOMAIN_SLUG>`.
- `DOMAIN_SUMMARY` — 2-4 sentences: what this domain is, who its users
  are, what's at stake.
- `DATA_SENSITIVITIES` — classes of sensitive data in scope (PII / PHI /
  financial / classified / safety-relevant).
- `APPLICABLE_REGULATIONS` — specific standards / regulations (cite
  articles where possible).
- `KEY_DOMAIN_TERMS` — domain terms with concise definitions. Minimum
  count depends on `SEVERITY_TIER`: ≥10 for `strict`, ≥5 for `normal`,
  ≥1 for `relaxed`.
- `RISK_PROFILE` — low / medium / high / safety-critical (drives
  confidence thresholds).

**Severity tier** (drives which checks BLOCK at validation time — see
`constitution.md` §1.4 and `VALIDATION.md` tier matrix):

- `SEVERITY_TIER` — one of `strict` / `normal` / `relaxed`. **Defaults to
  `strict`** if the user does not specify, which preserves the
  maximum-enforcement contract. Ask the user explicitly when the brief
  is silent — don't pick a looser tier on their behalf. Loosening a
  published tier later is a MAJOR semver bump per `constitution.md` §10.

  Plain-language anchors for the conversation:
  - `strict` — regulated, safety-critical, or high-blast-radius work.
    The safe default.
  - `normal` — mainstream business domain; rigour expected but
    pragmatism rewarded.
  - `relaxed` — internal tooling, prototypes, low-blast-radius
    derivations.

**Optional but recommended**:

- `JURISDICTIONS` (e.g. EU, UK, US).
- `TECH_STACK_PREFERENCES` (languages, frameworks, databases, identity
  provider).
- `ARCHITECTURE_STYLE` (monolith, microservices, event-driven, …).
- `DOMAIN_SPECIFIC_RISKS` (3–7 risks unique to this domain).
- `EXTRA_MANDATORY_SECTIONS` (e.g. `safety_case`, `clinical_workflow`).
- `MANDATORY_DOCUMENT_KINDS` — list of companion-document kinds with
  `applies_when`, `allowed_formats`, and `recognition` rules. See
  `MANUAL.md` §5.13 for the schema and recognition-rule design tips. If
  the brief omits this field, the derivation declares no companion
  documents (`mandatory_document_kinds: []` in §2.2.4).
- `SPECIALIST_WAIVERS` (rare; usually empty).
- `EXTERNAL_LINK_ALLOWLIST`.
- `DOMAIN_TAGS`.
- `DOMAIN_LOGICAL_ID_UUID` (generate one if not supplied).

**Save the brief** to `/tmp/esos-brief-<DOMAIN_SLUG>.md` so the work is
resumable if the session is interrupted, and so re-derivation on a future
base MAJOR bump can use the same inputs.

---

## Step 2 — Load the base files

Read each base file before deriving its counterpart. Do NOT skip — the
per-file production rules in `PROMPT.md` reference specific section
structures and anti-patterns in each file:

- `plugin.json` (manifest template).
- `constitution.md` (root + `esosAgentCatalog`).
- `README.md`, `CLAUDE.md`, `CHANGELOG.md` (onboarding + version-tracking
  templates).
- `VALIDATION.md` (the validate prompt, copied verbatim to derived).
- `rulesets/{analyst,security,compliance,coding,testing}.md`.
- `domain/{glossary,governance-policies,tech-stack}.md`.
- `shared/{normative-language,security-baseline,acceptance-criteria-format}.md`.
- `agents/esos-{analyst,security,compliance,coding,testing}.md` (subagent
  shells — copied verbatim aside from frontmatter description).
- `skills/esos-finding-emission/SKILL.md`,
  `skills/esos-ruleset-resolution/SKILL.md`,
  `skills/esos-severity-tier/SKILL.md` (common skills — copied verbatim).
- `skills/esos-{analyst,security,compliance,coding,testing}-review/SKILL.md`
  (review skills — copied; MAY append a domain-specific notes section).
- `skills/esos-validate-constitution/SKILL.md` (workflow skill — copied
  with two small tweaks).

Also read `PROMPT.md` for the per-file production rules and the
self-check.

---

## Step 3 — Produce the derived plugin directory

Output to the path resolved in Step 1:
**`<TARGET_DIRECTORY>/<DOMAIN_SLUG>/`**.

Before writing any file:

1. Confirm `TARGET_DIRECTORY` was supplied by the user.
2. Confirm `<TARGET_DIRECTORY>/<DOMAIN_SLUG>/` does not already exist
   with content. If it does, stop and ask.
3. State the final absolute path back to the user one more time before
   writing.

Mirror the base layout 1:1 (per `constitution.md` §9):

```
<DOMAIN_SLUG>/
├── plugin.json
├── constitution.md
├── README.md
├── CLAUDE.md
├── CHANGELOG.md
├── VALIDATION.md
├── rulesets/{analyst,security,compliance,coding,testing}.md
├── domain/{glossary,governance-policies,tech-stack}.md
├── shared/{normative-language,security-baseline,acceptance-criteria-format}.md
├── agents/esos-{analyst,security,compliance,coding,testing}.md
├── templates/                                # optional — only if any §2.2.4 kind
│   └── <kind>.<ext>                          #            declares `template_ref`
└── skills/
    ├── esos-finding-emission/SKILL.md
    ├── esos-ruleset-resolution/SKILL.md
    ├── esos-severity-tier/SKILL.md
    ├── esos-analyst-review/SKILL.md
    ├── esos-security-review/SKILL.md
    ├── esos-compliance-review/SKILL.md
    ├── esos-coding-review/SKILL.md
    ├── esos-testing-review/SKILL.md
    └── esos-validate-constitution/SKILL.md
```

`skills/esos-create-constitution/` is **intentionally absent** —
derivations don't derive further.

**Scaffold `templates/` when any kind declares `template_ref`.** For every
kind in the brief's `MANDATORY_DOCUMENT_KINDS` that declares `template_ref`,
create a placeholder file at the referenced path inside the derived plugin
tree (typically `<DOMAIN_SLUG>/templates/<kind>.<ext>`):

- `.md` / `.txt` / `.html` → one-line text file with
  `# {{ kind label }} template` and a trailing comment "(replace with the
  heading skeleton the team wants authors to start from)".
- `.docx` / `.xlsx` / `.vsdx` / `.pdf` → an empty file at the right path.
  Do NOT attempt to author Office content — the team replaces these stubs
  with real templates before publishing.

Surface every stubbed template in the derivation summary as a "template to
fill" task. If `MANDATORY_DOCUMENT_KINDS` is empty or omitted, do NOT
create `templates/`.

If any kind's `allowed_formats` lists an extension not in the default
extractor dispatch table (`rulesets/compliance.md` §6.2), emit a warning in
the summary: "declared format `<ext>` has no extractor — extend the
dispatch table or audit will fire `GEN-124`."

For each file, apply the per-file production rules in `PROMPT.md`. In
particular:

- **Replace every `{{ TOKEN }}`**. Tokens left in place are BLOCKING.
- **Rewrite every `<!-- esos:domain-customization -->` block**. Empty
  or near-empty rewrites are BLOCKING.
- **Preserve every `<!-- esos:keep -->` block** verbatim, or tighten it
  (never relax).
- **Common skills, review skill bodies, and subagent shell bodies are
  copied verbatim** from the base. Domain-specific notes go in clearly
  marked optional trailing sections — never by editing the base
  discipline.
- **`plugin.json`** in the derived plugin declares
  `inherits_from: "esos-generic-plugin-constitution@3.2.0"`,
  `kind: "derived"`, `self_containment.policy: "strict"`, and omits
  `esos-create-constitution` from `provides.skills.workflow`.
- **Cite regulations specifically** — vague "GDPR-compliant" is
  BLOCKING; cite Art. 5(1)(c) data minimisation, Art. 17 erasure, etc.
- **Surface every domain risk** in at least one of: ruleset focus
  area, glossary entry, governance rule, or security baseline check.
- **Define every domain term once** in `domain/glossary.md`; other
  files reference, not redefine.
- **Preserve the CODING ruleset role** as senior developer / software
  architect with all 14 focus areas. Removing or demoting any focus area
  is BLOCKING (GEN-103a).
- **Preserve the TESTING ruleset role** as senior tester / QA architect
  with all 14 focus areas, including the Test Anti-Patterns table
  (GEN-103b).
- **Propagate the severity tier** to: `constitution.md` §8
  (`severity_tier`), `constitution.md` §1.4 (one paragraph stating the
  chosen tier and why), `README.md` (one short paragraph), `CLAUDE.md`
  (one line under "Hard rules"), and `plugin.json` (manifest mirror if
  the manifest carries a tier field).

---

## Step 4 — Self-check before declaring done

Run the self-check from `PROMPT.md` §"Self-Check Before Output". The
key gates (full list in `PROMPT.md`):

- [ ] No `{{ TOKEN }}` remains anywhere.
- [ ] No `<!-- esos:domain-customization -->` block is unmodified or
      empty.
- [ ] Every `<!-- esos:keep -->` block is preserved or tightened.
- [ ] All five specialist classes wired in `esosAgentCatalog` with paths
      pointing to `rulesets/<role>.md`.
- [ ] `plugin.json` present at the plugin root with
      `inherits_from: "esos-generic-plugin-constitution@3.2.0"`,
      `kind: "derived"`, `self_containment.policy: "strict"`, no
      `esos-create-constitution` in `provides.skills.workflow`.
- [ ] `CHANGELOG.md` present with `[1.0.0]` entry.
- [ ] `VALIDATION.md` present at the plugin root.
- [ ] All three common skills present and byte-identical to the base.
- [ ] All five review skills present; bodies match base; any
      domain-specific notes are confined to a trailing
      `## Domain-specific notes — {{ DOMAIN_NAME }}` section.
- [ ] All five subagent shells present with domain-mentioning
      descriptions.
- [ ] `skills/esos-validate-constitution/SKILL.md` present, description
      tuned, default candidate path tuned.
- [ ] `skills/esos-create-constitution/` is **absent**.
- [ ] CODING ruleset's 14 focus areas all present.
- [ ] TESTING ruleset's 14 focus areas all present; Test Anti-Patterns
      table retained.
- [ ] No relative path in any file escapes the plugin root.
- [ ] `severity_tier` set everywhere it must be set (`constitution.md`
      §8, README, CLAUDE.md). If defaulted, the derivation summary
      records it.
- [ ] §2.2 / §2.2.1 / §2.2.2 / §2.2.3 of `constitution.md` are verbatim
      from the base; §2.2.4 is either `mandatory_document_kinds: []` or
      a fully-populated list from the brief.
- [ ] Every kind in §2.2.4 has a matching key in §8
      `mandatory_document_kinds:`.
- [ ] Every `template_ref` resolves to a stub file under `templates/`;
      stubs are listed in the summary as "team must fill".

Fix any failures before declaring done. **Do not emit a partial result.**

---

## Step 5 — Hand off to validation

After producing the directory, suggest the user run the validation skill:

> "Run `esos-validate-constitution` against `<DOMAIN_SLUG>/` to confirm
> the derivation passes all checks before publishing."

If the user agrees, invoke that skill with the derived plugin path.

---

## Step 6 — Summarize

Give the user a concise summary (under 200 words — the diff is the
artifact, not the summary):

- Plugin name (`esos-<DOMAIN_SLUG>`) and version (`1.0.0`).
- Files produced (count + key additions).
- Domain-specific additions (extra mandatory sections, raised
  thresholds, added focus areas, added glossary terms).
- Any inputs you defaulted (with rationale).
- Any open questions to resolve before publishing.

---

## Boundaries

- This skill produces files. It does NOT publish the plugin to the
  ESOS catalog (that is an administrative action via the ESOS UI / MCP).
- This skill does NOT modify the generic base. If a derivation reveals
  a gap in the base, surface it as a `BASE-FEEDBACK` finding and let
  the user decide whether to upstream a base change.
- This skill is **base-only**. Derived plugins do not ship it — they
  don't derive further plugins.
- This skill does NOT spawn ESOS specialist agents at runtime — those
  run inside the ESOS runtime, not Claude Code. The Claude Code
  subagents (`esos-analyst`, `esos-security`, `esos-compliance`,
  `esos-coding`, `esos-testing`) MAY be spawned during derivation to
  spot-check a draft section against its specialist's review skill.
