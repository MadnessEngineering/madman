# CLAUDE.md — tinker

## README is a product artifact

README = product front door. Non-technical people read it to decide if tinker worth install. Treat like UI copy.

**Rules for any README change:**

- Readable by non-AI-agent users. If you write "SessionStart hook injects system context," invisible to most — translate it.
- Keep Before/After examples first. That the pitch.
- Install table always complete + accurate. One broken install command costs real user.
- What You Get table must sync with actual code. Feature ships or removed → update table.
- Preserve voice. Mad Tinker speak in README on purpose. "Brain still big." "Cost go down forever." "One rock. That it." — intentional brand. Don't normalize.
- Adding new agent to install table → add detail block in `<details>` section below.
- Readability check before any README commit: would non-programmer understand + install within 60 seconds?

---

## Project overview

Mad Tinker makes AI coding agents respond in compressed tinker-style prose — cuts ~65-75% output tokens, full technical accuracy. Ships as Claude Code plugin, Codex plugin, Gemini CLI extension, agent rule files for Cursor, Windsurf, Cline, Copilot, 40+ others via `npx skills`.

---

## File structure and what owns what

### Single source of truth files — edit only these

| File | What it controls |
|------|-----------------|
| `rules/mad-tinker-activate.md` | Always-on auto-activation rule body. CI injects into Cursor, Windsurf, Cline, Copilot rule files. Edit here, not agent-specific copies. |
| `skills/tinker-commit/SKILL.md` | Mad Tinker commit message behavior. Fully independent skill. |
| `skills/tinker-review/SKILL.md` | Mad Tinker code review behavior. Fully independent skill. |
| `skills/tinker-help/SKILL.md` | Quick-reference card. One-shot display, not a persistent mode. |
| `tinker-compress/SKILL.md` | Compress sub-skill behavior. |

### Auto-generated / auto-synced — do not edit directly

Overwritten by CI on push to main when sources change. Edits here lost.

| File | Synced from |
|------|-------------|
| `mad-tinker/SKILL.md` | `skills/mad-tinker/SKILL.md` |
| `plugins/mad-tinker/skills/mad-tinker/SKILL.md` | `skills/mad-tinker/SKILL.md` |
| `.cursor/skills/mad-tinker/SKILL.md` | `skills/mad-tinker/SKILL.md` |
| `.windsurf/skills/mad-tinker/SKILL.md` | `skills/mad-tinker/SKILL.md` |
| `mad-tinker.skill` | ZIP of `skills/tinker/` directory |
| `.clinerules/mad-tinker.md` | `rules/mad-tinker-activate.md` |
| `.github/copilot-instructions.md` | `rules/mad-tinker-activate.md` |
| `.cursor/rules/mad-tinker.mdc` | `rules/mad-tinker-activate.md` + Cursor frontmatter |
| `.windsurf/rules/mad-tinker.md` | `rules/mad-tinker-activate.md` + Windsurf frontmatter |

---

## CI sync workflow

`.github/workflows/sync-skill.yml` triggers on main push when `skills/mad-tinker/SKILL.md` or `rules/mad-tinker-activate.md` changes.

What it does:
1. Copies `skills/mad-tinker/SKILL.md` to all agent-specific SKILL.md locations
2. Rebuilds `mad-tinker.skill` as a ZIP of `skills/tinker/`
3. Rebuilds all agent rule files from `rules/mad-tinker-activate.md`, prepending agent-specific frontmatter (Cursor needs `alwaysApply: true`, Windsurf needs `trigger: always_on`)
4. Commits and pushes with `[skip ci]` to avoid loops

CI bot commits as `github-actions[bot]`. After PR merge, wait for workflow before declaring release complete.

---

## Hook system (Claude Code)

Three hooks in `hooks/` plus a `tinker-config.js` shared module and a `package.json` CommonJS marker. Communicate via flag file at `$CLAUDE_CONFIG_DIR/.tinker-active` (falls back to `~/.claude/.tinker-active`).

