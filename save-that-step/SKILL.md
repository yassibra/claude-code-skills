---
name: save-that-step
description: Save a step-by-step procedure from the current conversation (a stepper widget, a numbered how-to, or commands that were just run) as a reusable runbook in Notion, with secrets redacted. Use when the user says "save that step", "save this procedure", "garde ça dans Notion", or runs /save-that-step [slug].
argument-hint: "[slug, e.g. creation_bot_telegram]"
---

# save-that-step

Turn the procedure the user just followed into a clean, searchable runbook page so they never have to dig through old conversations again.

## 1. Find the procedure

Look back through the conversation, most recent first, and pick the procedure the user means:

- a stepper widget (the `widget_code` of a `show_widget` call: read the steps array, not the HTML chrome),
- a numbered list of instructions,
- or commands the user actually ran and that worked (skip the attempts that failed, but keep the fix as a "pitfall").

If several procedures are candidates and the user didn't say which, list them in one line each and ask. Otherwise don't ask, just go.

## 2. Build the runbook

Extract, in the user's language:

- **Title**: short and action-oriented ("Création d'un bot Telegram").
- **Slug**: `$ARGUMENTS` if given, otherwise a snake_case slug derived from the title (`creation_bot_telegram`).
- **Tags**: 1 to 4 tools or platforms involved (Telegram, AWS, Slack, Python…).
- **Summary**: one sentence, what you end up with.
- **Prerequisites**: accounts, tools, values needed before step 1.
- **Steps**: each step has a short title, the instruction, and every command, URL or code snippet in its own code block, copy-paste ready.
- **Variables**: every placeholder used in the steps, what it is, and where to find it.
- **Pitfalls**: errors met during the conversation and how they were fixed. This is often the most valuable part, keep it.

### Redact secrets (mandatory)

Before writing anything anywhere, replace every secret with a named placeholder and list the placeholder under Variables:

| Looks like | Becomes |
|---|---|
| Telegram bot token `123456789:AA…` | `<TELEGRAM_BOT_TOKEN>` |
| `xoxb-…`, `xoxp-…`, Slack signing secrets | `<SLACK_BOT_TOKEN>`, `<SLACK_SIGNING_SECRET>` |
| `AKIA…`, AWS secret keys | `<AWS_ACCESS_KEY_ID>`, `<AWS_SECRET_ACCESS_KEY>` |
| `sk-…`, `sk-ant-…`, `ghp_…`, `github_pat_…`, JWTs, passwords, private keys | `<OPENAI_API_KEY>`, `<ANTHROPIC_API_KEY>`, `<GITHUB_TOKEN>`… |
| Any other long random string used as a credential | `<NAME_OF_THING>` |

Also redact secrets inside URLs (`https://api.telegram.org/bot<TELEGRAM_BOT_TOKEN>/getUpdates`). Chat ids, region names, function names and other non-secret identifiers stay as-is. When in doubt, redact.

## 3. Save it

### Notion (preferred)

Use the Notion MCP tools (`notion-search`, `notion-fetch`, `notion-create-pages`, `notion-update-page`, `notion-create-database`, or whatever the connected Notion server calls them; load them with ToolSearch if they're deferred).

1. **Find the runbook database.** Read `~/.claude/save-that-step/config.json`. If it has a `notion_database_id`, use it. Otherwise search Notion for a database named "Runbooks" (or "Procédures"). If none exists, ask the user which page to create it under, then create it with these properties:
   - `Name` (title), `Slug` (text), `Tags` (multi-select), `Last updated` (date), `Times recalled` (number).

   Save it to `~/.claude/save-that-step/config.json` as `{"notion_database_id": "...", "notion_data_source": "collection://..."}`. Pages are created with the **data source** (`collection://…`) as parent, not the database id; if only the database id is known, fetch the database to get it.
2. **Check for an existing runbook** with the same slug (or an obviously equivalent title). If one exists, update it instead of creating a duplicate: merge new steps and pitfalls, keep what's still true, and add a line to the History section.
3. **Add missing tags first.** Notion rejects a multi-select value that isn't already an option. Fetch the data source, and if a tag is new, add it with `notion-update-data-source` (`ALTER COLUMN "Tags" SET MULTI_SELECT(...)` listing all existing options plus the new ones, so none are dropped).
4. **Otherwise create the page** in the database with the properties filled and this body. Notion's Markdown treats `< > [ ] { } |` as special outside code: put placeholders like `<TELEGRAM_BOT_TOKEN>` in inline code or code blocks, or escape them with a backslash. Use a callout for the summary and a Notion table for Variables if the server's Markdown spec supports them.

```markdown
> <summary>

## Prerequisites
- …

## Steps
### 1. <step title>
<instruction>
```bash
<command>
```

### 2. …

## Variables
| Placeholder | What it is | Where to find it |
|---|---|---|

## Pitfalls
- **<symptom>**: <fix>

## History
- <YYYY-MM-DD>: created from Claude Code
```

### Markdown fallback

If no Notion server is connected, write the same content to `~/.claude/save-that-step/runbooks/<slug>.md` with a YAML frontmatter (`title`, `slug`, `tags`, `updated`), and tell the user in one line that connecting Notion would put it there instead.

## 4. Confirm

Reply in two lines: the title and the link (or file path), plus how to get it back: `/recall-step <slug>`.
