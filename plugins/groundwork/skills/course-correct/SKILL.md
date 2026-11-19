---
name: course-correct
description: Make a small correction to work that already has a plan, test plan, or code, without restarting the whole planning process. Writes a new, short correction plan and correction test plan that point back to the originals and describe only what changes, leaves the original files untouched, then follows the same flow as feature-development - the user approves the plan, then the test plan, and implementation and verification run on their own. Run on demand, for when something in the plan turned out to be wrong, or the code doesn't do what the plan says.
disable-model-invocation: true
---

# Course Correct

## What this is for

Sometimes you realize mid-build or after it that something is wrong: a rule in the plan was mistaken, a case was missed, or the code doesn't do what the plan says. Writing a full new plan and starting over is far too heavy for that. Just editing the code is too light, because the plan and test plan now describe something the code no longer does.

Editing the old plan in place is also a poor answer: nobody can tell afterwards what was changed, when, or why. So this skill leaves the original plan and test plan exactly as they are and writes a **correction plan** and a **correction test plan** next to them. Each points back to the original and contains only what changes. The history is then a chain of files that can be read in order, and the correction goes through the same review and build flow as any other work.

## Step 1 — Find what exists

Find the original plan and test plan for this work in `plans/`, matching on topic rather than recency. If one of them has a `Corrects:` line, or a later file corrects it, follow the chain to the newest correction, because that is the current truth. If more than one chain could match, ask which one.

If there is no plan at all, there's nothing to correct. Say so and suggest `feature-development` instead. Don't write a correction against nothing.

Check whether the work has a `-verification.md` file, and which tasks it shows as built and verified.

## Step 2 — Work out what kind of correction this is

The kind of correction decides what the correction plan says:

- **The plan is wrong** — it says the wrong thing or misses something it should have covered. The correction plan states the new truth for those parts.
- **The plan is right, the code is wrong** — this is a bug. The correction plan says the original stays valid and the work is bringing the code in line. If no test case catches the bug, that's the real gap, and the correction test plan adds the missing case.
- **The test plan is wrong** — the plan and code are right, but a test case expects the wrong thing. The correction test plan replaces that case. The correction plan can be a few lines saying nothing in the original plan changes.

Then trace what the correction touches: which original tasks, Contracts, Approaches, and test-plan IDs, and whether those tasks have been built.

## Step 3 — Check it's actually small

This skill suits a correction that stays contained. It's probably too big if it:

- breaks something listed under the original plan's Assumptions
- changes the original plan's Goal
- changes a Contract that other code or people already depend on
- reworks most of the tasks, or a lot of code that's already built

If any of these apply, tell the user what you found and why it looks bigger than a patch, then let them decide how to proceed. Don't start other skills yourself, and don't push on without their answer.

## Step 4 — Write the correction plan, then stop

Use the same structure and rules as `writing-plans` (Contract, Approach, database changes, flagged concurrency decisions, plain English, no code), but write only what changes. Get the real date and time from the system (e.g. `date +%Y-%m-%d-%H%M`), and save to:

```
plans/<YYYY-MM-DD-HHmm>-<slug>.md
```

`<slug>` names the correction, such as `fix-retry-duplicate-handling`.

```markdown
# Correction: <short title>
Corrects: plans/<original-plan>.md

Earlier corrections: plans/<earlier-correction>.md (only if there are any)

## Overview
What was wrong and why this correction is needed, in plain language.

## Goal
What is true once the correction is in.

## Tasks

### Task N: <short task name>
**Changes:** one line saying how this relates to the original, for example "replaces the Contract of original Task 2", "amends step 3 of the Workflow of original Task 3", or "new".

**Goal:** what this task achieves.

**Scenario / Depends on / Contract / Decisions / Rules / Workflow:** only the ones that change, in the same formats as `writing-plans`. Always include a Scenario that says what was wrong before and what is true after.

## Unchanged
Everything in the original plan that isn't listed above stays as it is.

## Out of scope
- ...
```

Number the correction's tasks from the next number after the highest task number used in the original plan and any earlier corrections (original has Tasks 1-4, so the correction starts at Task 5). Then the test-plan IDs stay unique across the whole chain, and test code that carries an ID never collides.

Put a blank line between every field and before and after every list or table.

Tell the user where it was saved, with a short recap. Then **stop and wait for approval**. This plan is where the user confirms the correction was understood correctly.

## Step 5 — Write the correction test plan, then stop

Once the plan is approved, follow `writing-test-plan` for the structure, the case format, and the rules on merging cases and on cases that come from the Approach, but cover only the correction's tasks. Use the same timestamp and slug as the correction plan:

```
plans/<YYYY-MM-DD-HHmm>-<slug>-test-plan.md
```

```markdown
# Test Plan: <correction title>
Plan: plans/<same-timestamp>-<same-slug>.md

Corrects: plans/<original-test-plan>.md

## How to run and observe
Same as the original test plan, or only what differs.

## Removed or replaced
| Original ID | What happens | Why |
|-------------|--------------|-----|
| 2.3 | replaced by 5.1 | ... |
| 2.4 | removed | ... |

## Still must hold
Original IDs that stay valid but sit close to the change, so they get checked again. IDs only, no copying of their content.
- 2.1, 2.2

## Task 5: <task name, matching the correction plan>

### 5.1 <case title>
...
```

Leave out `Removed or replaced` and `Still must hold` when they have nothing in them. Don't edit the original test plan, and don't reuse or renumber an original ID.

Then **stop and wait for approval**, as `writing-test-plan` does.

## Step 6 — Build and verify, on its own

Once the test plan is approved, run the same loop as `feature-development`: `test-driven-development` and `verify-change` over each task in the correction plan, without stopping between tasks. The stop conditions are the ones `test-driven-development` lists.

Two things are specific to a correction:

- The tests for IDs under `Removed or replaced` get deleted or replaced, so no test is left guarding behavior that was dropped.
- Run the whole test suite, not only the new tests, because a correction can break behavior other tasks rely on. A failure outside the correction is a regression to look into. Don't edit that test to make it pass unless the correction really does change what it should expect, and if it does, the correction plan and test plan need to say so.

## Step 7 — Report and stop

Tell the user:

- where the correction plan, test plan, and verification file are
- which test-plan IDs pass, including the `Still must hold` ones, and the result of the full test suite
- anything you noticed but left alone

Anything unrelated that you notice goes in this report, not in the change. Don't commit unless asked.
