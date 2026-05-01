# Module 3 — MCP Server Integration

## Detailed Lesson Plans with Examples

---

## Lesson 3.1 — What is MCP and Why It Matters

The Model Context Protocol (MCP) is an open standard that lets Claude Code interact with external tools and services. Without MCP, Claude Code can only work with what's on your local filesystem and terminal. With MCP, it can browse websites, manage Jira tickets, commit to GitHub, send Slack messages, query databases, and much more — all from the same conversational interface.

Think of MCP servers as plugins that give Claude Code new abilities. Each server exposes a set of tools (functions Claude Code can call), and Claude Code decides when and how to use them based on your prompts.

### The Architecture

```
You → Claude Code → MCP Server → External Service
                                    (Jira, GitHub, Slack, Browser, etc.)
```

Each MCP server runs as a separate process that Claude Code communicates with. You configure which servers are available, and Claude Code discovers their tools automatically.

### Good Practices

**Start with one MCP server and learn its tools before adding more:**

```
claude --mcp-config mcp-config.json
```

Where `mcp-config.json` starts simple:

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@anthropic-ai/mcp-playwright"]
    }
  }
}
```

Get comfortable with one server's capabilities before stacking five together. Each server adds tools that Claude Code needs to reason about, and too many at once can slow down tool selection.

**Check what tools are available after connecting:**

```
What MCP tools do you have available? List them with a short
description of what each one does.
```

This gives you a catalog of what Claude Code can now do. You'll often discover capabilities you didn't know existed.

### Things to Avoid

**Adding every MCP server you can find:**

```json
{
  "mcpServers": {
    "playwright": { "...": "..." },
    "github": { "...": "..." },
    "slack": { "...": "..." },
    "jira": { "...": "..." },
    "postgres": { "...": "..." },
    "redis": { "...": "..." },
    "notion": { "...": "..." },
    "linear": { "...": "..." },
    "figma": { "...": "..." }
  }
}
```

More servers means more tools for Claude Code to choose from, which can lead to confusion or slower responses. Only add servers you're actively using for the current project.

**Assuming MCP servers are stateless:**

Many MCP servers maintain state (browser sessions, API tokens, open connections). If something stops working mid-session, the server process may have crashed or lost its authentication. Check the server logs and restart if needed.

---

## Lesson 3.2 — Playwright MCP: Debugging Web Applications

The Playwright MCP server gives Claude Code the ability to control a real web browser — navigating pages, clicking buttons, filling forms, taking screenshots, reading page content, and inspecting console errors. This turns Claude Code into a powerful debugging partner for frontend issues.

### Setup

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@anthropic-ai/mcp-playwright"]
    }
  }
}
```

### Example 1: Diagnosing a Visual Bug

You've received a bug report that says "the checkout button is hidden on mobile." Instead of manually opening dev tools, ask Claude Code to investigate:

```
Navigate to http://localhost:3000/cart using a mobile viewport
(375x812). Take a screenshot and tell me if the checkout button
is visible. If it's not, inspect the CSS for the checkout button
and figure out why it's hidden.
```

Claude Code will:

1. Open the browser at mobile resolution
2. Navigate to the cart page
3. Take a screenshot and analyze it
4. If the button is hidden, inspect the element's computed styles
5. Report back something like: "The `.checkout-btn` has `position: fixed; bottom: 0` but the `.cart-summary` div has `overflow: hidden` which clips it. Removing the overflow or repositioning the button should fix it."

You can then follow up:

```
Fix the CSS in src/styles/cart.css so the checkout button is
visible on mobile. Then take another screenshot to confirm.
```

### Example 2: Debugging a Form Submission Failure

Users report that the registration form "doesn't work" — no error, no redirect, just nothing happens:

```
Go to http://localhost:3000/register. Fill in the form with:
- Name: "Test User"
- Email: "test@example.com"
- Password: "SecurePass123!"
- Confirm Password: "SecurePass123!"

Click the Register button and watch what happens. Check the
browser console for any JavaScript errors and check the network
tab for the API request and response.
```

Claude Code will:

1. Navigate to the registration page
2. Fill in each field
3. Click Register
4. Monitor console logs for errors
5. Check the network request to see if the API was called and what it returned

