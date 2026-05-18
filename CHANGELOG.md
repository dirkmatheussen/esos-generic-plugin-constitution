# Changelog — Generic Plugin Constitution

All notable changes to the **Generic Plugin Constitution** are recorded here. The base
follows semantic versioning per `constitution.md` §10.

The format is loosely based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).
Each entry records a version, ratification date, and the contract changes from the
previous version. Each derived domain plugin pins to a specific base version via
`plugin.json.inherits_from`; re-derivation is always a deliberate, conscious act.

---

## [4.0.0] — 2026-05-18

**Type**: MAJOR — replaces the inline `MANDATORY_DOCUMENT_KINDS` brief field
with a templates-directory discovery model. The published §2.2.4 contract is
unchanged in shape (the COMPLIANCE specialist still reads the same YAML at
runtime), but the brief schema and the create skill's workflow have changed
incompatibly. Existing derivations inheriting from v3.2.0 remain operational
against their pinned base; re-derivation against v4.0.0 requires migrating
the team's `MANDATORY_DOCUMENT_KINDS` brief block into a `TEMPLATES_DIR`
directory of real template files plus per-template `<kind>.esos.yaml`
sidecars.

### Removed

- **Brief field `MANDATORY_DOCUMENT_KINDS`** — gone. Teams no longer
  hand-author every kind's `kind`, `label`, `applies_when`,
  `allowed_formats`, `recognition`, `severity_on_missing`, and
  `severity_on_mismatch` inside the derivation brief.

### Added

- **Brief field `TEMPLATES_DIR`** (optional, default `./templates/`
  relative to the brief location) — points the create skill at a
  directory of actual template documents. Each non-sidecar file in the
  directory becomes one §2.2.4 entry.
- **Per-template sidecar `<kind>.esos.yaml`** — co-located with each
  template file in `TEMPLATES_DIR`, carries the **policy** fields the
  skill cannot infer from the file: `applies_when`,
  `severity_on_missing`, `severity_on_mismatch`. MAY also override the
  discovered `kind`, `label`, `allowed_formats`, or `recognition`.
  Sidecars are derivation inputs only — they are NOT copied into the
  published plugin tree.
- **Discovery algorithm** — `PROMPT.md` "§2.2.4 Discovery Algorithm"
  formalises how the skill derives `kind`, `label`, `template_ref`,
  `allowed_formats`, and `recognition` from each template file. The
  COMPLIANCE specialist's `rulesets/compliance.md` §6.2 extractor
  dispatch table is reused to produce the Normalized Document Model
  from which `recognition` rules are built.
