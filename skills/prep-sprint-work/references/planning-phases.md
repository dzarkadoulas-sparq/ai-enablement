# Planning vs. implementation (plan mode)

This skill is **doc-first** and **review-first**. The agent should **separate** these phases in behavior and, when the product supports it, in **mode** (e.g. **Plan** mode in Cursor) for **phases 1–4**; **open the docs PR** after **phase 5** when documentation edits exist.

| Phase | Activity | Commits? |
| ----- | -------- | -------- |
| 1. **Select** | User or MCP **chooses** story key(s), including _both_ / _all_ / `KEY1 and KEY2` | No |
| 2. **Ingest** | Pull **fields**, comments, attachment **metadata**; summarize | No |
| 3. **Research** | **Codebase** map to AC; **unknowns** listed | No |
| 4. **Architecture review** (plan mode) | Propose **decisions** and **impact**; **stop** for user to confirm or **revise**; **no** app code | No (unless user only wants analysis in chat) |
| 5. **Doc updates** | **ADR** / `docs/architecture/*` edits per [`arch-doc-generator/SKILL.md`](../../arch-doc-generator/SKILL.md) and repo layout | **Yes** — on a **dedicated** branch, **doc-only** for default scope |
| 6. **Implementation plan** | Written plan + **dependency** **graph** (Mermaid or table) | Same **branch/PR** as **docs** is ideal |
| 7. **PR** | **Open** the documentation PR; optional **Jira/ADO** comment with link | N/A (PR is the artifact) |
| 8. **(Out of default scope)** | **Implement** the feature in code in follow-up work / another PR | User must **explicitly** request |

**Exit “plan mode”** means: user has **agreed** to the architecture direction and **doc** files are **ready to review**; the next concrete action is **commit + push + open PR** for `docs/`, not **merge** application code the same day unless the user changed scope.
