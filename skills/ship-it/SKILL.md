---
name: ship-it
description: >-
  End-to-end flow from local changes to open pull request: run the pre-PR quality
  gate, optionally clean commit history, stage intelligently, commit with
  Conventional Commits, push, open a PR, link the work item (Jira or Azure DevOps),
  and draft or post Slack when the user asks. Use for ship it, push this, create
  PR, open PR, send for review, or put this up for review.
---

# Ship it

## Goal

Move the current work from a **feature branch** to an **open pull request** on the remote, with a **passing** quality gate, a **traceable** commit message, and **optional** updates to the work tracker and team chat. **Do not** skip checks, weaken tests, or push to protected branches to “go green.”

## Composes other skills (do not reimplement)

1. **Quality gate** — Follow the **`pre-pr-review`** skill: run the repo’s lint, tests, and stack checks, review the diff, and produce **P0–P2** findings. **Stop** on P0 blockers unless the user explicitly overrides (document that override).
2. **History (optional)** — If the branch has many WIP or fixup commits and the user wants a clean story, use the **`git-cleanup`** skill before the final commit/push. If the user only has uncommitted work, a single new commit is usually enough.

## Ideal dependencies

- **Git** — local branch, remote `origin` (or team default).
- **GitHub** — [GitHub MCP](https://github.com/github/github-mcp-server) or the **`gh` CLI** to create the PR and set metadata.
- **Work items (one of)** — [Atlassian (Jira) MCP](https://support.atlassian.com/rovo/docs/atlassian-remote-mcp-server/) or [Azure DevOps MCP](https://learn.microsoft.com/azure/devops/mcp-server/mcp-server-overview) to link the PR and transition status.
- **Slack** — [Slack MCP](https://api.slack.com/ai/slack-mcp-server) only when the user **asks** to notify the team; default to a **draft** in chat.

If GitHub MCP is missing, use **`gh`** when installed; otherwise print exact `git` / browser steps. If Jira/ADO/Slack is missing, see `references/integrations.md` (Fallback).

## Preconditions (stop if violated)

- Current branch is **not** `main`, `master`, or the team’s primary integration branch (e.g. `develop`) **unless** the user explicitly wants a hotfix flow and names the base.
- **Working tree** and **index** state are clear: either commit-ready after staging, or the user agreed to stash / WIP commit.
- **No** `--no-verify` and **no** bypassing required checks to pass the gate.

## Workflow (execute in order)

1. **Delegate pre-PR** — Run the full **`pre-pr-review`** workflow for this repo/stack. Address or report P0s; get user direction on P1s if they block ship.
2. **History choice** — If there are **multiple** small commits and the user asked for a “clean” branch, offer **`git-cleanup`**; otherwise prepare **one** Conventional commit for the current staged/unstaged work (see `references/conventional-commit.md`).
3. **Stage** — Stage **only** files that belong to this change (use `git status`, `git diff`; prefer **explicit paths** or **`git add -p`**). Avoid **`git add -A`** unless the user confirms the whole tree is intended.
4. **Commit** — Message format: Conventional Commits, optional ticket key in the subject or footer (see `pre-pr-review` / `git-cleanup` references in this repo). Derive `type`/`scope` from the diff (`references/conventional-commit.md`).
5. **Push** — `git push -u origin HEAD` (or the remote/branch the user names). **Never** force-push without explicit user request and team policy; prefer **`--force-with-lease`** if rewrite happened.
6. **Open PR** — Base branch from repo convention (`develop`, `main`, or `CLAUDE.md` / `CONTRIBUTING`). Title: concise; body: Summary, Test plan, ticket link / `Closes` / `Fixes` as appropriate. Use GitHub MCP or `gh pr create` (see `references/integrations.md`).
7. **Reviewers** — Prefer **`CODEOWNERS`** and team docs. Suggesting reviewers from **`git blame`** is **optional** and may be wrong; label as suggestions.
8. **Work item** — If Jira/ADO MCP is available: add a **remote link** or **comment** with the PR URL; transition (e.g. In Review) per team rules. If not available, give the user a **checklist** to paste.
9. **Slack** — Only if the user asked: draft or post a short line (title + URL + optional reviewers). **No** `@channel` / `@here` unless they asked.

10. **Report** — PR URL, branch, base, ticket id (if any), and what was **not** automated (MCP missing, manual step).

## Anti-patterns

- Opening a PR without running the same checks **`pre-pr-review`** would run (or clearly marking that the user skipped them).
- Staging unrelated files, secrets, or generated artifacts the team does not commit.
- Fabricating “PR created” or “ticket updated” when tools were not called or failed.
- Posting to Slack or pinging people without **explicit** user consent.

## Reference map

- `references/conventional-commit.md` — Deriving type, scope, and ticket from the diff.
- `references/integrations.md` — GitHub MCP vs `gh`, Jira vs ADO, Slack, fallbacks.

## Exemplar prompts (ai-enablement)

- Stack-specific **Jira** vs **Azure DevOps** ship recipes: `commands/<stack>/ship-jira.md` and `commands/<stack>/ship-ado.md`. Match the work-item system to the right MCP and command file.
