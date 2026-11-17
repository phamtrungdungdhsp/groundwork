# groundwork

A personal Claude Code and Codex skill kit for programming work — built to replace the `superpowers` plugin.

Six skills chained together, plus two standalone skills you run on demand.

## The chain

- `feature-development` — entry point. Figures out whether a request is new work or a continuation of something already in progress, then routes into the rest.
- `discovery` — clarifies an ambiguous request: confirms where to look for missing context, then reports back a plain-English mental model before any technical detail.
- `writing-plans` — turns an understood problem into a short plan (`plans/<timestamp>-<slug>.md`): overview, goal, assumptions, per-task goal/contract/approach/workflow, out of scope. The per-task approach records the technical mechanism chosen (sync vs async, REST vs gRPC, queue vs event emitter, what to reuse) so you can review it before any code exists.
- `writing-test-plan` — turns an approved plan into a plain-English test plan (`plans/<timestamp>-<slug>-test-plan.md`), one concrete step-by-step scenario per test case.
- `test-driven-development` — implements from the test plan: red → green → refactor, tests traceable to test-plan IDs, code placed in the right architectural layer (business vs infrastructure), with a checklist for concurrency/scale correctness (N+1, race conditions, atomicity, consistency, idempotency, timeouts, unbounded queries, authorization). It follows the plan's approach, and once the test plan is approved it keeps going task by task on its own.
- `verify-change` — runs after each task. Independently carries out each test-plan case against the real running system through its real interface, and saves evidence per case (pass / fail / not verified) to `plans/<timestamp>-<slug>-verification.md`, which is what you review instead of the diff.

## On demand

These aren't part of the chain and Claude won't start them by itself. Run them when you need them:

- `/groundwork:pair-planning` — an alternative to the `discovery` → `writing-plans` → `writing-test-plan` steps, for when you want to shape the plan yourself. You talk the problem through with Claude, then say what the tasks are, what each does and which test cases it needs. Claude acts as a peer: it gives its own view, drafts one piece when you're unsure, and writes each agreed piece straight into the plan and test plan files, without rewriting what you already agreed. It produces the same two files as the chain, so you approve once and `test-driven-development` and `verify-change` take over.
- `/groundwork:course-correct` — for a small correction to work that already has a plan. Leaves the original plan and test plan untouched and writes a short correction plan and correction test plan that point back to them and cover only what changes. You approve the plan, then the test plan, and then implementation and verification run on their own, the same as `feature-development`.
- `/groundwork:role-play-testing` — plays the system's real actors against a running local or staging system, then writes a findings report (`role-play-reports/<timestamp>-<slug>.md`). It reports only and never fixes anything.

## Install on a new machine

### Claude Code

```bash
# one-time per machine
/plugin marketplace add git@github.com:phamtrungdungdhsp/groundwork.git
/plugin install groundwork@groundwork
```

### Codex

```bash
# one-time per machine
codex plugin marketplace add phamtrungdungdhsp/groundwork
```

Then open the Plugins directory in the Codex desktop app, choose the `groundwork` marketplace and install `groundwork`.

The skills are shared: both tools read the same `plugins/groundwork/skills/` folder. Keep skill text tool-neutral (don't name Claude-only tools such as `AskUserQuestion` or the `Skill` tool). `disable-model-invocation` in the frontmatter of the on-demand skills is a Claude Code setting that Codex may ignore.

## Update after editing a skill

Push your changes to the git remote, then on each machine:

```bash
# Claude Code
/plugin marketplace update groundwork
/plugin update groundwork

# Codex
codex plugin marketplace upgrade
```

(exact update subcommands may vary by version — check `/plugin help` or `codex plugin marketplace --help` if these don't match)

## Repo layout

```
.claude-plugin/marketplace.json   <- Claude: lists the plugin below
.agents/plugins/marketplace.json  <- Codex: lists the plugin below
plugins/groundwork/
  .claude-plugin/plugin.json      <- Claude manifest
  plugin.json                     <- Codex manifest
  skills/
    feature-development/SKILL.md
    discovery/SKILL.md
    writing-plans/SKILL.md
    writing-test-plan/SKILL.md
    test-driven-development/SKILL.md
    verify-change/SKILL.md
    pair-planning/SKILL.md
    course-correct/SKILL.md
    role-play-testing/SKILL.md
```

To add a new skill to the kit later: create `plugins/groundwork/skills/<name>/SKILL.md`, bump the `version` in `plugins/groundwork/.claude-plugin/plugin.json`, `plugins/groundwork/plugin.json` and `.claude-plugin/marketplace.json` (keep all three equal), commit, push, and run the update commands above on each machine.
