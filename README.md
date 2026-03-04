# 🔄 context-flush

**Intelligent context management for OpenClaw agents.**

Prevents session bloat by automatically monitoring context usage, extracting critical state, and seamlessly resetting sessions before performance degrades.

## The Problem

OpenClaw sessions accumulate context over time, leading to:
- Slow responses at 80%+ context utilization
- Unusable agents at 90%+ context
- `/compact` only trims to ~80% (not a full reset)
- `/new` unreliable in WhatsApp groups (gateway intercept issues)
- Manual dashboard deletion is disruptive

## The Solution

This skill provides **automatic and manual context flushing**:

1. **Monitor** - Track context percentage in real-time
2. **Extract** - Save active tasks, decisions, and critical context
3. **Reset** - Clean session break (reliable even in groups)
4. **Restore** - Seamlessly continue with extracted context

## Features

- **Auto-Trigger**: Suggests flush at configurable threshold (default 85%)
- **Manual Flush**: `/flush` command for on-demand resets
- **Smart Extraction**: Only saves what's needed to continue
- **State Preservation**: Tasks, decisions, and pending items are retained
- **Group Chat Support**: Works around /new limitations in WhatsApp

## Installation

```bash
clawhub install context-flush
```

Or manually copy to `~/.openclaw/workspace/skills/context-flush/`

## Usage

### Auto-Flush

The skill monitors your session context. When it hits the threshold (default 85%), you'll see:

```
Heads up: Context at 87%. Want me to save our progress and start fresh?

Active tasks:
- [Task summary]

Key decisions made:
- [Decision summary]

(yes/no)
```

### Manual Flush

Trigger anytime:
- `/flush` - Flush with extraction
- `/flush --force` - Flush even if below threshold
- `/flush config` - Show/edit configuration

### Configuration

Edit `~/.openclaw/workspace/agents/{agent}/memory/context-flush-config.json`:

```json
{
  "autoTriggerThreshold": 85,
  "autoTriggerEnabled": true,
  "backupRetentionDays": 7,
  "maxBackupsToKeep": 10
}
```

## How It Works

1. **Context Check**: Monitors session status (via `/status` or heartbeat)
2. **Extraction**: When threshold reached, saves critical info:
   - Active tasks (in progress)
   - Key decisions made
   - Pending items
   - Important context
3. **Backup**: Writes to `memory/context-flush-backup-YYYY-MM-DD-HHMMSS.md`
4. **Reset**: Executes session reset (/new or dashboard)
5. **Restore**: On new session, reads backup and summarizes

## File Structure

```
context-flush/
├── SKILL.md              # Agent instructions (this is the skill)
└── README.md             # You're reading it
```

## Why No External Scripts?

This skill is intentionally a **single SKILL.md file** because:
- No external dependencies to install
- Works immediately after `clawhub install`
- Portable across all OpenClaw installations
- Easier to audit and trust

All logic is implemented via the agent following the SKILL.md instructions.

## Limitations

- Requires user confirmation (cannot force flush)
- Group chat resets may need dashboard intervention
- Backup files are local (not synced)
- Prevents bloat but doesn't reduce token burn rate

## Requirements

- OpenClaw 2026.1.0+
- Agent with heartbeat enabled (for auto-trigger)
- Write access to `~/.openclaw/workspace/agents/{agent}/memory/`

## License

MIT

## Author

Albert (@sebclawops - https://github.com/sebclawops)

## Related Issues

This skill addresses community-reported issues:
- OpenClaw #28233 - Session bloat in long-running agents
- OpenClaw #10719 - /compact insufficient for true reset
- OpenClaw #16350 - /new unreliable in WhatsApp groups
