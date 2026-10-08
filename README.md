# Claude Code skills

Reusable skills for [Claude Code](https://claude.com/claude-code), built for workflows I actually use.

## Skills

| Skill | What it does |
|---|---|
| [`save-that-step`](save-that-step/SKILL.md) | Turns a procedure from the current conversation (a stepper, a numbered how-to, or commands you just ran) into a runbook in Notion, with secrets redacted. |
| [`recall-step`](recall-step/SKILL.md) | Finds a saved runbook and walks you through it again, or answers a precise question from it. |

Together they give Claude Code a memory for procedures: do something once, save it, get it back months later with `/recall-step`.

<!-- TODO: add a demo GIF here -->

## Install

Copy the skill folders you want into your personal skills directory:

```bash
git clone https://github.com/yassibra/claude-code-skills.git
cp -r claude-code-skills/save-that-step claude-code-skills/recall-step ~/.claude/skills/
```

Restart Claude Code, then run `/save-that-step` or `/recall-step`.

## Requirements

- **Notion (recommended):** connect a Notion MCP server to Claude Code. On first use, `save-that-step` finds or creates a `Runbooks` database and stores its id in `~/.claude/save-that-step/config.json`. That file stays on your machine.
- **Without Notion:** runbooks are saved as Markdown files in `~/.claude/save-that-step/runbooks/`.

## Usage

```
/save-that-step docker_postgres
/recall-step how did I set up the postgres volume?
```

## License

[MIT](LICENSE)
