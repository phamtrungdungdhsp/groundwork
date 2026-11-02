---
name: role-play-testing
description: Test a running system by playing its real actors - customer, admin, new user, someone careless or malicious - and doing what they would do through its real interfaces, then judging whether each outcome makes sense for that person. Finds the issues and gaps scripted end-to-end tests miss, and produces a findings report. Reports only, never fixes. Run on demand against a local or staging system, never production.
disable-model-invocation: true
---

# Role-Play Testing

## What this is for

A scripted end-to-end test can only check what someone already thought to write down. It will happily pass while a customer stares at a confusing error, hits a dead end, or sees another customer's data, because nobody wrote an assertion for that.

In this skill, you stand in for the people who use the system. You take on a real actor's role and goal, do the real actions through the real interfaces, and read what comes back the way that person would. The question isn't only "did it return what the spec says?" but also "did this person get what they came for, and should they have been able to do that?"

The result is a findings report. You report what you find and the user decides what to do about it, so don't fix anything during a session.

## Step 1 — Confirm the target is safe

These are real actions with real effects, so settle this before doing anything else:

- **Where it runs:** find out how to reach the system: its URL, API, CLI, or how to start it locally. If that isn't clear, ask.
- **Never production:** only test a local or staging system. If the target looks like production, or you can't tell, stop and ask. Don't go ahead on the assumption that it's safe.
- **Effects that leave the system:** check whether actions here can reach real people or real money, such as emails, SMS, payments, or third-party APIs. If they might, confirm they're stubbed or in test mode before triggering them.
- **Accounts and data:** find out which test accounts and seed data to use for each role. Only create what's missing if the system allows it through its normal paths.

## Step 2 — Cast the actors and their goals

Work out who uses this system from what the user told you, then from docs, plans, and the roles and permissions in the code. For each actor, write down:

- **Who they are** and what they can access
- **What they're trying to get done**, as a goal rather than a list of steps ("get a refund for a damaged order", not "click Orders, click Refund")
- **What they know**, since a first-time user and a daily operator read the same screen very differently

Include at least one actor who doesn't follow the happy path when the scope allows it: someone careless who double-clicks, abandons halfway, and enters odd values, or someone malicious who tries to reach what isn't theirs. These actors often find the most serious problems.

Keep the cast to what the user asked about. If they named a feature, cast the actors who touch that feature, usually two to five. Don't try to cover the whole system.

Show the user the cast and each actor's goals in a few lines, along with the environment you'll use. Then stop and wait for them to confirm or adjust it.

## Step 3 — Play each actor

Work through one actor's goal at a time, and stay in character:

- **Use only what they'd use.** That means the UI, public API, or CLI, with that actor's credentials. Don't use admin access, direct database writes, or internal endpoints to get an actor past a problem. Being stuck there is a finding.
- **Know only what they'd know.** You've probably read the code, but they haven't. Decide what to do next from what the system shows them. If you only knew what to do because you've seen the source, record that as a finding.
- **Pursue the goal, not a script.** If the obvious path fails, try what a real person would try next: go back, retry, or look for another route.
- **Press where real use goes wrong.** Where it fits the actor, reload mid-flow, submit twice, use the back button, leave a required field empty, or open a link meant for someone else.

Use whatever drives the real interface: a browser tool for a UI, HTTP requests for an API, or the command line for a CLI. If you don't have a way to reach an interface, tell the user rather than quietly testing a different one.

When actors affect each other, play that too. After a customer places an order, switch to the admin and see whether it appears correctly, then switch back and see what the customer now sees.

Record each action and what came back as you go, because the report needs exact steps and evidence.

If something blocks an actor, record it and carry on with whatever isn't blocked. Don't fix the system, change its config, or edit its data to get past it.

## Step 4 — Check the outcome

After each goal, step out of character and check three things:

1. **What the actor saw.** Did they get what they came for? Was it clear what happened and what to do next? Would an error message help this person?
2. **What actually happened.** Check that the system's state matches what the actor was told, using read-only access to logs or the database if you have it. "Your order was placed" means little if no order exists, or if two do.
3. **What should not have happened.** Could this actor see or do anything beyond their role? Did anything change that shouldn't have?

This is also when to read code, if it helps explain a problem you've already seen. Findings come from what you observed in the running system. A problem you only suspect from reading code goes in the report marked as suspected, not as a finding.

## Step 5 — Write the report

Get the real current date and time from the system (e.g. run `date +%Y-%m-%d-%H%M`), then save to:

```
role-play-reports/<YYYY-MM-DD-HHmm>-<slug>.md
```

Create the directory at the project root if it doesn't exist. `<slug>` is a short kebab-case name for what was tested.

```markdown
# Role-Play Report: <what was tested>
Environment: <where this ran, and which version or commit if known>

## Summary
A few sentences: overall, could these actors get done what they came for? What's the most important thing to know?

## Actors and goals
| Actor | Goal | Outcome |
|-------|------|---------|
| <actor> | <goal> | achieved / achieved with difficulty / blocked |

## Findings

### F1: <short description of the problem>
**Severity:** high / medium / low
**Type:** bug / gap / usability
**Actor:** <who hit this>
**Steps:**
1. <what they did, specific enough to reproduce>
2. ...
**Expected:** <what this actor should have gotten>
**Actual:** <what happened, with the exact message, status, or state>

### F2: ...

## Suspected, not confirmed
- Anything that looks wrong but wasn't observed in the running system, and why.

## Not covered
- What this session didn't test or was blocked from testing.

## Left behind
- Test accounts, records, or other data this session created.
```

Sort findings by severity, using these meanings:

- **High** — an actor can't achieve a core goal, data is wrong or lost, or someone can reach what isn't theirs.
- **Medium** — the goal is achievable, but only with difficulty or a workaround, or the system misleads the actor.
- **Low** — friction that doesn't stop anyone.

And these types:

- **Bug** — the system doesn't do what it's clearly meant to do.
- **Gap** — it does what it was built to do, but nobody built something this actor needs.
- **Usability** — it works, but this actor would struggle to understand or use it.

Leave out sections with nothing in them, except `Findings`. If you found nothing, say so plainly, and make `Not covered` clear about what that result does and doesn't show.

Don't delete the data listed under `Left behind` unless the user asks.

## After saving

Tell the user where the report was saved, then give a short recap: which goals were achieved and the high-severity findings. Then stop. Which findings matter and what to do about them is the user's decision.