- **Mandatory/optional friendly prompt** — when a sidecar is absent or
  incomplete, the create skill asks two friendly questions per template
  ("under what condition does it apply?" and "is it mandatory or
  optional?") and maps the answers to the YAML fields. The binary
  mandatory/optional answer maps to BOTH `severity_on_missing` and
  `severity_on_mismatch` symmetrically (mandatory → BLOCKING/BLOCKING;
  optional → ADVISORY/ADVISORY). Teams that need asymmetric severities
  edit the sidecar directly after the run.
- **Sidecar write-back** — when a sidecar is absent, the skill offers
  to write the resolved policy fields back to
  `<TEMPLATES_DIR>/<kind>.esos.yaml` so the next re-derivation is
  reproducible.
- **`MANUAL.md` §5.13 rewrite** — full coverage of the templates-directory
  model, sidecar schema, discovery rules, mandatory/optional UX, and
  what gets published vs. what stays input.

### Changed

- **`constitution.md` §2.2.4** — customization block content rewritten
  from a parameterised YAML template (with `{{ KIND_KEY }}` /
  `{{ HEADING_A }}` tokens) to an explainer of the discovery-from-templates
  model plus a concrete populated-YAML example. The published derivation
  still emits the same shape of `mandatory_document_kinds:` YAML the
  COMPLIANCE specialist reads at runtime.
- **`constitution.md` §9** — `templates/` is now described as the input
  surface that drives §2.2.4 generation. The layout note now spells out
  that source `TEMPLATES_DIR` carries one template file per kind plus a
  sibling `<kind>.esos.yaml` sidecar.
- **`constitution.md` §11** — comparison rows for "Companion document
  kinds" and "Companion document templates" updated to reflect the new
  derivation flow.
- **`PROMPT.md`** — brief-table row replaced (`MANDATORY_DOCUMENT_KINDS`
  → `TEMPLATES_DIR`); §2.2.4 production rules rewritten to call the
  Discovery Algorithm; templates are now copied verbatim from the
  source rather than stubbed.
- **`skills/esos-create-constitution/SKILL.md`** — new Step 2.5
  ("Discover companion-document kinds from `TEMPLATES_DIR`") covers
  file enumeration, per-file discovery, sidecar reading, the friendly
  mandatory/optional prompt, sidecar write-back, orphan detection, and
  extractor-coverage warnings. Step 3 now copies real template files
  (not stubs); Step 6 records the discovery summary.
- **`plugin.json`** — `version` 3.2.0 → 4.0.0;
  `constitution.version` likewise;
  `derivation.derived_plugin_inherits_from` bumped to
  `esos-generic-plugin-constitution@4.0.0`. Description updated.
- **`.claude-plugin/marketplace.json`** — `metadata.version` and plugin
  entry's `version` bumped to `4.0.0`.
- **`skills/esos-validate-constitution/SKILL.md`** — description and
  body text updated to v4.0.0; audit logic itself is unchanged because
  the published §2.2.4 shape is unchanged.

### Compatibility

- **No content of existing `<!-- esos:keep -->` blocks was modified.**
- **No check ID changed semantics.** GEN-022, GEN-023, GEN-122,
  GEN-123, GEN-124 audit the published derivation exactly as in v3.2.0.
  A v3.2.0 derivation re-audited under v4.0.0's validator returns
  identical results.
- **Derivations inheriting from v3.2.0 remain valid against their
  pinned base.** Their `plugin.json.inherits_from` keeps pointing at
  `esos-generic-plugin-constitution@3.2.0` and they audit under the
  v3.2.0 contract.
- **Re-deriving against v4.0.0 is a deliberate migration**: the team
  moves their old `MANDATORY_DOCUMENT_KINDS` brief block to
  `TEMPLATES_DIR/<kind>.esos.yaml` sidecars and supplies real template
  files. The create skill's interactive prompts smooth the migration.
- The 8 foundational mandatory **sections** are unchanged.

---

## [3.2.0] — 2026-05-17

**Type**: MINOR — adds a new mandatory-axis (companion documents) without
removing or incompatibly redefining anything. Existing derivations remain valid;
re-derivation is required only if a derivation wants to opt into companion-document
checks.

### Added

- **`constitution.md` §2.2 — Mandatory Companion Documents.** New
  foundational-floor section defining companion document **kinds** (Threat
  Model, Risk Register, DPIA, etc.) identified by **kind**, not filename.
  Includes §2.2.1 Normalized Document Model, §2.2.2 Recognition Rules
  catalog, §2.2.3 Per-Kind Schema, and §2.2.4 the derivation-customization
  block. The base declares **no** required kinds; derivations declare zero
  or more. The existing "Section Discipline" subsection was renumbered
  §2.2 → §2.3 (no content change).
- **`constitution.md` §6.1 — three new severity-mapping rows** for
  companion documents (missing/unreachable, recognition mismatch, no
  extractor available).
- **`constitution.md` §8 — `mandatory_document_kinds:` catalog key**
  mirroring §2.2.4.
- **`constitution.md` §9 — optional `templates/` directory** in the
  layout, required iff any §2.2.4 kind declares `template_ref`.
- **`constitution.md` §11 — two new comparison rows** (companion document
  kinds, companion document templates) contrasting base vs derivation.
- **`VALIDATION.md` — five new checks**:
  - `GEN-022` (Section A, BLOCKING) — companion-document kind
    declarations complete; §2.2.4 ↔ §8 key-set match.
  - `GEN-023` (Section A, BLOCKING) — every `template_ref` resolves;
    `templates/` directory present iff any kind declares `template_ref`.
  - `GEN-122` (Section B, tier-aware: BLOCKING / BLOCKING / ADVISORY) —
    companion document instance present and reachable.
  - `GEN-123` (Section B, tier-aware: BLOCKING / BLOCKING / ADVISORY) —
    instance passes the kind's recognition signature.
  - `GEN-124` (Section B, tier-aware: BLOCKING / ADVISORY / ADVISORY) —
    extractor available for every declared `allowed_formats` extension.
- **`rulesets/compliance.md` §6 — Mandatory Companion Documents.**
  Discipline (presence + type-identity, never content quality) and a
  default extractor dispatch table covering `.docx`, `.xlsx`, `.pptx`
  (and legacy `.ppt` via conversion to `.pptx`), `.pdf`, `.vsdx`,
  `.html`, `.md`, `.txt`. Four new message keys
  (`compliance.companion_doc.*`). Existing "Domain-specific Compliance"
  subsection renumbered §6 → §7.
- **`agents/esos-compliance.md`** — companion-document presence and
  type-identity added to the specialist's specialty list (pointer only —
  the agent remains slim).