It might report: "The form sends a POST to `/api/register` which returns a 422 with the message 'email format invalid'. The email field has a hidden Unicode character. The form's validation passes client-side but the server rejects it."

### Example 3: Testing a Full User Flow End-to-End

```
Test the complete purchase flow on http://localhost:3000:
1. Log in with user test@shop.com / password123
2. Search for "wireless keyboard"
3. Add the first result to the cart
4. Go to the cart and click checkout
5. Fill in a shipping address (use any fake US address)
6. Select "Standard Shipping"
7. Submit the order

At each step, take a screenshot. If anything fails or looks
wrong, stop and tell me what happened. At the end, check the
console for any errors that occurred during the flow.
```

This is essentially an automated QA session. Claude Code walks through the entire flow, documenting what it sees at each step, and catches issues a unit test would never find.

### Example 4: Comparing Responsive Layouts

```
Navigate to http://localhost:3000/dashboard. Take screenshots
at three viewport sizes:
1. Desktop (1920x1080)
2. Tablet (768x1024)
3. Mobile (375x812)

Compare the three and identify any layout issues — overlapping
elements, text truncation, buttons too small for touch, or
content that disappears at smaller sizes.
```

### Good Practices

**Use Playwright MCP for visual and interaction bugs that are hard to reproduce in tests:**

```
There's a bug where the dropdown menu stays open after clicking
outside of it. Go to http://localhost:3000/settings, open the
"Language" dropdown, then click on the page body outside the
dropdown. Take a screenshot to show if it closed or not.
Then check the event listeners on the dropdown component.
```

This kind of interaction bug is notoriously hard to test with unit tests but trivial for Playwright to reproduce.

**Combine Playwright with code fixes in one workflow:**

```
Navigate to http://localhost:3000/profile and test the avatar
upload. Upload a 5MB PNG file. If it fails or the error message
is wrong, fix the code in src/components/AvatarUpload.tsx and
then test again in the browser to confirm the fix works.
```

This is the full loop: observe the bug in the browser → fix the code → verify the fix in the browser.

### Things to Avoid

**Using Playwright for things you can test with unit tests:**

```
Open the browser and check if the formatDate function returns
the right format.
```

Just run the unit test. Playwright is for visual/interaction testing, not for testing pure functions.

**Forgetting that the dev server needs to be running:**

```
Navigate to http://localhost:3000 and take a screenshot.
```

If the dev server isn't running, this will fail. Either start it first or tell Claude Code to start it:

```
Start the dev server with `npm run dev`, wait for it to be
ready, then navigate to http://localhost:3000 and take a
screenshot.
```

---

## Lesson 3.3 — Jira / Azure DevOps MCP: Sprint Management and Daily Standup Prep

Project management MCP servers connect Claude Code to your team's issue tracker. This turns Claude Code into a sprint assistant that can read stories, check statuses, summarize progress, and help you prepare for standups.

### Setup: Jira MCP

```json
{
  "mcpServers": {
    "jira": {
      "command": "npx",
      "args": ["mcp-remote", "https://<your-mcp-jira-server-url>/sse"]
    }
  }
}
```

### Setup: Azure DevOps MCP

```json
{
  "mcpServers": {
    "azure-devops": {
      "command": "npx",
      "args": ["@nicepkg/mcp-server-azure-devops"],
      "env": {
        "AZURE_DEVOPS_ORG": "your-org",
        "AZURE_DEVOPS_PROJECT": "your-project",
        "AZURE_DEVOPS_PAT": "your-personal-access-token"
      }
    }
  }
}
```

### Example 1: Daily Standup Preparation

Every morning before standup, you can ask Claude Code to prepare a summary:

```
Check Jira for my current sprint. Find all tickets assigned to
me (john.smith@company.com). Group them by status (To Do,
In Progress, Done).

For each In Progress ticket, summarize:
- Ticket ID and title
- How long it's been in progress
- Any blockers or comments from the last 24 hours

For each Done ticket completed since yesterday, just list the
ticket ID and title.

Format this as a standup update I can paste into Slack with
the structure: "Yesterday I... / Today I'm... / Blocked by..."
```

Claude Code will query Jira, assemble the data, and produce something like:

