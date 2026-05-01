Full "ship it" workflow for FastAPI: verify, commit, push, PR, link ticket, notify.

Uses **GitHub MCP** + **Jira MCP** + **Slack MCP**. Stop at first failure.

1. **Preconditions**
   - Branch must NOT be `main` or `develop`. If it is, stop.
   - Working tree must be non-empty. If empty, stop.

2. **Gate** — run everything `/pre-pr` runs: `ruff`, `mypy`, `pytest`
   (unit + integration), `pip-audit`. Stop on any failure.

3. **Extract the Jira key** from branch name (`feature/PROJ-123-...`).
   If not found, ask me.

4. **Commit**
   - Stage only files I touched. Never `git add -A`.
   - Conventional Commits message from the diff:
     `type(scope): summary [PROJ-123]`. Body when warranted.

5. **Push** — `git push -u origin HEAD`.

6. **Open the PR via GitHub MCP**
   - Base: `develop`. Head: current branch.
   - Title: `[PROJ-123] <one-line summary>` (≤ 70 chars).
   - Body: `## Summary` + `## Test plan` + `Closes PROJ-123`.
     Note breaking API changes and any new env vars.
   - Reviewers from `CODEOWNERS`.

7. **Link on Jira via Jira MCP**
   - Add a remote link to the PR URL on `PROJ-123`.
   - Transition `In Progress` → `In Review`.
   - Comment: `PR opened: <url>`.

8. **Post to Slack via Slack MCP**
   - Team channel for this repo.
   - `<PR title> — <PR url> — ready for review (<reviewers>)`.
   - No `@channel` / `@here` unless I ask.

9. **Report** — PR URL, Jira ticket URL, Slack message link.

Constraints: never force-push; never push to `main` / `develop`; never
`--no-verify`; never weaken tests or security to go green.
