# Conflicts during rebase (and merge)

## Diagnose

- `git status` — **both modified**, **unmerged** paths, **rebase in progress** vs **merge in progress**.

## Trivial (often)

- **Both sides** added **different** lines in **distant** regions—often safe to keep **both** with correct order.
- **One side** deleted, **one side** changed—needs **product** intent: keep **deletion** vs keep **new** line.

## Read the markers

```
<<<<<<< HEAD
ours (rebase: **current** replayed commit / **onto**)
=======
theirs (rebase: **incoming** patch)
>>>>>>> <commit>
```

- In **`git rebase`**, “ours”/“theirs” are **reversed** vs `git merge` mentally—**re-read** the conflict docs for the **current** command (`git help rebase` conflicts section).

## Resolution workflow

1. **Open** each file; **remove** conflict markers; leave **one** coherent result.
2. `git add <path>` when fixed.
3. `git rebase --continue` (or `git merge --continue`).

## When stuck

- **`git rebase --abort`** — back to state **before** this rebase started (if no **backup** ref, the user’s reflog is still a fallback; `git reflog` to find pre-rebase HEAD).
- **Restore** from `backup/...` branch:  
  `git checkout <feature>`  
  `git reset --hard backup/<feature>-<date>`

## Testing after a batch

- Per **conflict-resolver** skill, run **tests** after a **set** of files are resolved, not one line at a time, when **cheap**.

## Asking a human

- **Same line** different **semantics** (e.g. two “fixes” to **auth** that aren’t combinable) — **stop**; propose both snippets and a **recommendation**, let the user **choose** or get **code owner** input.

## Large conflicts

- Prefer **theirs** or **ours** **whole file** only when one side is clearly **obsolete** (e.g. **rename** completed on one side). **Never** without **reading** the diff of both.