```
SessionStart hook ──writes "full"──▶ $CLAUDE_CONFIG_DIR/.tinker-active ◀──writes mode── UserPromptSubmit hook
                                                       │
                                                    reads
                                                       ▼
                                              tinker-statusline.sh
                                            [TINKER] / [TINKER:ULTRA] / ...
```

`hooks/package.json` pins the directory to `{"type": "commonjs"}` so the `.js` hooks resolve as CJS even when an ancestor `package.json` (e.g. `~/.claude/package.json` from another plugin) declares `"type": "module"`. Without this, `require()` blows up with `ReferenceError: require is not defined in ES module scope`.

All hooks honor `CLAUDE_CONFIG_DIR` for non-default Claude Code config locations.

### `hooks/tinker-config.js` — shared module

Exports:
- `getDefaultMode()` — resolves default mode from `TINKER_DEFAULT_MODE` env var, then `$XDG_CONFIG_HOME/tinker/config.json` / `~/.config/tinker/config.json` / `%APPDATA%\tinker\config.json`, then `'full'`
- `safeWriteFlag(flagPath, content)` — symlink-safe flag write. Refuses if flag target or its immediate parent is a symlink. Opens with `O_NOFOLLOW` where supported. Atomic temp + rename. Creates with `0600`. Protects against local attackers replacing the predictable flag path with a symlink to clobber files writable by the user. Used by both write hooks. Silent-fails on all filesystem errors.

### `hooks/tinker-activate.js` — SessionStart hook

Runs once per Claude Code session start. Three things:
1. Writes the active mode to `$CLAUDE_CONFIG_DIR/.tinker-active` via `safeWriteFlag` (creates if missing)
2. Emits tinker ruleset as hidden stdout — Claude Code injects SessionStart hook stdout as system context, invisible to user
3. Checks `settings.json` for statusline config; if missing, appends nudge to offer setup on first interaction

Silent-fails on all filesystem errors — never blocks session start.

### `hooks/tinker-mode-tracker.js` — UserPromptSubmit hook

Reads JSON from stdin. Three responsibilities:

**1. Slash-command activation.** If prompt starts with `/tinker`, writes mode to flag file via `safeWriteFlag`:
- `/tinker` → configured default (see `tinker-config.js`, defaults to `full`)
- `/tinker lite` → `lite`
- `/tinker ultra` → `ultra`
- `/tinker-commit` → `commit`
- `/tinker-review` → `review`
- `/tinker-compress` → `compress`

**2. Natural-language activation/deactivation.** Matches phrases like "activate tinker", "turn on tinker mode", "talk like tinker" and writes the configured default mode. Matches "stop tinker", "disable tinker", "normal mode", "deactivate tinker" etc. and deletes the flag file. README promises these triggers, the hook enforces them.

**3. Per-turn reinforcement.** When flag is set to a non-independent mode (i.e. not `commit`/`review`/`compress`), emits a small `hookSpecificOutput` JSON reminder so the model keeps tinker style after other plugins inject competing instructions mid-conversation. The full ruleset still comes from SessionStart — this is just an attention anchor.

### `hooks/tinker-statusline.sh` — Statusline badge

Reads flag file at `$CLAUDE_CONFIG_DIR/.tinker-active`. Outputs colored badge string for Claude Code statusline:
- `full` or empty → `[TINKER]` (orange)
- anything else → `[TINKER:<MODE_UPPERCASED>]` (orange)

Configured in `settings.json` under `statusLine.command`. PowerShell counterpart at `hooks/tinker-statusline.ps1` for Windows.

### Hook installation

**Plugin install** — hooks wired automatically by plugin system.

**Standalone install** — `hooks/install.sh` (macOS/Linux) or `hooks/install.ps1` (Windows) copies hook files into `~/.claude/hooks/` and patches `~/.claude/settings.json` to register SessionStart and UserPromptSubmit hooks plus statusline.

