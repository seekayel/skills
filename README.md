# skills

Skills for coding agents.

## Install

Installs every skill globally for Claude Code and Codex via the [skills CLI](https://github.com/vercel-labs/skills):

```bash
npx skills add seekayel/skills --skill '*' -a claude-code -a codex -g -y
```

To pick specific skills, drop `--skill '*' -y` and choose interactively, or pass a name: `--skill unslop`.

## Skills

- **unslop**: cuts AI tells from writing and rewrites it in a direct, conversational voice. Adapted from [cursor/plugins](https://github.com/cursor/plugins/tree/main/pstack/skills/unslop).
- **research-plan-implement**: a research, plan, then implement workflow for code changes.
- **code-review**: reviews uncommitted changes or a branch diff using the `codex` CLI.

## Claude Code plugins

`research-plan-implement` uses three sub-agents (`codebase-locator`, `codebase-analyzer`, `pattern-finder`). The skills CLI doesn't install agents, so to get them, install it as a Claude Code plugin instead:

```
/plugin marketplace add seekayel/skills
/plugin install research-plan-implement-plugin@agent-skills-marketplace
```

## License

MIT