- **`MANUAL.md` §4.2 / §5.13** — new optional brief field
  `MANDATORY_DOCUMENT_KINDS` with worked-example block (recall impact
  assessment, supplier quality certificate, pricing change authorization)
  and per-field rationale on writing robust recognition rules.
- **`PROMPT.md`** — production rules for §2.2.4 population, `templates/`
  scaffolding, `rulesets/compliance.md` §7 preservation, and the
  extractor-coverage advisory. Self-check items added. Cross-reference to
  the renumbered §2.3 updated.
- **`skills/esos-create-constitution/SKILL.md`** — collects
  `MANDATORY_DOCUMENT_KINDS`, scaffolds `templates/` with stubs, warns on
  extractor coverage gaps, adds layout entry and self-check items.
- **`skills/esos-validate-constitution/SKILL.md`** — loads candidate
  specs' `companion_documents:` blocks and the candidate's `templates/`
  directory; walks the new checks; updates Section coverage banners
  (A: GEN-001..023, B: GEN-101..124).

### Changed

- **`plugin.json`** — `version` bumped 3.1.0 → 3.2.0;
  `constitution.version` likewise;
  `derivation.derived_plugin_inherits_from` bumped to
  `esos-generic-plugin-constitution@3.2.0`.
- **`.claude-plugin/marketplace.json`** — `metadata.version` and plugin
  entry's `version` bumped to `3.2.0`.
- **Section numbering in `constitution.md` §2** — existing "Section
  Discipline" promoted from §2.2 to §2.3 to make room for §2.2 Mandatory
  Companion Documents. Cross-references in `PROMPT.md` updated
  accordingly. No content change to the discipline subsection.

### Compatibility

- No content of existing `<!-- esos:keep -->` blocks was modified.
- No existing check ID changed semantics.
- Derived plugins inheriting from v3.1.0 remain valid; their auditor
  returns identical results because §2.2.4 is empty by default. Opting
  into companion-document checks requires a derivation amendment
  (declare kinds + optionally ship `templates/`).
- The 8 foundational mandatory **sections** are unchanged. §2.2
  introduces a *new axis* (mandatory documents), not a 9th mandatory
  section.

---

## [3.1.0] — 2026-05-17

