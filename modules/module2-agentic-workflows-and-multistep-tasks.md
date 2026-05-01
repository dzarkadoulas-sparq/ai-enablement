# Module 2 — Agentic Workflows & Multi-Step Tasks

## Detailed Lesson Plans with Examples

---

## Lesson 2.1 — Understanding Claude Code's Agentic Loop

Claude Code doesn't just answer questions — it operates as an autonomous agent that can read files, write code, run commands, examine output, and iterate until the task is done. This lesson explains the agentic loop: how Claude Code decides what to do next, how it uses tools (file reads, writes, bash commands), and how it self-corrects based on results.

Understanding this loop is key to working effectively with Claude Code. When you give it a task, it will typically: analyze the request, explore relevant code, form a plan (implicitly or explicitly), make changes, verify them (by running tests, linters, or the app), and iterate if something fails.

### Good Practices

**Trust the loop for well-defined tasks:**

```
Add a `GET /api/orders/:id/invoice` endpoint that generates a PDF
invoice for the given order. Use the existing PDFKit setup in
src/utils/pdf.ts. The invoice should include order items, prices,
tax, and total. Return the PDF as a downloadable response.

Run the existing tests after you're done to make sure nothing is broken.
```

This is a well-scoped task with clear inputs and outputs. Claude Code can explore the existing PDF utility, look at the order model, write the endpoint, and verify it compiles and passes tests — all without needing your input at each step. Let it work.

**Give Claude Code a way to verify its own work:**

```
Refactor the logger module from CommonJS to ESM. After making
the changes, run `npm run build` and `npm run test` to make
sure everything still compiles and passes. If something breaks,
fix it before moving on.
```

The "run and fix" instruction turns Claude Code into a self-correcting loop. It makes a change, checks the result, and fixes issues — often across several iterations — without you needing to intervene.

**Watch the tool calls to understand Claude Code's approach:**

When Claude Code is working, pay attention to which files it reads, what commands it runs, and in what order. This tells you:

- Whether it's exploring the right area of the codebase
- Whether it understood your request correctly
- Whether you should intervene early or let it continue

If you see it reading files that are completely unrelated to your task, that's a signal to interrupt and redirect rather than waiting for a wrong result.

### Things to Avoid

**Micromanaging every step:**

```
First read src/models/order.ts. What do you see?
```

_(waits for response)_

```
OK now read src/routes/orders.ts.
```

_(waits for response)_

```
Now create a new file at src/routes/invoices.ts.
```

_(waits for response)_

```
Now add a GET route handler...
```

You're manually driving what should be an autonomous workflow. Claude Code can figure out which files to read and in what order. Give it the destination, not turn-by-turn directions.

**Giving a task with no way to verify success:**

```
Optimize the database queries for better performance.
```

Claude Code can make changes, but it has no way to verify if performance actually improved. It can't run benchmarks it doesn't know about. Provide a verification step: "Run `npm run benchmark` after your changes and compare the output to the baseline in docs/benchmark-baseline.txt."

**Interrupting too early when Claude Code is on the right track:**

If Claude Code is reading files to understand context before making changes, don't interrupt with "just make the change already." The exploration phase is often essential — cutting it short leads to changes that miss edge cases or violate existing patterns. Let it finish reading, then see what it produces.

---

## Lesson 2.2 — The `/plan` Command: Think Before You Act

The `/plan` command is one of the most valuable features for complex tasks. When you prefix a task with `/plan`, Claude Code enters an extended thinking mode — it analyzes the request, explores the codebase, and produces a detailed plan of what it _would_ do, without actually making any changes. This gives you a review checkpoint before any code is touched.

### When to Use `/plan`

Use `/plan` when:

- The task touches multiple files or modules
- You're working in an area of the codebase you don't fully understand
- The wrong approach could create a mess that's hard to undo
- You want to validate Claude Code's understanding before it acts
- You're mentoring junior developers and want to discuss the approach first

### Good Practices

**Use `/plan` for complex, multi-file changes:**

```
/plan Migrate our authentication from session-based to JWT.
We're using Express with Passport.js. Sessions are stored in
Redis. We want to keep the Redis for refresh token blacklisting
but eliminate session storage. The app has 15 protected routes.
```

Claude Code will produce something like:

