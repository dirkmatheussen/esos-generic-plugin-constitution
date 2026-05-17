# ESOS Generic Plugin Constitution (Foundational)

> **Version**: 3.1.0 | **Ratified**: 2026-05-17 | **Last Amended**: 2026-05-17
> **Type**: Foundational, domain-agnostic constitution **packaged as a Claude
> Code plugin source** — the floor every derivation starts from. It does not
> inherit from any other document.

This repository is the **canonical base** from which domain-specific ESOS
plugins are derived (e.g. automotive, healthcare, fintech, public sector,
retail, AI/ML).

It serves three purposes:

1. **Generation** — combined with [`PROMPT.md`](PROMPT.md), it is the source an
   LLM uses to produce a fully populated, self-contained domain plugin from a
   derivation brief.
2. **Validation** — combined with [`VALIDATION.md`](VALIDATION.md), it is the
   reference an LLM uses to audit a derived plugin against this foundational
   base.
3. **Distribution** — combined with [`plugin.json`](plugin.json), the directory
   itself is a Claude Code plugin source. A team can install this directory as
   a plugin and immediately get the five specialist subagents, the three common
   skills, the five per-role review skills, and the two workflow skills
   (`esos-create-constitution`, `esos-validate-constitution`).

---

## What is ESOS?

**ESOS** — **Enterprise Specification Orchestration System** — is an
AI-assisted pipeline that turns governance into code and applies it before
software gets built. Five specialist agents (analyst, security, compliance,
coding, testing) read each specification, check it against a written
constitution, and emit structured findings. Findings are addressed by editing
the specification and re-running validation — not by overriding agents.

The premise is straightforward:

- **Specifications come before implementation.** A spec describes the user
  scenarios, requirements, success criteria, security and compliance posture,
  and key entities of a change. ESOS validates that spec against the
  organization's rules before any downstream artefact (design doc, code,
  tests) is generated.
- **The rules live in a constitution.** A constitution is a small repository
  of Markdown files (this repo is one) that declares which sections a spec
  MUST contain, what regulations apply, what data sensitivities are in scope,
  what tech stack is sanctioned, and what the per-domain risks are. It is
  governance-as-code: versioned, auditable, attached to workspaces explicitly.
- **Specialist agents enforce the constitution.** Each of the five
  specialists reads only its slice of the spec, runs against a role-specific
  ruleset, and produces BLOCKING or ADVISORY findings. The pipeline order —
  `ANALYST → SECURITY → COMPLIANCE → CODING → TESTING → VALIDATION` — means
  analysis precedes implementation review, every time.
- **Every check leaves a trace.** Trace events (`LOW_CONFIDENCE`,
  `CONSTITUTION_WAIVER`, finding emission) are immutable. The audit trail is
  the contract between the AI-assisted pipeline and the humans who sign off.

ESOS lets a team operate at AI speed without giving up the review discipline
that regulated, safety-critical, or high-blast-radius domains require — and
without re-litigating the same governance debates on every change.

### Where this plugin fits

ESOS organizes everything around two artefacts:

- **The pipeline** — the runtime that orchestrates the specialists, evaluates
  findings, and records trace events.
- **The constitutions** — domain-specific governance documents that the
  pipeline enforces. One per domain (automotive, healthcare, fintech, …);
  each derives from a single foundational base.

