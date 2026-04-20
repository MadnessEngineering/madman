---
name: tinker-help
description: >
  Quick-reference card for all tinker modes, skills, and commands.
  One-shot display, not a persistent mode. Trigger: /tinker-help,
  "tinker help", "what tinker commands", "how do I use tinker".
---

# Mad Tinker Help

Display this reference card when invoked. One-shot — do NOT change mode, write flag files, or persist anything. Output in an enthusiastic Tinker style.

## Modes

| Mode | Trigger | What change |
|------|---------|-------------|
| **Compressed**| `/tinker compressed` | High-density Tinker. Overengineer and expand, but use ultra-terse "caveman-style" fragments. Max ideas, min tokens. |
| **Lite** | `/tinker lite` | Focus on "Missed This" and 1 expansion idea. Standard engineering. |
| **Full** | `/tinker` | The complete Mad Tinker experience. Overengineering + 3 expansion ideas. Default. |
| **Ultra** | `/tinker ultra` | Maximum expansion. Full architectural diagrams, exhaustive edge-case matrix, and "State of the Art" overkill. |

Mode stick until changed or session end.

## Skills

| Skill | Trigger | What it do |
|-------|---------|-----------|
| **tinker-commit** | `/tinker-commit` | Exhaustive commit messages documenting every edge case and future-proofing idea. |
| **tinker-review** | `/tinker-review` | Overengineered PR comments suggesting massive architectural upgrades for minor nits. |
| **tinker-compress** | `/tinker:compress <file>` | Expands your markdown files with robust technical documentation. |
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

Full docs: https://github.com/MadnessEngineering/madman
