Audit caching + revalidation in the current branch's diff.

Next.js caching is easy to get subtly wrong. This command verifies that
every `fetch`, `unstable_cache`, and `revalidate*` call in the diff is
intentional and correct.

1. **Scope** — `git diff origin/develop...HEAD -- 'app/**/*.ts*' 'features/**/*.ts*' 'lib/**/*.ts*'`.

2. **For every `fetch(...)` in the diff (server-side only)**, confirm:
   - Explicit `cache: 'force-cache' | 'no-store' | 'default'` OR
     `next: { revalidate: N }` OR `next: { tags: [...] }`.
   - Tags are specific (`\`user-\${id}\``), not generic (`'users'` for a
     single-user read).
   - If the fetch takes user input in the URL, `cache: 'no-store'` or
     per-user tag — otherwise one user's cache can serve another's data.
   - `cookies()` / `headers()` aren't read inside the fetch helper unless
     the route is intentionally dynamic.

3. **For every Server Action in the diff**, confirm:
   - `revalidateTag(...)` / `revalidatePath(...)` after every mutation.
   - The tag matches what the read-side fetch declared.
   - No `revalidatePath('/')` or `revalidatePath('/', 'layout')` unless
     absolutely intended — those invalidate everything downstream.

4. **For every `unstable_cache(...)` wrapper**, confirm:
   - Explicit `tags` array.
   - Explicit `revalidate` duration.
   - Key function captures every variable that affects output.

5. **For every route with `export const dynamic = 'force-dynamic'` or
   `export const revalidate = 0`**, confirm there is a comment on the same
   line (or within 2 lines) explaining why per-request dynamic behavior is
   needed. If not, it's probably wrong.

6. **For every `generateStaticParams`**, confirm the returned shape matches
   the dynamic segments and the fetch inside `page.tsx` uses the same
   cache key strategy.

7. **Output**:

   ```
   [SEVERITY] [FILE:LINE] [CATEGORY] — one-line issue
     Impact: ... (stale data? over-invalidation? leaked per-user state?)
     Fix: ... (specific tag to add/use, or flag to set)
   ```

   Severity here: `CRITICAL` if it causes cross-user data leakage;
   `HIGH` if stale data is user-visible; `MEDIUM` if it's over-
   invalidation (slow, not wrong); `LOW` for style issues.

8. **No silent fixes** — report only. I'll triage.