> **Yesterday I:**
>
> - Completed PROJ-342: Add input validation to user registration endpoint
> - Completed PROJ-351: Fix timezone display in activity log
>
> **Today I'm working on:**
>
> - PROJ-348: Implement CSV export for admin reports (In Progress — 2 days, on track)
> - PROJ-355: Add retry logic to payment webhook handler (In Progress — started yesterday)
>
> **Blocked by:**
>
> - PROJ-348 depends on the new report schema from the data team (comment from @sarah.chen yesterday: "Schema review scheduled for Wednesday")

### Example 2: Understanding a User Story Before Coding

Before starting a new story, use Claude Code to get full context:

```
Look up PROJ-412 in Jira. Read the description, acceptance
criteria, and all comments. Also check if there are any linked
tickets (subtasks, blockers, or related issues).

Then read through our codebase to see which files are relevant
to this story. Give me a summary of:
1. What the story is asking for
2. Which files I'll likely need to modify
3. Any questions or ambiguities I should clarify before starting
```

This combines MCP (reading Jira) with local file analysis into a pre-implementation briefing. Claude Code might respond:

> **PROJ-412: Add bulk archive for completed projects**
>
> **What it asks for:** An admin endpoint and UI button to archive all projects with status "completed" older than 90 days. Archived projects should be soft-deleted and excluded from default queries.
>
> **Acceptance criteria:**
>
> - Admin-only endpoint `POST /api/admin/projects/bulk-archive`
> - Only archives projects completed > 90 days ago
> - Returns count of archived projects
> - Reversible (can unarchive individual projects)
>
> **Relevant files:**
>
> - `src/models/project.ts` — Already has `deletedAt` for soft-delete
> - `src/repositories/projectRepository.ts` — Needs a `bulkArchive` method
> - `src/routes/admin/projects.ts` — New endpoint here
> - `src/middleware/auth.ts` — Has `requireAdmin` middleware already
>
> **Questions to clarify:**
>
> - The story says "90 days" but doesn't specify timezone. Use UTC?
> - Should we send a notification to project owners before archiving?
> - The linked ticket PROJ-398 (add archive filter to project list) is still in backlog — should this story include the UI filter?

### Example 3: Azure DevOps — Sprint Progress Report

```
Query Azure DevOps for the current sprint in the "Backend" team.
Get all work items and calculate:
- Total story points committed
- Story points completed
- Story points remaining
- List any items that have been in "Active" status for more
  than 3 days (these might be stuck)

Format it as a sprint health check I can share with the team.
```

### Example 4: Updating Tickets After Completing Work

```
I just finished implementing the CSV export feature. Update
PROJ-348 in Jira:
- Move status to "Done"
- Add a comment: "Implemented CSV export with support for all
  report types. Added streaming for large datasets (>10k rows).
  Tests added in tests/integration/export.test.ts. PR: #247"
- Log 4 hours of work
```

This closes the loop — you code locally, and Claude Code updates the project tracker without you switching context.

### Good Practices

**Use Claude Code to cross-reference stories with code:**

```
Look at the last 5 tickets I completed in Jira this sprint.
For each one, check if there are corresponding test files in
the codebase. Flag any tickets that look like they might be
missing test coverage.
```

This combines project management data with codebase analysis — something no standalone tool does easily.

**Automate your standup prep as a daily habit:**

Create a prompt you reuse every morning:

```
Prepare my standup update. Check Jira for sprint BACKEND-24,
my assignments (john.smith@company.com), and any tickets where
I'm mentioned in comments from the last 24 hours. Use the
Yesterday/Today/Blockers format.
```

Some advanced users even save this as a custom slash command for one-keystroke standup prep.

### Things to Avoid

**Using MCP to make bulk changes without review:**

```
Move all "In Progress" tickets that are older than a week
to "Blocked" status.
```

This could cause confusion across the team. Bulk status changes should be reviewed before execution. Use `/plan` first:

```
/plan Show me all "In Progress" tickets older than a week.
For each one, what would be an appropriate action — mark as
blocked, add a "needs attention" label, or leave as-is?
Don't make any changes yet.
```

**Assuming Claude Code knows your team's Jira workflow:**

```
Transition PROJ-412 to the next status.
```

