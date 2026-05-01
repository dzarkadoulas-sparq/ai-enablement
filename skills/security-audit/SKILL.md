---
name: security-audit
description: >-
  Systematically reviews code and dependencies for common vulnerabilities: injection,
  broken auth, data exposure, unsafe redirects, file handling, secret leakage, and
  weak HTTP headers. Runs or recommends dependency scanners per stack. Produces a
  severity-ranked report with locations and fixes. Use when the user asks for a security
  scan, audit, OWASP check, or whether code is secure.
---

# Security audit

## Goal

Deliver a **prioritized report** (e.g. Critical / High / Medium / Low) with **file/line** (or dependency coordinates), **category** (CWE- or OWASP-mapped), and **concrete fix** or mitigation. This skill is **read-heavy**; run **tooling** the repo already defines (`package.json` scripts, Gradle tasks, `dotnet` CLI) rather than inventing new commands.

## When stack is unknown

1. **Detect** the stack (same heuristics as `project-bootstrap` / `pre-pr-review`).
2. **Read exactly one** stack file: `references/stacks/<id>.md` (commands + local focus).
3. **Read** the shared baselines (always):
   - `references/owasp-top10-checklist.md`
   - `references/secrets-patterns.md`
   - `references/header-recommendations.md`
4. **Read** the **language** guide: `references/language-specific/<file>.md` (mapped by stack file).

| `<id>`            | `language-specific` guide      |
| ----------------- | ------------------------------ |
| `react`           | `react-frontend-security.md`   |
| `next`            | `nextjs-security.md`           |
| `angular`         | `angular-frontend-security.md` |
| `node-express`    | `node-express-security.md`     |
| `python-fastapi`  | `python-fastapi-security.md`   |
| `java-springboot` | `java-spring-security.md`      |
| `dotnet`          | `dotnet-security.md`           |

## Workflow (map to the eight capability areas)

1. **Injection** — SQL/NoSQL/command/template/EL; map findings to `owasp-top10-checklist` A03.
2. **Authentication & authorization** — public vs protected routes, BOLA, session/JWT handling, method-level security (A07).
3. **Data exposure** — logs, error bodies, CORS, debug in prod, PII (A01/A04).
4. **Input & uploads** — missing validation, path traversal, content-type, size, redirects/open redirects (A10/A05).
5. **Dependencies** — run stack scanners from `references/stacks/<id>.md`; **CVE** and **transitive** risk; pin/upgrade path.
6. **Secrets** — `secrets-patterns.md` + `.env` hygiene; build-time leaks.
7. **HTTP security headers** — `header-recommendations.md` + how the stack applies them (middleware, `next.config`, `Program.cs`, Spring Security headers).
8. **Report** — table: **ID**, **Severity**, **Location**, **Finding**, **Fix**, **Tool** (or manual). No unfounded "pass" if tools did not run.

## Scope limits

- **DAST** and **formal pen-test** are out of scope for this skill unless the user runs them; note **gaps** in the report.
- If **MCP** or network tools are disabled, list **commands** the human can run locally/CI.
- **Do not** exfiltrate or paste **live** secrets; redact in the report.

## Exemplar repo (ai-enablement)

Stack-specific checks often align with **`claude.md/<id>/CLAUDE.md`** (Security sections) and **`commands/<id>/security-scan.md`** if present.

## Anti-patterns

- **False confidence** from `npm audit` only without reading **auth** and **injection** in changed files.
- **Dismissing** HIGH CVE on **unused** dep without verifying the dependency graph.
- **Missing** client-side **XSS** and **`NEXT_PUBLIC_*`** issues for full-stack and SPA stacks.

## Reference map

- `references/owasp-top10-checklist.md` — A01–A10 mapping and review prompts.
- `references/header-recommendations.md` — baseline headers and gaps.
- `references/secrets-patterns.md` — high-signal patterns; redact in output.
- `references/language-specific/*.md` — deep stack patterns.
- `references/stacks/<id>.md` — tool commands and 7-way entry point.

Valid `<id>` values: `react`, `next`, `angular`, `node-express`, `python-fastapi`, `java-springboot`, `dotnet`.
