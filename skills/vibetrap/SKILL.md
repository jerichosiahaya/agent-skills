---
name: vibetrap
description: >
  Perform a slow, conservative review of code changes before approval. Reconstruct
  the relevant stack, architecture, data flow, and business logic first; verify
  behavior with repository evidence and checks; identify concrete risks; and
  refuse to approve when important assumptions remain unverified. Use for code
  reviews, pull requests, AI-assisted changes, or any request to decide whether
  an implementation is safe to approve.
---

# Vibetrap

Review like the person who will own the incident after this change ships.

This skill is deliberately slow and conservative. The goal is not to detect
whether AI wrote the code. The goal is to prevent anyone from approving code
they cannot explain in the context of this repository, its architecture, and
its business rules.

Do not optimize for a quick approval. Optimize for a defensible decision.

## Non-negotiable rules

- Read the connected code, not only the diff.
- Understand the relevant stack before judging implementation choices.
- Reconstruct the relevant architecture and business logic before listing findings.
- Treat the author's summary, generated comments, and AI explanations as claims to verify, not as evidence.
- A passing test suite is evidence about those tests only. It is not proof that the design or business behavior is correct.
- Never guess a missing business rule. Ask a focused question or record the uncertainty.
- Do not approve code that you cannot explain in plain language.
- Prefer `CANNOT APPROVE — INSUFFICIENT CONTEXT` over a confident guess.
- Make no code changes during the review unless the user explicitly asks for fixes afterward.
- Do not judge the developer. Judge the change, the evidence, and the risk.
- Every finding needs a concrete failure scenario and a file/line reference when available.
- Do not report style preferences, vague concerns, or hypothetical risks without a plausible path to harm.

## Review target

First identify exactly what is being reviewed:

- Pull request, branch, commit, staged changes, uncommitted changes, or named files.
- Base revision and changed revision, when applicable.
- The stated purpose and expected behavior.
- Explicitly out-of-scope behavior.
- Whether this is a production, migration, hotfix, experiment, or internal-only change.

If the target is ambiguous, ask what to review. Do not silently choose an arbitrary
set of files.

If the user has not named a target, use the repository's uncommitted changes,
then the latest commit if there are no uncommitted changes. State which choice
was made.

## Review phases

Complete these phases in order. Do not jump directly from reading a diff to a
ship decision.

### 1. Establish the repository and stack

Inspect the relevant evidence before forming an opinion:

- README and local contribution instructions.
- Package manifests and lockfiles.
- Runtime, language, and framework versions.
- Build, test, lint, and type-check configuration.
- Application entry points and route/event registration.
- Database schema, migrations, seeds, and ORM configuration.
- Deployment, worker, queue, and environment configuration.
- Authentication, authorization, logging, and error-handling conventions.
- Tests, fixtures, neighboring implementations, and documented product behavior.

Write a short stack summary internally or in the report. It should answer:

- What runs this code?
- Where does the relevant input enter?
- Where is state stored?
- What leaves the system?
- Which external systems or asynchronous workers are involved?

Do not make framework-specific claims until the framework version and local usage
have been checked.

### 2. Reconstruct the relevant architecture

Trace the real flow through the code:

1. Input or event entry point.
2. Validation and normalization.
3. Authentication and authorization.
4. Domain or business logic.
5. Persistence and transaction boundaries.
6. External calls, queues, or events.
7. Response, output, or user-visible state.
8. Logging, metrics, retries, and recovery.

Read callers of changed functions and the important functions they call. When a
signature, return value, event, database field, or API contract changes, search
for every consumer. A small diff can still break untouched callers.

Record the relevant boundary assumptions:

- One process or multiple processes?
- Synchronous or asynchronous execution?
- At-least-once or at-most-once delivery?
- Shared or process-local state?
- Transactional or eventually consistent writes?
- Expected volume, latency, and concurrency?

If the repository does not make an assumption clear, mark it as unknown instead
of filling it in from general best practice.

### 3. Reconstruct the business logic

Explain what the change means in domain terms, not just what each function does.
Identify:

- The user or system action being supported.
- Relevant entities and their ownership.
- Valid states and state transitions.
- Business invariants that must always hold.
- Permissions and role rules.
- Validation and normalization rules.
- Duplicate, retry, and idempotency behavior.
- Empty, null, zero, expired, partial, and malformed input behavior.
- Failure behavior when a database or external service is unavailable.
- Whether writes can be partial or out of order.
- What existing behavior must remain compatible.

Separate these three things in the report when needed:

- **Code behavior:** what the implementation actually does.
- **Required behavior:** what the product or domain appears to require.
- **Unknown:** what cannot be established from repository evidence.

Never treat a plausible implementation as proof of the required behavior.

### 4. Understand the change

Read the complete diff, then read changed files in context. Check:

- Whether the implementation belongs in the existing architecture.
- Whether it duplicates an existing rule or helper.
- Whether it bypasses a validation, authorization, or transaction boundary.
- Whether it changes behavior for existing callers.
- Whether migrations are complete and safe for existing data.
- Whether tests exercise the risky behavior rather than only the happy path.
- Whether error handling preserves useful failures instead of hiding them.
- Whether comments describe verified behavior or merely repeat a guess.

Do not reward a change for being small, clean, or idiomatic if it violates the
repository's actual contracts.

