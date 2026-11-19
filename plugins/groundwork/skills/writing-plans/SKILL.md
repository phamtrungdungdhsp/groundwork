---
name: writing-plans
description: Use this once the problem is understood (typically right after a discovery/mental-model step) and it's time to turn it into a concrete plan before writing any code. Produces a short, plain-English plan file — an overview that tells the story (how it works before and after) with a glossary, goals, a single list of database changes (when there are any) mapped to the tasks that need them, a task breakdown where each task has a concrete scenario, a goal, what it depends on, the decisions and rules (the mechanisms chosen, such as sync vs async, REST vs gRPC, queue vs event emitter) and a workflow, open rollout questions, and an explicit out-of-scope list — saved to plans/ so it can be reviewed and later followed by an execution step. Trigger for "let's plan this out", "break this into tasks", "write a plan for X", or any multi-step coding work about to start. Do NOT use for a single, obvious, one-step change — planning ceremony for a one-line fix just adds friction.
---

# Writing Plans

## What this is for

A plan exists so that before anyone writes a line of code, both you and the user agree on *what* is going to be built, in *what order*, and by *which technical mechanisms* — not on the exact code. A plan full of function signatures and code snippets is really a first draft of the implementation wearing a plan's clothes; it's slower to review, harder to change your mind about, and it hides the one thing a plan should make obvious: the shape of the work. Keep every part of the plan in plain English.

There are two kinds of "how", and they belong in different places. The *mechanism* — does this call run synchronously or asynchronously, over REST or gRPC, through a queue or an in-process event — is the decision most worth the user's review, because a wrong choice here (a polling timer where a queue was available) is expensive to unwind and invisible in a diff. That goes in each task's **Decisions** (and **Rules**, see below), which together are the task's Approach. The *code-level* how — function internals, exact library calls — comes later, when the plan is executed.

## Before writing the plan

Make sure the problem itself is already understood — the mental model from discovery is confirmed, or the request is simple enough that there's nothing left to clarify. If it isn't, that's the discovery step's job, not this one; don't use plan-writing to paper over a fuzzy understanding of the problem.

## Structure

Every plan follows this shape:

```markdown
# <Plan title>

## Overview
The story in plain language: how things work today, what goes wrong or is missing, and what this plan changes. Use one concrete example (a named person, machine or request), not abstract wording.

## How it works
(skip for a small plan) A short before → after sketch of the main flow, for example two small arrows or a few lines, so a reader who doesn't know the system can see what moves.

## Glossary
(skip when every term is already common knowledge) Each domain term, acronym or internal name the plan uses, with a one-line plain meaning.

## Goal
The overall outcome this plan needs to achieve. What does "done" look like, described as an outcome — not a list of steps.

## Assumptions
Things this plan takes for granted — if one of these turns out to be false, the plan itself needs to change, not just a task. Skip this section entirely if there's genuinely nothing to flag.
- ...

## Database changes
(only when the plan changes the database; skip entirely otherwise) Every schema change in the plan, in one place.

| # | Change | Needed by |
|---|--------|-----------|
| D1 | What changes (new table / new column / index...) and what it holds, in plain English. | Task N (creates it), Task M (reads it), ... |

**Existing data:** whether current rows are safe, and whether anything needs backfilling.

**Order of migration:** which changes must land together or before others.

## Tasks

Task order: Task 1 → Task 2 → ... (one line, only if dependencies aren't a simple chain, say so here).

### Task 1: <short task name>

**Scenario:** a short concrete story of this task in use: who or what does what, what happened before, what happens after.

**Goal:** what this specific task achieves on its own.

**Depends on:** the earlier tasks (or an outside condition) this one needs.

**Database:** (only when this task touches a change listed above) the D-numbers, each marked creates / reads / writes / relies on.

**Contract:** (only when this task creates or changes something other code/people will call) the input/output shape of that boundary.

**Decisions:** (only when the task involves a real technical choice: talking to another service, deferred or repeated work, where state lives, failure handling, a new dependency, a rejected alternative) one line each: the choice, the reason, and the alternative that was rejected.

**Rules:** (only when the task has invariants that must always hold) things that are always true after this task, with no alternative to weigh.

**Workflow:**

1. First plain-English step of what gets built/changed.
2. Next step, in the order it should happen.
3. ...

### Task 2: <short task name>

**Scenario:** ...

**Goal:** ...

**Depends on:** ...

**Workflow:**

1. ...

## Rollout and open questions
(only when there are some) Things the plan can't answer yet or that matter when shipping, each marked `[open]` with the question: gaps while the change is live, rollback, ordering with other releases, and how to know when it is safe to remove a fallback.

## Out of scope
- Things this plan deliberately does not cover, so no one — human or AI — quietly wanders into them mid-execution.
```

