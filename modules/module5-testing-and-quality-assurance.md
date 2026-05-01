# Module 5 — Testing & Quality Assurance

## Detailed Lesson Plans with Examples

> **Note on language diversity:** This module deliberately mixes examples across
> Java (Spring Boot / JUnit), .NET (C# / xUnit), and React (TypeScript / Jest / React Testing Library)
> to show that Claude Code's testing capabilities are language-agnostic. The principles
> are the same — only the syntax changes.

---

## Lesson 5.1 — Having Claude Code Write Tests That Actually Matter

Anyone can ask Claude Code to "write tests." The skill is in getting tests that catch real bugs, cover meaningful edge cases, and don't just parrot the implementation back as assertions. This lesson covers how to prompt for high-value tests — tests that would actually fail if something broke.

### Example 1: Testing a Java Spring Boot Service with Edge Cases

**Good practice — specify the behaviors to test, not the methods:**

```
Write tests for the OrderService in
src/main/java/com/shop/service/OrderService.java.

Don't just test the happy path. I want tests for:
- Creating an order with valid items (happy path)
- Creating an order with an empty cart (should throw
  EmptyCartException)
- Creating an order when one item is out of stock (should
  throw InsufficientStockException and NOT create a partial order)
- Creating an order that exceeds the user's credit limit
- Applying a discount code that's expired
- Applying a discount code that's valid but for a different
  product category
- Concurrent orders that race for the last item in stock

Use JUnit 5 with Mockito. Mock the InventoryRepository and
PaymentGateway. Use @DisplayName for readable test names.
Follow the Arrange-Act-Assert pattern.
```

Claude Code produces tests like:

```java
@Test
@DisplayName("Should roll back entirely when one item is out of stock")
void createOrder_oneItemOutOfStock_rollsBackCompletely() {
    // Arrange
    var cartItems = List.of(
        new CartItem("SKU-001", 2),  // in stock
        new CartItem("SKU-002", 1)   // out of stock
    );
    when(inventoryRepo.checkStock("SKU-001")).thenReturn(10);
    when(inventoryRepo.checkStock("SKU-002")).thenReturn(0);

    // Act & Assert
    assertThrows(InsufficientStockException.class,
        () -> orderService.createOrder(userId, cartItems));

    // Verify no inventory was decremented (full rollback)
    verify(inventoryRepo, never()).decrementStock(anyString(), anyInt());
    verify(paymentGateway, never()).charge(any());
}
```

The key is testing _behavior_ ("should roll back entirely") not _implementation_ ("the method should call X then Y").

**Thing to avoid — asking for tests without specifying what matters:**

```
Write unit tests for OrderService.java.
```

Claude Code will generate tests for every public method with basic happy-path assertions. You'll get 100% method coverage but 20% behavior coverage. The tests pass today but won't catch tomorrow's bugs because they don't test edge cases, error paths, or invariants.

### Example 2: Testing a C# .NET Controller with Authentication Scenarios

**Good practice — test the integration boundaries:**

```
Write integration tests for the ProjectsController in
Controllers/ProjectsController.cs using xUnit and
Microsoft.AspNetCore.Mvc.Testing.

Test these scenarios for the GET /api/projects endpoint:
- Unauthenticated request returns 401
- Authenticated user sees only their own projects
- Admin user sees all projects
- Request with expired JWT returns 401 (not 500)
- Pagination: page=2&limit=10 returns correct slice and
  total count in response headers
- Filtering: status=active returns only active projects
- Empty result set returns 200 with empty array (not 404)

Use WebApplicationFactory<Program> for the test server.
Create test users with different roles using our
TestAuthHelper in Tests/Helpers/.
```

Claude Code produces structured tests:

```csharp
public class ProjectsControllerTests : IClassFixture<WebApplicationFactory<Program>>
{
    [Fact]
    public async Task GetProjects_AsRegularUser_ReturnsOnlyOwnProjects()
    {
        // Arrange
        var client = _factory.CreateAuthenticatedClient(TestUsers.RegularUser);

        // Seed: 3 projects for RegularUser, 2 for OtherUser
        await SeedProjectsAsync(userId: TestUsers.RegularUser.Id, count: 3);
        await SeedProjectsAsync(userId: TestUsers.OtherUser.Id, count: 2);

        // Act
        var response = await client.GetAsync("/api/projects");

        // Assert
        response.StatusCode.Should().Be(HttpStatusCode.OK);
        var projects = await response.DeserializeAsync<List<ProjectDto>>();
        projects.Should().HaveCount(3);
        projects.Should().OnlyContain(p => p.OwnerId == TestUsers.RegularUser.Id);
    }

    [Fact]
    public async Task GetProjects_WithExpiredJwt_Returns401NotServerError()
    {
        // Arrange
        var client = _factory.CreateClientWithExpiredToken(TestUsers.RegularUser);

        // Act
        var response = await client.GetAsync("/api/projects");

        // Assert
        response.StatusCode.Should().Be(HttpStatusCode.Unauthorized);
        // Critical: expired tokens should NOT cause 500
        response.StatusCode.Should().NotBe(HttpStatusCode.InternalServerError);
    }
}
```

**Thing to avoid — testing only the happy path:**

```
Write tests for the ProjectsController. Test that GET /api/projects
returns a list of projects.
```

One test, one assertion, zero edge cases. This test will pass even if authentication is completely broken, pagination doesn't work, or admins can't see anything.

### Example 3: Testing a React TypeScript Component with User Interactions

**Good practice — test from the user's perspective:**

```
Write tests for the CheckoutForm component in
src/components/CheckoutForm.tsx using React Testing Library
and Jest.

Test these user scenarios:
- User fills in all fields correctly and submits → loading
  spinner appears, then success message
- User submits with empty required fields → inline validation
  errors appear next to each empty field
- User enters invalid email format → specific email error shown
- User enters credit card number with wrong number of digits →
  card error shown
- Network error on submit → error banner with retry button
- User clicks retry after error → form resubmits with same data
- User navigates away during submission → no state update on
  unmounted component (no memory leak warning)

Use userEvent (not fireEvent) for realistic user interactions.
Mock the API call with msw or jest.mock. Don't test
implementation details like state variables — test what the
user sees.
```

Claude Code produces tests like:

```typescript
test('shows inline validation errors for empty required fields', async () => {
  const user = userEvent.setup();
  render(<CheckoutForm />);

  // Submit without filling anything
  await user.click(screen.getByRole('button', { name: /place order/i }));

  // Each required field shows its own error
  expect(screen.getByText(/name is required/i)).toBeInTheDocument();
  expect(screen.getByText(/email is required/i)).toBeInTheDocument();
  expect(screen.getByText(/address is required/i)).toBeInTheDocument();
  expect(screen.getByText(/card number is required/i)).toBeInTheDocument();

  // Form was NOT submitted
  expect(mockSubmitOrder).not.toHaveBeenCalled();
});

test('retains form data when retrying after network error', async () => {
  const user = userEvent.setup();
  mockSubmitOrder
    .mockRejectedValueOnce(new Error('Network error'))
    .mockResolvedValueOnce({ orderId: '12345' });

  render(<CheckoutForm />);

  // Fill form
  await user.type(screen.getByLabelText(/name/i), 'Jane Doe');
  await user.type(screen.getByLabelText(/email/i), 'jane@test.com');
  // ... fill other fields

  // First submit fails
  await user.click(screen.getByRole('button', { name: /place order/i }));
  expect(await screen.findByText(/something went wrong/i)).toBeInTheDocument();

  // Retry succeeds with same data
  await user.click(screen.getByRole('button', { name: /retry/i }));
  expect(mockSubmitOrder).toHaveBeenCalledTimes(2);
  expect(mockSubmitOrder).toHaveBeenLastCalledWith(
    expect.objectContaining({ name: 'Jane Doe', email: 'jane@test.com' })
  );
  expect(await screen.findByText(/order confirmed/i)).toBeInTheDocument();
});
```

**Thing to avoid — testing implementation details:**

```
Write tests that check the component's state after each action.
Verify that useState is called with the correct values and that
the error state is set to null after successful submission.
```

This tests _how_ the component works internally, not _what_ the user experiences. If you refactor from `useState` to `useReducer`, every test breaks even though the component still works perfectly. Test the rendered output, not the internals.

---

## Lesson 5.2 — Test-Driven Development (TDD) Workflows

TDD with Claude Code follows a disciplined cycle: you describe the behavior, Claude Code writes a failing test, then Claude Code writes the minimum code to make it pass. The test comes first, and the test defines the contract.

### Example 1: TDD for a Java Pricing Engine

```
We're building a pricing engine. Let's use TDD.

Step 1: Write a failing test for this rule:
"A standard customer ordering 10+ units of the same product
gets a 10% bulk discount applied to those items."

Use JUnit 5. The test should define the interface — what class
name, method signature, and return type you'd expect. Run the
test to confirm it fails (compilation error is fine at this stage).
```

Claude Code writes:

```java
@Test
@DisplayName("Standard customer gets 10% discount on 10+ units")
void calculatePrice_bulkOrder_appliesDiscount() {
    var engine = new PricingEngine();
    var lineItem = new LineItem("WIDGET-A", quantity: 10, unitPrice: 50.00);
    var customer = new Customer("C-001", CustomerTier.STANDARD);

    var result = engine.calculatePrice(lineItem, customer);

    assertThat(result.getSubtotal()).isEqualTo(500.00);
    assertThat(result.getDiscount()).isEqualTo(50.00);    // 10%
    assertThat(result.getTotal()).isEqualTo(450.00);
}
```

Then:

```
Good. Now write the minimum implementation to make this test
pass. Don't add any logic for cases we haven't tested yet —
no premium customer discounts, no quantity caps, nothing extra.
Just enough to pass this one test.
```

Then add the next test:

```
Step 2: Write a failing test for:
"A premium customer gets 15% bulk discount on 10+ units
instead of 10%."

Run both tests — the new one should fail, the old one should
still pass. Then implement to make both pass.
```

This cycle continues, building up the pricing engine rule by rule with full test coverage.

### Example 2: TDD for a C# Validation Pipeline

```
We need an input validation pipeline for user registration.
Let's TDD it in C#.

Write the first xUnit test:
"When email is null or empty, validation fails with error
code INVALID_EMAIL and message 'Email is required'."

The validator doesn't exist yet — let the test define
the interface. Use FluentAssertions for readable assertions.
```

Claude Code writes:

```csharp
[Theory]
[InlineData(null)]
[InlineData("")]
[InlineData("   ")]
public void Validate_MissingEmail_ReturnsInvalidEmailError(string? email)
{
    // Arrange
    var request = new RegistrationRequest { Email = email, Password = "Valid1!" };
    var validator = new RegistrationValidator();

    // Act
    var result = validator.Validate(request);

    // Assert
    result.IsValid.Should().BeFalse();
    result.Errors.Should().ContainSingle(e =>
        e.Code == "INVALID_EMAIL" &&
        e.Message == "Email is required");
}
```

```
Good, the test fails because RegistrationValidator doesn't
exist. Implement it — just enough to pass this test.

Then write the next test: "When email doesn't contain @ symbol,
validation fails with INVALID_EMAIL and 'Email format is invalid'."
```

### Good Practices

**Let the tests drive the API design:**

```
Don't create the interface first. Write the test as if the
ideal API already exists — whatever feels most natural to
call. Then implement that API. The test is the first consumer
of your code, so if it's awkward to test, it'll be awkward
to use.
```

**Keep each TDD cycle small:**

```
One behavior per cycle. Write one test, make it pass, then
write the next test. Don't write 10 tests at once and then
implement everything — you lose the feedback loop.
```

### Things to Avoid

**Writing the implementation first and tests second:**

```
Implement the PricingEngine with all the discount rules.
Then write tests for it.
```

This inverts TDD. The tests will unconsciously mirror the implementation rather than defining the behavior. You'll test that the code does what it does, not that it does what it _should_.

**Writing tests that are too tied to a specific implementation:**

```
Write a test that verifies PricingEngine calls
DiscountCalculator.calculate() exactly once with the
correct parameters.
```

This tests the wiring, not the behavior. If you refactor to inline the discount calculation, the test breaks even though the pricing is still correct. Test the _output_ (correct price), not the _internal calls_.

---

## Lesson 5.3 — Increasing Test Coverage on Legacy Codebases

Legacy code without tests is one of the hardest problems in software engineering. You can't refactor safely without tests, but the code is hard to test because it wasn't designed for testability. Claude Code can help by analyzing untested code, identifying the most critical gaps, and writing tests that pin down current behavior before you start changing things.

### Example 1: Characterization Tests for a Java Legacy Service

```
Look at src/main/java/com/legacy/billing/InvoiceProcessor.java.
This class has zero tests and we need to refactor it. Before
we change anything, write characterization tests that capture
the current behavior — even if the current behavior has bugs.

The goal is NOT to test what it should do. The goal is to test
what it ACTUALLY does right now, so we'll know if the refactor
changes anything.

Approach:
1. Read the class and identify all code paths
2. For each path, write a test that asserts the current output
3. If the code has side effects (database calls, file writes),
   mock them and verify the mock interactions
4. Name each test descriptively: "currentBehavior_when_X_does_Y"

Use JUnit 5 with Mockito. The class probably has dependencies
that are hard to inject — use reflection to set private fields
if needed. We'll fix the design later.
```

Claude Code reads a messy 300-line class and produces tests like:

```java
@Test
@DisplayName("Current behavior: negative line items are silently ignored")
void currentBehavior_negativeQuantity_isIgnored() {
    // Note: this is probably a bug, but we're capturing current
    // behavior, not desired behavior
    var items = List.of(
        new LineItem("A", 10, 5.00),
        new LineItem("B", -3, 5.00)  // negative quantity
    );

    var invoice = processor.generateInvoice(items, customer);

    // Currently calculates 10*5 = 50.00, ignores the negative item
    assertThat(invoice.getTotal()).isEqualTo(new BigDecimal("50.00"));
    assertThat(invoice.getLineItems()).hasSize(1); // negative item dropped
}
```

These tests are your safety net. Now you can refactor the class, and if any test breaks, you know exactly which behavior changed.

### Example 2: Finding Coverage Gaps in a C# Project

```
Run `dotnet test --collect:"XPlat Code Coverage"` and analyze
the coverage report. Find the 5 files with the lowest coverage
that have the highest risk (measured by: number of public methods,
cyclomatic complexity, and how many other files depend on them).

For the riskiest file, write tests that cover the untested paths.
Focus on error handling paths first — those are the most likely
to have bugs and the least likely to be tested.
```

### Example 3: Adding Tests to Untested React Components

```
Run `npx jest --coverage` and identify React components in
src/components/ with less than 30% coverage. For each one:

1. Read the component and list the user-visible behaviors
2. Check which behaviors are already tested and which aren't
3. Write tests for the untested behaviors

Prioritize components that handle user input or money (forms,
checkout, payment) over display-only components.

Use React Testing Library. Don't test styled-components or
CSS classes — test behavior.
```

### Good Practices

**Start with the highest-risk, lowest-coverage code:**

```
Don't aim for 100% coverage everywhere. Look at the coverage
report and our git history. Find files that are:
1. Changed frequently (high churn = high risk of regressions)
2. Have low test coverage
3. Handle critical business logic (payments, auth, data integrity)

These are the files where tests add the most value. Start there.
```

**Write tests incrementally, not all at once:**

```
Let's add coverage to the billing module. Today, just focus
on InvoiceProcessor. Write 5-7 tests that cover the most
important paths. We'll add more next week.

Run coverage after to see our improvement.
```

### Things to Avoid

**Chasing coverage numbers without testing meaningful behavior:**

```
Get this file to 100% code coverage.
```

Claude Code will write tests that execute every line — including trivial getters, constructors, and toString methods — without testing any meaningful behavior. 100% coverage with meaningless assertions is worse than 60% coverage with good tests, because it creates a false sense of safety.

**Writing tests that are too brittle for legacy code:**

```
Write exact assertion tests for every return value.
```

Legacy code often has inconsistent behavior. Overly precise assertions will break constantly as you refactor, turning the test suite into an obstacle rather than a safety net. Use broader assertions where appropriate:

```
For the characterization tests, use flexible assertions where
the exact value might change slightly during refactoring:
- Assert the total is positive (not the exact dollar amount)
  unless the exact amount is part of the business rule
- Assert the list is non-empty rather than checking exact size
  where the size isn't part of the contract
- Assert the response contains required fields rather than
  checking every field value
```

---

## Lesson 5.4 — API Testing with Postman MCP Server

The Postman MCP server lets Claude Code interact with Postman workspaces — running collections, inspecting API responses, managing environments, and validating API behavior. This turns Claude Code into an API testing assistant that can design, run, and debug API tests without leaving your development workflow.

### Setup

```json
{
  "mcpServers": {
    "postman": {
      "command": "npx",
      "args": ["-y", "@anthropic-ai/postman-mcp-server"],
      "env": {
        "POSTMAN_API_KEY": "PMAK-your-api-key-here"
      }
    }
  }
}
```

### Example 1: Exploring and Testing an Existing API Collection

```
Connect to my Postman workspace and list all collections.
Find the "Order Management API" collection and show me:
- All the endpoints organized by folder
- Which endpoints have tests and which don't
- Which environments are available (dev, staging, prod)
```

Then:

```
Run the "Order Management API" collection against the "dev"
environment. Show me the results — which requests passed,
which failed, and for any failures, show me the request,
expected response, and actual response.
```

Claude Code runs the collection and reports:

> **Collection run: Order Management API (dev)**
>
> ✅ POST /api/orders — 201 Created (142ms)
> ✅ GET /api/orders/:id — 200 OK (38ms)
> ❌ PUT /api/orders/:id/status — Expected 200, got 403 Forbidden
> ✅ GET /api/orders?status=pending — 200 OK (56ms)
> ❌ DELETE /api/orders/:id — Expected 204, got 500 Internal Server Error
>
> **Failure details:**
>
> PUT /api/orders/:id/status:
>
> - Request: `{ "status": "shipped" }` with Bearer token
> - Expected: 200 with updated order
> - Actual: 403 `{ "error": "Insufficient permissions" }`
> - Likely cause: The test user in the dev environment doesn't have the
>   `orders:update` permission.
>
> DELETE /api/orders/:id:
>
> - Request: DELETE with valid order ID
> - Expected: 204 No Content
> - Actual: 500 `{ "error": "Cannot delete order with active shipments" }`
> - Likely cause: Test data has a shipment attached. Need to cancel shipment first.

### Example 2: Creating API Tests from Scratch

```
I've built a new User Management API but haven't written any
Postman tests yet. Create a comprehensive test collection for
these endpoints:

POST   /api/users          — Create user
GET    /api/users/:id      — Get user by ID
PUT    /api/users/:id      — Update user
DELETE /api/users/:id      — Soft-delete user
GET    /api/users          — List users (paginated)
POST   /api/users/search   — Search users by criteria

For each endpoint, create requests that test:
- Happy path with valid data
- Validation errors (missing fields, wrong types, too long)
- Authentication (no token, expired token, wrong role)
- Edge cases specific to that endpoint

Add Postman test scripts that verify status codes, response
schema, and response times (should be under 500ms).

Use the "dev" environment and create variables for base_url,
auth_token, and test_user_id so the collection is portable
across environments.
```

Claude Code creates a structured Postman collection with folders for each endpoint, test scripts using `pm.test()`, and environment-aware variables.

### Example 3: Contract Testing Between Frontend and Backend

```
Our React frontend expects specific response shapes from the
API. Read the TypeScript interfaces in
src/types/api/responses.ts — these define what the frontend
expects.

Now run the Postman collection for the User API and compare
the actual responses against the TypeScript interfaces.
Flag any mismatches:
- Fields present in the API response but missing from the
  TypeScript type (frontend ignores extra data — low risk)
- Fields in the TypeScript type but missing from the API
  response (frontend will crash — HIGH risk)
- Type mismatches (API returns string, frontend expects number)

Create a report of all contract violations.
```

This cross-references your frontend types with your actual API responses — catching integration bugs before they reach production.

### Example 4: Testing API Workflows (Multi-Step Sequences)

```
Create a Postman collection that tests the full order lifecycle
as a workflow. Each request should use data from the previous
response:

1. POST /api/auth/login → capture the auth token
2. POST /api/cart/items → add 3 items (use token from step 1)
3. GET /api/cart → verify all 3 items are there
4. POST /api/orders → create order from cart
   → capture order_id from response
5. GET /api/orders/:order_id → verify order status is "pending"
6. PUT /api/orders/:order_id/status → change to "confirmed"
7. GET /api/orders/:order_id → verify status changed
8. DELETE /api/cart → verify cart is empty after order

Chain the requests using Postman collection variables.
Add test scripts at each step that verify the response
and extract variables for the next step.

Run the entire workflow against dev and show me the results.
```

### Example 5: Load Testing Preparation

```
I want to understand our API's performance characteristics
before we load test. Run these Postman requests 10 times each
against the dev environment and collect response times:

- GET /api/products (list — no filters)
- GET /api/products?category=electronics (filtered)
- GET /api/products/:id (single item)
- POST /api/orders (create — most complex endpoint)

For each endpoint, report:
- Average response time
- Min and max response time
- Any responses over 500ms (flag these)
- Any non-200 responses (reliability issues)

Based on the results, which endpoints should we focus our
load testing on?
```

### Example 6: Debugging a Failing API with Postman + Code Fix

```
The POST /api/users endpoint is returning 500 errors
intermittently. Use Postman to:

1. Send 20 requests with valid data and track which ones
   fail vs succeed
2. Compare the failing requests to the successful ones —
   is there a pattern? (request size, specific field values,
   timing?)
3. Check the response bodies of failed requests for error
   details

Then look at the handler code in our codebase
(src/controllers/UsersController.cs) and correlate the
Postman failures with the code. Find and fix the bug.

After fixing, re-run the 20 requests to confirm 100% pass.
```

This combines Postman MCP (API testing) with local code access (debugging and fixing) in a single diagnostic workflow.

### Good Practices

**Use environment variables for everything that changes between environments:**

```
Set up the Postman collection with these environment variables:
- base_url (dev: localhost:3000, staging: api-staging.company.com)
- auth_token (refreshed per run)
- test_user_id (different per environment)

No hardcoded URLs or tokens in the requests themselves.
```

**Test error responses as thoroughly as success responses:**

```
For the POST /api/users endpoint, I want MORE error tests
than success tests:
- Missing each required field individually
- Each field with invalid type (number instead of string, etc.)
- Email that already exists (409 Conflict)
- Request body exceeding size limit
- Malformed JSON
- SQL injection attempt in the name field
- XSS attempt in the bio field

Verify each returns the correct status code AND a useful
error message (not just "Internal Server Error").
```

**Chain Postman results back into code fixes:**

```
Run the full API collection. For any failing tests, look at
the corresponding code in our codebase, diagnose the issue,
and fix it. Then re-run just the failing tests to confirm
they pass. Repeat until the full collection is green.
```

### Things to Avoid

**Testing against production:**

```
Run the collection against the production environment to see
if everything works.
```

API tests often create, modify, and delete data. Running them against production can create test users, fake orders, or corrupt real data. Always use a dev or staging environment, and make sure the test data is clearly marked as test data.

**Creating tests that depend on specific database state:**

```
Test that GET /api/users/42 returns "John Doe".
```

If user 42 gets deleted or renamed, the test breaks for reasons unrelated to the code. Use setup/teardown steps:

```
For each test that needs specific data, create the data in
a setup step (POST to create the resource), test against it,
and clean it up in a teardown step (DELETE). Don't rely on
data already existing in the database.
```

**Writing Postman tests that only check status codes:**

```
pm.test("Status is 200", function () {
    pm.response.to.have.status(200);
});
```

A 200 with a completely wrong response body still passes this test. Always validate the response shape:

```
Add schema validation to every test. Check that:
- Required fields are present
- Field types are correct (string, number, array)
- Pagination metadata is correct (total, page, pageSize)
- IDs are valid formats (UUID, etc.)
- Dates are in ISO 8601 format
```

---

## Lesson 5.5 — Linting, Type Checking, and Static Analysis

Tests catch runtime bugs, but static analysis catches structural issues before the code even runs: unused imports, type mismatches, unreachable code, security patterns, and style violations. Claude Code can run these tools, interpret their output, and fix the issues in one pass.

### Example 1: Fixing All TypeScript Strict Mode Violations

```
We just enabled "strict": true in tsconfig.json and now there
are 147 TypeScript errors. Run `npx tsc --noEmit` and analyze
the output.

Categorize the errors:
1. Implicit `any` types — need explicit type annotations
2. Possible `null` or `undefined` — need null checks or
   optional chaining
3. Missing return types on functions
4. Other

Fix them in priority order: null safety issues first (these
cause runtime crashes), then implicit any, then return types.
Work through the files one at a time. After fixing each file,
run `npx tsc --noEmit` again to track progress.
```

### Example 2: Running and Fixing C# Analyzer Warnings

```
Run `dotnet build` and show me all the analyzer warnings.
Focus on these categories:
- CA2007 (ConfigureAwait) — fix with ConfigureAwait(false)
  on library code only, not on controller/UI code
- CA1062 (null parameter validation) — add null checks or
  use the [NotNull] attribute where appropriate
- CA2227 (collection properties should be read only) — evaluate
  case by case, some are DTOs that need setters for deserialization

Fix the CA2007 warnings first since those are mechanical.
Show me the CA1062 warnings for my review before fixing,
since some might need design changes.
```

### Example 3: Java SpotBugs and SonarQube Integration

```
Run SpotBugs on the project with `mvn spotbugs:check` and
analyze the report. For each finding:
- HIGH confidence bugs: Fix immediately
- MEDIUM confidence: Show me the code and your proposed fix
- LOW confidence: List them but don't fix (might be false positives)

Pay special attention to:
- NP_NULL_ON_SOME_PATH (null pointer dereferences)
- SQL_INJECTION (SQL injection vulnerabilities)
- EI_EXPOSE_REP (returning mutable internal state)

After fixing the HIGH findings, re-run SpotBugs to confirm
they're resolved and no new ones appeared.
```

### Good Practices

**Run static analysis before tests — fix structural issues first:**

```
Before writing any new tests, run the full lint and type-check
suite. Fix any existing issues first so we have a clean baseline.
Then write new tests against clean code.
```

**Configure Claude Code's CLAUDE.md with your lint rules:**

```markdown
## Static Analysis

- Run `npm run lint` before committing. Zero warnings allowed.
- TypeScript strict mode is ON. No @ts-ignore without a comment explaining why.
- ESLint rule no-floating-promises is critical — never ignore it.
- C# projects: treat all analyzer warnings as errors in CI.
```

### Things to Avoid

**Suppressing warnings instead of fixing them:**

```
Add // eslint-disable-next-line to all the lint errors so
the build passes.
```

This silences the warnings without fixing the problems. Claude Code should fix the underlying issue. Suppression is only acceptable with a comment explaining _why_:

```
If a lint warning is a genuine false positive, you may suppress
it, but add a comment explaining why. For example:
// eslint-disable-next-line @typescript-eslint/no-non-null-assertion
// We've already validated this is non-null in the guard clause above
```

---

## Lesson 5.6 — Continuous Integration: Having Claude Code Fix the Build

When CI fails, the traditional workflow is: read the CI log, find the failure, switch to your editor, make the fix, push, wait for CI again. Claude Code can compress this loop dramatically by reading CI output and fixing issues in one pass.

### Example 1: Fixing a Failing Java CI Build

```
Our GitHub Actions CI build failed. Here's the error output
from the build log:

[paste CI log or path to log file]

Diagnose the failure and fix it. The build runs:
1. mvn compile
2. mvn test
3. mvn spotbugs:check
4. mvn checkstyle:check

Identify which step failed and why. Fix the issue, run the
same commands locally to verify, then commit the fix.
```

### Example 2: Fixing a Failing .NET CI Pipeline

```
Our Azure DevOps pipeline failed at the test stage. The error is:

Tests.Integration.PaymentControllerTests.ProcessPayment_ValidCard_ReturnsSuccess
  Expected: 200
  Actual: 503

This test passes locally. The difference might be environment-related.
Read the test code, the controller code, and the CI pipeline
config (azure-pipelines.yml) to figure out why it fails in CI
but passes locally. Common causes:
- Missing environment variables in CI
- Database not seeded in CI
- External service not mocked in CI

Find the root cause and fix it so the test passes in both
environments.
```

### Example 3: React CI with Type Errors and Test Failures

```
Our CI runs these steps and is failing at step 2:
1. npm run lint ✅
2. npm run type-check ❌
3. npm run test
4. npm run build

The type-check error is:
src/hooks/useOrders.ts:34:5 - error TS2322: Type 'Order | undefined'
is not assignable to type 'Order'.

Fix the type error without using `as` casts or non-null
assertions. Use proper type narrowing or update the consuming
components to handle the undefined case.

After fixing, run all 4 steps locally to make sure the full
CI pipeline would pass.
```

### Good Practices

**Reproduce CI failures locally before fixing:**

```
Our CI uses these commands: [list]. Run them locally in the
same order. If the failure reproduces, fix it. If it doesn't,
the issue is environment-specific and we need to investigate
the CI config.
```

**Fix the root cause, not the symptom:**

```
The CI test failure is in the assertion, but the root cause
might be elsewhere. Before changing the test or the asserted
value, trace back to understand WHY the value is different
from expected. Fix the actual bug, not the test.
```

### Things to Avoid

**Pushing untested fixes hoping CI will pass:**

```
I think this fixes it. Push it and let's see if CI passes.
```

Running CI takes minutes. Run the same checks locally first — it takes seconds and saves a round-trip.

---

## Module 5 Assessment

### Project: Testing a Multi-Language Microservices App

Students receive a small application with three components:

1. **Java Spring Boot API** — Order management (30% test coverage)
2. **C# .NET API** — User and authentication service (20% test coverage)
3. **React TypeScript Frontend** — Dashboard and forms (15% test coverage)
4. **Postman Collection** — Partial, with several endpoints untested

**Part A: Coverage Analysis (graded)**

Analyze all three components. Identify the highest-risk, lowest-coverage areas. Create a prioritized testing plan.

**Part B: TDD New Feature (graded)**

Use TDD to add a new feature to the Java API: "Premium users get free shipping on orders over $50." Write failing tests first, then implement.

**Part C: Legacy Characterization Tests (graded)**

The C# authentication service has a complex `TokenValidator` class with zero tests. Write characterization tests that capture current behavior.

**Part D: React Component Tests (graded)**

The React checkout form has no tests. Write comprehensive user-perspective tests covering happy paths, validation, error states, and loading states.

**Part E: API Testing with Postman (graded)**

Complete the Postman collection to cover all endpoints across both APIs. Create a multi-step workflow test for the full order lifecycle. Run the collection and fix any API bugs discovered.

**Part F: CI Pipeline (graded)**

Configure and fix a CI pipeline that runs all test suites. The provided pipeline config has 3 deliberate issues that cause false failures in CI.

**Grading criteria:**

- Testing strategy prioritization (risk-based, not coverage-chasing)
- TDD discipline (tests before code, minimal implementation per cycle)
- Characterization test quality (captures actual behavior, not assumed behavior)
- React test quality (user-perspective, no implementation detail testing)
- Postman collection completeness (error cases, workflows, schema validation)
- CI pipeline reliability (all tests pass locally AND in CI)
