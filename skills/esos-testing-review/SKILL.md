---
name: esos-testing-review
description: Walk an artifact through the ESOS TESTING specialist's review discipline - all 14 protected focus areas. The TESTING specialist is positioned as a senior tester / QA architect (per VALIDATION.md GEN-103b) who ensures the system is verified at industrial level and produces test strategy reviews, traceability matrices, coverage assessments, skeletal Given/When/Then outlines, and outcome-handling reviews - NOT mass test-case generation from a template. Use when reviewing or auditing a specification, test plan, test suite, or CI configuration for industrial-level test quality - strategy fit, requirement-to-test traceability, test pyramid shape, test quality (no sleep / no real clock / no unseeded random), multi-dimensional coverage (requirement / code / branch / mutation / boundary / configuration), test types (unit / integration / contract / E2E / performance / security / accessibility / chaos), test data management (no production data, no real PII), hermetic environments, outcome handling (CI gates, flake quarantine), continuous verification, anti-patterns, and compliance verification. Read-only - produces strategy reviews, traceability matrices, coverage assessments, and findings against the testing.* message-key catalog.
---

# Skill: ESOS testing review

You apply the TESTING specialist's review discipline against an artifact.
The authoritative rules live in `rulesets/testing.md`, where all 14 focus
areas and the Test Anti-Patterns table are detailed. This skill codifies
the walk, the role-specific deliverables, and the message-key catalog.

The TESTING specialist's identity — **senior tester / QA architect**,
all 14 focus areas, the Test Anti-Patterns table retained — is protected
by `VALIDATION.md` GEN-103b. Do not collapse, demote, or skip any focus
area.

---

## How to work

1. Read `skills/esos-ruleset-resolution/SKILL.md` to load the primary
   ruleset (`rulesets/testing.md`) and its includes (`domain/glossary.md`,
   `domain/governance-policies.md`,
   `shared/acceptance-criteria-format.md`).
2. If the target lives in a plugin with a `severity_tier`, read
   `skills/esos-severity-tier/SKILL.md` and resolve the tier first.
3. Read the target. If unreachable, emit `testing.target_unreachable`
   and stop.
4. Walk **every** focus area below in order. Do not skip any.
5. Emit findings with
   `message_key: "testing.<category>.<stable_key>"` per
   `skills/esos-finding-emission/SKILL.md`.
6. Produce the role-specific deliverables (see "What you produce").
7. Close with the summary line.

---

## The 14 focus areas to walk (in order)

### 1. Test Strategy and Approach

Risk-based selection; shift-left; behaviour over implementation;
negative-space coverage (what should NOT happen); continuous
verification in production. A test plan that exercises only happy
paths = BLOCKING (`testing.scenario.no_negative_coverage`).

### 2. Requirement-to-Test Traceability

Every FR / SC / AC / UC has at least one test that proves it. Produce
a traceability matrix. Untraced requirement = BLOCKING
(`testing.traceability.fr_no_coverage`).

### 3. Test Pyramid and Right-Shape

Unit / integration / contract / E2E in healthy proportions. Detect
anti-shapes: ice-cream cone (E2E-heavy), cupcake (snapshot + E2E),
hourglass (no middle layer). Wrong shape = BLOCKING for new test
suites (`testing.pyramid.ice_cream_cone`,
`testing.pyramid.cupcake`, `testing.pyramid.hourglass`).

### 4. Test Quality

- Independence — tests run in any order, no shared mutable state.
- Determinism — no `sleep`, no real-clock dependency, no unseeded
  random. Each = BLOCKING (`testing.quality.sleep_synchronization`,
  `testing.quality.real_clock`, `testing.quality.unseeded_random`).
- Speed — unit tests under a few hundred ms each.
- Diagnostic failures — failure messages name the expectation.
- AAA structure (Arrange / Act / Assert) or equivalent.
- Descriptive names — the test name describes the behaviour, not the
  method.
- No test logic — branching inside a test is a smell.
- Behaviour over implementation — assertions on observable outcomes,
  not on internal call counts (unless the call is the behaviour).
- No excessive mocking — mocks at trust boundaries only.

### 5. Coverage (Multi-Dimensional)

Coverage is not a single number. Walk every dimension:

- Requirement coverage (linked to traceability).
- Code coverage (line, statement) — typically ≥ 80% on changed code.
- Branch coverage — typically ≥ 70% on changed code.
- Mutation coverage — surface untested logic that survives mutation.
- State coverage — every state transition tested.
- Configuration / feature-flag coverage — flag combinations tested.
- Boundary coverage — limits, off-by-one, empty, max.
- Equivalence-partition coverage — representative inputs per partition.
- Negative coverage — every error path.

A coverage threshold silently lowered = BLOCKING
(`testing.coverage.threshold_lowered`).

### 6. Test Types in Detail

Unit, integration, contract, E2E, performance, security,
resilience / chaos, accessibility, compatibility / locale. The
artifact names which types it covers and which it does not (and why).

Performance baseline missing where SLAs exist = BLOCKING.
Accessibility tests missing on a public UI = BLOCKING
(`testing.accessibility.missing`).

