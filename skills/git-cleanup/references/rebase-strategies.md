# Rebase strategies (cleanup)

## Choose the base

- **Merge base** with mainline: `git merge-base origin/main HEAD` (or `origin/develop`). Interactive rebase onto that: `git rebase -i <merge-base>`.
- **Simpler mental model** — rebase the **last N** commits: `git rebase -i HEAD~N` (count must include **all** work to squash).

## Interactive todo verbs

- **pick** — use commit as-is
- **reword** or **r** — change only message
- **edit** or **e** — stop to amend (split a commit: `git reset HEAD^` then re-commit in pieces)
- **squash** or **s** — meld into previous, combine messages
- **fixup** or **f** — like squash but **discard** this commit’s message
- **drop** or **d** — remove commit (danger: loses changes unless already elsewhere)

**Order** matters: commits are **replayed** top-to-bottom as **oldest to newest** in the file (first line = oldest of the selected range).

## Autosquash

- Commits with messages **`fixup! <subject>`** or **`squash! <subject>`** (git **commit --fixup/--squash**) are auto-reordered and folded when you run:
  - `git rebase -i --autosquash <base>`
- Requires `rebase.autoSquash` true (often default) or pass **`--autosquash`** explicitly.

## Splitting a commit (edit)

1. Mark as **edit** in the todo, save.
2. When rebase stops: `git reset HEAD^` (soft) or mixed, then `git add -p` and `git commit` in multiple commits.
3. `git rebase --continue`.

## Moving commits (`--onto`)

- **Remove** commits from the bottom of a stack or **reparent**:  
  `git rebase --onto <newbase> <oldbase> <branch>`
- Use when **base branch moved** or you need to **drop** a range of old commits. **Draw** the graph before running.

## After upstream changed (rebase vs merge)

- If **origin/main** advanced, **rebase** your work: `git fetch` then `git rebase origin/main` (or `origin/develop`) **before** or **after** local cleanup, depending on whether you want to **squeeze** first then rebase, or rebase first then **squeeze**— usually **rebase onto latest** first, **then** interactive **cleanup** to avoid re-editing twice.

## Safety

- **Never** `rebase` in the middle of **uncommitted** work without **stash** (or the user is aware). **`git rebase --abort`** restores pre-rebase state (except completed steps—**backup** branch is the net).

## Force-with-lease

- After any history rewrite: **`git push --force-with-lease`**, not `--force`, to avoid **clobbering** others’ pushes to the same branch.
