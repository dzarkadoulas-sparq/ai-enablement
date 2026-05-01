# Edge case catalog (for assertions)

Use to **complete** test suites after happy paths. Not every item applies to every feature—pick what the code branch handles.

## Numbers and money

- **Zero, negative, very large** integers; **float** rounding (use fixed-decimal or integer cents in tests if that’s the domain rule).
- **Division by zero**; **overflow** where language allows.
- **Currency**: non-2-decimal inputs, mixed currency (if applicable), locale formatting only at UI boundary.

## Strings

- **Empty string**, **whitespace-only**, **Unicode** (combining chars, emoji), **very long** strings, **null** where nullable.
- **Case sensitivity** for IDs, emails, natural sort vs ASCII.

## Dates and time

- **UTC vs local**; **DST transitions**; **leap years**; **end-of-month** roll.
- **System clock** — inject or fake time; never assert on `Date.now()` without control.

## Collections

- **Empty**, **single element**, **duplicates**, **order** (stable vs unstable sort).
- **Pagination**: first/last page, page size 0 or max+1.

## Identifiers and auth

- **Malformed** UUIDs; **wrong tenant** or user; **expired** token; **missing** role.

## HTTP / APIs

- **400/404/409/422** with body shape from the project’s error contract.
- **Idempotency-Key** replay; **concurrent** updates if the code claims to handle them.

## Files and uploads

- **Empty file**, **oversize**, **wrong MIME**, **path traversal** in names.

## Async

- **Timeout**, **cancellation**, **rejection**; **parallel** calls where ordering matters.
