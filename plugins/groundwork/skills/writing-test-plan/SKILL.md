---
name: writing-test-plan
description: Use this after a plan (from writing-plans) has been approved and before writing any test code or implementation, when the team is working test-driven. Turns the plan into a plain-English test plan — one set of test cases per task, each written as a concrete numbered scenario (setup and action steps ending in the expected result) — saved alongside the plan so the next step (writing the actual failing tests, then the code to pass them) has something concrete to follow. Trigger for "let's write the tests for this plan", "what should we test before coding this", or any point where a plan exists and TDD is the approach. Do NOT use this to write actual test code (assertions, test framework syntax) — that belongs to the implementation step, not this one.
---

# Writing Test Plan

## What this is for

In test-driven development, the tests are supposed to define what "correct" means before any implementation exists. That definition should be something a person can read and agree with in plain language — "if you send a retry request while one is already pending, it should be rejected" — before it's translated into `assertEquals` and framework syntax. A test plan is that plain-language definition, written down so both the reviewer and whoever writes the actual test code later are working from the same understanding of what needs to be true.

Keep this the same way as a plan: what needs to be verified and how you'd know it worked, not the test code itself. If you catch yourself writing `expect(...)`, mock setup, or framework-specific syntax, that's the next step's job, not this one.

A test case earns its place here only if someone could read it and go run it by hand — click the buttons, make the calls, check the outcome — without needing to see any code first. If a step is vague enough that two people could carry it out differently, it isn't done yet.

## Before writing the test plan

You need an approved plan (from `writing-plans`) to work from — read the plan file in `plans/`. Each task's Goal, Contract (if it has one), Approach (if it has one), and Workflow are what test cases are derived from: the Goal says what should end up being true, the Contract (when present) pins down the exact boundary behavior including its error cases, the Approach (when present, written as Decisions and Rules) names the mechanisms chosen and the invariants, and so what can go wrong with them, the Scenario shows the real situation the task is for and is a good source for the happy-path case, and the Workflow hints at what could go wrong at each step.

If a task clearly involves a mechanism choice (calling another service, deferred work, a queue) but the plan has no Approach for it, don't invent one to test against. Tell the user the plan is missing it and go back to `writing-plans` first.

## Structure

One test-plan file per plan, organized by the same tasks:

```markdown
# Test Plan: <Plan title>
Plan: plans/<same-timestamp>-<same-slug>.md

## How to run and observe

**Environment:** <local or staging, and how to start or reach it>

**Accounts and data:** <which test accounts and seed data to use>

**Where to see results:** <responses, a stubbed inbox, logs, a read-only view of the database or queue>

**Waiting and time:** <how a time-based case gets triggered without really waiting, when any case depends on time>

## Task 1: <task name, matching the plan>

### 1.1 <short test case title>
<one-line description of what this test case is checking>

**Type:** happy path

**Workflow:**
1. <setup or action, in plain English>
2. <next action>
3. ...
4. Result: <the expected outcome — this is what makes the test pass or fail>

### 1.2 <short test case title>
...
**Type:** edge case

**Workflow:**
1. ...
Result: ...

## Task 2: <task name, matching the plan>

### 2.1 <short test case title>
...
**Type:** error case

**Workflow:**
1. ...
Result: ...
```

**Formatting:** put a blank line between every field (`**Type:**`, `**Covers:**`, `**Workflow:**`, ...) and before/after each list or table. Markdown joins consecutive lines into one paragraph, so fields written on adjacent lines render run together on one line.

Number test cases `<task number>.<test number>` so they can be referenced unambiguously later (e.g. "test 2.1 is still failing"). `Result:` is always the last line of the Workflow, never a separate field — reading the whole numbered list top to bottom should feel like one continuous scenario ending in an outcome, not a setup paragraph followed by a separate verdict.

### Writing "How to run and observe"

Once the test plan is approved, implementation and verification run on their own, and the verification step has to run the real system and look at what happened. If that isn't written down, someone has to ask mid-way, which stops the whole loop. So settle it here, where the user is already reviewing:

- **Environment:** local or staging, and the command or address to reach it. Never production.
- **Accounts and data:** which test accounts and seed data exist, and what's missing and needs creating.
- **Where to see results:** how to read what the system did. If the task sends email, SMS or payments, say that they go to a stub or test mode, and where to look at them.
- **Waiting and time:** if any case depends on time (a reminder after 24 hours, an expiry, a retry delay), say how it gets triggered without really waiting: a configurable delay, a test clock, or seed data with timestamps already in the past. Choose a way that doesn't change the code under test. If the system has no such way, say so and ask the user how to handle it before finishing the test plan.

Find these out from the project (README, scripts, existing config) first. Only ask the user for what you can't find. A project with nothing to run, such as a library, says how it gets exercised instead (an example script, a REPL call). Keep the section short and factual. Don't pad it.

### Choosing test cases per task