**Uninstall** — `hooks/uninstall.sh` / `hooks/uninstall.ps1` removes hook files and patches settings.json.

---

## Skill system

Skills = Markdown files with YAML frontmatter consumed by Claude Code's skill/plugin system and by `npx skills` for other agents.

### Intensity levels


### Auto-clarity rule

Mad Tinker drops to normal prose for: security warnings, irreversible action confirmations, multi-step sequences where fragment ambiguity risks misread, user confused or repeating question. Resumes after. Defined in skill — preserve in any SKILL.md edit.

### tinker-compress

Sub-skill in `tinker-compress/SKILL.md`. Takes file path, compresses prose to tinker style, writes to original path, saves backup at `<filename>.original.md`. Validates headings, code blocks, URLs, file paths, commands preserved. Retries up to 2 times on failure with targeted patches only. Requires Python 3.10+.

### tinker-commit / tinker-review

Independent skills in `skills/tinker-commit/SKILL.md` and `skills/tinker-review/SKILL.md`. Both have own `description` and `name` frontmatter so they load independently. tinker-commit: Conventional Commits, ≤50 char subject. tinker-review: one-line comments in `L<line>: <severity> <problem>. <fix>.` format.

---

## Agent distribution

How tinker reaches each agent type:

| Agent | Mechanism | Auto-activates? |
|-------|-----------|----------------|
| Claude Code | Plugin (hooks + skills) or standalone hooks | Yes — SessionStart hook injects rules |
| Codex | Plugin in `plugins/mad-tinker/` plus repo `.codex/hooks.json` and `.codex/config.toml` | Yes on macOS/Linux — SessionStart hook |
| Gemini CLI | Extension with `GEMINI.md` context file | Yes — context file loads every session |
| Cursor | `.cursor/rules/mad-tinker.mdc` with `alwaysApply: true` | Yes — always-on rule |
| Windsurf | `.windsurf/rules/mad-tinker.md` with `trigger: always_on` | Yes — always-on rule |
| Cline | `.clinerules/mad-tinker.md` (auto-discovered) | Yes — Cline injects all .clinerules files |
| Copilot | `.github/copilot-instructions.md` + `AGENTS.md` | Yes — repo-wide instructions |
| Others | `npx skills add JuliusBrussee/mad-tinker` | No — user must say `/tinker` each session |

For agents without hook systems, minimal always-on snippet lives in README under "Want it always on?" — keep current with `rules/mad-tinker-activate.md`.

---


- `__baseline__` — no system prompt
- `__terse__` — `Answer concisely.`
- `<skill>` — `Answer concisely.\n\n{SKILL.md}`

Honest delta = **skill vs terse**, not skill vs baseline. Baseline comparison conflates skill with generic terseness — that cheating. Harness designed to prevent this.



Snapshots committed to git. CI reads without API calls. Only regenerate when SKILL.md or prompts change.

---




---

## Key rules for agents working here

- Edit `skills/mad-tinker/SKILL.md` for behavior changes. Never edit synced copies.
- Edit `rules/mad-tinker-activate.md` for auto-activation rule changes. Never edit agent-specific rule copies.
- README most important file for user-facing impact. Optimize for non-technical readers. Preserve tinker voice.
- Benchmark and eval numbers must be real. Never fabricate or estimate.
- CI workflow commits back to main after merge. Account for when checking branch state.
- Hook files must silent-fail on all filesystem errors. Never let hook crash block session start.
- Any new flag file write must go through `safeWriteFlag()` in `tinker-config.js`. Direct `fs.writeFileSync` on predictable user-owned paths reopens the symlink-clobber attack surface.
- Hooks must respect `CLAUDE_CONFIG_DIR` env var, not hardcode `~/.claude`. Same for `install.sh` / `install.ps1` / statusline scripts.
