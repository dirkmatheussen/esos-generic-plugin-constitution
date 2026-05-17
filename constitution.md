# Generic ESOS Project Constitution (Plugin-Shaped)

> **Type**: Foundational constitution — this is the floor every derived plugin starts
> from. It does not inherit from any other document.
> **Status**: Reference / template — this constitution is not itself attached to a workspace.
> **Form**: Packaged as a Claude Code plugin source. See [`plugin.json`](plugin.json).
>
> ---
>
> This document is intended for three uses:
>
> 1. **Generation base** — paired with [`PROMPT.md`](PROMPT.md), it is the source from
>    which a domain-specific plugin (e.g. automotive, healthcare, fintech, public
>    sector, retail) is derived.
> 2. **Validation reference** — paired with [`VALIDATION.md`](VALIDATION.md), it is the
>    baseline against which a derived plugin is audited for conformance to this
>    constitution and to the sensible defaults captured here.
> 3. **Plugin payload** — together with [`plugin.json`](plugin.json), the surrounding
>    directory is itself an installable Claude Code plugin source. Derived plugins
>    inherit this structure verbatim and are fully self-contained.
>
> **Customization markers**:
>
> - `{{ TOKEN }}` — inline substitution; the derived plugin MUST replace every
>   occurrence with a concrete value.
> - `<!-- esos:domain-customization -->` — the entire following block is expected to be
>   rewritten or extended by the derived plugin.
> - `<!-- esos:keep -->` — block is part of the inherited floor; derived plugins
>   MUST retain it (verbatim or stricter).

---

## 1. Context and Foundational Stance

<!-- esos:keep -->

This constitution is **foundational** — it is the floor every derived plugin
starts from. It establishes the **ground rules** for the ESOS pipeline:

- The five specialist agent classes and their pipeline ordering (§3 below).
- The minimum mandatory specification section set (§2 below).
- The traceability and audit requirements (§7 below).
- The customization boundaries — what derivations MAY and MUST NOT do (this section
  and the inheritance contract at the bottom of each `<!-- esos:keep -->` block).

A derived plugin **MAY** tighten any rule defined here. It **MUST NOT** weaken
any rule this constitution flags as a `MUST` or `MUST NOT`, and it **MUST NOT**
remove or relax any `<!-- esos:keep -->` block.

<!-- /esos:keep -->

### 1.1 Domain Description

<!-- esos:domain-customization -->

`{{ DOMAIN_NAME }}` is `{{ DOMAIN_SUMMARY_ONE_PARAGRAPH }}`.

Specifications under this constitution typically describe `{{ TYPICAL_SCOPE_OF_CHANGE }}`
and are reviewed by `{{ TYPICAL_STAKEHOLDER_ROLES }}`.

<!-- /esos:domain-customization -->

### 1.2 Ways of Working

- Specifications are produced before implementation; ESOS specialist agents validate
  each specification against this constitution before downstream artefacts are generated.
- Cross-functional review (product, security, compliance, engineering, operations) is
  the norm; specialist agents surface findings as input to that review, not as a
  replacement for it.
- Findings are addressed by editing the specification and re-running validation, not by
  overriding agents.

### 1.3 Governance

- The constitution is the authoritative description of how this domain validates
  specifications. Local team practices that contradict it must be reconciled in writing.
- Amendments follow the process in §10.
- Deviations from a published constitution version on a per-specification basis MUST be
  declared in the specification's `assumptions` section with explicit rationale.

### 1.4 Severity Tier

<!-- esos:keep -->

Every constitution derived from this base declares a **severity tier** that calibrates
how strictly the generic baseline checks (`VALIDATION.md` Sections B–D) gate publication
and how strictly mandatory-section presence is enforced. The tier does **not** relax any
foundational rule — Section A of `VALIDATION.md` (foundational conformance) remains
BLOCKING at every tier.

Three tiers are defined:

