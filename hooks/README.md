# Mad Tinker Hooks

These hooks are **bundled with the tinker plugin** and activate automatically when the plugin is installed. No manual setup required.

If you installed tinker standalone (without the plugin), you can use `bash hooks/install.sh` to wire them into your settings.json manually.

## What's Included

### `tinker-activate.js` — SessionStart hook

- Runs once when Claude Code starts
- Writes `full` to `~/.claude/.tinker-active` (flag file)
- Emits tinker rules as hidden SessionStart context
- Detects missing statusline config and emits setup nudge (Claude will offer to help)

### `tinker-mode-tracker.js` — UserPromptSubmit hook

- Fires on every user prompt, checks for `/tinker` commands
- Writes the active mode to the flag file when a tinker command is detected
- Supports: `full`, `lite`, `ultra`, `tinker`, `tinker-lite`, `tinker-ultra`, `commit`, `review`, `compress`

### `tinker-statusline.sh` / `tinker-statusline.ps1` — Statusline badge script

- Reads `~/.claude/.tinker-active` and outputs a colored badge
- Shows `[TINKER]`, `[TINKER:ULTRA]`, `[TINKER:TINKER]`, etc.

## Statusline Badge

The statusline badge shows which tinker mode is active directly in your Claude Code status bar.

**Plugin users:** If you do not already have a `statusLine` configured, Claude will detect that on your first session after install and offer to set it up for you. Accept and you're done.

If you already have a custom statusline, tinker does not overwrite it and Claude stays quiet. Add the badge snippet to your existing script instead.

**Standalone users:** `install.sh` / `install.ps1` wires the statusline automatically if you do not already have a custom statusline. If you do, the installer leaves it alone and prints the merge note.

**Manual setup:** If you need to configure it yourself, add one of these to `~/.claude/settings.json`:

```json
{
  "statusLine": {
    "type": "command",
    "command": "bash /path/to/tinker-statusline.sh"
  }
}
```

```json
{
  "statusLine": {
    "type": "command",
    "command": "powershell -ExecutionPolicy Bypass -File C:\\path\\to\\tinker-statusline.ps1"
  }
}
```

Replace the path with the actual script location (e.g. `~/.claude/hooks/` for standalone installs, or the plugin install directory for plugin installs).

**Custom statusline:** If you already have a statusline script, add this snippet to it:

```bash
tinker_text=""
tinker_flag="$HOME/.claude/.tinker-active"
if [ -f "$tinker_flag" ]; then
  tinker_mode=$(cat "$tinker_flag" 2>/dev/null)
  if [ "$tinker_mode" = "full" ] || [ -z "$tinker_mode" ]; then
    tinker_text=$'\033[38;5;172m[TINKER]\033[0m'
  else
    tinker_suffix=$(echo "$tinker_mode" | tr '[:lower:]' '[:upper:]')
    tinker_text=$'\033[38;5;172m[TINKER:'"${tinker_suffix}"$']\033[0m'
  fi
fi
```

Badge examples:
- `/tinker` → `[TINKER]`
- `/tinker ultra` → `[TINKER:ULTRA]`
- `/tinker tinker` → `[TINKER:TINKER]`
- `/tinker-commit` → `[TINKER:COMMIT]`
- `/tinker-review` → `[TINKER:REVIEW]`

## How It Works

```
SessionStart hook ──writes "full"──▶ ~/.claude/.tinker-active ◀──writes mode── UserPromptSubmit hook
                                              │
                                           reads
                                              ▼
                                     Statusline script
                                    [TINKER:ULTRA] │ ...
```

SessionStart stdout is injected as hidden system context — Claude sees it, users don't. The statusline runs as a separate process. The flag file is the bridge.

## Uninstall

If installed via plugin: disable the plugin — hooks deactivate automatically.

If installed via `install.sh`:
```bash
bash hooks/uninstall.sh
```

Or manually:
1. Remove `~/.claude/hooks/tinker-activate.js`, `~/.claude/hooks/tinker-mode-tracker.js`, and the matching statusline script (`tinker-statusline.sh` on macOS/Linux or `tinker-statusline.ps1` on Windows)
2. Remove the SessionStart, UserPromptSubmit, and statusLine entries from `~/.claude/settings.json`
3. Delete `~/.claude/.tinker-active`
