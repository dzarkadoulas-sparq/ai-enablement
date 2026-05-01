# OWASP Top 10 (2021) — review prompts

Use as a **structured pass** over the codebase or the **diff**. Map findings to **Axx** for the report.

## A01 — Broken Access Control

- Missing **authz** on new routes/APIs; **IDOR** (object IDs without tenant/user check); **path** vs **role** matrix; **CORS** allowing credentialed cross-origin to untrusted origins.
- **Directory listing**, **debug** endpoints in prod, **forced browsing** to admin URLs.

## A02 — Cryptographic Failures

- **Weak** or **hardcoded** secrets; **symmetric** keys in source; **HTTP** for auth; **MD5/SHA1** for passwords; **missing** TLS for external calls (config review).

## A03 — Injection

- **SQL** / **NoSQL** / **LDAP** / **OS command** / **template** injection (string concat, `fmt` with user input, unsafe `eval`). **ORM** raw methods with interpolations.

## A04 — Insecure Design

- **Threat modeling** gap: new flows without abuse cases (e.g. password reset, file share, webhooks). Flag **design** issues as **MEDIUM+** with mitigation ideas.

## A05 — Security Misconfiguration

- **Default** creds, **stack traces** in errors, **verbose** errors in prod, **open** admin interfaces, **unnecessary** HTTP methods, **debug=true**.

## A06 — Vulnerable and Outdated Components

- **Dependency** scans (per stack); **transitive** HIGH/CRIT; **unmaintained** packages.

## A07 — Identification and Authentication Failures

- **Session** fixation, **weak** session config, **JWT** `none` / `alg` confusion, **password** policy, **MFA** gaps on sensitive actions, **credential stuffing** resistance.

## A08 — Software and Data Integrity Failures

- **Unsigned** updates, **CI** without integrity checks, **insecure** deserialization (pickle, YAML `!!python/object`), **supply chain** (install scripts).

## A09 — Security Logging and Monitoring Failures

- **No** audit log for **admin** actions; **PII** in logs; **missing** correlation id for security incidents (note as gap, not always code fix).

## A10 — Server-Side Request Forgery (SSRF)

- **User-supplied** URL fetch; **open** redirects; **webhook** URL validation; **IP** allowlist missing for internal service calls.

---

**Client-heavy stacks** (React/Next/Angular): emphasize **A03** (XSS), **A05** (CSP, headers), **A01** (what the browser can call), **A02** (exposed env). **API stacks**: emphasize **A01**, **A03**, **A07**, **A10**, **A06**.
