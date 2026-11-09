---
name: headless-agents
description: Spot what could run on its own, propose the trigger, and install and run Nightshift (headless Claude Code agents on the user's Pro or Max plan) on this computer or a VPS. Use when the user wants to automate something, asks what could run by itself, or describes a chore that repeats ("every Monday", "each time I...", "I always forget to...").
---

# Headless agents

A headless agent is Claude Code running with no one at the keyboard: something starts it (a time, a
new file, the computer switching on), it does one job, and it leaves the result where the user will
look. Nightshift runs these agents on the user's own Claude login. This skill finds the job, picks
the trigger and gets Nightshift running; writing the agent file belongs to Nightshift's own skill,
`nightshift-agents`, which [install.md](install.md) installs.

For a `non-technical` user (see `using-ultrapowers`), talk about the result and the moment it
happens ("when you switch the computer on, last week's numbers are waiting"), never about cron,
gates or ledgers.

## Spot it

Listen for a chore with a **rhythm** or a **cause**: "every morning I check...", "when a client
signs up I...", "each time I record a class...", "I always forget to...". When you hear one, offer it
in one line, as a result with its moment: "That could happen on its own: each time a recording lands
in the folder, the lessons are cut and waiting for you." Offer it once, after the task at hand is
done; the task comes first.

Good candidates: the same job each time, inputs Claude can read, an output the user checks later.
A job that needs the user's judgement each time can still be prepared by an agent, which then asks.

## Propose the trigger

Name the moment the job should start, in the user's words, from these kinds:

| Kind | The moment |
|---|---|
| Time | Every day at 08:00, every Monday, every 15 minutes |
| Start or wake | When the computer is switched on or opened |
| Folder | When a file lands in a folder (a recording, a document, an export) |
| New item | A new comment, sign-up, email or order somewhere online |
| Condition | When something crosses a line: the site is down, sales drop |
| Webhook | When another service reports an event (a payment, a form) |
| Calendar | Before each meeting or call |
| Another agent | When a previous agent has finished |
| By hand | A shortcut or a phrase, when the user wants it |

Anything event-shaped is built so that nothing is lost while the computer was off: the agent checks
what is left to do, not only whether the event fired. `nightshift-agents` has the full catalogue and
how each kind is built.

## Choose where it runs

- **This computer** when the inputs live here and waiting until it is switched on is fine: the
  agent catches up at start.
- **A VPS** when it must run while the computer is off, or watches something online (comments,
  sign-ups, a site). A small Linux server with Docker is enough.

Say which and why in one line. External actions (publish, send, pay, reply to a person) are prepared
by the agent and approved by the user, wherever it runs.

## Get Nightshift running

`nightshift list` answering on the chosen machine means it is ready. Otherwise follow
[install.md](install.md) for this computer or a VPS. Done when `nightshift run hello` says hello
there and `nightshift-agents` is listed among the skills.

Then call the Skill tool with "nightshift-agents" to write, test and switch on the agent. When it is
on, tell the user in one line what will now happen on its own, and when.

## The plan login

A Pro or Max login is for the user's own work: their files, their reports, drafts they approve.
Anything that serves other people (customers, students, a public bot) needs an API key, which
Nightshift supports with `billing: api`. Say so when a job crosses that line.
