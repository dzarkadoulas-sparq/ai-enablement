Prepare my daily standup update using **Jira MCP** + git history.

1. **Yesterday — what I shipped**
   - `git log --author="$(git config user.email)" --since="24 hours ago" --all --no-merges --pretty=format:'%h %s'`.
   - Group by Jira key (e.g. `[PROJ-123]`).
   - Summarize each ticket from the diff, not the commit message.

2. **Today — what I'm on**
   - Via Jira MCP: `assignee = currentUser() AND sprint in openSprints()
AND status in ("In Progress", "In Review")`.
   - Include key, summary, status, and any linked PR.

3. **Blockers**
   - Via Jira MCP: `assignee = currentUser() AND sprint in openSprints()
AND status = "Blocked"`, plus In Progress with a comment in the
     last 48h mentioning "waiting", "blocked", or "need".
   - Include reason and who it's on.

4. **Format output exactly as**:

   ```
   **Yesterday:**
   - [PROJ-123] One-line summary of what shipped.

   **Today:**
   - [PROJ-124] One-line summary of what's next. (PR #456 open)

   **Blockers:**
   - [PROJ-125] One-line blocker with who it's on.
   ```

   Max 2 lines per bullet. No preamble, no sign-off.

5. **Do not post anywhere.** Just print the formatted block.
