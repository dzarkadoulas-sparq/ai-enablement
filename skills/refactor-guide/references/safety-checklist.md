# Refactor safety checklist

## Before starting a phase

- [ ] The **goal of this phase** is one clear sentence (e.g. “Introduce `PaymentPort`; no direct Stripe SDK in features.”).
- [ ] **Tests** cover behavior to preserve (characterization or existing tests on the hot path).
- [ ] **Rollback** is defined: revert commits, or flip a feature flag; note the last known good ref.
- [ ] **Migrations** / data: if schema or on-disk format changes, backup and reversibility (or one-way with explicit runbook) are decided.

## During the phase

- [ ] The diff stays in **scope** (no repo-wide reformat unless agreed).
- [ ] **Refactor** and **bugfix** are not conflated in the same commit when both are present—split for clarity and revert.

## After the phase (before merge or next phase)

- [ ] **Lint, typecheck, and tests** match CI (or the stack’s pre-PR command).
- [ ] **Snapshots / golden** files: each change is **reviewed** (moved output vs. regression).
- [ ] If **public** API, OpenAPI, or ADR should change, they are **updated** (including accidental export surfaces).

## Intentional behavior change

- If behavior changes on purpose, **label** it and use a separate PR or commit if your team’s policy requires clean reverts.

## If verification fails

- **Fix** or **revert** the phase before declaring it done. Avoid stacking unrelated fixes on top of a still-failing refactor diff.
