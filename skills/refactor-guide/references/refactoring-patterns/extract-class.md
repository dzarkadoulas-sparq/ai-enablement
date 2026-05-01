# Extract class / module / function

## When to use

- A **god class** or a file that is too large and has **multiple reasons to change** (single-responsibility split).
- **Duplicated** blocks of logic (repeated procedures or copy-paste clusters).

## Steps (mechanical)

1. **Name** the new abstraction: usually a **noun** for data+behavior, or a **verb**-based name for a pure use-case function.
2. **Create** the new type or module with the **smallest public API** you need for the first migration step.
3. **Move** fields and private methods (or file-local functions) into it.
4. **Update** call sites to use the new type. Optionally keep the old class as a **thin facade** for one PR, then remove it in a follow-up (strangler _inside_ the repo).

## Tests

- Prefer **moving code with existing tests** (cut-and-paste the test next to the code, then split).
- Or add a **characterization test** on the public entry before splitting, so behavior is locked.
- After the move, tests should still run, or be **split** to match new modules without losing assertions.

## Pitfalls

- Extracting **too many** tiny one-line utilities—aim for one **coherent role** per extracted unit.
