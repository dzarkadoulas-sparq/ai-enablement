# Debug report template

Use this structure in chat or in `docs/bugs/<slug>.md` if the user wants a file.

## Summary

- **Symptom:** one or two sentences.
- **URL / route:** full path or pattern.
- **Environment:** local dev, branch name, feature flag if any.

## Repro steps

1. …
2. …

## Expected vs. actual

|              |     |
| ------------ | --- |
| **Expected** | …   |
| **Actual**   | …   |

## Evidence (from Playwright MCP or manual)

- **Screenshots:** describe or attach; note viewport (e.g. 375×812 vs 1280×720).
- **Console:** error lines (redact tokens).
- **Network:** failed request method + path + status; CORS or mixed-content if applicable.

## Hypothesis

- Bullets tied to **files** or **APIs** you inspected.

## Fix (if applied)

- Files changed, **why** this addresses the evidence.

## Verification

- **After fix:** same steps → new screenshot or “passes.”
- **Regression:** optional link to a **Playwright** test file or “not added yet.”

## Metadata

- **Date** and **tooling** (e.g. `Playwright MCP` vs `manual DevTools`).