> **Plan: Migrate from session-based auth to JWT**
>
> **Files to modify:**
>
> - `src/auth/passport.ts` — Replace session strategy with JWT strategy
> - `src/auth/middleware.ts` — Change `isAuthenticated` to verify JWT from Bearer header
> - `src/routes/auth.ts` — Update login to return JWT + refresh token pair, add /refresh endpoint
> - `src/config/redis.ts` — Repurpose from session store to refresh token blacklist
> - `src/config/express.ts` — Remove express-session middleware
> - All 15 route files — No changes needed (middleware handles it)
>
> **New files:**
>
> - `src/auth/jwt.ts` — Token generation, verification, refresh logic
> - `src/types/auth.ts` — JWT payload interface, token pair type
>
> **Migration approach:**
>
> 1. Build JWT infrastructure first (new files)
> 2. Update auth middleware (swap from session to JWT check)
> 3. Update login/logout routes
> 4. Remove session dependencies
> 5. Run full test suite
>
> **Risks:**
>
> - Existing integration tests rely on session cookies — they'll all need updating
> - WebSocket connections in src/ws/ also use session auth — needs separate handling

Now you can review this plan before a single file is changed. You might respond:

```
Good plan, but two adjustments:
1. Don't touch the WebSocket auth yet — we'll handle that separately.
2. Add a migration period where both session and JWT work, so we can
   deploy gradually. Plan for a feature flag in src/config.ts.

Now go ahead and implement with these adjustments.
```

**Use `/plan` to compare approaches:**

```
/plan I need to add real-time notifications. Explore two options:
1. WebSocket with Socket.io
2. Server-Sent Events (SSE)

For each, outline which files would change, what new dependencies
are needed, and how it integrates with our existing Express + Redis
setup. Recommend one with reasoning.
```

This turns Claude Code into an architecture advisor. You get two detailed implementation sketches and a recommendation — without committing to either.

**Use `/plan` to scope a large feature into phases:**

```
/plan We need to add a full audit logging system that tracks
all data mutations (create, update, delete) across every model,
stores who did it and when, and provides an admin API to query
the audit log.

Break this into implementable phases, where each phase is
self-contained and deployable. Identify dependencies between
phases.
```

Claude Code might produce:

> **Phase 1: Core audit infrastructure**
>
> - Create AuditLog model and migration
> - Build audit service with `logEvent(actor, action, entity, changes)` method
> - Add to orderService as proof of concept
>
> **Phase 2: Middleware-based automatic capture**
>
> - Build Express middleware that intercepts mutations
> - Apply to all routes without manual service calls
>
> **Phase 3: Admin query API**
>
> - Add `GET /api/admin/audit-logs` with filtering by actor, entity, action, date range
> - Pagination support
>
> **Phase 4: Retention and archival**
>
> - Add TTL-based cleanup job
> - Archive to cold storage for compliance

Each phase is now a manageable chunk that you can implement in separate sessions.

### Things to Avoid

**Using `/plan` for simple, well-defined tasks:**

```
/plan Add a `createdAt` field to the User model.
```

This is a one-file, one-minute change. Planning it wastes time. `/plan` is for tasks where you genuinely need to review the approach before committing. Simple tasks should just be executed directly.

**Ignoring the plan and saying "just do it":**

```
/plan Refactor the payment processing module.
```

_(Claude Code produces a detailed plan)_

```
Looks fine, just do it.
```

If you're not actually going to review the plan, don't use `/plan`. The value is in the review step — reading the plan, catching issues, and adjusting the approach. If you trust Claude Code to get it right without review, skip `/plan` and let it work directly.

**Treating the plan as final without critical review:**

The plan is Claude Code's best _guess_ at the right approach based on reading the code. It might miss things:

- Files it didn't know about
- Side effects in other services
- Business rules that aren't in the code
- Recent changes by other team members

Always review the plan with your domain knowledge. Push back on anything that doesn't match your understanding. The plan is a starting point for discussion, not a final spec.

---

## Lesson 2.3 — Breaking Large Tasks into Sub-Tasks

The single biggest predictor of success with Claude Code on complex tasks is how you decompose them. A monolithic task produces monolithic (often broken) output. A well-decomposed task produces a series of small, verified wins.

### Good Practices

