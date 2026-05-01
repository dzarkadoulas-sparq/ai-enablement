Full "ship it" workflow: verify, commit, push, PR, link ticket, notify team.

Uses **GitHub MCP** + **Jira MCP** + **Slack MCP**.

Stop at the first failing step; never skip or bypass a failure.

1. **Preconditions**
   - Branch must NOT be `main` or `develop`. If it is, stop.
   - Working tree must be non-empty (`git status`). If empty, stop.

2. **Gate** — run everything `/pre-pr` runs: `spotlessApply`, `check`,
   `build`, `integrationTest`. If any fail, stop and surface the failure.

3. **Extract the Jira key** from the branch name (expected pattern:
   `feature/PROJ-123-short-description`). If no key is found, ask me.

4. **Commit**
   - Stage only files I touched. Never `git add -A`.
   - Write a Conventional Commits message from the diff:
     `type(scope): summary [PROJ-123]`. Include a body only when the diff
     warrants one.

5. **Push** — `git push -u origin HEAD`.

6. **Open the PR via GitHub MCP**
   - Base: `develop`. Head: current branch.
   - Title: `[PROJ-123] <one-line summary>` (≤ 70 chars).
   - Body: `## Summary` (2–3 bullets) + `## Test plan` (checklist) +
     `Closes PROJ-123`. Include any breaking-change callouts.
   - Request reviewers per `CODEOWNERS` if present, otherwise the team lead.

7. **Link the PR on the Jira ticket via Jira MCP**
   - Add a remote link to the PR URL on `PROJ-123`.
   - Transition the ticket from `In Progress` → `In Review`.
   - Add a one-line comment: `PR opened: <url>`.

8. **Post to Slack via Slack MCP**
   - Channel: `#backend-team` (or the team channel configured for the repo).
   - Message: `<PR title> — <PR url> — ready for review (<reviewers>)`.
   - No `@channel`, no `@here` unless I explicitly say so.

9. **Report** — print the PR URL, Jira ticket URL, and Slack message link.

Constraints: never force-push; never push to `main` / `develop`; never
bypass failing checks with `--no-verify`; never weaken security or tests
to go green.
