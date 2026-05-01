# Module 4 — Git & Version Control Workflows (Advanced)

## Detailed Lesson Plans with Examples

> **Prerequisites:** Module 3 covered basic Git operations through MCP — committing,
> pushing, creating PRs, and writing commit messages. This module focuses exclusively
> on advanced Git workflows where Claude Code provides the most leverage: cleaning up
> messy history, resolving painful conflicts, and conducting thorough code reviews.

---

## Lesson 4.1 — Interactive Rebasing with Claude Code

Interactive rebasing is one of Git's most powerful and most feared features. It lets you rewrite commit history — squashing, reordering, splitting, editing, and rewording commits before pushing or merging. Claude Code removes the fear factor by handling the mechanical complexity while you make the editorial decisions.

### When to Use Interactive Rebase

- Your branch has a messy history of "WIP", "fix typo", "actually fix it this time" commits
- You want to present a clean, logical sequence of changes for reviewers
- You need to split a large commit into smaller, focused ones
- You need to reorder commits so the PR tells a coherent story

### Example 1: Cleaning Up a Messy Branch Before PR

Your branch has 12 commits that look like this:

```
a1b2c3 Add user settings page
d4e5f6 WIP: settings form
g7h8i9 fix typo in settings
j1k2l3 Add email notification preferences
m4n5o6 forgot to add the CSS
p7q8r9 fix broken test
s1t2u3 actually fix the test for real
v4w5x6 Add password change form
y7z8a9 PR feedback: rename variable
b1c2d3 PR feedback: add input validation
e4f5g6 oops, fix import
h7i8j9 Add account deletion with confirmation modal
```

Ask Claude Code to clean this up:

```
Look at the commit history on this branch compared to main.
The history is messy — lots of WIP commits, typo fixes, and
incremental corrections that should be squashed together.

Propose a clean commit structure that groups related changes
into logical commits. Show me the plan before rebasing.
The final history should tell a clear story that a reviewer
can follow commit-by-commit.
```

Claude Code will analyze the commits and propose something like:

> **Proposed clean history (4 commits):**
>
> 1. `feat: add user settings page with form layout`
>    ← squash a1b2c3 + d4e5f6 + g7h8i9 + m4n5o6
> 2. `feat: add email notification preferences`
>    ← keep j1k2l3 standalone (already clean)
> 3. `feat: add password change form with validation`
>    ← squash v4w5x6 + y7z8a9 + b1c2d3 + e4f5g6
> 4. `feat: add account deletion with confirmation modal`
>    ← squash h7i8j9 + p7q8r9 + s1t2u3 (test fixes were for this feature)

You review and approve:

```
That grouping looks right, except move the test fixes (p7q8r9
and s1t2u3) into commit 1 — the tests were for the settings
page, not account deletion.

Go ahead and execute the interactive rebase with these
groupings. Write good commit messages for each squashed commit.
```

Claude Code will run the interactive rebase, handle the squash/fixup operations, write descriptive commit messages for each resulting commit, and force-push the cleaned branch.

### Example 2: Splitting a Commit That Does Too Much

Sometimes you have a single commit that changes three unrelated things:

```
Look at commit f8a3b21 on this branch. It changes the user model,
the auth middleware, AND the dashboard template. These are three
unrelated changes that got committed together by accident.

Split this commit into three separate commits:
1. User model changes only
2. Auth middleware changes only
3. Dashboard template changes only

Use interactive rebase with the `edit` command to split it.
Preserve the original commit's date. Write a clear message
for each new commit.
```

Claude Code will:

1. Start an interactive rebase marking that commit as `edit`
2. Soft-reset to unstage the changes
3. Stage and commit each group of files separately
4. Continue the rebase
5. Handle any conflicts that arise from the reorder

### Example 3: Rewording Commits to Follow Team Convention

```
Our team uses Conventional Commits. Look at all commits on this
branch and reword any that don't follow the format:
type(scope): description

Types we use: feat, fix, refactor, test, docs, chore, perf
Scopes should match our module names: auth, users, orders,
payments, admin

Don't change any code — only reword the commit messages.
Show me the before/after for each message so I can approve.
```

### Good Practices

**Always review the rebase plan before executing:**

```
/plan I want to clean up this branch's history before merging.
Show me the current commits and your proposed grouping.
Don't execute anything yet.
```

