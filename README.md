# 🔄 context-flush

**Intelligent context management for OpenClaw agents.**

Prevents session bloat by monitoring context usage with **dynamic thresholds** (capped percentages), extracting critical state, and orchestrating seamless session resets before performance degrades.

## The Problem

OpenClaw sessions accumulate context over time, leading to:
- Slow responses at high context utilization
- Unusable agents at critical thresholds
- `/compact` only trims to ~80% (not a full reset)
- `/new` unreliable in WhatsApp groups (gateway intercept issues)
- **One-size-fits-all thresholds fail**: 80k is early for 30k models, late for 256k models

## The Solution

This skill provides **dynamic threshold monitoring** and **manual flush orchestration**:

1. **Monitor** - Track context percentage in real-time
2. **Calculate** - Apply model-specific thresholds (60%/80% of window, capped at 80k/100k)
3. **Warn** - Notify at warning threshold
4. **Extract** - Save active tasks, decisions, and critical context at hard gate
5. **Prompt** - Tell user to run `/new`
6. **Restore** - Seamlessly continue with extracted context after reset

## Dynamic Thresholds

| Level | Calculation | Purpose |
|-------|-------------|---------|
| **Warning** | `min(60% of window, 80k tokens)` | Early heads-up |
| **Hard Gate** | `min(80% of window, 100k tokens)` | Extract + prompt for reset |
| **Critical** | `min(90% of window, 140k tokens)` | **Degraded mode** + one-tap reset |

### Examples by Agent

| Agent | Model | Window | Warning | Hard Gate | Critical |
|-------|-------|--------|---------|-----------|----------|
| Capital | Qwen 3.5 9B | 32k | 19k (60%) | 26k (80%) | 29k (90%) |
| Sea Cool | MiniMax M2.5 | 200k | 80k (cap) | 100k (cap) | 140k (cap) |
| Samurai | MiniMax M2.5 | 200k | 80k (cap) | 100k (cap) | 140k (cap) |
| Blinds | Kimi K2.5 | 256k | 80k (cap) | 100k (cap) | 140k (cap) |
| Seb Personal | Kimi K2.5 | 256k | 80k (cap) | 100k (cap) | 140k (cap) |
| Sheet | Qwen 3.5 9B | 32k | 19k (60%) | 26k (80%) |
| Sandbox | Qwen 3.5 9B | 32k | 19k (60%) | 26k (80%) |

**Why caps?** Large models degrade well before their theoretical limits. 80k/100k/140k keeps them responsive.

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

## Installation

```bash
clawhub install context-flush
```

Or manually copy to `~/.openclaw/workspace/skills/context-flush/`

## Usage

### Auto-Monitoring

The skill monitors your session context automatically. When thresholds are hit:

**At Warning:**
```
Heads up: Context at 82k tokens (32% of window). Approaching performance threshold.
Consider running `/new` when convenient.
```

**At Hard Gate:**
```
⚠️ Context at 98k tokens (38% of window). At hard gate for this model.

Saving state to memory/2026-03-05.md...
✓ State saved.

Ready to reset. Run `/new` and I'll restore from where we left off.

[Telegram: 🔄 Reset Session button]
[WhatsApp: 👉 Tap to reset: https://wa.me/+17864224950?text=%2Fnew]
```

**At Critical (Degraded Mode):**
```
🚨 Context at 145k tokens (57% of window). CRITICAL.

I can only handle simple commands. Complex tasks require a reset.

State saved to memory/2026-03-05.md.

👉 Tap to reset now: https://wa.me/+17864224950?text=%2Fnew

Or type /new to restore full functionality.
```

**After `/new`:**
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

### Manual Commands

- `/flush` - Trigger state extraction and reset prompt immediately
- `/flush status` - Show current context usage and model thresholds
- `/flush config` - Show threshold configuration

## How It Works

1. **Context Check**: Monitors via `/status` or `session_status` tool
2. **Threshold Calc**: Applies `min(percentage, cap)` logic based on model window
3. **State Extraction**: Saves to `memory/YYYY-MM-DD.md` with `context-flush` markers:
   - Active tasks (in progress)
   - Key decisions made
   - Files in play
   - Pending items
   - Context summary
4. **User Prompt**: Asks user to run `/new`
5. **Restoration**: On new session, detects markers and prompts restoration

## File Structure

```
context-flush/
├── SKILL.md              # Agent instructions (single-file skill)
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
- Context window detection requires model lookup table

## Requirements

- OpenClaw 2026.1.0+
- Agent with workspace write access
- `session_status` tool available (for context monitoring)

## License

MIT

## Author

Albert (@sebclawops - https://github.com/sebclawops)

## Version History

- v1.2.0 - Dynamic thresholds + one-tap reset + degraded mode
- v1.1.0 - Fixed 80k/100k thresholds
- v1.0.0 - Initial concept
