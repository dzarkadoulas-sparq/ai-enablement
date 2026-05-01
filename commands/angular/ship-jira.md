Full "ship it" for Angular: verify, commit, push, PR, link ticket, notify.

Uses **GitHub MCP** + **Jira MCP** + **Slack MCP** + **Playwright MCP**.
Stop at first failure.

1. **Preconditions**
   - Branch must NOT be `main` or `develop`. If it is, stop.
   - Working tree must be non-empty. If empty, stop.

2. **Gate** — run everything `/pre-pr` runs (lint, test, a11y, build
   with budgets, audit). Stop on any failure.

3. **Visual verification** — if the diff touches `src/app/**`:
   - Start `pnpm start` if not running.
   - Playwright MCP: navigate each affected route, screenshot, check
     browser console.
   - Abort on new console errors.

4. **Extract the Jira key** from branch name (`feature/PROJ-123-...`).
   If not found, ask me.

5. **Commit**
   - Stage only files I touched. Never `git add -A`.
   - Conventional Commits from the diff:
     `type(scope): summary [PROJ-123]`. Body when warranted.

6. **Push** — `git push -u origin HEAD`.

7. **Open the PR via GitHub MCP**
   - Base: `develop`. Head: current branch.
   - Title: `[PROJ-123] <one-line summary>` (≤ 70 chars).
   - Body: `## Summary` + `## Test plan` + `## Screenshots` +
     `## Bundle impact` (route chunk size delta) + `Closes PROJ-123`.
   - Reviewers from `CODEOWNERS`.

8. **Link on Jira via Jira MCP**
   - Remote link to the PR on `PROJ-123`.
   - Transition `In Progress` → `In Review`.
   - Comment: `PR opened: <url>`.

9. **Post to Slack via Slack MCP**
   - Team channel for this repo.
   - `<PR title> — <PR url> — ready for review (<reviewers>)`.
   - No `@channel` / `@here` unless I ask.

10. **Report** — PR URL, Jira ticket URL, Slack message link,
    attached screenshots.

Constraints: never force-push; never push to `main` / `develop`; never
`--no-verify`; never raise bundle budgets to go green; never remove
`OnPush` or a11y assertions to go green.