Interactive rebase rewrites history. Reviewing the plan first catches mistakes before they happen. A wrong squash can be painful to undo.

**Create a backup branch before rebasing:**

```
Before rebasing, create a backup branch called
`feat/settings-backup` pointing at the current HEAD.
That way we can recover if something goes wrong.
Then proceed with the interactive rebase.
```

This is cheap insurance. If the rebase goes sideways, you can `git reset --hard feat/settings-backup` and start over.

**Use `--autosquash` with fixup commits during development:**

Teach students to commit with `git commit --fixup=<hash>` during development, then ask Claude Code to clean up at the end:

```
I've been using --fixup commits during development. Run
`git rebase -i --autosquash main` to fold them into their
target commits. Show me the result before force-pushing.
```

### Things to Avoid

**Rebasing commits that are already on a shared branch:**

```
Rebase the last 5 commits on the main branch.
```

Never rebase commits that other people have already pulled. This rewrites shared history and causes merge hell for everyone. Rebase is for cleaning up _your_ feature branch before it's merged. Once it's on main, history is sacred.

**Force-pushing without checking if others are working on the branch:**

```
Force-push the rebased branch.
```

If another developer has pushed commits to your branch (pair programming, review fixes), force-pushing will erase their work. Always check first:

```
Before force-pushing, check if there are any commits on
origin/feat/settings that aren't in my local branch. If
someone else has pushed to this branch, warn me instead
of force-pushing.
```

**Rebasing a branch with 50+ commits in one go:**

If your branch has diverged significantly from main, rebasing everything at once can produce dozens of conflicts. Better to rebase in chunks or consider a merge instead:

```
This branch has 60 commits and main has changed a lot.
Should we rebase or merge? Check how many files would
conflict and recommend the safer approach.
```

---

## Lesson 4.2 — Conflict Resolution with Claude Code

Merge conflicts are one of the most stressful parts of Git. They require understanding two different changes, figuring out which should "win" (or how to combine them), and doing it correctly across potentially dozens of files. Claude Code excels here because it can read both sides of a conflict, understand the _intent_ of each change, and produce a correct resolution.

### Example 1: Resolving Conflicts During a Rebase

You're rebasing your feature branch onto the latest main and hit conflicts:

```
I'm rebasing feat/user-settings onto main and hit merge
conflicts. For each conflicted file:

1. Show me the conflict (both sides)
2. Read the git log to understand what each side intended
3. Propose a resolution that preserves both changes correctly
4. Explain your reasoning

Don't auto-resolve — show me each resolution and let me
approve before you mark it as resolved.
```

Claude Code might respond for one file:

> **Conflict in src/models/user.ts:**
>
> **Their side (main):** Added a `lastLoginAt: Date` field and updated the
> `UserSchema` validation to include it. This was part of PR #238 (login
> tracking feature, merged 2 days ago).
>
> **Our side (feat/user-settings):** Added `notificationPrefs: NotificationSettings`
> field and updated the `UserSchema` to include it.
>
> **Resolution:** Both sides added a new field to the same model. The changes
> don't conflict semantically — we need both fields. I'll combine them:
> keep `lastLoginAt` from main AND `notificationPrefs` from our branch.
> The schema validation needs both new fields added.
>
> Want me to apply this resolution?

You approve, and Claude Code resolves the conflict and continues the rebase.

### Example 2: Complex Conflict Where Both Sides Refactored the Same Code

```
We have a conflict in src/services/orderService.ts. Both branches
refactored the `calculateTotal` function — main changed it to
support multiple currencies, and my branch changed it to apply
discount codes.

Read both versions carefully. We need the final version to
support BOTH multiple currencies AND discount codes. This isn't
a simple "take both sides" — the function logic needs to be
genuinely merged. Propose the combined implementation.
```

This is where Claude Code shines over manual conflict resolution. A human staring at `<<<<<<<` markers would struggle to mentally combine two algorithmic changes. Claude Code can read both implementations, understand the logic, and write a new version that incorporates both.

### Example 3: Batch Conflict Resolution

When rebasing or merging produces conflicts in many files, some are trivial (both sides added imports, both sides modified a config file) while others need careful attention:

