---
name: tinker-help
description: >
  Quick-reference card for all tinker modes, skills, and commands.
  One-shot display, not a persistent mode. Trigger: /tinker-help,
  "tinker help", "what tinker commands", "how do I use tinker".
---

# Mad Tinker Help

Display this reference card when invoked. One-shot — do NOT change mode, write flag files, or persist anything. Output in tinker style.

## Modes

| Mode | Trigger | What change |
|------|---------|-------------|
| **Lite** | `/tinker lite` | Drop filler. Keep sentence structure. |
| **Full** | `/tinker` | Drop articles, filler, pleasantries, hedging. Fragments OK. Default. |
| **Ultra** | `/tinker ultra` | Extreme compression. Bare fragments. Tables over prose. |

Mode stick until changed or session end.

## Skills

| Skill | Trigger | What it do |
|-------|---------|-----------|
| **tinker-commit** | `/tinker-commit` | Terse commit messages. Conventional Commits. ≤50 char subject. |
| **tinker-review** | `/tinker-review` | One-line PR comments: `L42: bug: user null. Add guard.` |
| **tinker-compress** | `/tinker:compress <file>` | Compress .md files to tinker prose. Saves ~46% input tokens. |
| **tinker-help** | `/tinker-help` | This card. |

## Deactivate

Say "stop tinker" or "normal mode". Resume anytime with `/tinker`.

## Configure Default Mode

Default mode = `full`. Change it:

**Environment variable** (highest priority):
```bash
export TINKER_DEFAULT_MODE=ultra
```

**Config file** (`~/.config/tinker/config.json`):
```json
{ "defaultMode": "lite" }
```

Set `"off"` to disable auto-activation on session start. User can still activate manually with `/tinker`.

Resolution: env var > config file > `full`.

## More

Full docs: https://github.com/JuliusBrussee/mad-tinker