**Decompose by functional boundary, not by file:**

```
We're adding a shopping cart feature. Let's break it into parts:

Part 1 (this prompt): Data layer only.
- Create the Cart and CartItem models with migration
- Create CartRepository with: create, addItem, removeItem,
  getByUserId, clear
- Write integration tests for the repository
- Don't build any API endpoints yet.
```

Then later:

```
Part 2: Service layer.
- Create CartService that uses CartRepository
- Business logic: max 50 items per cart, validate product exists
  and is in stock before adding, calculate totals with tax
- Write unit tests mocking the repository
```

Then:

```
Part 3: API layer.
- POST /api/cart/items, DELETE /api/cart/items/:id, GET /api/cart
- Use CartService, don't put business logic in handlers
- Integration tests for all endpoints
- Update the API docs
```

Each part has a clear boundary, is independently testable, and builds on the last.

**Verify each sub-task before moving on:**

```
Before we move to Part 2, let's verify Part 1:
- Run the migration: `npm run migrate`
- Run just the cart tests: `npm run test -- --grep "CartRepository"`
- Show me the model files so I can review the schema.
```

Don't build Part 3 on top of a broken Part 1. Catch problems early when they're cheap to fix.

**Use a tracking file for multi-part work:**

```
Create a file at docs/cart-implementation-plan.md with our full
plan. Mark Part 1 as ✅ complete. We'll update this as we go
so we always know where we are.
```

This serves as shared state between you and Claude Code, and survives across sessions.

### Things to Avoid

**Dumping the entire feature spec in one prompt:**

```
Build a complete shopping cart: models, migrations, repository,
service with validation, API endpoints for CRUD, guest cart
support, cart merging on login, discount code application,
shipping calculation, tax by region, saved-for-later functionality,
cart expiration after 30 days, admin endpoint to view abandoned
carts, and analytics events for add/remove/checkout. Also add
tests for everything and update the docs.
```

This is a week of work in one prompt. Claude Code will either produce a superficial implementation of everything or get lost halfway through. Even the best model can't hold all of this in context coherently.

**Decomposing by file instead of by function:**

```
Step 1: Create src/models/cart.ts
Step 2: Create src/models/cartItem.ts
Step 3: Create src/repositories/cartRepository.ts
Step 4: Create src/services/cartService.ts
Step 5: Create src/routes/cart.ts
```

This is a file checklist, not a functional decomposition. There's no testing between steps, no verification, and no logical grouping. Claude Code can create all five files — but they might not work together because they were never tested as a unit.

**Moving on when tests are failing:**

```
OK the tests have 3 failures but that's fine, we'll fix them
later. Let's start on the API endpoints.
```

Building on broken foundations compounds the problem. Those 3 test failures will interact with new code in unpredictable ways. Fix them now while the context is fresh and the scope is small.

---

## Lesson 2.4 — Steering Claude Code Mid-Task

Even with good prompts, you'll sometimes need to redirect Claude Code while it's working. Maybe it's taking an unexpected approach, hitting a dead end, or you realized the requirements need adjusting. Knowing when and how to intervene is a key skill.

### Good Practices

**Intervene early with specific redirections:**