For each task, cover at minimum: the happy path (the thing working as intended), and any edge or error case that the task's Goal, Contract, or Workflow implies matters. A task with a Contract almost always needs one test case per distinct response in that contract — if the contract says "409 if a retry is already pending," that's its own error-case test, not a footnote on the happy-path test. A task with no Contract still needs at least the happy path, plus whatever edge cases are obvious from its Goal.

### Cases that come from the Approach

A chosen mechanism has behavior the user can observe, and that behavior is part of what "correct" means. When a task has an Approach, add test cases for the observable consequences of each decision, for example:

- **Async or queued work:** the caller gets its response before the work finishes; the work does complete afterwards; a failed job is retried; the same job delivered twice produces one effect; pending work isn't lost when the process restarts.
- **Calling another service:** what the caller sees when that service is slow, times out, or is down; whether anything is left half-done.
- **Where state lives:** the state is still there after a restart, or visible from a second instance, if the Approach relies on that.

Test these through what the system shows (responses, stored records, logs, an admin view), not through the mechanism's internals. "The queue's `add` method was called" belongs to test code and ties the test to one implementation. "The retry happens after the delay, and only once" is what the reader cares about, and it still holds if the mechanism changes.

These cases also catch an agent that quietly swaps in a different mechanism: an in-process timer would fail the "pending work survives a restart" case, so the test plan holds the implementation to the Approach the user approved.

Group cases that share a scenario as described in "Keeping the list short" below. Don't pad the list with test cases that don't correspond to anything in the task — every test case should trace back to something the Goal, Contract, or Workflow actually implies needs to hold.

### Keeping the list short: merge by shared scenario

A test plan with a hundred near-identical cases is exhausting to review, and the reviewer ends up skimming — which defeats the point. Merge cases that are really one scenario seen from several angles, so the list stays short without dropping coverage.

**Variation table.** If cases share the same setup and the same action, and differ only in the input used and the expected result, write one test case with a table instead of one case per input:

```markdown
### 2.3 Rejects invalid payment amounts
**Type:** error case

**Covers:** Contract, 400 responses

**Workflow:**
1. Open a payment that is eligible for retry.
2. Submit the retry using each amount below.
3. Result: each submission is rejected with the stated response, and nothing is scheduled.

| Amount | Expected response |
|--------|-------------------|
| zero | 400, "amount must be positive" |
| negative | 400, "amount must be positive" |
| missing | 400, "amount required" |
```

**Several checks on one outcome.** If a single action has several observable effects (the response, the stored record, an email, an audit entry), use one test case whose `Result:` lists each effect, rather than one case per effect.

**Rule of thumb.** If two cases would differ only in the values typed in or the field checked, they are rows or result items of one case. If they differ in what must be set up first, or in which response the Contract promises, they are separate cases. Also keep separate: security cases, concurrency/duplicate-call cases, and anything where you'd want to know exactly which one failed.

**Safeguards so merging doesn't hide gaps:**
- Every row still needs its own specific expected result. "Rejected" or "works" is not enough.
- Add a `**Covers:**` line to a merged case saying what it covers (e.g. "Contract, 400 responses"), so the reviewer can check coverage without reading every row.
- Number the case as usual (`2.3`). A row is referred to as "2.3, row 2".

### Writing a test case's Workflow

Break the scenario into discrete numbered steps — each one a concrete action or piece of setup, in the order it actually happens — ending in a `Result:` line that states the observable outcome. Don't compress the scenario into a summary paragraph; a reader should be able to follow the steps one at a time and know exactly what to do at each one, the way a manual test case reads.

Each step should be specific enough that two different people would carry it out the same way, without needing literal sample data (exact IDs, real JSON bodies) unless a specific value is actually what's being tested — the point is removing ambiguity about *what happens*, not supplying copy-pasteable fixtures for the next step.

Good:
> **Workflow:**
> 1. Set up a payment that already has a retry scheduled.
> 2. Attempt to trigger another retry on that same payment.
> 3. Result: the second attempt is rejected instead of being scheduled, and the original scheduled retry is left untouched.

Bad — collapsed into a summary instead of steps:
> **Workflow:** Given a payment that already has a retry scheduled, attempt another retry and expect it to be rejected.

Bad — this is test code, not a plan:
> 1. `payment = create_payment(status="pending_retry")`
> 2. `response = client.post(f"/payments/{payment.id}/retry")`
> 3. Result: `assert response.status_code == 409`

Bad — the result is too vague to actually verify:
> 3. Result: the retry logic works correctly.

## Saving the test plan

Use the exact same date-time and slug as the plan it's derived from — don't generate a new timestamp. Save to:

```
plans/<YYYY-MM-DD-HHmm>-<slug>-test-plan.md
```

So a plan at `plans/2026-08-26-1430-add-payment-retry.md` produces `plans/2026-08-26-1430-add-payment-retry-test-plan.md`.

## After saving

Tell the user where the test plan was saved, and give a short recap — how many test cases per task is usually enough, not the full file again. Then stop. Don't start writing actual test code or implementation until the user has reviewed the test plan; they may spot a missing case or disagree with an expected result, and that's much cheaper to fix here than after tests are already written against it.
