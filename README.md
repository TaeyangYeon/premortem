# 🔮 premortem

A Claude Code plugin that assumes your task **already failed** — and works
backwards to tell you why, before you write a line.

A postmortem explains the outage you already had. A premortem is the same
exercise moved to the only point where it's still cheap: the beginning.
Instead of asking "what could go wrong?" (which produces polite, generic
lists), you assert the failure as fact and force an explanation. People are
dramatically better at explaining a failure than predicting one.

`/premortem` runs that exercise against **your actual repository** — real
file paths, real callers, real tests — and hands back each failure mode
with the one cheap check that would have caught it.

## Example

```
/premortem add a last_seen timestamp to users, updated on every login
```

> 🔮 Premortem: adding a per-login `last_seen` write to the users table
>
> 1. The login path got 3x slower under load
>    Why: `auth/login.py` already writes the session row; a second UPDATE on
>    `users` puts two writes in the same request, and `users` is the hottest table
>    Cheap check: time 200 logins against a copy with the extra UPDATE in place
>
> 2. The migration locked the table in production
>    Why: `ALTER TABLE users ADD COLUMN ... DEFAULT now()` rewrites every row;
>    dev has 40 rows, production has millions
>    Cheap check: `SELECT count(*) FROM users` on prod, then test the DDL on a restore
>
> 3. It shipped and answered the wrong question
>    Why: "last seen" from login only counts sessions, not activity — an API-token
>    client stays "inactive" forever
>    Cheap check: ask whether inactive-user emails should count token usage
>
> Most likely to bite: #2
> Cheapest insurance before starting: run the DDL against a production restore first

Three minutes of reading. The alternative is finding out #2 during a
deploy window.

## Install

From a marketplace that includes this repo:

```
/plugin marketplace add TaeyangYeon/premortem
/plugin install premortem@premortem
```

Or drop the command in directly without the plugin system:

```bash
mkdir -p ~/.claude/commands
curl -o ~/.claude/commands/premortem.md \
  https://raw.githubusercontent.com/TaeyangYeon/premortem/main/commands/premortem.md
```

## Usage

```
/premortem <what you're about to do>
```

Best used right before anything you'd hesitate to revert: schema changes,
auth, payments, deletion logic, deploy config, anything touching a hot path.
Overkill for a typo fix.

Claude inspects the relevant code first, then outputs:

- **3–5 failure modes**, ordered by probability × damage, each stated as
  something that *already happened* — past tense, specific mechanism, real
  file paths. Vague hedges are explicitly banned.
- **A cheap check per failure** — minutes, not a full implementation. If a
  risk can't be cheaply tested, it gets dropped rather than padding the list.
- **Existing defenses get credit** — if a test or constraint already covers
  it, Claude says so instead of inventing a risk.
- **Two closing lines** — which one is most likely to bite, and the single
  cheapest piece of insurance to buy before starting.

Then it stops. It does not start implementing.

## The X-ray series

Sixth in a set of single-file plugins that make Claude Code's work
inspectable — and the first that runs *before* the work instead of after:

| Plugin | Question it answers | When |
|--------|--------------------|------|
| **premortem** | How is this about to go wrong? | before |
| [token-breakdown](https://github.com/TaeyangYeon/token-breakdown) | Where did the tokens go? | after |
| [assumption-log](https://github.com/TaeyangYeon/assumption-log) | What was assumed but not verified? | after |
| [road-not-taken](https://github.com/TaeyangYeon/road-not-taken) | What was considered but not chosen? | after |
| [best-before](https://github.com/TaeyangYeon/best-before) | What knowledge may have expired? | after |
| [undo-plan](https://github.com/TaeyangYeon/undo-plan) | What changed, and how do I take it back? | after |

The other five are skills that attach themselves to responses automatically.
This one is a slash command on purpose: a premortem you didn't ask for is
noise, and it's worth reading precisely because you chose to run it. Pairs
naturally with [undo-plan](https://github.com/TaeyangYeon/undo-plan) —
premortem tells you what to fear going in, undo-plan tells you how to get
back out.

## Structure

```
premortem/
├── .claude-plugin/
│   └── plugin.json
├── commands/
│   └── premortem.md
└── README.md
```
