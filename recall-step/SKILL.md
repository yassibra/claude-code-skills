---
name: recall-step
description: Bring back a runbook saved with save-that-step and walk the user through it again. Use when the user runs /recall-step <slug or question>, or asks how they did something before ("comment j'avais eu le chat id Telegram ?", "how did I deploy that lambda again?").
argument-hint: "<slug or question, e.g. creation_bot_telegram>"
---

# recall-step

The user did this once already. Find their runbook and hand it back, ready to follow.

## 1. Find the runbook

Query: `$ARGUMENTS` (if empty, use the user's last question).

**Notion**: read `notion_data_source` (or `notion_database_id`) from `~/.claude/save-that-step/config.json` and use the Notion MCP tools (load them with ToolSearch if deferred).

1. Look for an exact `Slug` match first.
2. Otherwise search the database by title, tags and content, and pick the best match.
3. Fetch the full page.

**Markdown fallback**: look in `~/.claude/save-that-step/runbooks/` for `<slug>.md`, then grep the folder for the query.

If nothing matches, list the closest 3 runbooks by title and slug. If there are none at all, say so and suggest running the procedure once, then `/save-that-step`.

If the user asked a precise question ("how do I get the chat id?"), answer that question first with the exact step and command, then offer the full runbook.

## 2. Show it

- If an inline widget tool is available (`show_widget`), render the runbook as an interactive stepper: one step per screen, numbered dots, previous and next buttons, commands in copyable code blocks, and a final "Done" button. Load the widget guidelines first (`read_me`) as that tool requires.
- Otherwise print the steps as a numbered list with code blocks.

List the Variables (placeholders) at the top so the user can gather their values before starting. Never ask the user to paste a secret into the chat to fill a placeholder; tell them where to put it (env var, `.env`, console field).

## 3. Keep it fresh

After showing it, increment `Times recalled` and set `Last updated` only if something changed. If during this session the user hits a new error or a step turns out to be outdated, offer to update the runbook with `/save-that-step <slug>`.
