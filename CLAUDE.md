# CLAUDE.md — ESOS Generic Plugin Constitution (v3.2.0)

This repository is the **plugin-shaped, domain-agnostic base** from which
domain-specific ESOS plugins are derived. The directory you are reading is itself
a Claude Code plugin source — its `plugin.json`, `agents/`, `skills/`,
`rulesets/`, `domain/`, and `shared/` together form an installable plugin payload.

## What you do here

When the user works in this repo, the typical intents are:

1. **Create a domain-specific plugin** (e.g. healthcare, fintech, automotive)
   → invoke the `esos-create-constitution` skill. The output is a structurally
   identical, fully self-contained sibling directory.
2. **Validate / audit a candidate domain plugin** → invoke the
   `esos-validate-constitution` skill.
3. **Modify the base itself** → see `constitution.md`, `rulesets/`, `agents/`,
   `skills/`, `domain/`, `shared/`. Changes to the base are versioned per
   `constitution.md` §10 (semver) and recorded in `CHANGELOG.md`. Confirm with
   the user before editing the base — most users intend to derive, not to
   amend the base.

## Hard rules (the floor)

This constitution is **foundational** — it is the floor. There is no document
above it. The rules below are asserted by `constitution.md` and the five
`rulesets/*.md` files; you MUST NOT propose changes that violate them.

- **Five specialist classes are fixed** (`constitution.md` §3):
  - ANALYST → `BUSINESS_REQUIREMENTS_VERIFICATION`
  - SECURITY → `SECURITY_REQUIREMENTS_VERIFICATION`
  - COMPLIANCE → `COMPLIANCE_REQUIREMENTS_VERIFICATION`
  - CODING → `IMPLEMENTATION_ARTIFACT_GENERATION` — senior developer /
    software architect
  - TESTING → `TEST_ARTIFACT_GENERATION` — senior tester / QA architect
- **Pipeline ordering** is `ANALYST → SECURITY → COMPLIANCE → CODING → TESTING
  → VALIDATION` (`constitution.md` §3). Analysis MUST precede implementation.
- **Mandatory section minimum** is the 8-key set in `constitution.md` §2 —
  derived plugins MAY add, MUST NOT remove.
- **No secrets, tokens, API keys, or credentials** anywhere in any artifact.
- **Audit trail integrity** — never propose changes that disable trace events
  (`constitution.md` §7) or permit retroactive trace edits.
- **Self-containment is the law** — derived plugins MUST contain every file
  they need to operate. No cross-plugin runtime dependencies between ESOS
  plugins. See `plugin.json` → `self_containment.policy: strict`.

## Layout — what lives where

| Path | What it is | Touch? |
|---|---|---|
| `plugin.json` | Manifest. `spec_version`, `inherits_from`, provided agents/skills, self-containment policy. | Only on base version bump. |
| `.claude-plugin/marketplace.json` | Single-plugin marketplace wrapper. Enables `/plugin marketplace add` + `/plugin install` for this directory. Keep `metadata.version` and the plugin entry's `version` in sync with `plugin.json.version`. | Only on base version bump. |
| `constitution.md` | Root constitution. Defines specialist classes, mandatory sections, pipeline, audit, severity tier system. | Only on base version bump. |
| `rulesets/<role>.md` | **Authoritative** specialist rulesets — the constitution's source of truth for each of the five specialists. Read by ESOS runtime AND by the review skills. | Only on base version bump. |
| `domain/` | Cross-domain glossary, governance principles, tech-stack guidance. Specialized in derivations. | Only on base version bump. |
| `shared/` | Domain-agnostic includes (RFC 2119, GWT format, security baseline). Reused as-is by derivations. | Only on base version bump. |
| `agents/esos-<role>.md` | Claude Code subagent shells. Slim — role identity + pipeline position + pointers to skills. | Only on base version bump. |
| `skills/esos-finding-emission/` | Common skill — YAML finding format and discipline. | Only on base version bump. |
| `skills/esos-ruleset-resolution/` | Common skill — how to find and apply the right ruleset. | Only on base version bump. |
| `skills/esos-severity-tier/` | Common skill — strict / normal / relaxed resolution. | Only on base version bump. |
| `skills/esos-<role>-review/` | Per-role review skill — focus-area walk, message-key catalog, role-specific deliverables. | Only on base version bump. |
| `skills/esos-create-constitution/` | Workflow skill — derive a domain plugin from this base. **Base only**; not shipped in derivations. | Only on base version bump. |
| `skills/esos-validate-constitution/` | Workflow skill — audit a candidate plugin. | Only on base version bump. |
| `PROMPT.md` | Long-form derivation prompt. The `esos-create-constitution` skill points to it. | Only on base version bump. |
| `VALIDATION.md` | Long-form audit prompt with ~40 numbered checks. The `esos-validate-constitution` skill points to it. | Only on base version bump. |
| `MANUAL.md` | End-user manual: build a domain plugin step-by-step. | Light edits OK without semver bump. |
| `CHANGELOG.md` | Per-version contract deltas. | Append on every release. |