| Tier      | Intent                                                                                  | Audience                                                                                  |
| --------- | --------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| `strict`  | Maximum enforcement. All baseline defaults are BLOCKING; quality heuristics are ADVISORY. | Regulated, safety-critical, or high-blast-radius domains; the conservative default.        |
| `normal`  | Balanced enforcement. Stylistic and concreteness checks demote to ADVISORY; structural and substantive checks remain BLOCKING. | Mainstream business domains where rigour is expected but pragmatism is rewarded.           |
| `relaxed` | Minimum enforcement above the foundational floor. Many baseline checks are ADVISORY; Section D heuristics are informational. | Internal tooling, prototypes, and low-blast-radius derivations where speed matters.        |

A derived plugin declares its tier in §8 (`severity_tier`). Tier-aware behavior is
defined in two places:

- `VALIDATION.md` — per-check tier matrix (which checks BLOCK at which tier).
- §6.1 below — default severity mapping for finding categories, qualified by tier.

The default tier when a derivation brief omits `SEVERITY_TIER` is **`strict`**. A
derivation MAY at any time tighten its tier (e.g. `normal` → `strict`) without a major
semver bump; loosening its tier (e.g. `strict` → `normal`) is a MAJOR change per §10
because it weakens the published contract.

A derivation MUST NOT mix tiers across files; the tier declared in §8 governs the entire
derived plugin.

<!-- /esos:keep -->

---

## 2. Mandatory Specification Sections

<!-- esos:keep -->

Every specification under a constitution derived from this base **MUST** contain the
following sections. These form the **foundational floor**: derived plugins MAY add
sections, but MUST NOT remove any of these.

| Section Key                | Label                       | Responsible Agent     | Description                                                                                |
| -------------------------- | --------------------------- | --------------------- | ------------------------------------------------------------------------------------------ |
| `use_case_traceability`    | Requirements Traceability   | ANALYST               | References to parent artifact(s) — Use Case (UC-xxx), Epic (EP-xxx), Feature (FE-xxx), or User Story (US-xxx) — with coverage and deltas. (Section key retained under its legacy name for tooling stability; it accepts traceability to any of the four agile-friendly artifact types.) |
| `user_scenarios`           | User Scenarios              | ANALYST               | Prioritized user stories (P1/P2/P3) with Given/When/Then acceptance scenarios; happy-path AND error-path. Features/epics may reference child stories rather than restate scenarios. |
| `functional_requirements`  | Functional Requirements     | ANALYST               | Numbered FR-xxx, RFC 2119 normative language, independently testable                       |
| `key_entities`             | Key Entities                | ANALYST               | Entities and relationships when data is involved                                           |
| `success_criteria`         | Success Criteria            | ANALYST + TESTING     | Numbered SC-xxx, measurable, technology-agnostic                                           |
| `assumptions`              | Assumptions                 | ANALYST               | Explicit assumptions and scope decisions                                                   |
| `security_compliance`      | Security & Compliance       | SECURITY              | Data classification, authN/Z, encryption, audit (when in scope)                            |
| `internationalization`     | Internationalisation        | COMPLIANCE            | Supported locales, fallback, formatting (when user-visible text is in scope)               |

<!-- /esos:keep -->

### 2.1 Domain-Specific Mandatory Sections

<!-- esos:domain-customization -->

The derived plugin adds the following sections required by `{{ DOMAIN_NAME }}`:

| Section Key                | Label                       | Responsible Agent     | Description                                                       |
| -------------------------- | --------------------------- | --------------------- | ----------------------------------------------------------------- |
| `{{ EXTRA_SECTION_KEY }}`  | `{{ EXTRA_SECTION_LABEL }}` | `{{ AGENT_CLASS }}`   | `{{ EXTRA_SECTION_DESCRIPTION }}`                                 |

Examples (delete the table above and use entries appropriate to the domain):

- `safety_case` (safety-critical software, e.g. ISO 26262, IEC 62304)
- `clinical_workflow` (healthcare)
- `regulatory_filings` (regulated finance)
- `accessibility_conformance` (public sector / WCAG)
- `model_governance` (AI/ML systems)

<!-- /esos:domain-customization -->

## 2.2 Mandatory Companion Documents

<!-- esos:keep -->

