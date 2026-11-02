---
name: course-correct
description: Make a small correction to work that already has a plan, test plan, or code, without restarting the whole planning process. Carries one change through every layer it touches, in order - plan, then test plan, then tests, then code - so they all still agree afterwards. Run on demand, for when something in the plan turned out to be wrong, or the code doesn't do what the plan says.
disable-model-invocation: true
---

# Course Correct

## What this is for

Sometimes you realize mid-build or after it that something is wrong: a rule in the plan was mistaken, a case was missed, or the code doesn't do what the plan says. Writing a new plan and starting over is far too heavy for that. Just editing the code is too light, because the plan and test plan now describe something the code no longer does, and the next person (or AI) to read them will trust the wrong one.

This skill is the middle path: one correction, carried through every layer it touches, from the top down. When it's finished, the plan, test plan, tests, and code all agree again.

## Step 1 — Find what exists

Find the plan and test plan for this work in `plans/`, matching on topic rather than recency. If more than one could match, ask which one.

If there's no plan or test plan, say so and carry on with the layers that do exist. Don't write a plan or test plan just to have something to edit.

## Step 2 — Work out what kind of correction this is

The kind of correction decides where the change starts:

- **The plan is wrong** — it says the wrong thing or misses something it should have covered. Start at the plan and go through every layer.
- **The plan is right, the code is wrong** — this is a bug. Leave the plan alone. If no test case catches the bug, that's the real gap, so start at the test plan.
- **The test plan is wrong** — the plan and code are right, but a test case expects the wrong thing. Start at the test plan. If the code already behaves correctly, there's no code to change.

Then trace what the correction touches: which tasks, Contracts, and test-plan IDs, and whether those tasks have been implemented yet. A task that hasn't been built only needs its plan and test plan edited. There's no code to change yet.

## Step 3 — Check it's actually small

This skill suits a correction that stays contained. It's probably too big if it:

- breaks something listed under the plan's Assumptions
- changes the plan's Goal
- changes a Contract that other code or people already depend on
- reworks most of the tasks, or a lot of code that's already built

If any of these apply, tell the user what you found and why it looks bigger than a patch, then let them decide how to proceed. Don't start other skills yourself, and don't push on without their answer.

## Step 4 — Confirm the change

Before editing anything, give the user a short summary:

- what kind of correction this is, in one sentence
- what will change in the plan, by task
- which test-plan IDs will be changed, added, or removed
- whether code will need to change

Then stop and wait. A misread correction would spread through every layer, and this is the cheapest place to catch it. Keep this to a few lines, since the correction is meant to be small. If the user already told you to go straight through, skip the wait and just show the summary.

## Step 5 — Update the plan and test plan

Edit the existing files in place, and don't create new ones. Each file should read as the current truth, not as a history of changes, so don't add "changed from X" notes. Git keeps the history.

Follow the formats those files already use. Keep the plan in plain English and write each test case as numbered steps ending in a `Result:` line.

Keep test-plan IDs stable, because test code refers to them:

- **Same scenario, new expectation** — keep the ID and edit the test case.
- **New scenario** — use the next unused number in that task.
- **Removed scenario** — delete the test case and leave a gap in the numbering. Never renumber or reuse an ID.

## Step 6 — Update the tests and confirm they fail

Change the test code to match, keeping one test per test-plan ID and carrying the ID over, as the existing tests do. Delete the test for any removed ID.

Run the changed tests before touching the code. Each one that describes new or corrected behavior should fail, because the behavior isn't there yet. Check the reason: a typo or broken setup isn't a meaningful failure, so fix the test first.

If a test passes when it should fail, stop before writing any code. Either the behavior already exists and the correction may be unnecessary, or the test isn't exercising what it claims to. Tell the user which it looks like.

## Step 7 — Change the code and confirm it passes

Write just enough code to make those tests pass, matching how the surrounding code is organized. Stay inside the correction: anything else you notice goes in your report, not in this change.

Then run the full test suite, not only the changed tests. A correction can break behavior that other tasks rely on. If a test outside the correction fails, treat it as a regression and look into it. Don't edit that test to make it pass unless the correction really does change what it should expect, and if it does, the plan and test plan need to say so too.

If the tests are still failing, read the failure and diagnose it before changing anything else. After 2-3 diagnosed attempts, stop and report what's failing and what you've ruled out.

## Step 8 — Report and stop

Tell the user:

- what changed in each layer: plan, test plan, tests, and code
- which test-plan IDs now pass, and the result of the full suite
- anything you noticed but left alone

Don't commit unless asked.
