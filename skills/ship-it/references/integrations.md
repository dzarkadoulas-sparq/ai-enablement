# PR, work items, and Slack

Official install pointers for MCP also appear in the **project root** `README` (MCP table).

## GitHub: MCP vs CLI

- **GitHub MCP** (when configured): create PR, set title/body, request reviewers if the server supports it.
- **`gh` CLI** (when logged in): `gh pr create --base <branch> --head <branch> --title "…" --body "…"`, `gh pr view --web`, `gh pr edit`.
- **Fallback:** Give the user the **compare** URL: `https://github.com/<owner>/<repo>/compare/<base>...<head>` and a paste-ready title/body.

## Reviewers

1. **CODEOWNERS** (if present) — primary signal.
2. **Team list** in `CONTRIBUTING` or `CLAUDE.md`.
3. **`git blame`** on hot paths — **suggestions only**; can mislead for moved files or refactors.

## Jira (Atlassian MCP)

- Link PR URL to the issue (remote link or comment, per org).
- Transition (e.g. to **In Review**) only if the workflow matches what the user expects.
- Docs: [Atlassian Rovo / MCP setup](https://support.atlassian.com/rovo/docs/atlassian-remote-mcp-server/).

## Azure DevOps (ADO MCP)

- Add PR link to work item; set state to match team process (e.g. **Active** / **Resolved** with PR link, depending on process).
- Docs: [Azure DevOps MCP overview](https://learn.microsoft.com/azure/devops/mcp-server/mcp-server-overview).

## Slack

- **Default:** Output a **draft** message the user can paste.
- **Post** only with explicit request and channel; avoid `@channel` / `@here` unless asked.
- Docs: [Slack MCP](https://api.slack.com/ai/slack-mcp-server).

## If nothing but Git works

- Ship = commit + push + “open PR from compare URL” + user manually links ticket and Slack.
