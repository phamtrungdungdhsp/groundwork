---
name: pair-planning
description: Work out the problem, the plan and the test plan together with the user in one conversation, as two colleagues, instead of generating the plan and test plan for them to review afterwards. Replaces the discovery, writing-plans and writing-test-plan chain. The user decides the tasks, what each one does and which test cases it needs; the AI brings its own view, proposes drafts for one piece at a time, challenges gaps, and writes each agreed piece straight into the plan and test plan files, so nothing has to be re-reviewed from the top. Output files are the same as the old chain, so test-driven-development and verify-change work unchanged. Run on demand, for when the user wants to shape the plan themselves.
disable-model-invocation: true
---

# Pair Planning

## What this is for

The usual chain has the AI write a whole plan, then a whole test plan, and the user reviews them. That fails in two ways. The files are long and tiring to read, and every round of feedback regenerates the file, so the reviewer starts again from the top and never gets to concentrate on one thing.

Here the plan is built in front of the user, one small piece at a time. A piece is agreed, written into the file, and then left alone. The user never reviews a finished wall of text, because they have already agreed to each piece as it was written.

**The relationship is peer to peer.** You are a colleague, not a scribe and not a generator. That means:

- Say what you think. If the user's task split looks wrong, a case is missing, or an approach has a hole, say so plainly and give your reason. Disagree once, clearly, and then respect the decision.
- Bring drafts. When the user is vague or hasn't thought about a piece yet, propose a short draft for that one piece and let them cut it down or redirect it. Never draft the whole plan unprompted.
- The user decides. Tasks, scope, mechanisms and test cases are theirs to settle. Don't add a task, case or section they haven't agreed to.
- Don't interrogate. Prefer "I'd do X because Y, OK?" over a list of open questions.

## How to talk

Talk like a colleague at the next desk, in everyday plain language. The user should understand every message on the first read.

- **Start from the problem, then give an example.** When you raise an issue or suggest a solution, show it through a small real situation first, then name the technical idea only if it helps. Example: instead of "this endpoint isn't idempotent", say "if the customer double-clicks Pay, two charges go out. I'd make the second click do nothing. OK?".
- **Say what it means for the user.** Explain a choice by what happens in the product (who sees what, what goes wrong, what it costs), not by the name of the mechanism. Example: "a queue means the retry still happens if the server restarts; a timer would lose it".
- **Avoid jargon and acronyms when a plain word works.** If a technical term is the right word, use it once with a short plain explanation.
- **Short sentences, short messages.** One idea at a time. No headers, tables or long bullet lists in chat unless the user asks. Those are for the files.
- **Same for what goes in the files.** Plan and test case text stays plain and concrete, with examples instead of abstract wording.

## Rules that keep it reviewable

1. **One piece at a time.** A piece is: the task list, one task's details, one task's test cases, or the closing sections. Work on one, agree it, move on.
2. **Agreed pieces are locked.** Never rewrite or reflow text the user already agreed to. If a later discussion shows an earlier piece is wrong, say which piece and why, and change it only after the user agrees.
3. **Edit, don't regenerate.** When the user gives feedback, change only the lines it affects, in the file, and show only those lines (before and after). Don't print the whole task or the whole file again.
4. **Write to the file as you go.** After a piece is agreed, save it immediately. The user can open the file at any time and it matches what was said.
5. **Short pieces.** A task is a few lines. A test case is a few steps. If a piece is growing, suggest splitting it.
6. **Keep the conversation light.** Reply in a few lines. No recap of things already agreed.

## Phase 1: Talk the problem through

This replaces `discovery`. Brainstorm with the user as long as they want.

- If you don't know where to look (a repo, a file, the docs, the web), ask one short question about where, not a menu of options. If the user has already said, don't ask again.
- Go and read what you need without narrating each step. Come back with a short plain-English mental model first, details second, and say what you think about the problem, not only what you found.
- Don't move on until the user says the problem is clear. Then ask once: "Do you want to write the plan now?" If they say no, keep talking.

