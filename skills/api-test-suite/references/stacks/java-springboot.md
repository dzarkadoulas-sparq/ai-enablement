# API test suite: Java + Spring Boot

## Discovery

- **OpenAPI** **(springdoc-openapi,** **Spring** **Data** **REST,** **etc.):** **use** **spec** from **`/v3/api-docs`** **(or** **yaml** **export** **in** **CI**).
- **Code:** `@GetMapping` `@PostMapping` on **@RestController** + **class-level** **@RequestMapping** **prefix**—**compose** **full** **path**.

## Test harness

- **MockMvc** for **unit**-**style** **slice**; `@SpringBootTest(webEnvironment = RANDOM_PORT)` + **TestRestTemplate** or **WebTestClient** (WebFlux) for **real** **port** **(integration**).
- **@AutoConfigureMockMvc** + **security** **mock** for **A/Z** **without** **full** **IdP** **(per** **repo** **pattern**).

## Schema

- **Assert** **JSON** **with** **JsonPath** or **JSONassert**; **align** **error** **format** with **@ControllerAdvice** **handler**.

## Postman

- **Import** **OpenAPI** **3** from **springdoc** **export**; **environments** **for** **baseUrl** + **Bearer** **from** **test** **IdP** **(Keycloak,** **etc.)** if **dev** **script** **exports** **token**.

## Coverage

- **List** **endpoints** from **OpenAPI** **path** **items** **vs** **@WebMvcTest** / **integration** **test** **class** **names** **(heuristic** **match** **per** **resource** **controller**).
