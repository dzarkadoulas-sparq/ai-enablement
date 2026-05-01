# Architecture migration (layers, monolith → modules, strangler)

## When to use

- Moving **cross-cutting** concerns (auth, logging) to new homes; adding a new **bounded context**; or **strangling** legacy (HTTP facade, adapter over old storage).

## Patterns (short)

- **Strangler fig** — add the new path (feature flag, route, or job), migrate **traffic** or **readers** in waves, retire the old path only when **unreferenced** and (if applicable) **metrics** look good. Use `impact-analysis` to answer “who still calls the old code?”
- **Adapter** at the old/new boundary: old DTO in, new model in. Stops old types from leaking into the new core and vice versa.
- **Anti-corruption layer (DDD)** — wrap a messy external or legacy API in a module with a clean **inward**-facing API.

## Phasing

- **Phase 1:** New abstraction + **one** production consumer (or shadow read if you must not cut over yet).
- **Later phases:** Move more call sites; add **deprecations** (docs, OpenAPI `@deprecated`, logs when old path is used).
- **Final:** Remove old code; re-scan for string-based references and config (`impact-analysis` dynamic section).

## Tests

- **Contract** or **integration** tests at the new boundary. For **data** backfills or **dual** read, document rollback and have a **reversible** or replayable migration story.

## Biggest pitfalls

- **One-shot** cutover with no rollback. **Hidden** consumers: queues, events, cron, and **config** that embeds class names or paths—search beyond static imports.
