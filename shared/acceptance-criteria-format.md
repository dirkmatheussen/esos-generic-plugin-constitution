# Acceptance Criteria Format

User stories use the **Given/When/Then (GWT)** format for acceptance scenarios.
GWT is the default — deviate only when another format communicates more clearly,
and only with rationale recorded in the spec's (or parent story's / feature's)
`assumptions` section.

This file is included by the ANALYST and TESTING specialists, and applies uniformly
across every constitution derived from the generic base.

---

## Where Acceptance Criteria Live

Acceptance criteria MAY live either inline in the spec markdown or in the parent
story's record in the team's work-tracking system (JIRA, Linear, GitHub Issues,
etc.). The artifact under ANALYST review MUST cite each acceptance scenario by
its stable identifier (`AC-001`, `AC-002`, …) so the validator can resolve every
reference. An acceptance scenario referenced from the artifact but not resolvable
to a definition (inline or in the linked ticket / record) is **BLOCKING**.

When the artifact under review is a **feature** or **epic**, its acceptance
criteria MAY reference the child user stories' acceptance scenarios by ID rather
than restate them. The feature- or epic-level AC is then satisfied when every
child-story AC is satisfied. Feature- or epic-level ACs that describe only the
*decomposition* (e.g. "FE-58 is complete when US-204, US-205, US-206 are all
delivered") are acceptable; feature- or epic-level ACs that introduce
*additional* behavioural assertions beyond the child stories MUST themselves be
in GWT form.

---

## Format

```
**Given** [a precondition describing the relevant system state and actor context]
**And**   [additional preconditions, each independently verifiable]
**When**  [the actor performs a specific action or event occurs]
**Then**  [the expected observable outcome]
**And**   [additional outcome assertions, chained to the Then]
```

---

## Rules

1. Each **Given** clause describes a concrete, testable state — not "the system is
   working normally" but a specific fixture (e.g. "Given a user with role `editor`
   and exactly 3 draft orders").

2. Each **When** clause describes a single trigger — one user action or one event —
   not multiple actions bundled into one step.

3. Each **Then** clause describes an observable, verifiable outcome — not "the
   request is processed" but something a test can assert (status code, message body
   field, persisted record state, audit log entry).

4. **And** clauses extend the Then; each is independently verifiable.

5. Every acceptance scenario carries a unique identifier (e.g. `AC-001`, `AC-002`)
   referenced from the parent user story and traced from the test plan.

6. Every user story has at minimum:
   - One **happy path** scenario (normal successful operation).
   - One **error / exception path** scenario (invalid input, permission denial, or
     external-service failure).

7. Scenarios are **deterministic** — the same Given + When always produces the same
   Then. Non-deterministic outcomes (e.g. probabilistic AI responses) are reframed as
   ranges or invariants ("Then the response field `category` is one of {A, B, C}").

8. Scenarios are **observable** — the Then describes what an external observer can
   see, not internal implementation state ("Then the database row's `state` column
   transitions to `APPROVED`" is acceptable as long as the database is the system's
   observable boundary; "Then the internal `OrderState` enum is `APPROVED`" is not).

---

## Example Patterns

### Pattern: Permission Denial

```
Given a user authenticated as [role-without-permission]
When the user attempts to perform [restricted-action]
Then the request is rejected with a 403 response
And the response body contains messageKey "auth.role.missing"
And the rejection is captured in the audit log with action "REJECT" and reason "role.missing"
And no change is made to the underlying data
```

### Pattern: Concurrent Operation — Single-Resource Race

```
Given exactly 1 unit of [resource] is available
And user A and user B simultaneously submit requests for 1 unit each
When both requests are processed
Then exactly one request succeeds with response status 200
And exactly one request is rejected with response status 409 and messageKey "resource.exhausted"
And no request results in a negative count
And both outcomes are captured in the audit log with the same correlation window
```

### Pattern: Idempotency-Key Replay

```
Given a request with idempotency key K has previously succeeded with response R
When a second request with the same key K and identical body is submitted
Then the response is identical to R
And no additional side effect (database write, event emission, external call) has occurred
```

### Pattern: Segregation of Duties

```
Given user [initiator] with role INITIATOR has submitted [request]
When user [initiator] attempts to approve the same [request]
Then the approval is rejected with messageKey "approval.self_service_not_permitted"
And the [request] remains in PENDING_APPROVAL state
And the rejection is captured in the audit log
```

### Pattern: Personal Data Erasure

```
Given a data subject with id [subject-id] has personal data across [tables/services]
When an erasure request for [subject-id] is processed
Then every PII field on the subject's records is removed or anonymized
And the erasure is captured in the audit log with the request id
And subsequent reads of those records do not return PII values
And the operation completes within the regulatory deadline declared in the specification
```

### Pattern: Multi-Tenant Isolation

```
Given user A is authenticated in tenant T1 and user B in tenant T2
And tenant T2 contains record R2 not visible to T1
When user A requests record R2
Then the response is 404 (not 403, to avoid existence leak) with messageKey "resource.not_found"
And no log statement contains the value of R2
```

---

## Anti-Patterns to Flag

These indicate vague or untestable acceptance criteria — flag as
**NEEDS CLARIFICATION**:

| Anti-Pattern              | Example                                          | Issue                                              |
| ------------------------- | ------------------------------------------------ | -------------------------------------------------- |
| Vague outcome             | "Then the request is processed correctly"        | No observable assertion.                           |
| No actor state            | "When an order is placed"                        | Who placed it? What were the preconditions?        |
| Multiple triggers         | "When the user submits and then confirms"        | Two actions in one step.                           |
| Implementation detail     | "Then the SQL record is updated"                 | Should describe business outcome at a stable boundary. |
| Non-verifiable outcome    | "Then the user is satisfied"                     | Cannot be tested.                                  |
| Time-dependent ambiguity  | "Then the system processes it eventually"        | No bound; cannot fail a test deterministically.    |
| Hidden coupling           | "Then the order is created (and the email is sent)" | Bundled assertions; split into independent Thens. |

---

## Domain-Specific Patterns

<!-- esos:domain-customization -->

The derived constitution adds domain-specific scenario patterns here. Examples:

- Automotive: VIN-supersession resolution; recall flag propagation; price-list
  override during open campaign.
- Healthcare: consent-gating; break-glass with audit; minimum-necessary scope
  enforcement.
- Fintech: idempotency-key replay; double-entry invariant; settlement-window cutoff.

<!-- /esos:domain-customization -->
