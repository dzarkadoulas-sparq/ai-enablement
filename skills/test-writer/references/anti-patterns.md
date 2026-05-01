# Testing anti-patterns

## What NOT to test

- **Private / internal** details that can change without behavior change (private methods—test via public API; React **state** if output is already asserted via DOM).
- **Third-party** library behavior (test **your** integration boundary and error mapping).
- **Trivial** getters/setters with no logic.
- **Constants** that are re-exported without transformation.

## Brittle tests

- **Over-mocking** — every dependency mocked and only verifying call order; prefer one layer of fakes for **I/O** only.
- **Snapshot** everything — use for **stable** UI or large serializations; not for business rules that need explicit asserts.
- **Tied to implementation** — tests that break when renaming private functions or splitting a class without behavior change.
- **Order-dependent** suite without `beforeEach` isolation (shared global state, leaked listeners).

## Async and time

- **Unmocked** `setTimeout` / `setInterval` / `Date` in fast unit tests.
- **Missing** `await` in tests (false positives).

## React / component

- **Testing** implementation details: internal hook state, `useEffect` call counts, prop drilling to every child. Prefer **user-centric** queries (`@testing-library/*`).
- **No `act` awareness** when state updates are batched (usually RTL wraps; avoid legacy patterns).

## API / service

- **Hitting** production URLs or real credentials in **unit** tests.
- **Shared** DB not rolled back or truncated between cases when integration tests run in parallel.

## Noise

- **`console.error` in tests** without assert or without suppressing when expected.

## Coverage theater

- **Excluding** the hard files from the coverage config to green CI — only acceptable with explicit `// c8 ignore` and reason for truly unreachable branches.
