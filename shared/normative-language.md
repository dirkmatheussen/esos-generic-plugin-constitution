# Normative Language Rules

Requirements in specifications validated by ESOS use normative language consistent
with **RFC 2119** (and RFC 8174 for case-sensitivity guidance). This file is included
by the ANALYST and COMPLIANCE specialists, and applies uniformly across every
constitution derived from the generic base.

---

## Requirement Keywords

| Keyword                | RFC 2119 Level                   | When to Use                                                              |
| ---------------------- | -------------------------------- | ------------------------------------------------------------------------ |
| **MUST** / **MUST NOT** | Absolute requirement / prohibition | Non-negotiable; failure = defect.                                       |
| **SHALL** / **SHALL NOT** | Equivalent to MUST / MUST NOT  | Acceptable alternative; use consistently within a single specification.  |
| **REQUIRED**           | Equivalent to MUST                | Acceptable alternative; common in regulatory contexts.                   |
| **SHOULD** / **SHOULD NOT** | Strong recommendation        | Expected in most cases; deviation MUST be documented with rationale.     |
| **RECOMMENDED**        | Equivalent to SHOULD              | Acceptable alternative.                                                  |
| **MAY**                | Optional                          | Permitted but not required; no assumption of implementation.             |
| **OPTIONAL**           | Equivalent to MAY                 | Acceptable alternative.                                                  |

Per RFC 8174, normative force applies only when the keyword appears in **uppercase**.
Mixed-case usage ("Must", "should") is treated as English prose, not a normative
requirement, and is flagged as **NEEDS CLARIFICATION**.

---

## Usage Rules

1. Every **Functional Requirement (FR-xxx)** MUST contain at least one normative
   keyword. FRs without normative language are flagged as **BLOCKING**.

2. **Security and compliance requirements** SHALL use **MUST** or **SHALL** —
   never SHOULD or MAY — unless the constitution explicitly allows flexibility for a
   specific control. A SHOULD on a security control without an exception cite =
   **BLOCKING**.

3. **Aspirational statements** ("the system will be fast", "users will find it easy")
   are NOT normative language and MUST NOT appear in functional requirements. They
   belong in success criteria, restated as measurable outcomes.

4. **Avoid weasel qualifiers** that defeat testability:
   - "appropriately", "reasonably", "as needed", "where applicable", "if necessary"
   - These are admissible only when the condition is defined elsewhere in the
     specification with concrete criteria.
   - Otherwise, they are flagged as **NEEDS CLARIFICATION**.

5. **Success Criteria (SC-xxx)** MUST NOT use normative keywords. They are stated as
   measurable outcomes ("Order placement completes in under 3 seconds for 95th
   percentile requests under expected peak load").

6. **Compound requirements** that bundle two assertions joined by "and" SHOULD be
   split into separate FRs. Compound FRs are flagged as **ADVISORY**.

7. **Negation discipline** — prefer the explicit positive form. "MUST NOT permit
   write access without role X" is clearer than "MUST permit write access only with
   role X" only in a context where the prohibition is the specification's emphasis.

---

## Examples

**Good FR** (normative, testable, single assertion):

> FR-005: The system MUST reject a write request from a caller lacking the `editor`
> role and MUST return a `403` response with a stable `messageKey` identifier.

(Note: this is a compound — the ANALYST may split it into FR-005a / FR-005b.)

**Better — split** (each independently testable):

> FR-005: The system MUST reject a write request from a caller lacking the `editor`
> role.
> FR-006: When a write request is rejected for missing role, the response MUST be a
> `403` carrying a stable `messageKey` identifier.

**Bad FR** (no normative keyword, vague):

> FR-005: The system should handle authorization appropriately.

**Good SC** (measurable, technology-agnostic):

> SC-003: 95% of write requests complete within 2 seconds under expected peak load.

**Bad SC** (uses normative keyword and is implementation-specific):

> SC-003: The API MUST respond in under 200 ms.

---

## Quick Lint Checklist

When reviewing a specification:

- [ ] Every FR contains MUST / SHALL / SHOULD / MAY in uppercase.
- [ ] No FR uses "Must" / "Should" / "may" mixed-case.
- [ ] No SC uses MUST / SHALL / SHOULD / MAY.
- [ ] No FR contains aspirational language ("happy", "great", "easy", "fast").
- [ ] No FR uses weasel qualifiers without defining the condition.
- [ ] Compound FRs are split or explicitly justified as compound.