### 5. Verify with evidence

Run the most relevant repository checks when possible:

- Focused tests for changed behavior.
- Related integration or end-to-end tests.
- Type checks.
- Linting and formatting checks.
- Build or packaging checks.
- Migration validation.
- A documented local reproduction or manual check.

Record each command and result. Also record checks that could not be run and why.
Explain what a passing check does not prove. For example, unit tests may not
prove authorization, migration safety, concurrency behavior, or compatibility
with an untested caller.

If a check fails, investigate whether the failure is caused by the change. Do not
silently ignore a failure because it appears unrelated; state the evidence and
remaining uncertainty.

### 6. Adversarial review

Look for concrete cases involving:

- Wrong business result or invalid state transition.
- Broken callers or backward-incompatible API behavior.
- Missing authorization or ownership checks.
- Injection, secret exposure, unsafe deserialization, or sensitive logging.
- Data loss, partial writes, incorrect transaction boundaries, or bad migrations.
- Race conditions, duplicate processing, retry storms, or non-idempotent jobs.
- Time zones, date boundaries, rounding, money, and precision.
- Empty, zero, null, malformed, duplicate, expired, and very large inputs.
- External API timeouts, partial failures, and changed response shapes.
- Unbounded memory, N+1 queries, excessive work, or incorrect concurrency assumptions.
- Missing monitoring, alerts, audit trails, or operational recovery.
- Tests that pass while the important business rule remains untested.

For each possible concern, re-read the code and confirm the path is real. Do not
report a finding just because a failure is theoretically imaginable.

## Approval gate

Only recommend `APPROVE` when all of these are true:

- The relevant stack is understood well enough to judge the implementation.
- The relevant architecture and data flow can be explained.
- The relevant business rules and invariants can be explained.
- Changed behavior matches those rules.
- Material callers, dependencies, migrations, and failure paths were checked.
- Appropriate validation commands passed or their limitations are immaterial.
- No unresolved material security, correctness, data, compatibility, or operational risk remains.
- Remaining assumptions are explicit and low-risk.

Use one of these verdicts:

- `REQUEST CHANGES` — a concrete material problem must be fixed before approval.
- `CANNOT APPROVE — INSUFFICIENT CONTEXT` — the change may be fine, but important architecture, business rules, code, or validation evidence is missing.
- `APPROVE WITH EXPLICIT FOLLOW-UPS` — no known release-blocking issue remains, and the follow-ups are genuinely low-risk and explicitly accepted.
- `APPROVE` — the change and its relevant risks were understood and checked.

Do not use `APPROVE WITH EXPLICIT FOLLOW-UPS` to hide unresolved correctness,
security, data integrity, compatibility, or migration risk.

## Findings standard

Findings are ordered by impact:

1. **Must fix** — correctness, security, data loss, broken compatibility, or a failure at expected load.
2. **Should fix** — real risk, missing protection or test, operational weakness, or meaningful performance problem.
3. **Follow-up** — low-risk cleanup or a small improvement that does not affect approval.

Every finding must include:

- A short title.
- File and line reference when available.
- A concrete input, state, timing, or failure scenario.
- What the code does.
- Why that violates a requirement or creates real risk.
- The smallest fix that resolves the problem.
- What happens if it is skipped.

If a concern has no concrete scenario, omit it or record it under unknowns rather
than presenting it as a finding.

## Required report format

```md
# Review: <change>

## Verdict

<one allowed verdict>

## Review target

- Target:
- Purpose:
- Base and changed revision:
- Scope limits:

## What this change does

<Two or three plain-language sentences.>

## Architecture understood

- Stack and runtime:
- Entry point:
- Main components:
- Data flow:
- Persistence and transactions:
- External systems or workers:
- Runtime assumptions:

## Business logic understood

- User/system action:
- Entities and ownership:
- State transitions:
- Invariants:
- Permissions:
- Failure and retry behavior:
- Compatibility expectations:

## Checks performed

- `<command>` — passed/failed/not run; <what it proves and does not prove>

## Findings

### Must fix

1. **<short title>** (`path/to/file.ext:line-line`)
   - **Scenario:** ...
   - **What happens:** ...
   - **Why it matters:** ...
   - **Fix:** ...
   - **If skipped:** ...

### Should fix

<Same format, or `None found.`>

### Follow-up

<Same format, or `None found.`>

## Unknowns and assumptions

- ...

## Final decision

<One sentence explaining why the verdict is justified by the evidence.>
```

Skip empty finding sections only when the report remains clear. Always include
architecture, business logic, checks, unknowns, and the final decision for a
serious approval review.

## Anti-rubber-stamping checklist

Before finalizing, ask:

- Can I explain the relevant request/event-to-output path without repeating the author's summary?
- Can I state the business invariant this change protects or changes?
- Did I inspect callers and consumers outside the diff?
- Did I inspect the failure path, not only the happy path?
- Did I check permissions and ownership at the real trust boundary?
- Did I check retries, duplicates, concurrency, and transaction behavior where relevant?
- Did I verify migrations and existing-data behavior?
- Did I run or inspect the right checks for this stack?
- Did I separate verified facts from assumptions?
- Would I still approve if the AI-generated explanation were removed?

If any answer is no and the gap matters, do not approve. State the gap plainly.
