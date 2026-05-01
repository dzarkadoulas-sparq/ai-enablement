Create a new Prisma migration for: `$ARGUMENTS`.

1. **Edit `prisma/schema.prisma`** first — add the model / column / index /
   relation you need. Do NOT generate a migration from an unchanged schema.

2. **Generate the migration SQL (without applying)**

   ```
   pnpm prisma migrate dev --name <snake_case_description> --create-only
   ```

   This produces `prisma/migrations/<timestamp>_<name>/migration.sql`
   without applying it. Review it before committing.

3. **Review the SQL**
   - Non-null on existing table? Split across migrations: add nullable,
     backfill in a data migration, set NOT NULL.
   - Indexes on large tables: rewrite to `CREATE INDEX CONCURRENTLY`
     (Postgres) and mark the migration as non-transactional per project
     convention. Prisma won't do this automatically.
   - No `DROP` in the same deploy as new code referencing the column.
     Deprecate first, drop later.
   - Foreign keys: explicit `ON DELETE` behavior. Default is `NO ACTION`
     but state it explicitly.

4. **Do NOT modify a migration that's been applied above dev.** Create a
   new one.

5. **Apply locally to smoke-test**

   ```
   pnpm prisma migrate dev
   pnpm test:integration
   ```

6. **Regenerate the Prisma client**

   ```
   pnpm prisma generate
   ```

   Commit the updated schema + migration folder together. Do NOT commit
   the generated client output (it's in `node_modules/.prisma/client/`).

7. **Summary** — migration name, intent, any backfill that must follow
   in a later migration, and whether any app code needs a matching change
   in this PR (usually yes — include both).