A constitution MAY require certain **companion documents** to accompany a specification
(e.g. Threat Model, Risk Register, Architecture Diagram, DPIA, Safety Case). Companion
documents are identified by **kind**, not by filename. They are commonly authored in
office or web formats — `.docx`, `.xlsx`, `.pptx` (and legacy `.ppt`), `.vsdx`, `.pdf`,
`.html`, `.md`, `.txt`, and others a derivation chooses to support — and may contain
images.

The COMPLIANCE specialist verifies, in this order:

1. **Presence** — a document is declared in the spec's `companion_documents:` manifest
   and the referenced `path` or `url` resolves (URL must be on the §5 allowlist).
2. **Type identity** — when opened and extracted into the normalized document model
   (§2.2.1), the document satisfies the kind's `recognition` block (§2.2.2).

This is **identity**, not **quality**. A signature-passing document with sparse content
is still recognized as the declared kind; content quality belongs to human review.

The foundational base declares **no required document kinds**. Derived plugins declare
zero or more kinds in §2.2.4.

### 2.2.1 Normalized Document Model

Every extractor produces:

| Field             | Source examples                                                                |
| ----------------- | ------------------------------------------------------------------------------ |
| `text`            | Full extracted plain text from any format.                                     |
| `headings`        | Word `Heading 1..N` styles; HTML `<h1>..<h6>`; Markdown `#..######`; PDF outline; PowerPoint slide titles. |
| `sections`        | Excel sheet names; Visio page names; PDF top-level outline entries; PowerPoint section names. |
| `tabular_headers` | Excel header rows; HTML `<th>` cells; CSV first row; PowerPoint embedded-table header rows. |
| `metadata.format` | Detected file extension.                                                       |

Unpopulated fields are empty lists. Recognition rules apply to whichever fields the
extractor filled in.

