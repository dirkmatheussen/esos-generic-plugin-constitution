# Building a Domain-Specific ESOS Constitution — End User Manual

> **Audience**: a governance owner, enterprise architect, solution architect, or platform owner  
> who needs to create a new domain-specific ESOS constitution (e.g. for healthcare,  
> fintech, automotive, public sector, AI/ML) from this generic base.
>
> **Goal**: take you from zero to a published, validated, Claude-native domain
> constitution.
>
> **Time**: 1–3 hours for a first derivation, depending on how well you've already
> thought through your domain's regulations and risks.

---

## Table of Contents

1. [What you'll build](#1-what-youll-build)
2. [Prerequisites](#2-prerequisites) — includes [§2.4 installing the generic base as a Claude Code plugin](#24-installing-the-generic-base-as-a-claude-code-plugin)
3. [Quick start (5 minutes)](#3-quick-start-5-minutes)
4. [Full walkthrough](#4-full-walkthrough)
5. [The derivation brief — every field explained](#5-the-derivation-brief--every-field-explained)
6. [Customization markers](#6-customization-markers-tokens-and-blocks)
7. [Working with the agent files](#7-working-with-the-agent-files)
8. [Validating the result](#8-validating-the-result)
9. [Publishing](#9-publishing) — includes [§9.4 installing the derived plugin in Claude Code](#94-installing-the-derived-plugin-in-claude-code)
10. [Maintaining a constitution over time](#10-maintaining-a-constitution-over-time)
11. [Troubleshooting](#11-troubleshooting)
12. [FAQ](#12-faq)
13. [Glossary](#13-glossary)

---

## 1. What you'll build

A **domain constitution** is a small repository of Markdown files (≈15 files,
2–4k lines) that tells ESOS how to validate specifications and repositories under
your domain. It's not application code — it's governance-as-code.

When you finish, you'll have a directory like this, sitting next to
`generic-plugin-constitution/`:

```
automotive-spare-parts/                # the derived plugin (installable as-is)
├── plugin.json                        # manifest
├── constitution.md                    # the root + esosAgentCatalog
├── README.md
├── CLAUDE.md
├── CHANGELOG.md
├── VALIDATION.md
├── rulesets/                          # domain-specific specialist rulesets
├── domain/                            # glossary, governance, tech-stack
├── shared/                            # RFC 2119, security baseline, GWT
├── agents/                            # Claude Code subagent shells (slim)
└── skills/                            # common + per-role review + validate workflow
```

This directory is:

- **Versioned in Git** alongside (or near) the code repositories it governs.
- **Published** to your organization's ESOS constitution catalog as version 1.
- **Attached** by ESOS workspaces that want to validate their specifications under
this domain's rules.
- **Audited** automatically by ESOS specialist agents (ANALYST, SECURITY, COMPLIANCE,
CODING, TESTING) every time a specification is validated.
- **Re-auditable** by you any time using the `esos-validate-constitution` skill.

---

## 2. Prerequisites

### 2.1 Tooling

- **Git** (for versioning and review).
- **Claude Code** (recommended) — the CLI, IDE extension, or web app.
  - [https://claude.com/claude-code](https://claude.com/claude-code)
  - The generic base ships with skills and subagents that Claude Code activates
  automatically.
- **A text editor** for hand-editing if needed.

If you don't have Claude Code, you can still derive manually using PROMPT.md as a
prompt template with another LLM, or by editing the base files yourself. Claude
Code is faster and catches more derivation defects.

### 2.2 Knowledge you should already have

Before you start, you should know:

- **Your domain** — what it does, who its users are, what's at stake when something
goes wrong.
- **Applicable regulations** — names AND specific articles where possible (not
"GDPR" but "GDPR Art. 5(1)(c) data minimisation, Art. 17 erasure").
- **Data sensitivities** — what kinds of data your domain processes (PII, PHI,
financial, classified, safety-relevant).
- **Top 3–7 domain risks** — what would make stakeholders unhappy / a regulator
call.
- **Your team's tech stack** — language, framework, database, identity provider,
CI/CD platform.

If any of these are unclear, **gather that input first**. The derivation tooling
will not invent these for you, and rightly so — getting them wrong leads to a
constitution that nominally passes validation but doesn't actually fit the domain.

### 2.3 Files you should have at hand

- The Generic Constitution (this repository). It is the foundational document — no
  document sits above it.

### 2.4 Installing the generic base as a Claude Code plugin

The repository ships as a Claude Code plugin source — `plugin.json` is at the
root and `.claude-plugin/marketplace.json` wraps it as a single-plugin
marketplace. Installing it gives you the five subagents (`esos-analyst`,
`esos-security`, `esos-compliance`, `esos-coding`, `esos-testing`), the three
common skills, the five per-role review skills, and both workflow skills
(`esos-create-constitution`, `esos-validate-constitution`) — all from one
command.

Pick one of three install paths:

#### Option A — install from a Git URL (recommended for teams)

```text
/plugin marketplace add dirkmatheussen/esos-generic-plugin-constitution
/plugin install esos-generic-plugin-constitution@esos-generic-plugin-constitution
```

The first command adds the repository as a marketplace (Claude Code reads
`.claude-plugin/marketplace.json` to learn what plugins it offers). The second
installs the plugin from that marketplace. Pin to a tag or commit for
production use:

```text
/plugin marketplace add dirkmatheussen/esos-generic-plugin-constitution@v3.1.0
```

For non-GitHub Git hosts, supply the full HTTPS URL (note the `.git` suffix —
it tells Claude Code to clone rather than treat the URL as a marketplace
manifest):

```text
/plugin marketplace add https://gitlab.example.com/platform/esos.git#v3.1.0
```

#### Option B — install from a local path (recommended while iterating)

If you've cloned the repo locally and want to use it before publishing:

```text
/plugin marketplace add /absolute/path/to/generic-plugin-constitution
/plugin install esos-generic-plugin-constitution@esos-generic-plugin-constitution
```

Or, for ad-hoc local testing without registering a marketplace, launch Claude
Code with the plugin loaded directly:

```bash
claude --plugin-dir /absolute/path/to/generic-plugin-constitution
```

#### Option C — declarative install via `settings.json` (recommended for shared team configs)

Add the following to `~/.claude/settings.json` (user scope) or
`.claude/settings.json` (project scope, checked into Git so the whole team
inherits the same plugin set):

```json
{
  "extraKnownMarketplaces": {
    "esos-generic-plugin-constitution": {
      "source": {
        "source": "github",
        "repo": "dirkmatheussen/esos-generic-plugin-constitution"
      }
    }
  },
  "enabledPlugins": {
    "esos-generic-plugin-constitution@esos-generic-plugin-constitution": true
  }
}
```

Restart Claude Code to apply.

#### Confirm the install

After install, in any Claude Code session run `/plugin` to list active plugins
and confirm `esos-generic-plugin-constitution` is present. Test that the
skills loaded by asking *"Create an ESOS constitution for a test domain"* —
the `esos-create-constitution` skill should auto-trigger.

> **Heads-up — naming**: in the recommended install command the plugin name
> and the marketplace name are both `esos-generic-plugin-constitution` (the
> marketplace wraps one plugin and reuses the name for unambiguity). Users
> MAY re-alias the marketplace on their side; the canonical name in
> `.claude-plugin/marketplace.json` is the one Claude Code displays.

---

## 3. Quick start (5 minutes)

If you already know your domain well and just want to see the workflow:

1. **Open `generic-plugin-constitution/` in Claude Code.**
2. **Tell Claude**: "Create a new ESOS constitution for `<your domain>`. Risk
  profile: `<low|medium|high|safety-critical>`. Severity tier:
   `<strict|normal|relaxed>`. Applicable regulations: `<list>`. Tech stack:
   `<list>`. Store the result in `<directory>`"
3. Claude triggers the `esos-create-constitution` skill, asks for any missing
  inputs, and produces the derived directory.
4. **Review the output**, then say "Validate it."
5. Claude triggers `esos-validate-constitution` and produces a findings report
  scoped to your tier. Fix any BLOCKING items.
6. **Commit** the new directory to Git and publish to the ESOS catalog.

That's the happy path. The rest of the manual covers what to do when reality
doesn't fit on one page.

If you don't know what tier to pick, omit it — the derivation defaults to
`strict` (maximum enforcement). See [§5.12](#512-severity_tier-strict--normal--relaxed)
for how to choose.

---

## 4. Full walkthrough

### 4.1 Step 1 — Understand the inheritance chain

Two layers govern your constitution. The Generic Constitution is **foundational** —
it is the floor; nothing sits above it.

```
Generic Plugin Constitution Base v3.1.0     ← the floor: 5 specialist classes, pipeline
        │                              ordering, mandatory section minimum, audit
        │                              and traceability rules, severity floor,
        │                              CODING agent as senior dev/architect (14 areas),
        │                              TESTING agent as senior tester/QA (14 areas)
        ▼
{{ Your Domain }} Constitution       ← your work: domain-specific glossary, regulations,
                                       tech stack, risks, severity overrides
```

Two important rules:

- You **MAY tighten** anything from the base (raise a threshold, add a mandatory
section, upgrade an ADVISORY finding to BLOCKING).
- You **MUST NOT relax** anything this constitution flags as a `MUST` / `MUST NOT`.
If you try, validation will block publication with a clear remediation message.

Within those bounds, your derivation declares a **severity tier** (`strict`,
`normal`, or `relaxed`) that calibrates how strictly the generic baseline checks
gate publication. The tier is recorded in `constitution.md` §8 and read by the
validator before any check runs. Foundational-floor rules are unaffected by the
tier — they are BLOCKING at every tier. See
[§5.12](#512-severity_tier-strict--normal--relaxed) for how to choose.

### 4.2 Step 2 — Gather the derivation brief

The brief is the input the derivation skill takes. **Spend real time on this.**
A weak brief produces a constitution that nominally validates but does not fit
your domain.

Required fields:


| Field                    | What it is                                                                             | Why it matters                                              |
| ------------------------ | -------------------------------------------------------------------------------------- | ----------------------------------------------------------- |
| `TARGET_DIRECTORY`       | Absolute or workspace-relative path the derived directory will be created under (the skill writes to `<TARGET_DIRECTORY>/<DOMAIN_SLUG>/`) | The create skill MUST stop and ask if this is missing — it does not assume any default. See [§5.1](#51-domain_name-domain_slug-and-target_directory). |
| `DOMAIN_NAME`            | Human-readable name (e.g. "Automotive Spare Parts")                                    | Used in the title of every file.                            |
| `DOMAIN_SLUG`            | kebab-case directory name (e.g. `automotive-spare-parts`)                              | The directory and a few file paths.                         |
| `DOMAIN_SUMMARY`         | 2–4 sentences describing the domain                                                    | Drives the README and constitution.md §1.1 framing.         |
| `DATA_SENSITIVITIES`     | Classes of sensitive data in scope                                                     | Drives SECURITY agent's classification rules.               |
| `APPLICABLE_REGULATIONS` | Specific standards/articles                                                            | Drives COMPLIANCE agent. **Cite articles, not just names.** |
| `KEY_DOMAIN_TERMS`       | Terms with concise definitions. Minimum: ≥10 (`strict`), ≥5 (`normal`), ≥1 (`relaxed`) | Becomes the domain glossary.                                |
| `RISK_PROFILE`           | low / medium / high / safety-critical                                                  | Drives confidence threshold.                                |


Optional but **strongly recommended**:


| Field                      | What it is                                                             | Default if omitted                                                 |
| -------------------------- | ---------------------------------------------------------------------- | ------------------------------------------------------------------ |
| `SEVERITY_TIER`            | `strict` / `normal` / `relaxed` — calibrates baseline-check strictness | `strict` (see [§5.12](#512-severity_tier-strict--normal--relaxed)) |
| `JURISDICTIONS`            | Legal jurisdictions in scope                                           | Flagged in `assumptions` for review.                               |
| `TECH_STACK_PREFERENCES`   | Languages, frameworks, identity, etc.                                  | "Use the team's standard stack" placeholders.                      |
| `ARCHITECTURE_STYLE`       | Monolith / microservices / event-driven                                | Generic guidance retained.                                         |
| `DOMAIN_SPECIFIC_RISKS`    | 3–7 unique risks                                                       | Generic risks only — much weaker.                                  |
| `EXTRA_MANDATORY_SECTIONS` | E.g. `recall_traceability`, `supplier_integration`, `pricing_controls` | Foundational minimum only.                                         |
| `SPECIALIST_WAIVERS`       | Classes to skip (rare; usually empty)                                  | None waived.                                                       |
| `EXTERNAL_LINK_ALLOWLIST`  | Allowed schemes/hosts                                                  | `https` only, no host allowlist.                                   |
| `DOMAIN_TAGS`              | Catalog tags for search                                                | Empty (poor catalog hygiene).                                      |
| `DOMAIN_LOGICAL_ID_UUID`   | UUID for stable identity across versions                               | Auto-generated.                                                    |


**A worked example brief** (automotive spare parts supply chain):

```yaml
TARGET_DIRECTORY: "../"   # write to <repo-parent>/automotive-spare-parts/
DOMAIN_NAME: "Automotive Spare Parts"
DOMAIN_SLUG: "automotive-spare-parts"
DOMAIN_SUMMARY: |
  A B2B platform for ordering, distributing, and tracking OEM and aftermarket
  automotive spare parts across a multi-tier supply chain. Users include OEM
  parts managers, tier-1 and tier-2 suppliers, central and regional distribution centres,
  franchise and independent dealers, and end-customer repair shops. Errors
  propagate into vehicle safety (wrong part fitted), commercial exposure
  (counterfeit parts, cross-region pricing collusion), and regulatory exposure
  (recall traceability, EU right-to-repair access for independent operators).

DATA_SENSITIVITIES:
  - PII (dealer staff accounts; end-customer accounts on the consumer-facing portal)
  - VIN (Vehicle Identification Number — links to vehicle ownership; treated as personal data when associated with an identifiable owner)
  - Supplier contract terms (commercially sensitive)
  - Pricing data (commercially sensitive; anti-trust exposure if leaked across regions)
  - Warranty and recall data (regulator-facing)
  - Safety-critical part identifiers (ECU firmware references, brake assemblies, airbag inflators, restraint systems)

APPLICABLE_REGULATIONS:
  - "GDPR Art. 5(1)(c) — data minimisation (limit VIN and PII exposure to authorised principals)"
  - "GDPR Art. 17 — right to erasure (with legal-retention exemption per Art. 17(3)(b) for warranty and recall records)"
  - "GDPR Art. 32 — security of processing"
  - "UK GDPR / DPA 2018 — equivalent obligations in the UK"
  - "EU Regulation 2018/858 (Type-Approval) Art. 61 — vehicle repair and maintenance information access for independent operators"
  - "EU Block Exemption Regulation 461/2010 — competition rules for motor vehicle aftermarket"
  - "ISO 26262 — functional safety, Part 8 §6 supporting processes for safety-related parts"
  - "PCI-DSS v4.0 — where the platform processes card payments"
  - "SOX §404 — internal financial controls over pricing and revenue flows (listed-company scope)"
  - "EU Article 101 TFEU — anti-trust (no price collusion across regions or tiers)"

KEY_DOMAIN_TERMS:
  - { term: "Original Equipment Manufacturer", abbr: "OEM", def: "The vehicle manufacturer or its captive parts arm; the authoritative source for OEM part numbers." }
  - { term: "Aftermarket Part", abbr: "—", def: "A part manufactured by a third party (not the OEM) for the same fitment; subject to different warranty and liability rules." }
  - { term: "Vehicle Identification Number", abbr: "VIN", def: "17-character identifier of an individual vehicle; personal data when linked to an owner." }
  - { term: "Electronic Control Unit", abbr: "ECU", def: "Embedded controller in the vehicle; ECU parts are safety-related per ISO 26262." }
  - { term: "Supersession", abbr: "—", def: "When one part number is replaced by another; orders for the superseded number MUST route to the successor." }
  - { term: "Bill of Materials", abbr: "BoM", def: "Structured parts list for a vehicle assembly or sub-assembly." }
  - { term: "Stock Keeping Unit", abbr: "SKU", def: "Inventory identifier; a single part number may map to several SKUs across packaging or region." }
  - { term: "Tier-1 Supplier", abbr: "—", def: "A direct supplier to the OEM; tier-2 and below sit further upstream." }
  - { term: "Cross-Reference", abbr: "—", def: "Mapping between OEM and aftermarket part numbers for the same fitment." }
  - { term: "Recall Campaign", abbr: "—", def: "Regulator-driven action to repair or replace a defective part across a population of vehicles." }
  - { term: "Warranty Claim", abbr: "—", def: "Claim against the OEM or supplier under the original-equipment warranty regime." }
  - { term: "Right to Repair", abbr: "—", def: "EU regulatory right of independent operators to access OEM repair and parts data under Regulation 2018/858 Art. 61." }
  - { term: "Safety-Critical Part", abbr: "—", def: "A part whose failure can directly cause injury (brakes, airbags, steering, restraint systems, certain ECUs); requires ISO 26262 process evidence." }
  - { term: "Counterfeit Part", abbr: "—", def: "A part falsely presented as OEM or as a certified aftermarket equivalent; a primary brand and safety risk." }

RISK_PROFILE: medium

SEVERITY_TIER: relaxed   # example only — a real spare-parts catalog would often choose `strict` given safety + pricing + recall exposure

DOMAIN_SPECIFIC_RISKS:
  - "Counterfeit parts entering the supply chain (consumer safety + brand integrity)"
  - "Recall-traceability gaps (cannot trace which vehicles received a recalled part)"
  - "Supersession errors leading to wrong part fitted (safety + warranty exposure)"
  - "VIN exposure outside authenticated, authorised scope (privacy + vehicle-history disclosure)"
  - "Anti-trust exposure from cross-region pricing data leakage or collusion patterns"
  - "Right-to-repair non-compliance (denial of parts data to independent operators)"
  - "Single-source supplier outage for safety-critical parts (availability + customer impact)"

JURISDICTIONS: [EU, UK, US]

TECH_STACK_PREFERENCES:
  backend_lang: Java 21
  backend_framework: Spring Boot 3.x
  frontend: TypeScript + React (dealer portal); React Native (mobile parts-scan app)
  database: PostgreSQL 16 (managed); Oracle 19c (read-only legacy DMS integration)
  cache: Redis (managed)
  messaging: Kafka (internal event bus); AS2 (EDI gateway for tier-1 supplier feeds)
  identity: Keycloak (B2B federation for dealers and suppliers); Auth0 (consumer-facing portal)
  secret_store: HashiCorp Vault
  iac: Terraform
  ci_cd: GitLab CI
  observability: OpenTelemetry → Datadog

ARCHITECTURE_STYLE: "event-driven microservices, BFF for the dealer portal, EDI gateway for legacy tier-1 supplier integration"

EXTRA_MANDATORY_SECTIONS:
  - recall_traceability
  - supplier_integration
  - pricing_controls

EXTERNAL_LINK_ALLOWLIST:
  schemes: [https]
  hosts: [confluence.example-auto.com, sharepoint.example-tenant.com, supplier-portal.example-tier1.com]

DOMAIN_TAGS: [automotive, spare-parts, supply-chain, b2b, oem, aftermarket]
```

**Save your brief** to a scratch file (e.g. `/tmp/esos-brief-automotive-spare-parts.md`)
so the work is resumable if the session is interrupted.

### 4.3 Step 3 — Invoke the create skill

In Claude Code, with `generic-plugin-constitution/` open:

**Option A** — natural language:

> "Create an ESOS domain constitution for the automotive spare parts supply
> chain. Here's the brief: paste brief"

The `esos-create-constitution` skill auto-triggers on phrases like "create a
constitution for…".

**Option B** — explicit:

> `/esos-create-constitution`

Claude will then read the skill instructions, ask for the brief if you haven't
pasted it, and walk through the production rules in `PROMPT.md`.

The skill will:

1. Validate the brief; ask for any missing required field — including
   `TARGET_DIRECTORY` (the skill does NOT default to a sibling of
   `generic-plugin-constitution/`; it MUST be told where to write).
2. Read every base file (`constitution.md`, all five `agents/*.md`, all three
  `domain/*.md`, all three `shared/*.md`, `CLAUDE.md`, the validate skill, all
   five subagents).
3. Confirm the resolved final path `<TARGET_DIRECTORY>/<DOMAIN_SLUG>/` back to
   you. If that path already exists and is non-empty, the skill stops and asks
   before doing anything destructive.
4. Produce the derived directory at the confirmed `<TARGET_DIRECTORY>/<DOMAIN_SLUG>/`.
5. Self-check before declaring done.
6. Suggest the next step (validate).

### 4.4 Step 4 — Review the output

Open the derived directory. Skim each file:

- `**constitution.md`** — confirm the catalog metadata, mandatory sections, pipeline,
thresholds, and external link policy reflect your brief.
- `**agents/*.md`** — confirm each specialist's domain-specific section names your
actual risks, regulations, and tech.
- `**domain/glossary.md`** — confirm every term from `KEY_DOMAIN_TERMS` is there.
- `**domain/governance-policies.md`** — confirm every regulation in
`APPLICABLE_REGULATIONS` is referenced with specific articles.
- `**domain/tech-stack.md`** — confirm the stack matches your brief; no `{{ TOKEN }}`
remains.
- `**CLAUDE.md**` — confirm it names your domain and references the right files.
- `**agents/esos-*.md**` — confirm each subagent's description mentions your
domain name.

If anything is generic-feeling or off, **iterate**. Tell Claude specifically what
to tighten, e.g. "Make `rulesets/security.md` §10 mention VIN exposure outside
authenticated, authorised scope explicitly as a BLOCKING finding when an API
response returns an unmasked VIN to a principal without the
`parts:lookup:vin` privilege."

### 4.5 Step 5 — Validate

Once the output looks reasonable:

> "Validate the automotive-spare-parts constitution."

The `esos-validate-constitution` skill auto-triggers, **resolves the severity tier**
from your candidate's `constitution.md` §8 (defaulting to `strict` if absent, with a
GEN-120 ADVISORY), and walks every check in `VALIDATION.md` (Sections A–D, GEN-001
to GEN-306, plus GEN-103a/b for role framings, GEN-116/117/118/119 for the Claude
toolkit, and GEN-120/121 for the tier declaration itself).

The output is a structured findings report:

```yaml
findings:
  - check_id: GEN-108
    severity: BLOCKING
    location: { file: "domain/governance-policies.md", section: "11. Domain-Specific Regulations" }
    message: "EU Regulation 2018/858 Art. 61 (right to repair) cited but no specific control implementing it is described."
    message_key: "compliance.right_to_repair.control_missing"
    remediation: |
      Add the concrete control: which API exposes repair and parts data to
      verified independent operators, what authentication regime gates it, and
      how the audit log proves access was granted to an operator with valid
      credentials.
  - check_id: GEN-109
    severity: BLOCKING
    location: { file: "rulesets/security.md", section: "10. Domain-Specific Security Concerns" }
    message: "Brief listed 'counterfeit parts entering the supply chain' as a domain risk, but no agent file surfaces it."
    message_key: "constitution.domain_risk_unaddressed"
    remediation: |
      Add a focus area in rulesets/security.md (or domain/governance-policies.md)
      that names the counterfeit-parts risk and the concrete checks the platform
      performs: supplier provenance attestation, serial-number validation against
      the OEM master, and anomaly detection on high-volume aftermarket inflows.
  - check_id: GEN-117
    severity: BLOCKING
    location: { file: "agents/esos-coding.md", section: "frontmatter description" }
    message: "Subagent description does not mention the domain (Automotive Spare Parts)."
    message_key: "constitution.subagent_description_no_domain"
    remediation: "Edit the description to reference the domain so auto-delegation routes correctly."
  ...

summary:
  severity_tier: strict
  tier_source: declared          # declared | defaulted | brief
  total_blocking: 3
  total_advisory: 5
  total_informational: 0          # Section D findings when tier is relaxed
  publishable: false
  files_audited: [...]
  checks_executed: 49
```

The candidate is **publishable only if `total_blocking == 0`**. The `severity_tier`
field records the tier under which publishability was assessed — re-auditing at a
stricter tier may surface additional BLOCKING findings.

### 4.6 Step 6 — Iterate on findings

For each BLOCKING finding:

- If the fix is mechanical (replace a token, add a missing reference, fix a header),
apply the fix directly.
- If the fix requires domain judgment (which control implements Right-to-Repair
Art. 61? what's the concrete check for counterfeit-part detection in the
supplier feed?), **decide and write it**. Do not let Claude invent — those are
your domain calls.

Re-run the validate skill until `total_blocking == 0`.

ADVISORY findings are recorded but do not block publication. Address them when
they reflect genuine improvements; ignore them when the brief explicitly justifies
the deviation.

### 4.7 Step 7 — Commit and publish

```bash
cd ../<DOMAIN_SLUG>
git init
git add .
git commit -m "Initial automotive-spare-parts ESOS constitution v1.0.0"
git remote add origin <your remote>
git push -u origin main
```

Then publish to the ESOS catalog:

- Through the ESOS UI: navigate to Constitutions → Upload, attach the repository
pointer (commit SHA preferred over branch — see `constitution.md` §9), and
populate the catalog row using the `constitution_catalog_row` block in your
`constitution.md` §8.
- Or via your organization's MCP / API integration if one exists.

Workspaces can now attach to your constitution.

---

## 5. The derivation brief — every field explained

This section expands §4.2 with rationale and worked examples per field.

### 5.1 `DOMAIN_NAME`, `DOMAIN_SLUG`, and `TARGET_DIRECTORY`

These three fields jointly determine the output location of the derived
constitution. The create skill enforces all three before writing anything.

- `DOMAIN_NAME` is human-readable, title case. It appears in headers, the catalog
display name, and the README title.
- `DOMAIN_SLUG` is kebab-case. It is the directory name and is referenced in
paths.
- `TARGET_DIRECTORY` is the **parent path** under which the derivation directory
will be created. It MAY be absolute (`/Users/me/work/constitutions/`) or
workspace-relative (`../`, `./derivations/`). The skill writes to
`<TARGET_DIRECTORY>/<DOMAIN_SLUG>/`.

**The create skill does NOT default `TARGET_DIRECTORY`.** If you don't supply it,
the skill stops and asks. Previous versions of this base defaulted to "sibling
of `generic-plugin-constitution/`"; that default has been removed because it silently
created directories in places users did not expect. Confirm the resolved final
path back to the skill before it writes.

If the resolved `<TARGET_DIRECTORY>/<DOMAIN_SLUG>/` already exists and is
non-empty, the skill stops and asks whether to abort, choose a different
location, or overwrite — never overwrite silently.

**Naming guidance**: name the *domain*, not the *team*. "Automotive Spare Parts"
is durable; "Acme Parts Platform Team v3" rots. If the constitution might apply
to multiple products, name the domain at the level all products share.

### 5.2 `DOMAIN_SUMMARY`

2–4 sentences. Cover: what the domain does, who uses it, what's at stake when
something goes wrong. This is what a new joiner reads first — make it concrete.

**Bad** (vague):

> "This is a system for managing things."

**Good** (concrete, scoped, stakes named):

> "A B2B platform for ordering, distributing, and tracking OEM and aftermarket
> automotive spare parts across a multi-tier supply chain. Users include OEM
> parts managers, tier-1 and tier-2 suppliers, regional distribution centres,
> franchise and independent dealers, and end-customer repair shops. Errors
> propagate into vehicle safety, commercial exposure, and regulatory exposure."

### 5.3 `DATA_SENSITIVITIES`

List every class of sensitive data your domain processes. Be specific: "PII" alone
is weak; "PII (customer name, email, billing address)" is useful.

Common classes: PII, PHI, payment-card data, financial transactions,
authentication credentials, health data, biometric, genetic, location, behavioral,
classified (Official / Secret), trade secret, supplier-contract terms.

### 5.4 `APPLICABLE_REGULATIONS`

The single biggest source of weak constitutions is vague regulation citation.
**Cite specific articles or sections.**

**Weak**: "GDPR-compliant"
**Strong**: "GDPR Art. 5(1)(c) data minimization; Art. 17 right to erasure (with
research exemption per Art. 17(3)(c)); Art. 32 security of processing"

If you don't know the specific articles, find them before deriving — your
COMPLIANCE specialist will need them and the validator will flag vague references
as ADVISORY (and missing ones as BLOCKING).

### 5.5 `KEY_DOMAIN_TERMS` (≥10)

The glossary is the foundation for terminology consistency across every
specification under your constitution. Get it right.

For each term: name, optional abbreviation, concise definition (1–2 sentences).
Define every domain term **once** in the glossary; other files reference it.

### 5.6 `RISK_PROFILE`

Four options: `low`, `medium`, `high`, `safety-critical`. The field calibrates
the **confidence threshold** for every specialist step (see `constitution.md`
§4.1) and threads through several conventions for the rest of the brief.

#### Direct impact — confidence threshold


| Profile           | Suggested threshold | Validator floor                                                                                  |
| ----------------- | ------------------- | ------------------------------------------------------------------------------------------------ |
| `low`             | 0.70                | Foundational floor: threshold MUST be ≥ 0.                                                       |
| `medium`          | 0.75                | Foundational floor: threshold MUST be ≥ 0.                                                       |
| `high`            | 0.80                | Foundational floor: threshold MUST be ≥ 0.                                                       |
| `safety-critical` | **≥ 0.85**          | **Hard floor — `safety-critical` with threshold < 0.85 is BLOCKING (`VALIDATION.md` GEN-007).**  |


**What the threshold does** (per `constitution.md` §4 keep-block): when a
specialist's output confidence falls below the configured threshold, ESOS:

1. Emits a `LOW_CONFIDENCE` trace event (score + threshold).
2. Runs allowed corrective paths — **rework** (up to 3 attempts per `constitution.md` §4),
   **escalate** to another specialist, or **pause and request additional review**.
3. Does NOT mark the run successful until thresholds are met or the run is
   explicitly stopped.

Higher profile → higher threshold → more reruns and escalations when an agent
isn't sufficiently sure. That is the only *automatic* behavioural difference
between profiles.

#### Convention-level (indirect) impact

These aren't enforced by the validator but are how risk profile typically
threads through the rest of the brief and the derived constitution:


| Decision                                                            | `low`              | `medium`          | `high`               | `safety-critical`                                              |
| ------------------------------------------------------------------- | ------------------ | ----------------- | -------------------- | -------------------------------------------------------------- |
| Typical [`SEVERITY_TIER`](#512-severity_tier-strict--normal--relaxed) | `relaxed` / `normal` | `normal`          | `normal` / `strict`  | `strict`                                                       |
| Extra mandatory sections                                            | none usually       | a couple          | several              | `safety_case` typically required (ISO 26262, IEC 62304, etc.)  |
| Regulations expected                                                | few                | privacy + sector  | privacy + financial / sector-heavy | safety + privacy + sector + ISO/IEC functional safety           |
| Independence of review                                              | team review        | + security        | + compliance         | + safety-case authority or regulator-facing review             |


`RISK_PROFILE` and `SEVERITY_TIER` are independent fields, but in practice the
two track together — decide them in the same conversation, not in isolation.

#### One-way ratchet

When in doubt, choose higher. Tightening `RISK_PROFILE` (and the threshold it
implies) later is straightforward; relaxing it under pressure during an incident
is much harder. Discovering after launch that you process regulated data and
need to retroactively raise the bar is a painful pivot.

#### What `RISK_PROFILE` does NOT do

- It does **not** change which checks fire — that is `SEVERITY_TIER`'s job
  (§5.12).
- It does **not** change the mandatory section keys — the foundational §2 floor is
  the same at every profile.
- It does **not** change pipeline order or the five specialist classes.
- It does **not** waive any specialist — waivers are a separate
  `SPECIALIST_WAIVERS` field with their own written rationale and
  `CONSTITUTION_WAIVER` trace requirement (§5.9).

### 5.7 `DOMAIN_SPECIFIC_RISKS`

3–7 risks unique to your domain. Every risk you list will be enforced by the
validator: it MUST surface in at least one agent file or shared baseline.
Skipping this field gives you a generic constitution that misses the point.

### 5.8 `EXTRA_MANDATORY_SECTIONS`

Additional sections beyond the foundational minimum. Examples:

- `safety_case` — for safety-critical software (ISO 26262, IEC 62304).
- `clinical_workflow` — for healthcare.
- `regulatory_filings` — for regulated finance.
- `accessibility_conformance` — for public sector / WCAG.
- `model_governance` — for AI/ML systems.
- `recall_traceability` — for automotive / consumer goods.

Adding a section means every spec under your constitution MUST contain it.
Don't add what you can't enforce.

### 5.9 `SPECIALIST_WAIVERS`

Almost always empty. Waiving a specialist is a strong signal that the constitution
is incomplete — this constitution allows it but treats waivers as suspect.

If you waive a specialist (e.g. SECURITY for an entirely public-data domain), you
MUST:

- Document the rationale in `constitution.md` §3.2.
- Reference the `CONSTITUTION_WAIVER` trace event ESOS will emit.
- Never waive ANALYST or VALIDATION (those are structural).

### 5.10 `EXTERNAL_LINK_ALLOWLIST`

Hosts and schemes permitted in constitution Markdown and specifications. Defaults
to `https` only with no host allowlist (which means external links are flagged).

If your team uses Confluence, SharePoint, Notion, etc., list the hosts. The link
policy is enforced at constitution publish time per `constitution.md` §5.

### 5.11 `DOMAIN_LOGICAL_ID_UUID`

The UUID identifies your constitution across versions. Version 2 of the same
constitution shares the logical_id with version 1; only the integer `version`
field increments. If you don't supply one, generate one (`uuidgen` or any RFC
4122 generator).

### 5.12 `SEVERITY_TIER` (`strict` / `normal` / `relaxed`)

The tier calibrates how strictly the generic baseline checks gate publication.
It does **not** relax any foundational-floor rule — Section A of `VALIDATION.md`
(inheritance, all five specialists wired, mandatory section keys, pipeline
ordering, no-secrets, etc.) is BLOCKING at every tier. The tier only affects
the discretionary baseline checks in Section B and C, plus the heuristic
Section D.


| Tier      | What demotes to ADVISORY relative to `strict`                                                                                                                                                                        | Section D heuristics       | Glossary minimum |
| --------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------- | ---------------- |
| `strict`  | Nothing — maximum enforcement. All baseline defaults are BLOCKING.                                                                                                                                                   | ADVISORY                   | ≥10 domain terms |
| `normal`  | Tech-stack file is generic; severity-label inconsistencies; i18n undeclared when UI in scope.                                                                                                                        | ADVISORY                   | ≥5 domain terms  |
| `relaxed` | All of `normal` plus: regulations cited without articles; domain risk not surfaced; GWT not mandated; CLAUDE.md / subagents / validate-skill description without domain mention; tech-stack alignment across agents. | informational, not counted | ≥1 domain term   |


**File presence is NEVER tier-aware.** Missing `CLAUDE.md`, a missing subagent
file, a missing validate skill, or a missing required directory remains BLOCKING
at every tier. The tier only relaxes how the *content* of those files is
audited.

**Always BLOCKING at every tier** (foundational floor and architectural protections):

- All Section A checks (inheritance, 5 specialists wired, mandatory section keys,
pipeline ordering, no secrets, no unreplaced tokens, `<!-- esos:keep -->`
preservation, …).
- The CODING agent's senior dev / software architect role and 14 focus areas
(GEN-103a).
- The TESTING agent's senior tester / QA architect role and 14 focus areas
(GEN-103b).
- RFC 2119 normative language; severity floor in agent files; agent severity
tables non-contradictory; `mandatory_section_keys` consistency.

**How to choose a tier**:


| If your domain is …                                                                                          | Pick      |
| ------------------------------------------------------------------------------------------------------------ | --------- |
| Regulated, safety-critical, or life-impacting (healthcare, automotive ECU, aviation, finance with custody)   | `strict`  |
| Mainstream business — customer-facing product, B2B SaaS, e-commerce — rigour expected but not life-impacting | `normal`  |
| Internal tooling, prototype, dev-productivity, low-blast-radius back-office where time-to-first-spec matters | `relaxed` |


**Semver implication of changing tiers**:

- Tightening (`relaxed → normal`, `normal → strict`) is a **MINOR** bump — you
are strengthening the contract; downstream workspaces benefit.
- Loosening (`strict → normal`, `normal → relaxed`) is a **MAJOR** bump — you
are weakening a published contract that downstream workspaces depend on.

If you omit `SEVERITY_TIER` from the brief, the derivation skill defaults to
`strict` and records the defaulting in the derivation summary. Pick deliberately
rather than letting the default decide for you.

---

## 6. Customization markers (tokens and blocks)

Three markers govern what derivations may change.

### 6.1 `{{ TOKEN }}` — inline placeholder

Every occurrence MUST be replaced. Examples in the base:

- `{{ DOMAIN_NAME }}` → "Automotive Spare Parts"
- `{{ DEFAULT_THRESHOLD }}` → "0.80"
- `{{ DOMAIN_LOGICAL_ID_UUID }}` → an actual UUID
- `{{ TECH_STACK_PREFERENCES }}` → expanded into the actual stack table

If a token has no sensible value for your domain, replace it with an explicit
statement, not silence. E.g.

> `{{ DOCUMENT_DB }}` → `Not in scope — this domain uses only a relational store.`

### 6.2 `<!-- esos:domain-customization -->` — domain-rewrite block

The block following the marker MUST be rewritten with domain content. The base
provides a generic example or placeholder; your job is to replace it with
domain-specific material.

Example (in `rulesets/security.md` §10):

```markdown
<!-- esos:domain-customization -->

The derived constitution lists domain-specific security concerns here. Examples:
[generic examples]

<!-- /esos:domain-customization -->
```

Becomes (for automotive spare parts):

```markdown
<!-- esos:domain-customization -->

VIN access logging hooks MUST capture: principal, VIN (masked in the log payload;
full VIN only in the encrypted audit store), accessed resource type (vehicle
history / parts cross-reference / recall lookup), access reason (dealer service
event / right-to-repair request / recall notification / warranty claim), and UTC
timestamp. Requests that return unmasked VINs to principals lacking
`parts:lookup:vin` MUST be denied at the service layer and tested with the
unauthorised-VIN-exposure negative-test pattern from
`shared/acceptance-criteria-format.md`. Counterfeit-detection signals
(unexpected serial-number ranges, supplier-of-record mismatch) MUST be raised as
events on the `parts.integrity` topic for the supply-chain integrity team to
triage.

<!-- /esos:domain-customization -->
```

### 6.3 `<!-- esos:keep -->` — inherited floor

The block is inherited from the foundational floor. Derivations MUST keep it
verbatim or **tighten** it (raise severity, add requirements). Never relax.

If you find yourself wanting to relax a `keep` block, you've found a foundational
issue — escalate it to base maintenance, don't paper over it in your derivation.

---

## 7. Working with the agent files

Each of the five `agents/*.md` files defines what one ESOS specialist checks. They
are also the system prompts ESOS uses at runtime — so wording matters.

### 7.1 Authoring discipline

- **Use normative language** (RFC 2119: MUST / SHALL / SHOULD / MAY in uppercase).
See `shared/normative-language.md`.
- **Be specific, not aspirational** ("FRs without a normative keyword = BLOCKING",
not "FRs should be testable").
- **Keep severity tables consistent** with the base floor — tighten, never relax.
- **Domain-specific subsections** name actual risks, regulations, and tech — not
generic placeholders.

### 7.2 The CODING agent — senior dev / software architect

The CODING agent in this base is positioned as a senior developer / software
architect. It enforces 14 focus areas:

1. No Shortcuts (highest priority — hardcoded values, magic constants, unreferenced TODOs)
2. Architectural Integrity (SOLID, layered separation, DDD, no god objects)
3. Enterprise Patterns (Repository, Saga, Outbox, Idempotency Key, …)
4. Scalability & Performance (statelessness, N+1, pagination, backpressure)
5. Resilience & Reliability (timeouts, circuit breakers, idempotency)
6. Maintainability & Code Quality (naming, complexity, dependency hygiene)
7. Operability (structured logging, externalized config, no permanent flags)
8. API Design (machine-readable contract, versioning, idempotency, rate limiting)
9. Data Model & Migration Safety (expand-and-contract, indexes, currency precision)
10. Security as Quality (platform identity, server-side authz, secret store)
11. CI/CD & DevSecOps (SAST, dependency scan, secret detection, canary)
12. Stack Alignment (match `domain/tech-stack.md`)
13. Documentation & Decision Records (ADRs)
14. Domain-Specific Quality Concerns

**You MUST keep all 14 focus areas.** Removing any one is BLOCKING (validator
GEN-103a). You MAY tighten any of them.

### 7.3 The TESTING agent — senior tester / QA architect

The TESTING agent enforces 14 focus areas covering test strategy, traceability,
pyramid shape, test quality (no flakiness sources), multi-dimensional coverage,
test types, test data management, environments, outcome handling, exploratory,
production verification, anti-patterns, compliance, and domain-specific concerns.

**You MUST keep all 14 focus areas and the anti-patterns table** (§12). Removing
either is BLOCKING (validator GEN-103b).

---

## 8. Validating the result

Validation is structured and **tier-aware**. The validate skill first resolves
your candidate's `severity_tier` from `constitution.md` §8 (falling back to
`strict` with a GEN-120 ADVISORY if absent), then walks ~50 numbered checks (the
`GEN-NNN` series in `VALIDATION.md`) covering:

- **Section A — Foundational conformance**: inheritance declared, all 5 specialists
wired, mandatory sections present, pipeline ordering correct, no secrets,
no unreplaced tokens. **All BLOCKING at every tier — never relaxed.**
- **Section B — Generic baseline**: severity floor preserved, RFC 2119, GWT,
glossary completeness, governance citations, domain risks surface, threshold
per risk profile, Claude Code toolkit present (CLAUDE.md, 5 subagents,
validate skill), tier declared (GEN-120) and consistent with brief (GEN-121).
Several Section B checks are **tier-aware** — see the matrix in
`VALIDATION.md` and the summary in [§5.12](#512-severity_tier-strict--normal--relaxed).
- **Section C — Cross-file consistency**: section keys match section
responsibilities, agent files mention the sections they own, severity tables
don't contradict each other, tech-stack alignment between security and coding
agents (GEN-205 is tier-aware).
- **Section D — Heuristic quality**: non-empty sections, tables with rows, dates
consistent. ADVISORY at `strict` and `normal`; **informational** at `relaxed`
(emitted as findings but counted in `total_informational`, not in the BLOCKING
or ADVISORY totals).

The audit summary records `severity_tier` and `tier_source` (`declared` /
`defaulted` / `brief`) so reviewers know the contract the audit enforced.

### 8.1 Common BLOCKING findings and how to fix


| Finding                                            | Fix                                                                                              |
| -------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| `constitution.missing_mandatory_section` (GEN-004) | Add the missing key to `mandatory_section_keys` in §8 and to the table in §2.                    |
| `constitution.section_unreplaced_token` (GEN-011)  | Replace every `{{ TOKEN }}` with concrete value (or explicit N/A statement).                     |
| `constitution.domain_block_unmodified` (GEN-012)   | Rewrite every `<!-- esos:domain-customization -->` block with domain content.                    |
| `compliance.privacy.dsr_missing` (GEN-108)         | For each named regulation, describe specific implementing controls and articles.                 |
| `coding.role_demoted` (GEN-103a)                   | Restore the senior dev / software architect framing in `rulesets/coding.md`.                       |
| `testing.role_demoted` (GEN-103b)                  | Restore the senior tester / QA architect framing in `rulesets/testing.md`.                         |
| `constitution.claude_md_missing` (GEN-116)         | Add a domain-tuned `CLAUDE.md` referencing the Generic Plugin Constitution version.              |
| `constitution.subagent_missing` (GEN-117)          | Ensure all 5 `agents/esos-*.md` exist with domain-mentioning descriptions.               |
| `constitution.tier_undeclared` (GEN-120)           | Add `severity_tier` (`strict`, `normal`, or `relaxed`) to the `constitution.md` §8 catalog row.    |
| `constitution.tier_brief_mismatch` (GEN-121)       | Make `severity_tier` in §8 match `SEVERITY_TIER` from the derivation brief, or update the brief. |


### 8.2 Treating a default-BLOCKING as ADVISORY

**The systematic way** to soften baseline enforcement is to choose a looser
[severity tier](#512-severity_tier-strict--normal--relaxed) up front. The tier
is the documented contract; switching tiers is one decision audited consistently
across every check, recorded in `constitution.md` §8, and reflected in the audit
summary.

**The ad-hoc way** — a few BLOCKING checks may be downgraded to ADVISORY with
an explicit written justification in `constitution.md` §3.2 or §6.1, regardless
of tier. The validator records the justification in the finding's `remediation`
field. Use this sparingly — per-check downgrades on top of a chosen tier are
governance debt with compounding interest. If you find yourself making many
such downgrades, you've picked the wrong tier; switch tiers (with the
appropriate semver bump) instead.

Foundational-floor checks (Section A) and the role-framing checks (GEN-103a /
GEN-103b) cannot be downgraded at any tier or with any justification.

---

## 9. Publishing

### 9.1 The catalog row

`constitution.md` §8 carries the row for the ESOS catalog. Confirm:

- `logical_id` — UUID, generated once and stable across versions.
- `version` — integer, starts at `1` for a new constitution.
- `name` — the human-readable display name.
- `inherits_from` — `"Generic Plugin Constitution v3.1.0"` (or the Generic Constitution
version active when you derived).
- `mandatory_section_keys` — every key from `constitution.md` §2 plus your additions.
- `domain_tags` — at least 2–3 useful tags for catalog search.

### 9.2 Repository pin

Per `constitution.md` §9, your catalog row carries a binding to the Git repository:

- `github_repository_full_name` — `<owner>/<repo>`
- `github_path_prefix` — usually `""` (the constitution lives at repo root)
- `resolution_pin_type` — `commit` (preferred), `tag`, or `branch`
- `resolution_pin_value` — the SHA, tag, or branch name

**Use commit SHA or tag for production.** Branch pins float and create drift —
acceptable for development but not for governance.

### 9.3 Approval

The constitution lifecycle is `draft → approved → deprecated`. Only Organization
Administrators (`constitution:admin`) can publish (move to approved). Workspaces
can only attach to approved constitutions.

After publishing v1, your constitution is **immutable**. Changes create new
versions.

### 9.4 Installing the derived plugin in Claude Code

The derived plugin is structurally identical to the generic base — it ships its
own `plugin.json` and `.claude-plugin/marketplace.json` and is installable the
same way (see [§2.4](#24-installing-the-generic-base-as-a-claude-code-plugin)
for the rationale; only the names change).

Replace `<owner>/<repo>` with the Git location you published to in §9.2, and
`{{ DOMAIN_SLUG }}` with the slug from your brief.

**Option A — Git URL marketplace** (most teams):

```text
/plugin marketplace add <owner>/<repo>@v1.0.0
/plugin install esos-{{ DOMAIN_SLUG }}@esos-{{ DOMAIN_SLUG }}
```

**Option B — local path** (during pre-publish review):

```text
/plugin marketplace add /absolute/path/to/<DOMAIN_SLUG>
/plugin install esos-{{ DOMAIN_SLUG }}@esos-{{ DOMAIN_SLUG }}
```

**Option C — `settings.json`** (shared team config, checked into the
governed-product repos so every contributor's Claude Code picks up the
domain's specialists automatically):

```json
{
  "extraKnownMarketplaces": {
    "esos-{{ DOMAIN_SLUG }}": {
      "source": {
        "source": "github",
        "repo": "<owner>/<repo>"
      }
    }
  },
  "enabledPlugins": {
    "esos-{{ DOMAIN_SLUG }}@esos-{{ DOMAIN_SLUG }}": true
  }
}
```

**Pick one — base OR derivation, usually not both**: a product team that
just *uses* the governance installs only the derived plugin — its five
domain-tuned subagents (`esos-analyst`, `esos-security`, `esos-compliance`,
`esos-coding`, `esos-testing`) and the `esos-validate-constitution` skill are
all it needs to author and re-audit specs. The generic base is the toolkit
for *deriving new* domain plugins, not for using existing ones.

If you genuinely need both (e.g. you maintain the derivation and another
domain plugin at the same time), be aware that the base and every derived
plugin ship subagents with the same five names — installing two of them
together will surface a name conflict in Claude Code. Resolve by disabling
one of the two via `/plugin` or by editing `enabledPlugins` in
`settings.json`.

---

## 10. Maintaining a constitution over time

### 10.1 Semantic versioning

Per `constitution.md` §10, versions follow semver:


| Bump  | Triggers                                                                                                                                                                                                                                                                                     |
| ----- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| MAJOR | Removing or incompatibly redefining a specialist class, mandatory section, or pipeline rule. **Loosening the severity tier** (`strict → normal`, `normal → relaxed`) — weakens a published contract that downstream workspaces depend on. Requires sign-off from constitution administrator. |
| MINOR | Adding a specialist skill / focus area, adding a mandatory section, raising a confidence threshold, tightening severity, **tightening the severity tier** (`relaxed → normal`, `normal → strict`). May be approved by domain governance group.                                               |
| PATCH | Clarifications, typo fixes, non-semantic edits. May be approved by the constitution maintainer.                                                                                                                                                                                              |


### 10.2 Amendment flow

1. Open a Pull Request against the constitution repository.
2. Include the version bump in `constitution.md` §8 and the footer.
3. Update `Last Amended` date.
4. Run `esos-validate-constitution` against the changed branch — must pass.
5. Get sign-off appropriate to the bump (MAJOR > MINOR > PATCH).
6. Merge.
7. Publish the new version to the catalog. Existing workspaces are NOT
  auto-migrated — they re-attach explicitly when ready.

### 10.3 Deprecation

When a constitution is superseded:

- Move it to `deprecated` status.
- Already-attached workspaces continue to work.
- New workspace attachments are blocked from selecting it.
- Provide a migration path to the successor constitution.

### 10.4 Re-deriving from a new generic-base version

When the generic base ships v1.1, v2.0, etc., your constitution does NOT
auto-update. Instead:

1. Compare your derivation's base-anchored content against the new base.
2. Decide: adopt, ignore, or partially adopt the upstream changes.
3. Bump your constitution version and publish.

The `<!-- esos:keep -->` blocks help — they mark the inherited floor that you
should re-sync from upstream.

---

## 11. Troubleshooting

### "Claude produces a generic-feeling derivation"

Cause: the brief was vague. Generic derivations come from generic briefs.

Fix: tighten the brief — specifically the `DOMAIN_SUMMARY`,
`DOMAIN_SPECIFIC_RISKS`, and `APPLICABLE_REGULATIONS` fields. Re-run.

### "Validator flags my regulation citations as too vague"

Cause: you cited "GDPR" without articles.

Fix: cite specific articles (Art. 5(1)(c), Art. 17, Art. 32, …). The COMPLIANCE
specialist needs them to map controls to obligations.

### "Validator flags GEN-117 — subagent description without domain"

Cause: the subagent files were copied from the base verbatim without tightening
descriptions.

Fix: edit each `agents/esos-*.md`, prepend the domain name to the
`description` field.

### "Validator flags GEN-103a or GEN-103b — role demoted"

Cause: the CODING or TESTING agent file lost a focus area or had its role-framing
preamble rewritten.

Fix: restore the senior dev / software architect (CODING) or senior tester / QA
architect (TESTING) preamble. Restore all 14 focus areas. The base file is
authoritative.

### "Claude references an 'ESOS Master Constitution' — does that exist?"

No. The Generic Constitution in this repository **is** the foundational document —
there is no separate master above it. If Claude mentions a "master constitution",
that is residual phrasing from earlier versions of this base (≤ v1.1.0); update
the prompt or correct Claude. The current foundational floor is `constitution.md`
in this repository.

### "I want a sub-domain constitution (a derivation of a derivation)"

Cause: rare but valid use case. The derived constitution itself becomes a base.

Fix: in the derivation brief, declare `BASE_OF_BASES: true`. Then keep the
`esos-create-constitution` skill in the derived's `skills/`. Validator
GEN-119 will not flag the create skill in this case.

### "The validate skill returns no output"

Cause: usually a path issue — the candidate directory wasn't found.

Fix: confirm the candidate path. Use absolute paths if relative ones aren't
resolving.

---

## 12. FAQ

**Q: Do I need Claude Code, or can I do this manually?**
A: Claude Code is recommended but not required. PROMPT.md is a self-contained
LLM prompt; you can paste it into any capable LLM with your brief. Or you can
hand-edit the base files into a derivation. The validate skill flow works the
same either way — VALIDATION.md is a self-contained audit prompt.

**Q: Can I derive without inheriting from the generic base?**
A: No. The Generic Constitution **is** the foundational document — there is no
separate base above it that you could inherit from instead. Every derived
constitution declares `inherits_from: "Generic Plugin Constitution vX.Y.Z"` in its
catalog metadata.

**Q: How long should my constitution be?**
A: A typical derivation is 2–4k lines across ≈15 files. Much shorter usually
means you skipped customization; much longer usually means you put implementation
detail in the constitution where it belongs in design docs.

**Q: Which severity tier should I pick?**
A: If your domain handles regulated data, makes life-impacting decisions, or
carries financial custody — pick `strict`. If it's a mainstream customer-facing
or B2B product where rigour is expected but life isn't on the line — pick
`normal`. If it's internal tooling, a prototype, or low-blast-radius
back-office work where time-to-first-spec matters more than every advisory —
pick `relaxed`. When in doubt, pick stricter: tightening later is a MINOR
bump, loosening is MAJOR. See [§5.12](#512-severity_tier-strict--normal--relaxed)
for the full matrix.

**Q: Can I change the severity tier after publishing?**
A: Yes. Tightening (`relaxed → normal`, `normal → strict`) is a MINOR bump and
goes through the normal amendment flow. Loosening (`strict → normal`,
`normal → relaxed`) is a MAJOR bump because downstream workspaces depend on
the published contract — it requires constitution-administrator sign-off and
should come with a written rationale.

**Q: Validator flags GEN-120 — what is it?**
A: Your `constitution.md` §8 is missing the `severity_tier` field. Add
`severity_tier: "strict"` (or `"normal"` / `"relaxed"`) to the catalog YAML.
Until then, the validator runs at `strict` by default.

**Q: Can two domains share a constitution?**
A: Yes — that's the design point. If two products genuinely share governance
needs, attach both workspaces to one constitution. If they diverge over time,
either tighten the constitution to fit both, or split into two derivations.

**Q: What if my domain has a regulation the COMPLIANCE agent doesn't know about?**
A: Add it. The COMPLIANCE agent's domain-customization block (§6) is exactly for
this — list the regulation, cite the articles, and describe the implementing
controls.

**Q: Can I waive the SECURITY specialist for an entirely public-data domain?**
A: You can, but think twice. "Entirely public" is rarer than it sounds; even
public-data systems have authentication, audit, and DoS concerns. If you genuinely
have no security surface, document the waiver in §3.2 with rationale.

**Q: How do I update the generic base itself?**
A: That's a separate workflow from this manual. See `CLAUDE.md` ("Modify the base
itself") and `constitution.md` §10. Base changes ripple to derivations on re-derivation.

**Q: My derivation produces too many BLOCKING findings — is the validator
broken?**
A: No, the validator is doing its job. BLOCKING findings on a first derivation
are normal — usually clustered around vague regulation citations, missing domain
risks, and unreplaced tokens. Iterate, fix, re-validate.

**Q: Can I publish a constitution that's still failing validation?**
A: Technically yes (the validator is advisory in this repo; the ESOS platform
applies its own publish-time gates). In practice, no — publishing a
known-failing constitution is governance debt with compounding interest.

---

## 13. Glossary


| Term                                 | Definition                                                                                                                                                                                                     |
| ------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ESOS                                 | Enterprise Specification Orchestration System — the AI-assisted pipeline that validates specifications and repositories against constitutions.                                                                 |
| Generic Constitution                 | This repository (`generic-plugin-constitution/`). The foundational, domain-agnostic constitution. Defines the 5 specialist classes, pipeline ordering, mandatory section minimum, audit rules, severity floor, severity tiers, and the CODING / TESTING agent framings. There is no document above it. |
| Domain Constitution                  | A derivation from the generic base for a specific domain (healthcare, fintech, etc.).                                                                                                                          |
| Specialist                           | One of the 5 ESOS agent classes: ANALYST, SECURITY, COMPLIANCE, CODING, TESTING.                                                                                                                               |
| Pipeline                             | The ordered sequence ANALYST → SECURITY → COMPLIANCE → CODING → TESTING → VALIDATION.                                                                                                                          |
| Confidence Threshold                 | The minimum specialist confidence below which corrective paths trigger. Default 0.70.                                                                                                                          |
| Mandatory Section                    | A section every specification under the constitution MUST contain. The Generic Constitution defines a minimum set (8 keys, §2); derivations may add.                                                           |
| Catalog                              | The ESOS database of available constitutions. Workspaces attach to approved entries.                                                                                                                           |
| Workspace                            | The unit ESOS attaches a constitution to; typically corresponds to a product or product area.                                                                                                                  |
| Logical ID                           | UUID identifying a constitution across versions.                                                                                                                                                               |
| `{{ TOKEN }}`                        | Inline placeholder; derivations MUST replace every occurrence.                                                                                                                                                 |
| `<!-- esos:domain-customization -->` | Block to be rewritten with domain content.                                                                                                                                                                     |
| `<!-- esos:keep -->`                 | Inherited-floor block; derivations MUST keep verbatim or tighten.                                                                                                                                              |
| Skill                                | A Claude Code workflow auto-triggered by phrasing or invoked explicitly (`esos-create-constitution`, `esos-validate-constitution`).                                                                            |
| Subagent                             | A Claude Code specialist agent (`esos-analyst`, `esos-security`, etc.) that produces findings against the corresponding ruleset.                                                                               |
| Finding                              | A structured audit result with severity (BLOCKING / ADVISORY), location, message, message_key, remediation.                                                                                                    |
| BLOCKING                             | A finding that prevents publication / submission.                                                                                                                                                              |
| ADVISORY                             | A finding recorded but not blocking.                                                                                                                                                                           |
| Severity Tier                        | One of `strict` / `normal` / `relaxed`, declared in `constitution.md` §8 (`severity_tier`). Calibrates how strictly the generic baseline checks gate publication. Never relaxes foundational-floor rules. See §5.12. |
| Tier-Aware Check                     | A `VALIDATION.md` check whose BLOCKING/ADVISORY severity depends on the candidate's severity tier. Listed in `VALIDATION.md` "Severity Tier (read first)".                                                     |
| Trace Event                          | An immutable audit-trail entry emitted by ESOS during validation runs.                                                                                                                                         |


---

## Where to go next

- Read the [README](README.md) for a 30-second overview of the base.
- Read [CLAUDE.md](CLAUDE.md) if you're using Claude Code — it tells the assistant
how to work with this repo.
- Read [PROMPT.md](PROMPT.md) for the full derivation prompt the create skill
drives.
- Read [VALIDATION.md](VALIDATION.md) for the full audit prompt the validate  
skill drives, including every numbered check..
- For governance questions, escalate to your organization's ESOS constitution
administrator.
- Add this new constitution in ESOS after it is pushed to a Github repository

Good luck. A well-built domain constitution pays dividends every time a
specification is validated under it.