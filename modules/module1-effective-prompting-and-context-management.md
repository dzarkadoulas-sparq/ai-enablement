# Module 1 — Effective Prompting & Context Management

## Detailed Lesson Plans with Examples

---

## Lesson 1.1 — The Anatomy of a High-Quality Prompt

The difference between a vague prompt and a precise one is the difference between "fix this code" and telling Claude Code exactly what's wrong, what the expected behavior is, and what constraints to respect. This lesson covers the building blocks: specifying intent clearly, providing constraints (language, framework, style), indicating scope (which files, which functions), and defining success criteria.

### Good Practices

**Be specific about what you want changed and why:**

```
The `calculateTotal` function in src/utils/pricing.ts is returning
incorrect values when a discount code is applied after tax.
Expected behavior: discount should be applied BEFORE tax calculation.
Fix the function, update the existing tests in
tests/pricing.test.ts to cover this case, and don't change
the function signature.
```

This works well because it names the exact file and function, describes the current (broken) behavior, states the expected behavior, defines scope (what to change, what not to change), and mentions tests.

**State success criteria explicitly:**

```
Add pagination to the /api/users endpoint. Requirements:
- Accept `page` and `limit` query params (default page=1, limit=20)
- Return a JSON envelope with `data`, `total`, `page`, `totalPages`
- Return 400 if page < 1 or limit > 100
- Add tests for all three cases
```

This gives Claude Code a checklist it can verify against.

**Provide format and style constraints when they matter:**

```
Write a migration script to add a `last_login` column to the users
table. Use our existing migration format (see db/migrations/ for
examples). Use Knex syntax, not raw SQL. Column should be a
nullable timestamp.
```

Referencing existing patterns in the codebase is one of the most effective ways to get consistent output.

### Things to Avoid

**Vague, underspecified prompts:**

```
fix the pricing bug
```

Claude Code doesn't know which bug, which file, what the correct behavior is, or what constraints to follow. It will guess — and often guess wrong, wasting a round of iteration.

**Combining too many unrelated tasks in one prompt:**

```
Fix the login bug, also add dark mode to the dashboard, refactor
the database layer to use connection pooling, and update the README.
```

This overloads the task. Claude Code may do a mediocre job on all four or get confused about dependencies between them. Break these into separate, focused prompts.

**Being specific about the solution when you should describe the problem:**

```
Change line 42 of server.js from `res.send(data)` to
`res.json(data)` and add `Content-Type: application/json` header.
```

If you already know the exact fix, just make the edit yourself. Claude Code is most valuable when you describe the _problem_ and let it figure out the _solution_, because it may find a better one you didn't think of — or catch related issues.

---

## Lesson 1.2 — Scoping Context: Files, Directories & Code Ranges

Claude Code can see your project, but it works best when you guide its attention. This lesson covers how to reference specific files and directories, how to narrow focus, and when to let Claude Code explore broadly versus constraining it.

### Good Practices

**Point to specific files when you know where the issue lives:**

```
Look at src/auth/middleware.ts and src/auth/tokenService.ts.
The JWT refresh logic is failing silently when the refresh token
is expired — it should return a 401 instead. Fix the error
handling in both files.
```

This prevents Claude Code from wandering through unrelated parts of the codebase.

**Tell Claude Code to explore when you genuinely don't know where the issue is:**

```
Users are reporting that the search feature returns stale results
after updating a product. I'm not sure where the caching layer is.
Find where search results are cached, then figure out why cache
invalidation isn't happening on product updates.
```

This is a legitimate time to let Claude Code explore — you're asking it to do the investigation. The prompt is still specific about the _symptom_, just not the _location_.

**Use directory scoping for focused refactors:**

```
In the src/components/forms/ directory, all form components are
using class components. Convert them to functional components
with hooks. Don't touch anything outside that directory.
```

The boundary is clear. Claude Code knows exactly what to change and what to leave alone.

### Things to Avoid

**Giving no context and expecting Claude Code to figure everything out:**

```
the app is slow, make it faster
```

"The app" is the entire codebase. "Slow" could be a frontend rendering issue, a database query, a network call, a memory leak. Claude Code will either pick something arbitrary to optimize or spend many tool calls reading the entire project.

**Over-constraining when you're wrong about the scope:**

```
The bug is definitely in src/api/handlers.ts, fix it there.
```

If the bug is actually in the middleware or in a shared utility, you've told Claude Code to look in the wrong place. It will either make a wrong "fix" in the file you specified, or have to push back. If you're not 100% sure, say: "I think the bug might be in src/api/handlers.ts but I'm not certain — investigate and fix wherever the root cause is."

**Referencing files that don't exist or using wrong paths:**

```
Check the utils/helpers/stringUtils.js file and fix the
capitalize function.
```