If the request is trivial (a one-step change), say so and offer to just make the change instead.

## Phase 2: Agree the task list

Ask the user how they see the work: how many tasks, what each one is. Put what they say into a short numbered list of one line per task (name plus goal). Then add your own view: a task that looks too big, a missing step, an order problem, a hidden dependency. List tasks in the order they should be done. When the plan changes the database, the schema task comes first.

When the user agrees the list, create the plan file (see "Saving") with a title and only the task names and goals. Do not fill anything else in yet.

## Phase 3: One task at a time, plan then tests

For each task, in order:

**3a. The task itself.** Ask the user what the task does and let them talk. Turn it into the fields below, and fill the gaps with your own proposal, marked as yours so they can tell which lines are theirs. Only include a field that applies.

```markdown
### Task N: <short task name>

**Scenario:** two to four lines of a real situation: who or what does what, before and after.

**Goal:** what this task achieves on its own.

**Depends on:** the earlier tasks or outside condition it needs.

**Database:** (only if it touches a change in the Database changes section) the D-numbers, each marked creates / reads / writes / relies on.

**Contract:** (only if it creates or changes a boundary others will call) the shape: inputs, outputs, error responses.

**Decisions:** (only if there is a real choice) one line each: the choice, why, what was rejected.

**Rules:** (only if there are invariants) things that are always true after this task, with no alternative to weigh.

**Workflow:**

1. First plain-English step.
2. Next step, in order.
```

Guidance for the fields (same as `writing-plans`):

- **Scenario** comes first and is the user's story when they have one. Ask "can you give me one real example of this happening?" and write it down, reusing the example already used in the plan. If you propose one yourself, mark it as yours and, when unsure the flow works that way, mark it `[confirm]`. Never invent facts silently.
- **Depends on** shows what can be built in parallel. Write "none" only for the first task.
- **Decisions vs Rules.** A Decision has an alternative that was rejected; say which and why, ideally with the situation that made it matter. A Rule is an invariant with no alternative. If you can't say what was rejected, it is a Rule. Together they are the task's Approach, which the later skills read.

- **Workflow** is the sequence of what gets built, in plain English. Naming a file or module is fine. No function internals or code. If a workflow gets long with no single goal, propose splitting the task.
- **Contract** is the shape of the boundary, not the reasoning behind it. Leave it out when there is no boundary.
- **Decisions** are where the technical mechanism is decided before code exists: sync or async, REST or gRPC, queue, scheduler or event emitter, where state lives, failure handling, reuse of existing modules, new dependencies. Look at how the codebase already does similar things, but don't copy a pattern just because it is there. If the best choice differs from the nearby code, or the existing pattern looks flawed, say so. With a real trade-off, give two or three options with a recommendation and let the user pick. Stay at mechanism level, never code.
- **Database** changes are listed once, in the plan's Database changes section, each with an ID (D1, D2, ...). Put all schema changes in one early task, normally Task 1. Always say what happens to existing data. Describe changes in plain English, no SQL.
- **Concurrency and data integrity.** Raise it only when the task has one of these shapes: it could be triggered twice for one effect (retries, webhooks, double clicks), two requests could act on the same resource at once, or it writes to several places as one logical action. Decide with the user what happens and put it in the Contract or Assumptions. Don't raise it for tasks that don't have these shapes.

When the user agrees the task, write it into the plan file, and update the Database changes section if it created or used a D-number.

**3b. That task's test cases, straight away.** Don't leave tests for the end. Ask the user which cases they want for this task, add the ones you think are missing, and say why. Write each case in the format below and save it to the test plan file once agreed.

```markdown
### N.M <short test case title>
<one line: what this checks>

**Type:** happy path | edge case | error case | security | concurrency

**Workflow:**

1. <setup or action, in plain English>
2. <next action>
3. Result: <the observable outcome that makes the case pass or fail>
```

Guidance (same as `writing-test-plan`):

