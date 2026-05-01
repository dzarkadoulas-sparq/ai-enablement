---
name: standup-prep
description: >-
  Prepares a daily standup summary from local git activity, work-item tools, and
  open pull requests, then formats Yesterday / Today / Blockers. Can draft a
  Slack message if Slack MCP is available. Use for standup, daily update, what
  I did yesterday, sprint status, or prep for a team status meeting.
---

# Standup prep

## Goal

Produce a short, factual standup the user can paste into a meeting, Slack, or a ticket—without **inventing** ticket or PR state. Prefer data from **git** and **MCPs**; when a source is missing, say so explicitly.

## Ideal dependencies

- **Git** (terminal in the repo) — for recent commits by the current author.
- **At least one** work-item source: **Jira** (e.g. [Atlassian Rovo MCP](https://support.atlassian.com/rovo/docs/atlassian-remote-mcp-server/)) or **Azure DevOps** [MCP](https://learn.microsoft.com/azure/devops/mcp-server/mcp-server-overview)—for assignee state, sprint scope, and blocked work.
- **GitHub** (MCP or `gh` CLI) — for open PRs and review state (optional but useful).
- **Slack** MCP — optional, for drafting or posting when the user asks and approves (see [Slack MCP](https://api.slack.com/ai/slack-mcp-server)).

If work-item or Git tools are **not** configured, use **`references/mcp-queries.md`** (Fallback) and ask the user to supply ticket titles or paste from their board.

## Privacy and safety

- Do **not** post to Slack or a channel without **explicit** user request; default to a **draft** in chat (copy-paste ready).
- If the standup is for a **wide** audience, avoid customer identifiers or internal codenames when the user asks for that; confirm if unsure.

## Workflow

1. **Timebox** — “Yesterday” = last calendar day (or last business day if the user says so). “Today” = planned work (often from the board or stated by the user).
2. **Git** — `git log` in the date range, filtered to the current user’s `user.email` / `user.name` (see `mcp-queries.md`). Summarize in themes (not a raw hash dump) unless the user wants detail.
3. **Work items** — Query Jira/ADO MCP for in-progress, blocked, and “today’s” work when available; otherwise use a placeholder and ask the user to paste 2–3 items.
4. **PRs** — List open PRs and review state via GitHub MCP or `gh` when available; else mark TBD or point to the GitHub UI.
5. **Blockers** — From tooling (e.g. blocked status) **plus** anything the user names (CI, people, open questions).
6. **Format** — Use `references/standup-format.md`. Keep it short enough to read in ~30–60 seconds unless a longer 1:1 is requested.
7. **Slack** — If asked and the Slack MCP is available, post to the user-specified channel or thread; otherwise output Slack-flavored text for manual paste.

## Anti-patterns

- Fabricating ticket status or PR counts when tools were not used (say “not retrieved” or “see board” instead).
- Listing more than about **five** bullets for “Yesterday”—roll up into themes.

## Reference map

- `references/standup-format.md` — Yesterday / Today / Blockers templates.
- `references/mcp-queries.md` — Git commands, MCP query ideas, fallbacks, and links to host install docs (also see the project root `README` MCP table).

## Exemplar prompts (ai-enablement)

- Jira- and Azure DevOps–oriented command examples live under `commands/<stack>/standup-jira.md` and `commands/<stack>/standup-ado.md` in this repository. Use the same Jira vs. ADO distinction when choosing which work-item MCP to prefer.