If the actual path is `src/lib/string-utils.ts`, Claude Code will waste time looking for a nonexistent file and then have to search. Use the actual path — check it in your editor first if needed.

---

## Lesson 1.3 — CLAUDE.md: Persistent Project Intelligence

CLAUDE.md files shape Claude Code's behavior persistently. This lesson covers the three levels — project, user, and workspace — and how to write effective rules.

### Good Practices

**Write concrete, actionable rules:**

```markdown
# CLAUDE.md

## Code Style

- Use `zod` for all input validation. Never use `joi` or manual checks.
- All API responses must use the `ApiResponse<T>` wrapper from src/types/api.ts.
- Prefer `const` over `let`. Never use `var`.
- Error messages must be user-facing friendly. No stack traces in API responses.

## Testing

- Run tests with: `npm run test`
- Every new API endpoint must have integration tests in tests/integration/.
- Use `factories/` for test data, never hardcode IDs or emails.

## Architecture

- Database access only through repository classes in src/repositories/.
- Never import from src/repositories/ in route handlers directly — use services.
- Background jobs go in src/jobs/ and must be idempotent.

## Common Pitfalls

- The `users` table has a soft-delete (`deleted_at`). Always filter for
  `deleted_at IS NULL` unless explicitly querying deleted users.
- The Redis connection requires TLS in production. Don't change the
  connection config without checking env-specific settings.
```

Every rule here is specific enough that Claude Code can follow it without interpretation. The "Common Pitfalls" section is especially valuable — it encodes tribal knowledge that Claude Code would have no way to discover on its own.

**Layer your CLAUDE.md files appropriately:**

Project-level `CLAUDE.md` (committed to repo, shared with team):

```markdown
## Build & Test

- Build: `npm run build`
- Test: `npm run test`
- Lint: `npm run lint`

## Architecture

- This is a monorepo managed with Turborepo.
- Packages: packages/api, packages/web, packages/shared.
- Shared types go in packages/shared/types/.
```

User-level `~/.claude/CLAUDE.md` (personal preferences):

```markdown
- I prefer verbose commit messages with a "Why" section.
- When explaining code changes, include before/after comparisons.
- I use VS Code — reference keyboard shortcuts accordingly.
```

Workspace-level `.claude/CLAUDE.md` (feature-branch specific):

```markdown
- Currently working on the Stripe integration (branch: feat/stripe).
- Stripe webhook handler is in src/webhooks/stripe.ts.
- Test Stripe key is in .env.test, do not use production keys.
```

Each level handles different concerns — shared project knowledge, personal workflow, and temporary context.

**Evolve CLAUDE.md as you discover repeated corrections:**

If you find yourself telling Claude Code the same thing three times across sessions ("don't use `any` type", "always destructure props", "use our custom logger, not console.log"), that's a signal to add it to CLAUDE.md. The file should grow organically from real friction.

### Things to Avoid

**Vague, aspirational guidelines:**

```markdown
# CLAUDE.md

- Write clean code
- Follow best practices
- Make sure everything works
- Keep things simple
```

None of these are actionable. "Clean code" and "best practices" mean different things in every project. Claude Code can't follow a rule it can't evaluate concretely.

**Contradictory rules:**

```markdown
- Always use functional components with hooks.
- Follow the existing patterns in the codebase.
```

If the codebase is full of class components, these two rules conflict. Claude Code will have to pick one, and it might pick the wrong one. Be explicit about which takes priority: "Convert to functional components when modifying existing class components, but don't refactor files you're not otherwise touching."

**Stuffing the entire project documentation into CLAUDE.md:**

```markdown
# CLAUDE.md

[500 lines of API documentation, database schema, deployment
procedures, onboarding guide, team contact info, meeting notes...]
```

CLAUDE.md is read at the start of every session. If it's enormous, the important rules get diluted by noise. Keep it focused on things that directly affect how Claude Code writes and modifies code. Link to external docs instead: "Full API docs: see docs/api.md."

---

## Lesson 1.4 — Prompt Patterns for Common Workflows

This lesson catalogs reusable prompt structures for recurring situations.

### Pattern 1: Explore-then-Act

**Good practice:**

```
First, read through src/services/orderService.ts and
src/repositories/orderRepository.ts to understand how order
creation works — particularly how inventory is checked and
decremented.

Once you understand the flow, add a check that prevents
creating an order if any line item's quantity exceeds available
stock. Throw an InsufficientStockError with the item ID and
requested vs available quantities.
```

Separating the "understand" phase from the "act" phase lets Claude Code build a mental model before making changes, which dramatically reduces errors in complex code.

**Thing to avoid:**

```
Add stock validation to order creation.
```

