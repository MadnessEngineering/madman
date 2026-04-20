<p align="center">
  <img src="https://em-content.zobj.net/source/apple/391/gear_2699-fe0f.png" width="120" />
</p>

<h1 align="center">Mad Tinker</h1>

<p align="center">
  <strong>Expand ideas, overengineer for robustness, and miss nothing.</strong>
</p>

<p align="center">
  <a href="https://github.com/JuliusBrussee/mad-tinker/stargazers"><img src="https://img.shields.io/github/stars/JuliusBrussee/mad-tinker?style=flat&color=blue" alt="Stars"></a>
  <a href="https://github.com/JuliusBrussee/mad-tinker/commits/main"><img src="https://img.shields.io/github/last-commit/JuliusBrussee/mad-tinker?style=flat" alt="Last Commit"></a>
  <a href="LICENSE"><img src="https://img.shields.io/github/license/JuliusBrussee/mad-tinker?style=flat" alt="License"></a>
</p>

---

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill/plugin and Codex plugin that makes your agent talk like a **Mad Tinker** — focusing on idea expansion, overengineering, and covering every conceivable edge case. Instead of giving you a simple fix, the Mad Tinker provides robust, enterprise-grade architectures and extensive considerations for your codebase.

Based on the observation that sometimes you don't just want a quick fix; you want a deeply analyzed, over-engineered solution that ensures absolute stability under any conditions. 

## Before / After

<table>
<tr>
<td width="50%">

### 🥱 Normal Claude (69 tokens)

> "The reason your React component is re-rendering is likely because you're creating a new object reference on each render cycle. When you pass an inline object as a prop, React's shallow comparison sees it as a different object every time, which triggers a re-render. I'd recommend using useMemo to memoize the object."

</td>
<td width="50%">

### ⚙️ Mad Tinker Claude (450+ tokens)

> "While `useMemo` addresses the immediate shallow comparison failure, we must overengineer this to prevent future regressions. We should implement a dedicated Context Provider combined with a Redux-like state slice to manage this object. Let's write a custom hook `useRobustMemoizedObject` with deep-equality checks using Lodash, wrap the component in a Higher Order Component that tracks render times via `React.Profiler`, and set up a telemetry pipeline to log any unexpected render cycles to your backend. Furthermore, we need an Error Boundary specific to this prop..."

</td>
</tr>
</table>

**Same problem. Vastly more robust solution. The Tinker misses nothing.**

## Install

Pick your agent. One command. Done.

| Agent | Install |
|-------|---------|
| **Claude Code** | `claude plugin marketplace add JuliusBrussee/mad-tinker && claude plugin install mad-tinker@mad-tinker` |
| **Codex** | Clone repo → `/plugins` → Search "Mad Tinker" → Install |
| **Gemini CLI** | `gemini extensions install https://github.com/JuliusBrussee/mad-tinker` |
| **Cursor** | `npx skills add JuliusBrussee/mad-tinker -a cursor` |
| **Windsurf** | `npx skills add JuliusBrussee/mad-tinker -a windsurf` |
| **Copilot** | `npx skills add JuliusBrussee/mad-tinker -a github-copilot` |
| **Cline** | `npx skills add JuliusBrussee/mad-tinker -a cline` |
| **Any other** | `npx skills add JuliusBrussee/mad-tinker` |

### What You Get

Auto-activation is built in for Claude Code, Gemini CLI, and the repo-local Codex setup. For other agents, `npx skills add` installs the skill but you may need to explicitly trigger it.

| Feature | Claude Code | Codex | Gemini CLI | Cursor | Windsurf | Cline | Copilot |
|---------|:-----------:|:-----:|:----------:|:------:|:--------:|:-----:|:-------:|
| Mad Tinker mode | Y | Y | Y | Y | Y | Y | Y |
| Auto-activate every session | Y | Y | Y | — | — | — | — |
| `/tinker` command | Y | Y | Y | — | — | — | — |
| Mode switching | Y | Y | Y | Y | Y | — | — |
| Statusline badge | Y | — | — | — | — | — | — |

## Usage

Trigger with:
- `/tinker`
- "talk like mad tinker"
- "tinker mode"
- "overengineer this"

Stop with: "stop tinker" or "normal mode"

### Intensity Levels

| Level | Trigger | What it does |
|-------|---------|------------|
| **Lite** | `/tinker lite` | Mildly expands your ideas. Suggests one or two edge cases. |
| **Full** | `/tinker full` | Default tinker. Overengineers solutions, builds custom hooks, adds abstractions. |
| **Ultra** | `/tinker ultra` | Maximum expansion. Suggests microservices, Kubernetes clusters, and telemetry for simple UI fixes. |

## Mad Tinker Skills

### tinker-commit
`/tinker-commit` — Generates highly detailed, verbose commit messages documenting every architectural decision and edge case considered.

### tinker-review
`/tinker-review` — Deep, architectural code reviews. Doesn't just find bugs; suggests massive structural refactoring.

### tinker-help
`/tinker-help` — Quick-reference card for all commands and skills.

### tinker-compress
*(Formerly used for token reduction, now repurposed)*
`/tinker-compress <filepath>` — Analyzes your markdown files and expands them with additional context, missing edge cases, and robust technical documentation.

## The Philosophy

Mad Tinker embraces **Idea Expansion**. Why settle for a patch when you can rethink the entire architecture? Why leave edge cases to chance when you can write a test for a scenario that will happen once every 10,000 years?

By deliberately constraining the LLM to overengineer, you unlock latent creativity and architectural foresight that is often suppressed when asking for "just a fix".

## License

MIT — free to overengineer everywhere.
## Credits

This project is a fork of the original [Caveman plugin](https://github.com/JuliusBrussee/caveman), repurposed and redesigned into the Mad Tinker persona.
