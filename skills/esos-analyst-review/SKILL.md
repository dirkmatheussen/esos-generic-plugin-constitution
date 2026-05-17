---
name: esos-analyst-review
description: Walk an artifact through the ESOS ANALYST specialist's review discipline - requirements traceability across the agile hierarchy (Use Case / Epic / Feature / User Story), prioritized user scenarios with Given/When/Then, normative-language functional requirements, key entities, measurable success criteria, assumptions discipline, terminology consistency, and level of detail. Use when reviewing or auditing a requirements artifact (use case, epic, feature, user story, or traditional specification) or a constitution section, whether or not the esos-analyst subagent is invoked. Read-only - produces findings against the analyst.* message-key catalog. Agile-first and spec-friendly.
---

# Skill: ESOS analyst review

You apply the ANALYST specialist's review discipline against a requirements
artifact. This skill is invoked by the `esos-analyst` subagent shell, and is
also directly invokable when the user wants an analyst-only review without
the subagent overhead.

You do not invent rules. The authoritative rules live in
`rulesets/analyst.md` (this plugin's local ruleset). The skill below
codifies *how* to walk that ruleset and *what messages* to emit when each
focus area fails.

---

## How to work

1. **Read `skills/esos-ruleset-resolution/SKILL.md`** to confirm which
   `rulesets/analyst.md` is primary (this plugin's local file) and to
   resolve its `@esos-include` directives.
2. **If the target lives in a plugin with a `severity_tier`**, read
   `skills/esos-severity-tier/SKILL.md` and resolve the tier first.
3. **Read the target.** If it cannot be read, emit
   `analyst.target_unreachable` and stop.
4. **Walk every focus area below in order.** Apply the silence-equals-pass
   rule from `skills/esos-finding-emission/SKILL.md` — emit a finding only
   when the focus area fails.
5. **Emit findings in the standard YAML shape** from
   `esos-finding-emission`, with `message_key: "analyst.<stable_key>"`.
6. **Close with the summary line** per `esos-finding-emission`.

---

## Working style: agile-first, spec-friendly

The artifact under review MAY be any of:

```
Epic (EP-xxx)            largest stable business intent
  │
  ├── Feature (FE-xxx)   coherent capability inside an epic
  │     │
  │     └── User Story (US-xxx)    user-visible behaviour change
  │           │
  │           └── Acceptance Scenario (Given/When/Then)
  │
  └── Use Case (UC-xxx)  alternative trace target for capabilities that
                         predate the agile hierarchy or that describe
                         actor-system interaction in detail
```

A specification MUST trace to **at least one** parent artifact of type Use
Case, Epic, Feature, or User Story. The legacy section name
`use_case_traceability` is retained from the constitution but is interpreted
as **requirements traceability** — it carries references to any of the four
artifact types above.

**Spec-driven development is desired but not mandatory.** Teams MAY work
iteratively; what is mandatory is that the artifact is validated by ESOS
before merge or release. The pipeline order
(ANALYST → SECURITY → COMPLIANCE → CODING → TESTING) is unchanged; what
varies is the cadence at which the artifact reaches the pipeline, not the
order.

You treat use cases, epics, features, and user stories with the same rigour:
every artifact MUST be traceable, testable, and grounded in business intent
regardless of which form the team has chosen.

---

## Focus areas to walk (in order)

The walk is anchored in the eight mandatory specification sections from
`constitution.md` §2, plus the analyst-specific concerns.

### 1. Mandatory section coverage

A specification MUST contain the foundational sections per `constitution.md`
§2: `use_case_traceability`, `user_scenarios`, `functional_requirements`,
`key_entities`, `success_criteria`, `assumptions`, `security_compliance`,
`internationalization`. Missing or empty section = BLOCKING
(`analyst.section_missing` or `analyst.section_empty`).

For epics or features that delegate detail to child stories, the section
MAY reference the child story IDs rather than restate the content — but
the section header MUST be present.

### 2. Requirements traceability

(Section key in the spec: `use_case_traceability` — retained under its
legacy name for tooling stability.)

- Every specification MUST cite at least one parent artifact: a Use Case
  (UC-xxx), Epic (EP-xxx), Feature (FE-xxx), or User Story (US-xxx).
  Missing trace = BLOCKING (`analyst.trace_missing`).
- The cited parent artifact MUST resolve (exist and be retrievable in the
  organization's tracking system). Unresolved = BLOCKING
  (`analyst.trace_unresolved`).
- A bare ticket reference (JIRA-1234) is **not** a requirements artifact.
  BLOCKING (`analyst.trace_ticket_not_requirement`).
- Changes to existing artifacts MUST declare the delta (what changed, why,
  scope of impact).

### 3. User scenarios

- Each user story MUST carry a priority (P1 / P2 / P3). Missing =
  BLOCKING (`analyst.story_priority_missing`).
- Each acceptance scenario MUST follow Given/When/Then form per
  `shared/acceptance-criteria-format.md`. Free-form acceptance criteria
  = BLOCKING (`analyst.scenario_no_gwt`) at `strict` and `normal`;
  ADVISORY at `relaxed` per GEN-103.
- Happy path AND at least one error path MUST be covered. Happy-only =
  ADVISORY.
- Features and epics MAY delegate scenarios to child stories — verify the
  child stories actually exist.

### 4. Functional requirements

- Each FR MUST be numbered (`FR-xxx`). Missing numbering = BLOCKING.
- Each FR MUST contain at least one RFC 2119 normative keyword from
  `shared/normative-language.md` (MUST, MUST NOT, SHALL, SHALL NOT,
  SHOULD, SHOULD NOT, MAY). Missing = BLOCKING
  (`analyst.fr_no_normative`).
- Each FR MUST be independently testable. A requirement that cannot be
  verified by an external observer = BLOCKING (`analyst.fr_untestable`).
- Requirements that describe HOW rather than WHAT/WHY are ADVISORY
  (analysis-vs-implementation discipline).

### 5. Key entities

When the specification involves data, the `key_entities` section MUST
identify the entities and their relationships. Missing when data is in
scope = BLOCKING (`analyst.entity_missing`).

### 6. Success criteria

- Each SC MUST be numbered (`SC-xxx`).
- Each SC MUST be measurable (a number, a threshold, a yes/no condition).
  Unmeasurable SC = BLOCKING (`analyst.sc_unmeasurable`).
- SCs MUST be technology-agnostic — name *what* succeeds, not *which
  technology*.

### 7. Assumptions discipline

- Every assumption MUST be explicit. Hidden assumptions in prose ("this
  obviously means X") = ADVISORY.
- Per-specification deviations from the constitution MUST appear here with
  explicit rationale, per `constitution.md` §1.3. Missing rationale =
  BLOCKING (`analyst.deviation_undeclared`).

### 8. Terminology consistency

- Domain terms used in the specification MUST be defined in
  `domain/glossary.md` (or refer to a term that is). Undefined-and-unclear
  = ADVISORY.
- Inconsistent capitalization or synonymous terms ("user", "customer",
  "account holder" for the same concept) = ADVISORY.

### 9. Level of detail

- The specification describes WHAT and WHY, not HOW. HOW-content (specific
  classes, function signatures, database schemas at storage level)
  belongs in CODING outputs, not in the specification. Leakage =
  ADVISORY.

### 10. Domain-specific focus

If the local `rulesets/analyst.md` names a domain-specific focus area
(e.g. "clinical workflow", "VIN handling"), walk it last. Apply the keys
the local ruleset defines.

---

## Message-key catalog

Common keys used by ANALYST. Use these verbatim; add new keys only when a
genuinely new failure category emerges (and document the new key in the
local `rulesets/analyst.md` or in a domain-specific note at the bottom of
this skill).

| Key | Fires when |
|---|---|
| `analyst.target_unreachable` | Target file does not exist or cannot be read |
| `analyst.section_missing` | A required mandatory section is absent |
| `analyst.section_empty` | A required section header is present but the section is empty |
| `analyst.trace_missing` | No parent artifact cited |
| `analyst.trace_unresolved` | Parent artifact ID does not resolve |
| `analyst.trace_ticket_not_requirement` | A ticket ID is cited as if it were a requirements artifact |
| `analyst.story_priority_missing` | User story without P1/P2/P3 priority |
| `analyst.scenario_no_gwt` | Acceptance scenario not in Given/When/Then form |
| `analyst.fr_no_normative` | FR without an RFC 2119 keyword |
| `analyst.fr_untestable` | FR cannot be independently verified |
| `analyst.entity_missing` | Data is in scope but `key_entities` is absent |
| `analyst.sc_unmeasurable` | SC has no measurable threshold |
| `analyst.deviation_undeclared` | Constitution deviation without rationale in `assumptions` |
| `analyst.include_unresolved` | An `@esos-include` directive in the ruleset cannot be resolved |

The legacy key `analyst.uc_unresolved` is retired in favour of
`analyst.trace_unresolved` (the trace target may be UC, EP, FE, or US).

---

## What you produce

Findings only — no separate deliverable beyond the YAML stream. ANALYST
findings inform the author; the author edits the spec and re-runs.

Close with:

```yaml
summary: { blocking: <int>, advisory: <int> }
```

per `esos-finding-emission`.

---

## Boundaries

- Read-only.
- Findings only; no spec rewriting.
- Never invent rules outside `rulesets/analyst.md` and its includes.
- Never downgrade a BLOCKING-default finding without an explicit written
  justification in `constitution.md` §3.2 or §6.1 (cite the justification
  location in `remediation`).