"Next status" depends on your workflow configuration. Be explicit: "Move PROJ-412 from 'In Review' to 'Done'." Or better, ask first: "What are the available transitions for PROJ-412?"

---

## Lesson 3.4 — GitHub MCP + Slack MCP: The Code-to-Communication Pipeline

One of the most powerful MCP combinations is GitHub + Slack. This lets Claude Code handle the full lifecycle: commit code, create a pull request, and notify your team — all without leaving the terminal.

### Setup

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "ghp_your_token_here"
      }
    },
    "slack": {
      "command": "npx",
      "args": ["mcp-remote", "https://mcp.slack.com/sse"]
    }
  }
}
```

### Example 1: The Full Commit → PR → Notify Flow

This is the flagship workflow. You've finished a feature and want to ship it with one compound prompt:

```
I've finished the CSV export feature on the `feat/csv-export`
branch. Do the following:

1. Stage and commit my changes with a descriptive commit message.
   Look at the diff to understand what changed and write a good
   commit message with a summary line and bullet points for the
   key changes.

2. Push the branch to origin.

3. Create a pull request on GitHub:
   - Target branch: main
   - Title: "Add CSV export for admin reports"
   - Description: Summarize what changed, why, and how to test it.
     Include a "Testing" section with manual test steps.
   - Add label: "feature"
   - Request review from @sarah-chen and @mike-jones

4. Send a message to the #backend-team Slack channel:
   "🚀 PR ready for review: Add CSV export for admin reports
   [link to PR]. Reviewers: @sarah @mike. This adds streaming
   CSV export for all report types with support for large
   datasets. Appreciate a review when you get a chance!"
```

Claude Code will execute each step sequentially:

1. Run `git diff` to see changes, craft a commit message, stage and commit
2. Push to the remote
3. Create the PR via GitHub MCP with the full description
4. Send the Slack notification with the PR link

The entire workflow that normally takes 10-15 minutes of context-switching (terminal → GitHub UI → Slack) happens in one prompt.

### Example 2: Smart Commit Messages from Diffs

```
Look at my current staged changes with `git diff --staged`.
Write a commit message following the Conventional Commits format.
The summary line should be under 72 characters. Include a body
that explains WHY the change was made (not just what changed).
Then commit with that message.
```

Claude Code reads the diff, understands the semantic meaning of the changes, and produces something like:

```
feat(export): add streaming CSV export for admin reports

The existing export loaded all rows into memory before generating
the CSV, which caused OOM errors for reports with >10k rows.

This replaces the batch approach with a streaming pipeline:
- Uses Node.js Transform streams to process rows incrementally
- Memory usage stays constant regardless of dataset size
- Adds Content-Disposition header for proper file download

Closes PROJ-348
```

### Example 3: Creating a Well-Structured PR

```
Create a pull request for the current branch against main.
Read through all the commits on this branch and generate:

- A clear title (imperative mood, under 72 chars)
- A description with these sections:
  ## What
  (one-paragraph summary of the change)

  ## Why
  (the problem this solves, link to Jira ticket if relevant)

  ## How
  (key implementation decisions and trade-offs)

  ## Testing
  (how to manually verify this works)

  ## Screenshots
  (note if any UI changes — say "N/A" if backend only)

Request review from the last 2 people who modified files
in the same directories I changed.
```

The "request review from the last 2 people who modified these files" is especially smart — Claude Code can check git blame via GitHub MCP to find the most relevant reviewers automatically.

### Example 4: Slack Notification with Context

```
Send a message to #backend-team on Slack:

"Hey team — I just opened PR #247 (CSV Export) which touches
the reporting module. Heads up for @data-team: the export
query uses the new `reports_v2` view, so if you're planning
schema changes this sprint, let's sync first.

PR: [include the URL from the PR you just created]
Estimated review time: ~30 min (450 lines, mostly new code)"
```

This is a thoughtful team notification — not just "PR created" but context about what's affected and who should care.

### Example 5: End-of-Day Summary to Slack

```
Look at all my git commits from today across all branches.
Also check my Jira tickets for any status changes today.