**Formatting:** put a blank line between every field (`**Type:**`, `**Covers:**`, `**Workflow:**`, ...) and before/after each list or table. Markdown joins consecutive lines into one paragraph, so fields written on adjacent lines render run together on one line.

List tasks in the order they should be done — the order in the list *is* the execution order, no separate dependency diagram needed. When the plan changes the database, the schema task comes first.

### Writing the Overview, How it works and Glossary

A reader who didn't take part in the discussion has to understand the plan from the page alone. So the Overview tells a story with one concrete example rather than listing mechanisms, and every term of the domain is explained the first time the reader needs it.

- **Overview:** today's situation, what hurts or is missing, what changes. Name an example (Alice, the instance "api-prod", order #123) and reuse the same one in the task Scenarios.
- **How it works:** a before → after sketch for the main flow. Add it when the plan changes how several parts talk to each other; skip it for small plans.
- **Glossary:** one line per term the reader may not know (domain words, acronyms, internal names, the old mechanism being replaced). Define a term only if it appears in the plan. If you inferred a meaning rather than read it, mark it `[confirm]` so the user checks it. Never guess silently.

### Writing a task's Scenario, Depends on and rules for tasks

- **Scenario:** two to four lines of a real situation, before and after if the task changes behavior. It is the first thing in the task because it answers "what is this about?" before any technical field. Reuse the plan's example. A task with no visible behavior (a migration) still gets one: say who or what is affected and how. Don't invent facts; if you aren't sure the flow works that way, mark it `[confirm]`.
- **Depends on:** the earlier tasks this one needs, or an outside condition (a previous phase shipped). Put "none" only for the first task. It shows what can be built in parallel.

### Writing Assumptions

An assumption is something the plan is relying on without verifying — "the auth service already returns a stable user ID", "this only needs to handle USD". These aren't goals and they aren't tasks; they're the ground the plan is standing on. Write them down because if one is wrong, it's cheaper to find out now, from a five-second read, than after several tasks are built on top of it. Most plans only have one or two, if any — don't manufacture assumptions just to fill the section, and skip it entirely when there's nothing worth flagging.

### Writing Database changes

Database changes are the thing most easily lost in a plan: they get mentioned in whichever task happens to need them, so nobody can see the whole schema picture without reading every task. Gather them into the **Database changes** section so a reader can see all of them at once and see which tasks depend on each one.

- Give each change an ID (D1, D2, ...) and list which tasks create, read, write, or rely on it. Each task that touches a change repeats the IDs on its own `Database:` line, so the link can be read from either side. The two must agree.
- **Put all schema changes in one early task, normally Task 1.** The schema is the foundation the other tasks build on; once it exists, tasks that don't depend on each other can be built in parallel. Don't scatter migrations into the tasks that first use them. Task 1 is the only task that *creates* a D-number.
- Always say what happens to existing data (safe as-is, needs a default, needs a backfill). This is the part that is risky and easiest to skip.
- Data-only work, such as backfills or seed data, is not a schema change but belongs in the same section if the plan needs it.
- Describe the change in plain English: what it holds and why. Column types and SQL belong to execution, not the plan.
- If the plan makes no database changes, leave the section and the `Database:` lines out entirely.

### Writing a task's Contract

