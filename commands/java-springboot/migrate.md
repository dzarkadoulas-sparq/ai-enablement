Create a new Flyway migration for: `$ARGUMENTS`.

1. **Pick the version** — look at `src/main/resources/db/migration/` and pick
   the next timestamp-based version: `V{yyyyMMddHHmm}`. Use the current UTC
   minute so versions sort correctly.

2. **Pick the filename** — `V{version}__{snake_case_description}.sql`
   (double underscore between version and description).
   - Example: `V202604221430__add_email_verified_to_users.sql`.

3. **Write the SQL** — Postgres dialect unless the project overrides it.
   - Use `IF NOT EXISTS` on `CREATE TABLE` / `CREATE INDEX` for idempotency
     where the project convention allows.
   - New tables must include: `id UUID PRIMARY KEY DEFAULT gen_random_uuid()`,
     `created_at TIMESTAMPTZ NOT NULL DEFAULT now()`, `updated_at TIMESTAMPTZ`,
     `deleted_at TIMESTAMPTZ`, `version BIGINT NOT NULL DEFAULT 0`.
   - `ALTER TABLE ... ADD COLUMN` with `NULL` first, then a data backfill,
     then `SET NOT NULL` — never add `NOT NULL` without a default or
     backfill on a large table.
   - No `DROP COLUMN` / `DROP TABLE` in the same migration that deploys
     new code. Deprecate first, drop in a later release.

4. **Do NOT modify existing migration files.** Flyway checksums them; editing
   a deployed file breaks every environment. If a change is needed, write a
   new migration.

5. **Verify** — `./gradlew flywayInfo` should list the new migration as
   `Pending`. `./gradlew flywayValidate` should pass.
   Do NOT run `flywayMigrate` automatically — let the developer apply it.

6. **Update the JPA entity** if the migration changes a table the app
   reads/writes. Adjust the entity class accordingly. Do not introduce
   breaking field removals without confirming with me.

7. **Summary** — filename, version, what it does in one sentence,
   entity changes (if any), and how to roll forward (never roll back in
   production — forward migration only).