Compose a Slack message for #backend-team with my end-of-day
summary:
- What I committed today (with PR links if applicable)
- Which Jira tickets moved forward
- Any items I'll carry over to tomorrow
- Any blockers the team should know about

Keep it concise — no more than 10 lines.
```

This combines GitHub MCP (commit history) with Jira MCP (ticket status) and Slack MCP (notification) in a single workflow.

### Example 6: Reviewing a PR Before Merging

```
Check the open pull request #253 on GitHub. Read the diff and
tell me:
- What does this PR change?
- Are there any files with more than 200 lines changed?
  (those might need extra review attention)
- Are there test files included? Does the test coverage
  look reasonable for the changes?
- Are there any obvious issues — console.logs left in,
  TODO comments, commented-out code, or API keys?

Don't merge — just give me a review summary.
```

Claude Code reads the PR diff through GitHub MCP and gives you a pre-review briefing so you can focus your attention on the parts that matter.

### Good Practices

**Chain MCP actions in a logical pipeline:**

```
Here's my full end-of-feature workflow:

1. Run `npm run test` and `npm run lint` locally. If anything
   fails, fix it before continuing.
2. Stage all changes, generate a commit message from the diff,
   and commit.
3. Push to origin.
4. Create a PR on GitHub with a full description.
5. Update PROJ-348 in Jira — move to "In Review" and add a
   comment with the PR link.
6. Send a Slack message to #backend-team about the PR.

Execute all steps in order. If any step fails, stop and tell
me what went wrong.
```

This is a full CI-like pipeline from your terminal — test, commit, PR, ticket update, team notification. The "stop on failure" instruction prevents Claude Code from creating a PR for broken code.

**Save complex workflows as reusable prompts:**

If you do the commit → PR → Slack flow regularly, save the prompt template and adapt it each time. You can even put workflow templates in your CLAUDE.md:

```markdown
## Workflows

### Ship Feature

When I say "ship it", execute the standard workflow:

1. Run tests and lint
2. Commit with a Conventional Commits message from the diff
3. Push and create PR against main
4. Notify #backend-team on Slack with PR link
```

Then all you need to type is:

```
Ship it. Reviewers: @sarah and @mike. Jira ticket: PROJ-348.
```

### Things to Avoid

**Sending Slack messages without reviewing them first:**

```
Send a message to #engineering-all about the outage we just fixed.
```

Messages to broad channels (especially about incidents) should be reviewed before sending. Use the Explain-Before-Changing pattern:

```
Draft a message for #engineering-all about the outage fix.
Show me the draft before sending so I can review it.
```

**Creating PRs against the wrong branch:**

```
Create a PR for my changes.
```

Without specifying the target branch, Claude Code will guess (usually `main`). Be explicit:

```
Create a PR from `feat/csv-export` targeting the `develop` branch.
```

**Committing sensitive data through the automation flow:**

When Claude Code commits automatically, it commits whatever is staged — including files you may not have intended to commit. Be explicit about what to include:

```
Stage only the files in src/ and tests/. Do NOT stage any
.env files, config files with credentials, or anything in
the scripts/local/ directory. Show me `git status` after
staging so I can verify before you commit.
```

---

## Lesson 3.5 — Building Custom MCP Servers

When no existing MCP server fits your needs, you can build your own. Claude Code can even help you build it. Custom MCP servers let you expose internal tools, proprietary APIs, or company-specific workflows to Claude Code.

### Good Practices

**Start with the minimal viable server:**

```
Help me build a custom MCP server that connects to our internal
inventory API at https://inventory.internal.company.com/api.

Start with just two tools:
1. `getProductStock(productId)` — returns current stock level
2. `searchProducts(query)` — searches by name, returns top 10

