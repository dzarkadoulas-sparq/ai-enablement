# HTTP security headers (baseline)

Compare **actual** response headers (or framework config) to this baseline. **Report** missing or dangerous values. Framework-specific wiring lives in `language-specific` guides.

## Recommended (typical BFF/API behind reverse proxy)

| Header | Purpose | Notes |
| ------ | ------- | ----- |
| `Strict-Transport-Security` | HSTS | `max-age` + `includeSubDomains` when TLS everywhere |
| `X-Content-Type-Options` | Sniffing | `nosniff` |
| `X-Frame-Options` or `frame-ancestors` in **CSP** | Clickjacking     | `DENY` or `SAMEORIGIN` or CSP `frame-ancestors` |
| `Content-Security-Policy` | XSS / injection  | stricter is better; avoid `unsafe-inline` / `unsafe-eval` unless documented |
| `Referrer-Policy` | Leakage | `strict-origin-when-cross-origin` or stricter |
| `Permissions-Policy` | Feature lockdown | disable unused: geolocation, camera, etc. |

## Cookies (when Set-Cookie is set)

- **`Secure`**, **`HttpOnly`**, **`SameSite=Strict` or `Lax`** for session cookies; path and domain **minimal**.

## CORS (APIs)

- **Never** `Access-Control-Allow-Origin: *` with **`Access-Control-Allow-Credentials: true`**.
- Echo **origin** only for **known** allowlist; **Vary: Origin** when origin-reflecting.

## Anti-patterns to flag

- **Removing** security headers to “fix” broken assets without tightening CSP or using **nonces**.
- **Stack traces** or **X-Powered-By** / **Server** versions in **production** (information disclosure).

## Verification

- `curl -I` against staging/prod or local; **OWASP** Secure Headers **project** as reference; note **dev** may differ from **prod** (call out the gap).