Without the exploration step, Claude Code may not discover that inventory lives in a separate repository, that stock is tracked with eventual consistency, or that there's already a partial check somewhere else. It will write something that looks correct but misses the project's actual patterns.

### Pattern 2: The Constraint Sandwich

**Good practice:**

```
Refactor the email notification system to use a queue instead
of sending synchronously.

Do NOT change the EmailTemplate interface or the existing
template files. Do NOT modify how templates are rendered.
The queue consumer should call the same `sendEmail()` function
that's used today.

The end result should be: API endpoints return faster because
email sending is async, but users receive the exact same emails
they do today.
```

The structure is: what you want → what you don't want → restate the goal. This "sandwich" prevents Claude Code from making sweeping changes that technically accomplish the goal but break other things.

**Thing to avoid:**

```
Refactor emails to be async. Use a queue. Make it fast.
```

No constraints on what to preserve. Claude Code might rewrite the templates, change the email API, or restructure things in a way that requires changes across dozens of files.

### Pattern 3: Test-First

**Good practice:**

```
I want to add rate limiting to the /api/login endpoint.

Step 1: Write failing tests in tests/integration/auth.test.ts:
  - Test that 5 requests in 60 seconds succeed
  - Test that the 6th request returns 429
  - Test that the counter resets after 60 seconds

Step 2: Implement the rate limiting to make the tests pass.
Use our Redis instance for the counter. The limit config
should come from src/config.ts.
```

This forces a TDD workflow. Claude Code writes tests that define the behavior contract, then writes code to satisfy them.

**Thing to avoid:**

```
Add rate limiting and some tests for it.
```

When tests are an afterthought, they tend to test the implementation rather than the behavior — and they often just confirm that whatever Claude Code wrote works, even if it's wrong.

### Pattern 4: Explain-Before-Changing

**Good practice:**

```
I want to replace Moment.js with date-fns across the project.
Before making any changes, give me a plan:
- Which files import Moment?
- What Moment functions are used and what are the date-fns equivalents?
- Are there any tricky cases (timezones, locales, relative time)?
- What's your proposed order of changes?

Wait for my approval before editing any files.
```

This gives you a review checkpoint. You can catch misunderstandings before Claude Code makes 30 file changes that you'll need to untangle.

**Thing to avoid:**

```
Replace Moment.js with date-fns everywhere.
```

Claude Code will start editing immediately. If it misunderstands a timezone conversion or misses a locale dependency, you'll discover the problem after it has already modified dozens of files.

### Pattern 5: Incremental

**Good practice:**

```
Let's add authentication to the app. We'll do this in steps.

Step 1 (this prompt): Set up the User model and migration
with email, hashed password, and timestamps. Use bcrypt for
hashing. Add the model to src/models/ and the migration to
db/migrations/. That's all for now — don't build login
endpoints or middleware yet.
```

Then in the next prompt:

```
Step 2: Now add a POST /api/auth/register endpoint that
creates a user using the model from Step 1. Validate input
with zod. Return 201 with the user (without password) on
success, 409 if email exists.
```

Each step is small, testable, and verifiable before moving on.

**Thing to avoid:**

```
Add full authentication to the app — registration, login,
JWT tokens, refresh tokens, password reset, email verification,
middleware for protected routes, and role-based access control.
```

This is an enormous scope in a single prompt. Claude Code will either produce a massive, hard-to-review changeset, or get confused partway through and produce inconsistent code across the features.

---

## Lesson 1.5 — Managing Long Sessions & Context Windows

Claude Code sessions can get long. Context quality degrades when the conversation becomes bloated. This lesson covers strategies for maintaining quality over extended work sessions.

### Good Practices

**Write checkpoint summaries before starting a new session:**

```
We've completed the following in this session:
1. Added the `orders` table migration (db/migrations/20240115_orders.ts)
2. Created OrderRepository with create, findById, findByUser methods
3. Added integration tests for all three repository methods — all passing
4. Created OrderService with createOrder (includes stock validation)

Still TODO:
- Order API endpoints (POST /orders, GET /orders/:id, GET /users/:id/orders)
- Order service tests (unit tests, mocking the repository)
- Update the API docs in docs/api.md

Key decisions made:
- Orders use soft-delete (deleted_at column) matching users pattern
- Stock validation throws InsufficientStockError (src/errors/)
- Order status enum: pending, confirmed, shipped, delivered, cancelled
```

When you start a new session, paste this summary so Claude Code has full context without needing the entire previous conversation.

**Use TODO files as external memory:**

```
Create a TODO.md file tracking our progress on the Stripe
integration. Include what's done, what's in progress, and
what's remaining. Update it at the end of each session.
```

This lives in the filesystem and persists between sessions. Claude Code can read it at the start of the next session.

**Recognize when context is getting noisy and start fresh:**