A contract is the shape of a boundary something else will call or depend on: an API endpoint, a function signature other code will use, an event payload, a CLI flag, a database schema change. It's different from the Workflow in kind, not just detail — the workflow is a sequence of steps, the contract is a fixed shape that has to be pinned down precisely because getting it wrong is expensive to fix later (other code will already be written against it). That precision is why it gets its own line instead of being folded into a Goal written in looser prose.

Only add a Contract to a task that actually introduces or changes one of these boundaries. A task like "refactor internal helper" or "clean up logging" has nothing to put here — leave the line out entirely rather than writing "N/A".

Keep it to the shape, not the reasoning behind it — inputs, outputs, and the error/edge cases that are part of the same boundary:

Good:
> **Contract:** `POST /payments/:id/retry`, no body. Returns `200` with `{ retryScheduledAt }`, or `409` if a retry is already pending.

Bad (this is now describing internals, not the boundary):
> **Contract:** Calls `RetryPolicy.schedule()` which pushes onto the `retry_queue` Redis list with a computed TTL...

How the retry gets scheduled is a real decision, but it goes under the task's Decisions (see below), not in the Contract.

### Writing a task's Decisions and Rules (the Approach)

Decisions and Rules together are the task's **Approach**: where the technical solution gets decided *before* code exists, so the user can review it. Without it, an agent fills the gap by copying whatever the nearest existing code does, which is how a polling `setInterval` ends up in a codebase that already has a job queue. The execution skills (`test-driven-development`, `verify-change`) and `writing-test-plan` read both fields as the Approach.

The two fields are different in kind:

- **Decisions** are choices where an alternative existed. Each has the choice, the reason, and what was rejected, ideally with the situation that made the reason matter ("keeping the copies was rejected: they still open with a leaked key").
- **Rules** are invariants: things that must always be true and have no alternative to weigh ("the two keys are written in the same transaction, so they never disagree"). Compatibility statements ("accounts not touched keep the old data until it's removed") also go here.

If you can't say what was rejected, it's a Rule, not a Decision. Keep the Workflow free of both: it only says what gets built, in order.

Only add Decisions or Rules to a task that involves a real mechanism choice or invariant. A task that adds a field to a form, or renames something, has none: leave the lines out rather than writing "N/A". Cover only the decisions the task actually involves, drawn from:

- **Talking to another service or component:** sync or async; REST, gRPC, or another protocol; request/response or event; what the caller does while it waits.
- **Deferred or repeated work:** a queue or job runner, a scheduler or cron, an in-process timer, or an event emitter; what happens to the work if the process restarts; whether it can run twice.
- **Where state lives:** database, cache, or memory; which component owns it.
- **Failure handling:** timeout, retry, fallback, and what the caller sees when a dependency is down.
- **Reuse:** which existing module, service, or library to use, versus building something new.
- **New dependencies:** any new library, service, or infrastructure the task would add. Always call these out.

**How to decide.**

1. Look at how the codebase already does similar things, and at what's available to build on (a queue, a job runner, an event bus, a shared client). Existing code shows what's *available*, not what's *right*. Don't copy a pattern just because it's there, and don't ignore it either.
2. If the best choice differs from what the nearby code does, say so in the Decisions and give the reason. If the existing pattern looks flawed, flag it to the user instead of silently repeating it or silently diverging from it.
3. When there's a real trade-off (cost, operational burden, latency, a new dependency), don't pick silently. Give the user two or three options with a recommendation and ask. When one choice is clearly right, just state it.

**Format.** One line per decision: the choice, why, and what was rejected. Stay at the mechanism level.

Good:
> **Decisions:**
> - **Scheduling the retry:** put a delayed job on the existing job queue, so retries survive restarts and work across several instances. Considered an in-process timer; rejected because pending retries would be lost on restart.
> - **Charging the card:** the job worker calls the payment provider synchronously with a timeout; on failure the job goes back on the queue.

Bad — too vague to review:
> **Decisions:** use an appropriate async mechanism for the retry.

