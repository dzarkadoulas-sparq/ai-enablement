---
name: conflict-resolver
description: >-
  Triages merge or rebase conflicts, auto-resolves trivial non-overlapping edits,
  uses history to explain intent for overlapping changes, proposes combined resolutions,
  runs tests after batches of fixes, and reports what was auto-fixed vs what needs a
  human. Use when resolving merge conflicts, rebase conflicts, or when a conflict
  marker appears in a file path the user names.
---

# Conflict resolver

## Goal

Get the repository from **conflicted** to **resolvable** or **clean** with **minimal** wrong merges: **never** drop a side’s change without **understanding** it. Stack-agnostic (Git only). For **aborting** a rebase and **backup** recovery, see [`git-cleanup`](../git-cleanup/SKILL.md) and [`../git-cleanup/references/conflict-resolution-patterns.md`](../git-cleanup/references/conflict-resolution-patterns.md).

## Preconditions

- **Know the operation:** `git merge` vs `git rebase` / `git cherry-pick` (see `references/merge-strategies.md` for **ours/theirs** meaning).
- **State:** `git status` — list **unmerged** paths and whether **rebase** or **merge** is in progress.

## Workflow (execute in order)

1. **Inventory** — `git status --porcelain` and `git diff --name-only --diff-filter=U`. List every **conflicted** file. Note **renames** and **binary** files (special handling in `merge-strategies.md`).

2. **Triage** — For each file, open the conflicted regions (or `git diff`):
   - **Trivial** — non-overlapping adds on different lines, one side only touches imports/whitespace and the other logic (still verify), duplicate conflict blocks that are identical on both sides.
   - **Complex** — same line or **overlapping** **semantic** edits (auth, API shape, two “fixes” that disagree), **binary**, **lockfile** with both sides changed, **generated** file where both sides regenerated.

3. **Trivial: auto-resolve** — Produce a merged file: **include both** non-overlapping hunks; **normalize** conflict markers **removed**; **format** to project style. `git add` the path.

4. **Complex: intent** — For each side, use **`git log --oneline -1 <commit>`** and **`git show <commit> -- path`** (or the commits Git prints during merge/rebase) to see **author intent**. Prefer a resolution that **preserves both** user-visible behaviors when possible; if impossible, **state the tradeoff** and recommend one side **plus** a follow-up task.

5. **Propose, don’t silently choose (complex)** — Output: **Option A** / **Option B** or a **synthetic** hunk. If the user must **decide** (e.g. product/ownership), mark **HUMAN** in the report and **leave** `<<<<<<<` in the file only if the tool workflow requires a manual edit—or use a **comment** with both alternatives and ask.

6. **Test batch** — After **all** files in a **coherent** batch (e.g. all conflicts in one package) are `git add`’d, run the **smallest** meaningful check: `pnpm test` subset, `dotnet test` on affected projects, or `./gradlew :module:test` if fast. If **no** test target is obvious, run **lint** or **build** for the touched **scope**. See **Test cadence** below.

7. **Continue the operation** — `git rebase --continue` / `git merge --continue` if Git reports ready; if **new** conflicts appear, return to step 1.

8. **Report** — Table: **file**, **trivial/auto** or **complex/manual**, **summary** of resolution, **test** result. **Remaining** **HUMAN** items with file:line or instruction.

## Test cadence

- **Prefer** one test run per **batch** of resolutions (5–20 files) to **balance** signal vs time—not after **every** single file unless the user asks.
- If tests are **expensive**, run **targeted** tests when the user or project docs name them; otherwise run **default** `test` script once at the end.

## Anti-patterns

- **Taking “ours” or “theirs”** for a **whole** file without reading (unless a side is **truly** obsolete and **git log** confirms).
- **Deleting** all conflict markers and leaving **invalid** syntax or **half** imports.
- **Resolving** **lockfile** by hand when **regenerating** (`pnpm install` / `npm` / `dotnet restore` per policy) is the team norm—**say so** in the report.

## Reference map

- `references/merge-strategies.md` — **merge** vs **rebase** semantics, `diff3`, checkout strategies, **binary** / **lockfile** notes, **intent** from history.