Recognition is **text-based**. Documents whose text is not machine-extractable (scanned
PDFs without OCR, image-only Visio, password-protected files) yield an empty model and
fail recognition. Derived plugins targeting `strict` SHOULD require text-bearing
surfaces (labelled shapes, captions, OCR'd scans).

### 2.2.2 Recognition Rules

| Rule                       | Operates on        | Meaning                                                       |
| -------------------------- | ------------------ | ------------------------------------------------------------- |
| `keywords_min_hits`        | `text`             | At least `count` of `of:` items appear (case-insensitive).    |
| `regex_any_of`             | `text`             | At least one regex matches.                                   |
| `text_contains_all_of`     | `text`             | All listed strings appear.                                    |
| `headings_any_of`          | `headings`         | At least one listed heading appears.                          |
| `headings_all_of`          | `headings`         | All listed headings appear.                                   |
| `sections_any_of`          | `sections`         | At least one listed section name appears.                     |
| `tabular_headers_all_of`   | `tabular_headers`  | All listed column headers appear in some table.               |
| `any_of: [<rule block>]`   | model              | Multi-format alternative — passes if any inner block passes.  |

Rules inside a single `recognition` block are **AND**-combined; `any_of` introduces OR.
Matching is case-insensitive unless a rule uses a case-sensitive regex.

### 2.2.3 Per-Kind Schema

| Field                  | Purpose                                                                         |
| ---------------------- | ------------------------------------------------------------------------------- |
| `kind`                 | Machine key, snake_case (e.g. `threat_model`).                                  |
| `label`                | Human label used in findings.                                                   |
| `applies_when`         | Predicate COMPLIANCE can evaluate from the spec body.                           |
| `template_ref`         | Optional. Starting-point template shipped in the plugin's `templates/` dir.     |
| `allowed_formats`      | List of accepted extensions (e.g. `[docx, pdf, html, md, txt]`).                 |
| `recognition`          | Rule block — see §2.2.2.                                                        |
| `severity_on_missing`  | BLOCKING or ADVISORY (subject to §6.1 tier matrix).                             |
| `severity_on_mismatch` | BLOCKING or ADVISORY (subject to §6.1 tier matrix).                             |

Every instance is referenced from the spec's `companion_documents:` block:

```yaml
companion_documents:
  - kind: threat_model
    path: docs/security/spec-2026-0042-threat-model.docx
  - kind: risk_register
    url: "https://tenant.sharepoint.com/.../risks.xlsx"
```

Each entry MUST provide either `path` (relative to the workspace) or `url` (subject to
§5 External Link Policy). The auditor verifies presence (`path` resolves, or `url` is on
the §5 allowlist) and then opens the artefact to evaluate the kind's `recognition` block.

Responsibility for surfacing missing or mismatched companion documents sits with the
**COMPLIANCE** specialist (§3).

<!-- /esos:keep -->

### 2.2.4 Domain-Specific Companion Document Kinds

<!-- esos:domain-customization -->

```yaml
mandatory_document_kinds:
  - kind: {{ KIND_KEY }}
    label: "{{ KIND_LABEL }}"
    applies_when: "{{ PREDICATE_OR_DESCRIPTION }}"
    template_ref: "templates/{{ KIND_KEY }}.docx"     # optional
    allowed_formats: [docx, pdf]
    recognition:
      any_of:
        - headings_all_of: ["{{ HEADING_A }}", "{{ HEADING_B }}"]
        - keywords_min_hits:
            count: 3
            of: ["{{ KW1 }}", "{{ KW2 }}", "{{ KW3 }}", "{{ KW4 }}"]
    severity_on_missing: BLOCKING
    severity_on_mismatch: BLOCKING
```

Examples by domain:

- `threat_model`, `dpia` — security/privacy-heavy domains
- `risk_register`, `risk_assessment` — risk-managed domains
- `architecture_diagram`, `adr` — architecture-change-heavy domains
- `safety_case` — ISO 26262 / IEC 62304 domains
- `accessibility_report` — public-sector / WCAG

<!-- /esos:domain-customization -->

### 2.3 Section Discipline

The 8 foundational section **keys** (§2) MUST appear in `mandatory_section_keys` (§8) at
every severity tier. How strictly *presence and content* are enforced varies by tier:

| Tier      | All 8 sections required present & non-empty                                                     | `security_compliance` / `internationalization` exception                                                                                       |
| --------- | ----------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| `strict`  | Yes. Empty headers do not count. `N/A — {{ reason }}` permitted only when the change has no plausible impact on that section, and the reason is specific. | Same rule — N/A requires a specific reason. ANALYST treats free-form "N/A" as BLOCKING.                                                         |
| `normal`  | Yes for the 6 non-conditional sections. Empty headers do not count.                              | `N/A — {{ reason }}` is freely permitted for `security_compliance` and `internationalization`; ANALYST treats a free-form N/A as ADVISORY.       |
| `relaxed` | Yes for the 6 non-conditional sections. Empty headers do not count.                              | `security_compliance` and `internationalization` MAY be scoped out via an `assumptions` declaration; the section header MAY then be omitted entirely. |

- The ANALYST specialist treats missing-but-required sections as **BLOCKING** at all tiers
  (the section keys themselves are the foundational floor — the tier only relaxes how
  "present and non-empty" is enforced for the two conditional sections).
- Domain-added sections from §2.1 inherit the tier's discipline unless the derivation
  declares stricter local rules.

---

## 3. Specialist Agent Catalog

<!-- esos:keep -->

This constitution wires the five foundational specialist classes to local agent
prompts. The ordering is fixed:

```
ANALYST → SECURITY → COMPLIANCE → CODING → TESTING → VALIDATION
```

**Analysis steps (ANALYST, SECURITY, COMPLIANCE) MUST execute before implementation
steps (CODING, TESTING).** Derived plugins MAY reorder within those groups but
MUST NOT violate this rule.

<!-- /esos:keep -->

### 3.1 Agent Catalog (esosAgentCatalog)

```json
{
  "esosAgentCatalog": {
    "version": 1,
    "resolvePromptFilesFrom": "CONSTITUTION_REPO",
    "agents": {
      "BUSINESS_REQUIREMENTS_VERIFICATION": {
        "promptFile": "rulesets/analyst.md"
      },
      "SECURITY_REQUIREMENTS_VERIFICATION": {
        "promptFile": "rulesets/security.md"
      },
      "COMPLIANCE_REQUIREMENTS_VERIFICATION": {
        "promptFile": "rulesets/compliance.md"
      },
      "IMPLEMENTATION_ARTIFACT_GENERATION": {
        "promptFile": "rulesets/coding.md"
      },
      "TEST_ARTIFACT_GENERATION": {
        "promptFile": "rulesets/testing.md"
      }
    }
  }
}
```

### 3.2 Specialist Waivers

A derived plugin **MAY** declare a specialist class as waived, but each waiver:

- MUST emit a `CONSTITUTION_WAIVER` trace event with rationale (§7 below).
- MUST be justified in writing in this section.
- MUST NOT waive ANALYST or VALIDATION (the structural backbone).

This generic base **declares no waivers**. Derived plugins extend with rationale
in this section if they need to.

---

## 4. Confidence Thresholds

<!-- esos:keep -->

Default confidence threshold for every specialist step: **0.70** (confidence-based
routing). When a step's confidence falls below the threshold:

1. A `LOW_CONFIDENCE` trace event is emitted with score and threshold.
2. Allowed corrective paths execute (rework by same specialist, escalation, additional
   review) — but only as permitted by this constitution.
3. The run is not marked successful until thresholds are met or the run is explicitly
   stopped.

<!-- /esos:keep -->

### 4.1 Per-Domain Threshold Adjustments

<!-- esos:domain-customization -->

Derived plugins SHOULD raise the threshold for safety-critical, regulated, or
high-blast-radius domains. Suggested defaults (delete and replace with this domain's
choice):

| Risk Profile                                              | Suggested Threshold |
| --------------------------------------------------------- | ------------------- |
| Internal tooling, low blast radius                        | 0.70                |
| Customer-facing, regulated data                           | 0.80                |
| Safety-critical, life-impacting, financial-control-bearing | ≥ 0.85              |

The derived plugin declares the actual threshold(s) used:

```yaml
confidence_thresholds:
  default: {{ DEFAULT_THRESHOLD }}
  per_step:
    SECURITY: {{ SECURITY_THRESHOLD }}
    COMPLIANCE: {{ COMPLIANCE_THRESHOLD }}
```

<!-- /esos:domain-customization -->

### 4.2 Allowed Corrective Paths

A specialist whose output is below threshold MAY:

- **Rework** — re-execute with a refined prompt, up to 3 total attempts.
- **Escalate** — pass to another specialist with a hand-off note.
- **Request additional review** — pause the run and request human review.

A specialist MAY NOT bypass its own findings, mark BLOCKING findings as ADVISORY, or
suppress trace events.

---

## 5. External Link Policy

<!-- esos:keep -->

Constitution Markdown and specifications MAY contain hyperlinks to external resources.
Derived plugins MUST enforce the following at publish time:

- Schemes, hosts, and folder path patterns are validated against the organization's
  allowlist.
- Disallowed targets BLOCK constitution publication with actionable diagnostics.
- ESOS MUST NOT blindly fetch external links; participants follow links with their own
  credentials.
- Any preflight check (e.g., reachability) MUST use controlled egress, MUST deny private
  IP ranges, and MUST deny cloud metadata endpoints (SSRF).
- Non-TLS (`http://`) URLs are rejected by default; the derived plugin MAY allow
  specific intranet exceptions.

<!-- /esos:keep -->

### 5.1 Domain Link Allowlist

<!-- esos:domain-customization -->

```yaml
external_links:
  allowed_schemes: ["https"]
  allowed_hosts:
    - "{{ DOMAIN_DOC_HOST }}"
    - "{{ TENANT_SHAREPOINT_HOST }}"
  blocked_hosts: []
  notes: "{{ ANY_DOMAIN_SPECIFIC_LINK_NOTES }}"
```

<!-- /esos:domain-customization -->

---

## 6. Validation Rules for Repository Content

<!-- esos:keep -->

When a specification is validated:

- Validation runs against **Git-held content at a recorded commit SHA** — not a database
  copy, not a UI draft, not "the latest". For queued runs, the revision is resolved at
  promotion to executing, not at enqueue time.
- Findings reference the specific Git revision validated.
- MCP-initiated and UI-initiated validation MUST produce identical outcomes for the same
  content + constitution (channel parity).
- Findings carry: severity (BLOCKING / ADVISORY), location (file + section), message
  (human-readable), message key (stable identifier for tooling), specialist class, and
  remediation guidance.

<!-- /esos:keep -->

### 6.1 Default Severity Mapping

The categories below have a **base severity** that applies at the `strict` tier (the
default) and a **tier-aware floor** that defines how the category may be demoted at
looser tiers.

**Always BLOCKING — at every tier (foundational floor, untouchable):**

- A mandatory section *key* is missing from `mandatory_section_keys` (§8).
- A requirement is untestable, vague, or contains no RFC 2119 normative keyword.
- Embedded secrets, tokens, or credentials of any kind.
- Missing requirements traceability — no parent artifact (Use Case, Epic, Feature, or User Story) cited for a change that has one.
- An external link target fails the allowlist policy.
- An authorization rule is missing on a sensitive operation.
- A breaking API change is proposed without a migration / deprecation plan.
- A `<!-- esos:keep -->` block is removed or weakened.
- The CODING or TESTING specialist role framing or 14 focus areas are removed or demoted.

**BLOCKING by default; tier-aware demotion permitted:**

| Finding category                                              | `strict`  | `normal`  | `relaxed` |
| ------------------------------------------------------------- | --------- | --------- | --------- |
| `internationalization` declaration absent when user-facing UI is in scope | BLOCKING  | ADVISORY  | ADVISORY  |
| `tech-stack.md` content is generic rather than concrete       | BLOCKING  | ADVISORY  | ADVISORY  |
| Severity-table labels inconsistent (`MAJOR` / `MINOR` / etc.) | BLOCKING  | ADVISORY  | ADVISORY  |
| Applicable regulations cited without specific articles        | BLOCKING  | BLOCKING  | ADVISORY  |
| A `DOMAIN_SPECIFIC_RISKS` entry surfaces nowhere              | BLOCKING  | BLOCKING  | ADVISORY  |
| Acceptance scenarios not in Given/When/Then form              | BLOCKING  | BLOCKING  | ADVISORY  |
| Tech-stack mechanisms named in agents inconsistent with `tech-stack.md` | BLOCKING  | BLOCKING  | ADVISORY  |
| Claude Code subagent or skill file absent                     | BLOCKING  | BLOCKING  | ADVISORY (file presence still BLOCKING under §9) |
| Mandatory companion document missing or unreachable           | BLOCKING  | BLOCKING  | ADVISORY  |
| Companion document fails recognition signature                | BLOCKING  | BLOCKING  | ADVISORY  |
| Companion document has no available extractor                 | BLOCKING  | ADVISORY  | ADVISORY  |

**ADVISORY by default at every tier:**

- Style and clarity suggestions.
- Coverage gaps that don't prevent submission.
- Inconsistent or undefined terminology where the meaning is otherwise clear.
- Missing audit declaration on a low-impact, non-security-relevant operation.
- Section D heuristic findings (`VALIDATION.md` GEN-301..306). At `relaxed`, Section D
  findings are emitted as informational only; they do not appear in the BLOCKING /
  ADVISORY counts.

Derived plugins MAY upgrade any ADVISORY → BLOCKING. They MUST NOT downgrade any
"Always BLOCKING" item, and MUST NOT demote a finding below the floor declared for their
selected tier in the table above.

---

## 7. Traceability and Audit

<!-- esos:keep -->

Every orchestration and validation run produces an immutable audit trail. The trail
MUST be sufficient to answer: who ran what, when, against which content, using which
constitution, with what outcomes.

The following trace events MUST be emitted by ESOS and MUST NOT be disabled by any
derived plugin:

- `RUN_STARTED`, `RUN_COMPLETED`, `RUN_FAILED` — orchestration lifecycle.
- `SPECIALIST_INVOKED`, `SPECIALIST_COMPLETED` — per-step lifecycle, with confidence.
- `LOW_CONFIDENCE` — emitted when a specialist's confidence is below the configured
  threshold (§4).
- `FINDING_EMITTED` — every BLOCKING / ADVISORY finding with its location and message
  key.
- `CONSTITUTION_WAIVER` — recorded when a specialist class is waived (§3.2) or a
  default-BLOCKING finding is downgraded by written justification.
- `VALIDATION_RESULT` — terminal outcome of the validation run.

A derived plugin MUST NOT:

- Disable trace event emission for any of the events listed above.
- Permit traces to contain secrets, tokens, or full sensitive payloads.
- Permit retroactive editing of trace records by non-administrative principals.

<!-- /esos:keep -->

---

## 8. Catalog Metadata

<!-- esos:domain-customization -->

```yaml
constitution_catalog_row:
  logical_id: "{{ DOMAIN_LOGICAL_ID_UUID }}"     # stable across versions
  version: {{ VERSION_INTEGER }}                  # increment on every published change
  name: "{{ DOMAIN_NAME }} Constitution"
  inherits_from: "Generic Plugin Constitution v3.2.0" # this base; required for derivations
  severity_tier: "{{ SEVERITY_TIER }}"            # strict | normal | relaxed (see §1.4)
  mandatory_section_keys:
    - use_case_traceability
    - user_scenarios
    - functional_requirements
    - key_entities
    - success_criteria
    - assumptions
    - security_compliance
    - internationalization
    # add domain-specific keys from §2.1 below
    # - {{ EXTRA_SECTION_KEY }}
  mandatory_document_kinds:
    # Machine-readable mirror of §2.2.4; list the kind keys declared above.
    # - {{ KIND_KEY }}
  domain_tags:
    - "{{ DOMAIN_TAG_1 }}"
    - "{{ DOMAIN_TAG_2 }}"
```

<!-- /esos:domain-customization -->

---

## 9. Repository Layout

A plugin derived from this base MUST keep the following layout. The directory IS the
plugin — `plugin.json` at the root is the manifest, and the whole tree is the plugin
payload. Files marked `required` MUST exist; files marked `optional` MAY be added.

```
{{ DOMAIN_SLUG }}/                              # the plugin root
├── plugin.json                                # required — manifest (spec_version,
│                                                #            inherits_from, provides)
├── constitution.md                            # required — root, contains esosAgentCatalog
├── README.md                                  # required — onboarding & overview
├── CLAUDE.md                                  # required — Claude Code rules for the derivation
├── CHANGELOG.md                               # required — per-version contract deltas
├── VALIDATION.md                              # required — audit prompt
│
├── rulesets/                                  # required — authoritative specialist rulesets
│   ├── analyst.md                             # required
│   ├── security.md                            # required
│   ├── compliance.md                          # required
│   ├── coding.md                              # required — senior dev / software architect
│   └── testing.md                             # required — senior tester / QA architect
│
├── domain/                                    # required
│   ├── glossary.md                            # required
│   ├── governance-policies.md                 # required
│   └── tech-stack.md                          # required
│
├── shared/                                    # required
│   ├── normative-language.md                  # required
│   ├── security-baseline.md                   # required
│   └── acceptance-criteria-format.md          # required
│
├── agents/                                    # required — Claude Code subagent shells
│   ├── esos-analyst.md                        # required
│   ├── esos-security.md                       # required
│   ├── esos-compliance.md                     # required
│   ├── esos-coding.md                         # required
│   └── esos-testing.md                        # required
│
├── templates/                                 # optional — required iff any §2.2.4
│   │                                            #            kind declares `template_ref`.
│   │                                            #            Files may be of any format
│   │                                            #            (.docx, .xlsx, .vsdx, .pdf,
│   │                                            #            .html, .md, .txt, …).
│   └── <kind>.<ext>                           # one per kind that declares template_ref
│
└── skills/                                    # required — Claude Code skills
    ├── esos-finding-emission/                 # required — common
    │   └── SKILL.md                           # required
    ├── esos-ruleset-resolution/               # required — common
    │   └── SKILL.md                           # required
    ├── esos-severity-tier/                    # required — common
    │   └── SKILL.md                           # required
    ├── esos-analyst-review/                   # required — per-role
    │   └── SKILL.md                           # required
    ├── esos-security-review/                  # required
    │   └── SKILL.md                           # required
    ├── esos-compliance-review/                # required
    │   └── SKILL.md                           # required
    ├── esos-coding-review/                    # required
    │   └── SKILL.md                           # required
    ├── esos-testing-review/                   # required
    │   └── SKILL.md                           # required
    └── esos-validate-constitution/            # required — workflow
        └── SKILL.md                           # required
```

Adding files (e.g. `rulesets/safety.md`, `domain/regulations.md`, additional skills) is
permitted; removing required files is not.

The `templates/` directory is **optional** — it MUST exist if and only if any §2.2.4
kind declares a `template_ref`. Files in `templates/` may be of any format the
constitution accepts; the auditor only verifies their existence (`GEN-023`).

The `skills/esos-create-constitution/` skill is **intentionally absent** from
derivations — derived plugins do not derive further plugins. That skill lives only
in the generic base.

**Self-containment** (`plugin.json` → `self_containment.policy: strict`): a derived
plugin MUST contain every file it needs. It MUST NOT introduce a runtime dependency
on another ESOS plugin or on the generic base. If it needs functionality, it
inlines it. This guarantees that installing a domain plugin installs the complete
contract — no surprises later from a base upgrade.

---

## 10. Governance and Amendment

<!-- esos:keep -->

Both this foundational constitution and every derived plugin follow
**semantic versioning**:

- **MAJOR** — removes or incompatibly redefines a specialist class, mandatory section,
  the inheritance contract, or a pipeline rule. Requires sign-off from the
  organization's constitution administrator.
- **MINOR** — adds a specialist skill, adds a mandatory section, raises a threshold,
  tightens severity. May be approved by the domain governance group.
- **PATCH** — clarifications and non-semantic edits. May be approved by the constitution
  maintainer.

Every published version is **immutable**. Changes create new versions. Workspaces
re-attach to the new version explicitly; they are not auto-migrated.

**Re-derivation on MAJOR base bumps.** Because each derived plugin is fully
self-contained (`plugin.json` → `self_containment.policy: strict`), a base MAJOR
bump never silently changes a derived plugin. When the base ships a MAJOR version,
derived plugins continue to operate against the version recorded in their
`plugin.json.inherits_from`. To adopt the new contract, the maintainer re-runs
`esos-create-constitution` from the new base against the original derivation
brief and publishes a new derived plugin version. The ESOS catalog tracks which
base version each plugin inherits from.

Each plugin (the base and every derivation) maintains its own `CHANGELOG.md`
recording the contract delta for every published version.

<!-- /esos:keep -->

---

## 11. How This Base Relates to Derived Plugins

| Aspect                       | Generic Base (this file)              | Derived Plugin (e.g. automotive)                  |
| ---------------------------- | ------------------------------------- | ------------------------------------------------- |
| Mandatory sections           | Foundational minimum set (§2)         | Foundational minimum set + domain-specific additions |
| Confidence threshold         | 0.70 default                          | May raise per risk profile (§4.1)                 |
| Pipeline                     | Fixed order (§3)                      | Same order; specialists may be reordered within groups |
| Specialist waivers           | None                                  | Permitted with rationale (§3.2)                   |
| Tech stack guidance          | Generic principles only               | Concrete stack, versions, exceptions              |
| Glossary                     | Cross-domain terms only               | Domain-specific terms added                       |
| Governance policies          | Cross-domain principles               | Concrete policies, regulations, audit regimes     |
| Severity overrides           | Defaults from §6.1                    | May tighten; never loosen foundational-floor items |
| Severity tier (§1.4)         | Defines three tiers; no tier of its own | Declares one of `strict` / `normal` / `relaxed` in §8 |
| Companion document kinds (§2.2) | None declared                            | May declare any number; each kind has a recognition signature |
| Companion document templates | None                                        | `templates/<kind>.<ext>` shipped when a kind declares `template_ref` |

---

**Version**: 3.2.0 | **Ratified**: 2026-05-17 | **Last Amended**: 2026-05-17
**Inherits from**: none (foundational) | **Manifest**: [`plugin.json`](plugin.json) | **Changes**: [`CHANGELOG.md`](CHANGELOG.md)
