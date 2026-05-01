# Data sources: git, GitHub, Jira, ADO, Slack

Use the **MCPs** the user has enabled. If a tool is missing or returns an error, say so in the standup and use **Fallback** at the end.

## Git (local)

```bash
# Recent commits by the configured author (adjust the date as needed)
git log --since="1 day ago" --author="$(git config user.email)" --oneline --no-merges
```

- If your commits use a **different** email, try `git log --all --format='%ae' | sort -u` to find the right `--author=…` value or match on name.

## GitHub

- **MCP:** use list/search for **open pull requests** where the user is author, assignee, or requested reviewer—exact tool names depend on the GitHub MCP in use.
- **CLI** fallback: `gh pr list` with filters, e.g. `gh pr list --author @me` or `gh pr status` (see [GitHub CLI](https://cli.github.com/manual/gh_pr_list)).

## Jira / Atlassian Cloud

- Query work items where **assignee = current user**, status in **In Progress** / **Blocked** (or the team’s equivalents), limited to the **active sprint** or board the user names.
- Setup: [Atlassian Rovo MCP Server](https://support.atlassian.com/rovo/docs/atlassian-remote-mcp-server/); for IDE flow see [Setting up IDEs (desktop clients)](https://support.atlassian.com/atlassian-rovo-mcp-server/docs/setting-up-ides/).

## Azure DevOps

- Query **assigned** work items in **Active** (or your team’s states), scoped to a sprint/iteration when possible.
- Docs: [Azure DevOps MCP Server](https://learn.microsoft.com/azure/devops/mcp-server/mcp-server-overview) and the **remote** server preview: [Set up the remote Azure DevOps MCP Server](https://learn.microsoft.com/azure-devops/mcp-server/remote-mcp-server) (or this repo’s root `README`).

## Slack (post only if the user asked)

- Draft in chat first. Posting requires channel/thread details from the user. Product overview: [Slack MCP server](https://api.slack.com/ai/slack-mcp-server).

## Host install (Cursor & Claude Code)

- [Model Context Protocol (MCP) — Cursor](https://cursor.com/docs/mcp)
- [Connect Claude Code to tools via MCP](https://docs.anthropic.com/en/docs/claude-code/mcp/)

## Fallback (no Jira, ADO, or GitHub data)

1. **Yesterday** — from **git** commit themes only, clearly labeled.
2. **Today** / **tickets** — ask the user to paste **2–3** active ticket keys/titles, or a link to the sprint board.
3. **PRs** — “Open PRs: _check GitHub manually_” or `gh pr list` if the CLI is installed.
4. **Never** invent work-item state; use “TBD” or “not synced.”
