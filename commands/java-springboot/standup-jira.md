Prepare my daily standup update using **Jira MCP** + git history.

1. **Yesterday — what I shipped**
   - `git log --author="$(git config user.email)" --since="24 hours ago" --all --no-merges --pretty=format:'%h %s'`.
   - Group commits by Jira ticket key (e.g. `[PROJ-123]` from commit message).
   - For each ticket, summarize in one sentence based on the diff — not the
     commit message (commit messages lie).

2. **Today — what I'm on**
   - Via Jira MCP: query `assignee = currentUser() AND sprint in openSprints()
AND status in ("In Progress", "In Review")`.
   - For each, include: key, summary, status, and any PR linked to it
     (check ticket's remote links or commit history).

3. **Blockers**
   - Via Jira MCP: query `assignee = currentUser() AND sprint in openSprints()
AND status = "Blocked"` — AND any In Progress ticket with a comment
     mentioning "waiting", "blocked", or "need" in the last 48h.
   - For each blocker, include the reason (from the latest comment) and who
     it's blocked on.

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

5. **Do not post anywhere.** Just print the formatted block for me to paste.
