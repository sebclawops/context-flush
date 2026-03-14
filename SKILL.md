---
name: context-flush
version: 2.0.0
description: Honest session reset playbook for OpenClaw. Check context on demand, save a handoff note, reset cleanly, and restore fast.
metadata:
  openclaw:
    emoji: 🔄
    requires:
      files:
        - "SKILL.md"
    user_invocable: true
    command_dispatch: prompt
---

# Context Flush

Use this skill when a session is getting bloated, slow, unreliable, or close to the context cliff.

## What this skill is

A manual recovery playbook.

It does **not** auto-monitor the session, auto-run resets, or invent commands that OpenClaw does not support.

## When to use it

Use it when:
- replies are slowing down
- context feels messy or fragile
- `session_status` shows heavy usage
- `/compact` is no longer enough
- you want to save state before `/new`

## Core workflow

### 1) Check context

Use `session_status` to inspect the current session.

Interpret results practically:
- **Small local models, around 32k context:** be cautious around 20k to 24k
- **Large hosted models:** start planning a reset somewhere around 80k to 100k even if the model advertises much more

Do not pretend precision where none exists. Use judgment.

### 2) Choose the right move

Pick one:

- **Keep going** if the session is still healthy
- **Use `/compact`** if a light trim is enough
- **Use `/new`** if you need a real reset
- **Use dashboard delete** only as fallback for stuck or broken cases

## Important reality checks

- `/compact` reduces context but does not create a clean slate
- `/new` is the real reset path when available
- the agent cannot force a reset programmatically
- do not invent commands like `/flush`
- do not claim automatic monitoring is happening unless an actual scheduler or runtime feature exists

### 3) Save a handoff note before reset

Before asking for `/new`, save a short handoff note in the workspace or daily memory file.

Include only what matters:
- active task
- current status
- key decisions
- files in play
- exact next step

Good handoff notes are short enough to read in one glance.

### 4) Ask for the reset

Tell the user clearly what to do:
- run `/new` for a clean reset
- use dashboard delete only if the session is stuck and `/new` or `/compact` is not working

Keep it direct.

### 5) Restore after reset

On the fresh session:
- read the handoff note
- restate the active task and next step
- continue without making the user re-explain everything

## Handoff template

```markdown
# Context Flush Handoff - YYYY-MM-DD HH:MM TZ

## Active task
- ...

## Current status
- ...

## Key decisions
- ...

## Files in play
- ...

## Next step
- ...
```

## Style rules

- be honest about what the runtime can and cannot do
- keep the handoff short
- optimize for recovery speed, not theory
- boring is good

## Recommended user-facing language

### When context is getting heavy
"This session is getting bloated. I can keep going a bit, but we should plan a reset soon."

### When reset is the right move
"This session is too heavy. I saved a handoff note. Run `/new` and I’ll pick up right where we left off."

### When dashboard delete is needed
"`/new` is not the cleanest option here. If the session is stuck, use the dashboard delete workaround, then start fresh and I’ll restore from the handoff note."

## Anti-patterns

Do not:
- claim the skill monitors every N messages unless something actually does
- claim hard token thresholds are exact science
- promise one-tap reset flows unless verified on the current channel
- bury the next step in a giant summary

This skill exists to make resets clean, not clever.