Use the MCP TypeScript SDK. Keep it simple — we'll add more
tools later.
```

Don't try to wrap your entire API surface at once. Start with the two most useful tools and expand from there.

**Let Claude Code build and test the server:**

```
Build the MCP server, then test it by connecting to it locally
and calling each tool with sample data. Verify the responses
look correct.
```

Claude Code can scaffold the server, write the code, and validate that it works — all in one session.

### Things to Avoid

**Exposing destructive operations without safeguards:**

```
Add a `deleteAllProducts()` tool to the MCP server.
```

Any tool Claude Code can call, it _might_ call if it thinks it's relevant. Destructive operations should either not be exposed through MCP, or should have confirmation steps built in.

**Hardcoding credentials in the server code:**

```
Use our API key: sk-prod-12345 in the server.
```

Use environment variables instead. The MCP server config supports env vars:

```json
{
  "mcpServers": {
    "inventory": {
      "command": "node",
      "args": ["./mcp-servers/inventory/index.js"],
      "env": {
        "INVENTORY_API_KEY": "your-key-here"
      }
    }
  }
}
```

---

## Lesson 3.6 — Combining Multiple MCP Servers for Complex Workflows

The real power of MCP emerges when you combine servers. A single prompt can read a Jira ticket, explore the codebase, implement the feature, run Playwright tests, commit to GitHub, and notify the team on Slack.

### Good Practices

**Design end-to-end workflows that cross service boundaries:**

```
Here's what I need to ship the login rate-limiting feature:

1. Read PROJ-355 from Jira to get the full requirements and
   acceptance criteria.
2. Implement the feature based on those requirements in our
   codebase.
3. Run the existing tests to make sure nothing is broken.
4. Use Playwright to test the login page at http://localhost:3000
   — verify that after 5 failed login attempts, the 6th attempt
   shows a "Too many attempts" error message.
5. If everything passes, commit, push, and create a PR on GitHub.
6. Update PROJ-355 in Jira to "In Review" with the PR link.
7. Send a message to #backend-team on Slack about the PR.

Stop at any step if something fails and tell me what went wrong.
```

This is a full development cycle in one prompt — from reading the spec to notifying the team. Each MCP server handles its part: Jira for requirements, Playwright for browser testing, GitHub for version control, Slack for communication.

**Use `/plan` before complex multi-server workflows:**

```
/plan I want to do a full sprint review automation:
1. Pull all completed tickets from this sprint in Jira
2. For each ticket, find the associated PR on GitHub
3. Aggregate: total story points completed, PRs merged,
   lines changed, test coverage delta
4. Generate a sprint review summary
5. Post it to #sprint-review on Slack

Plan this out — which MCP tools will you use at each step?
Are there any steps where you'd need information you can't
get from the available MCP servers?
```

Planning first reveals gaps — maybe you need deployment metrics from a server you don't have, or the Jira-GitHub link requires a specific field.

### Things to Avoid

**Assuming all MCP servers share context:**

Each MCP server is independent. The GitHub server doesn't know what you just read from Jira. Claude Code is the bridge — it reads from one server and uses that information when calling another. This usually works transparently, but can be an issue if you expect one server to "know" about another.

**Running destructive actions across multiple servers without checkpoints:**

```
Close all completed Jira tickets, merge their PRs on GitHub,
and announce it on Slack.
```

If something goes wrong at step 2, you've already closed tickets in step 1. Use checkpoints:

```
First, show me the list of completed Jira tickets and their
associated PRs. Don't take any action yet — just show me
the list so I can confirm before you close/merge anything.
```

---

## Module 3 Assessment

### Project: Full-Cycle Development with MCP Integration

Students set up a development environment with at least three MCP servers (Playwright + one project management + GitHub or Slack).

**Part A: Setup and Discovery (graded)**
Configure the MCP servers, verify they're connected, and document the available tools. Students should explain which tools they'll use and why.

**Part B: Sprint Workflow (graded)**
Execute a complete mini-sprint cycle:

1. Read a user story from the project management tool
2. Plan the implementation using `/plan`
3. Implement the feature
4. Test it with Playwright in the browser
5. Commit and create a PR via GitHub MCP
6. Update the ticket status
7. Notify the team via Slack

**Part C: Custom Workflow (graded)**
Design and execute a custom multi-server workflow relevant to their own work. Examples: automated code review prep, sprint report generation, deployment checklist execution.

**Grading criteria:**

- Correct MCP server configuration and security (no hardcoded credentials)
- Effective use of MCP tools (choosing the right tool for each step)
- Quality of the cross-server workflow (logical ordering, error handling, checkpoints)
- PR quality (description, commit messages, reviewer selection)
- Team communication quality (Slack messages are clear, contextual, not spammy)
- Standup prep demonstrates real time savings over manual process