Signs it's time for a new session:

- Claude Code starts repeating mistakes you already corrected
- It references a file's old content (before your latest edits)
- It forgets constraints you stated earlier in the conversation
- Responses get slower and less focused

When you see these signs:

```
Let's pause here. Summarize everything we've done and what's
left, then I'll start a fresh session.
```

### Things to Avoid

**Running a single session for hours without checkpoints:**

If you've been going for 30+ prompts in one session, the early context is getting compressed or lost. Claude Code may forget your CLAUDE.md rules, earlier decisions, or even which files it already modified.

**Starting a new session with no context at all:**

```
(new session)
Continue working on the project.
```

Claude Code has no memory of the previous session. It doesn't know what you did, what decisions were made, or what's left. Always provide a summary or point to a TODO file.

**Expecting Claude Code to remember everything from a very long conversation:**

```
(prompt #45 in the same session)
Use the same approach we discussed for the user service.
```

"The approach we discussed" was 40 prompts ago. Be explicit: "Use the repository pattern we used for OrderRepository — separate data access from business logic, with the same error handling pattern."

---

## Lesson 1.6 — Debugging Bad Outputs: Prompt Forensics

When Claude Code produces something wrong, the instinct is to say "that's wrong, fix it." This lesson teaches a more effective approach: diagnosing why the output went wrong and adjusting the prompt.

### Good Practices

**Diagnose before correcting — tell Claude Code what's specifically wrong:**

```
The function you wrote works for single items, but it breaks
for batch operations. The issue is that you're calling
`db.save()` inside the loop, which means a failure on item 3
leaves items 1-2 committed. Wrap the entire loop in a
transaction instead.
```

This explains the _specific failure mode_, not just "it's wrong." Claude Code can now make a targeted fix rather than guessing what you didn't like.

**Reference Claude Code's reasoning when it went off track:**

```
I see you used `Array.reduce()` to build the summary object,
but the accumulator type is wrong — it's losing the optional
fields on each iteration. The root cause is the initial value
you passed to reduce. Use a typed initial value that matches
the SummaryReport interface, or switch to a simple for-loop
with a pre-typed variable.
```

Pointing to the specific reasoning step that went wrong helps Claude Code avoid the same mistake in the fix.

**Ask Claude Code to explain its approach when the output is surprising:**

```
You added a new database index on (user_id, created_at) but
I expected you to modify the existing composite index. Before
changing anything, explain why you chose to add a new index
instead of modifying the existing one.
```

Sometimes Claude Code made a deliberate choice that's actually correct. Sometimes it didn't realize the existing index existed. The explanation reveals which — and saves you from "fixing" something that wasn't broken.

### Things to Avoid

**Vague rejection with no diagnostic information:**

```
That's wrong. Try again.
```

Claude Code has no idea what's wrong. It will either make the same mistake with minor variations or make a completely different (possibly worse) attempt. Always say _what_ is wrong.

**Repeatedly saying "no, that's still wrong" without adding new information:**

```
Prompt 1: Fix the date parsing.
Response: [changes code]
Prompt 2: Still broken. Fix it.
Response: [changes code differently]
Prompt 3: Nope. Try again.
```

Each correction should add information. If the fix is still wrong, explain what's still failing, show the error, or provide a concrete example of input vs. expected output:

```
Still failing. Here's a concrete case:
Input: "2024-01-15T08:30:00+05:30"
Expected output: "2024-01-15 03:00:00 UTC"
Actual output: "2024-01-15 08:30:00 UTC" (timezone offset not applied)
```

**Blaming Claude Code instead of improving the prompt:**

```
Why can't you get this right? I already told you what to do.
```

If Claude Code keeps getting it wrong, the prompt is probably ambiguous or missing information. Re-read your original prompt as if you knew nothing about the codebase — would you be able to follow it? The fix is almost always a better prompt, not a harsher one.

---

## Module 1 Assessment

Students receive an unfamiliar codebase and a set of requirements. They must:

1. **Write a CLAUDE.md** for the project based on reading the code and identifying conventions, pitfalls, and architecture patterns.
2. **Plan a prompting strategy** — decide which prompt patterns to use for each requirement, in what order, and why.
3. **Execute across multiple sessions** with clean checkpoint summaries and handoff notes.
4. **Encounter and debug at least two "bad outputs"** — diagnose the root cause and refine the prompt.
5. **Present a retrospective** on what worked, what didn't, and how they'd approach it differently next time.

Grading criteria:

- Quality and specificity of CLAUDE.md rules
- Appropriateness of prompt pattern selection
- Clarity of session handoffs (could another student continue from the notes alone?)
- Quality of prompt forensics (did they identify root causes, not just symptoms?)
- Final output quality (does the code work, follow conventions, have tests?)