**Type**: MINOR — additive packaging release. No constitution-content contract
changed. Derivations from v3.0.0 SHOULD re-derive (or hand-add the new
`.claude-plugin/marketplace.json`) so they become installable through
`/plugin marketplace add` + `/plugin install`. Existing v3.0.0 derivations
remain valid governance documents; only their *installability* benefits from
re-derivation.

### Added

- **`.claude-plugin/marketplace.json`** at the root of the generic base. Wraps
  the repository as a single-plugin Claude Code marketplace so it can be
  installed with `/plugin marketplace add` + `/plugin install`. Before 3.1.0,
  the only install paths were `claude --plugin-dir <path>` (local-test) or a
  manual `enabledPlugins` entry in `settings.json` — both viable but
  undocumented and less discoverable.
- **PROMPT.md** — new per-file production rule for
  `.claude-plugin/marketplace.json` in derivations, and a new self-check item
  ensuring the file is present, well-formed, and version-aligned with
  `plugin.json`.
- **MANUAL.md** — two new install sections:
  - §2.4 "Installing the generic base as a Claude Code plugin" — Git URL,
    local path, and `settings.json` install paths for the base.
  - §9.4 "Installing the derived plugin in Claude Code" — equivalent flow for
    the derived plugin a team has just published.
- **README.md** — top-level "Install" section showing the three install paths
  for the base, with a pointer to MANUAL.md for detail.
- **plugin.json** — `.claude-plugin/marketplace.json` added to
  `derived_plugin_required`. Derived plugins MUST now ship a marketplace
  wrapper.

### Changed

- **plugin.json** — `version` bumped 3.0.0 → 3.1.0;
  `constitution.version` likewise; `derivation.derived_plugin_inherits_from`
  bumped to `esos-generic-plugin-constitution@3.1.0`.
- **PROMPT.md** — base-version references bumped to 3.1.0; derived plugin
  inherits-from string updated.
- **VALIDATION.md GEN-014** — required-files list extended with
  `.claude-plugin/marketplace.json`. Missing this file in a v3.1.0-derived
  plugin is BLOCKING at every tier.

### Migration from v3.0.0

A domain plugin derived from v3.0.0 continues to validate against v3.0.0
indefinitely. To pick up installability:

1. Add `.claude-plugin/marketplace.json` to the derived plugin (see PROMPT.md
   per-file production rule for the template).
2. Bump the derived plugin's `inherits_from` in `plugin.json` to
   `esos-generic-plugin-constitution@3.1.0`.
3. Bump the derived plugin's own `version` per its `constitution.md` §10
   semver (PATCH is appropriate for this packaging-only change).
4. Re-run `esos-validate-constitution` to confirm GEN-014 now passes.

Re-derivation (`esos-create-constitution`) is the cleaner path if the
derivation brief is still available — it produces the marketplace wrapper
automatically.

---

## [3.0.0] — 2026-05-17

**Type**: MAJOR — first plugin-shaped release. Re-derivation required for any domain
plugin previously derived from Generic Constitution v2.x.

### Added

- `plugin.json` manifest at the root of every plugin (base and derived). Carries
  `spec_version`, `inherits_from`, the list of provided agents/skills, the
  self-containment policy, and the forward-compatibility contract.
- `CHANGELOG.md` — this file. Every base version is now logged with explicit
  contract deltas so derived plugins know exactly what changed when they
  re-derive.
- Three **common skills** that capture cross-cutting discipline once, instead of
  duplicating it across the five subagents:
  - `esos-finding-emission` — YAML finding format, silence-equals-pass,
    message-key naming, BLOCKING-never-downgraded discipline, target-unreachable
    handling, summary line.
  - `esos-ruleset-resolution` — base-vs-derived precedence, tighten-not-relax,
    `@esos-include` resolution, severity-floor preservation across base and
    derivation.
  - `esos-severity-tier` — strict / normal / relaxed resolution, tier-aware
    demotion rules, the never-tier-aware floor.