Bad — this is code, not an approach:
> **Decisions:** call `queue.add('retry', { paymentId }, { delay: 5000 })` and handle it in `RetryProcessor.process()`.

### Flag concurrency and data-integrity decisions while planning

Some tasks touch things that go wrong in ways that are easy to miss until code already exists: a resource more than one request could act on at the same time, an action that might get triggered twice for what should be a single effect (a retried request, a redelivered webhook), or a step that only makes sense if several writes succeed or fail together. These are business decisions, not implementation details — whether a duplicate retry gets silently ignored or rejected outright changes what the API *is*, not just how it's coded. Deciding it now, with the user, is cheaper than leaving it for whoever implements the task to guess.

While writing a task, check whether it has one of these shapes, and if it does, raise it with the user before finalizing the plan:

- Could this be called more than once for what should be a single effect (retries, webhooks, double-clicks)? Decide what a duplicate call should do, and put it in the task's Contract — this is exactly the kind of thing that belongs in a `409` or similar response, as in the example above.
- Could two requests act on the same resource at the same time? Decide whether that's fine as-is (last-write-wins) or needs to be prevented, and note whichever it is — in the Contract if it changes the boundary's behavior, in Assumptions if it's a decision that applies more broadly than one task.
- Does this task write to more than one place as one logical action? Decide whether partial failure is acceptable, and note it if that isn't already obvious from the Workflow.

Most tasks are none of these — a task that reads one row once has nothing to flag here. Only raise it when a task genuinely has one of these shapes; asking about locking strategy for a simple read is the same kind of overthinking this whole kit exists to avoid.

### Writing a task's Workflow

The workflow is the sequence of what gets built, not how. Naming a specific file, class, or module is fine when it helps locate the work (e.g. "add a retry check in the payment handler") — but stop there. Don't describe the internals of a function, the exact library call, or a code snippet; that's implementation, and belongs to the execution step, not the plan. Mechanism choices (queue, protocol, sync or async) belong in the task's Decisions and Rules, not buried inside workflow steps, and the workflow should follow whatever they decided. Don't repeat Decisions or Rules in the workflow as steps' explanations.

Good:
> 1. Add a check before charging that looks up whether this payment already has a pending retry.
> 2. If a retry is scheduled, skip charging and log why instead.
> 3. Wire the webhook handler to clear the retry once a charge succeeds.

Bad (this is implementation, not a plan):
> 1. Add `if retry_queue.exists(payment_id): return` at the top of `charge()`.
> 2. Call `logger.info(f"skipped retry for {payment_id}")`.
> 3. In `handle_webhook()`, call `retry_queue.delete(payment_id)` inside the `charge.succeeded` branch.

If a task's workflow is ballooning into many steps that don't share one clear goal, that's usually a sign it's actually two tasks — split it. A task should be small enough that "is this task done?" has an obvious yes/no answer.

### Writing Rollout and open questions

Use this section for what the plan can't settle by itself but the reader must see: a stretch of time where something is unavailable while the change is live, how to roll back once data has been deleted, what has to ship together with another release, and what measurable condition says it is safe to remove a temporary fallback. Write each as a question marked `[open]` and say what depends on the answer. Don't answer them silently, and skip the section when there are none.

## Saving the plan

Get the real current date and time from the system (e.g. run `date +%Y-%m-%d-%H%M`) — never guess or estimate it.

Save the plan to:

```
plans/<YYYY-MM-DD-HHmm>-<slug>.md
```

- Create the `plans/` directory at the project root if it doesn't exist yet.
- `<slug>` is a short kebab-case name (3-6 words) describing what the plan is about, drawn from the Goal — not a generic name like `plan` or `update`. Example: `2026-08-26-1430-add-payment-retry.md`.

## After saving

Tell the user where the plan was saved, then give a short recap — the Overview and the task names/goals are usually enough, not the full file contents again since they can open it. Then stop. Don't move on to executing the plan or writing code until the user has reviewed it and says to proceed; they may want to reorder, merge, split, or cut tasks first, and building on top of an unapproved plan risks redoing the work.
