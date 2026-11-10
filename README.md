# skills

Skills for coding agents.

## Install

Installs every skill globally for Claude Code and Codex, after you confirm the list, via the [skills CLI](https://github.com/vercel-labs/skills):

```bash
npx skills add seekayel/skills --skill '*' -a claude-code -a codex -g
```

To pick specific skills, drop `--skill '*'` and choose interactively, or pass a name: `--skill unslop`.

## Skills

- **unslop**: cuts AI tells from writing and rewrites it in a direct, conversational voice. Adapted from [cursor/plugins](https://github.com/cursor/plugins/tree/main/pstack/skills/unslop).
- **research-plan-implement**: a research, plan, then implement workflow for code changes.
- **code-review**: reviews uncommitted changes or a branch diff using the `codex` CLI.
- **council** ("council"): reviews a change with Claude Opus and GPT Sol, plus Claude Fable 5.2 when it's available, Muse and Grok when they're authenticated, and Gemini as a contrarian "jester" when it's authenticated. Loops until the regular seats agree. Every seat reviews blind. It never sees the target score or the other reviews.
- **tenth-man** ("10th man"): runs `council` and adds a rebel on Claude Opus 5.5 or GPT 6.1 Sol. The rebel reads every other review and argues they're all wrong, the way Israeli military intelligence has used a "tenth man" since 1973. Install it with `council`.
- **full-send** ("full send"): takes a change to a merged PR with nobody watching. It proves the change, gets a council review, files the PR, watches it, answers review bots, and merges. Uses `council`, so install both.

`council` and `full-send` are adapted from [t3dotgg/tstack](https://github.com/t3dotgg/tstack). They use [T3 Code](https://github.com/pingdotgg/t3code) tools when they're available, and fall back to native subagents and the bundled `pr-wait` script (needs `python3` and an authenticated `gh`) when they're not.

## Claude Code plugins

`research-plan-implement` uses three sub-agents (`codebase-locator`, `codebase-analyzer`, `pattern-finder`). The skills CLI doesn't install agents, so to get them, install it as a Claude Code plugin instead:

```
/plugin marketplace add seekayel/skills
/plugin install research-plan-implement-plugin@agent-skills-marketplace
```

## License

MIT
