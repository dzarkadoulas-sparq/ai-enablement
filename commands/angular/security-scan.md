Security scan of the current branch's diff against `origin/develop`.

1. **Scope** — `git diff origin/develop...HEAD -- '*.ts' '*.html' '*.scss' 'angular.json' 'package.json' 'pnpm-lock.yaml' 'src/environments/**'`.

2. **Code checks**
   - **XSS**: `DomSanitizer.bypassSecurityTrust*` with user-supplied
     content; `[innerHTML]` bound to untrusted content; dynamic
     `setAttribute('href', ...)` or `('src', ...)` with user input;
     `<a target="_blank">` without `rel="noopener noreferrer"`.
   - **Token handling**: `localStorage.setItem` with auth tokens;
     tokens added to URL query strings; tokens echoed into templates.
   - **Environments**: `src/environments/environment*.ts` containing
     anything that looks like a server secret — these files ship to
     the browser.
   - **HTTP**: `HttpClient` call bypassing the project's `AuthInterceptor`;
     `withCredentials: true` paired with a permissive backend CORS.
   - **Input validation**: route params / query params used without
     parsing (no Zod or explicit type narrowing) before being sent to
     the API or rendered.
   - **Logging**: `console.log(user)` or service logging full request
     bodies / tokens.
   - **Forms**: a Reactive Form accepting arbitrary `patchValue(obj)`
     from a remote source without validating the object shape.
   - **Third-party**: `<script>` tags added to `index.html` without
     `integrity=` + `crossorigin="anonymous"` when served from a CDN.

3. **Dependencies**
   - `pnpm audit --audit-level=high`.
   - Flag HIGH/CRITICAL CVEs in any dependency introduced in this branch.

4. **GitHub advisories** — via **GitHub MCP**, check Dependabot alerts
   for dependencies touched in this diff.

5. **Output** — structured findings:

   ```
   [SEVERITY] [FILE:LINE] [CATEGORY] — one-line description
     Why it's a risk: ...
     Fix: ... (specific, not just "sanitize input")
   ```

   Severities: `CRITICAL` / `HIGH` / `MEDIUM` / `LOW`. No emojis.

6. **No silent fixes** — report only.
