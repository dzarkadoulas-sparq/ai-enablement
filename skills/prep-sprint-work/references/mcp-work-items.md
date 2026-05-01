# Jira, Azure DevOps, and fallbacks

Official links and host config also appear in the **project root** `README` (MCP table) and in [`standup-prep/references/mcp-queries.md`](../../standup-prep/references/mcp-queries.md) (sibling under `skills/`).

## When no story key is given

- **Jira (Atlassian MCP):** Query for issues **assigned to the current user**, typically **status** in the team’s in-progress (or _To Do_ + ordered backlog for “next to prep”). Restrict by **board**, **sprint** or **JQL** if the user names a project, e.g. `project = PROJ AND assignee = currentUser() AND sprint in openSprints() AND status in ("To Do", "In Progress")` (exact JQL is team-specific).
- **Azure DevOps MCP:** **Wiql** or the server’s **work item** API for **@Me**-assigned, **In Progress** / **Active** in the current **Iteration Path** the user names.

Present results as: **key**, **title**, **type**, **status** — one table or short list — then: “Which one(s) should I prepare? You can also say _both_, _all_, or `KEY1 and KEY2`.”

## For each selected issue, fetch (when the API allows)

- Title, **description** (or ADF/plain), **issuetype**
- **Acceptance criteria** (custom field, checklist, or embedded in description — parse and repeat clearly)
- **Comments** — list with author, date, excerpt/summary
- **Attachments** — **filename, size, mime**; _do not_ auto-execute. Prefer asking the user for a short description of the attachment or paste if small text
- **Links** to parent, **blocks**, or **relates to**; include related keys for multi-item prep
- **Labels**, **components**, **fix version** (for scope and release planning in the document)

## GitHub (optional)

- If the story is linked to a **spike** PR or design doc, use **GitHub MCP** / `gh` to pull the PR **title and body** (same repo). Not required for every run.

## Fallback (no Jira/ADO)

1. Ask: “Paste **keys and titles** (e.g. `PROJ-123: …`, `PROJ-124: …`) or a **sprint** link and which rows apply.”
2. Ask for **description and acceptance criteria** in **paste** form for each.
3. **Continue** with codebase research; label work-item section **TBD: not in tracker**.

## Security

- **Credentials** in descriptions/comments — **redact** in all outputs and in PR body.
- **Attachments** — do not run executable attachments; for PDFs, prefer a **one-line** user summary when OCR is unavailable.
