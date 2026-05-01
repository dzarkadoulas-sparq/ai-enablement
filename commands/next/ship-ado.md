Full "ship it" for Next.js: verify, commit, push, PR, link work item, notify.

Uses **GitHub MCP** + **Azure DevOps MCP** + **Slack MCP** + **Playwright MCP**.
Stop at first failure.

1. **Preconditions**
   - Branch must NOT be `main` or `develop`. If it is, stop.
   - Working tree must be non-empty. If empty, stop.

2. **Gate** — run everything `/pre-pr` runs. Stop on any failure.

3. **Visual verification** — if the diff touches `app/**`, `components/**`,
   or `features/**`:
   - Start `pnpm dev` if not running.
   - Playwright MCP: navigate each affected route, screenshot, check
     browser console AND dev-server stdout.
   - Abort on any new error.

4. **Extract the ADO work item ID** from branch name
   (`feature/AB1234-...` or `feature/1234-...`). If not found, ask me.

5. **Commit**
   - Stage only files I touched. Never `git add -A`.
   - Conventional Commits from the diff:
     `type(scope): summary (AB#1234)`. Body when warranted.

6. **Push** — `git push -u origin HEAD`.

7. **Open the PR via GitHub MCP**
   - Base: `develop`. Head: current branch.
   - Title: `AB#1234 <one-line summary>` (≤ 70 chars).
   - Body: `## Summary` + `## Test plan` + `## Screenshots` +
     `## Caching impact` + `Fixes AB#1234`.
   - Reviewers from `CODEOWNERS`.

8. **Link the work item via Azure DevOps MCP**
   - `wit_link_work_item_to_pull_request` — `1234` ↔ PR URL.
   - `wit_update_work_item` — state `Active` → `Code Review`.
   - `wit_add_work_item_comment` — `PR opened: <url>`.

9. **Post to Slack via Slack MCP**
   - Team channel for this repo.
   - `<PR title> — <PR url> — ready for review (<reviewers>)`.
   - No `@channel` / `@here` unless I ask.

10. **Report** — PR URL, ADO work item URL, Slack message link,
    attached screenshots.

Constraints: never force-push; never push to `main` / `develop`; never
`--no-verify`; never weaken tests or security; never add `"use client"`
just to go green.
