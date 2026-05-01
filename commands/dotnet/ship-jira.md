Full "ship it" workflow for .NET: verify, commit, push, PR, link ticket, notify.

Uses **GitHub MCP** + **Jira MCP** + **Slack MCP**. Stop at first failure.

1. **Preconditions**
   - Branch must NOT be `main` or `develop`. If it is, stop.
   - Working tree must be non-empty. If empty, stop.

2. **Gate** — run everything `/pre-pr` runs: `dotnet format`,
   `dotnet build -warnaserror`, unit tests, integration tests.
   If any fail, stop and surface the failure.

3. **Extract the Jira key** from branch name (expected:
   `feature/PROJ-123-...`). If not found, ask me.

4. **Commit**
   - Stage only files I touched. Never `git add -A`.
   - Conventional Commits message from the diff:
     `type(scope): summary [PROJ-123]`. Body only when the diff warrants.

5. **Push** — `git push -u origin HEAD`.

6. **Open the PR via GitHub MCP**
   - Base: `develop`. Head: current branch.
   - Title: `[PROJ-123] <one-line summary>` (≤ 70 chars).
   - Body: `## Summary` (2–3 bullets) + `## Test plan` (checklist) +
     `Closes PROJ-123`. Note breaking changes.
   - Request reviewers from `CODEOWNERS` or the team lead.

7. **Link the PR on Jira via Jira MCP**
   - Add a remote link to the PR on `PROJ-123`.
   - Transition `In Progress` → `In Review`.
   - Comment: `PR opened: <url>`.

8. **Post to Slack via Slack MCP**
   - Channel: the team channel for this repo.
   - Message: `<PR title> — <PR url> — ready for review (<reviewers>)`.
   - No `@channel` / `@here` unless I explicitly ask.

9. **Report** — PR URL, Jira ticket URL, Slack message link.

Constraints: never force-push; never push to `main` / `develop`; never
bypass failing checks with `--no-verify`; never weaken tests or security
to go green.
