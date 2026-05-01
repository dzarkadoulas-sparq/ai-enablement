Prepare my daily standup update using **Azure DevOps MCP** + git history.

1. **Yesterday — what I shipped**
   - `git log --author="$(git config user.email)" --since="24 hours ago" --all --no-merges --pretty=format:'%h %s'`.
   - Group commits by ADO work item ID (`#1234`, `AB#1234`, or trailer).
   - Summarize each item in one sentence from the diff, not the commit message.

2. **Today — what I'm on**
   - Via ADO MCP: `wit_my_work_items` in the current iteration, states
     `Active` / `In Progress` / `Committed` / `Code Review`.
     Resolve the current iteration via `work_list_team_iterations`
     (`timeframe = current`).
   - Include ID, Title, State, and any linked PR from `wit_get_work_item`.

3. **Blockers**
   - Via ADO MCP: items in `Blocked` state in this iteration, plus `Active`
     items whose latest comment (`wit_list_work_item_comments`) in the last
     48h mentions "blocked", "waiting", or "need".
   - Include the reason and who it's on.

4. **Format output exactly as**:

   ```
   **Yesterday:**
   - [AB#1234] One-line summary of what shipped.

   **Today:**
   - [AB#1235] One-line summary of what's next. (PR !567 open)

   **Blockers:**
   - [AB#1236] One-line blocker with who it's on.
   ```

   Max 2 lines per bullet. No preamble, no sign-off.

5. **Do not post anywhere.** Just print the formatted block.