```
We have merge conflicts in 14 files after rebasing onto main.
Triage them for me:

- TRIVIAL: Both sides added non-conflicting content (imports,
  config entries, test cases). Auto-resolve these by keeping both.
- NEEDS REVIEW: The changes overlap semantically and need
  a human decision.

Auto-resolve all the trivial ones and show me only the ones
that need my review, with your proposed resolution for each.
```

This saves enormous time. Instead of manually resolving 14 conflicts, you only need to review the 2-3 that actually require judgment.

### Example 4: Understanding What Changed on Main While You Were Working

Before even attempting to rebase, get a briefing:

```
I'm about to rebase feat/user-settings onto main. Before we
start, analyze the risk:

1. How many commits are on main since we branched off?
2. Which files were changed on main that we also changed?
3. For each overlapping file, are the changes in the same
   functions/sections or different parts of the file?
4. Rate the likely conflict severity: none, trivial, moderate,
   or complex.

Based on this analysis, should we rebase or merge?
```

This pre-flight check tells you whether you're about to have a smooth rebase or a painful one, and lets you prepare accordingly.

### Good Practices

**Let Claude Code explain the intent behind each side of a conflict:**

```
For each conflict, check the git log and PR descriptions to
understand WHY each change was made, not just WHAT changed.
The "why" should drive the resolution.
```

Two changes that look contradictory at the code level might be perfectly compatible at the intent level — or genuinely incompatible in ways that the code alone doesn't make obvious.

**Test after resolving conflicts:**

```
After resolving all conflicts, run `npm run test` and
`npm run build`. Conflict resolution is error-prone —
verify that the merged code actually works.
```

A "successfully resolved" conflict can still produce broken code if the resolution was wrong. Always verify.

**Resolve conflicts in small batches, not all at once:**

If you have 20 conflicted files, resolve them in groups and verify between groups:

```
Let's resolve conflicts in batches. Start with the model
files (src/models/), resolve those, and run the model tests.
Then move to services, then routes. Test between each batch.
```

### Things to Avoid

**Blindly accepting one side:**

```
Just take our version for all conflicts.
```

"Accept ours" or "accept theirs" across the board almost always loses important changes. Even when one side is clearly "more correct," the other side often has small changes (new imports, updated types, added fields) that need to be preserved.

**Resolving conflicts without understanding the codebase:**

```
Fix the merge conflicts.
```

Without guidance about which changes are more important or what the intended behavior is, Claude Code has to guess. Provide context:

```
Fix the merge conflicts. For context: our branch (feat/user-settings)
is the priority — it's the feature we're shipping. Main's changes
are infrastructure updates that should be preserved, but if there's
a genuine conflict in business logic, our branch's intent wins.
```

**Forgetting to `git add` resolved files and continue the rebase:**

After Claude Code resolves each file, make sure it marks the resolution:

```
After resolving each conflicted file, run `git add` on it.
Once all conflicts in this commit are resolved, run
`git rebase --continue`. If new conflicts appear in the
next commit, repeat the process.
```

---

## Lesson 4.3 — Code Reviews with Claude Code

Code review is where bugs are caught, knowledge is shared, and code quality is maintained. Claude Code can act as a thorough first-pass reviewer — catching issues that human reviewers often miss because they're tedious to check manually (naming consistency, error handling completeness, test coverage gaps). This doesn't replace human review but makes it significantly more effective.

### Example 1: Pre-Submission Self-Review

Before requesting human review, use Claude Code to catch issues you'd be embarrassed about:

```
I'm about to submit PR #254 for review. Before I do, review
my own changes as if you were a strict senior engineer.
Check for:

1. Code quality: naming, readability, DRY violations, dead code
2. Error handling: are all error paths covered? any swallowed
   errors or missing try/catch?
3. Security: SQL injection, XSS, exposed secrets, auth checks
4. Types: any `any` types, unsafe casts, or missing null checks?
5. Tests: are there tests for the new code? are edge cases
   covered? are there tests for error paths, not just happy paths?
6. Consistency: does the new code follow the patterns in the
   rest of the codebase?

Be harsh. I'd rather fix issues now than get them in review.
```

Claude Code will diff the branch against main and produce findings like:

> **Issues found:**
>
> **P0 — Must fix:**
>
> - `src/services/exportService.ts:47` — The `catch` block logs the error but
>   doesn't re-throw or return an error response. The caller gets an undefined
>   result and will crash later with a confusing error.
> - `src/routes/export.ts:23` — Missing `requireAuth` middleware on the new
>   endpoint. Any unauthenticated user can trigger exports.
>
> **P1 — Should fix:**
>
> - `src/services/exportService.ts:31` — The `query` parameter is interpolated
>   directly into the SQL string. Use a parameterized query to prevent SQL injection.
> - `tests/export.test.ts` — No tests for the error paths (invalid report ID,
>   database connection failure, user without export permission).
>
> **P2 — Consider fixing:**
>
> - `src/services/exportService.ts:15` — The function is 85 lines long.
>   Consider extracting the CSV formatting into a separate `formatAsCsv` helper.
> - `src/types/export.ts:8` — `options: any` should be typed. There's already
>   an `ExportOptions` interface in the same file that isn't being used.

### Example 2: Reviewing a Teammate's PR

```
Read the diff for PR #251 on GitHub. This is a PR from a
teammate — I'm the reviewer. Analyze it and help me write
a thorough review.

For each issue you find, classify it:
- 🔴 BLOCKER: Must fix before merge (bugs, security, data loss)
- 🟡 SUGGESTION: Should fix but not blocking (code quality,
  naming, patterns)
- 🟢 NIT: Minor style/preference (take it or leave it)
- 💬 QUESTION: Something I want the author to explain

Also note what's GOOD about the PR — what's well-designed,
well-tested, or clever. Reviews shouldn't be all negative.

Format each comment with the file, line number, and a clear
explanation I can paste into the GitHub review.
```

Claude Code produces review comments you can use directly:

> **🟢 Overall: Well-structured PR. Good separation of concerns between the
> queue producer and consumer. Tests are thorough.**
>
> ---
>
> 🔴 **src/queue/emailConsumer.ts:34**
> `await emailService.send(job.data)` is called without a try/catch.
> If sending fails, the job will be lost (not retried). Wrap in try/catch
> and either retry or move to a dead letter queue.
>
> 🟡 **src/queue/emailProducer.ts:12**
> The queue name `"emails"` is hardcoded in both the producer and consumer.
> Extract to a constant in `src/config/queues.ts` so they can't drift.
>
> 🟢 **tests/queue/emailConsumer.test.ts:45-60**
> Nice — testing both the success path and the "malformed job data" path.
> This is exactly the kind of coverage that prevents production incidents.
>
> 💬 **src/queue/emailConsumer.ts:52**
> The consumer processes jobs one at a time. Was this intentional, or should
> we set a concurrency limit (e.g., 5) for throughput? What's the expected
> volume?

### Example 3: Checking for Regressions in a Large PR

```
PR #260 changes 42 files across the auth module. I'm worried
about regressions. Do the following:

1. Read the full diff.
2. For each changed file, check if the existing tests still
   cover the modified behavior. Flag any file where behavior
   changed but tests weren't updated.
3. Look for any deleted or modified assertions in test files
   — these are especially suspicious (tests might have been
   weakened to make the PR pass).
4. Check if any public function signatures changed — these
   could break callers in other parts of the codebase. Search
   for all call sites.
```

This is the kind of thorough regression check that human reviewers want to do but rarely have time for. Claude Code can systematically check every file.

### Example 4: Architecture-Level Review

```
This PR introduces a new caching layer. Before I look at the
code details, review the architectural approach:

1. Read all the new and modified files in the PR.
2. Draw me a picture of the data flow: where does data come
   from, where is it cached, and how is cache invalidation
   triggered?
3. Does this introduce any new single points of failure?
4. What happens if the cache goes down — does the app degrade
   gracefully or crash?
5. Is this consistent with how we handle caching elsewhere in
   the codebase (check src/cache/ for existing patterns)?
```

This high-level review catches design issues that line-by-line review misses entirely.

### Example 5: Reviewing for Consistency with Project Standards

```
Review PR #258 against our CLAUDE.md coding standards. Check:
- Are repository methods used for data access (no raw SQL in services)?
- Does error handling follow our ErrorResponse pattern?
- Are all new endpoints behind the correct auth middleware?
- Do new types use our naming convention (PascalCase with module prefix)?
- Are test files co-located in the right directory?

Flag any violations with the specific rule and the offending line.
```

This uses your project's documented standards (from CLAUDE.md or a contributing guide) as a review checklist — something that's tedious for humans but trivial for Claude Code.

### Example 6: Generating Review Comments in Batch

