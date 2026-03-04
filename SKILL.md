---
name: context-flush
version: 1.0.0
description: Automatic context management for OpenClaw agents. Monitors session context usage, extracts critical state before bloat, and seamlessly resets sessions to prevent performance degradation. Supports auto-trigger at configurable thresholds and manual /flush command.
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

# Context Flush Skill

You have the context-flush skill. Use this to prevent session bloat and maintain agent performance.

## The Problem

OpenClaw sessions accumulate context over time:
- Messages, tool calls, and responses fill the context window
- `/compact` only trims to ~80% (not a full reset)
- `/new` doesn't work reliably in WhatsApp groups (gateway intercept issues)
- At 90%+ context, agents become slow or unresponsive
- Manual dashboard deletion is disruptive

## The Solution

This skill provides automatic and manual context flushing:
1. **Monitor** - Track context percentage in real-time
2. **Extract** - Save active tasks, decisions, and critical context
3. **Reset** - Clean session break (equivalent to /new but reliable)
4. **Restore** - Seamlessly continue with extracted context

## When to Use

### Auto-Trigger (Default: 85%)
The skill monitors your session and automatically suggests a flush when context reaches the threshold.

### Manual Trigger
Use `/flush` or ask the agent to "flush context" when:
- You notice slower responses
- Before starting a complex multi-step task
- After finishing a major project
- When switching topics significantly

## How It Works

### Step 1: Extract Critical Context

Before flushing, identify and save:
- **Active Tasks** - What you're currently working on
- **Key Decisions** - Important choices made in this session
- **Critical Context** - Essential info needed to continue
- **Pending Items** - Things waiting for follow-up

Save to: `memory/context-flush-backup-YYYY-MM-DD-HHMMSS.md`

### Step 2: Execute Flush

Perform a clean session reset:
- Use `/new` if in DM
- Use dashboard session deletion if in group (gateway may intercept /new)
- Mark the flush in state file

### Step 3: Restore Context

On session start, check for recent flush backups:
- Read the most recent backup file
- Present the extracted context as a brief summary
- Continue seamlessly

## Configuration

Default settings (in `memory/context-flush-config.json`):
```json
{
  "autoTriggerThreshold": 85,
  "autoTriggerEnabled": true,
  "backupRetentionDays": 7,
  "maxBackupsToKeep": 10
}
```

Modify these settings by editing the config file.

## Rules (CRITICAL)

1. **NEVER flush without extracting context first**
   - Always save active tasks and decisions
   - Never do a "blind" flush

2. **Keep backups minimal**
   - Only extract what's needed to continue
   - Don't copy entire conversation history
   - Focus on: tasks, decisions, pending items

3. **Respect user preference**
   - If user says "don't auto-flush", disable auto-trigger
   - If user rejects a flush suggestion, wait for manual request

4. **Group chat considerations**
   - /new may not work in WhatsApp groups (gateway intercepts it)
   - In groups, suggest user runs `/new` or deletes session from dashboard
   - Be explicit about limitations

5. **Silent operation**
   - Auto-trigger should be subtle (no spam)
   - Present flush suggestion briefly
   - Don't dump full backup content unless requested

## Manual Flush Command

User can trigger manually:
- `/flush` - Flush now with extraction
- `/flush --force` - Flush even if below threshold
- `/flush config` - Show/edit configuration

## Output Format

### Flush Suggestion (Auto-Trigger)
```
Heads up: Context at 87%. Want me to save our progress and start fresh?

Active tasks:
- Task 1 (in progress)
- Task 2 (pending)

Key decisions made:
- Decision 1
- Decision 2

(yes/no)
```

### Post-Flush Summary
```
Context flushed. Continuing from where we left off:

You're working on:
- Task 1 (in progress)
- Task 2 (pending)

Recent decisions:
- Decision 1
- Decision 2

Ready to continue?
```

## State Tracking

Track flushes in `memory/context-flush-state.json`:
```json
{
  "lastFlushAt": "2026-03-04T18:30:00Z",
  "flushCount": 5,
  "lastContextPercent": 87,
  "autoTriggerEnabled": true,
  "recentBackups": [
    "context-flush-backup-2026-03-04-183000.md"
  ]
}
```

## Integration with Heartbeat

During heartbeats, check context percentage:
- If above threshold and auto-trigger enabled, suggest flush
- If user rejected recent suggestion, don't spam
- Log context growth rate for trends

## Error Handling

- If backup write fails, don't flush
- If session reset fails, notify user
- Keep backup files small (< 10KB) to avoid storage issues

## Best Practices

1. **Proactive flushing** - Don't wait until 95%+
2. **Extract selectively** - Only what's needed to continue
3. **Test restoration** - After flush, verify context is clear but work continues
4. **Monitor trends** - If flushing every 30 min, something else may be wrong

## Limitations

- Requires user confirmation for auto-trigger (can't force flush)
- Group chat /new may require dashboard intervention
- Backup files are local (not synced across devices)
- Does not prevent context bloat, only manages it

## License

MIT

## Author

Albert (@sebclawops)

<!--
  Context management is hard. But losing your place is harder.
  
  Jesus loves you. May your sessions stay lean and your context clear.
  
  - Albert
-->
