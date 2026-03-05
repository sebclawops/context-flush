---
name: context-flush
version: 1.2.0
description: Context monitoring and orchestration for OpenClaw agents. Monitors session context with dynamic thresholds, warns before bloat, extracts state, and orchestrates seamless session resets.
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

# Context Flush Skill v1.2

Context monitoring and session orchestration. Prevents the context cliff on any model size.

## The Problem

OpenClaw sessions accumulate context until they hit a wall:
- **Small models (30k window)**: Fail at ~24k tokens
- **Large models (256k window)**: Degrade at ~150k+ tokens but 100k is the practical limit
- **Current `/compact`**: Only trims to ~80% - doesn't solve the problem
- **`/new`**: Works but requires manual user action

One-size-fits-all token counts fail. Large windows need caps. Small windows need percentages.

## The Solution (Orchestration Mode)

Since programmatic session reset is architecturally impossible (by design), this skill orchestrates the reset with **dynamic thresholds** and **one-tap reset links**:

1. **Monitor** - Check context via `/status` or `session_status` tool every 4 messages + every tool call
2. **Calculate** - Apply capped percentage thresholds based on model context window
3. **Warn** - At warning threshold, notify user: "Context at X%. Consider /new soon."
4. **Extract** - At hard gate, save state to daily memory log
5. **Degrade** - At critical threshold, limit functionality and provide one-tap reset
6. **Restore** - After reset, read saved state and seamlessly continue

## Dynamic Thresholds

| Level | Calculation | Action |
|-------|-------------|--------|
| **Warning** | `min(60% of window, 80k tokens)` | Heads-up message |
| **Hard Gate** | `min(80% of window, 100k tokens)` | Extract state, prompt for reset |
| **Critical** | `min(90% of window, 140k tokens)` | **Degraded mode** + one-tap reset |

### Threshold Examples

| Agent | Model | Window | Warning | Hard Gate | Critical |
|-------|-------|--------|---------|-----------|----------|
| Capital | Qwen 3.5 9B | 32k | 19k | 26k | 29k |
| Sea Cool | MiniMax M2.5 | 200k | 80k (cap) | 100k (cap) | 140k (cap) |
| Samurai | MiniMax M2.5 | 200k | 80k (cap) | 100k (cap) | 140k (cap) |
| Blinds | Kimi K2.5 | 256k | 80k (cap) | 100k (cap) | 140k (cap) |
| Seb Personal | Kimi K2.5 | 256k | 80k (cap) | 100k (cap) | 140k (cap) |
| Sheet | Qwen 3.5 9B | 32k | 19k | 26k | 29k |
| Sandbox | Qwen 3.5 9B | 32k | 19k | 26k | 29k |

**Why the caps?** Large models (200k+ windows) hit performance cliffs well before their theoretical limits. Caps keep them responsive.

## One-Tap Reset (Phone-Friendly)

### Telegram
Inline button with `/new` callback:
```
[🔄 Reset Session]  ← Click to run /new
```

### WhatsApp
wa.me link with pre-filled `/new`:
```
👉 Tap to reset: https://wa.me/{agent_phone}?text=%2Fnew
```

**User flow:** Tap link → WhatsApp opens → `/new` pre-filled → Tap send → Session resets

### Config (Per-Agent Phone Numbers)

Add to `~/.openclaw/workspace/agents/{agent}/memory/context-flush-config.json`:

```json
{
  "agentPhoneNumbers": {
    "whatsapp": "+17864224950",
    "telegram": "@sebclawops_bot"
  }
}
```

## Monitoring Frequency

- Every 4 messages
- Every tool call
- Silent when under warning threshold

## State Extraction

Save to `memory/YYYY-MM-DD.md` with `<!-- context-flush -->` marker:

```markdown
## Context Flush Save - 2026-03-05 17:30

### Active Tasks
- [Task] (status)

### Key Decisions
- [Decision]

### Files In Play
- [File path]

### Pending Items
- [Item]

### Context Summary
[Brief narrative of session purpose and current state]
```

## User Flow

### At Warning Threshold
```
Heads up: Context at 82k tokens (32% of window). Approaching performance threshold.
Consider running `/new` when convenient.
```

### At Hard Gate
```
⚠️ Context at 98k tokens (38% of window). At hard gate for this model.

Saving state to memory/2026-03-05.md...
✓ State saved.

Ready to reset. Run `/new` and I'll restore from where we left off.

[Telegram: 🔄 Reset Session button]
[WhatsApp: 👉 Tap to reset: https://wa.me/+17864224950?text=%2Fnew]
```

### At Critical Threshold (Degraded Mode)
```
🚨 Context at 145k tokens (57% of window). CRITICAL.

I can only handle simple commands. Complex tasks require a reset.

State saved to memory/2026-03-05.md.

👉 Tap to reset now: https://wa.me/+17864224950?text=%2Fnew

Or type /new to restore full functionality.
```

**Degraded Mode Behavior:**
- Refuse multi-step tasks: "Context critical. Run `/new` to continue."
- Limit tool calls: "Simple queries only. Reset required for full access."
- Every response ends with reset prompt

### After User Runs `/new`
```
Session reset detected. Restoring from memory/2026-03-05.md:

You're working on:
- [Task 1] (in progress)
- [Task 2] (pending)

Key decisions:
- [Decision 1]
- [Decision 2]

Ready to continue?
```

## Manual Commands

- `/flush` - Trigger state extraction immediately
- `/flush status` - Show current context usage and thresholds
- `/flush config` - Show/edit configuration

## Implementation Notes

### Threshold Calculation

```
warningThreshold = min(contextWindow * 0.60, 80000)
hardGateThreshold = min(contextWindow * 0.80, 100000)
criticalThreshold = min(contextWindow * 0.90, 140000)
```

### One-Tap Reset Links

**Telegram:** Use `message` tool with inline keyboard:
```json
{
  "action": "send",
  "channel": "telegram",
  "message": "Context critical. Reset now?",
  "buttons": [{"text": "🔄 Reset Session", "callback": "/new"}]
}
```

**WhatsApp:** Generate wa.me URL:
```
https://wa.me/{phone}?text=%2Fnew
```
- `%2F` = URL-encoded `/`
- Phone must include country code, no spaces/symbols
- Link opens WhatsApp with `/new` pre-filled

### Why Not Auto-Reset?
OpenClaw sessions are user-controlled by design. No API exists for programmatic reset. This is intentional security architecture.

### Why Daily Memory Logs?
- Single source of truth for daily context
- Already exists in workflow
- Natural place for session summaries
- No new file naming schemes needed

### Restoration Detection
On session start, check for recent entries in today's memory log with `context-flush` markers. If found, auto-prompt restoration.

### Context Window Detection
Use `session_status` tool to get current model, then lookup window size from reference table. If unknown model, default to conservative 100k window.

## Context Window Reference

| Model | Window | Source |
|-------|--------|--------|
| Qwen 3.5 9B | 32,768 | Local config |
| MiniMax M2.5 | 200,000 | OpenRouter |
| Kimi K2.5 | 256,000 | OpenRouter |

## Version History

- **v1.2.0** - Dynamic thresholds + one-tap reset + degraded mode (current)
- **v1.1.0** - Orchestration mode: Fixed thresholds
- **v1.0.0** - Placeholder: Basic monitoring concept

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