After Claude Code identifies issues, have it format them for direct submission:

```
Based on your review, generate GitHub review comments I can
submit. For each comment:
- Specify the exact file and line range
- Write the comment as if I'm the reviewer (first person)
- Keep the tone constructive and specific
- For blockers, suggest a concrete fix, not just "this is wrong"
- Group related comments together so the author sees the pattern

Draft the review summary (the top-level comment) too. Start
with what's good about the PR, then list the key issues.
```

### Good Practices

**Use Claude Code as a first-pass reviewer, not a replacement for human review:**

```
Do a first-pass review of PR #251. I'll use your findings to
focus my own review on the areas that need the most attention.
Flag anything you're uncertain about so I give it extra scrutiny.
```

Claude Code catches mechanical issues (missing error handling, type safety, test gaps) so humans can focus on design, business logic, and nuanced trade-offs.

**Review in layers — architecture first, then details:**

```
First, give me the architectural summary: what does this PR
change at a high level? What design decisions did the author make?

Then, only after I've understood the big picture, go through
the line-by-line details.
```

Starting with details and losing the forest for the trees is the most common review mistake. Force the architecture-first approach.

**Ask Claude Code to check what the PR does NOT do:**

```
This PR adds a new API endpoint. Check if the author
remembered to:
- Add the endpoint to the API docs (docs/api.md)
- Add the route to the rate limiting config
- Add the endpoint to the integration test suite
- Update the OpenAPI spec if we have one
- Add the permission to the RBAC config

These are easy to forget and often missed in review.
```

The "checklist of things people forget" is one of the highest-value review techniques, and Claude Code can check them systematically.

### Things to Avoid

**Submitting Claude Code's review comments without reading them:**

```
Review this PR and post all your comments directly to GitHub.
```

Claude Code might misunderstand a deliberate design choice, flag something that's intentional, or miss context that a human would catch. Always read and edit the review comments before submitting. You're putting your name on them.

**Using Claude Code to rubber-stamp PRs:**

```
This PR looks fine to me. Generate an approval comment.
```

If you haven't actually reviewed the PR, an AI-generated "LGTM" is worse than no review at all. It gives false confidence that someone checked the code.

**Being unnecessarily harsh in AI-assisted reviews:**

Because Claude Code can find many issues quickly, it's tempting to dump 30 comments on a PR. This demoralizes the author and makes review feel adversarial. Curate the feedback:

```
You found 25 issues. Prioritize them:
- Which 3-5 are the most important (blockers or significant
  quality issues)?
- Which can be grouped into a single "pattern" comment
  (e.g., "missing null checks in 8 places — here's the pattern
  to apply everywhere")?
- Which are so minor they're not worth commenting on?

I want a review that's thorough but respectful of the author's time.
```

---

## Module 4 Assessment

### Project: The Messy Branch Challenge

Students receive a Git repository with a deliberately messy feature branch:

- 20+ commits with poor messages, WIP states, and accidental commits
- The branch is 30 commits behind main
- Rebasing onto main produces conflicts in 8 files
- Two of the conflicts involve semantic overlap (both sides changed the same logic)

**Part A: History Cleanup (graded)**

Clean the branch history using interactive rebase:

- Squash related commits into logical groups
- Reword all commit messages to follow Conventional Commits
- Split any commit that touches unrelated areas
- Result should be 4-6 clean, focused commits that tell a coherent story

Students submit the before/after commit log and explain their grouping decisions.

**Part B: Conflict Resolution (graded)**

Rebase the cleaned branch onto main:

- Triage conflicts into trivial vs. needs-review
- Auto-resolve trivial conflicts
- Resolve complex conflicts with explanations for each decision
- Run tests after resolution to verify correctness

Students submit their resolution strategy and test results.

**Part C: Self-Review (graded)**

Conduct a thorough self-review of the final PR:

- Run a full code quality, security, and test coverage review
- Generate prioritized findings with severity levels
- Fix all P0 and P1 issues
- Write the PR description and review summary

Students submit the review findings, fixes applied, and final PR.

**Grading criteria:**

- Clean, logical commit history (would a reviewer understand the story?)
- Conflict resolution correctness (both sides' intent preserved)
- Thoroughness of self-review (did they catch the planted issues?)
- Quality of review comments (constructive, specific, prioritized)
- Final test suite passes with no regressions