## Customization markers

Three markers govern what a derivation may change:

| Marker | Meaning |
|---|---|
| `{{ TOKEN }}` | Inline placeholder; every occurrence MUST be replaced in derivations. |
| `<!-- esos:domain-customization -->` | The block that follows MUST be rewritten with domain content. |
| `<!-- esos:keep -->` | Inherited floor; derivations MUST keep this verbatim or tighten it (never relax). |

A candidate that still contains any unreplaced `{{ }}` token or unmodified
`<!-- esos:domain-customization -->` block has not finished derivation.

## Severity tiers (derivation contract)

Every derivation declares one of three **severity tiers** in `constitution.md`
§8 — `strict`, `normal`, `relaxed`. The tier calibrates which baseline checks
BLOCK vs ADVISE at validation time. It never relaxes Section A (foundational
conformance) of `VALIDATION.md` or any other "Never tier-aware" check.

The full per-check matrix is in `VALIDATION.md` under "Severity Tier (read
first)"; conceptual definitions are in `constitution.md` §1.4; tier-resolution
discipline is in the `esos-severity-tier` skill.

When deriving, ask the user for the tier if the brief is silent — do not pick
a looser tier on their behalf. When auditing, resolve the tier from the
candidate's `constitution.md` §8 before walking checks, and record it in the
summary.

## How the pieces fit (the three rings)

```
   ┌────────────────────────────────────────────────────────────────┐
   │ Workflow skills                                                 │
   │ esos-create-constitution    esos-validate-constitution           │
   └────────────────────────────────┬───────────────────────────────┘
                                    │ compose
                                    ▼
   ┌────────────────────────────────────────────────────────────────┐
   │ Common skills (cross-cutting discipline, written ONCE)          │
   │ esos-finding-emission    esos-ruleset-resolution                 │
   │ esos-severity-tier                                              │
   └────────────────────────────────┬───────────────────────────────┘
                                    │ are used by
                                    ▼
   ┌────────────────────────────────────────────────────────────────┐
   │ Per-role review skills (focus areas + message keys + outputs)   │
   │ esos-analyst-review   esos-security-review   esos-compliance-… │
   │ esos-coding-review    esos-testing-review                      │
   └────────────────────────────────┬───────────────────────────────┘
                                    │ are invoked by
                                    ▼
   ┌────────────────────────────────────────────────────────────────┐
   │ Subagent shells (role identity + pipeline position; SLIM)       │
   │ esos-analyst   esos-security   esos-compliance                  │
   │ esos-coding    esos-testing                                     │
   └────────────────────────────────┬───────────────────────────────┘
                                    │ read
                                    ▼
   ┌────────────────────────────────────────────────────────────────┐
   │ Authoritative rulesets (the constitution's source of truth)     │
   │ rulesets/{analyst,security,compliance,coding,testing}.md        │
   │  +  domain/    +  shared/                                       │
   └────────────────────────────────────────────────────────────────┘
```

## Default behavior

- **Treat `constitution.md` as the source of truth** for ESOS rules. Never
  invent.
- **Before editing the base**, confirm with the user that this is a base-edit
  (which triggers a semver bump and a CHANGELOG entry) and not a derivation.
- **Use the skills** for create/validate workflows — don't freelance a
  derivation prompt or audit checklist.
- **Spawn the matching subagent** when deep-diving on a specialist's
  territory. The subagent will pull in `esos-<role>-review` plus the three
  common skills.
- **Memory hygiene**: when the user sets up a derivation brief, save the
  brief to a scratch file (e.g. `/tmp/esos-brief-<slug>.md`) so the work is
  resumable, not in conversation memory only.

## What you do NOT do here

- Do NOT write production source code in this repo. This is a constitution
  base, not an implementation.
- Do NOT propose deriving by copying anything other than this base.
- Do NOT remove or weaken `<!-- esos:keep -->` blocks during derivation.
- Do NOT collapse the 14 focus areas in `rulesets/coding.md` or
  `rulesets/testing.md` — those role framings are protected by
  `VALIDATION.md` checks GEN-103a / GEN-103b.
- Do NOT introduce cross-plugin runtime dependencies. Self-containment is
  the law (`plugin.json` → `self_containment.policy`).

## When asked something the skills don't cover

If the user asks a question outside create/validate (e.g. "what's the
difference between this base and v2.x?", "explain the pipeline ordering",
"why are sections mandatory here but recommended elsewhere?"), answer from
`constitution.md` directly. Cite section numbers. Don't invent rationale.
For version-history questions, cite `CHANGELOG.md`.
