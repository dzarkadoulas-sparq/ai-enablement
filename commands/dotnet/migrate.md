Create a new EF Core migration for: `$ARGUMENTS`.

1. **Pick the migration name** — PascalCase describing the change.
   Examples: `AddEmailVerifiedToUsers`, `CreateOrdersTable`,
   `AddIndexOnOrdersCustomerId`.

2. **Generate the migration**

   ```
   dotnet ef migrations add <Name> \
     -p src/Infrastructure \
     -s src/Api \
     -o Persistence/Migrations
   ```

   If the project uses a different layout, match the existing one.

3. **Review the generated migration**
   - Check the `Up` method is correct and won't lock a large table for
     an unacceptable duration.
   - For non-null columns on existing tables: split into (a) add nullable,
     (b) backfill, (c) set NOT NULL — usually across multiple migrations.
   - Postgres: `CREATE INDEX CONCURRENTLY` goes in a separate migration
     with `migrationBuilder.Sql(...)` and the `--idempotent` script path,
     since EF can't do it inside a transaction.
   - No `DROP COLUMN` / `DROP TABLE` in the same deploy as app changes —
     deprecate first, drop later.

4. **Review the generated `Down` method**
   - Data migrations should have a safe rollback OR explicitly throw if
     rollback would lose data. Don't silently drop user data.

5. **Do NOT modify migrations that have been applied above dev.** If a
   change is needed, add a new migration.

6. **Apply locally for smoke test**

   ```
   dotnet ef database update -p src/Infrastructure -s src/Api
   ```

   Run the integration tests after the migration applies.

7. **Update `IEntityTypeConfiguration<T>`** for any schema change. Do not
   inline fluent API in `OnModelCreating`.

8. **Summary** — migration name, Up/Down intent, any data backfill needed
   in a later step, and the next forward migration required (if this one
   is part of a deprecation sequence).
