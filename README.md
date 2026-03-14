# context-flush

Lean session reset playbook for OpenClaw.

## Purpose

This skill is for one job: helping an agent recover cleanly when a session gets bloated.

It does not pretend to auto-monitor, auto-reset, or control the runtime. It gives the agent a real workflow that matches how OpenClaw actually works today.

## What this skill does

- checks current session context on demand
- decides whether to keep going, compact, reset, or use dashboard delete as fallback
- saves a short handoff note before reset
- restores from that handoff note after reset
- keeps the process simple enough to trust under pressure

## What it does not do

- no fake auto-monitoring
- no made-up `/flush` command
- no pretending the agent can force `/new`
- no magic buttons unless the current channel actually supports them and the flow is confirmed

## Recommended workflow

### 1. Check context
Use `session_status` when the conversation feels slow, large, or risky.

### 2. Choose the move
- **Keep going** if context is still comfortable
- **Use `/compact`** if a light trim is enough
- **Use `/new`** when context is heavy and a real reset is the right move
- **Use dashboard delete** only as fallback for stuck or broken sessions

### 3. Save a handoff note before reset
Write a short note to the agent workspace or daily memory file with:
- active task
- current status
- key decisions
- files in play
- exact next step

### 4. Reset
The user runs `/new`.

### 5. Restore
On the fresh session, read the handoff note and resume fast.

## Handoff note format

Keep it short and useful:

```markdown
# Context Flush Handoff - 2026-03-14 13:05 ET

## Active task
- Clean up context-flush skill and sync GitHub repo

## Current status
- presidio-pii synced to GitHub
- context-flush being rewritten as honest manual playbook

## Key decisions
- do not fake automatic monitoring
- use session_status on demand
- use /new for real resets

## Files in play
- workspace/skills/context-flush/SKILL.md
- workspace/skills/context-flush/README.md

## Next step
- finish SKILL.md rewrite, then commit and push to sebclawops/context-flush
```

## Threshold guidance

This skill uses practical guidance, not fake precision.

### Small-window models
For models around 32k context, get nervous around 20k to 24k.

### Large-window models
For large hosted models, performance can still get sloppy well before the advertised limit. Treat roughly 80k to 100k as the zone where reset planning starts.

## Notes

- `/compact` is a trim, not a true reset
- `/new` is the clean reset path when available
- in some channels or stuck sessions, dashboard delete may still be the only reliable fallback
- this skill should stay honest, boring, and operationally useful
