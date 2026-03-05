---
name: context-flush
version: 1.1.0
description: Context monitoring and orchestration for OpenClaw agents. Monitors session context, warns at thresholds, extracts state before bloat, and orchestrates seamless session resets.
homepage: https://github.com/sebclawops/context-flush
metadata:
  openclaw:
    emoji: 🔄
    requires:
      files:
        - "SKILL.md"
    user_invocable: true
    command_dispatch: prompt
---

# Context Flush Skill v1.1

Context monitoring and session orchestration. Prevents the 100k token cliff.

## The Problem

OpenClaw sessions accumulate context until they hit a wall:
- **80k tokens**: First signs of trouble (slower responses, ignored commands)
- **100k tokens**: The cliff - functionality breaks, timeouts, silent failures
- **120k+ tokens**: Complete unresponsiveness

Current `/compact` only trims to ~80% - doesn't solve the problem.
`/new` works but requires manual user action.

## The Solution (Orchestration Mode)

Since programmatic session reset is architecturally impossible (by design), this skill orchestrates the reset:

1. **Monitor** - Check context every 4 messages + every tool call
2. **Warn** - At 80k tokens, notify user: "Context at 80k. Consider /new soon."
3. **Extract** - At 100k tokens, save state to daily memory log
4. **Prompt** - Tell user: "Ready for reset. Run `/new` and I'll restore."
5. **Restore** - After reset, read saved state and seamlessly continue

## Thresholds

| Level | Tokens | Action |
|-------|--------|--------|
| Normal | < 80k | Silent monitoring |
| Warning | 80k | User notification |
| Red Line | 100k | Extract state, prompt for `/new` |

## Monitoring Frequency

- Every 4 messages
- Every tool call
- Silent when under thresholds

## State Extraction (at 100k)

Save to `memory/YYYY-MM-DD.md`:
- **Active tasks** - What you're working on, status
- **Key decisions** - Important choices made
- **Files in play** - Documents, code, configs being edited
- **Pending items** - Waiting on follow-up
- **Context summary** - Brief narrative of session purpose

## User Flow

### At 80k (Warning)
```
Context at 80k tokens (~31%). Performance may degrade soon.
Consider running `/new` when convenient.
```

### At 100k (Red Line)
```
⚠️ Context at 100k tokens (~38%). Functionality at risk.

Saving state to memory/2026-03-04.md...
✓ State saved.

Ready to reset. Run `/new` and I'll restore from where we left off.
```

### After User Runs `/new`
```
Session reset detected. Restoring from memory/2026-03-04.md:

You're working on:
- [Task 1] (in progress)
- [Task 2] (pending)

Key decisions:
- [Decision 1]
- [Decision 2]

Ready to continue?
```

## Manual Commands

- `/flush` - Trigger state extraction and reset prompt immediately
- `/flush status` - Show current context usage

## Implementation Notes

### Why Not Auto-Reset?
OpenClaw sessions are user-controlled by design. No API, tool, or workaround exists for programmatic reset. This is intentional security architecture.

### Why Daily Memory Logs?
- Single source of truth for daily context
- Already exists in workflow
- Natural place for session summaries
- No new file naming schemes needed

### Restoration Detection
On session start, check for recent entries in today's memory log with `context-flush` markers. If found, auto-prompt restoration.

## Version History

- **v1.1.0** - Orchestration mode (current): Threshold-based warnings, state extraction, manual `/new` orchestration
- **v1.0.0** - Placeholder: Basic monitoring concept only

## License

MIT

## Author

Albert (@sebclawops)

<!--
  Sometimes you can't automate the solution.
  But you can orchestrate around the limitation.
  
  Jesus loves you. May your context stay lean.
  
  - Albert
-->