- Five **per-role review skills**, one per specialist, holding that role's
  focus-area walk, message-key catalog, and role-specific deliverables:
  `esos-analyst-review`, `esos-security-review`, `esos-compliance-review`,
  `esos-coding-review`, `esos-testing-review`.

### Changed

- **`agents/` directory renamed to `rulesets/`** at the constitution-content
  level. The old name collided with Claude Code's `agents/` convention for
  subagent shells. Subagent shells now live exclusively under `agents/`; the
  constitution's authoritative specialist rulesets now live exclusively under
  `rulesets/`. Cross-document references in `constitution.md`, `PROMPT.md`,
  `VALIDATION.md`, `MANUAL.md`, and the five rulesets themselves are updated
  accordingly.
- The five Claude Code subagent shells under `agents/esos-*.md` are slimmed
  down to role identity + pipeline position + pointers to the common and
  review skills. The boilerplate they used to carry (YAML finding format,
  ruleset-resolution discipline, focus-area walks, message-key catalogs,
  boundaries) now lives in the skills, written once.
- `esos-validate-constitution` composes the common and review skills instead
  of restating them. The check list in `VALIDATION.md` is unchanged in
  numbering but updated in file-path references (`rulesets/` not `agents/`)
  and gains new checks for `plugin.json` presence, self-containment, and the
  ten skill files.
- `esos-create-constitution` produces a structurally identical, fully
  self-contained domain plugin: it inlines all rulesets, all common skills,
  all review skills, all subagent shells, all shared and domain docs, and the
  validate workflow skill into the derived plugin directory. The derived
  plugin omits only `esos-create-constitution` itself (derivations don't
  derive further).
- `constitution.md` §3 catalog `promptFile` paths updated from
  `agents/<role>.md` to `rulesets/<role>.md`.
- `constitution.md` §9 (Repository Layout) updated to the plugin-shaped layout
  and lists `plugin.json` as required.
- `constitution.md` §10 (Governance) gains a clause: re-derivation is required
  when a base version's `MAJOR` bump changes the contract. Domain plugins MAY
  pin to an older `MAJOR` indefinitely; the ESOS catalog tracks which base
  version each domain plugin inherits from.

### Removed

- The `.claude/` wrapper directory used in v2.x. The whole plugin directory
  is now the plugin; there is no nested wrapper. Subagents under `agents/`
  and skills under `skills/` are first-class siblings of `constitution.md`,
  `rulesets/`, `domain/`, and `shared/`.

### Migration from v2.x

A domain plugin derived from Generic Constitution v2.x cannot be automatically
migrated. The intended path is to re-run `esos-create-constitution` against
the same derivation brief used originally:

1. Read the original brief (saved by the v2.x create skill at
   `/tmp/esos-brief-<DOMAIN_SLUG>.md`, if still present, or reconstructed by
   the maintainer).
2. Run `esos-create-constitution` from this base (v3.0.0).
3. Validate the new derived plugin with `esos-validate-constitution`.
4. Update the ESOS catalog row to point at the new plugin and version.
5. Retire the v2.x plugin once dependents have re-attached.

Re-derivation is intentionally not automatic: a MAJOR base bump changes the
contract, and each domain team should consciously confirm the new contract
fits their domain.

---

## [2.0.0] — 2026-05-03 (legacy)

The pre-plugin base. Lived in `generic-constitution/`. Five specialist agent
classes, the pipeline ordering, the eight mandatory specification sections, the
audit trail rules, the severity tier system (strict / normal / relaxed) — all
established here and carried forward verbatim into v3.0.0.

See `generic-constitution/` for the v2.0.0 source. Any domain plugin
`inherits_from: "Generic Constitution v2.0.0"` continues to validate against
that earlier base until it is re-derived.

---

**Maintained by**: ESOS Constitution Administrator.
**Reporting issues**: surface BASE-FEEDBACK findings during derivation or
validation; they will be triaged into the next MINOR/PATCH release.
