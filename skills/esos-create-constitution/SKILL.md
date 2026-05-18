---
name: esos-create-constitution
description: Use when the user wants to create, derive, or generate a domain-specific ESOS plugin from the generic plugin constitution. Triggers on phrases like "create a constitution for healthcare", "derive an automotive plugin", "new ESOS plugin for fintech", "generate a domain constitution", "make me a healthcare ESOS plugin". Walks the user through a derivation brief and produces a complete, fully self-contained derived plugin directory mirroring the generic base with all customization markers replaced. Base-only — derived plugins do NOT ship this skill (they don't derive further plugins).
---

# Skill: Create a domain-specific ESOS plugin

You are deriving a new domain plugin from the Generic Plugin Constitution
(v4.0.0). The Generic Plugin Constitution lives in this repository
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
- `TEMPLATES_DIR` — path to a directory of template documents that drives
  companion-document discovery. Default `./templates/` relative to the brief.
  Each non-sidecar file in this directory becomes one §2.2.4 entry; each MUST
  be accompanied by a `<kind>.esos.yaml` sidecar declaring `applies_when`,
  `severity_on_missing`, `severity_on_mismatch` (and optionally overriding any
  discovered field). See `MANUAL.md` §5.13 for the full schema and
  `PROMPT.md` "§2.2.4 Discovery Algorithm" for the inference rules. If the
  directory is omitted, missing, or empty, the derivation declares no
  companion documents (`mandatory_document_kinds: []` in §2.2.4).
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

## Step 2.5 — Discover companion-document kinds from `TEMPLATES_DIR`

If the brief supplied (or defaulted) `TEMPLATES_DIR`, walk that directory
to build the §2.2.4 `mandatory_document_kinds` list. This step replaces
the v3.2.0 inline `MANDATORY_DOCUMENT_KINDS` brief field.

1. **Resolve `TEMPLATES_DIR`**.
   - If unset, default to `./templates/` relative to the brief location.
   - If the resolved path does not exist or contains no non-sidecar files,
     record "no companion documents" and skip this step — §2.2.4 will be
     `mandatory_document_kinds: []`.

2. **Enumerate files**. List every regular file in `TEMPLATES_DIR` whose
   extension is NOT `.yaml` (sidecars are not templates).

3. **For each template file**, follow the full **§2.2.4 Discovery
   Algorithm** in `PROMPT.md`:
   1. Derive `kind` from the filename stem (snake_case). Sidecar `kind:`
      overrides.
   2. Group sibling files with the same stem → multi-format
      `allowed_formats`. Sidecar `allowed_formats:` overrides.
   3. Extract the Normalized Document Model using the extractor for the
      file's format (`rulesets/compliance.md` §6.2).
   4. Derive `label` from the document title / H1. Sidecar `label:`
      overrides. Fallback: Title Case of the kind.
   5. Build `recognition` from the model: prefer `headings_all_of` (top 3
      headings), then `sections_any_of` (top 5 section names for `.xlsx`
      / `.vsdx`), then `tabular_headers_all_of` (first table's headers),
      and always also add a `keywords_min_hits` fallback with the 4–8
      most characteristic terms and `count = floor(len(of) * 0.6)`. Wrap
      multiple rule paths in `any_of:`. Sidecar `recognition:` overrides
      the entire discovered block (no merge).
   6. **If the model is empty** (no headings, no sheets, no tables, no
      characteristic words — common for scanned PDFs, password-protected
      files, or image-only Visio), stop and ask the user to supply
      `recognition` in the sidecar. **Do not fabricate a recognition
      block.**

4. **Read the sidecar `<TEMPLATES_DIR>/<kind>.esos.yaml`**.
   - If present, parse YAML and require `applies_when`,
     `severity_on_missing`, and `severity_on_mismatch`. Optional override
     keys: `kind`, `label`, `allowed_formats`, `recognition`.
   - If **absent**, stop and ask the user **two friendly questions per
     template** (do not surface the raw severity field names — that's
     YAML detail):
     1. *"Under what condition does `<label>` apply to a specification?
        (Predicate the COMPLIANCE agent evaluates from the spec body —
        e.g. 'spec touches recall_traceability or safety-critical
        parts'.)"* — this becomes `applies_when`.
     2. *"Is `<label>` **mandatory** or **optional** for applicable
        specifications?"* — the answer maps to both severity fields:
        - **mandatory** → `severity_on_missing: BLOCKING`,
          `severity_on_mismatch: BLOCKING`.
        - **optional** → `severity_on_missing: ADVISORY`,
          `severity_on_mismatch: ADVISORY`.
     Then **offer to write a sidecar back to
     `<TEMPLATES_DIR>/<kind>.esos.yaml`** containing the values you
     resolved so re-derivation is reproducible. Wait for the user to
     confirm before writing. If a team needs the asymmetric case
     (e.g. *missing* is ADVISORY but *failed recognition* is BLOCKING),
     mention that they can edit the sidecar after the run to pull the
     two severities apart — the binary prompt is the friendly default,
     not a constraint.
   - If **present but missing a required field**, stop and ask for the
     specific field — use the same friendly mandatory/optional prompt
     when the missing field is one or both severities. Offer to update
     the sidecar.

5. **Compose the in-memory §2.2.4 list**. One entry per discovered kind
   (de-duplicated by `kind`), combining discovered properties and
   sidecar policy.

6. **Flag orphans and gaps**:
   - Sidecar present without a matching template file → log a warning,
     skip the sidecar (do not invent a kind).
   - Template's discovered `allowed_formats` includes an extension
     outside the default extractor dispatch table (`rulesets/compliance.md`
     §6.2) → log a warning that `GEN-124` will fire unless the
     derivation extends the dispatch table.

7. **Record the discovery summary** for Step 6. For every kind, note:
   `kind`, sidecar status (`loaded` / `written-back` / `interactively-supplied`),
   which fields were discovered vs. overridden, and the resolved
   `template_ref` path the published derivation will use.

This in-memory list is what populates `constitution.md` §2.2.4 and §8
`mandatory_document_kinds:` in Step 3. The actual template files
(and any same-stem siblings) are copied into the derived plugin's
`templates/` directory in Step 3. **Sidecars are never copied into the
derived plugin** — they are derivation inputs only.

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
├── templates/                                # populated from the source
│   └── <kind>.<ext>                          #            TEMPLATES_DIR (Step 2.5);
│                                             #            absent if no kinds were
│                                             #            discovered. Sidecars are
│                                             #            NOT copied here.
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

**Populate `templates/` from the discovered kinds.** For every kind in the
in-memory list produced by Step 2.5, copy the template file (and any
same-stem siblings) from the source `TEMPLATES_DIR` into the derived plugin
at `<DOMAIN_SLUG>/templates/<basename>`. The files are copied verbatim —
no stubbing, no auto-authoring. **Do NOT copy any `<kind>.esos.yaml`
sidecar** — sidecars are derivation inputs and stay with the brief.

If the in-memory list is empty (no `TEMPLATES_DIR`, or the directory was
absent / empty), do NOT create `templates/` in the derived plugin and write
`mandatory_document_kinds: []` into §2.2.4 and §8.

If any discovered (or sidecar-overridden) `allowed_formats` includes an
extension not in the default extractor dispatch table (`rulesets/compliance.md`
§6.2), emit a warning in the derivation summary: "declared format `<ext>`
has no extractor — extend the dispatch table or audit will fire `GEN-124`."

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
  `inherits_from: "esos-generic-plugin-constitution@4.0.0"`,
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
      `inherits_from: "esos-generic-plugin-constitution@4.0.0"`,
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
- [ ] Every `template_ref` in §2.2.4 resolves to a real file under the
      derived plugin's `templates/`, copied verbatim from the source
      `TEMPLATES_DIR`. No `<kind>.esos.yaml` sidecar appears under
      `templates/` (sidecars are inputs, not published artefacts).
- [ ] Step 2.5's discovery summary is included in the Step 6 derivation
      summary, listing each kind, sidecar status (loaded /
      written-back / interactively-supplied), and any
      discovered-vs-overridden fields.

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
