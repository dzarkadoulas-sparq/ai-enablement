# Angular — client security focus

## Template and DOM

- **`[innerHTML]`** without **DomSanitizer** + policy; **`bypassSecurityTrustHtml`** / **Trust** methods only with **documented** exception and safe input.
- **Dynamic** component loaders loading **user-controlled** module paths (rare) — path traversal / malicious code.

## HttpClient

- **Interceptor** order: **auth** then **error** — ensure **tokens** are not logged.
- **withCredentials** + **CORS** — only for **known** API origins; avoid `*` with credentials.

## Environment

- **`environment.ts`**: **no** production secrets; **apiUrl** is often public — still no **admin** keys.

## Csrf

- If using **cookie** session with **server**: Angular often uses **XSRF** from framework — confirm **withCredentials** and **header** name match **backend** (no double-submit mistakes).

## Tools

- `pnpm audit --audit-level=high`; **npm** overrides for **CVE** resolution when policy allows.

## See also

`commands/angular/security-scan.md` in ai-enablement.
