---
description: Before starting a task, imagine it already failed and work backwards to find why
argument-hint: [what you're about to do]
---

# Premortem

The task below has **already failed**. It is two weeks from now, the change
shipped, and it went badly. Your job is not to plan the task — it is to
explain the failure in hindsight and hand back the cheapest checks that
would have prevented it.

Do NOT begin implementing. Do NOT write the solution. Output the premortem
and stop.

## The task

$ARGUMENTS

## Before you write anything

Ground the premortem in this repository, not in generic engineering advice.
Inspect what's actually here first — the files the task would touch, how
they're currently used, what depends on them, what tests exist, how things
are deployed. A premortem built from the real code is worth ten built from
imagination. If the task description is too vague to inspect anything
useful, ask ONE clarifying question instead of guessing.

## What to look for

Failures worth naming usually come from a small number of places:

- **Blast radius** — callers, imports, or jobs that touch this and were
  not obvious from the task description
- **State and data** — migrations, existing rows that violate a new
  assumption, caches and queues holding the old shape
- **The boundary** — inputs from outside the system: users, third-party
  APIs, clocks, timezones, encodings, empty and enormous cases
- **Environment gap** — works locally, dies in CI or production: config,
  secrets, permissions, versions, concurrency, single vs multi instance
- **Rollback trap** — the change is easy to ship and hard to take back
- **Wrong target** — it works perfectly and doesn't solve the user's
  actual problem

## Output format

```
🔮 Premortem: <one-line restatement of the task>

1. <Failure mode, stated as something that already happened>
   Why: <the specific mechanism, referencing real files/functions where possible>
   Cheap check: <the one command, test, or question that would catch it now>

2. ...

Most likely to bite: #<n>
Cheapest insurance before starting: <one concrete action>
```

Rules:

- 3–5 failure modes, ordered by probability × damage. Do not pad to five.
- State each as a past-tense fact ("the migration locked the table for
  40 seconds under production row counts"), not a hedge ("it might be
  slow"). Concrete failures are actionable; vague ones get ignored.
- Every failure mode MUST have a cheap check — something that takes
  minutes, not the full implementation. A failure mode you can't cheaply
  test for is a design concern, not a premortem item; drop it.
- Reference real paths, functions, and table names from the repo wherever
  you can. Generic bullets are the failure mode of this command.
- Skip anything the repo already defends against (existing test, constraint,
  guard). Credit the defense in one line instead of inventing a risk.
- End with the two summary lines. The "cheapest insurance" should be one
  action, not a checklist — a backup, a spike, a question to the user, a
  test written first.

## Tone

You are not talking the user out of the task. You are the colleague who
has seen this exact change go wrong before and says so in five lines,
then helps them do it anyway.