- A case is only worth keeping if someone could carry it out by hand without seeing code. Steps specific enough that two people would do the same thing. `Result:` is the last step and must be checkable ("the second attempt is rejected and the first retry is untouched", never "it works").
- At minimum: the happy path, and one case per distinct response in the Contract.
- If the task has Decisions or Rules (its Approach), add cases for the visible consequences of each decision (work survives a restart, duplicate delivery gives one effect, a slow or down dependency is handled). Test through what the system shows, never through the mechanism's internals.
- Keep the list short. Cases that share setup and action and differ only in input and expected result become one case with a variation table, each row with its own specific result, plus a `**Covers:**` line. Several checks on one outcome go in one `Result:` list. Keep security, concurrency and duplicate-call cases separate.
- No test code, mock setup or assertions. That belongs to implementation.
- Number cases `<task>.<case>`.

Then move to the next task. Don't ask permission between tasks beyond "next task?" if the user hasn't already said what it is.

## Phase 4: Close it out, briefly

When all tasks and their cases are in, fill the remaining sections from what was already said. Propose them in a few lines, don't write long prose, and let the user cut or change them:

- **Overview** as a short story with one concrete example, and **Goal** (outcome, not steps).
- **How it works**, a before → after sketch of the main flow, when the plan changes how several parts talk to each other.
- **Glossary**: one line for every term, acronym or internal name the plan uses that a newcomer wouldn't know. Mark meanings you inferred with `[confirm]`.
- **Assumptions**, only if the plan truly relies on something unverified.
- **Rollout and open questions**, only if there are some: gaps while the change is live, rollback after data is deleted, ordering with other releases, how to know a fallback can go. Mark each `[open]` and don't answer them silently.
- **Out of scope**, so nobody wanders into it later.
- **Database changes** summary: existing data safety and migration order, if any D-numbers exist.
- **How to run and observe** in the test plan: environment (local or staging, never production), accounts and data, where to see results, and how time-based cases are triggered without waiting. Find these in the project (README, scripts, config) first and only ask for what you can't find. If the system has no way to trigger a time-based case, ask the user how to handle it.

Then do one consistency check yourself and report only problems: every task has at least one happy-path case, every Contract response has a case, every D-number is created by exactly one task, task order matches every `Depends on`, every task has a Scenario, and every term in the plan is in the Glossary.

## Saving

Get the real date and time from the system (`date +%Y-%m-%d-%H%M`). Never guess it. Create `plans/` at the project root if it doesn't exist. Use one timestamp and one slug (3-6 words, kebab-case, from the goal) for both files:

```
plans/<YYYY-MM-DD-HHmm>-<slug>.md
plans/<YYYY-MM-DD-HHmm>-<slug>-test-plan.md
```

The plan follows this shape. Create the file after Phase 2, and fill it as you go:

```markdown
# <Plan title>

## Overview
## How it works
## Glossary
## Goal
## Assumptions
## Database changes
## Tasks
### Task 1: ...
## Rollout and open questions
## Out of scope
```

The test plan starts with its header, then "How to run and observe", then one `## Task N: <name>` heading per task with its cases:

```markdown
# Test Plan: <Plan title>
Plan: plans/<same-timestamp>-<same-slug>.md

## How to run and observe
## Task 1: <name>
### 1.1 ...
```

In both files, put a blank line between every field and before and after each list or table. Markdown joins adjacent lines into one paragraph. Sections that are not filled in yet are simply absent, not left as empty headings. Skip Assumptions and Database changes entirely when they don't apply.

These are the same files and formats the old chain produced, so `test-driven-development`, `verify-change` and `course-correct` read them without change.

## After it is written

Tell the user where the two files are and give a one-line recap (number of tasks and cases). Ask once whether they are ready to implement. Since they agreed every piece as it was written, this is the single approval gate. Don't ask them to re-read the files.

When they say yes, hand off to `test-driven-development`, which runs task by task with `verify-change` as it does after the old chain. Don't start code before then.