This repository is **that foundational base** — packaged as a Claude Code
plugin so any team can install the specialists and the create/validate
workflows in one command (see [Install](#install) below). The base itself
isn't attached to a workspace; it's the floor every domain constitution
starts from. The rest of this README describes how to use it.

---

## Why a generic plugin base

This constitution is **foundational** — it is the floor. It defines the ground
rules every domain plugin must satisfy: the five specialist agent classes, the
pipeline ordering, the minimum mandatory specification sections, the audit and
traceability rules, the severity tier system.

Without it, every domain team would re-derive the same scaffolding from
scratch and would inevitably drift (forgetting a mandatory section,
mis-ordering the pipeline, weakening a `MUST`). The base captures the durable
cross-domain choices once and ships them as an installable plugin.

```
Generic Plugin Constitution (this)     ground rules — the floor
        │
        ▼ derive
Domain Plugins                          concrete domain rules — the substance
  e.g. esos-automotive-spare-parts,
       esos-healthcare,
       esos-fintech, ...
```

**Every derived plugin is fully self-contained.** It carries its own complete
copy of the rulesets, the common and review skills, the subagent shells, and
the validate workflow skill. No cross-plugin runtime dependencies. Updating
the base requires conscious re-derivation; nothing under a domain plugin
changes silently.

---

## Repository structure

```
generic-plugin-constitution/
├── plugin.json                                # manifest (spec_version, inherits_from)
├── .claude-plugin/
│   └── marketplace.json                       # single-plugin marketplace
│                                              # wrapper — enables /plugin install
├── README.md                                  # this file
├── CLAUDE.md                                  # rules orienting Claude in this repo
├── MANUAL.md                                  # end-user manual
├── CHANGELOG.md                               # per-version contract deltas
├── PROMPT.md                                  # derivation prompt — generation flow
├── VALIDATION.md                              # audit prompt — validation flow
├── constitution.md                            # root + esosAgentCatalog wiring
│
├── rulesets/                                  # authoritative specialist rulesets
│   ├── analyst.md                             #   (was: agents/ in v2.x; renamed
│   ├── security.md                            #    in v3.0.0 to disambiguate from
│   ├── compliance.md                          #    Claude Code's `agents/` term)
│   ├── coding.md                              #   senior dev / software architect
│   └── testing.md                             #   senior tester / QA architect
│
├── domain/
│   ├── glossary.md                            # cross-domain terms
│   ├── governance-policies.md                 # cross-domain governance principles
│   └── tech-stack.md                          # generic tech-stack guidance
│
├── shared/
│   ├── normative-language.md                  # RFC 2119
│   ├── security-baseline.md                   # cross-cutting security checklist
│   └── acceptance-criteria-format.md          # Given/When/Then
│
├── agents/                                    # Claude Code subagent shells (SLIM)
│   ├── esos-analyst.md
│   ├── esos-security.md
│   ├── esos-compliance.md
│   ├── esos-coding.md
│   └── esos-testing.md
│
└── skills/
    ├── esos-finding-emission/SKILL.md         # COMMON  — finding format & discipline
    ├── esos-ruleset-resolution/SKILL.md       # COMMON  — ruleset precedence
    ├── esos-severity-tier/SKILL.md            # COMMON  — strict/normal/relaxed
    ├── esos-analyst-review/SKILL.md           # REVIEW  — analyst focus walk
    ├── esos-security-review/SKILL.md          # REVIEW  — security focus walk
    ├── esos-compliance-review/SKILL.md        # REVIEW  — compliance focus walk
    ├── esos-coding-review/SKILL.md            # REVIEW  — 14 coding focus areas
    ├── esos-testing-review/SKILL.md           # REVIEW  — 14 testing focus areas
    ├── esos-create-constitution/SKILL.md      # WORKFLOW — base only; derive new plugin
    └── esos-validate-constitution/SKILL.md    # WORKFLOW — audit a candidate plugin
```

> **Why `rulesets/` and not `agents/`?** In v2.x, `agents/<role>.md` held the
> constitution's authoritative specialist rulesets, and `.claude/agents/esos-<role>.md`
> held the Claude Code subagent shells. The shared name was a permanent source
> of confusion. In v3.0.0, the constitution's rulesets are renamed `rulesets/`
> and the subagent shells live unambiguously under `agents/`. See
> [`CHANGELOG.md`](CHANGELOG.md) for the full migration note.

---

## Three rings of behavior

```
   Workflow skills            esos-create-constitution
                              esos-validate-constitution
                                      │ compose
                                      ▼
   Common skills              esos-finding-emission
   (cross-cutting             esos-ruleset-resolution
    discipline)               esos-severity-tier
                                      │ used by
                                      ▼
   Per-role review skills     esos-analyst-review
   (focus areas +             esos-security-review
    message keys +            esos-compliance-review
    role-specific             esos-coding-review
    deliverables)             esos-testing-review
                                      │ invoked by
                                      ▼
   Subagent shells            esos-{analyst,security,compliance,
                                    coding,testing}  (SLIM)
                                      │ read
                                      ▼
   Authoritative              rulesets/{analyst,security,compliance,
   rulesets                            coding,testing}.md
                              domain/   shared/
```

Read [CLAUDE.md](CLAUDE.md) for the editing rules; read [`constitution.md`](constitution.md)
for the rules of ESOS itself.

---

## Customization markers

The base uses three marker conventions to indicate where derivation is
required:

| Marker | Meaning |
|---|---|
| `{{ TOKEN }}` | Inline placeholder — every occurrence MUST be replaced in a derivation. |
| `<!-- esos:domain-customization -->` | The block that follows MUST be rewritten or extended by the derived plugin. |
| `<!-- esos:keep -->` | The block is part of the inherited floor — derivations MUST keep it (verbatim or stricter). |

A derived plugin that still contains any `{{ TOKEN }}` or unreplaced
`<!-- esos:domain-customization -->` block has not finished derivation —
validation will flag it as BLOCKING.

---

## Severity tiers

Every derived plugin declares one of three **severity tiers** in its
`constitution.md` §8. The tier calibrates how strictly the generic baseline
checks (Sections B–D of `VALIDATION.md`) gate publication and how strictly
mandatory-section presence is enforced. It does **not** relax any
foundational-floor rule — Section A of `VALIDATION.md` remains BLOCKING at
every tier.

| Tier | Intent | Typical fit |
|---|---|---|
| `strict` | Maximum enforcement — all baseline defaults BLOCKING; quality heuristics ADVISORY. | Regulated, safety-critical, or high-blast-radius domains. **Default when the brief omits the tier.** |
| `normal` | Balanced — stylistic and concreteness checks demote to ADVISORY; structural checks stay BLOCKING. | Mainstream business domains where rigour is expected but pragmatism is rewarded. |
| `relaxed` | Minimum above the foundational floor — many baseline checks ADVISORY; Section D informational. | Internal tooling, prototypes, low-blast-radius derivations. |

The full per-check tier matrix lives in [`VALIDATION.md`](VALIDATION.md) under
"Severity Tier (read first)". The conceptual definitions are in
[`constitution.md`](constitution.md) §1.4. The resolution discipline (how to
read it, when to default, how to handle a brief/candidate mismatch) lives in
the `esos-severity-tier` common skill.

Tightening a published tier (`normal` → `strict`) is a MINOR semver change.
Loosening (`strict` → `normal`) is a MAJOR change — it weakens the published
contract that downstream workspaces depend on.

---

## Install

The repo ships `plugin.json` plus `.claude-plugin/marketplace.json` and is
installable in Claude Code via the standard `/plugin` flow. Pick one:

```text
# From GitHub (recommended; pin to @<tag> or @<sha> for production):
/plugin marketplace add dirkmatheussen/esos-generic-plugin-constitution
/plugin install esos-generic-plugin-constitution@esos-generic-plugin-constitution

# From a local clone:
/plugin marketplace add /absolute/path/to/generic-plugin-constitution
/plugin install esos-generic-plugin-constitution@esos-generic-plugin-constitution

# Ad-hoc local test, no marketplace registration:
claude --plugin-dir /absolute/path/to/generic-plugin-constitution
```

For a declarative team setup via `settings.json` and the equivalent flow for a
**derived** domain plugin, see [`MANUAL.md` §2.4](MANUAL.md#24-installing-the-generic-base-as-a-claude-code-plugin)
and [§9.4](MANUAL.md#94-installing-the-derived-plugin-in-claude-code).

---

## Using this with Claude Code

This base ships as a **Claude-native toolkit**. Open the repo in Claude Code
and:

| To do this | Use this |
|---|---|
| Create a new domain plugin from a brief | Skill `esos-create-constitution` (or describe the task — the skill auto-triggers on phrases like "create a constitution for healthcare") |
| Audit a candidate domain plugin | Skill `esos-validate-constitution` (or "validate this constitution") |
| Deep-dive on a specialist's territory | Spawn the matching subagent: `esos-analyst`, `esos-security`, `esos-compliance`, `esos-coding`, `esos-testing` |
| Apply a single specialist's review without spawning the subagent | Invoke the matching skill: `esos-analyst-review`, `esos-security-review`, `esos-compliance-review`, `esos-coding-review`, `esos-testing-review` |
| Understand the rules of this repo | Read [`CLAUDE.md`](CLAUDE.md) (Claude Code reads it automatically) |
| Understand version history | Read [`CHANGELOG.md`](CHANGELOG.md) |

**Single source of truth**: subagent shells are thin wrappers — their
authoritative ruleset is the corresponding `rulesets/<role>.md` file. Their
focus-area discipline lives in the matching `esos-<role>-review` skill.
Updates to the ruleset flow to the subagent automatically; no duplication.

**Portability**: a derived plugin is the same shape as this base, minus
`esos-create-constitution`. Install it like any Claude Code plugin.

---

## Generation flow (creating a domain plugin)

1. The constitution administrator collects a **derivation brief** for the new
   domain. See `esos-create-constitution` and `PROMPT.md` for the field list.
2. The brief is supplied to an LLM together with this base and `PROMPT.md`.
3. The LLM produces a candidate domain plugin that mirrors this base's
   structure 1:1, with all `{{ TOKEN }}`s replaced and all
   `<!-- esos:domain-customization -->` blocks rewritten. The candidate is
   **fully self-contained** — every ruleset, every skill, every shared file
   is inlined.
4. The candidate is validated (see below) before publication.

## Validation flow (auditing a derived plugin)

1. The candidate domain plugin is supplied to an LLM together with this base
   and [`VALIDATION.md`](VALIDATION.md).
2. The LLM walks the derived plugin against every check in `VALIDATION.md`
   and produces a finding set with severity, location, message, and
   remediation.
3. BLOCKING findings prevent publication; ADVISORY findings are recorded for
   the maintainer's attention.
4. Findings are addressed by editing the derived plugin and re-running
   validation.

---

## Relationship to derived plugins

- This foundational base describes **what every plugin must satisfy** and
  provides a sensible starting point.
- A derived plugin describes **a specific domain's rules within that starting
  point**.

A derived plugin may tighten anything in this base. It may extend with
additional sections, agents, glossary entries, policies, or stack rules. It
may **not** weaken any rule this base flags as a `MUST` or `MUST NOT`, and
its `plugin.json` MUST declare
`inherits_from: "esos-generic-plugin-constitution@3.1.0"` (or higher).

A derived plugin MUST NOT introduce a runtime dependency on another plugin
(see `plugin.json` → `self_containment.policy: strict`). If it needs
functionality, it inlines it.

---

## Related resources

- [`MANUAL.md`](MANUAL.md) — end-user manual: build a domain plugin step-by-step
- [`constitution.md`](constitution.md) — root document
- [`CLAUDE.md`](CLAUDE.md) — rules orienting Claude Code in this repo
- [`PROMPT.md`](PROMPT.md) — derivation prompt template (drives the create skill)
- [`VALIDATION.md`](VALIDATION.md) — validation prompt template (drives the validate skill, ~40 numbered checks)
- [`CHANGELOG.md`](CHANGELOG.md) — per-version contract deltas
- [`plugin.json`](plugin.json) — plugin manifest
