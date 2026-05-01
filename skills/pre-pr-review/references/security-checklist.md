# Security checklist (pre-PR)

Use on **changed** code paths and **new or upgraded** dependencies. Cite file:line in findings.

## Injection

- **SQL** — no string interpolation or concat of user/URL/path input into raw SQL, ORM raw methods, or `text(f"...")`-style dynamic SQL. Parameterized queries / bound parameters only.
- **Command / shell** — no `exec`/`spawn` with unsanitized user input.
- **SSRF** — if the code fetches user-supplied URLs, note lack of allowlist, DNS pinning, or network policy.

## Authn / authz

- New or changed **routes/endpoints**: authentication required unless explicitly public; role checks on sensitive actions.
- **BOLA/IDOR** — IDs in URLs/body: verify ownership/tenant scoping, not just “is logged in”.

## Secrets and config

- No new **API keys, tokens, connection strings, private keys** in source or committed `.env`.
- **Client-exposed** env (e.g. `VITE_*`, `NEXT_PUBLIC_*`) must not hold server-only secrets.
- **Logs** — no PII, tokens, or full auth headers in new log lines.

## Data exposure

- **Error responses** — no stack traces or internal paths in production error payloads (unless the stack’s pattern says otherwise in `CLAUDE.md`).
- **CORS** — new wide-open `*` on credentialed APIs is a flag (context-dependent).

## Deserialization and uploads

- **YAML/XML** — safe parsers only; no unsafe `pickle` / arbitrary object graphs from untrusted input.
- **File upload** — content-type, size, path; no execution of user-uploaded content without sandboxing.

## Dependencies

- **HIGH/CRITICAL** CVEs on the dependency graph **touched** by the branch: flag and suggest upgrade or pin.
- **Deprecated** critical packages: note for follow-up (P1).

## Web-specific (if applicable)

- **HTML** — if `innerHTML` / `dangerouslySetInnerHTML` / `DomSanitizer` bypass: require sanitization and documented exception.
- **CSP/headers** — if the PR changes middleware or `next.config` / Helmet, verify no broad unsafe-inline unless justified.

Stack-specific bullets also appear in `references/stacks/<id>.md` (diff hygiene).
