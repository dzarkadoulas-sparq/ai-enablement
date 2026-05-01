---
name: git-cleanup
description: >-
  Analyzes a feature branch against its merge base, finds squash and fixup
  candidates, proposes a Conventional Commits history, creates a backup ref before
  rewriting history, guides or runs interactive rebase, verifies tests after rewrite,
  and reports before/after. Use when the user wants to clean up a branch, squash
  commits, interactive rebase, fix commit history, or prepare a branch for merge.
---

# Git cleanup (history rewrite)

## Goal

Turn a **noisy** branch (many WIP, fixup, typo, or “address review” commits) into a **small set of logical commits** with **Conventional Commits**-style messages—**without** losing work. This skill is **VCS operations + analysis**; it does not depend on app stack (Node vs Java, etc.).

## Preconditions (verify first)

- **Working tree** clean or user accepts **stash** / **WIP commit** (state explicitly).
- **Current branch** is a **feature branch**, not the shared default (`main` / `master` / `develop` unless the user says otherwise).
- **Upstream** — know the **base** to compare: `origin/main`, `origin/develop`, or `git merge-base` with the team’s **integration** branch. **Ask** if multiple remotes or unclear default.
- **Force-push policy** — after rewrite, **only** the user runs `git push --force-with-lease` (or the team’s policy). **Never** suggest blind `--force` to `main`.

## Workflow (execute in order)

1. **Fetch and measure** — `git fetch` (relevant remotes). List commits: `git log --oneline <base>..HEAD` and optionally `git log <base>..HEAD --stat` to see file churn. Record **count** and **date range**.

2. **Classify commits** (see `references/rebase-strategies.md`) — mark candidates for **squash**, **fixup**, **reword**, or **split** (rare: note as manual).

3. **Group by intent** — cluster commits that touch the **same feature** or **same ticket**. Use `git show --stat` / `git diff` between commits if needed. One ticket may map to **one** or **several** commits (e.g. `feat` + `test` if team prefers).

4. **Propose new history** — table: **old** (hash + subject) → **new** (single line **Conventional** message + optional body). Read `references/conventional-commits.md`. **P0** on the user’s **Conventional** / team rules from `CONTRIBUTING` if present.

5. **Backup** — `git branch backup/<user-branch>-<YYYYMMDD>` (or `refs/heads/backup/...`) pointing at **current HEAD** before any rewrite. Confirm with `git show-ref` / `git log -1 backup/...`.

6. **Rewrite** — **Interactive rebase:** `git rebase -i <base>` (or `git rebase -i --onto` if the situation matches `rebase-strategies.md`). If the environment cannot open an editor, use **`GIT_SEQUENCE_EDITOR`** with a script that applies the **pick/squash/fixup/reword** list you already proposed (document the exact file content). **Autosquash:** if commits are already `fixup!` / `squash!`, `git rebase -i --autosquash <base>` can reduce manual editing.

7. **Verify** — run the project’s **test** command from `package.json` / `Makefile` / CI, or the **smallest** check the user names (`pnpm test`, `./gradlew test`, `dotnet test`, etc.). If **no** test command in 30s of inspection, run `git diff <base>..HEAD --stat` and state **test gap**.

8. **Report** — **Before:** N commits, summary. **After:** M commits, new subjects. **Backup ref** name. **Next steps:** e.g. `git push --force-with-lease origin <branch>` (user runs), or open PR if not yet.

## If rebase hits conflicts

Stop the automated narrative; apply `references/conflict-resolution-patterns.md`. Either **help resolve** (user or agent edits files) or **abort** with `git rebase --abort` and restore from **backup** branch: `git reset --hard backup/...`.

## Anti-patterns

- **Rewriting** **shared** / **protected** **default** branch history without explicit org policy and user confirmation.
- **Dropping** commits that contain **co-authored** or **merge** of **other people’s** work without checking **authorship** (`git log --format=fuller`).
- **Omitting** a **backup** ref before `rebase` / `reset`.
- **Assuming** `main` is the base when the team uses **`develop`** or **trunk** naming.

## Reference map

- `references/conventional-commits.md` — types, scopes, body, breaking changes.
- `references/rebase-strategies.md` — squash, fixup, autosquash, onto, range.
- `references/conflict-resolution-patterns.md` — when rebase stops; recover or abort.
