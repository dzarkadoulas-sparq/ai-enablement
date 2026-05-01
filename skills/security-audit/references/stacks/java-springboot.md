# Security audit: Java + Spring Boot

## Read first

- `references/language-specific/java-spring-security.md`
- `commands/java-springboot/security-scan.md` (ai-enablement)

## Dependency scan

- `./gradlew dependencyCheckAnalyze` (OWASP **Dependency-Check** when configured); **Trivy/Snyk** in CI if present. Review **transitive** matches on **touched** libs.

## Code scope (typical)

- `*Controller*`, `*Config*`, `*Security*`, DTOs, JPA `nativeQuery`, `application*.yml`.

## Focus

- **SecurityFilterChain** (Spring Security 6) — `authorizeHttpRequests`, **CSRF** mode, **CORS** `allowedOriginPatterns` vs `*`.
- **@PreAuthorize**; **AOP** auth on **new** public beans.
- **Actuator** **exposure**; **Jackson** polymorphic defaults.

## Build

- `./gradlew check` (SpotBugs, PMD, Checkstyle) for **static** **security** rules the project enabled.
