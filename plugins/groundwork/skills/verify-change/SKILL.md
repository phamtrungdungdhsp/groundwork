---
name: verify-change
description: Use this after test-driven-development has finished a task (tests green, refactor done), before moving to the next task. Independently checks the finished work by running the real system through its real interface and carrying out each test-plan case for that task step by step, then saves evidence for every case (what was done, what was observed, pass / fail / not verified) to a verification file next to the plan. Confirms the work does what the plan promised, instead of trusting the tests the implementer wrote. Reports and hands failures back, never edits the test plan or expected results to make a case pass. Never run against production.
---

# Verify Change

## What this is for

The tests written during `test-driven-development` were written by the same agent that wrote the code, so they share its blind spots. They can be green while the real system does something else: a route that isn't wired up, a queue job that never runs, a response that differs from the Contract. This step is a second, independent look. It doesn't re-run the test suite. It uses the system the way the test plan describes, through the real interface, and keeps evidence of what it saw.

The point is that the user can review a short list of proven results instead of reading the code diff.

## Before starting

You need the plan and the approved test plan for this work (`plans/`), and a task that has just gone green in `test-driven-development`. Verify one task at a time, the task that just finished.

## Step 1 — Confirm the target is safe, and how to run it

- **Never production.** Only run against a local or staging system. If the target looks like production, or you can't tell, stop and ask.
- **Effects that leave the system:** emails, SMS, payments, third-party APIs. Confirm they're stubbed or in test mode before triggering them.
- **How to run it:** the test plan's "How to run and observe" section says how to start or reach the system, which accounts and data to use, where to see results, and how to trigger time-based cases. Use it. If a project script already drives these checks, use that too.
- **If that section is missing, or doesn't work when you try it:** stop and report it as blocked. Don't guess, and don't invent your own way to run or observe the system. The user can fix the test plan and the loop continues.
- **Accounts and data:** use the ones it names. Only create what's missing through the system's normal paths.

## Step 2 — Keep the check independent

- **Work from the test plan, not the code.** The steps come from each case's Workflow in the test plan. Don't read the implementation to decide what to do or what "correct" looks like.
- **Use the real interface** (UI, public API, CLI) with the credentials a real caller would have. Don't call internal functions, write to the database directly, or use admin access to get past a problem. If something blocks the case, that is the result.
- **Read-only checks of state.** To confirm what the system stored or did, read the database, logs, or queue through read-only access if you have it. A "success" response means little if no record exists, or two do.
- **Don't change anything to make a case pass.** Not the code, not the test plan, not the expected result. If a case fails, that's a result to record. Using the means the test plan names for shortening time (a config value, a test clock, seed data) is part of running the case, not a change.

## Step 3 — Run each case for the task

Go through the task's test-plan IDs one at a time, including the cases that come from the plan's Approach (duplicate delivery, dependency down, restart). For a case with a variation table, check every row.

When the test plan comes from `course-correct`, skip the IDs under `Removed or replaced`. After the correction's own cases, re-check the original IDs under `Still must hold` the same way, using the original test plan for their steps, and record them in the same file.

For each case, record:

- the steps you actually did, in short
- what you observed, as an exact excerpt (status and body, a log line, a stored row), not a paraphrase
- the result: **pass** (matches the case's `Result:`), **fail** (doesn't), or **not verified** (couldn't be run, with the reason)

Keep the cost proportional to what's at stake:

- **High stakes** (money, permissions, data writes, concurrency, anything the plan flagged): verify every case.
- **Low stakes** (display text, a simple read): the happy path and one error case are enough.
- Choose the cheapest channel that can show the result. For a backend, HTTP calls or the CLI with text evidence are cheaper and clearer than driving a browser. Use a browser only when the case is about what the UI shows.
- Keep evidence short: the relevant lines, not whole logs.
- If the project has a script that already drives these checks, use it. If you'd be re-creating the same driver for every case and none exists, suggest adding one to the project and ask first. Don't add it silently.

## Step 4 — Decide what happens next

- **Every case passes:** write the report and continue to the next task. No need to stop for approval.
- **A case fails and the code is at fault:** hand it back to `test-driven-development` to fix, then verify that task again. Stay within the same limit TDD uses: at most 2-3 diagnosed attempts, then stop and report what's failing and what you've ruled out.
- **The system's behavior looks reasonable but differs from the test plan:** the test plan may be wrong. Stop and ask the user. Do not edit the test plan or its expected results yourself. If the user agrees, `course-correct` is the way to change it.
- **Behavior that doesn't match the plan's Approach** (a timer where the plan says a queue): that is a failure, even if the visible result looks right.
- **A case can't be run:** mark it **not verified** and say why. Never mark it pass.

## Step 5 — Write the verification file

Use the same date-time and slug as the plan. Save to:

```
plans/<YYYY-MM-DD-HHmm>-<slug>-verification.md
```

Create it when the first task is verified, then add a section for each later task. If a task is verified again after a fix, replace its section and note the earlier failure briefly.

```markdown
# Verification: <Plan title>
Plan: plans/<same-timestamp>-<same-slug>.md

Test plan: plans/<same-timestamp>-<same-slug>-test-plan.md

Environment: <where this ran, and the commit if known>

## Task 1: <task name>

**Summary:** <passed / failed / partly verified, in one sentence>

| ID | Result |
|----|--------|
| 1.1 | pass |
| 1.2 | fail |
| 1.3 | not verified |

### 1.1 <case title>

**Steps:** <what was done, in short>

**Observed:** <exact excerpt>

**Result:** pass

### 1.2 <case title>

**Steps:** ...

**Observed:** ...

**Expected:** <the test plan's Result, only for failures>

**Result:** fail

## Not verified
- <ID, and why it couldn't be run>

## Left behind
- Test accounts, records, or other data this check created.
```

Put a blank line between every field and before and after every list or table, so they render apart. Leave out sections with nothing in them. Don't delete the data under `Left behind` unless the user asks.

## After a task, and at the end

After a passing task, say in one line which IDs passed and move on. Only stop when something needs the user: a failure that stays after the allowed attempts, a test plan that looks wrong, or a decision that was flagged in the plan.

When the last task is verified, tell the user where the verification file is and give a short recap: how many cases passed, and any fail or not-verified items. Then stop. Whether the work is accepted is the user's decision.