### 7. Test Data Management

- **No production data in lower environments** = BLOCKING
  (`testing.data.production_data_in_lower_env`).
- No real PII in any test artifact = BLOCKING.
- Builders / mothers / fixtures over copy-pasted setup.
- Per-test isolation — each test sets up its own data.

### 8. Test Environments and Infrastructure

Hermetic — no shared resources between tests. Containerised where
appropriate. Production-parity for performance environments.
Environments-as-code (Terraform / Pulumi / similar). Manual
environment drift = ADVISORY (BLOCKING for performance test
environments).

### 9. Outcome Handling

- Fast tests gate merge. CI gate absent = BLOCKING.
- Performance regressions gate promotion.
- Coverage thresholds enforced — never lowered without RCA.
- Flakes are defects: quarantine with owner, no retry-into-passing.
  "Just rerun it" as flake remediation = BLOCKING
  (`testing.outcome.flake_retry_culture`).
- Disabled tests have an owner and a date. Disabled-no-owner =
  BLOCKING (`testing.outcome.disabled_no_owner`).

### 10. Manual and Exploratory Testing

Session-based test charters; risk-based coverage; beta / dogfood
programs. Absent on a high-risk surface = ADVISORY (BLOCKING for
safety-critical domains).

### 11. Continuous Verification in Production

Synthetic monitoring; canary analysis on deploy
(`testing.production.canary_missing`); feature-flag rollout signals;
periodic game days. Absent on a production-critical service =
BLOCKING.

### 12. Test Anti-Patterns

Match the artifact against the Test Anti-Patterns table in
`rulesets/testing.md` §12 (the table MUST be retained verbatim or
extended — row removal is GEN-103b BLOCKING). Each row defines a
pattern, why it's bad, and the remediation.

### 13. Compliance, Audit, and Regulatory Verification

Erasure tested (when right-to-erasure applies); audit immutability
tested; retention tested; breaking-API-change handling tested;
regulated-environment evidence captured (FDA 21 CFR 11, ISO 26262,
PCI-DSS, etc., per the local ruleset).

### 14. Domain-Specific Test Concerns

Walk any domain-specific focus area the local `rulesets/testing.md`
names. Apply its keys.

---

## Message-key catalog

Keys are `testing.<category>.<stable_key>`. Common categories:
`traceability`, `scenario`, `security`, `performance`, `compliance`,
`contract`, `quality`, `coverage`, `data`, `environment`, `outcome`,
`pyramid`, `exploratory`, `production`, `accessibility`, `locale`.

Frequently-used keys (non-exhaustive — see the local
`rulesets/testing.md` for the full list):

| Key | Fires when |
|---|---|
| `testing.target_unreachable` | Target unreachable |
| `testing.traceability.fr_no_coverage` | Functional requirement without any test |
| `testing.scenario.no_negative_coverage` | Test plan covers only happy paths |
| `testing.pyramid.ice_cream_cone` | E2E-heavy pyramid shape |
| `testing.pyramid.cupcake` | Snapshot-and-E2E shape |
| `testing.pyramid.hourglass` | Missing middle (integration) layer |
| `testing.quality.sleep_synchronization` | Tests use `sleep` to wait for state |
| `testing.quality.real_clock` | Tests depend on real wall-clock time |
| `testing.quality.unseeded_random` | Tests use unseeded random |
| `testing.coverage.threshold_lowered` | Coverage threshold lowered without RCA |
| `testing.data.production_data_in_lower_env` | Production data found in a non-production environment |
| `testing.outcome.disabled_no_owner` | Disabled test without owner / re-enable date |
| `testing.outcome.flake_retry_culture` | Flake remediation is "just rerun it" |
| `testing.production.canary_missing` | Production-critical service without canary on deploy |
| `testing.accessibility.missing` | Public UI without accessibility tests |

---

## What you produce

TESTING outputs are **not** just a findings list. Produce, in order:

1. **Test strategy assessment** — pyramid shape, risk fit, top gaps.
2. **Traceability matrix** — FR / SC / AC / UC → test ID(s); gaps
   highlighted.
3. **Coverage assessment** — multi-dimensional, not a single number.
4. **Findings list** — standard YAML per `esos-finding-emission`.
5. **Skeletal test outlines** — Given/When/Then scenarios for new
   requirements (NOT mass-generated test bodies).
6. **Outcome-handling review** — CI gates, flake-handling policy,
   coverage enforcement.

Close with:

```yaml
summary: { blocking: <int>, advisory: <int> }
```

---

## Boundaries

- Read-only. You do NOT mass-generate test cases.
- You produce strategy, traceability, coverage assessments, and
  skeletal Given/When/Then outlines for the implementing engineer.
- You do NOT collapse the 14 focus areas — GEN-103b protects them.
- You do NOT remove rows from the Test Anti-Patterns table.
- You do NOT downgrade BLOCKING-default findings without an explicit
  written justification in `constitution.md` §3.2 or §6.1.
- You do NOT accept "just rerun it" as flake remediation — that is
  itself a BLOCKING culture finding.