If you see Claude Code heading down the wrong path (reading wrong files, installing an unexpected dependency, using a pattern you don't want):

```
Stop — I see you're installing `express-validator`. We use `zod`
for all validation in this project (check CLAUDE.md). Use
zod schemas and the `validateRequest` middleware in
src/middleware/validation.ts instead.
```

Early, specific corrections are cheap. Waiting until Claude Code has built an entire feature with the wrong library is expensive.

**Provide new information when requirements change mid-task:**

```
Update on the requirements: the product team just told me that
cart items also need a `notes` field where customers can add
special instructions (e.g., "gift wrap this"). It's an optional
string, max 500 chars.

Add this to the CartItem model, migration, and the add-to-cart
endpoint. Don't start over — just layer it into what you've
already built.
```

The key phrase is "don't start over." Claude Code can incrementally add to its existing work.

**Ask Claude Code to pause and explain when you're unsure:**

```
Before you continue — I see you've created a separate
`CartPricingService` alongside the `CartService`. I wasn't
expecting that separation. Explain why you split them before
making more changes.
```

Sometimes Claude Code makes architectural decisions that are actually smart. Sometimes they're wrong. Pausing to ask is better than letting it build further on a foundation you're not sure about.

### Things to Avoid

**Saying "start over" when a small correction would work:**

```
That's not what I wanted. Start from scratch.
```

If Claude Code got 80% right and 20% wrong, throwing everything away wastes the 80%. Be specific about what needs changing:

```
The endpoint logic is correct, but you put the validation in
the route handler instead of using middleware. Move the zod
schema to src/schemas/cart.ts and use validateRequest middleware.
Keep everything else as-is.
```

**Giving contradictory mid-task corrections:**

```
(initial prompt): Use REST endpoints for the cart API.
(mid-task): Actually let's make it GraphQL.
(later): Hmm, go back to REST but keep the GraphQL types.
```

Each pivot forces Claude Code to undo and redo work, losing context quality each time. If you're uncertain about a major decision, use `/plan` first to think it through. Save the mid-task corrections for genuine requirement changes, not indecision.

**Silently hoping Claude Code will figure out your unstated preference:**

If Claude Code uses callbacks and you prefer async/await, don't just think "I wish it used async/await" and then be frustrated when the next function also uses callbacks. Say it:

```
Use async/await throughout — no callbacks. This applies to
everything in this session.
```

Or better yet, add it to your CLAUDE.md so you never have to say it again.

---

## Lesson 2.5 — Multi-File Changes and Dependency Awareness

Real-world tasks rarely touch just one file. A typical feature might involve a model, a service, route handlers, tests, configuration, and documentation. Claude Code handles multi-file changes well — but only if you help it understand the dependencies between files.

### Good Practices

**Specify the dependency chain explicitly:**

```
Add a soft-delete feature to the Project model.

This will need changes in:
1. Model: Add `deletedAt` field to src/models/project.ts
2. Migration: New migration to add the column
3. Repository: Update all find methods to filter `deletedAt IS NULL`
   by default, add a `findIncludingDeleted` method
4. Service: Add `softDelete` and `restore` methods
5. Routes: Add DELETE (soft-delete) and POST /restore endpoints
6. Tests: Update existing tests (they shouldn't see soft-deleted
   records) and add new tests for delete/restore

Work through these in order. Run tests after completing the
repository layer and again after completing all changes.
```

The numbered dependency chain helps Claude Code work systematically rather than jumping between layers.

**Ask Claude Code to check for ripple effects:**

```
After making the changes, search the codebase for any other
places that query the projects table directly (not through the
repository). There might be raw queries in reporting or admin
scripts that need the `deleted_at IS NULL` filter added.
```

This catches the "hidden dependencies" that cause bugs in production — code that bypasses the normal abstraction layers.

**Reference existing patterns for consistency:**

```
Add the soft-delete feature to Projects following the exact same
pattern we use for Users. Check src/models/user.ts,
src/repositories/userRepository.ts, and the related tests to
see how it's implemented there. Match that approach.
```

This is far more effective than describing the pattern yourself. The reference code _is_ the specification.

### Things to Avoid

**Changing a shared utility without checking consumers:**

```
Refactor the `formatCurrency` function in src/utils/format.ts
to support multiple locales.
```

If 30 files import `formatCurrency`, changing its signature or behavior without checking all consumers will break things silently. Better:

```
Refactor `formatCurrency` to support multiple locales. Before
changing the function signature, find all files that import it
and make sure the changes are backward compatible — existing
callers with no locale argument should default to 'en-US'
and behave identically to today.
```

**Making changes in tests and source simultaneously without running tests in between:**

```
Update the validation logic and update the tests to match.
```

If you change both at the same time, the tests will pass — but they might pass for the wrong reasons. Better to either change the logic first and see which tests fail (confirming the tests were actually testing the right thing), or write the tests first and then implement.

**Assuming Claude Code knows about your deployment architecture:**

```
This needs to work in both the monolith and the microservices.
```

Claude Code can see your codebase, but it doesn't know about your deployment topology, which services share databases, or which modules get deployed independently. Be explicit:

```
This change needs to work in two contexts:
1. In the monolith (src/monolith/) where all services share a DB connection
2. In the orders microservice (services/orders/) which has its own DB
Both import from packages/shared/ — so any shared type changes
need to be backward compatible.
```

---

## Lesson 2.6 — Autonomous Recovery: Letting Claude Code Fix Its Own Mistakes

One of Claude Code's most powerful agentic capabilities is self-correction. When it runs a command and it fails, it can read the error, diagnose the cause, and fix it — often through several iterations. Learning to trust (and guide) this recovery loop is essential.

### Good Practices

**Give Claude Code permission to iterate:**

```
Implement the export-to-CSV feature for the admin dashboard.
After writing the code, run the tests. If any tests fail,
read the error output and fix the issues. Keep going until
all tests pass.
```

The "keep going until all tests pass" instruction activates Claude Code's full agentic loop. It will cycle through fix → test → read error → fix until it succeeds (or hits a genuinely hard problem).

**Provide fallback instructions for known tricky cases:**

```
Try using `sharp` for the image resizing. If the installation
fails (it sometimes has native dependency issues in this
environment), fall back to `jimp` instead.
```

This saves Claude Code from getting stuck on an environment issue. Instead of going back and forth trying to install a broken dependency, it has a clear fallback path.

**Let Claude Code troubleshoot by adding diagnostic output:**

```
The WebSocket connection is dropping intermittently and I don't
know why. Add detailed logging to the connection lifecycle in
src/ws/handler.ts — log on connect, disconnect, error, and
ping/pong. Then run the integration tests that exercise
WebSocket connections and analyze the logs to find the cause.
```

You're asking Claude Code to instrument, observe, and diagnose — a full debugging cycle. This often works better than trying to spot-debug a tricky issue yourself.

### Things to Avoid

**Giving up on the agentic loop too early:**

```
(Claude Code writes code, test fails)
User: That test is failing. What's wrong?
```

Let Claude Code finish its loop first. If it ran the test and saw the failure, it's probably already working on a fix. Interrupting to ask "what's wrong" breaks the flow. Wait for it to either fix the issue or tell you it's stuck.

**Leaving Claude Code in an infinite recovery loop with no escape:**

If Claude Code has tried three or four times to fix the same test failure and keeps failing, it's time to intervene with more information:

```
You've tried fixing this three times and the same assertion is
still failing. Here's additional context: the `createdAt`
timestamp uses the database server's timezone, not UTC. The
test is comparing against a UTC timestamp. You need to either
normalize both to UTC or mock the database clock in the test.
```

The agentic loop works great for mechanical errors (wrong import paths, type mismatches, missing semicolons). For logic errors that require domain knowledge, you need to provide that knowledge.

**Assuming Claude Code will always catch its own mistakes:**

Claude Code can verify things it can _observe_ — test failures, lint errors, compilation errors. It cannot verify things it can't observe:

- Business logic correctness (it doesn't know your business rules unless told)
- Performance characteristics (it can't run load tests without setup)
- Security vulnerabilities (it can catch obvious ones but not subtle authorization gaps)
- UX quality (it can't see the rendered UI)

For these, you still need human review. The agentic loop handles mechanical correctness, not holistic quality.

---

## Module 2 Assessment

### Project: Multi-Phase Feature Implementation

Students receive a medium-sized Express/Node.js application and a feature specification: **Add a team collaboration system** — teams, invitations, roles, and shared resources.

**Part A: Planning (graded)**
Use `/plan` to produce a phased implementation plan. The plan should identify all files to create or modify, dependencies between phases, and risks.

**Part B: Execution (graded)**
Implement at least 3 phases using the techniques from this module:

- Proper sub-task decomposition
- Verification between phases
- At least one mid-task correction when they encounter something unexpected
- A tracking document that records progress

**Part C: Recovery (graded)**
The instructor will deliberately introduce 2 bugs into the student's implementation between phases. Students must let Claude Code's agentic loop detect and fix them autonomously, intervening only when Claude Code gets stuck — and providing the minimum information needed to unstick it.

**Grading criteria:**

- Quality of the `/plan` output and student's review/adjustments
- Appropriate decomposition into sub-tasks (not too big, not too small)
- Use of verification checkpoints between phases
- Quality of mid-task steering (specific, constructive redirections)
- Recovery handling (did they let the loop work? did they intervene effectively when needed?)
- Final implementation quality (does the feature work end-to-end?)
