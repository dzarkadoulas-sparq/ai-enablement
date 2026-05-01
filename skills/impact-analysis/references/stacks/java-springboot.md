# Impact analysis: Java + Spring Boot

## Tracing

- **Types** — **find usages** in **IDE**; **`@Autowired` / constructor** **injection** of **the** **bean** **type** you **change**.
- **Interfaces** — **all** **implementations**; **`@Service`** **name** if **injected** **by** **name** (rare).

## Dynamic

- **SpEL**; **`@Value("${...}")`** **if** you **rename** **config** **keys**; **`@ConditionalOnClass`**, **`@Entity`** **name** = **table** (DB **mapping**).
- **Native** **queries** **string** **in** **repository** **methods**.

## Web

- **`@RequestMapping` / `GetMapping`** **path** **arrays**; **OpenAPI** **tag** if **codegen** **downstream** **exists** **elsewhere**.

## Tools

- **IntelliJ** / **Eclipse** **“Find references”**; **`./gradlew :module:compileJava`**; **Grep** for **FQCN** in **non-Java** **config**.

## Modulith

- **Module** **boundaries** in **Gradle** **include**; **public** **API** **of** **module** **=** **small** **surface**—**trace** **across** **only** **exported** **packages** if **documented**.
