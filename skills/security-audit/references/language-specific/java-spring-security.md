# Java + Spring Boot — security focus

## Web layer

- **Spring Security** — **authorizeHttpRequests** / **antMatchers** / `requestMatchers`: **default** deny; **permitAll** only for public API/docs.
- **Method security** — `@PreAuthorize` / `@PostAuthorize` on service methods for **row-level** rules when **URLs** are not enough.

## Injection

- **JPA** — `nativeQuery` with concat; use **parameters**. **JPQL** string building with user input.
- **SpEL** in `@Value` or **custom** eval with user data.

## Deserialization

- **Jackson** — **default typing** / `ObjectMapper` **enableDefaultTyping** (older patterns) — dangerous; **@JsonTypeInfo** on polymorphic DTOs from **untrusted** input.

## Actuator / management

- **/actuator/** exposed on **public** without auth — **CRITICAL** in prod; **health** may be public, **env** and **heapdump** must not.

## Sessions

- **SameSite**, **httpOnly**, **secure**; **CSRF** for state-changing browser requests if session-based (Spring Security default when needed).

## Tools

- **OWASP Dependency-Check** (`./gradlew dependencyCheckAnalyze` when configured); **SpotBugs** / **PMD** security rules.
- **Snyk** / **Trivy** in CI if present.

## See also

`commands/java-springboot/security-scan.md` in ai-enablement; Spring Security reference docs for **CORS**, **CSP** headers in **SecurityFilterChain**.
