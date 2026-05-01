Security scan of the current branch's diff against `origin/develop`.

1. **Scope** — `git diff origin/develop...HEAD -- '*.ts' '*.tsx' 'next.config.*' 'middleware.ts' 'package.json' 'pnpm-lock.yaml' '.env.example'`.

2. **Code checks**
   - **Server/client boundary**: server-only modules imported into a
     Client Component (could leak secrets); missing `import 'server-only';`
     at the top of modules that read secrets.
   - **Env vars**: server secrets behind `NEXT_PUBLIC_*` (these are
     PUBLIC — assume attackers read them). Secrets hardcoded anywhere
     outside `env.ts` / `.env*` / secret manager.
   - **Server Actions**: missing `requireUser()` / `requireRole()` as the
     first statement; missing Zod validation; leaking raw error messages.
   - **Route Handlers**: missing Zod validation; missing auth check;
     CORS config permitting credentialed wildcard.
   - **XSS**: `dangerouslySetInnerHTML` without sanitizer;
     `<Link href={userInput}>` without validation (`javascript:` URLs);
     `target="_blank"` without `rel="noopener noreferrer"`.
   - **Caching / SSRF**: `fetch(userProvidedUrl, ...)` in a Server
     Component or action — could be used to probe internal hosts. Allow-list
     the target domain(s).
   - **Cookies**: session cookies not `httpOnly` + `Secure` + `SameSite`.
   - **Security headers**: `next.config.mjs` `headers()` or `middleware.ts`
     missing HSTS, CSP (no `unsafe-*`), `X-Content-Type-Options`,
     `Referrer-Policy`, `Permissions-Policy`.
   - **Rate limiting**: auth / billing / destructive actions without a
     KV/Redis-backed limiter in `middleware.ts` or the action itself.
   - **Deserialization**: `JSON.parse(userInput)` fed into
     `dangerouslySetInnerHTML` or similar flows.

3. **Dependencies**
   - `pnpm audit --audit-level=high`.
   - Flag HIGH/CRITICAL in any dependency introduced in this branch.

4. **GitHub advisories** — via **GitHub MCP**, check Dependabot alerts
   for dependencies touched in this diff.

5. **Output** — structured findings:

   ```
   [SEVERITY] [FILE:LINE] [CATEGORY] — one-line description
     Why it's a risk: ...
     Fix: ... (specific, not just "add auth")
   ```

   Severities: `CRITICAL` / `HIGH` / `MEDIUM` / `LOW`. No emojis.

6. **No silent fixes** — report only.
