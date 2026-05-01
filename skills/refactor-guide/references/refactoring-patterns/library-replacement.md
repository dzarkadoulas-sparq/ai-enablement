# Library replacement (same role)

## When to use

- Deprecation, **security** advisories, **license** change, performance, or consolidating duplicate capabilities (e.g. one date library).

## Steps

1. **Inventory** imports and the **subset of the old API** the app actually uses.
2. **Pick** the new library; optionally **spike** in a branch behind an **adapter** that matches _your_ minimal needs (not the vendor’s full surface).
3. **Introduce** a facade/adapter in one or a few boundary modules.
4. **Swap** call sites in **waves** (by feature module or by directory) to keep reviews small.

## Tests

- **Contract tests** for _your_ adapter (inputs/outputs the app needs), with mapping from third-party types at the edge if needed.

## Pitfalls

- **Leaking** new library types across the whole codebase—narrow the import surface in your own modules and re-export only what callers need.

## Rollout

- If two libraries must coexist briefly (bundle or deploy size), time-box the period or use a feature flag to choose the implementation.
